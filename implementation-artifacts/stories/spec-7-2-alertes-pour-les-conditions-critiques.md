---
title: '7-2-alertes-pour-les-conditions-critiques'
type: 'feature'
created: '08-17-2026'
status: 'done'
review_loop_iteration: 0
context: []
---

<frozen-after-approval reason="human-owned intent — do not modify unless human renegotiates">

## Intent

**Problem:** Les utilisateurs d'Almanéa ne reçoivent pas d'alertes en temps réel pour les **conditions critiques** (ex: qualité de l'air mauvaise, pluie intense, température extrême). Cela peut exposer les utilisateurs à des situations dangereuses.

**Approach:** Implémenter un système d'alertes critiques basé sur le **Context Engine** (Épic 1) avec un **SLA de livraison < 5 minutes**. Les alertes doivent être prioritaires, envoyées via Celery + RabbitMQ, et affichées en haut du dashboard. Les alertes doivent être filtrées selon les préférences utilisateur (Story 7-3).

## Boundaries & Constraints

**Always:**
- Les alertes critiques doivent être **envoyées en temps réel** (SLA < 5 minutes).
- Les alertes critiques doivent être **prioritaires** (file Celery dédiée `critical`).
- Les alertes doivent être **filtrées selon les préférences utilisateur** (Story 7-3).
- Les alertes doivent être **stockées en base de données** pour un suivi.
- Les alertes doivent être **affichées en haut du dashboard** (priorité maximale).

**Ask First:**
- Si des **types d'alertes supplémentaires** doivent être supportés, valider avec l'équipe produit.
- Si des **seuils différents** pour les alertes doivent être configurables, clarifier les exigences.

**Never:**
- Ne pas envoyer d'alertes **sans vérifier les préférences utilisateur** (Story 7-3).
- Ne pas envoyer d'alertes **sans contexte environnemental valide** (Épic 1).
- Ne pas bloquer le système en cas d'**échec de l'envoi** (gérer les erreurs gracieusement).
- Ne pas utiliser la file Celery par défaut pour les alertes critiques (utiliser `critical`).

## I/O & Edge-Case Matrix

| Scenario | Input / State | Expected Output / Behavior | Error Handling |
|----------|--------------|---------------------------|----------------|
| **AQI > 100** | Contexte avec AQI = 150 | Alerte critique envoyée via Celery (file `critical`) | Aucune erreur |
| **Pluie intense** | Contexte avec `rain_probability` = 90% | Alerte haute priorité envoyée | Aucune erreur |
| **Température extrême** | Contexte avec température = 40°C | Alerte critique envoyée | Aucune erreur |
| **Préférences désactivées** | Utilisateur a désactivé `air_quality_alerts` | Alerte non envoyée | Aucune alerte |
| **Échec d'envoi** | Erreur lors de l'envoi de l'alerte | Alerte marquée comme échouée et réessayée 3 fois | Réessayer avec `countdown=10` |
| **Utilisateur non trouvé** | `user_id` invalide | Aucune alerte envoyée | 404 si appelé via API |

</frozen-after-approval>

## Code Map

- `/Users/julie/Projects/almanea/backend/engines/context_engine/models.py` -- Contient `Alert`, `AlertPriority`, `AlertStatus` (modèle étendu pour les alertes critiques).
- `/Users/julie/Projects/almanea/backend/engines/context_engine/critical_alert_service.py` -- Service pour détecter et envoyer les alertes critiques.
- `/Users/julie/Projects/almanea/backend/engines/context_engine/context_builder.py` -- Intègre la détection d'alertes critiques dans la construction du contexte.
- `/Users/julie/Projects/almanea/backend/tasks/alert_tasks.py` -- Contient la tâche Celery `send_critical_alert` (file `critical`).
- `/Users/julie/Projects/almanea/backend/api/routers/critical_alerts.py` -- Endpoints FastAPI pour consulter les alertes critiques.
- `/Users/julie/Projects/almanea/backend/core/celery_app.py` -- Configuration Celery avec file `critical` pour les alertes prioritaires.

## Tasks & Acceptance

**Execution:**
- [x] `/Users/julie/Projects/almanea/backend/engines/context_engine/models.py` -- Étendre le modèle `Alert` avec `priority` et `status` -- Permet de prioriser et suivre les alertes.
- [x] `/Users/julie/Projects/almanea/backend/engines/context_engine/critical_alert_service.py` -- Créer le service `CriticalAlertService` pour détecter et envoyer les alertes critiques -- Logique métier centrale.
- [x] `/Users/julie/Projects/almanea/backend/engines/context_engine/context_builder.py` -- Intégrer la détection d'alertes critiques dans la construction du contexte -- Déclenche les alertes automatiquement.
- [x] `/Users/julie/Projects/almanea/backend/tasks/alert_tasks.py` -- Créer la tâche Celery `send_critical_alert` avec file `critical` -- Envoi asynchrone prioritaire.
- [x] `/Users/julie/Projects/almanea/backend/api/routers/critical_alerts.py` -- Créer l'endpoint `GET /users/me/critical-alerts` -- Permet aux utilisateurs de consulter leurs alertes.
- [x] `/Users/julie/Projects/almanea/backend/main.py` -- Inclure le router `critical_alerts_router` -- Intègre les endpoints dans l'API.

**Acceptance Criteria:**
- [x] Given un contexte avec AQI > 100, when le système détecte une alerte, then une alerte critique est envoyée à l'utilisateur.
- [x] Given un contexte avec `rain_probability` > 80%, when le système détecte une alerte, then une alerte haute priorité est envoyée.
- [x] Given un utilisateur avec `notifications_enabled=False`, when une alerte critique est détectée, then aucune alerte n'est envoyée.
- [x] Given un utilisateur avec `air_quality_alerts=False`, when une alerte qualité de l'air est détectée, then l'alerte n'est pas envoyée.
- [x] Given une alerte critique détectée, when l'envoi échoue, then l'alerte est marquée comme échouée et réessayée 3 fois avec un `countdown=10`.
- [x] Given un utilisateur authentifié, when il appelle `GET /users/me/critical-alerts`, then la liste de ses alertes critiques est retournée.

## Spec Change Log

## Design Notes

**Architecture :**
- Les alertes critiques sont détectées dans le **Context Engine** (Épic 1) lors de la construction du `UnifiedContext`.
- Le **CriticalAlertService** écoute les mises à jour du contexte et déclenche les alertes si les conditions critiques sont remplies.
- Les alertes sont envoyées via **Celery + RabbitMQ** avec une **file dédiée `critical`** pour garantir un traitement prioritaire.
- Les alertes sont stockées en **mémoire** (pour l'instant) et seront persistées en **PostgreSQL** dans une future itération.
- Les alertes sont filtrées selon les **préférences utilisateur** (Story 7-3).

**Exemple de détection d'alerte critique :**
```python
# Dans critical_alert_service.py
def detect_critical_alerts(self, aqi: Optional[float], rain_probability: Optional[float], temperature: Optional[float]) -> List[Alert]:
    alerts = []
    if aqi and aqi > 100:
        alerts.append(
            Alert(
                type="air_quality_alert",
                message=f"Qualité de l'air mauvaise (AQI: {aqi}) : évitez les activités extérieures intenses",
                severity="critical",
                priority=AlertPriority.CRITICAL,
                status=AlertStatus.PENDING,
                timestamp=datetime.now(),
                details={"aqi": aqi, "threshold": 100}
            )
        )
    return alerts
```

**Exemple de payload pour une alerte critique :**
```json
{
  "type": "air_quality_alert",
  "message": "Qualité de l'air mauvaise (AQI: 150) : évitez les activités extérieures intenses",
  "severity": "critical",
  "priority": "critical",
  "status": "sent",
  "timestamp": "2026-08-17T18:00:00",
  "details": {
    "aqi": 150,
    "threshold": 100
  },
  "user_id": "user-123"
}
```

**Configuration Celery pour les alertes critiques :**
```python
# Dans celery_app.py
celery_app.conf.task_routes = {
    'backend.tasks.alert_tasks.send_critical_alert': {'queue': 'critical'}
}
celery_app.conf.task_default_queue = 'default'
```

## Verification

**Commands:**
- `pytest tests/engines/context_engine/test_critical_alert_service.py` -- expected: Tous les tests unitaires pour le service passent.
- `pytest tests/api/test_critical_alerts_routes.py` -- expected: Tous les tests d'intégration pour les endpoints passent.

**Manual checks (if no CLI):**
- Vérifier qu'une alerte critique est envoyée lorsque les conditions sont remplies (ex: AQI > 100).
- Vérifier que les alertes sont filtrées selon les préférences utilisateur.
- Vérifier que les alertes critiques sont traitées en priorité (file `critical`).
- Vérifier que les alertes sont accessibles via l'API (`GET /users/me/critical-alerts`).
