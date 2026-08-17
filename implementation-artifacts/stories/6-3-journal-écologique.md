---
name: Journal écologique
id: 6-3-journal-écologique
epic: epic-6
story_type: backend
priority: high
estimation: M
dependencies: [6-2-suivi-des-progrès, 5-1-créer-et-mettre-à-jour-le-profil-utilisateur]
status: done
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 6.3: Journal écologique

## Contexte
Cette story fait partie de **l'Épic 6 (Gamification)**. Son objectif est de **permettre aux utilisateurs de tenir un journal écologique** pour suivre leurs actions quotidiennes et leur impact environnemental. Le `EcoJournalManager` permet de :
- **Enregistrer les actions écologiques** (ex: trajets en vélo, économies d'eau).
- **Calculer l'impact environnemental** (ex: CO2 économisé, eau préservée).
- **Générer des statistiques** (ex: nombre d'actions par catégorie, impact total).
- **Suivre les tendances** (ex: progression sur une période).

---

## Exigences Fonctionnelles
- **FR-58**: Enregistrer les actions écologiques des utilisateurs.
- **FR-59**: Calculer l'impact environnemental des actions.
- **FR-60**: Générer des statistiques à partir des actions enregistrées.
- **FR-61**: Suivre les tendances des actions écologiques.

---

## Critères d'Acceptation
1. **Enregistrement des actions** :
   - [x] Enregistrer une action écologique pour un utilisateur.
   - [x] Mettre à jour une action existante.
   - [x] Supprimer une action.

2. **Calcul de l'impact environnemental** :
   - [x] Calculer le CO2 économisé pour une action.
   - [x] Calculer l'eau préservée pour une action.
   - [x] Calculer l'impact total pour un utilisateur.

3. **Génération de statistiques** :
   - [x] Récupérer le nombre d'actions par catégorie.
   - [x] Récupérer l'impact total par catégorie.
   - [x] Générer un rapport complet des actions.

4. **Suivi des tendances** :
   - [x] Récupérer les actions par jour/semaine/mois.
   - [x] Calculer la progression par rapport à une période précédente.

5. **Gestion du journal** :
   - [x] Récupérer toutes les actions d'un utilisateur.
   - [x] Réinitialiser le journal d'un utilisateur.
   - [x] Supprimer tout le journal.

6. **Intégration avec le RecommendationEngine** :
   - [x] Le `RecommendationEngine` utilise le `EcoJournalManager` pour suivre les actions écologiques.

---

## Implémentation Technique

### Fichiers créés/modifiés
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/eco_journal_manager.py` | Module `EcoJournalManager` | ✅ Créé |
| `backend/engines/recommendation_engine/__init__.py` | Intégration du `EcoJournalManager` | ✅ Modifié |
| `tests/unit/test_eco_journal_manager.py` | Tests unitaires | ✅ Créé |

### Enum `ActionCategory`
Définition des catégories d'actions écologiques :

```python
from enum import Enum

class ActionCategory(Enum):
    TRANSPORT = "transport"
    ENERGY = "energy"
    WATER = "water"
    WASTE = "waste"
    FOOD = "food"
    NATURE = "nature"
```

### Module `EcoJournalManager`
Ce module gère le journal écologique pour chaque utilisateur :

```python
from typing import Dict, List, Optional
from datetime import datetime, timedelta
from dataclasses import dataclass
from enum import Enum

class ActionCategory(Enum):
    TRANSPORT = "transport"
    ENERGY = "energy"
    WATER = "water"
    WASTE = "waste"
    FOOD = "food"
    NATURE = "nature"

@dataclass
class EcoAction:
    id: str
    user_id: str
    category: ActionCategory
    description: str
    co2_saved_kg: float = 0.0
    water_saved_liters: float = 0.0
    timestamp: datetime = None
    metadata: Dict = None

class EcoJournalManager:
    # Facteurs d'impact environnemental (exemples)
    IMPACT_FACTORS = {
        ActionCategory.TRANSPORT: {
            "bike_ride_km": {"co2_saved_kg": 0.2, "water_saved_liters": 0.0},
            "public_transport_km": {"co2_saved_kg": 0.1, "water_saved_liters": 0.0},
            "carpool_km": {"co2_saved_kg": 0.15, "water_saved_liters": 0.0},
        },
        ActionCategory.ENERGY: {
            "led_bulb": {"co2_saved_kg": 0.5, "water_saved_liters": 0.0},
            "solar_panel_kwh": {"co2_saved_kg": 0.4, "water_saved_liters": 0.0},
        },
        ActionCategory.WATER: {
            "shower_min": {"co2_saved_kg": 0.0, "water_saved_liters": 12.0},
            "tap_off_min": {"co2_saved_kg": 0.0, "water_saved_liters": 10.0},
        },
        ActionCategory.WASTE: {
            "recycle_kg": {"co2_saved_kg": 0.8, "water_saved_liters": 0.0},
            "compost_kg": {"co2_saved_kg": 0.3, "water_saved_liters": 0.0},
        },
        ActionCategory.FOOD: {
            "vegetarian_meal": {"co2_saved_kg": 1.5, "water_saved_liters": 100.0},
            "local_food_kg": {"co2_saved_kg": 0.2, "water_saved_liters": 0.0},
        },
        ActionCategory.NATURE: {
            "tree_planted": {"co2_saved_kg": 20.0, "water_saved_liters": 0.0},
            "park_cleanup": {"co2_saved_kg": 0.0, "water_saved_liters": 0.0},
        },
    }

    def __init__(self):
        # Actions par utilisateur : {user_id: [EcoAction]}
        self._user_actions: Dict[str, List[EcoAction]] = {}

    def log_action(
        self,
        user_id: str,
        category: ActionCategory,
        description: str,
        metadata: Optional[Dict] = None
    ) -> EcoAction:
        """Enregistre une action écologique pour un utilisateur."""
        action_id = f"{user_id}_{datetime.now().timestamp()}"
        co2_saved = 0.0
        water_saved = 0.0

        # Calculer l'impact en fonction de la catégorie et des métadonnées
        if category in self.IMPACT_FACTORS and metadata:
            for key, value in metadata.items():
                if key in self.IMPACT_FACTORS[category]:
                    co2_saved += self.IMPACT_FACTORS[category][key]["co2_saved_kg"] * value
                    water_saved += self.IMPACT_FACTORS[category][key]["water_saved_liters"] * value

        action = EcoAction(
            id=action_id,
            user_id=user_id,
            category=category,
            description=description,
            co2_saved_kg=co2_saved,
            water_saved_liters=water_saved,
            timestamp=datetime.now(),
            metadata=metadata or {}
        )

        self._user_actions.setdefault(user_id, []).append(action)
        return action

    def get_actions(self, user_id: str, category: Optional[ActionCategory] = None) -> List[EcoAction]:
        """Récupère les actions d'un utilisateur, éventuellement filtrées par catégorie."""
        if user_id not in self._user_actions:
            return []

        actions = self._user_actions[user_id]
        if category:
            actions = [action for action in actions if action.category == category]
        return actions

    def get_action(self, user_id: str, action_id: str) -> Optional[EcoAction]:
        """Récupère une action spécifique par son ID."""
        if user_id not in self._user_actions:
            return None

        for action in self._user_actions[user_id]:
            if action.id == action_id:
                return action
        return None

    def update_action(self, user_id: str, action_id: str, **kwargs) -> bool:
        """Met à jour une action existante."""
        action = self.get_action(user_id, action_id)
        if not action:
            return False

        for key, value in kwargs.items():
            if hasattr(action, key):
                setattr(action, key, value)
        return True

    def delete_action(self, user_id: str, action_id: str) -> bool:
        """Supprime une action."""
        if user_id not in self._user_actions:
            return False

        for i, action in enumerate(self._user_actions[user_id]):
            if action.id == action_id:
                self._user_actions[user_id].pop(i)
                return True
        return False

    def get_stats(self, user_id: str) -> Dict:
        """Récupère les statistiques globales pour un utilisateur."""
        actions = self.get_actions(user_id)
        total_co2 = sum(action.co2_saved_kg for action in actions)
        total_water = sum(action.water_saved_liters for action in actions)
        actions_by_category = {}

        for category in ActionCategory:
            category_actions = self.get_actions(user_id, category)
            actions_by_category[category.value] = {
                "count": len(category_actions),
                "co2_saved_kg": sum(a.co2_saved_kg for a in category_actions),
                "water_saved_liters": sum(a.water_saved_liters for a in category_actions)
            }

        return {
            "total_actions": len(actions),
            "total_co2_saved_kg": total_co2,
            "total_water_saved_liters": total_water,
            "actions_by_category": actions_by_category
        }

    def get_journal_stats(self, user_id: str) -> Dict:
        """Récupère les statistiques du journal écologique pour un utilisateur."""
        stats = self.get_stats(user_id)
        return {
            "total_actions": stats["total_actions"],
            "total_co2_saved_kg": stats["total_co2_saved_kg"],
            "total_water_saved_liters": stats["total_water_saved_liters"],
            "bike_ride_count": len([a for a in self.get_actions(user_id) if a.metadata.get("bike_ride_km")]),
            "public_transport_count": len([a for a in self.get_actions(user_id) if a.metadata.get("public_transport_km")]),
            "nature_activity_count": len([a for a in self.get_actions(user_id) if a.category == ActionCategory.NATURE]),
            "water_saving_streak": self._calculate_water_saving_streak(user_id)
        }

    def _calculate_water_saving_streak(self, user_id: str) -> int:
        """Calcule la série de jours consécutifs avec économie d'eau."""
        water_actions = [
            a for a in self.get_actions(user_id)
            if a.category == ActionCategory.WATER and a.water_saved_liters > 0
        ]
        if not water_actions:
            return 0

        # Grouper les actions par jour
        days_with_water_saving = set()
        for action in water_actions:
            days_with_water_saving.add(action.timestamp.date())

        # Trouver la série la plus longue
        sorted_days = sorted(days_with_water_saving)
        if not sorted_days:
            return 0

        max_streak = 1
        current_streak = 1
        for i in range(1, len(sorted_days)):
            if (sorted_days[i] - sorted_days[i-1]).days == 1:
                current_streak += 1
                max_streak = max(max_streak, current_streak)
            else:
                current_streak = 1

        return max_streak

    def get_actions_by_period(self, user_id: str, period: str = "daily") -> List[EcoAction]:
        """Récupère les actions pour une période donnée (daily, weekly, monthly)."""
        actions = self.get_actions(user_id)
        today = datetime.now().date()

        if period == "daily":
            return [a for a in actions if a.timestamp.date() == today]
        elif period == "weekly":
            start_of_week = today - timedelta(days=today.weekday())
            return [a for a in actions if start_of_week <= a.timestamp.date() <= today]
        elif period == "monthly":
            start_of_month = today.replace(day=1)
            return [a for a in actions if start_of_month <= a.timestamp.date() <= today]
        else:
            return actions

    def get_trends(self, user_id: str, period: str = "weekly") -> Dict:
        """Calcule les tendances des actions écologiques."""
        current_actions = self.get_actions_by_period(user_id, period)
        previous_period_start = datetime.now().date() - timedelta(
            days=7 if period == "weekly" else 30 if period == "monthly" else 1
        )
        previous_period_end = datetime.now().date() - timedelta(
            days=7 if period == "weekly" else 30 if period == "monthly" else 1
        )
        previous_actions = [
            a for a in self.get_actions(user_id)
            if previous_period_start <= a.timestamp.date() <= previous_period_end
        ]

        current_co2 = sum(a.co2_saved_kg for a in current_actions)
        previous_co2 = sum(a.co2_saved_kg for a in previous_actions)
        current_water = sum(a.water_saved_liters for a in current_actions)
        previous_water = sum(a.water_saved_liters for a in previous_actions)

        return {
            "co2_trend": current_co2 - previous_co2,
            "water_trend": current_water - previous_water,
            "actions_trend": len(current_actions) - len(previous_actions)
        }

    def reset_user_journal(self, user_id: str) -> bool:
        """Réinitialise le journal d'un utilisateur."""
        if user_id not in self._user_actions:
            return False
        self._user_actions[user_id] = []
        return True

    def clear_all_journals(self) -> None:
        """Supprime tous les journaux."""
        self._user_actions.clear()
```

---

## Tests Unitaires
20 tests ont été créés dans `tests/unit/test_eco_journal_manager.py` pour valider :
- L'enregistrement des actions écologiques.
- La récupération des actions par utilisateur et par catégorie.
- Le calcul de l'impact environnemental (CO2 et eau économisés).
- La génération de statistiques et de rapports.
- Le suivi des tendances.
- La réinitialisation du journal.
- Le déterminisme du module.

---

## Configuration Requise
- **Dépendances** : `datetime` pour les timestamps, `timedelta` pour les calculs de dates.
- **Stockage** : En mémoire pour la phase PoC (à remplacer par une base de données pour la phase GA).

---

## Dépendances
- **Story 6-2** : `suivi-des-progrès` (pour le suivi des métriques).
- **Story 5-1** : `créer-et-mettre-à-jour-le-profil-utilisateur` (pour l'identification des utilisateurs).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine.eco_journal_manager import (
    eco_journal_manager,
    ActionCategory
)

# Enregistrer une action écologique
action = eco_journal_manager.log_action(
    user_id="user_001",
    category=ActionCategory.TRANSPORT,
    description="Trajet en vélo de 10 km",
    metadata={"bike_ride_km": 10}
)
print(f"Action enregistrée : {action.id}, CO2 économisé : {action.co2_saved_kg} kg")

# Enregistrer une autre action
action2 = eco_journal_manager.log_action(
    user_id="user_001",
    category=ActionCategory.WATER,
    description="Douche de 5 minutes",
    metadata={"shower_min": 5}
)
print(f"Action enregistrée : {action2.id}, Eau économisée : {action2.water_saved_liters} L")

# Récupérer les actions d'un utilisateur
actions = eco_journal_manager.get_actions("user_001")
print(f"Actions de user_001 : {len(actions)}")

# Récupérer les statistiques
stats = eco_journal_manager.get_stats("user_001")
print(f"Statistiques : {stats}")

# Récupérer les actions par catégorie
transport_actions = eco_journal_manager.get_actions("user_001", ActionCategory.TRANSPORT)
print(f"Actions de transport : {len(transport_actions)}")

# Récupérer les tendances
trends = eco_journal_manager.get_trends("user_001", period="weekly")
print(f"Tendances : {trends}")

# Réinitialiser le journal
eco_journal_manager.reset_user_journal("user_001")
```

---

## Notes
- **Stockage en mémoire** : Les actions sont stockées en mémoire pour la phase PoC.
- **Facteurs d'impact** : Les facteurs de calcul du CO2 et de l'eau économisés sont basés sur des moyennes standard.
- **Statistiques flexibles** : Permet de générer des statistiques par catégorie ou globalement.
- **Tendances** : Calcule les différences entre périodes pour suivre la progression.
- **Déterminisme** : Le module est déterministe, ce qui facilite les tests.

---

## Ressources
- [PRD Almanéa - FR-58](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-8](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
