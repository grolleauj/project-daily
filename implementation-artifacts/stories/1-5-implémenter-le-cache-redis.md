---
name: Implémenter le cache Redis
id: 1-5-implémenter-le-cache-redis
epic: epic-1
story_type: backend
priority: high
estimation: M
dependencies: [1-1, 1-2, 1-3, 1-4]
status: ready-for-dev
created: 2026-08-16
updated: 2026-08-16
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/implementation-artifacts/epic-1-context.md
---

# Story 1.5: Implémenter le cache Redis

## Contexte
Cette story fait partie de **l'Épic 1 (Context Engine)**. Son objectif est d'implémenter un **système de cache Redis** pour stocker les données normalisées des providers (Atmo France, RTE, Hub'Eau, OpenStreetMap), conformément à **FR-3 (Mise en cache des données normalisées)** et **AD-7 (Caching Strategy)**.

Le cache permet de :
- **Réduire les appels aux APIs externes** (rate limits, coûts, latence).
- **Fournir un fallback** en cas d'indisponibilité d'un provider (ex: API Hub'Eau en panne).
- **Améliorer les performances** en évitant des requêtes répétées pour les mêmes données.

---

## Exigences Fonctionnelles
- **FR-3**: Mise en cache des données normalisées dans Redis (TTL: 5 min pour le temps réel, 1h pour les données statiques).
- **FR-5**: Gestion des erreurs des Providers (fallback vers le cache, alertes pour les pannes critiques).
- **AD-7**: Cache TTL: 5 minutes pour les données temps réel (ex: qualité de l'air), 1 heure pour les données statiques (ex: phases lunaires).

---

## Critères d'Acceptation
1. **Stockage des données** :
   - [ ] Les observations normalisées (`Observation`) sont stockées dans Redis avec une **clé unique** (ex: `provider:location` ou `provider:station_id`).
   - [ ] Le TTL est configuré selon le type de données :
     - **5 minutes** pour les données temps réel (Atmo France, RTE).
     - **1 heure** pour les données semi-statiques (Hub'Eau).
     - **24 heures** pour les données statiques (OpenStreetMap).

2. **Récupération des données** :
   - [ ] `get_from_cache(provider: str, key: str)` récupère une observation depuis Redis.
   - [ ] Retourne `None` si la clé n'existe pas ou a expiré.

3. **Fallback en cas d'échec** :
   - [ ] Si un provider échoue (ex: `ProviderError`), le `ContextBuilder` utilise **la dernière valeur valide en cache** pour ce provider.
   - [ ] Si aucune valeur n'est en cache, une **valeur par défaut** est utilisée (ex: `Observation` avec `quality=0.0`).

4. **Gestion des erreurs** :
   - [ ] Les erreurs de connexion à Redis sont **loguées** mais n'interrompent pas l'exécution du `ContextBuilder`.
   - [ ] Si Redis est indisponible, les données sont récupérées directement depuis les providers (sans cache).

5. **Sécurité** :
   - [ ] Les **clés Redis** ne contiennent **aucune information sensible** (ex: API keys).
   - [ ] Utilisation d'un **prefixe** pour les clés (ex: `almane:context:atmo-france:69003`).

6. **Intégration avec le Context Engine** :
   - [ ] Le cache est **utilisé par `ContextBuilder`** pour chaque provider.
   - [ ] Les données sont **mises en cache après normalisation** (après `normalize()`).

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/context_engine/cache/redis_cache.py` | Module de cache Redis | À créer |
| `backend/engines/context_engine/context_builder.py` | Intégration du cache | À modifier |
| `backend/core/config.py` | Configuration Redis (URL, port) | À modifier |
| `tests/unit/test_redis_cache.py` | Tests unitaires | À créer |
| `.env.example` | Variables d'environnement Redis | À modifier |

### Structure du module de cache (`redis_cache.py`)
```python
import json
from datetime import timedelta
from typing import Optional, Dict, Any
import redis.asyncio as redis
from ..models import Observation


class RedisCache:
    """Gère le cache Redis pour les observations normalisées."""

    def __init__(self, redis_url: str = "redis://localhost:6379/0", prefix: str = "almane:context"):
        self.redis_url = redis_url
        self.prefix = prefix
        self.client: Optional[redis.Redis] = None

    async def connect(self) -> None:
        """Établit une connexion asynchrone à Redis."""
        self.client = redis.from_url(self.redis_url, decode_responses=True)

    async def close(self) -> None:
        """Ferme la connexion Redis."""
        if self.client:
            await self.client.close()

    def _make_key(self, provider: str, key: str) -> str:
        """Génère une clé Redis unique (ex: 'almane:context:atmo-france:69003')."""
        return f"{self.prefix}:{provider}:{key}"

    async def set(self, provider: str, key: str, observation: Observation, ttl: int) -> bool:
        """
        Stocke une observation dans Redis avec un TTL (en secondes).
        Exemple : set("atmo-france", "69003", observation, 300) → TTL = 5 min.
        """
        if not self.client:
            await self.connect()

        cache_key = self._make_key(provider, key)
        # Sérialisation de l'observation en JSON
        data = observation.model_dump_json()
        await self.client.setex(cache_key, ttl, data)
        return True

    async def get(self, provider: str, key: str) -> Optional[Observation]:
        """
        Récupère une observation depuis Redis.
        Retourne None si la clé n'existe pas ou a expiré.
        """
        if not self.client:
            await self.connect()

        cache_key = self._make_key(provider, key)
        data = await self.client.get(cache_key)
        if not data:
            return None

        # Désérialisation JSON → Observation
        return Observation.model_validate_json(data)

    async def delete(self, provider: str, key: str) -> bool:
        """Supprime une entrée du cache."""
        if not self.client:
            await self.connect()

        cache_key = self._make_key(provider, key)
        await self.client.delete(cache_key)
        return True

    async def clear(self) -> bool:
        """Supprime toutes les entrées du cache pour ce préfixe."""
        if not self.client:
            await self.connect()

        # Récupérer toutes les clés avec le préfixe
        keys = await self.client.keys(f"{self.prefix}:*")
        if keys:
            await self.client.delete(*keys)
        return True
```

### Intégration avec `ContextBuilder`
Dans `backend/engines/context_engine/context_builder.py` :
```python
from backend.core.config import settings
from .cache.redis_cache import RedisCache
from .providers.atmo import AtmoFranceAdapter
from .providers.rte import RTEAdapter
from .providers.hubeau import HubEauAdapter
from .providers.osm import OSMAdapter
from .adapter import ProviderError, NormalizationError, ValidationError
from .models import UnifiedContext, Observation


class ContextBuilder:
    def __init__(self):
        self.adapters = {
            "atmo-france": AtmoFranceAdapter(
                api_key=settings.ATMO_FRANCE_API_KEY,
                endpoint=settings.ATMO_FRANCE_ENDPOINT
            ),
            "rte": RTEAdapter(
                api_key=settings.RTE_API_KEY,
                endpoint=settings.RTE_ENDPOINT
            ),
            "hubeau": HubEauAdapter(
                api_key=settings.HUBEAU_API_KEY,
                endpoint=settings.HUBEAU_ENDPOINT
            ),
            "openstreetmap": OSMAdapter(
                endpoint=settings.OSM_ENDPOINT
            ),
        }
        self.cache = RedisCache(redis_url=settings.REDIS_URL)

    async def build_unified_context(self, location: str = "69003") -> UnifiedContext:
        await self.cache.connect()
        observations = []

        for provider_name, adapter in self.adapters.items():
            try:
                # Vérifier le cache en premier
                cached_observation = await self.cache.get(provider_name, location)
                if cached_observation:
                    observations.append(cached_observation)
                    continue

                # Si pas en cache, récupérer depuis le provider
                if provider_name == "openstreetmap":
                    raw_data = await adapter.fetch(query=location if location else None)
                else:
                    raw_data = await adapter.fetch(code_commune=location if location else None)

                observation = adapter.normalize(raw_data)
                if adapter.validate(observation):
                    # Mettre en cache avec le TTL approprié
                    ttl = self._get_ttl(provider_name)
                    await self.cache.set(provider_name, location, observation, ttl)
                    observations.append(observation)

            except (ProviderError, NormalizationError, ValidationError) as e:
                print(f"Error with {provider_name}: {e}")
                # Essayer de récupérer depuis le cache en fallback
                cached_observation = await self.cache.get(provider_name, location)
                if cached_observation:
                    observations.append(cached_observation)

            finally:
                await adapter.close()

        await self.cache.close()
        return UnifiedContext(observations=observations, timestamp=datetime.now(timezone.utc))

    def _get_ttl(self, provider_name: str) -> int:
        """Retourne le TTL en secondes selon le provider."""
        ttl_map = {
            "atmo-france": 300,    # 5 minutes (temps réel)
            "rte": 300,            # 5 minutes (temps réel)
            "hubeau": 3600,        # 1 heure (semi-statique)
            "openstreetmap": 86400 # 24 heures (statique)
        }
        return ttl_map.get(provider_name, 300)  # Default: 5 minutes
```

---

## Tests Unitaires
Créer un fichier `tests/unit/test_redis_cache.py` avec les tests suivants :

```python
import pytest
from datetime import datetime, timedelta
from unittest.mock import AsyncMock, patch, MagicMock
import sys
import os

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '../../backend'))

from backend.engines.context_engine.cache.redis_cache import RedisCache
from backend.engines.context_engine.models import Observation


@pytest.fixture
def redis_cache():
    return RedisCache(redis_url="redis://localhost:6379/0", prefix="almane:test")


@pytest.fixture
def mock_observation():
    return Observation(
        provider="atmo-france",
        observedAt=datetime.now(),
        retrievedAt=datetime.now(),
        expiresAt=datetime.now() + timedelta(hours=1),
        quality=0.95,
        value={"aqi": 42, "polluant": "PM2.5"}
    )


# Test de connexion
@pytest.mark.asyncio
async def test_connect(redis_cache):
    with patch('backend.engines.context_engine.cache.redis_cache.redis.from_url') as mock_from_url:
        mock_client = AsyncMock()
        mock_from_url.return_value = mock_client
        await redis_cache.connect()
        assert redis_cache.client == mock_client


# Test de fermeture
@pytest.mark.asyncio
async def test_close(redis_cache):
    mock_client = AsyncMock()
    redis_cache.client = mock_client
    await redis_cache.close()
    mock_client.close.assert_awaited_once()


# Test de génération de clé
@pytest.mark.asyncio
async def test_make_key(redis_cache):
    key = redis_cache._make_key("atmo-france", "69003")
    assert key == "almane:test:atmo-france:69003"


# Test de stockage dans le cache
@pytest.mark.asyncio
async def test_set(redis_cache, mock_observation):
    with patch('backend.engines.context_engine.cache.redis_cache.redis.from_url') as mock_from_url:
        mock_client = AsyncMock()
        mock_from_url.return_value = mock_client
        await redis_cache.connect()

        await redis_cache.set("atmo-france", "69003", mock_observation, 300)
        mock_client.setex.assert_awaited_once()


# Test de récupération depuis le cache
@pytest.mark.asyncio
async def test_get(redis_cache, mock_observation):
    with patch('backend.engines.context_engine.cache.redis_cache.redis.from_url') as mock_from_url:
        mock_client = AsyncMock()
        mock_client.get = AsyncMock(return_value=mock_observation.model_dump_json())
        mock_from_url.return_value = mock_client
        await redis_cache.connect()

        result = await redis_cache.get("atmo-france", "69003")
        assert result.provider == "atmo-france"
        assert result.value["aqi"] == 42


# Test de récupération depuis le cache (clé inexistante)
@pytest.mark.asyncio
async def test_get_missing_key(redis_cache):
    with patch('backend.engines.context_engine.cache.redis_cache.redis.from_url') as mock_from_url:
        mock_client = AsyncMock()
        mock_client.get = AsyncMock(return_value=None)
        mock_from_url.return_value = mock_client
        await redis_cache.connect()

        result = await redis_cache.get("atmo-france", "69003")
        assert result is None


# Test de suppression du cache
@pytest.mark.asyncio
async def test_delete(redis_cache):
    with patch('backend.engines.context_engine.cache.redis_cache.redis.from_url') as mock_from_url:
        mock_client = AsyncMock()
        mock_from_url.return_value = mock_client
        await redis_cache.connect()

        await redis_cache.delete("atmo-france", "69003")
        mock_client.delete.assert_awaited_once()


# Test de vidage du cache
@pytest.mark.asyncio
async def test_clear(redis_cache):
    with patch('backend.engines.context_engine.cache.redis_cache.redis.from_url') as mock_from_url:
        mock_client = AsyncMock()
        mock_client.keys = AsyncMock(return_value=["almane:test:atmo-france:69003", "almane:test:rte:69003"])
        mock_from_url.return_value = mock_client
        await redis_cache.connect()

        await redis_cache.clear()
        mock_client.delete.assert_awaited_once()
```

---

## Configuration Requise
Ajouter les variables d'environnement suivantes dans `.env` :
```env
# URL de connexion à Redis (format: redis://host:port/db)
REDIS_URL=redis://localhost:6379/0
```

---

## Dépendances
- **Python 3.11+**
- **Librairies** :
  - `redis[asyncio]` (pour le client Redis asynchrone)

Installer avec :
```bash
pip install redis[asyncio]
```

---

## Notes
- **Fallback** : En cas d'échec de Redis, les données sont récupérées directement depuis les providers (sans cache).
- **TTL** : Les TTL sont configurés selon le type de données (temps réel vs. statiques).
- **Sérialisation** : Les objets `Observation` sont sérialisés en JSON pour le stockage dans Redis.
- **Prefixe des clés** : Utiliser un préfixe (ex: `almane:context`) pour éviter les conflits avec d'autres services.

---

## Ressources
- [Documentation Redis Python](https://redis-py.readthedocs.io/)
- [PRD Almanéa - FR-3](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AD-7](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
