---
name: Implémenter Rule Engine
id: 2-1-implémenter-rule-engine
epic: epic-2
story_type: backend
priority: high
estimation: L
dependencies: [1-1, 1-2, 1-3, 1-4, 1-5, 1-6]
status: ready-for-dev
created: 2026-08-16
updated: 2026-08-16
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/implementation-artifacts/epic-1-context.md
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 2.1: Implémenter `Rule Engine`

## Contexte
Cette story fait partie de **l'Épic 2 (Recommendation Engine)**. Son objectif est de créer un **moteur de règles déterministes** pour générer des **candidats de recommandations** à partir d'un `UnifiedContext` et d'un `UserProfile`, conformément à **FR-6 (Génération de candidats de recommandations)** et **FR-10 (Recommandations déterministes)**.

Le `Rule Engine` est le **cœur du Recommendation Engine** : il applique des règles métiers pour proposer des actions pertinentes (ex: "Balade à vélo", "Arrosage des plantes", "Réparer un objet") en fonction du contexte environnemental et des préférences utilisateur.

---

## Exigences Fonctionnelles
- **FR-6**: Génération de candidats de recommandations via `Rule Engine` (règles déterministes).
- **FR-10**: Génération de recommandations déterministes (sans LLM pour le core).
- **NFR-1**: Déterminisme — Les recommandations doivent être reproductibles.
- **AR-5**: Abstraction pour les providers (APIs) et LLMs.

---

## Critères d'Acceptation
1. **Génération de candidats** :
   - [ ] Le `Rule Engine` génère une liste de **candidats de recommandations** à partir d'un `UnifiedContext` et d'un `UserProfile`.
   - [ ] Chaque candidat est **valide** (ex: pas de recommandation de vélo si la qualité de l'air est mauvaise).
   - [ ] Les candidats sont **testables indépendamment du LLM**.

2. **Règles déterministes** :
   - [ ] Les règles sont **déterministes** (ex: `IF air_quality = GOOD AND rain_probability < 30% THEN candidate = BIKE_ACTIVITY`).
   - [ ] Les mêmes entrées (`UnifiedContext`, `UserProfile`) produisent **les mêmes candidats**.

3. **Types de recommandations** :
   - [ ] Le `Rule Engine` supporte au moins les **catégories suivantes** :
     - **Mobilité** (ex: vélo, marche, covoiturage).
     - **Nature** (ex: balade en forêt, jardinage).
     - **Énergie** (ex: réduire la consommation, utiliser des énergies renouvelables).
     - **Eau** (ex: arrosage, économie d'eau).
     - **Réparation/Réemploi** (ex: réparer un objet, donner des vêtements).
     - **Bien-être** (ex: méditation, yoga).

4. **Intégration avec `UnifiedContext`** :
   - [ ] Le `Rule Engine` utilise les **données du `UnifiedContext`** (ex: `global_aqi`, `energy_mix`, `avg_water_level`).
   - [ ] Le `Rule Engine` utilise les **préférences utilisateur** (ex: `user.likes_cycling`, `user.max_duration`).

5. **Extensibilité** :
   - [ ] Les règles sont **modulares** (ajout/suppression sans modifier le code existant).
   - [ ] Les règles sont **configurables** (ex: via un fichier JSON ou une base de données).

6. **Validation des candidats** :
   - [ ] Chaque candidat est **validé** avant d'être ajouté à la liste (ex: vérifier que les conditions météo sont toujours valides).
   - [ ] Les candidats invalides sont **exclus** ou **marqués comme non pertinents**.

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/rules/__init__.py` | Module des règles | À créer |
| `backend/engines/recommendation_engine/rules/base.py` | Classe de base `Rule` | À créer |
| `backend/engines/recommendation_engine/rules/rule_engine.py` | Moteur `RuleEngine` | À créer |
| `backend/engines/recommendation_engine/rules/mobility.py` | Règles pour la mobilité | À créer |
| `backend/engines/recommendation_engine/rules/nature.py` | Règles pour la nature | À créer |
| `backend/engines/recommendation_engine/rules/energy.py` | Règles pour l'énergie | À créer |
| `backend/engines/recommendation_engine/rules/water.py` | Règles pour l'eau | À créer |
| `backend/engines/recommendation_engine/rules/repair.py` | Règles pour la réparation/réemploi | À créer |
| `backend/engines/recommendation_engine/rules/wellbeing.py` | Règles pour le bien-être | À créer |
| `backend/engines/recommendation_engine/models.py` | Modèle `Recommendation` | À créer |
| `backend/engines/recommendation_engine/__init__.py` | Initialisation du module | À créer |
| `tests/unit/test_rule_engine.py` | Tests unitaires | À créer |

### Structure du module

#### 1. Modèle `Recommendation` (`models.py`)
```python
from datetime import datetime
from typing import Dict, Any, List, Optional
from enum import Enum
from pydantic import BaseModel, Field


class RecommendationCategory(Enum):
    """Catégories de recommandations."""
    MOBILITY = "mobility"
    NATURE = "nature"
    ENERGY = "energy"
    WATER = "water"
    REPAIR = "repair"
    WELLBEING = "wellbeing"


class Recommendation(BaseModel):
    """Représente une recommandation générée par le Rule Engine."""
    id: str = Field(..., description="ID unique de la recommandation")
    title: str = Field(..., description="Titre de la recommandation")
    description: str = Field(..., description="Description détaillée")
    category: RecommendationCategory = Field(..., description="Catégorie de la recommandation")
    action: str = Field(..., description="Action recommandée (ex: 'Faire du vélo')")
    duration_minutes: Optional[int] = Field(None, description="Durée estimée en minutes")
    location: Optional[str] = Field(None, description="Localisation (ex: 'Parc de la Tête d'Or')")
    conditions: Dict[str, Any] = Field(
        default_factory=dict,
        description="Conditions qui ont déclenché la recommandation (ex: {'aqi': 42, 'rain_probability': 10})"
    )
    score: Optional[float] = Field(None, description="Score de pertinence (0-1)")
    metadata: Dict[str, Any] = Field(
        default_factory=dict,
        description="Métadonnées supplémentaires (ex: distance, difficulté)"
    )
    created_at: datetime = Field(default_factory=datetime.now, description="Horodatage de la création")
```

#### 2. Classe de base `Rule` (`base.py`)
```python
from abc import ABC, abstractmethod
from typing import List, Optional
from ..models import Recommendation, UnifiedContext, UserProfile


class Rule(ABC):
    """Classe de base pour une règle du Rule Engine."""

    def __init__(self, rule_id: str, category: str, priority: int = 0):
        self.rule_id = rule_id
        self.category = category
        self.priority = priority  # Priorité pour le classement

    @abstractmethod
    def evaluate(self, context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> Optional[Recommendation]:
        """
        Évalue si la règle s'applique au contexte donné.
        Retourne une Recommendation si la règle est déclenchée, sinon None.
        """
        pass

    @abstractmethod
    def get_conditions(self) -> dict:
        """Retourne les conditions de la règle sous forme de dictionnaire."""
        pass
```

#### 3. Moteur `RuleEngine` (`rule_engine.py`)
```python
from typing import List, Optional
from ..models import UnifiedContext, UserProfile, Recommendation
from .base import Rule


class RuleEngine:
    """Moteur de règles pour générer des candidats de recommandations."""

    def __init__(self):
        self.rules: List[Rule] = []

    def add_rule(self, rule: Rule) -> None:
        """Ajoute une règle au moteur."""
        self.rules.append(rule)

    def remove_rule(self, rule_id: str) -> bool:
        """Supprime une règle du moteur."""
        for i, rule in enumerate(self.rules):
            if rule.rule_id == rule_id:
                self.rules.pop(i)
                return True
        return False

    def get_rules_by_category(self, category: str) -> List[Rule]:
        """Retourne les règles d'une catégorie spécifique."""
        return [rule for rule in self.rules if rule.category == category]

    def evaluate_all(self, context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> List[Recommendation]:
        """
        Évalue toutes les règles et retourne une liste de recommandations.
        """
        recommendations = []
        for rule in self.rules:
            recommendation = rule.evaluate(context, user_profile)
            if recommendation:
                recommendations.append(recommendation)
        return recommendations

    def clear_rules(self) -> None:
        """Supprime toutes les règles."""
        self.rules.clear()
```

#### 4. Exemples de règles (`mobility.py`, `nature.py`, etc.)

##### Règles pour la mobilité (`mobility.py`)
```python
from typing import Optional
from ..models import Recommendation, UnifiedContext, UserProfile, RecommendationCategory
from .base import Rule


class BikeActivityRule(Rule):
    """Règle pour recommander une balade à vélo."""

    def __init__(self):
        super().__init__(
            rule_id="bike_activity",
            category=RecommendationCategory.MOBILITY.value,
            priority=10
        )

    def evaluate(self, context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> Optional[Recommendation]:
        # Conditions pour déclencher la recommandation
        aqi = context.derived_data.get("global_aqi", 50)
        rain_probability = context.get_value("atmo-france", "rain_probability", 0)
        wind_speed = context.get_value("atmo-france", "wind_speed", 0)

        # Vérifier les conditions
        if aqi <= 50 and rain_probability < 30 and wind_speed < 20:
            # Vérifier les préférences utilisateur
            if user_profile and not user_profile.preferences.get("likes_cycling", True):
                return None

            return Recommendation(
                id="bike_activity_001",
                title="Balade à vélo",
                description="Profitez de la bonne qualité de l'air pour une balade à vélo.",
                category=RecommendationCategory.MOBILITY,
                action="Faire du vélo",
                duration_minutes=30,
                conditions={
                    "aqi": aqi,
                    "rain_probability": rain_probability,
                    "wind_speed": wind_speed
                },
                metadata={
                    "difficulty": "easy",
                    "estimated_distance_km": 5.0
                }
            )
        return None

    def get_conditions(self) -> dict:
        return {
            "aqi": {"max": 50},
            "rain_probability": {"max": 30},
            "wind_speed": {"max": 20}
        }


class WalkActivityRule(Rule):
    """Règle pour recommander une marche."""

    def __init__(self):
        super().__init__(
            rule_id="walk_activity",
            category=RecommendationCategory.MOBILITY.value,
            priority=5
        )

    def evaluate(self, context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> Optional[Recommendation]:
        aqi = context.derived_data.get("global_aqi", 50)
        rain_probability = context.get_value("atmo-france", "rain_probability", 0)

        if aqi <= 70 and rain_probability < 50:
            return Recommendation(
                id="walk_activity_001",
                title="Promenade à pied",
                description="Une marche est idéale avec la qualité de l'air actuelle.",
                category=RecommendationCategory.MOBILITY,
                action="Marcher",
                duration_minutes=20,
                conditions={
                    "aqi": aqi,
                    "rain_probability": rain_probability
                }
            )
        return None

    def get_conditions(self) -> dict:
        return {
            "aqi": {"max": 70},
            "rain_probability": {"max": 50}
        }


class CarpoolingRule(Rule):
    """Règle pour recommander le covoiturage."""

    def __init__(self):
        super().__init__(
            rule_id="carpooling",
            category=RecommendationCategory.MOBILITY.value,
            priority=8
        )

    def evaluate(self, context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> Optional[Recommendation]:
        # Exemple : Recommander le covoiturage si la qualité de l'air est moyenne
        aqi = context.derived_data.get("global_aqi", 50)

        if 50 < aqi <= 100:
            return Recommendation(
                id="carpooling_001",
                title="Covoiturage",
                description="Réduisez votre empreinte carbone en partageant votre trajet.",
                category=RecommendationCategory.MOBILITY,
                action="Faire du covoiturage",
                conditions={"aqi": aqi},
                metadata={"carbon_savings_kg": 2.5}
            )
        return None

    def get_conditions(self) -> dict:
        return {"aqi": {"min": 50, "max": 100}}
```

##### Règles pour la nature (`nature.py`)
```python
from typing import Optional
from ..models import Recommendation, UnifiedContext, UserProfile, RecommendationCategory
from .base import Rule


class ForestWalkRule(Rule):
    """Règle pour recommander une balade en forêt."""

    def __init__(self):
        super().__init__(
            rule_id="forest_walk",
            category=RecommendationCategory.NATURE.value,
            priority=10
        )

    def evaluate(self, context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> Optional[Recommendation]:
        aqi = context.derived_data.get("global_aqi", 50)
        rain_probability = context.get_value("atmo-france", "rain_probability", 0)

        if aqi <= 50 and rain_probability < 20:
            return Recommendation(
                id="forest_walk_001",
                title="Balade en forêt",
                description="Explorez les sentiers locaux avec une excellente qualité de l'air.",
                category=RecommendationCategory.NATURE,
                action="Balade en forêt",
                duration_minutes=45,
                location="Forêt locale",
                conditions={"aqi": aqi, "rain_probability": rain_probability},
                metadata={"difficulty": "medium", "biodiversity_benefit": "high"}
            )
        return None

    def get_conditions(self) -> dict:
        return {"aqi": {"max": 50}, "rain_probability": {"max": 20}}


class GardeningRule(Rule):
    """Règle pour recommander du jardinage."""

    def __init__(self):
        super().__init__(
            rule_id="gardening",
            category=RecommendationCategory.NATURE.value,
            priority=7
        )

    def evaluate(self, context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> Optional[Recommendation]:
        aqi = context.derived_data.get("global_aqi", 50)
        rain_probability = context.get_value("atmo-france", "rain_probability", 0)
        temperature = context.get_value("atmo-france", "temperature", 20)

        if aqi <= 70 and rain_probability < 40 and 15 <= temperature <= 30:
            return Recommendation(
                id="gardening_001",
                title="Jardinage",
                description="Conditions idéales pour jardiner aujourd'hui.",
                category=RecommendationCategory.NATURE,
                action="Jardiner",
                duration_minutes=60,
                conditions={"aqi": aqi, "rain_probability": rain_probability, "temperature": temperature},
                metadata={"difficulty": "easy", "water_needed": False}
            )
        return None

    def get_conditions(self) -> dict:
        return {
            "aqi": {"max": 70},
            "rain_probability": {"max": 40},
            "temperature": {"min": 15, "max": 30}
        }
```

##### Règles pour l'énergie (`energy.py`)
```python
from typing import Optional
from ..models import Recommendation, UnifiedContext, UserProfile, RecommendationCategory
from .base import Rule


class ReduceEnergyConsumptionRule(Rule):
    """Règle pour recommander de réduire la consommation d'énergie."""

    def __init__(self):
        super().__init__(
            rule_id="reduce_energy",
            category=RecommendationCategory.ENERGY.value,
            priority=10
        )

    def evaluate(self, context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> Optional[Recommendation]:
        energy_mix = context.derived_data.get("energy_mix", {})
        co2 = context.get_value("rte", "co2", 0)

        # Recommander si le mix énergétique est très carboné
        if co2 > 500:  # Seuil élevé de CO2 (g/kWh)
            return Recommendation(
                id="reduce_energy_001",
                title="Réduire la consommation d'énergie",
                description=f"Le mix énergétique actuel émet {co2} gCO2/kWh. Éteignez les appareils inutiles.",
                category=RecommendationCategory.ENERGY,
                action="Réduire la consommation",
                conditions={"co2": co2, "energy_mix": energy_mix},
                metadata={"estimated_savings_kg": 1.5}
            )
        return None

    def get_conditions(self) -> dict:
        return {"co2": {"min": 500}}


class UseRenewableEnergyRule(Rule):
    """Règle pour recommander d'utiliser des énergies renouvelables."""

    def __init__(self):
        super().__init__(
            rule_id="use_renewable",
            category=RecommendationCategory.ENERGY.value,
            priority=8
        )

    def evaluate(self, context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> Optional[Recommendation]:
        energy_mix = context.derived_data.get("energy_mix", {})

        # Recommander si le mix contient peu d'énergies renouvelables
        renewable_percentage = energy_mix.get("renewable", 0) + energy_mix.get("hydro", 0) + energy_mix.get("wind", 0) + energy_mix.get("solar", 0)

        if renewable_percentage < 30:
            return Recommendation(
                id="use_renewable_001",
                title="Utiliser des énergies renouvelables",
                description=f"Seulement {renewable_percentage}% d'énergies renouvelables dans le mix. Privilégiez les appareils solaires ou éoliens.",
                category=RecommendationCategory.ENERGY,
                action="Utiliser des énergies renouvelables",
                conditions={"renewable_percentage": renewable_percentage},
                metadata={"renewable_types": ["solar", "wind", "hydro"]}
            )
        return None

    def get_conditions(self) -> dict:
        return {"renewable_percentage": {"max": 30}}
```

##### Règles pour l'eau (`water.py`)
```python
from typing import Optional
from ..models import Recommendation, UnifiedContext, UserProfile, RecommendationCategory
from .base import Rule


class WateringRule(Rule):
    """Règle pour recommander l'arrosage des plantes."""

    def __init__(self):
        super().__init__(
            rule_id="watering",
            category=RecommendationCategory.WATER.value,
            priority=10
        )

    def evaluate(self, context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> Optional[Recommendation]:
        avg_water_level = context.derived_data.get("avg_water_level", 0)
        rain_probability = context.get_value("atmo-france", "rain_probability", 0)

        # Recommander l'arrosage si le niveau d'eau est bas et qu'il ne pleut pas
        if avg_water_level < 50 and rain_probability < 10:
            return Recommendation(
                id="watering_001",
                title="Arroser les plantes",
                description="Niveau d'eau bas et pas de pluie prévue. Pensez à arroser vos plantes.",
                category=RecommendationCategory.WATER,
                action="Arroser les plantes",
                duration_minutes=15,
                conditions={"avg_water_level": avg_water_level, "rain_probability": rain_probability},
                metadata={"water_savings_liters": 10}
            )
        return None

    def get_conditions(self) -> dict:
        return {"avg_water_level": {"max": 50}, "rain_probability": {"max": 10}}


class SaveWaterRule(Rule):
    """Règle pour recommander d'économiser l'eau."""

    def __init__(self):
        super().__init__(
            rule_id="save_water",
            category=RecommendationCategory.WATER.value,
            priority=8
        )

    def evaluate(self, context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> Optional[Recommendation]:
        avg_water_level = context.derived_data.get("avg_water_level", 0)

        # Recommander d'économiser l'eau si le niveau est très bas
        if avg_water_level < 20:
            return Recommendation(
                id="save_water_001",
                title="Économiser l'eau",
                description="Niveau d'eau critique. Évitez les usages non essentiels (ex: lavage de voiture).",
                category=RecommendationCategory.WATER,
                action="Économiser l'eau",
                conditions={"avg_water_level": avg_water_level},
                metadata={"urgency": "high"}
            )
        return None

    def get_conditions(self) -> dict:
        return {"avg_water_level": {"max": 20}}
```

##### Règles pour la réparation/réemploi (`repair.py`)
```python
from typing import Optional
from ..models import Recommendation, UnifiedContext, UserProfile, RecommendationCategory
from .base import Rule


class RepairRule(Rule):
    """Règle pour recommander de réparer un objet."""

    def __init__(self):
        super().__init__(
            rule_id="repair",
            category=RecommendationCategory.REPAIR.value,
            priority=10
        )

    def evaluate(self, context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> Optional[Recommendation]:
        # Exemple : Recommander la réparation si l'utilisateur a des préférences pour ça
        if user_profile and user_profile.preferences.get("likes_repair", False):
            return Recommendation(
                id="repair_001",
                title="Réparer un objet",
                description="Donnez une seconde vie à vos objets en les réparant.",
                category=RecommendationCategory.REPAIR,
                action="Réparer un objet",
                duration_minutes=60,
                conditions={"user_likes_repair": True},
                metadata={"difficulty": "medium", "environmental_impact": "high"}
            )
        return None

    def get_conditions(self) -> dict:
        return {"user_likes_repair": True}


class ReuseRule(Rule):
    """Règle pour recommander le réemploi."""

    def __init__(self):
        super().__init__(
            rule_id="reuse",
            category=RecommendationCategory.REPAIR.value,
            priority=8
        )

    def evaluate(self, context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> Optional[Recommendation]:
        if user_profile and user_profile.preferences.get("likes_reuse", False):
            return Recommendation(
                id="reuse_001",
                title="Réemployer un objet",
                description="Donnez ou vendez vos objets inutilisés au lieu de les jeter.",
                category=RecommendationCategory.REPAIR,
                action="Réemployer un objet",
                duration_minutes=30,
                conditions={"user_likes_reuse": True},
                metadata={"difficulty": "easy", "environmental_impact": "high"}
            )
        return None

    def get_conditions(self) -> dict:
        return {"user_likes_reuse": True}
```

##### Règles pour le bien-être (`wellbeing.py`)
```python
from typing import Optional
from ..models import Recommendation, UnifiedContext, UserProfile, RecommendationCategory
from .base import Rule


class MeditationRule(Rule):
    """Règle pour recommander la méditation."""

    def __init__(self):
        super().__init__(
            rule_id="meditation",
            category=RecommendationCategory.WELLBEING.value,
            priority=10
        )

    def evaluate(self, context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> Optional[Recommendation]:
        aqi = context.derived_data.get("global_aqi", 50)

        # Recommander la méditation si la qualité de l'air est bonne (pour une séance en extérieur)
        if aqi <= 50:
            return Recommendation(
                id="meditation_001",
                title="Méditation en plein air",
                description="Profitez de l'air pur pour une séance de méditation.",
                category=RecommendationCategory.WELLBEING,
                action="Méditer",
                duration_minutes=20,
                conditions={"aqi": aqi},
                metadata={"difficulty": "easy", "wellbeing_benefit": "high"}
            )
        return None

    def get_conditions(self) -> dict:
        return {"aqi": {"max": 50}}


class YogaRule(Rule):
    """Règle pour recommander le yoga."""

    def __init__(self):
        super().__init__(
            rule_id="yoga",
            category=RecommendationCategory.WELLBEING.value,
            priority=8
        )

    def evaluate(self, context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> Optional[Recommendation]:
        aqi = context.derived_data.get("global_aqi", 50)

        if aqi <= 70:
            return Recommendation(
                id="yoga_001",
                title="Séance de yoga",
                description="Une séance de yoga pour vous détendre.",
                category=RecommendationCategory.WELLBEING,
                action="Faire du yoga",
                duration_minutes=30,
                conditions={"aqi": aqi},
                metadata={"difficulty": "medium", "wellbeing_benefit": "high"}
            )
        return None

    def get_conditions(self) -> dict:
        return {"aqi": {"max": 70}}
```

#### 5. Initialisation du `RuleEngine` (`__init__.py`)
```python
from .rule_engine import RuleEngine
from .mobility import BikeActivityRule, WalkActivityRule, CarpoolingRule
from .nature import ForestWalkRule, GardeningRule
from .energy import ReduceEnergyConsumptionRule, UseRenewableEnergyRule
from .water import WateringRule, SaveWaterRule
from .repair import RepairRule, ReuseRule
from .wellbeing import MeditationRule, YogaRule


# Créer une instance du RuleEngine avec toutes les règles par défaut
def create_rule_engine() -> RuleEngine:
    engine = RuleEngine()

    # Ajouter les règles de mobilité
    engine.add_rule(BikeActivityRule())
    engine.add_rule(WalkActivityRule())
    engine.add_rule(CarpoolingRule())

    # Ajouter les règles de nature
    engine.add_rule(ForestWalkRule())
    engine.add_rule(GardeningRule())

    # Ajouter les règles d'énergie
    engine.add_rule(ReduceEnergyConsumptionRule())
    engine.add_rule(UseRenewableEnergyRule())

    # Ajouter les règles d'eau
    engine.add_rule(WateringRule())
    engine.add_rule(SaveWaterRule())

    # Ajouter les règles de réparation/réemploi
    engine.add_rule(RepairRule())
    engine.add_rule(ReuseRule())

    # Ajouter les règles de bien-être
    engine.add_rule(MeditationRule())
    engine.add_rule(YogaRule())

    return engine


# Instance globale du RuleEngine (optionnel)
rule_engine = create_rule_engine()
```

---

## Tests Unitaires
Créer un fichier `tests/unit/test_rule_engine.py` avec les tests suivants :

```python
import pytest
from datetime import datetime, timedelta
from unittest.mock import AsyncMock, patch, MagicMock
import sys
import os

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '../../backend'))

from backend.engines.recommendation_engine.models import UnifiedContext, Observation, UserProfile, RecommendationCategory
from backend.engines.recommendation_engine.rules.rule_engine import RuleEngine
from backend.engines.recommendation_engine.rules.mobility import BikeActivityRule, WalkActivityRule
from backend.engines.recommendation_engine.rules.nature import ForestWalkRule
from backend.engines.recommendation_engine.rules.energy import ReduceEnergyConsumptionRule
from backend.engines.recommendation_engine.rules.water import WateringRule
from backend.engines.recommendation_engine.rules.repair import RepairRule
from backend.engines.recommendation_engine.rules.wellbeing import MeditationRule


@pytest.fixture
def mock_unified_context():
    observations = [
        Observation(
            provider="atmo-france",
            observedAt=datetime.now(),
            retrievedAt=datetime.now(),
            expiresAt=datetime.now() + timedelta(hours=1),
            quality=0.95,
            value={"aqi": 42, "polluant": "PM2.5", "rain_probability": 10, "wind_speed": 5}
        ),
        Observation(
            provider="rte",
            observedAt=datetime.now(),
            retrievedAt=datetime.now(),
            expiresAt=datetime.now() + timedelta(hours=1),
            quality=0.95,
            value={"mix": {"nuclear": 0.5, "hydro": 0.3}, "co2": 100}
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
    return UnifiedContext(
        observations=observations,
        timestamp=datetime.now(),
        derived_data={
            "global_aqi": 42,
            "energy_mix": {"nuclear": 0.5, "hydro": 0.3},
            "avg_water_level": 120.5
        }
    )


@pytest.fixture
def mock_user_profile():
    return UserProfile(
        user_id="user123",
        location="69003",
        preferences={
            "likes_cycling": True,
            "likes_repair": True,
            "likes_reuse": False
        }
    )


@pytest.fixture
def rule_engine():
    engine = RuleEngine()
    engine.add_rule(BikeActivityRule())
    engine.add_rule(WalkActivityRule())
    engine.add_rule(ForestWalkRule())
    engine.add_rule(ReduceEnergyConsumptionRule())
    engine.add_rule(WateringRule())
    engine.add_rule(RepairRule())
    engine.add_rule(MeditationRule())
    return engine


# Test de création du RuleEngine
def test_create_rule_engine(rule_engine):
    assert len(rule_engine.rules) == 7


# Test d'ajout de règle
def test_add_rule(rule_engine):
    from backend.engines.recommendation_engine.rules.wellbeing import YogaRule
    rule_engine.add_rule(YogaRule())
    assert len(rule_engine.rules) == 8


# Test de suppression de règle
def test_remove_rule(rule_engine):
    assert rule_engine.remove_rule("bike_activity") is True
    assert len(rule_engine.rules) == 6


# Test de suppression de règle inexistante
def test_remove_nonexistent_rule(rule_engine):
    assert rule_engine.remove_rule("nonexistent") is False


# Test de récupération des règles par catégorie
def test_get_rules_by_category(rule_engine):
    mobility_rules = rule_engine.get_rules_by_category("mobility")
    assert len(mobility_rules) == 2  # BikeActivityRule et WalkActivityRule


# Test d'évaluation d'une règle (BikeActivityRule)
def test_bike_activity_rule(mock_unified_context, mock_user_profile):
    rule = BikeActivityRule()
    recommendation = rule.evaluate(mock_unified_context, mock_user_profile)
    assert recommendation is not None
    assert recommendation.id == "bike_activity_001"
    assert recommendation.category == RecommendationCategory.MOBILITY
    assert recommendation.title == "Balade à vélo"


# Test d'évaluation d'une règle (BikeActivityRule - conditions non remplies)
def test_bike_activity_rule_not_triggered():
    observations = [
        Observation(
            provider="atmo-france",
            observedAt=datetime.now(),
            retrievedAt=datetime.now(),
            expiresAt=datetime.now() + timedelta(hours=1),
            quality=0.95,
            value={"aqi": 150, "polluant": "PM2.5", "rain_probability": 10, "wind_speed": 5}  # AQI trop élevé
        )
    ]
    context = UnifiedContext(
        observations=observations,
        timestamp=datetime.now(),
        derived_data={"global_aqi": 150}
    )
    rule = BikeActivityRule()
    recommendation = rule.evaluate(context)
    assert recommendation is None


# Test d'évaluation d'une règle (RepairRule - dépend des préférences utilisateur)
def test_repair_rule_with_preferences(mock_unified_context, mock_user_profile):
    rule = RepairRule()
    recommendation = rule.evaluate(mock_unified_context, mock_user_profile)
    assert recommendation is not None
    assert recommendation.id == "repair_001"


# Test d'évaluation d'une règle (RepairRule - sans préférences utilisateur)
def test_repair_rule_without_preferences(mock_unified_context):
    rule = RepairRule()
    recommendation = rule.evaluate(mock_unified_context, None)
    assert recommendation is None


# Test d'évaluation de toutes les règles
def test_evaluate_all_rules(rule_engine, mock_unified_context, mock_user_profile):
    recommendations = rule_engine.evaluate_all(mock_unified_context, mock_user_profile)
    assert len(recommendations) > 0
    assert any(rec.category == RecommendationCategory.MOBILITY for rec in recommendations)
    assert any(rec.category == RecommendationCategory.NATURE for rec in recommendations)


# Test de vidage des règles
def test_clear_rules(rule_engine):
    rule_engine.clear_rules()
    assert len(rule_engine.rules) == 0


# Test des conditions d'une règle
def test_get_conditions():
    rule = BikeActivityRule()
    conditions = rule.get_conditions()
    assert conditions["aqi"]["max"] == 50
    assert conditions["rain_probability"]["max"] == 30
    assert conditions["wind_speed"]["max"] == 20


# Test de déterminisme
def test_deterministic_recommendations(rule_engine, mock_unified_context, mock_user_profile):
    # Exécuter le RuleEngine deux fois avec les mêmes entrées
    recommendations1 = rule_engine.evaluate_all(mock_unified_context, mock_user_profile)
    recommendations2 = rule_engine.evaluate_all(mock_unified_context, mock_user_profile)

    # Vérifier que les mêmes recommandations sont générées
    assert len(recommendations1) == len(recommendations2)
    for rec1, rec2 in zip(recommendations1, recommendations2):
        assert rec1.id == rec2.id
        assert rec1.title == rec2.title
        assert rec1.category == rec2.category
```

---

## Configuration Requise
Aucune configuration supplémentaire nécessaire. Utilise les modèles existants (`UnifiedContext`, `UserProfile`) de l'**Épic 1**.

---

## Dépendances
- **Python 3.11+**
- **Librairies** :
  - `pydantic` (pour la validation des données)

Les dépendances sont déjà installées (voir `requirements.txt`).

---

## Notes
- **Déterminisme** : Les règles doivent être **100% déterministes** (mêmes entrées → mêmes sorties).
- **Extensibilité** : Le `RuleEngine` est conçu pour être **modulaire** (ajout/suppression de règles sans modifier le code existant).
- **Catégories** : Les recommandations sont classées par catégorie pour faciliter le filtrage et l'affichage.
- **Conditions** : Chaque règle définit ses propres conditions (ex: AQI < 50, pluie < 30%).
- **Préférences utilisateur** : Certaines règles dépendent des préférences utilisateur (ex: `likes_cycling`).

---

## Exemples de Règles Supplémentaires
Voici des idées pour étendre le `RuleEngine` :

### Règles pour la mobilité
- **Covoiturage** : Si la qualité de l'air est moyenne (50 < AQI ≤ 100).
- **Transports en commun** : Si la qualité de l'air est mauvaise (AQI > 100).
- **Télétravail** : Si la qualité de l'air est très mauvaise (AQI > 150).

### Règles pour la nature
- **Balade en montagne** : Si AQI ≤ 30 et pas de pluie.
- **Pique-nique** : Si AQI ≤ 50 et température entre 20°C et 30°C.
- **Observation des étoiles** : Si AQI ≤ 20 et nuit claire.

### Règles pour l'énergie
- **Utiliser des appareils bas consommation** : Si le mix énergétique est très carboné (CO2 > 800 g/kWh).
- **Éteindre les lumières** : Si la consommation énergétique est élevée (ex: en soirée).

### Règles pour l'eau
- **Récupération d'eau de pluie** : Si le niveau d'eau est bas (avg_water_level < 30).
- **Arrosage goutte-à-goutte** : Si le niveau d'eau est critique (avg_water_level < 20).

### Règles pour la réparation/réemploi
- **Donner des vêtements** : Si l'utilisateur a des préférences pour le réemploi.
- **Acheter en vrac** : Si l'utilisateur a des préférences pour le zéro déchet.

### Règles pour le bien-être
- **Respiration profonde** : Si la qualité de l'air est excellente (AQI ≤ 20).
- **Sieste** : Si la température est élevée (temp > 25°C) et AQI ≤ 50.

---

## Ressources
- [PRD Almanéa - FR-6](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [PRD Almanéa - FR-10](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-5](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
- [Épic 1 - Context Engine](file:///Users/julie/Projects/_bmad-output/implementation-artifacts/epic-1-context.md)
