---
title: '7-3-gérer-les-préférences-de-notification'
type: 'feature'
created: '08-17-2026'
status: 'in-progress'
review_loop_iteration: 0
context: []
baseline_commit: NO_VCS
---

<frozen-after-approval reason="human-owned intent — do not modify unless human renegotiates">

## Intent

**Problem:** Les utilisateurs d'Almanéa ne peuvent pas personnaliser les notifications qu'ils reçoivent (ex: types d'alertes, fréquence, canaux). Cela limite l'expérience utilisateur et peut entraîner des notifications non pertinentes ou intrusives.

**Approach:** Implémenter un système de gestion des préférences de notification permettant aux utilisateurs de configurer leurs préférences via une API. Ces préférences seront stockées en base de données et utilisées pour filtrer les notifications et alertes générées par le Context Engine.

## Boundaries & Constraints

**Always:**
- Les préférences de notification doivent être **persistantes** (stockées en base de données PostgreSQL).
- Les préférences doivent être **appliquées en temps réel** pour filtrer les notifications et alertes.
- Le système doit respecter les **contraintes de confidentialité** (pas de données personnelles exposées).
- Les préférences doivent être **modifiables à tout moment** par l'utilisateur.

**Ask First:**
- Si des **canaux de notification supplémentaires** (ex: SMS, push mobile) doivent être supportés, valider avec l'équipe produit.
- Si des **règles de priorité complexes** (ex: notifications urgentes uniquement) doivent être implémentées, clarifier les exigences.

**Never:**
- Ne pas stocker les préférences **uniquement en mémoire** (risque de perte de données).
- Ne pas envoyer de notifications **sans vérifier les préférences utilisateur**.
- Ne pas exposer les préférences via une API **non sécurisée** (authentification requise).

## I/O & Edge-Case Matrix

| Scenario | Input / State | Expected Output / Behavior | Error Handling |
|----------|--------------|---------------------------|----------------|
| **Récupération des préférences** | Utilisateur authentifié avec `user_id` valide | Retourne les préférences de notification de l'utilisateur | 404 si aucune préférence n'existe |
| **Mise à jour des préférences** | Utilisateur authentifié + payload valide | Met à jour les préférences en base de données | 400 si le payload est invalide |
| **Préférences par défaut** | Nouveau utilisateur sans préférences | Crée des préférences par défaut (ex: notifications activées, canaux email) | Aucune erreur |
| **Désactivation des notifications** | Utilisateur met `notifications_enabled=False` | Aucune notification n'est envoyée à cet utilisateur | Aucune notification envoyée |
| **Filtrage des alertes** | Utilisateur désactive `weather_alerts` | Les alertes météo ne sont pas envoyées | Alertes ignorées |

</frozen-after-approval>

## Code Map

- `/Users/julie/Projects/almanea/backend/engines/recommendation_engine/models.py` -- Contient `UserProfile` et `UserNotificationPreferences` (à étendre pour inclure les préférences de notification génériques).
- `/Users/julie/Projects/almanea/backend/engines/recommendation_engine/user_profile_manager.py` -- Gestionnaire des profils utilisateurs (à étendre pour inclure les préférences de notification).
- `/Users/julie/Projects/almanea/backend/engines/context_engine/models.py` -- Contient `UnifiedContext` et `Alert` (à intégrer avec les préférences pour filtrer les alertes).
- `/Users/julie/Projects/almanea/backend/core/config.py` -- Configuration centrale (à étendre avec les paramètres RabbitMQ/Celery).
- `/Users/julie/Projects/almanea/backend/engines/recommendation_engine/` -- Dossier où ajouter le nouveau gestionnaire `notification_preferences_manager.py`.

## Tasks & Acceptance

**Execution:**
- [ ] `/Users/julie/Projects/almanea/backend/engines/recommendation_engine/models.py` -- Ajouter le modèle `NotificationPreferences` avec les champs : `notifications_enabled`, `alert_types`, `frequency`, `channels` -- Nécessaire pour structurer les données des préférences.
- [ ] `/Users/julie/Projects/almanea/backend/engines/recommendation_engine/notification_preferences_manager.py` -- Créer un gestionnaire pour les opérations CRUD sur les préférences de notification -- Centralise la logique métier.
- [ ] `/Users/julie/Projects/almanea/backend/engines/recommendation_engine/user_profile_manager.py` -- Étendre pour intégrer `NotificationPreferences` dans `UserProfile` -- Permet de lier les préférences à un utilisateur.
- [ ] `/Users/julie/Projects/almanea/backend/core/config.py` -- Ajouter les configurations pour RabbitMQ et Celery -- Nécessaire pour l'envoi asynchrone de notifications.
- [ ] `/Users/julie/Projects/almanea/backend/engines/context_engine/context_builder.py` -- Intégrer le filtrage des alertes en fonction des préférences utilisateur -- Garantit que seules les alertes pertinentes sont envoyées.
- [ ] `/Users/julie/Projects/almanea/backend/` -- Créer les endpoints FastAPI pour : `GET /users/{user_id}/notification-preferences` et `PUT /users/{user_id}/notification-preferences` -- Permet aux utilisateurs de gérer leurs préférences via l'API.

**Acceptance Criteria:**
- Given un utilisateur authentifié, when il appelle `GET /users/{user_id}/notification-preferences`, then ses préférences de notification sont retournées.
- Given un utilisateur authentifié, when il appelle `PUT /users/{user_id}/notification-preferences` avec un payload valide, then ses préférences sont mises à jour en base de données.
- Given un utilisateur avec `notifications_enabled=False`, when une alerte est générée, then aucune notification n'est envoyée.
- Given un utilisateur avec `weather_alerts=False`, when une alerte météo est générée, then l'alerte n'est pas envoyée.
- Given un utilisateur avec des préférences par défaut, when il ne modifie pas ses préférences, then les préférences par défaut sont appliquées.

## Spec Change Log

## Design Notes

**Architecture :**
- Les préférences de notification seront stockées dans **PostgreSQL** pour la persistance, avec un cache **Redis** pour les accès fréquents.
- Les notifications seront envoyées de manière **asynchrone** via **Celery + RabbitMQ** pour garantir un SLA de livraison inférieur à 5 minutes pour les alertes critiques.
- Le modèle `NotificationPreferences` étend le modèle existant `UserNotificationPreferences` pour inclure des options génériques (ex: types d'alertes, fréquence).

**Exemple de payload pour les préférences :**
```json
{
  "notifications_enabled": true,
  "alert_types": {
    "weather_alerts": true,
    "air_quality_alerts": true,
    "energy_alerts": false
  },
  "frequency": "real_time",
  "channels": ["email", "push"]
}
```

## Verification

**Commands:**
- `pytest tests/engines/recommendation_engine/test_notification_preferences.py` -- expected: Tous les tests unitaires pour le gestionnaire de préférences passent.
- `pytest tests/api/test_notification_preferences_routes.py` -- expected: Tous les tests d'intégration pour les endpoints API passent.

**Manual checks (if no CLI):**
- Vérifier que les préférences de notification sont correctement stockées et récupérées via l'API.
- Vérifier que les alertes sont filtrées en fonction des préférences utilisateur.
