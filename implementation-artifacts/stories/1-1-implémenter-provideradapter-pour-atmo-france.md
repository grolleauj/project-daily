---
name: Implémenter ProviderAdapter pour Atmo France
id: 1-1-implémenter-provideradapter-pour-atmo-france
epic: epic-1
story_type: backend
priority: high
estimation: M
dependencies: []
status: done
created: 2026-08-16
updated: 2026-08-16
---

# Story 1.1: Implémenter `ProviderAdapter` pour Atmo France

## Contexte
Cette story fait partie de **l'Épic 1 (Context Engine)**. Son objectif est de créer un **adapter dédié** pour récupérer, normaliser et valider les données de qualité de l'air depuis **Atmo France**, conformément à **AD-2 (Provider-Agnostic Adapters)** et **FR-1 (Collecte des données externes)**.

---

## Exigences Fonctionnelles
- **FR-1**: Collecte des données externes depuis les Providers via `ProviderAdapter`.
- **FR-2**: Normalisation des données brutes en format `Observation`.
- **AR-3**: Chaque provider a un adapter dédié implémentant `ProviderAdapter` (`fetch()`, `normalize()`, `validate()`).

---

## Critères d'Acceptation
1. **Récupération des données** :
   - [ ] `fetch()` appelle l'API Atmo France avec une requête HTTP valide.
   - [ ] Retourne les données brutes avec un statut HTTP 200.
   - [ ] Gère les erreurs (timeout, rate limit, API indisponible).

2. **Normalisation des données** :
   - [ ] `normalize()` convertit les données brutes en un objet `Observation` avec les champs :
     - `provider: str` (ex: `"atmo-france"`)
     - `observedAt: datetime` (horodatage de l'observation)
     - `retrievedAt: datetime` (horodatage de la récupération)
     - `expiresAt: datetime` (date d'expiration, ex: +5 min)
     - `quality: float` (score de qualité, ex: 0.95)
     - `value: dict` (données normalisées, ex: `{"aqi": 42, "pollutant": "PM2.5"}`)

3. **Validation des données** :
   - [ ] `validate()` rejette les valeurs hors plage : **0 ≤ AQI ≤ 500**.
   - [ ] Génère une alerte si la valeur est invalide (ex: `"Valeur AQI invalide : 1000 (max: 500). Utilisation de la dernière valeur valide en cache."`).

4. **Sécurité** :
   - [ ] Les **secrets** (API keys) ne sont **jamais exposés** côté frontend.
   - [ ] Utilisation de variables d'environnement (ex: `ATMO_FRANCE_API_KEY`).

5. **Intégration avec le Context Engine** :
   - [ ] L'adapter est **enregistré** dans le `ContextEngine` pour être utilisé.
   - [ ] Les données normalisées sont **transmises au cache Redis** (US-1.5).

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/context_engine/providers/atmo.py` | Adapter Atmo France | À créer |
| `backend/engines/context_engine/adapter.py` | Interface `ProviderAdapter` (ABC) | À créer |
| `backend/core/config.py` | Configuration (API key, endpoints) | À mettre à jour |

### Structure de l'adapter (`atmo.py`)
```python
from abc import ABC, abstractmethod
from datetime import datetime
from typing import Dict, Any, Optional
import httpx
from pydantic import BaseModel, Field, validator

# Modèle pour les données normalisées (Observation)
class Observation(BaseModel):
    provider: str = Field(..., description="Nom du provider (ex: 'atmo-france')")
    observedAt: datetime = Field(..., description="Horodatage de l'observation")
    retrievedAt: datetime = Field(..., description="Horodatage de la récupération")
    expiresAt: datetime = Field(..., description="Date d'expiration")
    quality: float = Field(..., ge=0.0, le=1.0, description="Score de qualité (0-1)")
    value: Dict[str, Any] = Field(..., description="Données normalisées")

# Interface ProviderAdapter (ABC)
class ProviderAdapter(ABC):
    @abstractmethod
    async def fetch(self) -> Dict[str, Any]:
        """Récupère les données brutes depuis le provider."""
        pass

    @abstractmethod
    def normalize(self, raw_data: Dict[str, Any]) -> Observation:
        """Normalise les données brutes en Observation."""
        pass

    @abstractmethod
    def validate(self, observation: Observation) -> bool:
        """Valide les données normalisées. Retourne True si valide."""
        pass

# Implémentation pour Atmo France
class AtmoFranceAdapter(ProviderAdapter):
    def __init__(self, api_key: str, endpoint: str = "https://api.atmo-france.org/v1/indice/"):
        self.api_key = api_key
        self.endpoint = endpoint
        self.client = httpx.AsyncClient(timeout=10.0)

    async def fetch(self) -> Dict[str, Any]:
        """
        Récupère les données de qualité de l'air depuis Atmo France.
        Endpoint : https://api.atmo-france.org/v1/indice/{code_commune}
        Exemple : https://api.atmo-france.org/v1/indice/69003 (Lyon)
        """
        headers = {"Authorization": f"Bearer {self.api_key}"}
        try:
            response = await self.client.get(self.endpoint, headers=headers)
            response.raise_for_status()
            return response.json()
        except httpx.HTTPStatusError as e:
            # Gérer les erreurs HTTP (ex: 429 = rate limit)
            raise ProviderError(f"Atmo France API error: {e.response.status_code}")
        except httpx.RequestError as e:
            # Gérer les erreurs de connexion
            raise ProviderError(f"Atmo France API connection error: {str(e)}")

    def normalize(self, raw_data: Dict[str, Any]) -> Observation:
        """
        Normalise les données brutes d'Atmo France en Observation.
        Exemple de données brutes :
        {
            "date": "2026-08-16",
            "code_commune": "69003",
            "nom_commune": "Lyon",
            "indice": 42,
            "polluant": "PM2.5",
            "couleur": "vert"
        }
        """
        try:
            observed_at = datetime.strptime(raw_data["date"], "%Y-%m-%d")
            retrieved_at = datetime.utcnow()
            expires_at = retrieved_at + timedelta(minutes=5)  # TTL = 5 min
            
            # Normalisation des données
            value = {
                "aqi": raw_data["indice"],
                "pollutant": raw_data["polluant"],
                "color": raw_data["couleur"],
                "location": {
                    "code": raw_data["code_commune"],
                    "name": raw_data["nom_commune"]
                }
            }
            
            return Observation(
                provider="atmo-france",
                observedAt=observed_at,
                retrievedAt=retrieved_at,
                expiresAt=expires_at,
                quality=0.95,  # Score de qualité élevé pour Atmo France
                value=value
            )
        except KeyError as e:
            raise NormalizationError(f"Missing key in Atmo France data: {e}")

    def validate(self, observation: Observation) -> bool:
        """
        Valide l'observation selon les seuils définis dans FR-1.
        Pour Atmo France : 0 ≤ AQI ≤ 500.
        """
        aqi = observation.value.get("aqi")
        if aqi is None:
            raise ValidationError("AQI value is missing")
        if not (0 <= aqi <= 500):
            raise ValidationError(f"Invalid AQI value: {aqi} (must be between 0 and 500)")
        return True

# Exceptions personnalisées
class ProviderError(Exception):
    """Erreur liée à la récupération des données."""
    pass

class NormalizationError(Exception):
    """Erreur liée à la normalisation des données."""
    pass

class ValidationError(Exception):
    """Erreur liée à la validation des données."""
    pass
```

---

## Tests Unitaires
Créer un fichier `tests/unit/test_atmo_adapter.py` avec les tests suivants :

```python
import pytest
from datetime import datetime, timedelta
from backend.engines.context_engine.providers.atmo import AtmoFranceAdapter, Observation

@pytest.fixture
def mock_atmo_data():
    return {
        "date": "2026-08-16",
        "code_commune": "69003",
        "nom_commune": "Lyon",
        "indice": 42,
        "polluant": "PM2.5",
        "couleur": "vert"
    }

@pytest.fixture
def atmo_adapter():
    return AtmoFranceAdapter(api_key="test_key")

# Test de normalisation
def test_normalize(atmo_adapter, mock_atmo_data):
    observation = atmo_adapter.normalize(mock_atmo_data)
    assert observation.provider == "atmo-france"
    assert observation.value["aqi"] == 42
    assert observation.value["polluant"] == "PM2.5"
    assert observation.quality == 0.95

# Test de validation
def test_validate_valid_aqi(atmo_adapter, mock_atmo_data):
    observation = atmo_adapter.normalize(mock_atmo_data)
    assert atmo_adapter.validate(observation) is True

# Test de validation invalide (AQI > 500)
def test_validate_invalid_aqi(atmo_adapter):
    invalid_data = {
        "date": "2026-08-16",
        "code_commune": "69003",
        "nom_commune": "Lyon",
        "indice": 1000,  # Valeur invalide
        "polluant": "PM2.5",
        "couleur": "rouge"
    }
    observation = atmo_adapter.normalize(invalid_data)
    with pytest.raises(ValidationError):
        atmo_adapter.validate(observation)

# Test de données manquantes
def test_normalize_missing_key(atmo_adapter):
    incomplete_data = {
        "date": "2026-08-16",
        "code_commune": "69003"
        # Manque "indice", "polluant", etc.
    }
    with pytest.raises(NormalizationError):
        atmo_adapter.normalize(incomplete_data)
```

---

## Configuration Requise
Ajouter les variables d'environnement suivantes dans `.env` :
```env
# Clé API Atmo France (à obtenir via https://www.atmo-france.org/)
ATMO_FRANCE_API_KEY=your_api_key_here

# Endpoint de l'API (par défaut)
ATMO_FRANCE_ENDPOINT=https://api.atmo-france.org/v1/indice/
```

---

## Intégration avec le Context Engine
Dans `backend/engines/context_engine/context_builder.py` :
```python
from backend.engines.context_engine.providers.atmo import AtmoFranceAdapter
from backend.core.config import settings

class ContextBuilder:
    def __init__(self):
        self.adapters = {
            "atmo-france": AtmoFranceAdapter(
                api_key=settings.ATMO_FRANCE_API_KEY,
                endpoint=settings.ATMO_FRANCE_ENDPOINT
            ),
            # Ajouter les autres adapters ici (RTE, Hub'Eau, OSM)
        }

    async def build_unified_context(self, location: str) -> UnifiedContext:
        observations = []
        for provider_name, adapter in self.adapters.items():
            try:
                raw_data = await adapter.fetch()
                observation = adapter.normalize(raw_data)
                if adapter.validate(observation):
                    observations.append(observation)
            except (ProviderError, NormalizationError, ValidationError) as e:
                # Log l'erreur et utilise le cache si disponible
                print(f"Error with {provider_name}: {e}")
                # TODO: Utiliser le cache Redis (US-1.5)
        return UnifiedContext(observations=observations, timestamp=datetime.utcnow())
```

---

## Dépendances
- **Python 3.11+**
- **Librairies** :
  - `httpx` (pour les requêtes HTTP asynchrones)
  - `pydantic` (pour la validation des données)
  - `python-dotenv` (pour la gestion des variables d'environnement)

Installer avec :
```bash
pip install httpx pydantic python-dotenv
```

---

## Notes
- **Rate Limit** : Atmo France permet **100 requêtes/minute**. À surveiller en production.
- **Fallback** : En cas d'échec, utiliser la **dernière valeur valide en cache** (Redis, US-1.5).
- **Documentation API** : https://api.atmo-france.org/documentation

---

## Ressources
- [Documentation API Atmo France](https://api.atmo-france.org/documentation)
- [PRD Almanéa - FR-1](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AD-2](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
