---
name: Récompenses personnalisables par les collectivités
id: 6-6-récompenses-personnalisables-par-les-collectivités
story_type: backend
epic: epic-6
priority: high
estimation: M
dependencies: [6-5-mécanismes-anti-triche, 6-4-débloquer-des-badges, 6-1-système-de-points]
status: backlog
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 6.6: Récompenses personnalisables par les collectivités

## Contexte
Cette story fait partie de **l'Épic 6 (Gamification)**. Son objectif est de **permettre aux collectivités de personnaliser les récompenses** offertes aux utilisateurs en fonction de leurs actions écologiques. Le `CommunityRewardSystem` permet de :
- **Définir des récompenses spécifiques** pour chaque collectivité.
- **Associer des récompenses à des badges ou des seuils de points**.
- **Gérer les récompenses disponibles** (ajout, modification, suppression).
- **Suivre les récompenses attribuées** aux utilisateurs.

---

## Exigences Fonctionnelles
- **FR-71**: Permettre aux collectivités de définir des récompenses personnalisées.
- **FR-72**: Associer des récompenses à des badges ou des seuils de points.
- **FR-73**: Gérer les récompenses disponibles (ajout, modification, suppression).
- **FR-74**: Suivre les récompenses attribuées aux utilisateurs.

---

## Critères d'Acceptation
1. **Définition des récompenses** :
   - [ ] Définir une récompense avec un identifiant unique, un nom, une description, une valeur en points et une collectivité associée.
   - [ ] Définir des récompenses de différents types (ex: bons d'achat, réductions, accès à des événements).

2. **Association des récompenses** :
   - [ ] Associer une récompense à un badge spécifique.
   - [ ] Associer une récompense à un seuil de points.

3. **Gestion des récompenses** :
   - [ ] Ajouter une nouvelle récompense.
   - [ ] Modifier une récompense existante.
   - [ ] Supprimer une récompense.
   - [ ] Lister toutes les récompenses disponibles pour une collectivité.

4. **Suivi des récompenses attribuées** :
   - [ ] Enregistrer l'attribution d'une récompense à un utilisateur.
   - [ ] Récupérer les récompenses attribuées à un utilisateur.
   - [ ] Vérifier si un utilisateur a déjà reçu une récompense.

5. **Intégration avec le RecommendationEngine** :
   - [ ] Le `RecommendationEngine` utilise le `CommunityRewardSystem` pour gérer les récompenses.

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/community_reward_system.py` | Module `CommunityRewardSystem` | ⏳ À créer |
| `backend/engines/recommendation_engine/__init__.py` | Intégration du `CommunityRewardSystem` | ⏳ À modifier |
| `tests/unit/test_community_reward_system.py` | Tests unitaires | ⏳ À créer |

### Enum `RewardType`
Définition des types de récompenses :

```python
from enum import Enum

class RewardType(Enum):
    DISCOUNT = "discount"
    GIFT_CARD = "gift_card"
    EVENT_ACCESS = "event_access"
    PRODUCT = "product"
    SERVICE = "service"
```

### Module `CommunityRewardSystem`
Ce module gère les récompenses personnalisables par les collectivités :

```python
from typing import Dict, List, Optional
from dataclasses import dataclass, field
from datetime import datetime
from enum import Enum

class RewardType(Enum):
    DISCOUNT = "discount"
    GIFT_CARD = "gift_card"
    EVENT_ACCESS = "event_access"
    PRODUCT = "product"
    SERVICE = "service"

@dataclass
class CommunityReward:
    id: str
    name: str
    description: str
    community_id: str
    reward_type: RewardType
    points_required: int
    badge_id: Optional[str] = None
    max_uses: Optional[int] = None
    start_date: Optional[datetime] = None
    end_date: Optional[datetime] = None
    is_active: bool = True

@dataclass
class UserReward:
    reward_id: str
    user_id: str
    community_id: str
    claimed_at: datetime
    used: bool = False

class CommunityRewardSystem:
    def __init__(self):
        # Récompenses par collectivité : {community_id: [CommunityReward]}
        self._community_rewards: Dict[str, List[CommunityReward]] = {}
        # Récompenses attribuées aux utilisateurs : {user_id: [UserReward]}
        self._user_rewards: Dict[str, List[UserReward]] = {}

    def add_reward(self, reward: CommunityReward) -> bool:
        """Ajoute une nouvelle récompense pour une collectivité."""
        if reward.community_id not in self._community_rewards:
            self._community_rewards[reward.community_id] = []
        self._community_rewards[reward.community_id].append(reward)
        return True

    def get_reward(self, reward_id: str, community_id: str) -> Optional[CommunityReward]:
        """Récupère une récompense par son ID et l'ID de la collectivité."""
        if community_id not in self._community_rewards:
            return None
        for reward in self._community_rewards[community_id]:
            if reward.id == reward_id:
                return reward
        return None

    def get_rewards_by_community(self, community_id: str) -> List[CommunityReward]:
        """Récupère toutes les récompenses pour une collectivité."""
        return self._community_rewards.get(community_id, [])

    def get_rewards_by_badge(self, badge_id: str, community_id: str) -> List[CommunityReward]:
        """Récupère les récompenses associées à un badge pour une collectivité."""
        rewards = self.get_rewards_by_community(community_id)
        return [reward for reward in rewards if reward.badge_id == badge_id]

    def get_rewards_by_points(self, points: int, community_id: str) -> List[CommunityReward]:
        """Récupère les récompenses accessibles avec un nombre de points donné."""
        rewards = self.get_rewards_by_community(community_id)
        return [reward for reward in rewards if reward.points_required <= points and reward.is_active]

    def update_reward(self, reward_id: str, community_id: str, **kwargs) -> bool:
        """Met à jour une récompense existante."""
        reward = self.get_reward(reward_id, community_id)
        if not reward:
            return False
        for key, value in kwargs.items():
            if hasattr(reward, key):
                setattr(reward, key, value)
        return True

    def delete_reward(self, reward_id: str, community_id: str) -> bool:
        """Supprime une récompense."""
        if community_id not in self._community_rewards:
            return False
        for i, reward in enumerate(self._community_rewards[community_id]):
            if reward.id == reward_id:
                self._community_rewards[community_id].pop(i)
                return True
        return False

    def claim_reward(self, user_id: str, reward_id: str, community_id: str) -> Optional[UserReward]:
        """Attribue une récompense à un utilisateur."""
        reward = self.get_reward(reward_id, community_id)
        if not reward or not reward.is_active:
            return None

        # Vérifier si l'utilisateur a déjà reçu cette récompense
        if self.has_user_claimed_reward(user_id, reward_id):
            return None

        # Vérifier si la récompense a une limite d'utilisation
        if reward.max_uses is not None:
            claimed_count = len([
                ur for ur in self._user_rewards.get(user_id, [])
                if ur.reward_id == reward_id
            ])
            if claimed_count >= reward.max_uses:
                return None

        user_reward = UserReward(
            reward_id=reward_id,
            user_id=user_id,
            community_id=community_id,
            claimed_at=datetime.now()
        )
        self._user_rewards.setdefault(user_id, []).append(user_reward)
        return user_reward

    def has_user_claimed_reward(self, user_id: str, reward_id: str) -> bool:
        """Vérifie si un utilisateur a déjà reçu une récompense."""
        if user_id not in self._user_rewards:
            return False
        return any(ur.reward_id == reward_id for ur in self._user_rewards[user_id])

    def get_user_rewards(self, user_id: str, community_id: Optional[str] = None) -> List[UserReward]:
        """Récupère les récompenses attribuées à un utilisateur."""
        if user_id not in self._user_rewards:
            return []
        if community_id:
            return [ur for ur in self._user_rewards[user_id] if ur.community_id == community_id]
        return self._user_rewards[user_id]

    def mark_reward_as_used(self, user_id: str, reward_id: str) -> bool:
        """Marque une récompense comme utilisée."""
        if user_id not in self._user_rewards:
            return False
        for user_reward in self._user_rewards[user_id]:
            if user_reward.reward_id == reward_id:
                user_reward.used = True
                return True
        return False

    def get_available_rewards(self, user_id: str, community_id: str, user_points: int) -> List[CommunityReward]:
        """Récupère les récompenses disponibles pour un utilisateur."""
        rewards = self.get_rewards_by_points(user_points, community_id)
        claimed_rewards = [ur.reward_id for ur in self.get_user_rewards(user_id, community_id)]
        return [reward for reward in rewards if reward.id not in claimed_rewards]

    def clear_community_rewards(self, community_id: str) -> bool:
        """Supprime toutes les récompenses d'une collectivité."""
        if community_id not in self._community_rewards:
            return False
        self._community_rewards[community_id] = []
        return True

    def clear_all_rewards(self) -> None:
        """Supprime toutes les récompenses et les attributions."""
        self._community_rewards.clear()
        self._user_rewards.clear()
```

---

## Tests Unitaires
Les tests suivants seront créés dans `tests/unit/test_community_reward_system.py` :
- Ajout, modification et suppression de récompenses.
- Récupération des récompenses par collectivité, badge ou seuil de points.
- Attribution de récompenses aux utilisateurs.
- Vérification des récompenses attribuées.
- Gestion des limites d'utilisation.
- Réinitialisation des données.

---

## Configuration Requise
- **Dépendances** : `datetime` pour les timestamps, `dataclasses` pour les structures de données.
- **Stockage** : En mémoire pour la phase PoC (à remplacer par une base de données pour la phase GA).

---

## Dépendances
- **Story 6-5** : `mécanismes-anti-triche` (pour éviter les abus).
- **Story 6-4** : `débloquer-des-badges` (pour associer des récompenses à des badges).
- **Story 6-1** : `système-de-points` (pour associer des récompenses à des seuils de points).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine.community_reward_system import (
    community_reward_system,
    CommunityReward,
    RewardType
)
from datetime import datetime

# Ajouter une récompense pour une collectivité
reward = CommunityReward(
    id="reward_001",
    name="Réduction de 10%",
    description="Réduction de 10% dans les magasins partenaires",
    community_id="community_001",
    reward_type=RewardType.DISCOUNT,
    points_required=100,
    badge_id=None,
    max_uses=1
)
community_reward_system.add_reward(reward)

# Récupérer les récompenses pour une collectivité
rewards = community_reward_system.get_rewards_by_community("community_001")
print(f"Récompenses pour community_001 : {len(rewards)}")

# Attribuer une récompense à un utilisateur
user_reward = community_reward_system.claim_reward(
    user_id="user_001",
    reward_id="reward_001",
    community_id="community_001"
)
print(f"Récompense attribuée : {user_reward is not None}")

# Récupérer les récompenses d'un utilisateur
user_rewards = community_reward_system.get_user_rewards("user_001", "community_001")
print(f"Récompenses de l'utilisateur : {len(user_rewards)}")

# Vérifier si un utilisateur a déjà reçu une récompense
has_claimed = community_reward_system.has_user_claimed_reward("user_001", "reward_001")
print(f"Récompense déjà reçue : {has_claimed}")

# Récupérer les récompenses disponibles pour un utilisateur
available_rewards = community_reward_system.get_available_rewards(
    user_id="user_001",
    community_id="community_001",
    user_points=200
)
print(f"Récompenses disponibles : {len(available_rewards)}")

# Marquer une récompense comme utilisée
community_reward_system.mark_reward_as_used("user_001", "reward_001")

# Supprimer une récompense
community_reward_system.delete_reward("reward_001", "community_001")
```

---

## Notes
- **Stockage en mémoire** : Les récompenses sont stockées en mémoire pour la phase PoC.
- **Personnalisation** : Chaque collectivité peut définir ses propres récompenses.
- **Flexibilité** : Les récompenses peuvent être associées à des badges ou à des seuils de points.
- **Limites d'utilisation** : Les récompenses peuvent avoir une limite d'utilisation par utilisateur.
- **Période de validité** : Les récompenses peuvent avoir une période de validité (date de début et de fin).

---

## Ressources
- [PRD Almanéa - FR-71](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-8](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
