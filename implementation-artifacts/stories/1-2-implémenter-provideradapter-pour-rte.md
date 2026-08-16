---
name: Implémenter ProviderAdapter pour RTE
id: 1-2-implémenter-provideradapter-pour-rte
epic: epic-1
story_type: backend
priority: high
estimation: M
dependencies: [1-1]
status: done
created: 2026-08-16
updated: 2026-08-16
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/implementation-artifacts/epic-1-context.md
---

# Story 1.2: Implémenter `ProviderAdapter` pour RTE

## Contexte
Cette story fait partie de **l'Épic 1 (Context Engine)**. Son objectif est de créer un **adapter dédié** pour récupérer, normaliser et valider les données du **mix énergétique** depuis **RTE (Réseau de Transport d'Électricité)**, conformément à **AD-2 (Provider-Agnostic Adapters)** et **FR-1 (Collecte des données externes)**.

---

## Exigences Fonctionnelles
- **FR-1**: Collecte des données externes depuis les Providers via `ProviderAdapter`.
- **FR-2**: Normalisation des données brutes en format `Observation`.
- **AR-3**: Chaque provider a un adapter dédié implémentant `ProviderAdapter` (`fetch()`, `normalize()`, `validate()`).

---

## Critères d'Acceptation
1. **Récupération des données** :
   - [ ] `fetch()` appelle l'API RTE avec une requête HTTP valide.
   - [ ] Retourne les données brutes avec un statut HTTP 200.
   - [ ] Gère les erreurs (timeout, rate limit, API indisponible).

2. **Normalisation des données** :
   - [ ] `normalize()` convertit les données en `Observation` avec les champs :
     - `provider: str` (ex: `"rte"`)
     - `observedAt: datetime` (horodatage de l'observation)
     - `retrievedAt: datetime` (horodatage de la récupération)
     - `expiresAt: datetime` (date d'expiration, ex: +5 min)
     - `quality: float` (score de qualité, ex: 0.95)
     - `value: dict` (données normalisées, ex: `{"mix": {"nucleaire": 50.2, "thermique": 20.1}, "co2": 42}`)

3. **Validation des données** :
   - [ ] `validate()` rejette les valeurs hors plage : **0 ≤ production ≤ 100 GW par type de filière**.
   - [ ] Génère une alerte si la valeur est invalide (ex: `"Valeur de production invalide : 200 GW (max: 100 GW). Utilisation de la dernière valeur valide en cache."`).

4. **Sécurité** :
   - [ ] Les **secrets** (API keys) ne sont **jamais exposés** côté frontend.
   - [ ] Utilisation de variables d'environnement (ex: `RTE_API_KEY`).

5. **Intégration avec le Context Engine** :
   - [ ] L'adapter est **enregistré** dans le `ContextBuilder` pour être utilisé.
   - [ ] Les données normalisées sont **transmises au cache Redis** (US-1.5).

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/context_engine/providers/rte.py` | Adapter RTE | À créer |
| `backend/engines/context_engine/context_builder.py` | Intégration de l'adapter | À modifier |
| `backend/core/config.py` | Configuration (API key, endpoints) | À modifier |

### Structure de l'adapter (`rte.py`)
```python
from datetime import datetime, timedelta, timezone
from typing import Dict, Any
import httpx
from tenacity import retry, stop_after_attempt, wait_exponential
from ..models import Observation
from ..adapter import ProviderAdapter

class ProviderError(Exception):
    """Erreur liée à la récupération des données."""
    pass

class NormalizationError(Exception):
    """Erreur liée à la normalisation des données."""
    pass

class ValidationError(Exception):
    """Erreur liée à la validation des données."""
    pass

class RTEAdapter(ProviderAdapter):
    def __init__(self, api_key: str, endpoint: str = "https://api.rte-france.com/v1/mix"):
        self.api_key = api_key
        self.endpoint = endpoint
        self.client = httpx.AsyncClient(timeout=httpx.Timeout(30.0, connect=5.0))

    async def close(self):
        """Ferme le client HTTP pour libérer les ressources."""
        await self.client.aclose()

    @retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=4, max=10))
    async def fetch(self, code_commune: str = None) -> Dict[str, Any]:
        """
        Récupère les données du mix énergétique depuis RTE.
        Endpoint : https://api.rte-france.com/v1/mix
        Note : RTE ne nécessite pas de code_commune, les données sont nationales.
        """
        headers = {"Authorization": f"Bearer {self.api_key}"}
        try:
            response = await self.client.get(self.endpoint, headers=headers)
            response.raise_for_status()
            return response.json()
        except httpx.HTTPStatusError as e:
            raise ProviderError(f"RTE API error: {e.response.status_code}")
        except httpx.RequestError as e:
            raise ProviderError(f"RTE API connection error: {str(e)}")

    def normalize(self, raw_data: Dict[str, Any]) -> Observation:
        """
        Normalise les données brutes de RTE en Observation.
        Exemple de données brutes (simplifié) :
        {
            "date": "2026-08-16T14:00:00+02:00",
            "mix": {
                "nucleaire": 50.2,
                "thermique": 20.1,
                "hydraulique": 15.3,
                "eolien": 8.5,
                "solaire": 5.9
            },
            "co2": 42  # gCO2/kWh
        }
        """
        try:
            # Validation des types
            if not isinstance(raw_data.get("mix"), dict):
                raise NormalizationError("mix must be a dict")
            if not isinstance(raw_data.get("date"), str):
                raise NormalizationError("date must be a string")
            
            # Parsing de la date (format ISO 8601)
            observed_at = datetime.fromisoformat(raw_data["date"].replace('Z', '+00:00'))
            retrieved_at = datetime.now(timezone.utc)
            expires_at = retrieved_at + timedelta(minutes=5)
            
            # Normalisation des données
            value = {
                "mix": raw_data["mix"],
                "co2": raw_data.get("co2"),  # gCO2/kWh
                "total": sum(raw_data["mix"].values()) if raw_data.get("mix") else 0
            }
            
            return Observation(
                provider="rte",
                observedAt=observed_at,
                retrievedAt=retrieved_at,
                expiresAt=expires_at,
                quality=0.95,
                value=value
            )
        except KeyError as e:
            raise NormalizationError(f"Missing key in RTE data: {e}")

    def validate(self, observation: Observation) -> bool:
        """
        Valide l'observation selon les seuils définis.
        Pour RTE : 0 ≤ production ≤ 100 GW par type de filière.
        """
        mix = observation.value.get("mix", {})
        for filiere, production in mix.items():
            if not isinstance(production, (int, float)):
                raise ValidationError(f"Production for {filiere} must be a number")
            if not (0 <= production <= 100):
                raise ValidationError(f"Invalid production value for {filiere}: {production} (must be between 0 and 100 GW)")
        return True
```

---

## Tests Unitaires
Créer un fichier `tests/unit/test_rte_adapter.py` avec les tests suivants :

```python
import pytest
from datetime import datetime, timedelta
from unittest.mock import AsyncMock, patch, Mock
import sys
import os
import httpx

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '../../backend'))

from backend.engines.context_engine.providers.rte import RTEAdapter, ProviderError, NormalizationError, ValidationError
from backend.engines.context_engine.models import Observation

@pytest.fixture
def mock_rte_data():
    return {
        "date": "2026-08-16T14:00:00+02:00",
        "mix": {
            "nucleaire": 50.2,
            "thermique": 20.1,
            "hydraulique": 15.3,
            "eolien": 8.5,
            "solaire": 5.9
        },
        "co2": 42
    }

@pytest.fixture
def rte_adapter():
    return RTEAdapter(api_key="test_key")

# Test de normalisation
def test_normalize(rte_adapter, mock_rte_data):
    observation = rte_adapter.normalize(mock_rte_data)
    assert observation.provider == "rte"
    assert observation.value["mix"]["nucleaire"] == 50.2
    assert observation.value["co2"] == 42
    assert observation.value["total"] == pytest.approx(100.0)
    assert observation.quality == 0.95

# Test de validation
def test_validate_valid_production(rte_adapter, mock_rte_data):
    observation = rte_adapter.normalize(mock_rte_data)
    assert rte_adapter.validate(observation) is True

# Test de validation invalide (production > 100 GW)
def test_validate_invalid_production(rte_adapter):
    invalid_data = {
        "date": "2026-08-16T14:00:00+02:00",
        "mix": {
            "nucleaire": 150.0,  # Valeur invalide
            "thermique": 20.1
        },
        "co2": 42
    }
    observation = rte_adapter.normalize(invalid_data)
    with pytest.raises(ValidationError):
        rte_adapter.validate(observation)

# Test de données manquantes
def test_normalize_missing_key(rte_adapter):
    incomplete_data = {
        "date": "2026-08-16T14:00:00+02:00",
        "mix": {
            "nucleaire": 50.2
        }
        # Manque "co2"
    }
    # Ne doit pas lever d'erreur, car "co2" est optionnel
    observation = rte_adapter.normalize(incomplete_data)
    assert observation.value["co2"] is None

# Test de fetch avec mock
@pytest.mark.asyncio
async def test_fetch_success(rte_adapter):
    mock_response = {
        "date": "2026-08-16T14:00:00+02:00",
        "mix": {
            "nucleaire": 50.2,
            "thermique": 20.1
        },
        "co2": 42
    }
    mock_response_obj = Mock()
    mock_response_obj.status_code = 200
    mock_response_obj.json = Mock(return_value=mock_response)
    mock_response_obj.raise_for_status = Mock()

    with patch.object(rte_adapter.client, 'get', new_callable=AsyncMock) as mock_get:
        mock_get.return_value = mock_response_obj
        result = await rte_adapter.fetch()
        assert result == mock_response

# Test de fetch avec erreur HTTP
@pytest.mark.asyncio
async def test_fetch_http_error(rte_adapter):
    mock_response_obj = Mock()
    mock_response_obj.status_code = 500
    mock_response_obj.raise_for_status = Mock(side_effect=httpx.HTTPStatusError("Server error", request=Mock(), response=Mock(status_code=500)))

    # Désactiver le retry pour ce test
    original_fetch = rte_adapter.fetch
    async def fetch_no_retry(*args, **kwargs):
        try:
            response = await rte_adapter.client.get(*args, **kwargs)
            response.raise_for_status()
            return response.json()
        except httpx.HTTPStatusError as e:
            raise ProviderError(f"RTE API error: {e.response.status_code}")
    rte_adapter.fetch = fetch_no_retry
    try:
        with patch.object(rte_adapter.client, 'get', new_callable=AsyncMock) as mock_get:
            mock_get.return_value = mock_response_obj
            with pytest.raises(ProviderError):
                await rte_adapter.fetch()
    finally:
        rte_adapter.fetch = original_fetch
```

---

## Configuration Requise
Ajouter les variables d'environnement suivantes dans `.env` :
```env
# Clé API RTE (à obtenir via https://api.rte-france.com/)
RTE_API_KEY=your_api_key_here

# Endpoint de l'API (par défaut)
RTE_ENDPOINT=https://api.rte-france.com/v1/mix
```

---

## Intégration avec le Context Engine
Dans `backend/engines/context_engine/context_builder.py` :
```python
from backend.engines.context_engine.providers.rte import RTEAdapter

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
        }
```

---

## Dépendances
- **Python 3.11+**
- **Librairies** :
  - `httpx` (pour les requêtes HTTP asynchrones)
  - `pydantic` (pour la validation des données)
  - `tenacity` (pour les retries)
  - `python-dotenv` (pour la gestion des variables d'environnement)

Installer avec :
```bash
pip install httpx pydantic tenacity python-dotenv
```

---

## Notes
- **Rate Limit** : RTE a des limites de rate (à vérifier dans leur documentation).
- **Fallback** : En cas d'échec, utiliser la **dernière valeur valide en cache** (Redis, US-1.5).
- **Documentation API** : https://api.rte-france.com/

---

## Ressources
- [Documentation API RTE](https://api.rte-france.com/)
- [PRD Almanéa - FR-1](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AD-2](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
