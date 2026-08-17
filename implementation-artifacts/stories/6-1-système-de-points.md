---
name: Système de points
id: 6-1-système-de-points
epic: epic-6
story_type: backend
priority: high
estimation: M
dependencies: [5-1-créer-et-mettre-à-jour-le-profil-utilisateur]
status: done
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 6.1: Système de points

## Contexte
Cette story fait partie de **l'Épic 6 (Gamification)**. Son objectif est de **mettre en place un système de points** pour récompenser les utilisateurs lorsqu'ils :
- **Suivent des recommandations** (ex: +10 points).
- **Complètent des défis** (ex: +50 points).
- **Donnent un feedback** (ex: +5 points).
- **Partagent des recommandations** (ex: +15 points).

Le `PointsSystem` permet de :
- **Attribuer des points** aux utilisateurs pour des actions spécifiques.
- **Gérer le solde de points** de chaque utilisateur.
- **Définir des règles de points personnalisables** (ex: +20 points pour une action spécifique).
- **Retirer des points** (ex: pour échanger contre des récompenses).

---

## Exigences Fonctionnelles
- **FR-50**: Attribuer des points aux utilisateurs pour des actions spécifiques.
- **FR-51**: Gérer le solde de points de chaque utilisateur.
- **FR-52**: Définir des règles de points personnalisables.
- **FR-53**: Permettre le retrait de points (ex: pour échanger contre des récompenses).

---

## Critères d'Acceptation
1. **Attribution de points** :
   - [x] Ajouter des points à un utilisateur pour une action (ex: `FOLLOW_RECOMMENDATION`).
   - [x] Utiliser des règles de points par défaut (ex: +10 points pour `FOLLOW_RECOMMENDATION`).
   - [x] Permettre des points personnalisés pour une action.

2. **Gestion du solde** :
   - [x] Récupérer le solde de points d'un utilisateur.
   - [x] Réinitialiser le solde de points d'un utilisateur.

3. **Règles de points** :
   - [x] Définir une règle de points pour une action.
   - [x] Récupérer la règle de points pour une action.
   - [x] Récupérer toutes les règles de points.

4. **Retrait de points** :
   - [x] Retirer des points à un utilisateur.
   - [x] Vérifier que l'utilisateur a assez de points avant de retirer.

5. **Historique des points** :
   - [x] Enregistrer l'historique des transactions de points (ajout/retrait).
   - [x] Récupérer l'historique des points pour un utilisateur.

6. **Classement** :
   - [x] Générer un classement des utilisateurs par nombre de points.

7. **Intégration avec le RecommendationEngine** :
   - [x] Le `RecommendationEngine` utilise le `PointsSystem` pour attribuer des points.

---

## Implémentation Technique

### Fichiers créés/modifiés
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/points_system.py` | Module `PointsSystem` | ✅ Créé |
| `backend/engines/recommendation_engine/__init__.py` | Intégration du `PointsSystem` | ✅ Modifié |
| `tests/unit/test_points_system.py` | Tests unitaires | ✅ Créé |

### Enum `PointsAction`
Définition des actions qui rapportent des points :

```python
from enum import Enum

class PointsAction(Enum):
    FOLLOW_RECOMMENDATION = "follow_recommendation"  # +10 points
    COMPLETE_CHALLENGE = "complete_challenge"        # +50 points
    GIVE_FEEDBACK = "give_feedback"                  # +5 points
    SHARE_RECOMMENDATION = "share_recommendation"    # +15 points
    INVITE_FRIEND = "invite_friend"                  # +25 points
    LOGIN = "login"                                  # +2 points
```

### Module `PointsSystem`
Ce module gère les points des utilisateurs :

```python
from typing import Dict, List, Optional
from enum import Enum
from datetime import datetime
import uuid

class PointsSystem:
    def __init__(self):
        # Règles de points par défaut
        self._points_rules = {
            PointsAction.FOLLOW_RECOMMENDATION: 10,
            PointsAction.COMPLETE_CHALLENGE: 50,
            PointsAction.GIVE_FEEDBACK: 5,
            PointsAction.SHARE_RECOMMENDATION: 15,
            PointsAction.INVITE_FRIEND: 25,
            PointsAction.LOGIN: 2
        }
        # Solde de points par utilisateur
        self._user_points: Dict[str, int] = {}
        # Historique des transactions
        self._points_history: Dict[str, List[Dict]] = {}

    def add_points(self, user_id: str, action: PointsAction, custom_points: Optional[int] = None) -> int:
        """Ajoute des points à un utilisateur pour une action."""
        points = custom_points if custom_points is not None else self._points_rules.get(action, 0)
        if user_id not in self._user_points:
            self._user_points[user_id] = 0
        self._user_points[user_id] += points
        # Enregistrer dans l'historique
        self._points_history.setdefault(user_id, []).append({
            "action": action,
            "points": points,
            "timestamp": datetime.now()
        })
        return points

    def subtract_points(self, user_id: str, points: int) -> bool:
        """Retire des points à un utilisateur."""
        if user_id not in self._user_points or self._user_points[user_id] < points:
            return False
        self._user_points[user_id] -= points
        self._points_history.setdefault(user_id, []).append({
            "action": "subtract",
            "points": -points,
            "timestamp": datetime.now()
        })
        return True

    def get_points_balance(self, user_id: str) -> int:
        """Récupère le solde de points d'un utilisateur."""
        return self._user_points.get(user_id, 0)

    def get_points_history(self, user_id: str, limit: Optional[int] = None) -> List[Dict]:
        """Récupère l'historique des transactions de points."""
        history = self._points_history.get(user_id, [])
        return history[-limit:] if limit else history

    def set_points_rule(self, action: PointsAction, points: int) -> None:
        """Définit une règle de points pour une action."""
        self._points_rules[action] = points

    def get_points_rule(self, action: PointsAction) -> int:
        """Récupère la règle de points pour une action."""
        return self._points_rules.get(action, 0)

    def get_leaderboard(self, limit: int = 10) -> List[Dict]:
        """Récupère le classement des utilisateurs par nombre de points."""
        sorted_users = sorted(self._user_points.items(), key=lambda x: x[1], reverse=True)
        return [{"user_id": user_id, "points": points} for user_id, points in sorted_users[:limit]]

    def reset_user_points(self, user_id: str) -> bool:
        """Réinitialise le solde de points d'un utilisateur."""
        if user_id not in self._user_points:
            return False
        self._user_points[user_id] = 0
        return True

    def clear_all_points(self) -> None:
        """Supprime tous les points et l'historique."""
        self._user_points.clear()
        self._points_history.clear()
```

---

## Tests Unitaires
20 tests ont été créés dans `tests/unit/test_points_system.py` pour valider :
- L'ajout de points pour différentes actions.
- Le retrait de points.
- La récupération du solde et de l'historique.
- La définition et la récupération des règles de points.
- Le classement des utilisateurs.
- Le déterminisme du module.

---

## Configuration Requise
- **Dépendances** : `uuid` pour générer des IDs uniques, `datetime` pour les timestamps.
- **Stockage** : En mémoire pour la phase PoC (à remplacer par une base de données pour la phase GA).

---

## Dépendances
- **Story 5-1** : `créer-et-mettre-à-jour-le-profil-utilisateur` (pour l'identification des utilisateurs).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine.points_system import points_system, PointsAction

# Ajouter des points pour une action
points = points_system.add_points(
    user_id="user_001",
    action=PointsAction.FOLLOW_RECOMMENDATION
)
print(f"Points ajoutés : {points}")

# Récupérer le solde de points
balance = points_system.get_points_balance("user_001")
print(f"Solde : {balance}")

# Ajouter des points avec une valeur personnalisée
points_system.add_points(
    user_id="user_001",
    action=PointsAction.COMPLETE_CHALLENGE,
    custom_points=100
)

# Récupérer le classement
leaderboard = points_system.get_leaderboard(limit=5)
print(f"Classement : {leaderboard}")

# Retirer des points
success = points_system.subtract_points("user_001", 50)
print(f"Retrait réussi : {success}")

# Récupérer l'historique des points
history = points_system.get_points_history("user_001")
print(f"Historique : {history}")

# Définir une règle de points personnalisée
points_system.set_points_rule(PointsAction.FOLLOW_RECOMMENDATION, 20)

# Récupérer une règle de points
rule = points_system.get_points_rule(PointsAction.FOLLOW_RECOMMENDATION)
print(f"Règle pour FOLLOW_RECOMMENDATION : {rule}")
```

---

## Notes
- **Stockage en mémoire** : Les points sont stockés en mémoire pour la phase PoC.
- **Règles personnalisables** : Les règles de points peuvent être modifiées dynamiquement.
- **Historique des transactions** : Toutes les transactions (ajout/retrait) sont enregistrées.
- **Classement** : Permet de générer un classement des utilisateurs pour la gamification.
- **Déterminisme** : Le module est déterministe, ce qui facilite les tests.

---

## Ressources
- [PRD Almanéa - FR-50](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-8](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
