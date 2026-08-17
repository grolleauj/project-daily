---
name: Notifications pour les récompenses
id: 6-10-notifications-pour-les-récompenses
story_type: backend
epic: epic-6
priority: high
estimation: M
dependencies: [6-9-gestion-des-récompenses, 6-7-échange-de-points-contre-des-récompenses]
status: backlog
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 6.10: Notifications pour les récompenses

## Contexte
Cette story fait partie de **l'Épic 6 (Gamification)**. Son objectif est de **notifier les utilisateurs lorsqu'ils débloquent des récompenses ou des badges**, ou lorsqu'ils peuvent échanger des points contre des récompenses. Le `RewardNotificationSystem` permet de :
- **Envoyer des notifications** aux utilisateurs pour les informer des nouvelles récompenses disponibles.
- **Notifier les utilisateurs** lorsqu'ils débloquent un badge ou une récompense.
- **Rappeler aux utilisateurs** qu'ils ont des points à échanger.
- **Gérer les préférences de notification** (ex: désactiver les notifications pour certaines récompenses).
- **Suivre les notifications envoyées**.

---

## Exigences Fonctionnelles
- **FR-90**: Envoyer des notifications aux utilisateurs pour les nouvelles récompenses.
- **FR-91**: Notifier les utilisateurs lorsqu'ils débloquent un badge ou une récompense.
- **FR-92**: Rappeler aux utilisateurs qu'ils ont des points à échanger.
- **FR-93**: Gérer les préférences de notification des utilisateurs.
- **FR-94**: Suivre les notifications envoyées.

---

## Critères d'Acceptation
1. **Notifications pour les nouvelles récompenses** :
   - [ ] Envoyer une notification lorsqu'une nouvelle récompense est disponible pour l'utilisateur.
   - [ ] Inclure les détails de la récompense dans la notification.

2. **Notifications pour les badges débloqués** :
   - [ ] Envoyer une notification lorsqu'un utilisateur débloque un badge.
   - [ ] Inclure les détails du badge dans la notification.

3. **Rappels pour les points à échanger** :
   - [ ] Envoyer un rappel périodique aux utilisateurs ayant des points non échangés.
   - [ ] Inclure le solde de points dans le rappel.

4. **Gestion des préférences** :
   - [ ] Permettre aux utilisateurs d'activer/désactiver les notifications.
   - [ ] Permettre aux utilisateurs de choisir les types de notifications (badges, récompenses, rappels).

5. **Suivi des notifications** :
   - [ ] Enregistrer les notifications envoyées.
   - [ ] Récupérer l'historique des notifications d'un utilisateur.

6. **Intégration avec le RecommendationEngine** :
   - [ ] Le `RecommendationEngine` utilise le `RewardNotificationSystem` pour gérer les notifications.

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/reward_notification_system.py` | Module `RewardNotificationSystem` | ⏳ À créer |
| `backend/engines/recommendation_engine/__init__.py` | Intégration du `RewardNotificationSystem` | ⏳ À modifier |
| `tests/unit/test_reward_notification_system.py` | Tests unitaires | ⏳ À créer |

### Enum `NotificationType` et `NotificationStatus`
Définition des types et statuts de notifications :

```python
from enum import Enum

class NotificationType(Enum):
    NEW_REWARD = "new_reward"
    BADGE_UNLOCKED = "badge_unlocked"
    POINTS_REMINDER = "points_reminder"
    REWARD_REDEEMED = "reward_redeemed"

class NotificationStatus(Enum):
    PENDING = "pending"
    SENT = "sent"
    READ = "read"
    FAILED = "failed"
```

### Module `RewardNotificationSystem`
Ce module gère les notifications pour les récompenses et les badges :

```python
from typing import Dict, List, Optional, Callable
from dataclasses import dataclass, field
from datetime import datetime
from enum import Enum

class NotificationType(Enum):
    NEW_REWARD = "new_reward"
    BADGE_UNLOCKED = "badge_unlocked"
    POINTS_REMINDER = "points_reminder"
    REWARD_REDEEMED = "reward_redeemed"

class NotificationStatus(Enum):
    PENDING = "pending"
    SENT = "sent"
    READ = "read"
    FAILED = "failed"

@dataclass
class Notification:
    id: str
    user_id: str
    notification_type: NotificationType
    title: str
    message: str
    data: Dict = field(default_factory=dict)
    status: NotificationStatus = NotificationStatus.PENDING
    created_at: datetime = field(default_factory=datetime.now)
    sent_at: Optional[datetime] = None
    read_at: Optional[datetime] = None

@dataclass
class UserNotificationPreferences:
    user_id: str
    notifications_enabled: bool = True
    badge_notifications: bool = True
    reward_notifications: bool = True
    points_reminder_notifications: bool = True

class RewardNotificationSystem:
    def __init__(self):
        # Notifications : {notification_id: Notification}
        self._notifications: Dict[str, Notification] = {}
        # Préférences des utilisateurs : {user_id: UserNotificationPreferences}
        self._user_preferences: Dict[str, UserNotificationPreferences] = {}
        # Callbacks pour l'envoi de notifications (ex: email, push)
        self._notification_callbacks: List[Callable] = []

    def send_notification(
        self,
        user_id: str,
        notification_type: NotificationType,
        title: str,
        message: str,
        data: Optional[Dict] = None
    ) -> Optional[Notification]:
        """Envoye une notification à un utilisateur."""
        # Vérifier les préférences de l'utilisateur
        if not self._can_send_notification(user_id, notification_type):
            return None

        # Créer la notification
        notification_id = f"{user_id}_{notification_type.value}_{datetime.now().timestamp()}"
        notification = Notification(
            id=notification_id,
            user_id=user_id,
            notification_type=notification_type,
            title=title,
            message=message,
            data=data or {},
            status=NotificationStatus.PENDING
        )
        self._notifications[notification_id] = notification

        # Envoyer la notification via les callbacks
        for callback in self._notification_callbacks:
            try:
                callback(notification)
                notification.status = NotificationStatus.SENT
                notification.sent_at = datetime.now()
            except Exception:
                notification.status = NotificationStatus.FAILED

        return notification

    def _can_send_notification(self, user_id: str, notification_type: NotificationType) -> bool:
        """Vérifie si une notification peut être envoyée à un utilisateur."""
        preferences = self._user_preferences.get(user_id, UserNotificationPreferences(user_id=user_id))
        if not preferences.notifications_enabled:
            return False

        if notification_type == NotificationType.BADGE_UNLOCKED and not preferences.badge_notifications:
            return False
        if notification_type == NotificationType.NEW_REWARD and not preferences.reward_notifications:
            return False
        if notification_type == NotificationType.POINTS_REMINDER and not preferences.points_reminder_notifications:
            return False

        return True

    def notify_new_reward(self, user_id: str, reward_data: Dict) -> Optional[Notification]:
        """Notifie un utilisateur qu'une nouvelle récompense est disponible."""
        return self.send_notification(
            user_id=user_id,
            notification_type=NotificationType.NEW_REWARD,
            title="Nouvelle récompense disponible !",
            message=f"Une nouvelle récompense est disponible : {reward_data.get('name', 'Récompense')}",
            data={"reward": reward_data}
        )

    def notify_badge_unlocked(self, user_id: str, badge_data: Dict) -> Optional[Notification]:
        """Notifie un utilisateur qu'il a débloqué un badge."""
        return self.send_notification(
            user_id=user_id,
            notification_type=NotificationType.BADGE_UNLOCKED,
            title="Badge débloqué !",
            message=f"Vous avez débloqué le badge : {badge_data.get('name', 'Badge')}",
            data={"badge": badge_data}
        )

    def notify_points_reminder(self, user_id: str, points_balance: int) -> Optional[Notification]:
        """Rappelle à un utilisateur qu'il a des points à échanger."""
        return self.send_notification(
            user_id=user_id,
            notification_type=NotificationType.POINTS_REMINDER,
            title="Points à échanger !",
            message=f"Vous avez {points_balance} points à échanger contre des récompenses.",
            data={"points_balance": points_balance}
        )

    def notify_reward_redeemed(self, user_id: str, reward_data: Dict, points_used: int) -> Optional[Notification]:
        """Notifie un utilisateur qu'il a échangé des points contre une récompense."""
        return self.send_notification(
            user_id=user_id,
            notification_type=NotificationType.REWARD_REDEEMED,
            title="Récompense échangée !",
            message=f"Vous avez échangé {points_used} points contre : {reward_data.get('name', 'Récompense')}",
            data={"reward": reward_data, "points_used": points_used}
        )

    def get_user_notifications(self, user_id: str, status: Optional[NotificationStatus] = None) -> List[Notification]:
        """Récupère les notifications d'un utilisateur."""
        notifications = [
            n for n in self._notifications.values() if n.user_id == user_id
        ]
        if status:
            notifications = [n for n in notifications if n.status == status]
        return sorted(notifications, key=lambda x: x.created_at, reverse=True)

    def get_unread_notifications(self, user_id: str) -> List[Notification]:
        """Récupère les notifications non lues d'un utilisateur."""
        return [
            n for n in self.get_user_notifications(user_id)
            if n.status == NotificationStatus.PENDING or n.status == NotificationStatus.SENT
        ]

    def mark_notification_as_read(self, notification_id: str) -> bool:
        """Marque une notification comme lue."""
        if notification_id not in self._notifications:
            return False
        self._notifications[notification_id].status = NotificationStatus.READ
        self._notifications[notification_id].read_at = datetime.now()
        return True

    def mark_all_notifications_as_read(self, user_id: str) -> int:
        """Marque toutes les notifications d'un utilisateur comme lues."""
        count = 0
        for notification in self.get_user_notifications(user_id):
            if self.mark_notification_as_read(notification.id):
                count += 1
        return count

    def set_user_notification_preferences(
        self,
        user_id: str,
        notifications_enabled: Optional[bool] = None,
        badge_notifications: Optional[bool] = None,
        reward_notifications: Optional[bool] = None,
        points_reminder_notifications: Optional[bool] = None
    ) -> bool:
        """Définit les préférences de notification d'un utilisateur."""
        if user_id not in self._user_preferences:
            self._user_preferences[user_id] = UserNotificationPreferences(user_id=user_id)

        preferences = self._user_preferences[user_id]
        if notifications_enabled is not None:
            preferences.notifications_enabled = notifications_enabled
        if badge_notifications is not None:
            preferences.badge_notifications = badge_notifications
        if reward_notifications is not None:
            preferences.reward_notifications = reward_notifications
        if points_reminder_notifications is not None:
            preferences.points_reminder_notifications = points_reminder_notifications

        return True

    def get_user_notification_preferences(self, user_id: str) -> Optional[UserNotificationPreferences]:
        """Récupère les préférences de notification d'un utilisateur."""
        return self._user_preferences.get(user_id)

    def register_notification_callback(self, callback: Callable) -> None:
        """Enregistre un callback pour l'envoi de notifications."""
        self._notification_callbacks.append(callback)

    def get_notification(self, notification_id: str) -> Optional[Notification]:
        """Récupère une notification par son ID."""
        return self._notifications.get(notification_id)

    def clear_user_notifications(self, user_id: str) -> int:
        """Supprime toutes les notifications d'un utilisateur."""
        count = 0
        to_delete = [
            notification_id for notification_id, notification in self._notifications.items()
            if notification.user_id == user_id
        ]
        for notification_id in to_delete:
            del self._notifications[notification_id]
            count += 1
        return count

    def clear_all_notifications(self) -> None:
        """Supprime toutes les notifications."""
        self._notifications.clear()

    def get_notification_stats(self, user_id: str) -> Dict:
        """Récupère les statistiques de notifications pour un utilisateur."""
        notifications = self.get_user_notifications(user_id)
        unread = len(self.get_unread_notifications(user_id))
        
        return {
            "total": len(notifications),
            "unread": unread,
            "by_type": {
                "new_reward": len([n for n in notifications if n.notification_type == NotificationType.NEW_REWARD]),
                "badge_unlocked": len([n for n in notifications if n.notification_type == NotificationType.BADGE_UNLOCKED]),
                "points_reminder": len([n for n in notifications if n.notification_type == NotificationType.POINTS_REMINDER]),
                "reward_redeemed": len([n for n in notifications if n.notification_type == NotificationType.REWARD_REDEEMED])
            }
        }
```

---

## Tests Unitaires
Les tests suivants seront créés dans `tests/unit/test_reward_notification_system.py` :
- Envoi de notifications pour les nouvelles récompenses.
- Envoi de notifications pour les badges débloqués.
- Envoi de rappels pour les points à échanger.
- Gestion des préférences de notification.
- Suivi des notifications envoyées.
- Marquage des notifications comme lues.

---

## Configuration Requise
- **Dépendances** : `datetime` pour les timestamps, `dataclasses` pour les structures de données.
- **Intégration** : Dépend du `BadgeSystem` et du `CommunityRewardSystem` pour les données.
- **Callbacks** : Permet d'enregistrer des callbacks pour l'envoi de notifications (ex: email, push).

---

## Dépendances
- **Story 6-9** : `gestion-des-récompenses` (pour gérer les récompenses).
- **Story 6-7** : `échange-de-points-contre-des-récompenses` (pour les notifications d'échange).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine.reward_notification_system import (
    reward_notification_system,
    NotificationType,
    NotificationStatus
)

# Enregistrer un callback pour l'envoi de notifications (ex: envoi d'email)
def send_email_notification(notification):
    print(f"Envoi d'email à {notification.user_id} : {notification.title}")

reward_notification_system.register_notification_callback(send_email_notification)

# Notifier un utilisateur d'une nouvelle récompense
reward_data = {"id": "reward_001", "name": "Réduction de 20%", "points_required": 200}
notification = reward_notification_system.notify_new_reward("user_001", reward_data)
print(f"Notification envoyée : {notification is not None}")

# Notifier un utilisateur qu'il a débloqué un badge
badge_data = {"id": "bike_star", "name": "Vélo Star", "description": "10 trajets en vélo"}
notification = reward_notification_system.notify_badge_unlocked("user_001", badge_data)
print(f"Notification de badge envoyée : {notification is not None}")

# Rappeler à un utilisateur qu'il a des points à échanger
notification = reward_notification_system.notify_points_reminder("user_001", 500)
print(f"Rappel envoyé : {notification is not None}")

# Récupérer les notifications d'un utilisateur
notifications = reward_notification_system.get_user_notifications("user_001")
print(f"Notifications de user_001 : {len(notifications)}")

# Marquer une notification comme lue
reward_notification_system.mark_notification_as_read(notification.id)

# Définir les préférences de notification d'un utilisateur
reward_notification_system.set_user_notification_preferences(
    "user_001",
    badge_notifications=False  # Désactiver les notifications de badges
)

# Récupérer les statistiques de notifications
stats = reward_notification_system.get_notification_stats("user_001")
print(f"Statistiques : {stats}")
```

---

## Notes
- **Callbacks** : Permet d'intégrer différents canaux de notification (email, push, SMS).
- **Préférences utilisateur** : Chaque utilisateur peut personnaliser ses préférences de notification.
- **Suivi** : Toutes les notifications sont enregistrées pour un suivi complet.
- **Statistiques** : Permet de récupérer des statistiques sur les notifications (nombre total, non lues, par type).

---

## Ressources
- [PRD Almanéa - FR-90](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-8](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
