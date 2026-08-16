---
name: Implémenter ProviderAdapter pour Hub'Eau
id: 1-3-implémenter-provideradapter-pour-hubeau
epic: epic-1
story_type: backend
priority: high
estimation: M
dependencies: [1-1, 1-2]
status: ready-for-dev
created: 2026-08-16
updated: 2026-08-16
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/implementation-artifacts/epic-1-context.md
---

# Story 1.3: Implémenter `ProviderAdapter` pour Hub'Eau

## Contexte
Cette story fait partie de **l'Épic 1 (Context Engine)**. Son objectif est de créer un **adapter dédié** pour récupérer, normaliser et valider les données des **niveaux d'eau** depuis **Hub'Eau**, conformément à **AD-2 (Provider-Agnostic Adapters)** et **FR-1 (Collecte des données externes)**.

---

## Exigences Fonctionnelles
- **FR-1**: Collecte des données externes depuis les Providers via `ProviderAdapter`.
- **FR-2**: Normalisation des données brutes en format `Observation`.
- **AR-3**: Chaque provider a un adapter dédié implémentant `ProviderAdapter` (`fetch()`, `normalize()`, `validate()`).

---

## Critères d'Acceptation
1. **Récupération des données** :
   - [ ] `fetch()` appelle l'API Hub'Eau avec une requête HTTP valide.
   - [ ] Retourne les données brutes avec un statut HTTP 200.
   - [ ] Gère les erreurs (timeout, rate limit, API indisponible).

2. **Normalisation des données** :
   - [ ] `normalize()` convertit les données en `Observation` avec les champs :
     - `provider: str` (ex: `"hubeau"`)
     - `observedAt: datetime` (horodatage de l'observation)
     - `retrievedAt: datetime` (horodatage de la récupération)
     - `expiresAt: datetime` (date d'expiration, ex: +1h pour les données statiques)
     - `quality: float` (score de qualité, ex: 0.95)
     - `value: dict` (données normalisées, ex: `{"niveau": 120.5, "code_station": "H12345", "libelle_station": "Lyon"}`)

3. **Validation des données** :
   - [ ] `validate()` rejette les valeurs hors plage : **0 ≤ niveau ≤ 1000 cm** (d'après FR-1).
   - [ ] Génère une alerte si la valeur est invalide (ex: `"Valeur de niveau invalide : 2000 cm (max: 1000 cm). Utilisation de la dernière valeur valide en cache."`).

4. **Sécurité** :
   - [ ] Les **secrets** (API keys) ne sont **jamais exposés** côté frontend.
   - [ ] Utilisation de variables d'environnement (ex: `HUBEAU_API_KEY`).

5. **Intégration avec le Context Engine** :
   - [ ] L'adapter est **enregistré** dans le `ContextBuilder` pour être utilisé.
   - [ ] Les données normalisées sont **transmises au cache Redis** (US-1.5).

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/context_engine/providers/hubeau.py` | Adapter Hub'Eau | À créer |
| `backend/engines/context_engine/context_builder.py` | Intégration de l'adapter | À modifier |
| `backend/core/config.py` | Configuration (API key, endpoints) | À modifier |
| `tests/unit/test_hubeau_adapter.py` | Tests unitaires | À créer |
| `.env.example` | Variables d'environnement | À modifier |

### Structure de l'adapter (`hubeau.py`)
```python
from datetime import datetime, timedelta, timezone
from typing import Dict, Any, Optional
from tenacity import retry, stop_after_attempt, wait_exponential
import httpx
from ..models import Observation
from ..adapter import ProviderAdapter, ProviderError, NormalizationError, ValidationError

class HubEauAdapter(ProviderAdapter):
    def __init__(self, api_key: str, endpoint: str = "https://hubeau.eaufrance.fr/api/v1/niveaux_nappes"):
        self.api_key = api_key
        self.endpoint = endpoint
        self.client = httpx.AsyncClient(timeout=httpx.Timeout(30.0, connect=5.0))

    async def close(self):
        """Ferme le client HTTP pour libérer les ressources."""
        await self.client.aclose()

    @retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=4, max=10))
    async def fetch(self, code_commune: Optional[str] = None) -> Dict[str, Any]:
        """
        Récupère les données des niveaux d'eau depuis Hub'Eau.
        Endpoint : https://hubeau.eaufrance.fr/api/v1/niveaux_nappes
        Note : Hub'Eau utilise des codes de stations (ex: "H12345") plutôt que des codes communes.
        """
        headers = {"Authorization": f"Bearer {self.api_key}"}
        try:
            # Si code_commune est fourni, on peut essayer de le mapper à une station
            # Sinon, on récupère les données pour toutes les stations (ou une station par défaut)
            params = {}
            if code_commune:
                # TODO: Mapper code_commune à code_station (nécessite une table de correspondance)
                params["code_station"] = code_commune  # Simplification pour l'exemple
            
            response = await self.client.get(self.endpoint, headers=headers, params=params)
            response.raise_for_status()
            return response.json()
        except httpx.HTTPStatusError as e:
            raise ProviderError(f"Hub'Eau API error: {e.response.status_code}")
        except httpx.RequestError as e:
            raise ProviderError(f"Hub'Eau API connection error: {str(e)}")

    def normalize(self, raw_data: Dict[str, Any]) -> Observation:
        """
        Normalise les données brutes de Hub'Eau en Observation.
        Exemple de données brutes (simplifié) :
        {
            "date_obs": "2026-08-16T14:00:00+02:00",
            "code_station": "H12345",
            "libelle_station": "Lyon - Nappe phréatique",
            "niveau_eau": 120.5,
            "unite": "cm"
        }
        """
        try:
            # Validation des types
            if not isinstance(raw_data.get("niveau_eau"), (int, float)):
                raise NormalizationError("niveau_eau must be a number")
            if not isinstance(raw_data.get("date_obs"), str):
                raise NormalizationError("date_obs must be a string")
            
            # Parsing de la date (format ISO 8601)
            observed_at = datetime.fromisoformat(raw_data["date_obs"].replace('Z', '+00:00'))
            retrieved_at = datetime.now(timezone.utc)
            expires_at = retrieved_at + timedelta(hours=1)  # TTL = 1h pour les données statiques
            
            # Normalisation des données
            value = {
                "niveau": raw_data["niveau_eau"],
                "unite": raw_data.get("unite", "cm"),
                "code_station": raw_data["code_station"],
                "libelle_station": raw_data.get("libelle_station", "")
            }
            
            return Observation(
                provider="hubeau",
                observedAt=observed_at,
                retrievedAt=retrieved_at,
                expiresAt=expires_at,
                quality=0.95,
                value=value
            )
        except KeyError as e:
            raise NormalizationError(f"Missing key in Hub'Eau data: {e}")

    def validate(self, observation: Observation) -> bool:
        """
        Valide l'observation selon les seuils définis.
        Pour Hub'Eau : 0 ≤ niveau ≤ 1000 cm.
        """
        niveau = observation.value.get("niveau")
        if niveau is None:
            raise ValidationError("Niveau value is missing")
        if not (0 <= niveau <= 1000):
            raise ValidationError(f"Invalid niveau value: {niveau} (must be between 0 and 1000 cm)")
        return True
```

---

## Tests Unitaires
Créer un fichier `tests/unit/test_hubeau_adapter.py` avec les tests suivants :

```python
import pytest
from datetime import datetime, timedelta
from unittest.mock import AsyncMock, patch, Mock
import sys
import os
import httpx

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '../../backend'))

from backend.engines.context_engine.providers.hubeau import HubEauAdapter, ProviderError, NormalizationError, ValidationError
from backend.engines.context_engine.models import Observation

@pytest.fixture
def mock_hubeau_data():
    return {
        "date_obs": "2026-08-16T14:00:00+02:00",
        "code_station": "H12345",
        "libelle_station": "Lyon - Nappe phréatique",
        "niveau_eau": 120.5,
        "unite": "cm"
    }

@pytest.fixture
def hubeau_adapter():
    return HubEauAdapter(api_key="test_key")

# Test de normalisation
def test_normalize(hubeau_adapter, mock_hubeau_data):
    observation = hubeau_adapter.normalize(mock_hubeau_data)
    assert observation.provider == "hubeau"
    assert observation.value["niveau"] == 120.5
    assert observation.value["code_station"] == "H12345"
    assert observation.quality == 0.95

# Test de validation
def test_validate_valid_niveau(hubeau_adapter, mock_hubeau_data):
    observation = hubeau_adapter.normalize(mock_hubeau_data)
    assert hubeau_adapter.validate(observation) is True

# Test de validation invalide (niveau > 1000 cm)
def test_validate_invalid_niveau(hubeau_adapter):
    invalid_data = {
        "date_obs": "2026-08-16T14:00:00+02:00",
        "code_station": "H12345",
        "libelle_station": "Lyon - Nappe phréatique",
        "niveau_eau": 2000,  # Valeur invalide
        "unite": "cm"
    }
    observation = hubeau_adapter.normalize(invalid_data)
    with pytest.raises(ValidationError):
        hubeau_adapter.validate(observation)

# Test de données manquantes
def test_normalize_missing_key(hubeau_adapter):
    incomplete_data = {
        "date_obs": "2026-08-16T14:00:00+02:00",
        "code_station": "H12345"
        # Manque "niveau_eau"
    }
    with pytest.raises(NormalizationError):
        hubeau_adapter.normalize(incomplete_data)

# Test de fetch avec mock
@pytest.mark.asyncio
async def test_fetch_success(hubeau_adapter):
    mock_response = {
        "date_obs": "2026-08-16T14:00:00+02:00",
        "code_station": "H12345",
        "libelle_station": "Lyon - Nappe phréatique",
        "niveau_eau": 120.5,
        "unite": "cm"
    }
    mock_response_obj = AsyncMock()
    mock_response_obj.status_code = 200
    mock_response_obj.json = AsyncMock(return_value=mock_response)
    mock_response_obj.raise_for_status = AsyncMock()

    with patch.object(hubeau_adapter.client, 'get', new_callable=AsyncMock) as mock_get:
        mock_get.return_value = mock_response_obj
        result = await hubeau_adapter.fetch(code_commune="H12345")
        assert result == mock_response

# Test de fetch avec erreur HTTP
@pytest.mark.asyncio
async def test_fetch_http_error(hubeau_adapter):
    # Désactiver le retry pour ce test
    from backend.engines.context_engine.providers.hubeau import HubEauAdapter
    original_fetch = HubEauAdapter.fetch
    
    async def fetch_no_retry(self, code_commune: Optional[str] = None):
        headers = {"Authorization": f"Bearer {self.api_key}"}
        try:
            params = {}
            if code_commune:
                params["code_station"] = code_commune
            response = await self.client.get(self.endpoint, headers=headers, params=params)
            response.raise_for_status()
            return response.json()
        except httpx.HTTPStatusError as e:
            raise ProviderError(f"Hub'Eau API error: {e.response.status_code}")
        except httpx.RequestError as e:
            raise ProviderError(f"Hub'Eau API connection error: {str(e)}")
    
    HubEauAdapter.fetch = fetch_no_retry
    try:
        with patch.object(hubeau_adapter.client, 'get', new_callable=AsyncMock) as mock_get:
            mock_get.side_effect = httpx.HTTPStatusError("Server error", request=Mock(), response=Mock(status_code=500))
            with pytest.raises(ProviderError):
                await hubeau_adapter.fetch(code_commune="H12345")
    finally:
        HubEauAdapter.fetch = original_fetch
```

---

## Configuration Requise
Ajouter les variables d'environnement suivantes dans `.env` :
```env
# Clé API Hub'Eau (à obtenir via https://hubeau.eaufrance.fr/)
HUBEAU_API_KEY=your_api_key_here

# Endpoint de l'API (par défaut)
HUBEAU_ENDPOINT=https://hubeau.eaufrance.fr/api/v1/niveaux_nappes
```

---

## Intégration avec le Context Engine
Dans `backend/engines/context_engine/context_builder.py` :
```python
from backend.engines.context_engine.providers.hubeau import HubEauAdapter

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
- **Rate Limit** : Hub'Eau a des limites de rate (1000 requêtes/jour pour la clé publique).
- **Fallback** : En cas d'échec, utiliser la **dernière valeur valide en cache** (Redis, US-1.5).
- **Documentation API** : https://hubeau.eaufrance.fr/
- **Mapping code_commune → code_station** : Nécessite une table de correspondance (à implémenter ultérieurement).

---

## Ressources
- [Documentation API Hub'Eau](https://hubeau.eaufrance.fr/)
- [PRD Almanéa - FR-1](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AD-2](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
