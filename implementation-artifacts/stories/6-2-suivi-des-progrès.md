---
name: Suivi des progrès
id: 6-2-suivi-des-progrès
epic: epic-6
story_type: backend
priority: high
estimation: M
dependencies: [6-1-système-de-points, 5-1-créer-et-mettre-à-jour-le-profil-utilisateur]
status: done
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 6.2: Suivi des progrès

## Contexte
Cette story fait partie de **l'Épic 6 (Gamification)**. Son objectif est de **permettre aux utilisateurs de suivre leurs progrès** au fil du temps, en fournissant des métriques comme :
- **Nombre de recommandations suivies**.
- **Points accumulés**.
- **Défis complétés**.
- **Série de jours consécutifs avec activité** (streak).

Le `ProgressTracker` permet de :
- **Suivre les métriques de progrès** pour chaque utilisateur.
- **Générer des rapports de progrès** (quotidien, hebdomadaire, mensuel).
- **Calculer des statistiques** (ex: nombre total de recommandations suivies).
- **Suivre les séries d'activité** (streaks).

---

## Exigences Fonctionnelles
- **FR-54**: Suivre les métriques de progrès pour chaque utilisateur.
- **FR-55**: Générer des rapports de progrès (quotidien, hebdomadaire, mensuel).
- **FR-56**: Calculer des statistiques à partir des métriques de progrès.
- **FR-57**: Suivre les séries de jours consécutifs avec activité.

---

## Critères d'Acceptation
1. **Suivi des métriques** :
   - [x] Mettre à jour une métrique de progrès pour un utilisateur (ex: `RECOMMENDATIONS_FOLLOWED`).
   - [x] Incrémenter une métrique d'une valeur personnalisée.

2. **Récupération des métriques** :
   - [x] Récupérer la valeur d'une métrique spécifique pour un utilisateur.
   - [x] Récupérer toutes les métriques pour un utilisateur.

3. **Rapports de progrès** :
   - [x] Générer un rapport de progrès quotidien.
   - [x] Générer un rapport de progrès hebdomadaire.
   - [x] Générer un rapport de progrès mensuel.

4. **Suivi des séries (streaks)** :
   - [x] Calculer la série de jours consécutifs avec activité pour un utilisateur.

5. **Statistiques** :
   - [x] Générer un rapport complet avec les métriques, la série et le total de points.

6. **Gestion du progrès** :
   - [x] Réinitialiser le progrès d'un utilisateur.
   - [x] Supprimer tout le progrès.

7. **Intégration avec le RecommendationEngine** :
   - [x] Le `RecommendationEngine` utilise le `ProgressTracker` pour suivre les métriques.

---

## Implémentation Technique

### Fichiers créés/modifiés
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/progress_tracker.py` | Module `ProgressTracker` | ✅ Créé |
| `backend/engines/recommendation_engine/__init__.py` | Intégration du `ProgressTracker` | ✅ Modifié |
| `tests/unit/test_progress_tracker.py` | Tests unitaires | ✅ Créé |

### Enum `ProgressMetric`
Définition des métriques de progrès suivies :

```python
from enum import Enum

class ProgressMetric(Enum):
    RECOMMENDATIONS_FOLLOWED = "recommendations_followed"
    POINTS_EARNED = "points_earned"
    CHALLENGES_COMPLETED = "challenges_completed"
    FEEDBACKS_GIVEN = "feedbacks_given"
    DAYS_STREAK = "days_streak"
```

### Module `ProgressTracker`
Ce module suit les métriques de progrès pour chaque utilisateur :

```python
from typing import Dict, List, Optional
from datetime import datetime, timedelta
from .points_system import PointsAction

class ProgressTracker:
    def __init__(self):
        # Métriques par utilisateur : {user_id: {ProgressMetric: value}}
        self._user_metrics: Dict[str, Dict[ProgressMetric, int]] = {}
        # Historique des activités : {user_id: [{"metric": ProgressMetric, "value": int, "timestamp": datetime}]}
        self._user_activity_history: Dict[str, List[Dict]] = {}
        # Dernière activité par utilisateur : {user_id: datetime}
        self._last_activity: Dict[str, datetime] = {}

    def update_metric(self, user_id: str, metric: ProgressMetric, increment: int = 1) -> None:
        """Met à jour une métrique de progrès pour un utilisateur."""
        if user_id not in self._user_metrics:
            self._user_metrics[user_id] = {m: 0 for m in ProgressMetric}
        self._user_metrics[user_id][metric] += increment
        # Enregistrer dans l'historique
        self._user_activity_history.setdefault(user_id, []).append({
            "metric": metric,
            "value": increment,
            "timestamp": datetime.now()
        })
        self._last_activity[user_id] = datetime.now()

    def get_metric(self, user_id: str, metric: ProgressMetric) -> int:
        """Récupère la valeur d'une métrique pour un utilisateur."""
        if user_id not in self._user_metrics:
            return 0
        return self._user_metrics[user_id].get(metric, 0)

    def get_all_metrics(self, user_id: str) -> Dict[ProgressMetric, int]:
        """Récupère toutes les métriques pour un utilisateur."""
        if user_id not in self._user_metrics:
            return {m: 0 for m in ProgressMetric}
        return self._user_metrics[user_id]

    def get_daily_progress(self, user_id: str) -> Dict[str, int]:
        """Récupère le progrès quotidien pour un utilisateur."""
        today = datetime.now().date()
        daily_metrics = {m.value: 0 for m in ProgressMetric}
        for activity in self._user_activity_history.get(user_id, []):
            if activity["timestamp"].date() == today:
                daily_metrics[activity["metric"].value] += activity["value"]
        return daily_metrics

    def get_weekly_progress(self, user_id: str) -> Dict[str, int]:
        """Récupère le progrès hebdomadaire pour un utilisateur."""
        today = datetime.now().date()
        start_of_week = today - timedelta(days=today.weekday())
        weekly_metrics = {m.value: 0 for m in ProgressMetric}
        for activity in self._user_activity_history.get(user_id, []):
            if start_of_week <= activity["timestamp"].date() <= today:
                weekly_metrics[activity["metric"].value] += activity["value"]
        return weekly_metrics

    def get_monthly_progress(self, user_id: str) -> Dict[str, int]:
        """Récupère le progrès mensuel pour un utilisateur."""
        today = datetime.now().date()
        start_of_month = today.replace(day=1)
        monthly_metrics = {m.value: 0 for m in ProgressMetric}
        for activity in self._user_activity_history.get(user_id, []):
            if start_of_month <= activity["timestamp"].date() <= today:
                monthly_metrics[activity["metric"].value] += activity["value"]
        return monthly_metrics

    def get_streak(self, user_id: str) -> int:
        """Récupère la série de jours consécutifs avec activité."""
        if user_id not in self._last_activity:
            return 0
        today = datetime.now().date()
        last_activity_date = self._last_activity[user_id].date()
        if last_activity_date != today:
            return 0
        # Compter les jours consécutifs
        streak = 1
        current_date = last_activity_date - timedelta(days=1)
        while current_date >= (last_activity_date - timedelta(days=365)):
            has_activity = any(
                activity["timestamp"].date() == current_date
                for activity in self._user_activity_history.get(user_id, [])
            )
            if has_activity:
                streak += 1
                current_date -= timedelta(days=1)
            else:
                break
        return streak

    def get_progress_report(self, user_id: str, period: str = "daily") -> Dict:
        """Génère un rapport de progrès pour un utilisateur."""
        if period == "daily":
            metrics = self.get_daily_progress(user_id)
        elif period == "weekly":
            metrics = self.get_weekly_progress(user_id)
        elif period == "monthly":
            metrics = self.get_monthly_progress(user_id)
        else:
            metrics = {m.value: self.get_metric(user_id, m) for m in ProgressMetric}
        return {
            "period": period,
            "metrics": metrics,
            "streak": self.get_streak(user_id),
            "total_points": self.get_metric(user_id, ProgressMetric.POINTS_EARNED),
            "total_recommendations_followed": self.get_metric(user_id, ProgressMetric.RECOMMENDATIONS_FOLLOWED)
        }

    def reset_user_progress(self, user_id: str) -> bool:
        """Réinitialise le progrès d'un utilisateur."""
        if user_id not in self._user_metrics:
            return False
        self._user_metrics[user_id] = {m: 0 for m in ProgressMetric}
        self._user_activity_history[user_id] = []
        if user_id in self._last_activity:
            del self._last_activity[user_id]
        return True

    def clear_all_progress(self) -> None:
        """Supprime tout le progrès des utilisateurs."""
        self._user_metrics.clear()
        self._user_activity_history.clear()
        self._last_activity.clear()
```

---

## Tests Unitaires
17 tests ont été créés dans `tests/unit/test_progress_tracker.py` pour valider :
- La mise à jour des métriques.
- La récupération des métriques par utilisateur.
- La génération de rapports de progrès (quotidien, hebdomadaire, mensuel).
- Le calcul des séries (streaks).
- La réinitialisation du progrès.
- Le déterminisme du module.

---

## Configuration Requise
- **Dépendances** : `datetime` pour les timestamps, `timedelta` pour les calculs de dates.
- **Stockage** : En mémoire pour la phase PoC (à remplacer par une base de données pour la phase GA).

---

## Dépendances
- **Story 6-1** : `système-de-points` (pour le suivi des points).
- **Story 5-1** : `créer-et-mettre-à-jour-le-profil-utilisateur` (pour l'identification des utilisateurs).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine.progress_tracker import progress_tracker, ProgressMetric

# Mettre à jour une métrique
progress_tracker.update_metric(
    user_id="user_001",
    metric=ProgressMetric.RECOMMENDATIONS_FOLLOWED
)

# Mettre à jour une autre métrique
progress_tracker.update_metric(
    user_id="user_001",
    metric=ProgressMetric.POINTS_EARNED,
    increment=50
)

# Récupérer une métrique
recommendations_followed = progress_tracker.get_metric(
    user_id="user_001",
    metric=ProgressMetric.RECOMMENDATIONS_FOLLOWED
)
print(f"Recommandations suivies : {recommendations_followed}")

# Récupérer toutes les métriques
metrics = progress_tracker.get_all_metrics("user_001")
print(f"Métriques : {metrics}")

# Générer un rapport de progrès quotidien
report = progress_tracker.get_progress_report("user_001", period="daily")
print(f"Rapport quotidien : {report}")

# Récupérer la série de jours
streak = progress_tracker.get_streak("user_001")
print(f"Série de jours : {streak}")

# Générer un rapport hebdomadaire
weekly_report = progress_tracker.get_progress_report("user_001", period="weekly")
print(f"Rapport hebdomadaire : {weekly_report}")

# Réinitialiser le progrès
progress_tracker.reset_user_progress("user_001")
```

---

## Notes
- **Stockage en mémoire** : Les métriques sont stockées en mémoire pour la phase PoC.
- **Rapports flexibles** : Permet de générer des rapports pour différentes périodes (quotidien, hebdomadaire, mensuel).
- **Séries (streaks)** : Calcule automatiquement les séries de jours consécutifs avec activité.
- **Statistiques** : Fournit des métriques clés comme le total de points et de recommandations suivies.
- **Déterminisme** : Le module est déterministe, ce qui facilite les tests.

---

## Ressources
- [PRD Almanéa - FR-54](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-8](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
