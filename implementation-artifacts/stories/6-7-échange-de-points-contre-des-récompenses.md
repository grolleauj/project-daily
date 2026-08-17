---
name: Échange de points contre des récompenses
id: 6-7-échange-de-points-contre-des-récompenses
story_type: backend
epic: epic-6
priority: high
estimation: M
dependencies: [6-6-récompenses-personnalisables-par-les-collectivités, 6-1-système-de-points]
status: backlog
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 6.7: Échange de points contre des récompenses

## Contexte
Cette story fait partie de **l'Épic 6 (Gamification)**. Son objectif est de **permettre aux utilisateurs d'échanger leurs points contre des récompenses** définies par les collectivités. Le `RewardRedemptionSystem` permet de :
- **Échanger des points contre des récompenses** disponibles.
- **Vérifier que l'utilisateur a suffisamment de points** pour une récompense.
- **Mettre à jour le solde de points** après un échange.
- **Suivre les échanges effectués** par les utilisateurs.
- **Gérer les remboursements** en cas d'erreur.

---

## Exigences Fonctionnelles
- **FR-75**: Permettre aux utilisateurs d'échanger des points contre des récompenses.
- **FR-76**: Vérifier que l'utilisateur a suffisamment de points pour une récompense.
- **FR-77**: Mettre à jour le solde de points après un échange.
- **FR-78**: Suivre les échanges effectués par les utilisateurs.
- **FR-79**: Permettre les remboursements en cas d'erreur.

---

## Critères d'Acceptation
1. **Échange de points** :
   - [ ] Permettre à un utilisateur d'échanger des points contre une récompense.
   - [ ] Vérifier que l'utilisateur a suffisamment de points.
   - [ ] Vérifier que la récompense est disponible.

2. **Mise à jour du solde de points** :
   - [ ] Déduire les points de l'utilisateur après un échange réussi.
   - [ ] Empêcher l'échange si l'utilisateur n'a pas suffisamment de points.

3. **Suivi des échanges** :
   - [ ] Enregistrer chaque échange effectué.
   - [ ] Récupérer l'historique des échanges d'un utilisateur.
   - [ ] Vérifier si un utilisateur a déjà échangé une récompense.

4. **Gestion des remboursements** :
   - [ ] Permettre le remboursement d'un échange.
   - [ ] Restituer les points à l'utilisateur.
   - [ ] Marquer la récompense comme non utilisée.

5. **Intégration avec le RecommendationEngine** :
   - [ ] Le `RecommendationEngine` utilise le `RewardRedemptionSystem` pour gérer les échanges.

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/reward_redemption_system.py` | Module `RewardRedemptionSystem` | ⏳ À créer |
| `backend/engines/recommendation_engine/__init__.py` | Intégration du `RewardRedemptionSystem` | ⏳ À modifier |
| `tests/unit/test_reward_redemption_system.py` | Tests unitaires | ⏳ À créer |

### Enum `RedemptionStatus`
Définition des statuts d'échange :

```python
from enum import Enum

class RedemptionStatus(Enum):
    PENDING = "pending"
    COMPLETED = "completed"
    CANCELLED = "cancelled"
    REFUNDED = "refunded"
```

### Module `RewardRedemptionSystem`
Ce module gère l'échange de points contre des récompenses :

```python
from typing import Dict, List, Optional
from dataclasses import dataclass, field
from datetime import datetime
from enum import Enum

class RedemptionStatus(Enum):
    PENDING = "pending"
    COMPLETED = "completed"
    CANCELLED = "cancelled"
    REFUNDED = "refunded"

@dataclass
class RedemptionRecord:
    """
    Enregistrement d'un échange de points contre une récompense.
    
    Attributes:
        redemption_id: Identifiant unique de l'échange.
        user_id: ID de l'utilisateur.
        reward_id: ID de la récompense.
        community_id: ID de la collectivité.
        points_used: Nombre de points utilisés.
        status: Statut de l'échange.
        created_at: Date de création de l'échange.
        completed_at: Date de complétion (optionnel).
        refunded_at: Date de remboursement (optionnel).
    """
    redemption_id: str
    user_id: str
    reward_id: str
    community_id: str
    points_used: int
    status: RedemptionStatus
    created_at: datetime
    completed_at: Optional[datetime] = None
    refunded_at: Optional[datetime] = None

class RewardRedemptionSystem:
    def __init__(self, points_system, community_reward_system):
        self.points_system = points_system
        self.community_reward_system = community_reward_system
        # Historique des échanges : {user_id: [RedemptionRecord]}
        self._redemption_history: Dict[str, List[RedemptionRecord]] = {}

    def redeem_reward(
        self,
        user_id: str,
        reward_id: str,
        community_id: str
    ) -> Optional[RedemptionRecord]:
        """
        Échange des points contre une récompense.
        
        Args:
            user_id: ID de l'utilisateur.
            reward_id: ID de la récompense.
            community_id: ID de la collectivité.
        
        Returns:
            Optional[RedemptionRecord]: L'enregistrement de l'échange ou None si l'échange a échoué.
        """
        # Récupérer la récompense
        reward = self.community_reward_system.get_reward(reward_id, community_id)
        if not reward or not reward.is_active:
            return None
        
        # Vérifier que l'utilisateur a suffisamment de points
        user_points = self.points_system.get_user_points(user_id)
        if user_points < reward.points_required:
            return None
        
        # Vérifier que la récompense est disponible pour l'utilisateur
        user_badges = [badge.badge_id for badge in self.community_reward_system.get_user_rewards(user_id, community_id)]
        available_rewards = self.community_reward_system.get_available_rewards(
            user_id, community_id, user_points, user_badges
        )
        if reward.id not in [r.id for r in available_rewards]:
            return None
        
        # Créer l'enregistrement de l'échange
        redemption_id = f"{user_id}_{reward_id}_{datetime.now().timestamp()}"
        redemption = RedemptionRecord(
            redemption_id=redemption_id,
            user_id=user_id,
            reward_id=reward_id,
            community_id=community_id,
            points_used=reward.points_required,
            status=RedemptionStatus.COMPLETED,
            created_at=datetime.now(),
            completed_at=datetime.now()
        )
        
        # Déduire les points de l'utilisateur
        self.points_system.deduct_points(user_id, reward.points_required)
        
        # Attribuer la récompense à l'utilisateur
        self.community_reward_system.claim_reward(user_id, reward_id, community_id)
        
        # Enregistrer l'échange
        self._redemption_history.setdefault(user_id, []).append(redemption)
        
        return redemption

    def get_redemption_history(self, user_id: str) -> List[RedemptionRecord]:
        """
        Récupère l'historique des échanges d'un utilisateur.
        
        Args:
            user_id: ID de l'utilisateur.
        
        Returns:
            List[RedemptionRecord]: Historique des échanges.
        """
        return self._redemption_history.get(user_id, [])

    def get_redemption(self, redemption_id: str, user_id: str) -> Optional[RedemptionRecord]:
        """
        Récupère un échange spécifique.
        
        Args:
            redemption_id: ID de l'échange.
            user_id: ID de l'utilisateur.
        
        Returns:
            Optional[RedemptionRecord]: L'enregistrement de l'échange ou None.
        """
        if user_id not in self._redemption_history:
            return None
        for redemption in self._redemption_history[user_id]:
            if redemption.redemption_id == redemption_id:
                return redemption
        return None

    def refund_redemption(self, user_id: str, redemption_id: str) -> bool:
        """
        Rembourse un échange (restitue les points et annule l'attribution de la récompense).
        
        Args:
            user_id: ID de l'utilisateur.
            redemption_id: ID de l'échange.
        
        Returns:
            bool: True si le remboursement a réussi.
        """
        redemption = self.get_redemption(redemption_id, user_id)
        if not redemption or redemption.status != RedemptionStatus.COMPLETED:
            return False
        
        # Restituer les points à l'utilisateur
        self.points_system.add_points(user_id, redemption.points_used, "refund")
        
        # Marquer la récompense comme non utilisée (si elle a été marquée comme utilisée)
        self.community_reward_system.mark_reward_as_used(user_id, redemption.reward_id)
        
        # Mettre à jour le statut de l'échange
        redemption.status = RedemptionStatus.REFUNDED
        redemption.refunded_at = datetime.now()
        
        return True

    def get_user_redemptions(self, user_id: str, community_id: Optional[str] = None) -> List[Dict]:
        """
        Récupère les échanges d'un utilisateur avec leurs détails.
        
        Args:
            user_id: ID de l'utilisateur.
            community_id: ID de la collectivité (optionnel).
        
        Returns:
            List[Dict]: Liste des échanges avec leurs détails.
        """
        redemptions = self.get_redemption_history(user_id)
        if community_id:
            redemptions = [r for r in redemptions if r.community_id == community_id]
        
        return [
            {
                "redemption_id": r.redemption_id,
                "reward_id": r.reward_id,
                "points_used": r.points_used,
                "status": r.status.value,
                "created_at": r.created_at.isoformat(),
                "completed_at": r.completed_at.isoformat() if r.completed_at else None,
                "refunded_at": r.refunded_at.isoformat() if r.refunded_at else None
            }
            for r in redemptions
        ]

    def get_total_points_redeemed(self, user_id: str) -> int:
        """
        Récupère le total des points échangés par un utilisateur.
        
        Args:
            user_id: ID de l'utilisateur.
        
        Returns:
            int: Total des points échangés.
        """
        redemptions = self.get_redemption_history(user_id)
        return sum(r.points_used for r in redemptions if r.status == RedemptionStatus.COMPLETED)

    def clear_user_redemptions(self, user_id: str) -> bool:
        """
        Supprime l'historique des échanges d'un utilisateur.
        
        Args:
            user_id: ID de l'utilisateur.
        
        Returns:
            bool: True si l'historique a été supprimé.
        """
        if user_id not in self._redemption_history:
            return False
        self._redemption_history[user_id] = []
        return True

    def clear_all_redemptions(self) -> None:
        """Supprime tous les échanges."""
        self._redemption_history.clear()
```

---

## Tests Unitaires
Les tests suivants seront créés dans `tests/unit/test_reward_redemption_system.py` :
- Échange de points contre une récompense.
- Vérification des points suffisants.
- Mise à jour du solde de points après échange.
- Suivi des échanges effectués.
- Remboursement d'un échange.
- Récupération de l'historique des échanges.
- Réinitialisation des données.

---

## Configuration Requise
- **Dépendances** : `datetime` pour les timestamps, `dataclasses` pour les structures de données.
- **Intégration** : Dépend du `PointsSystem` et du `CommunityRewardSystem`.
- **Stockage** : En mémoire pour la phase PoC (à remplacer par une base de données pour la phase GA).

---

## Dépendances
- **Story 6-6** : `récompenses-personnalisables-par-les-collectivités` (pour accéder aux récompenses).
- **Story 6-1** : `système-de-points` (pour gérer les points des utilisateurs).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine.reward_redemption_system import (
    reward_redemption_system,
    RedemptionStatus
)

# Initialiser le système avec les dépendances
# reward_redemption_system = RewardRedemptionSystem(points_system, community_reward_system)

# Échanger des points contre une récompense
redemption = reward_redemption_system.redeem_reward(
    user_id="user_001",
    reward_id="reward_001",
    community_id="community_001"
)
print(f"Échange réussi : {redemption is not None}")

# Récupérer l'historique des échanges
redemptions = reward_redemption_system.get_redemption_history("user_001")
print(f"Nombre d'échanges : {len(redemptions)}")

# Récupérer les échanges avec détails
user_redemptions = reward_redemption_system.get_user_redemptions("user_001")
print(f"Détails des échanges : {user_redemptions}")

# Rembourser un échange
result = reward_redemption_system.refund_redemption("user_001", redemption.redemption_id)
print(f"Remboursement réussi : {result}")

# Récupérer le total des points échangés
total_redeemed = reward_redemption_system.get_total_points_redeemed("user_001")
print(f"Total des points échangés : {total_redeemed}")
```

---

## Notes
- **Stockage en mémoire** : Les échanges sont stockés en mémoire pour la phase PoC.
- **Intégration avec PointsSystem** : Déduit automatiquement les points lors d'un échange.
- **Intégration avec CommunityRewardSystem** : Attribue automatiquement la récompense à l'utilisateur.
- **Remboursement** : Permet de restituer les points en cas d'erreur ou d'annulation.
- **Suivi** : Enregistre tous les échanges pour un suivi complet.

---

## Ressources
- [PRD Almanéa - FR-75](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-8](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
