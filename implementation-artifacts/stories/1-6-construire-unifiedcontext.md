---
name: Construire UnifiedContext
id: 1-6-construire-unifiedcontext
epic: epic-1
story_type: backend
priority: high
estimation: M
dependencies: [1-1, 1-2, 1-3, 1-4, 1-5]
status: ready-for-dev
created: 2026-08-16
updated: 2026-08-16
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/implementation-artifacts/epic-1-context.md
---

# Story 1.6: Construire `UnifiedContext`

## Contexte
Cette story fait partie de **l'Épic 1 (Context Engine)**. Son objectif est de **construire et finaliser** le modèle `UnifiedContext` pour agréger les observations normalisées des différents providers (Atmo France, RTE, Hub'Eau, OpenStreetMap), conformément à **FR-4 (Enrichissement du contexte avec des données dérivées)** et **AD-4 (Unified Context Contract)**.

Le `UnifiedContext` est le **contrat standardisé** qui sera utilisé par le **Recommendation Engine** (Épic 2) pour générer des recommandations.

---

## Exigences Fonctionnelles
- **FR-2**: Normalisation des données brutes en format `Observation`.
- **FR-4**: Enrichissement du contexte avec des données dérivées (ex: indice de qualité de l'air global).
- **AD-4**: `UnifiedContext` inclut `observations` (list of `Observation`), `user_profile` (optionnel), `timestamp`.

---

## Critères d'Acceptation
1. **Structure du `UnifiedContext`** :
   - [ ] `UnifiedContext` contient une liste d'`Observation` (une par provider).
   - [ ] `UnifiedContext` contient un `timestamp` (horodatage de la création du contexte).
   - [ ] `UnifiedContext` peut inclure un `user_profile` (optionnel, pour les données personnalisées).

2. **Agrégation des observations** :
   - [ ] `ContextBuilder.build_unified_context()` retourne un `UnifiedContext` valide.
   - [ ] Les observations sont **filtrées** pour exclure les données invalides (ex: `quality < 0.5`).

3. **Enrichissement du contexte** :
   - [ ] Ajouter des **données dérivées** dans `UnifiedContext` (ex: `global_aqi`, `energy_mix_summary`).
   - [ ] Calculer un **score global** à partir des observations (ex: moyenne pondérée des AQI).

4. **Sérialisation** :
   - [ ] `UnifiedContext` peut être **sérialisé en JSON** pour le stockage ou la transmission.
   - [ ] `UnifiedContext` peut être **désérialisé depuis JSON**.

5. **Validation** :
   - [ ] `UnifiedContext` valide que toutes les `Observation` ont un `provider` unique.
   - [ ] `UnifiedContext` valide que le `timestamp` est récent (ex: < 1 heure).

6. **Intégration avec le Context Engine** :
   - [ ] `UnifiedContext` est **utilisé par le Recommendation Engine** (Épic 2).
   - [ ] `UnifiedContext` est **stocké dans Redis** (optionnel, pour le cache global).

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/context_engine/models.py` | Modèle `UnifiedContext` | À modifier |
| `backend/engines/context_engine/context_builder.py` | Construction du `UnifiedContext` | À modifier |
| `tests/unit/test_unified_context.py` | Tests unitaires | À créer |

### Structure du `UnifiedContext` (`models.py`)
```python
from datetime import datetime
from typing import Dict, Any, List, Optional
from pydantic import BaseModel, Field, validator


class Observation(BaseModel):
    provider: str = Field(..., description="Nom du provider (ex: 'atmo-france')")
    observedAt: datetime = Field(..., description="Horodatage de l'observation")
    retrievedAt: datetime = Field(..., description="Horodatage de la récupération")
    expiresAt: datetime = Field(..., description="Date d'expiration")
    quality: float = Field(..., ge=0.0, le=1.0, description="Score de qualité (0-1)")
    value: Dict[str, Any] = Field(..., description="Données normalisées")


class UserProfile(BaseModel):
    """Profil utilisateur optionnel pour personnaliser le contexte."""
    user_id: str = Field(..., description="ID unique de l'utilisateur")
    location: str = Field(..., description="Localisation par défaut (ex: code commune)")
    preferences: Dict[str, Any] = Field(default_factory=dict, description="Préférences utilisateur")


class UnifiedContext(BaseModel):
    """
    Contexte unifié pour le Recommendation Engine.
    Contient une liste d'observations normalisées et des données dérivées.
    """
    observations: List[Observation] = Field(..., description="Liste des observations des providers")
    timestamp: datetime = Field(..., description="Horodatage de la création du contexte")
    user_profile: Optional[UserProfile] = Field(None, description="Profil utilisateur (optionnel)")
    derived_data: Dict[str, Any] = Field(
        default_factory=dict,
        description="Données dérivées (ex: global_aqi, energy_mix_summary)"
    )

    @validator('observations')
    def validate_observations(cls, v):
        """Valide que toutes les observations ont un provider unique."""
        providers = [obs.provider for obs in v]
        if len(providers) != len(set(providers)):
            raise ValueError("Duplicate provider in observations")
        return v

    @validator('timestamp')
    def validate_timestamp(cls, v):
        """Valide que le timestamp est récent (moins de 1 heure)."""
        if (datetime.now() - v).total_seconds() > 3600:
            raise ValueError("Timestamp is too old (max 1 hour)")
        return v

    def get_observation(self, provider: str) -> Optional[Observation]:
        """Récupère une observation par son provider."""
        for obs in self.observations:
            if obs.provider == provider:
                return obs
        return None

    def get_value(self, provider: str, key: str, default: Any = None) -> Any:
        """Récupère une valeur spécifique depuis une observation."""
        obs = self.get_observation(provider)
        if obs:
            return obs.value.get(key, default)
        return default

    def calculate_global_aqi(self) -> Optional[float]:
        """Calcule un AQI global à partir des observations disponibles."""
        aqi_values = []
        for obs in self.observations:
            if obs.provider == "atmo-france" and "aqi" in obs.value:
                aqi_values.append(obs.value["aqi"])
        if aqi_values:
            return sum(aqi_values) / len(aqi_values)
        return None

    def calculate_energy_mix_summary(self) -> Optional[Dict[str, float]]:
        """Calcule un résumé du mix énergétique à partir des observations RTE."""
        for obs in self.observations:
            if obs.provider == "rte" and "mix" in obs.value:
                return obs.value["mix"]
        return None
```

### Mise à jour de `ContextBuilder` (`context_builder.py`)
```python
from datetime import datetime, timezone
from typing import Optional

from backend.core.config import settings
from .cache.redis_cache import RedisCache
from .providers.atmo import AtmoFranceAdapter
from .providers.rte import RTEAdapter
from .providers.hubeau import HubEauAdapter
from .providers.osm import OSMAdapter
from .adapter import ProviderError, NormalizationError, ValidationError
from .models import UnifiedContext, Observation, UserProfile


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

    async def build_unified_context(
        self, 
        location: str = "69003", 
        user_profile: Optional[UserProfile] = None
    ) -> UnifiedContext:
        """
        Construit un UnifiedContext à partir des données des providers.
        - Récupère les données depuis les providers ou le cache.
        - Normalise et valide les données.
        - Ajoute des données dérivées (ex: global_aqi).
        """
        await self.cache.connect()
        observations = []

        for provider_name, adapter in self.adapters.items():
            try:
                cached_observation = await self.cache.get(provider_name, location)
                if cached_observation:
                    observations.append(cached_observation)
                    continue

                if provider_name == "openstreetmap":
                    raw_data = await adapter.fetch(query=location if location else None)
                else:
                    raw_data = await adapter.fetch(code_commune=location if location else None)

                observation = adapter.normalize(raw_data)
                if adapter.validate(observation):
                    ttl = self._get_ttl(provider_name)
                    await self.cache.set(provider_name, location, observation, ttl)
                    observations.append(observation)

            except (ProviderError, NormalizationError, ValidationError) as e:
                print(f"Error with {provider_name}: {e}")
                cached_observation = await self.cache.get(provider_name, location)
                if cached_observation:
                    observations.append(cached_observation)

            finally:
                await adapter.close()

        await self.cache.close()

        # Créer le UnifiedContext avec les observations et les données dérivées
        unified_context = UnifiedContext(
            observations=observations,
            timestamp=datetime.now(timezone.utc),
            user_profile=user_profile,
            derived_data=self._calculate_derived_data(observations)
        )

        return unified_context

    def _get_ttl(self, provider_name: str) -> int:
        """Retourne le TTL en secondes selon le provider."""
        ttl_map = {
            "atmo-france": 300,
            "rte": 300,
            "hubeau": 3600,
            "openstreetmap": 86400
        }
        return ttl_map.get(provider_name, 300)

    def _calculate_derived_data(self, observations: List[Observation]) -> Dict[str, Any]:
        """Calcule les données dérivées à partir des observations."""
        derived_data = {}

        # Calculer l'AQI global
        aqi_values = []
        for obs in observations:
            if obs.provider == "atmo-france" and "aqi" in obs.value:
                aqi_values.append(obs.value["aqi"])
        if aqi_values:
            derived_data["global_aqi"] = sum(aqi_values) / len(aqi_values)

        # Calculer le résumé du mix énergétique
        for obs in observations:
            if obs.provider == "rte" and "mix" in obs.value:
                derived_data["energy_mix"] = obs.value["mix"]
                break

        # Calculer le niveau d'eau moyen (si plusieurs stations)
        water_levels = []
        for obs in observations:
            if obs.provider == "hubeau" and "niveau" in obs.value:
                water_levels.append(obs.value["niveau"])
        if water_levels:
            derived_data["avg_water_level"] = sum(water_levels) / len(water_levels)

        return derived_data
```

---

## Tests Unitaires
Créer un fichier `tests/unit/test_unified_context.py` avec les tests suivants :

```python
import pytest
from datetime import datetime, timedelta
from unittest.mock import AsyncMock, patch, MagicMock
import sys
import os

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '../../backend'))

from backend.engines.context_engine.models import UnifiedContext, Observation, UserProfile


@pytest.fixture
def mock_observations():
    return [
        Observation(
            provider="atmo-france",
            observedAt=datetime.now(),
            retrievedAt=datetime.now(),
            expiresAt=datetime.now() + timedelta(hours=1),
            quality=0.95,
            value={"aqi": 42, "polluant": "PM2.5"}
        ),
        Observation(
            provider="rte",
            observedAt=datetime.now(),
            retrievedAt=datetime.now(),
            expiresAt=datetime.now() + timedelta(hours=1),
            quality=0.95,
            value={"mix": {"nuclear": 0.5, "hydro": 0.3}, "co2": 10}
        ),
        Observation(
            provider="hubeau",
            observedAt=datetime.now(),
            retrievedAt=datetime.now(),
            expiresAt=datetime.now() + timedelta(hours=1),
            quality=0.95,
            value={"niveau": 120.5, "unite": "cm"}
        ),
    ]


@pytest.fixture
def mock_user_profile():
    return UserProfile(
        user_id="user123",
        location="69003",
        preferences={"notifications": True}
    )


# Test de création de UnifiedContext
def test_create_unified_context(mock_observations):
    context = UnifiedContext(
        observations=mock_observations,
        timestamp=datetime.now()
    )
    assert len(context.observations) == 3
    assert context.timestamp is not None


# Test de validation des observations uniques
def test_validate_duplicate_providers():
    observations = [
        Observation(
            provider="atmo-france",
            observedAt=datetime.now(),
            retrievedAt=datetime.now(),
            expiresAt=datetime.now() + timedelta(hours=1),
            quality=0.95,
            value={"aqi": 42}
        ),
        Observation(
            provider="atmo-france",  # Duplicate provider
            observedAt=datetime.now(),
            retrievedAt=datetime.now(),
            expiresAt=datetime.now() + timedelta(hours=1),
            quality=0.95,
            value={"aqi": 50}
        ),
    ]
    with pytest.raises(ValueError):
        UnifiedContext(observations=observations, timestamp=datetime.now())


# Test de validation du timestamp
@pytest.mark.asyncio
async def test_validate_old_timestamp():
    old_timestamp = datetime.now() - timedelta(hours=2)
    observations = [
        Observation(
            provider="atmo-france",
            observedAt=datetime.now(),
            retrievedAt=datetime.now(),
            expiresAt=datetime.now() + timedelta(hours=1),
            quality=0.95,
            value={"aqi": 42}
        )
    ]
    with pytest.raises(ValueError):
        UnifiedContext(observations=observations, timestamp=old_timestamp)


# Test de get_observation
def test_get_observation(mock_observations):
    context = UnifiedContext(
        observations=mock_observations,
        timestamp=datetime.now()
    )
    obs = context.get_observation("atmo-france")
    assert obs is not None
    assert obs.provider == "atmo-france"
    assert obs.value["aqi"] == 42


# Test de get_observation (provider inexistant)
def test_get_observation_missing(mock_observations):
    context = UnifiedContext(
        observations=mock_observations,
        timestamp=datetime.now()
    )
    obs = context.get_observation("unknown")
    assert obs is None


# Test de get_value
def test_get_value(mock_observations):
    context = UnifiedContext(
        observations=mock_observations,
        timestamp=datetime.now()
    )
    aqi = context.get_value("atmo-france", "aqi")
    assert aqi == 42


# Test de get_value (clé inexistante)
def test_get_value_missing_key(mock_observations):
    context = UnifiedContext(
        observations=mock_observations,
        timestamp=datetime.now()
    )
    value = context.get_value("atmo-france", "unknown", default=0)
    assert value == 0


# Test de calculate_global_aqi
def test_calculate_global_aqi(mock_observations):
    context = UnifiedContext(
        observations=mock_observations,
        timestamp=datetime.now()
    )
    global_aqi = context.calculate_global_aqi()
    assert global_aqi == 42  # Seul Atmo France a un AQI dans les mocks


# Test de calculate_energy_mix_summary
def test_calculate_energy_mix_summary(mock_observations):
    context = UnifiedContext(
        observations=mock_observations,
        timestamp=datetime.now()
    )
    energy_mix = context.calculate_energy_mix_summary()
    assert energy_mix == {"nuclear": 0.5, "hydro": 0.3}


# Test de sérialisation JSON
def test_serialization(mock_observations):
    context = UnifiedContext(
        observations=mock_observations,
        timestamp=datetime.now()
    )
    json_data = context.model_dump_json()
    assert "atmo-france" in json_data
    assert "rte" in json_data


# Test de désérialisation JSON
def test_deserialization(mock_observations):
    context = UnifiedContext(
        observations=mock_observations,
        timestamp=datetime.now()
    )
    json_data = context.model_dump_json()
    restored_context = UnifiedContext.model_validate_json(json_data)
    assert len(restored_context.observations) == 3
    assert restored_context.observations[0].provider == "atmo-france"


# Test de UnifiedContext avec user_profile
def test_unified_context_with_user_profile(mock_observations, mock_user_profile):
    context = UnifiedContext(
        observations=mock_observations,
        timestamp=datetime.now(),
        user_profile=mock_user_profile
    )
    assert context.user_profile is not None
    assert context.user_profile.user_id == "user123"
    assert context.user_profile.location == "69003"
```

---

## Configuration Requise
Aucune configuration supplémentaire nécessaire. Utilise les variables existantes dans `.env`.

---

## Dépendances
- **Python 3.11+**
- **Librairies** :
  - `pydantic` (pour la validation des données)

Les dépendances sont déjà installées (voir `requirements.txt`).

---

## Notes
- **Données dérivées** : Les données comme `global_aqi` ou `energy_mix_summary` sont calculées à partir des observations et stockées dans `derived_data`.
- **Sérialisation** : `UnifiedContext` utilise Pydantic pour la sérialisation/désérialisation JSON.
- **Validation** : Les validateurs garantissent que les observations ont des `provider` uniques et que le `timestamp` est récent.
- **Extensibilité** : Le modèle `UnifiedContext` peut être étendu avec de nouveaux champs si nécessaire.

---

## Ressources
- [PRD Almanéa - FR-4](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AD-4](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
