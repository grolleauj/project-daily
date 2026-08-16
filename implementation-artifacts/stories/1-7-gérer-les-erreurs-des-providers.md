---
name: Gérer les erreurs des Providers
id: 1-7-gérer-les-erreurs-des-providers
epic: epic-1
story_type: backend
priority: medium
estimation: S
dependencies: [1-1, 1-2, 1-3, 1-4, 1-5, 1-6]
status: ready-for-dev
created: 2026-08-16
updated: 2026-08-16
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/implementation-artifacts/epic-1-context.md
---

# Story 1.7: Gérer les erreurs des Providers

## Contexte
Cette story fait partie de **l'Épic 1 (Context Engine)**. Son objectif est d'implémenter un **système de gestion des erreurs** pour les providers, conformément à **FR-5 (Gestion des erreurs des Providers)**. Ce système doit :
- **Détecter les pannes critiques** (ex: >50% des providers indisponibles).
- **Générer des alertes** (logs, notifications).
- **Fournir un fallback** vers le cache Redis ou des valeurs par défaut.

---

## Exigences Fonctionnelles
- **FR-5**: Gestion des erreurs des Providers (fallback vers le cache, alertes pour les pannes critiques).
- **AD-7**: Cache TTL: 5 minutes pour les données temps réel, 1 heure pour les données statiques.

---

## Critères d'Acceptation
1. **Détection des erreurs** :
   - [ ] Chaque erreur de provider (`ProviderError`, `NormalizationError`, `ValidationError`) est **loguée** avec un niveau approprié (WARNING pour les erreurs temporaires, ERROR pour les pannes critiques).
   - [ ] Les erreurs sont **catégorisées** par type (ex: timeout, rate limit, API indisponible).

2. **Fallback vers le cache** :
   - [ ] Si un provider échoue, le `ContextBuilder` utilise **la dernière valeur valide en cache** (Redis).
   - [ ] Si aucune valeur n'est en cache, une **valeur par défaut** est utilisée (ex: `Observation` avec `quality=0.0` et `value={}`).

3. **Alertes pour les pannes critiques** :
   - [ ] Si **>50% des providers critiques** (Atmo France, RTE, Hub'Eau) sont indisponibles, une **alerte critique** est générée.
   - [ ] Les alertes sont **loguées** et peuvent être **envoyées à un système de monitoring** (ex: Sentry, Datadog).

4. **Métriques d'erreurs** :
   - [ ] Un **compteur d'erreurs** est maintenu pour chaque provider (ex: `errors:atmo-france:count`).
   - [ ] Les métriques sont **exposées** via une API ou un endpoint de monitoring.

5. **Sécurité** :
   - [ ] Les **logs** ne contiennent **aucune information sensible** (ex: API keys, données utilisateur).
   - [ ] Les alertes sont **anonymisées** si nécessaire.

6. **Intégration avec le Context Engine** :
   - [ ] Le système de gestion des erreurs est **intégré dans `ContextBuilder`**.
   - [ ] Les alertes sont **accessibles via `UnifiedContext`** (ex: `context.alerts`).

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/context_engine/errors.py` | Module de gestion des erreurs | À créer |
| `backend/engines/context_engine/context_builder.py` | Intégration des alertes | À modifier |
| `backend/engines/context_engine/models.py` | Modèle `Alert` et mise à jour de `UnifiedContext` | À modifier |
| `tests/unit/test_error_handling.py` | Tests unitaires | À créer |

### Structure du module de gestion des erreurs (`errors.py`)
```python
import logging
from datetime import datetime
from typing import Dict, List, Optional, Any
from enum import Enum

from .models import Alert


class ErrorType(Enum):
    """Types d'erreurs pour les providers."""
    TIMEOUT = "timeout"
    RATE_LIMIT = "rate_limit"
    API_UNAVAILABLE = "api_unavailable"
    INVALID_DATA = "invalid_data"
    CONNECTION_ERROR = "connection_error"
    UNKNOWN = "unknown"


class ErrorSeverity(Enum):
    """Niveaux de gravité des erreurs."""
    WARNING = "warning"
    ERROR = "error"
    CRITICAL = "critical"


class ProviderErrorHandler:
    """Gère les erreurs des providers et génère des alertes."""

    def __init__(self):
        self.logger = logging.getLogger("context_engine.errors")
        self.errors: Dict[str, List[Dict[str, Any]]] = {}
        self.alerts: List[Alert] = []
        self.critical_providers = {"atmo-france", "rte", "hubeau"}  # Providers critiques

    def log_error(self, provider: str, error_type: ErrorType, message: str, severity: ErrorSeverity = ErrorSeverity.WARNING):
        """
        Log une erreur et stocke les détails.
        """
        error_entry = {
            "timestamp": datetime.now(),
            "provider": provider,
            "type": error_type.value,
            "message": message,
            "severity": severity.value
        }

        if provider not in self.errors:
            self.errors[provider] = []
        self.errors[provider].append(error_entry)

        # Log selon la gravité
        if severity == ErrorSeverity.CRITICAL:
            self.logger.critical(f"[{provider}] {error_type.value}: {message}")
        elif severity == ErrorSeverity.ERROR:
            self.logger.error(f"[{provider}] {error_type.value}: {message}")
        else:
            self.logger.warning(f"[{provider}] {error_type.value}: {message}")

    def check_critical_failure(self) -> Optional[Alert]:
        """
        Vérifie si >50% des providers critiques sont en échec.
        Retourne une alerte si c'est le cas.
        """
        total_critical = len(self.critical_providers)
        failed_critical = 0

        for provider in self.critical_providers:
            if provider in self.errors:
                # Si le provider a des erreurs récentes (ex: dans les 5 dernières minutes)
                recent_errors = [
                    e for e in self.errors[provider]
                    if (datetime.now() - e["timestamp"]).total_seconds() < 300
                ]
                if recent_errors:
                    failed_critical += 1

        if failed_critical > total_critical / 2:
            alert = Alert(
                type="critical_failure",
                message=f"Plus de 50% des providers critiques sont indisponibles ({failed_critical}/{total_critical})",
                severity=ErrorSeverity.CRITICAL.value,
                timestamp=datetime.now(),
                details={
                    "failed_providers": list(self.errors.keys()),
                    "total_critical": total_critical,
                    "failed_critical": failed_critical
                }
            )
            self.alerts.append(alert)
            return alert
        return None

    def get_error_count(self, provider: str) -> int:
        """Retourne le nombre d'erreurs pour un provider."""
        return len(self.errors.get(provider, []))

    def get_all_errors(self) -> Dict[str, List[Dict[str, Any]]]:
        """Retourne toutes les erreurs enregistrées."""
        return self.errors

    def clear_errors(self, provider: Optional[str] = None):
        """Efface les erreurs pour un provider ou tous les providers."""
        if provider:
            self.errors.pop(provider, None)
        else:
            self.errors.clear()
```

### Mise à jour du modèle `Alert` (`models.py`)
```python
from datetime import datetime
from typing import Dict, Any, List, Optional
from pydantic import BaseModel, Field


class Alert(BaseModel):
    """Représente une alerte générée par le Context Engine."""
    type: str = Field(..., description="Type de l'alerte (ex: 'critical_failure')")
    message: str = Field(..., description="Message de l'alerte")
    severity: str = Field(..., description="Gravité de l'alerte (ex: 'warning', 'error', 'critical')")
    timestamp: datetime = Field(..., description="Horodatage de l'alerte")
    details: Dict[str, Any] = Field(default_factory=dict, description="Détails supplémentaires")
```

### Mise à jour de `UnifiedContext` (`models.py`)
```python
class UnifiedContext(BaseModel):
    """
    Contexte unifié pour le Recommendation Engine.
    Contient une liste d'observations normalisées, des données dérivées et des alertes.
    """
    observations: List[Observation] = Field(..., description="Liste des observations des providers")
    timestamp: datetime = Field(..., description="Horodatage de la création du contexte")
    user_profile: Optional[UserProfile] = Field(None, description="Profil utilisateur (optionnel)")
    derived_data: Dict[str, Any] = Field(
        default_factory=dict,
        description="Données dérivées (ex: global_aqi, energy_mix_summary)"
    )
    alerts: List[Alert] = Field(
        default_factory=list,
        description="Liste des alertes générées pendant la construction du contexte"
    )
```

### Mise à jour de `ContextBuilder` (`context_builder.py`)
```python
from datetime import datetime, timezone
from typing import Optional, List

from backend.core.config import settings
from .cache.redis_cache import RedisCache
from .errors import ProviderErrorHandler, ErrorType, ErrorSeverity
from .providers.atmo import AtmoFranceAdapter
from .providers.rte import RTEAdapter
from .providers.hubeau import HubEauAdapter
from .providers.osm import OSMAdapter
from .adapter import ProviderError, NormalizationError, ValidationError
from .models import UnifiedContext, Observation, UserProfile, Alert


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
        self.error_handler = ProviderErrorHandler()

    async def build_unified_context(
        self, 
        location: str = "69003", 
        user_profile: Optional[UserProfile] = None
    ) -> UnifiedContext:
        """
        Construit un UnifiedContext à partir des données des providers.
        - Récupère les données depuis les providers ou le cache.
        - Normalise et valide les données.
        - Ajoute des données dérivées et des alertes.
        """
        await self.cache.connect()
        observations: List[Observation] = []
        alerts: List[Alert] = []

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

            except ProviderError as e:
                error_message = str(e)
                self.error_handler.log_error(
                    provider=provider_name,
                    error_type=ErrorType.CONNECTION_ERROR,
                    message=error_message,
                    severity=ErrorSeverity.ERROR
                )
                # Fallback vers le cache
                cached_observation = await self.cache.get(provider_name, location)
                if cached_observation:
                    observations.append(cached_observation)
                else:
                    # Utiliser une valeur par défaut
                    observations.append(self._get_default_observation(provider_name))

            except NormalizationError as e:
                error_message = str(e)
                self.error_handler.log_error(
                    provider=provider_name,
                    error_type=ErrorType.INVALID_DATA,
                    message=error_message,
                    severity=ErrorSeverity.WARNING
                )
                # Fallback vers le cache
                cached_observation = await self.cache.get(provider_name, location)
                if cached_observation:
                    observations.append(cached_observation)

            except ValidationError as e:
                error_message = str(e)
                self.error_handler.log_error(
                    provider=provider_name,
                    error_type=ErrorType.INVALID_DATA,
                    message=error_message,
                    severity=ErrorSeverity.WARNING
                )
                # Fallback vers le cache
                cached_observation = await self.cache.get(provider_name, location)
                if cached_observation:
                    observations.append(cached_observation)

            except Exception as e:
                error_message = str(e)
                self.error_handler.log_error(
                    provider=provider_name,
                    error_type=ErrorType.UNKNOWN,
                    message=error_message,
                    severity=ErrorSeverity.ERROR
                )

            finally:
                await adapter.close()

        # Vérifier les pannes critiques
        critical_alert = self.error_handler.check_critical_failure()
        if critical_alert:
            alerts.append(critical_alert)

        # Ajouter les alertes existantes
        alerts.extend(self.error_handler.alerts)

        await self.cache.close()

        derived_data = self._calculate_derived_data(observations)

        return UnifiedContext(
            observations=observations,
            timestamp=datetime.now(timezone.utc),
            user_profile=user_profile,
            derived_data=derived_data,
            alerts=alerts
        )

    def _get_ttl(self, provider_name: str) -> int:
        """Retourne le TTL en secondes selon le provider."""
        ttl_map = {
            "atmo-france": 300,
            "rte": 300,
            "hubeau": 3600,
            "openstreetmap": 86400
        }
        return ttl_map.get(provider_name, 300)

    def _calculate_derived_data(self, observations: List[Observation]) -> dict:
        """Calcule les données dérivées à partir des observations."""
        derived_data = {}

        aqi_values = []
        for obs in observations:
            if obs.provider == "atmo-france" and "aqi" in obs.value:
                aqi_values.append(obs.value["aqi"])
        if aqi_values:
            derived_data["global_aqi"] = sum(aqi_values) / len(aqi_values)

        for obs in observations:
            if obs.provider == "rte" and "mix" in obs.value:
                derived_data["energy_mix"] = obs.value["mix"]
                break

        water_levels = []
        for obs in observations:
            if obs.provider == "hubeau" and "niveau" in obs.value:
                water_levels.append(obs.value["niveau"])
        if water_levels:
            derived_data["avg_water_level"] = sum(water_levels) / len(water_levels)

        return derived_data

    def _get_default_observation(self, provider: str) -> Observation:
        """Retourne une observation par défaut pour un provider."""
        return Observation(
            provider=provider,
            observedAt=datetime.now(timezone.utc),
            retrievedAt=datetime.now(timezone.utc),
            expiresAt=datetime.now(timezone.utc),
            quality=0.0,
            value={}
        )
```

---

## Tests Unitaires
Créer un fichier `tests/unit/test_error_handling.py` avec les tests suivants :

```python
import pytest
from datetime import datetime, timedelta
from unittest.mock import AsyncMock, patch, MagicMock
import sys
import os

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '../../backend'))

from backend.engines.context_engine.errors import (
    ProviderErrorHandler, 
    ErrorType, 
    ErrorSeverity,
    Alert
)
from backend.engines.context_engine.models import Observation


@pytest.fixture
def error_handler():
    return ProviderErrorHandler()


# Test de log d'erreur
def test_log_error(error_handler):
    error_handler.log_error(
        provider="atmo-france",
        error_type=ErrorType.CONNECTION_ERROR,
        message="Connection failed",
        severity=ErrorSeverity.ERROR
    )
    assert "atmo-france" in error_handler.errors
    assert len(error_handler.errors["atmo-france"]) == 1
    assert error_handler.errors["atmo-france"][0]["type"] == "connection_error"


# Test de détection de panne critique
def test_critical_failure_detection(error_handler):
    # Simuler des erreurs pour 2 providers critiques sur 3
    error_handler.log_error(
        provider="atmo-france",
        error_type=ErrorType.API_UNAVAILABLE,
        message="API down",
        severity=ErrorSeverity.ERROR
    )
    error_handler.log_error(
        provider="rte",
        error_type=ErrorType.API_UNAVAILABLE,
        message="API down",
        severity=ErrorSeverity.ERROR
    )

    alert = error_handler.check_critical_failure()
    assert alert is not None
    assert alert.type == "critical_failure"
    assert alert.severity == "critical"
    assert alert.details["failed_critical"] == 2


# Test de non-détection de panne critique
def test_no_critical_failure(error_handler):
    # Simuler une erreur pour 1 provider critique sur 3
    error_handler.log_error(
        provider="atmo-france",
        error_type=ErrorType.API_UNAVAILABLE,
        message="API down",
        severity=ErrorSeverity.ERROR
    )

    alert = error_handler.check_critical_failure()
    assert alert is None


# Test de comptage des erreurs
def test_get_error_count(error_handler):
    error_handler.log_error(
        provider="atmo-france",
        error_type=ErrorType.CONNECTION_ERROR,
        message="Connection failed",
        severity=ErrorSeverity.ERROR
    )
    error_handler.log_error(
        provider="atmo-france",
        error_type=ErrorType.TIMEOUT,
        message="Timeout",
        severity=ErrorSeverity.WARNING
    )
    assert error_handler.get_error_count("atmo-france") == 2
    assert error_handler.get_error_count("rte") == 0


# Test de récupération de toutes les erreurs
def test_get_all_errors(error_handler):
    error_handler.log_error(
        provider="atmo-france",
        error_type=ErrorType.CONNECTION_ERROR,
        message="Connection failed",
        severity=ErrorSeverity.ERROR
    )
    error_handler.log_error(
        provider="rte",
        error_type=ErrorType.RATE_LIMIT,
        message="Rate limit exceeded",
        severity=ErrorSeverity.WARNING
    )

    errors = error_handler.get_all_errors()
    assert len(errors) == 2
    assert "atmo-france" in errors
    assert "rte" in errors


# Test d'effacement des erreurs
def test_clear_errors(error_handler):
    error_handler.log_error(
        provider="atmo-france",
        error_type=ErrorType.CONNECTION_ERROR,
        message="Connection failed",
        severity=ErrorSeverity.ERROR
    )
    error_handler.clear_errors("atmo-france")
    assert error_handler.get_error_count("atmo-france") == 0


# Test d'effacement de toutes les erreurs
def test_clear_all_errors(error_handler):
    error_handler.log_error(
        provider="atmo-france",
        error_type=ErrorType.CONNECTION_ERROR,
        message="Connection failed",
        severity=ErrorSeverity.ERROR
    )
    error_handler.log_error(
        provider="rte",
        error_type=ErrorType.RATE_LIMIT,
        message="Rate limit exceeded",
        severity=ErrorSeverity.WARNING
    )
    error_handler.clear_errors()
    assert len(error_handler.get_all_errors()) == 0
```

---

## Configuration Requise
Aucune configuration supplémentaire nécessaire. Utilise les variables existantes dans `.env`.

---

## Dépendances
- **Python 3.11+**
- **Librairies** :
  - Aucune dépendance supplémentaire (utilise le module `logging` standard).

---

## Notes
- **Providers critiques** : Atmo France, RTE et Hub'Eau sont considérés comme critiques. OpenStreetMap est optionnel.
- **Seuil d'alerte** : >50% des providers critiques en échec → alerte critique.
- **Fallback** : En cas d'échec, le système utilise le cache Redis ou une valeur par défaut.
- **Logs** : Les erreurs sont loguées avec des niveaux appropriés (WARNING, ERROR, CRITICAL).
- **Extensibilité** : Le système peut être étendu pour envoyer des alertes à des services externes (ex: Sentry, Datadog).

---

## Ressources
- [PRD Almanéa - FR-5](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AD-4](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
