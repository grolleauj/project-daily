---
title: '7-1-notifications-pour-les-opportunités-temporaires'
type: 'feature'
created: '08-17-2026'
status: 'done'
review_loop_iteration: 0
context: []
---

<frozen-after-approval reason="human-owned intent — do not modify unless human renegotiates">

## Intent

**Problem:** Les utilisateurs d'Almanéa ne reçoivent pas de notifications pour les **opportunités temporaires** (ex: qualité de l'air excellente pour une balade à vélo). Cela limite leur capacité à profiter des meilleures conditions en temps réel.

**Approach:** Implémenter un système de détection d'opportunités basé sur le **Context Engine** (Épic 1) et envoyer des notifications personnalisées via email (PoC) ou appli native (EA). Les notifications doivent être filtrées selon les préférences utilisateur (Story 7-3).

## Boundaries & Constraints

**Always:**
- Les notifications doivent être **basées sur le contexte environnemental** (ex: qualité de l'air, météo).
- Les notifications doivent être **personnalisées** selon les préférences utilisateur (Story 7-3).
- Les notifications doivent être **envoyées via Celery** pour un traitement asynchrone.
- Les notifications doivent être **stockées en base de données** pour un suivi.

**Ask First:**
- Si des **canaux de notification supplémentaires** (ex: SMS) doivent être supportés, valider avec l'équipe produit.
- Si des **règles de détection complexes** doivent être implémentées, clarifier les exigences.

**Never:**
- Ne pas envoyer de notifications **sans vérifier les préférences utilisateur** (Story 7-3).
- Ne pas envoyer de notifications **sans contexte environnemental valide** (Épic 1).
- Ne pas bloquer le système en cas d'**échec de l'envoi** (gérer les erreurs gracieusement).

## I/O & Edge-Case Matrix

| Scenario | Input / State | Expected Output / Behavior | Error Handling |
|----------|--------------|---------------------------|----------------|
| **Opportunité détectée** | Contexte avec AQI excellent + météo favorable | Notification envoyée via email | Aucune erreur |
| **Préférences désactivées** | Utilisateur a désactivé `weather_alerts` | Notification non envoyée | Aucune notification |
| **Contexte invalide** | Données manquantes (ex: AQI non disponible) | Aucune notification envoyée | Loguer l'erreur |
| **Échec d'envoi** | Erreur lors de l'envoi de l'email | Notification marquée comme échouée | Réessayer 3 fois |
| **Utilisateur non trouvé** | `user_id` invalide | Aucune notification envoyée | 404 si appelé via API |

</frozen-after-approval>

## Code Map

- `/Users/julie/Projects/almanea/backend/engines/context_engine/models.py` -- Contient `UnifiedContext` et `Observation` (utilisé pour détecter les opportunités).
- `/Users/julie/Projects/almanea/backend/engines/context_engine/context_builder.py` -- Construit le contexte unifié (intègre la détection d'opportunités).
- `/Users/julie/Projects/almanea/backend/engines/recommendation_engine/models.py` -- Contient `OpportunityNotification`, `OpportunityType`, `OpportunityNotificationStatus` (modèles pour les notifications d'opportunités).
- `/Users/julie/Projects/almanea/backend/engines/recommendation_engine/opportunity_notification_service.py` -- Service pour détecter et envoyer les notifications d'opportunités.
- `/Users/julie/Projects/almanea/backend/engines/recommendation_engine/notification_preferences_manager.py` -- Gestionnaire des préférences (à réutiliser pour vérifier `can_send_notification`).
- `/Users/julie/Projects/almanea/backend/tasks/opportunity_tasks.py` -- Contient la tâche Celery `send_opportunity_notification`.
- `/Users/julie/Projects/almanea/backend/api/routers/opportunity_notifications.py` -- Endpoints FastAPI pour consulter les notifications d'opportunités.

## Tasks & Acceptance

**Execution:**
- [x] `/Users/julie/Projects/almanea/backend/engines/recommendation_engine/models.py` -- Ajouter le modèle `OpportunityNotification` avec les champs : `user_id`, `message`, `opportunity_type`, `context`, `status`, `created_at` -- Structurer les données des notifications d'opportunités.
- [x] `/Users/julie/Projects/almanea/backend/engines/recommendation_engine/opportunity_notification_service.py` -- Créer un service pour détecter les opportunités temporaires (ex: AQI excellent + météo favorable) et déclencher les notifications -- Logique métier centrale.
- [x] `/Users/julie/Projects/almanea/backend/engines/context_engine/context_builder.py` -- Intégrer la détection d'opportunités dans la construction du contexte -- Déclencher les notifications automatiquement.
- [x] `/Users/julie/Projects/almanea/backend/api/routers/opportunity_notifications.py` -- Créer les endpoints FastAPI pour : `GET /users/{user_id}/opportunity-notifications` (lister les notifications) -- Permet aux utilisateurs de consulter leurs notifications.
- [x] `/Users/julie/Projects/almanea/backend/tasks/opportunity_tasks.py` -- Créer une tâche Celery `send_opportunity_notification` pour envoyer les notifications d'opportunités -- Envoi asynchrone.
- [x] `/Users/julie/Projects/almanea/backend/main.py` -- Inclure le router `opportunity_notifications_router` -- Intègre les endpoints dans l'API.

**Acceptance Criteria:**
- [x] Given un contexte avec AQI excellent et météo favorable, when le système détecte une opportunité, then une notification est envoyée à l'utilisateur.
- [x] Given un utilisateur avec `notifications_enabled=False`, when une opportunité est détectée, then aucune notification n'est envoyée.
- [x] Given un utilisateur avec `weather_alerts=False`, when une opportunité météo est détectée, then la notification n'est pas envoyée.
- [x] Given une opportunité détectée, when l'envoi échoue, then la notification est marquée comme échouée et réessayée 3 fois.
- [x] Given un utilisateur authentifié, when il appelle `GET /users/{user_id}/opportunity-notifications`, then la liste de ses notifications d'opportunités est retournée.

## Spec Change Log

## Design Notes

**Architecture :**
- Les opportunités temporaires sont détectées dans le **Context Engine** (Épic 1) lors de la construction du `UnifiedContext`.
- Le **OpportunityNotificationService** écoute les mises à jour du contexte et déclenche les notifications si les conditions sont remplies.
- Les notifications sont envoyées via **Celery + RabbitMQ** (tâche `send_opportunity_notification`).
- Les notifications sont stockées en **mémoire** (pour l'instant) et seront persistées en **PostgreSQL** dans une future itération.
- Les notifications sont filtrées selon les **préférences utilisateur** (Story 7-3).

**Exemple de détection d'opportunité :**
```python
# Dans opportunity_notification_service.py
def detect_opportunities(self, unified_context: UnifiedContext) -> List[OpportunityNotification]:
    opportunities = []
    aqi = unified_context.get_value("atmo-france", "aqi")
    rain_probability = unified_context.get_value("rte", "rain_probability")
    
    if aqi and aqi < 50 and rain_probability and rain_probability < 30:
        opportunities.append(
            OpportunityNotification(
                user_id=unified_context.user_profile.user_id,
                message="Qualité de l'air excellente : balade à vélo recommandée !",
                opportunity_type="bike_ride",
                context={"aqi": aqi, "rain_probability": rain_probability},
                status="pending"
            )
        )
    return opportunities
```

**Exemple de payload pour une notification d'opportunité :**
```json
{
  "user_id": "user-123",
  "message": "Qualité de l'air excellente : balade à vélo recommandée !",
  "opportunity_type": "bike_ride",
  "context": {
    "aqi": 42,
    "rain_probability": 10,
    "temperature": 22
  },
  "status": "sent",
  "created_at": "2026-08-17T18:00:00"
}
```

## Verification

**Commands:**
- `pytest tests/engines/recommendation_engine/test_opportunity_notification_service.py` -- expected: Tous les tests unitaires pour le service passent.
- `pytest tests/api/test_opportunity_notifications_routes.py` -- expected: Tous les tests d'intégration pour les endpoints passent.

**Manual checks (if no CLI):**
- Vérifier qu'une notification d'opportunité est envoyée lorsque les conditions sont remplies.
- Vérifier que les notifications sont filtrées selon les préférences utilisateur.
- Vérifier que les notifications sont stockées et accessibles via l'API (`GET /users/me/opportunity-notifications`).
