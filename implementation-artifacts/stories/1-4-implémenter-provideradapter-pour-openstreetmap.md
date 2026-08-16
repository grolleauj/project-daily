---
name: Implémenter ProviderAdapter pour OpenStreetMap
id: 1-4-implémenter-provideradapter-pour-openstreetmap
epic: epic-1
story_type: backend
priority: high
estimation: M
dependencies: [1-1, 1-2, 1-3]
status: ready-for-dev
created: 2026-08-16
updated: 2026-08-16
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/implementation-artifacts/epic-1-context.md
---

# Story 1.4: Implémenter `ProviderAdapter` pour OpenStreetMap

## Contexte
Cette story fait partie de **l'Épic 1 (Context Engine)**. Son objectif est de créer un **adapter dédié** pour récupérer, normaliser et valider les données géographiques depuis **OpenStreetMap (OSM)**, conformément à **AD-2 (Provider-Agnostic Adapters)** et **FR-1 (Collecte des données externes)**.

OpenStreetMap fournit des données géographiques comme les coordonnées, les limites administratives, ou les points d'intérêt (POI). Ces données seront utilisées pour enrichir le contexte avec des informations de localisation précises.

---

## Exigences Fonctionnelles
- **FR-1**: Collecte des données externes depuis les Providers via `ProviderAdapter`.
- **FR-2**: Normalisation des données brutes en format `Observation`.
- **AR-3**: Chaque provider a un adapter dédié implémentant `ProviderAdapter` (`fetch()`, `normalize()`, `validate()`).

---

## Critères d'Acceptation
1. **Récupération des données** :
   - [ ] `fetch()` appelle l'API OpenStreetMap (via **Nominatim** ou **Overpass API**) avec une requête HTTP valide.
   - [ ] Retourne les données brutes avec un statut HTTP 200.
   - [ ] Gère les erreurs (timeout, rate limit, API indisponible).

2. **Normalisation des données** :
   - [ ] `normalize()` convertit les données en `Observation` avec les champs :
     - `provider: str` (ex: `"openstreetmap"`)
     - `observedAt: datetime` (horodatage de l'observation, généralement `datetime.now()` pour les données statiques)
     - `retrievedAt: datetime` (horodatage de la récupération)
     - `expiresAt: datetime` (date d'expiration, ex: +24h pour les données géographiques statiques)
     - `quality: float` (score de qualité, ex: 0.98 pour OSM)
     - `value: dict` (données normalisées, ex: `{"latitude": 45.76, "longitude": 4.84, "display_name": "Lyon, France", "osm_id": 123456}`)

3. **Validation des données** :
   - [ ] `validate()` rejette les coordonnées invalides : **-90 ≤ latitude ≤ 90** et **-180 ≤ longitude ≤ 180**.
   - [ ] Génère une alerte si les coordonnées sont hors plage (ex: `"Coordonnées invalides : latitude=200 (max: 90). Utilisation de la dernière valeur valide en cache."`).

4. **Sécurité** :
   - [ ] Les **secrets** (clés API si utilisées) ne sont **jamais exposés** côté frontend.
   - [ ] Utilisation de variables d'environnement (ex: `OSM_API_KEY` si nécessaire).

5. **Intégration avec le Context Engine** :
   - [ ] L'adapter est **enregistré** dans le `ContextBuilder` pour être utilisé.
   - [ ] Les données normalisées sont **transmises au cache Redis** (US-1.5).

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/context_engine/providers/osm.py` | Adapter OpenStreetMap | À créer |
| `backend/engines/context_engine/context_builder.py` | Intégration de l'adapter | À modifier |
| `backend/core/config.py` | Configuration (endpoints) | À modifier |
| `tests/unit/test_osm_adapter.py` | Tests unitaires | À créer |

### Structure de l'adapter (`osm.py`)
```python
from datetime import datetime, timedelta, timezone
from typing import Dict, Any, Optional
from tenacity import retry, stop_after_attempt, wait_exponential
import httpx
from ..models import Observation
from ..adapter import ProviderAdapter, ProviderError, NormalizationError, ValidationError


class OSMAdapter(ProviderAdapter):
    def __init__(self, endpoint: str = "https://nominatim.openstreetmap.org/search"):
        self.endpoint = endpoint
        self.client = httpx.AsyncClient(timeout=httpx.Timeout(30.0, connect=5.0))

    async def close(self):
        """Ferme le client HTTP pour libérer les ressources."""
        await self.client.aclose()

    @retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=4, max=10))
    async def fetch(self, query: Optional[str] = None, lat: Optional[float] = None, lon: Optional[float] = None) -> Dict[str, Any]:
        """
        Récupère les données géographiques depuis OpenStreetMap (Nominatim).
        - Si `query` est fourni : recherche par nom (ex: "Lyon, France").
        - Si `lat` et `lon` sont fournis : recherche inverse (reverse geocoding).
        """
        try:
            params = {"format": "json", "limit": 1}
            if query:
                params["q"] = query
            elif lat is not None and lon is not None:
                params["lat"] = lat
                params["lon"] = lon
            else:
                raise ProviderError("OSM fetch requires either 'query' or 'lat/lon'")

            response = await self.client.get(self.endpoint, params=params)
            response.raise_for_status()
            return response.json()
        except httpx.HTTPStatusError as e:
            raise ProviderError(f"OSM API error: {e.response.status_code}")
        except httpx.RequestError as e:
            raise ProviderError(f"OSM API connection error: {str(e)}")

    def normalize(self, raw_data: Dict[str, Any]) -> Observation:
        """
        Normalise les données brutes d'OSM en Observation.
        Exemple de données brutes (Nominatim) :
        [
            {
                "place_id": 123456,
                "licence": "...",
                "osm_type": "node",
                "osm_id": 789,
                "lat": "45.764043",
                "lon": "4.835659",
                "display_name": "Lyon, Métropole de Lyon, Auvergne-Rhône-Alpes, France",
                "class": "city",
                "type": "city"
            }
        ]
        """
        try:
            if not raw_data or not isinstance(raw_data, list):
                raise NormalizationError("OSM data must be a non-empty list")

            first_result = raw_data[0]
            if not isinstance(first_result, dict):
                raise NormalizationError("OSM result must be a dictionary")

            # Validation des types
            if "lat" not in first_result or "lon" not in first_result:
                raise NormalizationError("OSM data must contain 'lat' and 'lon'")

            # Parsing des coordonnées (str -> float)
            latitude = float(first_result["lat"])
            longitude = float(first_result["lon"])

            observed_at = datetime.now(timezone.utc)  # Données géo = statiques
            retrieved_at = datetime.now(timezone.utc)
            expires_at = retrieved_at + timedelta(hours=24)  # TTL = 24h pour les données géo

            # Normalisation des données
            value = {
                "latitude": latitude,
                "longitude": longitude,
                "display_name": first_result.get("display_name", ""),
                "osm_id": first_result.get("osm_id", 0),
                "osm_type": first_result.get("osm_type", ""),
                "place_id": first_result.get("place_id", 0),
                "class": first_result.get("class", ""),
                "type": first_result.get("type", "")
            }

            return Observation(
                provider="openstreetmap",
                observedAt=observed_at,
                retrievedAt=retrieved_at,
                expiresAt=expires_at,
                quality=0.98,  # OSM est très fiable pour les données géo
                value=value
            )
        except (KeyError, ValueError, IndexError) as e:
            raise NormalizationError(f"Invalid OSM data format: {e}")

    def validate(self, observation: Observation) -> bool:
        """
        Valide l'observation selon les seuils définis.
        Pour OSM : -90 ≤ latitude ≤ 90 et -180 ≤ longitude ≤ 180.
        """
        latitude = observation.value.get("latitude")
        longitude = observation.value.get("longitude")

        if latitude is None or longitude is None:
            raise ValidationError("Latitude or longitude is missing")

        if not (-90 <= latitude <= 90):
            raise ValidationError(f"Invalid latitude: {latitude} (must be between -90 and 90)")

        if not (-180 <= longitude <= 180):
            raise ValidationError(f"Invalid longitude: {longitude} (must be between -180 and 180)")

        return True
```

---

## Tests Unitaires
Créer un fichier `tests/unit/test_osm_adapter.py` avec les tests suivants :

```python
import pytest
from datetime import datetime, timedelta
from typing import Optional
from unittest.mock import AsyncMock, patch, Mock
import sys
import os
import httpx

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '../../backend'))

from backend.engines.context_engine.providers.osm import OSMAdapter, ProviderError, NormalizationError, ValidationError
from backend.engines.context_engine.models import Observation


@pytest.fixture
def mock_osm_data():
    return [
        {
            "place_id": 123456,
            "licence": "Data © OpenStreetMap contributors",
            "osm_type": "node",
            "osm_id": 789,
            "lat": "45.764043",
            "lon": "4.835659",
            "display_name": "Lyon, Métropole de Lyon, Auvergne-Rhône-Alpes, France",
            "class": "city",
            "type": "city"
        }
    ]


@pytest.fixture
def osm_adapter():
    return OSMAdapter()


# Test de normalisation
def test_normalize(osm_adapter, mock_osm_data):
    observation = osm_adapter.normalize(mock_osm_data)
    assert observation.provider == "openstreetmap"
    assert observation.value["latitude"] == 45.764043
    assert observation.value["longitude"] == 4.835659
    assert observation.value["display_name"] == "Lyon, Métropole de Lyon, Auvergne-Rhône-Alpes, France"
    assert observation.quality == 0.98


# Test de validation
def test_validate_valid_coordinates(osm_adapter, mock_osm_data):
    observation = osm_adapter.normalize(mock_osm_data)
    assert osm_adapter.validate(observation) is True


# Test de validation invalide (latitude > 90)
def test_validate_invalid_latitude(osm_adapter):
    invalid_data = [
        {
            "place_id": 123456,
            "lat": "200",  # Latitude invalide
            "lon": "4.835659",
            "display_name": "Invalid Location"
        }
    ]
    observation = osm_adapter.normalize(invalid_data)
    with pytest.raises(ValidationError):
        osm_adapter.validate(observation)


# Test de validation invalide (longitude < -180)
def test_validate_invalid_longitude(osm_adapter):
    invalid_data = [
        {
            "place_id": 123456,
            "lat": "45.764043",
            "lon": "-200",  # Longitude invalide
            "display_name": "Invalid Location"
        }
    ]
    observation = osm_adapter.normalize(invalid_data)
    with pytest.raises(ValidationError):
        osm_adapter.validate(observation)


# Test de données manquantes
def test_normalize_missing_coordinates(osm_adapter):
    incomplete_data = [
        {
            "place_id": 123456,
            "display_name": "Lyon"
            # Manque "lat" et "lon"
        }
    ]
    with pytest.raises(NormalizationError):
        osm_adapter.normalize(incomplete_data)


# Test de fetch avec mock (recherche par nom)
@pytest.mark.asyncio
async def test_fetch_by_query(osm_adapter):
    mock_response = [
        {
            "place_id": 123456,
            "lat": "45.764043",
            "lon": "4.835659",
            "display_name": "Lyon, France"
        }
    ]
    mock_response_obj = Mock()
    mock_response_obj.status_code = 200
    mock_response_obj.json = Mock(return_value=mock_response)
    mock_response_obj.raise_for_status = Mock()

    with patch.object(osm_adapter.client, 'get', new_callable=AsyncMock) as mock_get:
        mock_get.return_value = mock_response_obj
        result = await osm_adapter.fetch(query="Lyon, France")
        assert result == mock_response


# Test de fetch avec mock (recherche inverse)
@pytest.mark.asyncio
async def test_fetch_by_coordinates(osm_adapter):
    mock_response = [
        {
            "place_id": 123456,
            "lat": "45.764043",
            "lon": "4.835659",
            "display_name": "Lyon, France"
        }
    ]
    mock_response_obj = Mock()
    mock_response_obj.status_code = 200
    mock_response_obj.json = Mock(return_value=mock_response)
    mock_response_obj.raise_for_status = Mock()

    with patch.object(osm_adapter.client, 'get', new_callable=AsyncMock) as mock_get:
        mock_get.return_value = mock_response_obj
        result = await osm_adapter.fetch(lat=45.764043, lon=4.835659)
        assert result == mock_response


# Test de fetch avec erreur HTTP
@pytest.mark.asyncio
async def test_fetch_http_error(osm_adapter):
    from backend.engines.context_engine.providers.osm import OSMAdapter
    original_fetch = OSMAdapter.fetch

    async def fetch_no_retry(self, query: Optional[str] = None, lat: Optional[float] = None, lon: Optional[float] = None):
        try:
            params = {"format": "json", "limit": 1}
            if query:
                params["q"] = query
            elif lat is not None and lon is not None:
                params["lat"] = lat
                params["lon"] = lon
            else:
                raise ProviderError("OSM fetch requires either 'query' or 'lat/lon'")

            response = await self.client.get(self.endpoint, params=params)
            response.raise_for_status()
            return response.json()
        except httpx.HTTPStatusError as e:
            raise ProviderError(f"OSM API error: {e.response.status_code}")
        except httpx.RequestError as e:
            raise ProviderError(f"OSM API connection error: {str(e)}")

    OSMAdapter.fetch = fetch_no_retry
    try:
        with patch.object(osm_adapter.client, 'get', new_callable=AsyncMock) as mock_get:
            mock_response = Mock()
            mock_response.status_code = 429
            mock_get.side_effect = httpx.HTTPStatusError("Rate limit exceeded", request=Mock(), response=mock_response)
            with pytest.raises(ProviderError):
                await osm_adapter.fetch(query="Lyon, France")
    finally:
        OSMAdapter.fetch = original_fetch


# Test de fetch sans paramètre
@pytest.mark.asyncio
async def test_fetch_no_parameters(osm_adapter):
    with pytest.raises(ProviderError):
        await osm_adapter.fetch()
```

---

## Configuration Requise
Ajouter les variables d'environnement suivantes dans `.env` :
```env
# Endpoint de l'API Nominatim (OpenStreetMap)
OSM_ENDPOINT=https://nominatim.openstreetmap.org/search
```

> **Note** : L'API Nominatim d'OpenStreetMap est **publique et ne nécessite pas de clé API**, mais elle a des **limites strictes** (1 requête/seconde, pas de requêtes massives). Pour un usage intensif, utiliser une instance locale de Nominatim ou un service comme **Photon** (https://photon.komoot.io/).

---

## Intégration avec le Context Engine
Dans `backend/engines/context_engine/context_builder.py` :
```python
from backend.engines.context_engine.providers.osm import OSMAdapter

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
```

---

## Dépendances
- **Python 3.11+**
- **Librairies** :
  - `httpx` (pour les requêtes HTTP asynchrones)
  - `tenacity` (pour les retries)

Installer avec :
```bash
pip install httpx tenacity
```

---

## Notes
- **Rate Limit** : Nominatim limite à **1 requête/seconde**. Utiliser `time.sleep(1)` entre les requêtes ou un service alternatif comme **Photon** (https://photon.komoot.io/).
- **Fallback** : En cas d'échec, utiliser la **dernière valeur valide en cache** (Redis, US-1.5).
- **Documentation API** :
  - [Nominatim (OpenStreetMap)](https://nominatim.org/)
  - [Photon (Alternative)](https://photon.komoot.io/)
- **Données statiques** : Les coordonnées géographiques changent rarement, donc un TTL de **24h** est approprié.

---

## Ressources
- [Documentation Nominatim](https://nominatim.org/release-docs/latest/api/Reverse.html)
- [PRD Almanéa - FR-1](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AD-2](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
