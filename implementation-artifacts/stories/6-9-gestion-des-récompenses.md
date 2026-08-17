---
name: Gestion des récompenses
id: 6-9-gestion-des-récompenses
story_type: backend
epic: epic-6
priority: high
estimation: M
dependencies: [6-8-personnalisation-des-règles-de-points, 6-6-récompenses-personnalisables-par-les-collectivités]
status: backlog
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 6.9: Gestion des récompenses

## Contexte
Cette story fait partie de **l'Épic 6 (Gamification)**. Son objectif est de **fournir une interface complète pour gérer les récompenses** dans l'application Almanéa, incluant la création, la modification, la suppression et le suivi des récompenses. Le `RewardManagementSystem` permet de :
- **Créer des récompenses** avec des critères spécifiques.
- **Modifier des récompenses** existantes.
- **Supprimer des récompenses**.
- **Activer/désactiver des récompenses**.
- **Suivre l'utilisation des récompenses** (nombre de fois attribuées, utilisateurs concernés).
- **Gérer les catégories de récompenses**.

---

## Exigences Fonctionnelles
- **FR-85**: Créer des récompenses avec des critères spécifiques.
- **FR-86**: Modifier des récompenses existantes.
- **FR-87**: Supprimer des récompenses.
- **FR-88**: Activer ou désactiver des récompenses.
- **FR-89**: Suivre l'utilisation des récompenses.

---

## Critères d'Acceptation
1. **Création de récompenses** :
   - [ ] Créer une récompense avec un nom, une description, un type, des points requis et des critères optionnels (badge, période de validité).
   - [ ] Vérifier que la récompense est disponible pour les utilisateurs.

2. **Modification de récompenses** :
   - [ ] Modifier les attributs d'une récompense existante.
   - [ ] Vérifier que les modifications sont appliquées immédiatement.

3. **Suppression de récompenses** :
   - [ ] Supprimer une récompense existante.
   - [ ] Vérifier que la récompense supprimée n'est plus disponible.

4. **Activation/désactivation** :
   - [ ] Activer une récompense désactivée.
   - [ ] Désactiver une récompense active.
   - [ ] Vérifier que les récompenses désactivées ne sont pas disponibles.

5. **Suivi de l'utilisation** :
   - [ ] Récupérer le nombre de fois qu'une récompense a été attribuée.
   - [ ] Récupérer la liste des utilisateurs ayant reçu une récompense.
   - [ ] Récupérer les statistiques d'utilisation (récompenses les plus populaires, etc.).

6. **Gestion des catégories** :
   - [ ] Ajouter une nouvelle catégorie de récompenses.
   - [ ] Lister les récompenses par catégorie.

7. **Intégration avec le RecommendationEngine** :
   - [ ] Le `RecommendationEngine` utilise le `RewardManagementSystem` pour gérer les récompenses.

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/reward_management_system.py` | Module `RewardManagementSystem` | ⏳ À créer |
| `backend/engines/recommendation_engine/__init__.py` | Intégration du `RewardManagementSystem` | ⏳ À modifier |
| `tests/unit/test_reward_management_system.py` | Tests unitaires | ⏳ À créer |

### Module `RewardManagementSystem`
Ce module gère la création, la modification et la suppression des récompenses :

```python
from typing import Dict, List, Optional
from dataclasses import dataclass, field
from datetime import datetime
from enum import Enum

class RewardStatus(Enum):
    ACTIVE = "active"
    INACTIVE = "inactive"
    ARCHIVED = "archived"

class RewardCategory(Enum):
    DISCOUNT = "discount"
    GIFT = "gift"
    EVENT = "event"
    SERVICE = "service"
    PRODUCT = "product"
    CUSTOM = "custom"

@dataclass
class Reward:
    id: str
    name: str
    description: str
    category: RewardCategory
    points_required: int
    status: RewardStatus = RewardStatus.ACTIVE
    badge_id: Optional[str] = None
    start_date: Optional[datetime] = None
    end_date: Optional[datetime] = None
    max_uses: Optional[int] = None
    current_uses: int = 0
    created_at: datetime = field(default_factory=datetime.now)
    updated_at: datetime = field(default_factory=datetime.now)

class RewardManagementSystem:
    def __init__(self):
        # Récompenses : {reward_id: Reward}
        self._rewards: Dict[str, Reward] = {}
        # Utilisations par récompense : {reward_id: [user_id]}
        self._reward_uses: Dict[str, List[str]] = {}

    def create_reward(self, reward_data: Dict) -> Optional[Reward]:
        """Crée une nouvelle récompense."""
        reward_id = reward_data.get("id", f"reward_{datetime.now().timestamp()}")
        reward = Reward(
            id=reward_id,
            name=reward_data["name"],
            description=reward_data.get("description", ""),
            category=RewardCategory(reward_data["category"]),
            points_required=reward_data["points_required"],
            status=RewardStatus(reward_data.get("status", "active")),
            badge_id=reward_data.get("badge_id"),
            start_date=reward_data.get("start_date"),
            end_date=reward_data.get("end_date"),
            max_uses=reward_data.get("max_uses")
        )
        self._rewards[reward_id] = reward
        return reward

    def get_reward(self, reward_id: str) -> Optional[Reward]:
        """Récupère une récompense par son ID."""
        return self._rewards.get(reward_id)

    def get_all_rewards(self, status: Optional[RewardStatus] = None) -> List[Reward]:
        """Récupère toutes les récompenses, éventuellement filtrées par statut."""
        rewards = list(self._rewards.values())
        if status:
            rewards = [r for r in rewards if r.status == status]
        return rewards

    def update_reward(self, reward_id: str, **kwargs) -> bool:
        """Met à jour une récompense existante."""
        if reward_id not in self._rewards:
            return False
        reward = self._rewards[reward_id]
        for key, value in kwargs.items():
            if hasattr(reward, key):
                setattr(reward, key, value)
        reward.updated_at = datetime.now()
        return True

    def delete_reward(self, reward_id: str) -> bool:
        """Supprime une récompense."""
        if reward_id not in self._rewards:
            return False
        del self._rewards[reward_id]
        if reward_id in self._reward_uses:
            del self._reward_uses[reward_id]
        return True

    def activate_reward(self, reward_id: str) -> bool:
        """Active une récompense."""
        return self.update_reward(reward_id, status=RewardStatus.ACTIVE)

    def deactivate_reward(self, reward_id: str) -> bool:
        """Désactive une récompense."""
        return self.update_reward(reward_id, status=RewardStatus.INACTIVE)

    def archive_reward(self, reward_id: str) -> bool:
        """Archive une récompense."""
        return self.update_reward(reward_id, status=RewardStatus.ARCHIVED)

    def record_reward_use(self, reward_id: str, user_id: str) -> bool:
        """Enregistre l'utilisation d'une récompense par un utilisateur."""
        if reward_id not in self._rewards:
            return False
        
        reward = self._rewards[reward_id]
        
        # Vérifier si la récompense a atteint sa limite d'utilisation
        if reward.max_uses and reward.current_uses >= reward.max_uses:
            return False
        
        # Enregistrer l'utilisation
        if reward_id not in self._reward_uses:
            self._reward_uses[reward_id] = []
        self._reward_uses[reward_id].append(user_id)
        
        # Mettre à jour le compteur
        reward.current_uses += 1
        return True

    def get_reward_uses(self, reward_id: str) -> List[str]:
        """Récupère la liste des utilisateurs ayant utilisé une récompense."""
        return self._reward_uses.get(reward_id, [])

    def get_reward_usage_count(self, reward_id: str) -> int:
        """Récupère le nombre d'utilisations d'une récompense."""
        return len(self._reward_uses.get(reward_id, []))

    def get_reward_stats(self, reward_id: str) -> Dict:
        """Récupère les statistiques d'utilisation d'une récompense."""
        reward = self.get_reward(reward_id)
        if not reward:
            return {}
        
        return {
            "reward_id": reward.id,
            "name": reward.name,
            "points_required": reward.points_required,
            "current_uses": reward.current_uses,
            "max_uses": reward.max_uses,
            "status": reward.status.value,
            "users": self.get_reward_uses(reward_id)
        }

    def get_all_reward_stats(self) -> List[Dict]:
        """Récupère les statistiques d'utilisation de toutes les récompenses."""
        return [self.get_reward_stats(reward_id) for reward_id in self._rewards]

    def get_rewards_by_category(self, category: RewardCategory) -> List[Reward]:
        """Récupère les récompenses par catégorie."""
        return [r for r in self._rewards.values() if r.category == category]

    def get_popular_rewards(self, limit: int = 5) -> List[Dict]:
        """Récupère les récompenses les plus populaires."""
        stats = self.get_all_reward_stats()
        sorted_stats = sorted(stats, key=lambda x: x["current_uses"], reverse=True)
        return sorted_stats[:limit]

    def get_recent_rewards(self, limit: int = 5) -> List[Reward]:
        """Récupère les récompenses récemment créées."""
        rewards = sorted(
            self._rewards.values(),
            key=lambda x: x.created_at,
            reverse=True
        )
        return rewards[:limit]

    def search_rewards(self, query: str) -> List[Reward]:
        """Recherche des récompenses par nom ou description."""
        query = query.lower()
        return [
            r for r in self._rewards.values()
            if query in r.name.lower() or query in r.description.lower()
        ]

    def clear_all_rewards(self) -> None:
        """Supprime toutes les récompenses."""
        self._rewards.clear()
        self._reward_uses.clear()
```

---

## Tests Unitaires
Les tests suivants seront créés dans `tests/unit/test_reward_management_system.py` :
- Création, modification et suppression de récompenses.
- Activation/désactivation de récompenses.
- Enregistrement et suivi de l'utilisation des récompenses.
- Récupération des statistiques d'utilisation.
- Gestion des catégories de récompenses.
- Recherche de récompenses.

---

## Configuration Requise
- **Dépendances** : `datetime` pour les timestamps, `dataclasses` pour les structures de données.
- **Stockage** : En mémoire pour la phase PoC (à remplacer par une base de données pour la phase GA).

---

## Dépendances
- **Story 6-8** : `personnalisation-des-règles-de-points` (pour adapter les points requis).
- **Story 6-6** : `récompenses-personnalisables-par-les-collectivités` (pour la base des récompenses).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine.reward_management_system import (
    reward_management_system,
    RewardStatus,
    RewardCategory
)

# Créer une récompense
reward_data = {
    "name": "Réduction de 20%",
    "description": "Réduction de 20% dans les magasins partenaires",
    "category": "discount",
    "points_required": 200,
    "badge_id": "bike_star",
    "max_uses": 100
}
reward = reward_management_system.create_reward(reward_data)
print(f"Récompense créée : {reward.id}")

# Récupérer une récompense
retrieved_reward = reward_management_system.get_reward(reward.id)
print(f"Récompense récupérée : {retrieved_reward.name}")

# Mettre à jour une récompense
reward_management_system.update_reward(reward.id, points_required=250)

# Désactiver une récompense
reward_management_system.deactivate_reward(reward.id)

# Enregistrer l'utilisation d'une récompense
reward_management_system.record_reward_use(reward.id, "user_001")

# Récupérer les statistiques d'une récompense
stats = reward_management_system.get_reward_stats(reward.id)
print(f"Statistiques : {stats}")

# Récupérer les récompenses par catégorie
discount_rewards = reward_management_system.get_rewards_by_category(RewardCategory.DISCOUNT)
print(f"Récompenses de type discount : {len(discount_rewards)}")

# Récupérer les récompenses populaires
popular_rewards = reward_management_system.get_popular_rewards(limit=3)
print(f"Récompenses populaires : {popular_rewards}")

# Supprimer une récompense
reward_management_system.delete_reward(reward.id)
```

---

## Notes
- **Stockage en mémoire** : Les récompenses sont stockées en mémoire pour la phase PoC.
- **Statuts** : Les récompenses peuvent être `ACTIVE`, `INACTIVE` ou `ARCHIVED`.
- **Suivi d'utilisation** : Chaque utilisation d'une récompense est enregistrée pour un suivi complet.
- **Statistiques** : Permet de récupérer des statistiques sur l'utilisation des récompenses.
- **Recherche** : Permet de rechercher des récompenses par nom ou description.

---

## Ressources
- [PRD Almanéa - FR-85](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-8](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
