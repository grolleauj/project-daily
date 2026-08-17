---
name: Débloquer des badges
id: 6-4-débloquer-des-badges
story_type: backend
epic: epic-6
priority: high
estimation: M
dependencies: [6-3-journal-écologique, 6-2-suivi-des-progrès, 6-1-système-de-points]
status: done
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 6.4: Débloquer des badges

## Contexte
Cette story fait partie de **l'Épic 6 (Gamification)**. Son objectif est de **permettre aux utilisateurs de débloquer des badges** en fonction de leurs actions écologiques et de leur progression. Le `BadgeSystem` permet de :
- **Définir des badges** avec des conditions de déblocage.
- **Suivre les badges débloqués** par chaque utilisateur.
- **Notifier les utilisateurs** lorsqu'ils débloquent un nouveau badge.
- **Fournir des informations** sur les prochains badges à débloquer.

---

## Exigences Fonctionnelles
- **FR-62**: Définir des badges avec des conditions de déblocage.
- **FR-63**: Suivre les badges débloqués par chaque utilisateur.
- **FR-64**: Notifier les utilisateurs lorsqu'ils débloquent un badge.
- **FR-65**: Fournir des informations sur les prochains badges à débloquer.

---

## Critères d'Acceptation
1. **Définition des badges** :
   - [x] Définir des badges avec un identifiant unique, un nom, une description, une rareté et une catégorie.
   - [x] Définir des conditions de déblocage pour chaque badge (ex: nombre d'actions, points accumulés).

2. **Suivi des badges** :
   - [x] Vérifier si un badge est débloqué pour un utilisateur.
   - [x] Récupérer tous les badges débloqués par un utilisateur.
   - [x] Récupérer les badges par rareté ou catégorie.

3. **Déblocage des badges** :
   - [x] Mettre à jour la progression d'un utilisateur et débloquer les badges correspondants.
   - [x] Empêcher le déblocage multiple du même badge.

4. **Notifications** :
   - [x] Notifier les utilisateurs lorsqu'ils débloquent un nouveau badge.

5. **Prochains badges** :
   - [x] Récupérer les prochains badges que l'utilisateur peut débloquer.
   - [x] Afficher le progrès vers chaque badge.

6. **Gestion des badges** :
   - [x] Réinitialiser les badges d'un utilisateur.
   - [x] Supprimer tous les badges.

7. **Intégration avec le RecommendationEngine** :
   - [x] Le `RecommendationEngine` utilise le `BadgeSystem` pour gérer les badges.

---

## Implémentation Technique

### Fichiers créés/modifiés
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/badge_system.py` | Module `BadgeSystem` | ✅ Créé |
| `backend/engines/recommendation_engine/__init__.py` | Intégration du `BadgeSystem` | ✅ Modifié |
| `tests/unit/test_badge_system.py` | Tests unitaires | ✅ Créé |

### Enum `BadgeRarity` et `BadgeCategory`
Définition des raretés et catégories de badges :

```python
from enum import Enum

class BadgeRarity(Enum):
    COMMON = "common"
    RARE = "rare"
    EPIC = "epic"
    LEGENDARY = "legendary"

class BadgeCategory(Enum):
    TRANSPORT = "transport"
    ENERGY = "energy"
    WATER = "water"
    WASTE = "waste"
    NATURE = "nature"
    GENERAL = "general"
```

### Module `BadgeSystem`
Ce module gère les badges et leur déblocage pour chaque utilisateur :

```python
from typing import Dict, List, Optional, Callable
from datetime import datetime
from dataclasses import dataclass, field
from enum import Enum

class BadgeRarity(Enum):
    COMMON = "common"
    RARE = "rare"
    EPIC = "epic"
    LEGENDARY = "legendary"

class BadgeCategory(Enum):
    TRANSPORT = "transport"
    ENERGY = "energy"
    WATER = "water"
    WASTE = "waste"
    NATURE = "nature"
    GENERAL = "general"

@dataclass
class Badge:
    id: str
    name: str
    description: str
    rarity: BadgeRarity
    category: BadgeCategory
    icon: str = ""
    unlock_condition: Dict = field(default_factory=dict)
    points_reward: int = 0

@dataclass
class UserBadge:
    badge_id: str
    user_id: str
    unlocked_at: datetime
    progress: Optional[Dict] = None

class BadgeSystem:
    def __init__(self):
        self._badges: Dict[str, Badge] = self._initialize_badges()
        self._user_badges: Dict[str, List[UserBadge]] = {}
        self._notification_callbacks: List[Callable] = []

    def _initialize_badges(self) -> Dict[str, Badge]:
        """Initialise les badges disponibles."""
        return {
            "first_step": Badge(
                id="first_step",
                name="Premier Pas",
                description="Effectuez votre première action écologique.",
                rarity=BadgeRarity.COMMON,
                category=BadgeCategory.GENERAL,
                icon="🌱",
                unlock_condition={"total_actions": 1},
                points_reward=10
            ),
            "bike_star": Badge(
                id="bike_star",
                name="Vélo Star",
                description="Effectuez 10 trajets en vélo.",
                rarity=BadgeRarity.COMMON,
                category=BadgeCategory.TRANSPORT,
                icon="🚴",
                unlock_condition={"bike_ride_count": 10},
                points_reward=50
            ),
            "eco_hero": Badge(
                id="eco_hero",
                name="Héros Éco",
                description="Économisez 100 kg de CO2.",
                rarity=BadgeRarity.RARE,
                category=BadgeCategory.GENERAL,
                icon="🌍",
                unlock_condition={"total_co2_saved_kg": 100},
                points_reward=100
            ),
            "nature_explorer": Badge(
                id="nature_explorer",
                name="Explorateur Nature",
                description="Participez à 5 activités en pleine nature.",
                rarity=BadgeRarity.RARE,
                category=BadgeCategory.NATURE,
                icon="🌳",
                unlock_condition={"nature_activity_count": 5},
                points_reward=75
            ),
            "public_transport_champion": Badge(
                id="public_transport_champion",
                name="Champion des Transports en Commun",
                description="Utilisez les transports en commun 20 fois.",
                rarity=BadgeRarity.RARE,
                category=BadgeCategory.TRANSPORT,
                icon="🚆",
                unlock_condition={"public_transport_count": 20},
                points_reward=75
            ),
            "water_saver": Badge(
                id="water_saver",
                name="Économiseur d'Eau",
                description="Économisez de l'eau pendant 30 jours consécutifs.",
                rarity=BadgeRarity.EPIC,
                category=BadgeCategory.WATER,
                icon="💧",
                unlock_condition={"water_saving_streak": 30},
                points_reward=150
            ),
            "marathon_runner": Badge(
                id="marathon_runner",
                name="Marathonien",
                description="Parcourez 100 km à pied ou à vélo.",
                rarity=BadgeRarity.EPIC,
                category=BadgeCategory.TRANSPORT,
                icon="🏃",
                unlock_condition={"total_distance_km": 100},
                points_reward=200
            ),
            "legendary_eco": Badge(
                id="legendary_eco",
                name="Légende Éco",
                description="Débloquez tous les badges.",
                rarity=BadgeRarity.LEGENDARY,
                category=BadgeCategory.GENERAL,
                icon="🏆",
                unlock_condition={"total_badges": 6},
                points_reward=500
            )
        }

    def get_badge(self, badge_id: str) -> Optional[Badge]:
        """Récupère un badge par son ID."""
        return self._badges.get(badge_id)

    def get_all_badges(self) -> List[Badge]:
        """Récupère tous les badges."""
        return list(self._badges.values())

    def get_badges_by_category(self, category: BadgeCategory) -> List[Badge]:
        """Récupère les badges par catégorie."""
        return [badge for badge in self._badges.values() if badge.category == category]

    def get_badges_by_rarity(self, rarity: BadgeRarity) -> List[Badge]:
        """Récupère les badges par rareté."""
        return [badge for badge in self._badges.values() if badge.rarity == rarity]

    def check_unlock_condition(self, user_id: str, badge: Badge, user_stats: Dict) -> bool:
        """Vérifie si les conditions de déblocage d'un badge sont remplies."""
        for key, required_value in badge.unlock_condition.items():
            if key == "total_badges":
                user_badges_count = len(self.get_user_badges(user_id))
                if user_badges_count < required_value:
                    return False
            elif key not in user_stats or user_stats[key] < required_value:
                return False
        return True

    def update_progress(self, user_id: str, user_stats: Dict) -> List[UserBadge]:
        """Met à jour la progression et débloque les badges correspondants."""
        unlocked_badges = []
        for badge in self._badges.values():
            if self.is_badge_unlocked(user_id, badge.id):
                continue
            if self.check_unlock_condition(user_id, badge, user_stats):
                user_badge = UserBadge(
                    badge_id=badge.id,
                    user_id=user_id,
                    unlocked_at=datetime.now()
                )
                self._user_badges.setdefault(user_id, []).append(user_badge)
                unlocked_badges.append(user_badge)
                self._notify_badge_unlocked(user_id, badge)
        return unlocked_badges

    def is_badge_unlocked(self, user_id: str, badge_id: str) -> bool:
        """Vérifie si un badge est débloqué pour un utilisateur."""
        if user_id not in self._user_badges:
            return False
        return any(badge.badge_id == badge_id for badge in self._user_badges[user_id])

    def get_user_badges(self, user_id: str) -> List[UserBadge]:
        """Récupère tous les badges débloqués par un utilisateur."""
        return self._user_badges.get(user_id, [])

    def get_unlocked_badges_by_rarity(self, user_id: str, rarity: BadgeRarity) -> List[UserBadge]:
        """Récupère les badges débloqués par rareté."""
        user_badges = self.get_user_badges(user_id)
        return [
            badge for badge in user_badges
            if self.get_badge(badge.badge_id).rarity == rarity
        ]

    def get_next_badges(self, user_id: str, user_stats: Dict) -> List[Dict]:
        """Récupère les prochains badges que l'utilisateur peut débloquer."""
        next_badges = []
        for badge in self._badges.values():
            if self.is_badge_unlocked(user_id, badge.id):
                continue
            progress = self._calculate_progress(user_id, badge, user_stats)
            next_badges.append({"badge": badge, "progress": progress})
        return sorted(next_badges, key=lambda x: x["progress"]["percentage"], reverse=True)

    def _calculate_progress(self, user_id: str, badge: Badge, user_stats: Dict) -> Dict:
        """Calcule le progrès vers un badge."""
        for key, required_value in badge.unlock_condition.items():
            if key == "total_badges":
                current_value = len(self.get_user_badges(user_id))
            else:
                current_value = user_stats.get(key, 0)
            percentage = min((current_value / required_value) * 100, 100) if required_value > 0 else 100
            return {
                "current": current_value,
                "required": required_value,
                "percentage": percentage
            }
        return {"current": 0, "required": 0, "percentage": 0}

    def register_notification_callback(self, callback: Callable) -> None:
        """Enregistre un callback pour les notifications de déblocage."""
        self._notification_callbacks.append(callback)

    def _notify_badge_unlocked(self, user_id: str, badge: Badge) -> None:
        """Notifie les callbacks enregistrés qu'un badge a été débloqué."""
        for callback in self._notification_callbacks:
            callback(user_id, badge)

    def clear_user_badges(self, user_id: str) -> bool:
        """Supprime tous les badges d'un utilisateur."""
        if user_id not in self._user_badges:
            return False
        self._user_badges[user_id] = []
        return True

    def clear_all_badges(self) -> None:
        """Supprime tous les badges de tous les utilisateurs."""
        self._user_badges.clear()
```

---

## Tests Unitaires
21 tests ont été créés dans `tests/unit/test_badge_system.py` pour valider :
- La récupération des badges par ID, catégorie ou rareté.
- La vérification des conditions de déblocage.
- La mise à jour de la progression et le déblocage des badges.
- La récupération des badges débloqués par un utilisateur.
- La récupération des prochains badges à débloquer.
- Le calcul du progrès vers un badge.
- Les notifications de déblocage.
- La réinitialisation des badges.
- Le déterminisme du module.

---

## Configuration Requise
- **Dépendances** : `datetime` pour les timestamps, `dataclasses` pour les structures de données.
- **Stockage** : En mémoire pour la phase PoC (à remplacer par une base de données pour la phase GA).

---

## Dépendances
- **Story 6-3** : `journal-écologique` (pour suivre les actions écologiques).
- **Story 6-2** : `suivi-des-progrès` (pour suivre les métriques de progrès).
- **Story 6-1** : `système-de-points` (pour attribuer des points aux badges).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine.badge_system import badge_system, BadgeRarity, BadgeCategory

# Récupérer un badge
badge = badge_system.get_badge("bike_star")
print(f"Badge : {badge.name}, Description : {badge.description}")

# Récupérer tous les badges
badges = badge_system.get_all_badges()
print(f"Nombre total de badges : {len(badges)}")

# Récupérer les badges par catégorie
transport_badges = badge_system.get_badges_by_category(BadgeCategory.TRANSPORT)
print(f"Badges de transport : {len(transport_badges)}")

# Mettre à jour la progression et débloquer des badges
user_stats = {
    "bike_ride_count": 10,
    "total_actions": 1,
    "total_co2_saved_kg": 100
}
unlocked_badges = badge_system.update_progress("user_001", user_stats)
print(f"Badges débloqués : {[badge.badge_id for badge in unlocked_badges]}")

# Vérifier si un badge est débloqué
is_unlocked = badge_system.is_badge_unlocked("user_001", "bike_star")
print(f"Badge 'Vélo Star' débloqué : {is_unlocked}")

# Récupérer les badges d'un utilisateur
user_badges = badge_system.get_user_badges("user_001")
print(f"Badges de l'utilisateur : {[badge.badge_id for badge in user_badges]}")

# Récupérer les prochains badges à débloquer
next_badges = badge_system.get_next_badges("user_001", user_stats)
print(f"Prochains badges : {[item['badge'].name for item in next_badges]}")

# Enregistrer un callback de notification
def notification_callback(user_id: str, badge):
    print(f"Utilisateur {user_id} a débloqué le badge : {badge.name}")

badge_system.register_notification_callback(notification_callback)

# Réinitialiser les badges d'un utilisateur
badge_system.clear_user_badges("user_001")
```

---

## Notes
- **Stockage en mémoire** : Les badges débloqués sont stockés en mémoire pour la phase PoC.
- **Conditions de déblocage** : Les conditions sont basées sur des métriques spécifiques (ex: nombre d'actions, CO2 économisé).
- **Notifications** : Permet d'enregistrer des callbacks pour notifier les utilisateurs lorsqu'ils débloquent un badge.
- **Prochains badges** : Fournit une liste des badges que l'utilisateur peut débloquer ensuite, avec leur progrès.
- **Déterminisme** : Le module est déterministe, ce qui facilite les tests.

---

## Ressources
- [PRD Almanéa - FR-62](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-8](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
