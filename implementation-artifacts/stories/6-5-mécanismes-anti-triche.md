---
name: Mécanismes anti-triche
id: 6-5-mécanismes-anti-triche
story_type: backend
epic: epic-6
priority: high
estimation: M
dependencies: [6-4-débloquer-des-badges, 6-3-journal-écologique, 6-2-suivi-des-progrès, 6-1-système-de-points]
status: done
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 6.5: Mécanismes anti-triche

## Contexte
Cette story fait partie de **l'Épic 6 (Gamification)**. Son objectif est de **détecter et empêcher les comportements frauduleux** dans l'application Almanéa, afin de garantir l'intégrité du système de points, de badges et de récompenses. Le `AntiCheatSystem` permet de :
- **Détecter les actions suspectes** (ex: doublons, motifs répétitifs).
- **Limiter le nombre d'actions par période** (rate limiting).
- **Vérifier la cohérence des données** (ex: distance parcourue vs. temps).
- **Bloquer les utilisateurs** en cas de triche répétée.
- **Notifier les administrateurs** lorsqu'une triche est détectée.

---

## Exigences Fonctionnelles
- **FR-66**: Détecter les actions en double.
- **FR-67**: Limiter le nombre d'actions par période (rate limiting).
- **FR-68**: Vérifier la cohérence des données (ex: vitesse irréaliste).
- **FR-69**: Bloquer les utilisateurs en cas de triche répétée.
- **FR-70**: Notifier les administrateurs lorsqu'une triche est détectée.

---

## Critères d'Acceptation
1. **Détection des actions en double** :
   - [x] Détecter si une action identique est enregistrée dans un court laps de temps.
   - [x] Empêcher l'attribution de points ou de badges pour les actions en double.

2. **Rate Limiting** :
   - [x] Limiter le nombre d'actions par minute/heure/jour pour chaque catégorie.
   - [x] Bloquer les actions excédant les limites.

3. **Vérification de la cohérence des données** :
   - [x] Détecter les valeurs impossibles (ex: vitesse irréaliste).
   - [x] Détecter les valeurs négatives.

4. **Détection de motifs suspects** :
   - [x] Détecter les motifs répétitifs (ex: 3 actions identiques en 1 minute).

5. **Gestion des utilisateurs bloqués** :
   - [x] Bloquer un utilisateur après 3 actions suspectes.
   - [x] Déblocage automatique après une durée définie.

6. **Notifications** :
   - [x] Notifier les administrateurs lorsqu'une triche est détectée.

7. **Intégration avec le RecommendationEngine** :
   - [x] Le `RecommendationEngine` utilise le `AntiCheatSystem` pour vérifier les actions.

---

## Implémentation Technique

### Fichiers créés/modifiés
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/anti_cheat_system.py` | Module `AntiCheatSystem` | ✅ Créé |
| `backend/engines/recommendation_engine/__init__.py` | Intégration du `AntiCheatSystem` | ✅ Modifié |
| `tests/unit/test_anti_cheat_system.py` | Tests unitaires | ✅ Créé |

### Enum `CheatDetectionType` et `ActionCategory`
Définition des types de détection de triche et des catégories d'actions :

```python
from enum import Enum

class CheatDetectionType(Enum):
    RATE_LIMIT_EXCEEDED = "rate_limit_exceeded"
    DUPLICATE_ACTION = "duplicate_action"
    INCONSISTENT_DATA = "inconsistent_data"
    SUSPICIOUS_PATTERN = "suspicious_pattern"
    IMPOSSIBLE_VALUE = "impossible_value"

class ActionCategory(Enum):
    TRANSPORT = "transport"
    ENERGY = "energy"
    WATER = "water"
    WASTE = "waste"
    FOOD = "food"
    NATURE = "nature"
    GENERAL = "general"
```

### Module `AntiCheatSystem`
Ce module gère la détection et la prévention de la triche :

```python
from typing import Dict, List, Optional, Callable
from datetime import datetime, timedelta
from dataclasses import dataclass, field
from enum import Enum

class AntiCheatSystem:
    # Limites par défaut (actions par période)
    DEFAULT_RATE_LIMITS = {
        ActionCategory.TRANSPORT: {"per_minute": 5, "per_hour": 20, "per_day": 100},
        ActionCategory.ENERGY: {"per_minute": 3, "per_hour": 10, "per_day": 50},
        ActionCategory.WATER: {"per_minute": 10, "per_hour": 50, "per_day": 200},
        ActionCategory.WASTE: {"per_minute": 5, "per_hour": 20, "per_day": 100},
        ActionCategory.FOOD: {"per_minute": 3, "per_hour": 10, "per_day": 50},
        ActionCategory.NATURE: {"per_minute": 2, "per_hour": 5, "per_day": 20},
        ActionCategory.GENERAL: {"per_minute": 10, "per_hour": 50, "per_day": 200}
    }

    # Seuil pour la détection de motifs suspects
    SUSPICIOUS_PATTERN_THRESHOLD = 3

    # Durée de blocage en minutes
    BLOCK_DURATION_MINUTES = 30

    # Vitesse maximale réaliste (en km/h)
    MAX_REALISTIC_SPEED_KMH = 60

    def __init__(self):
        self._action_history: Dict[str, List[ActionRecord]] = {}
        self._rate_limits: Dict[str, Dict[ActionCategory, Dict[str, int]]] = {}
        self._blocked_users: Dict[str, datetime] = {}
        self._notification_callbacks: List[Callable] = {}
        self._suspicious_actions: Dict[str, List[CheatDetectionResult]] = {}

    def check_action(self, user_id: str, action_type: str, category: ActionCategory, metadata: Dict) -> CheatDetectionResult:
        """Vérifie si une action est suspecte ou frauduleuse."""
        # 1. Vérifier si l'utilisateur est bloqué
        if self.is_user_blocked(user_id):
            return CheatDetectionResult(is_cheating=True, detection_type=CheatDetectionType.RATE_LIMIT_EXCEEDED)

        # 2. Vérifier le rate limiting
        rate_limit_result = self._check_rate_limit(user_id, category)
        if rate_limit_result.is_cheating:
            return rate_limit_result

        # 3. Vérifier les doublons
        duplicate_result = self._check_duplicate_action(user_id, action_type, metadata)
        if duplicate_result.is_cheating:
            return duplicate_result

        # 4. Vérifier la cohérence des données
        consistency_result = self._check_data_consistency(action_type, metadata)
        if consistency_result.is_cheating:
            return consistency_result

        # 5. Vérifier les motifs suspects
        pattern_result = self._check_suspicious_pattern(user_id, action_type, category)
        if pattern_result.is_cheating:
            return pattern_result

        # Si tout est OK, enregistrer l'action
        self._record_action(user_id, action_type, category, metadata)
        self._update_rate_limit(user_id, category, "per_minute")
        return CheatDetectionResult(is_cheating=False, message="Action valide.")

    def is_user_blocked(self, user_id: str) -> bool:
        """Vérifie si un utilisateur est bloqué."""
        if user_id not in self._blocked_users:
            return False
        if datetime.now() >= self._blocked_users[user_id]:
            del self._blocked_users[user_id]
            return False
        return True

    def block_user(self, user_id: str, duration_minutes: int = None) -> None:
        """Bloque un utilisateur pour une durée donnée."""
        duration = duration_minutes if duration_minutes is not None else self.BLOCK_DURATION_MINUTES
        self._blocked_users[user_id] = datetime.now() + timedelta(minutes=duration)

    def record_suspicious_action(self, user_id: str, result: CheatDetectionResult) -> None:
        """Enregistre une action suspecte et bloque l'utilisateur après 3 actions suspectes."""
        self._suspicious_actions.setdefault(user_id, []).append(result)
        if len(self._suspicious_actions[user_id]) >= 3:
            self.block_user(user_id)
            self._notify_cheat_detected(user_id, result)

    def register_notification_callback(self, callback: Callable) -> None:
        """Enregistre un callback pour les notifications de triche."""
        self._notification_callbacks.append(callback)

    def get_user_stats(self, user_id: str) -> Dict:
        """Récupère les statistiques anti-triche pour un utilisateur."""
        return {
            "total_actions": len(self._action_history.get(user_id, [])),
            "suspicious_actions": len(self._suspicious_actions.get(user_id, [])),
            "is_blocked": self.is_user_blocked(user_id)
        }

    def reset_user_data(self, user_id: str) -> None:
        """Réinitialise toutes les données d'un utilisateur."""
        if user_id in self._action_history:
            del self._action_history[user_id]
        if user_id in self._rate_limits:
            del self._rate_limits[user_id]
        if user_id in self._suspicious_actions:
            del self._suspicious_actions[user_id]
        if user_id in self._blocked_users:
            del self._blocked_users[user_id]

    def clear_all_data(self) -> None:
        """Efface toutes les données du système anti-triche."""
        self._action_history.clear()
        self._rate_limits.clear()
        self._suspicious_actions.clear()
        self._blocked_users.clear()
```

---

## Tests Unitaires
23 tests ont été créés dans `tests/unit/test_anti_cheat_system.py` pour valider :
- La vérification des actions valides.
- La détection de rate limiting (par minute, heure, jour).
- La détection des actions en double.
- La détection des valeurs impossibles (ex: vitesse irréaliste).
- La détection des valeurs négatives.
- La détection de motifs suspects.
- Le blocage et le déblocage des utilisateurs.
- L'enregistrement des actions suspectes.
- Les notifications de triche.
- La réinitialisation des données.
- Le déterminisme du module.

---

## Configuration Requise
- **Dépendances** : `datetime` pour les timestamps, `hashlib` pour le hachage des actions.
- **Stockage** : En mémoire pour la phase PoC (à remplacer par une base de données pour la phase GA).

---

## Dépendances
- **Story 6-4** : `débloquer-des-badges` (pour éviter l'attribution frauduleuse de badges).
- **Story 6-3** : `journal-écologique` (pour vérifier les actions écologiques).
- **Story 6-2** : `suivi-des-progrès` (pour suivre les métriques de progrès).
- **Story 6-1** : `système-de-points` (pour éviter l'attribution frauduleuse de points).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine.anti_cheat_system import (
    anti_cheat_system,
    CheatDetectionType,
    ActionCategory
)

# Vérifier une action valide
result = anti_cheat_system.check_action(
    user_id="user_001",
    action_type="bike_ride",
    category=ActionCategory.TRANSPORT,
    metadata={"distance_km": 10, "time_min": 30}
)
print(f"Action valide : {not result.is_cheating}")

# Vérifier une action en double
result = anti_cheat_system.check_action(
    user_id="user_001",
    action_type="bike_ride",
    category=ActionCategory.TRANSPORT,
    metadata={"distance_km": 10, "time_min": 30}
)
print(f"Action en double : {result.is_cheating and result.detection_type == CheatDetectionType.DUPLICATE_ACTION}")

# Vérifier une vitesse irréaliste
result = anti_cheat_system.check_action(
    user_id="user_001",
    action_type="bike_ride",
    category=ActionCategory.TRANSPORT,
    metadata={"distance_km": 100, "time_min": 1}  # 6000 km/h
)
print(f"Vitesse irréaliste : {result.is_cheating and result.detection_type == CheatDetectionType.IMPOSSIBLE_VALUE}")

# Bloquer un utilisateur
anti_cheat_system.block_user("user_001", duration_minutes=60)
print(f"Utilisateur bloqué : {anti_cheat_system.is_user_blocked('user_001')}")

# Enregistrer une action suspecte
anti_cheat_system.record_suspicious_action("user_001", result)

# Récupérer les statistiques
stats = anti_cheat_system.get_user_stats("user_001")
print(f"Statistiques : {stats}")

# Enregistrer un callback de notification
def notification_callback(user_id: str, result: CheatDetectionResult):
    print(f"Triche détectée pour {user_id} : {result.detection_type}")

anti_cheat_system.register_notification_callback(notification_callback)

# Réinitialiser les données d'un utilisateur
anti_cheat_system.reset_user_data("user_001")
```

---

## Notes
- **Stockage en mémoire** : Les données anti-triche sont stockées en mémoire pour la phase PoC.
- **Rate Limiting** : Les limites sont configurables par catégorie d'action.
- **Détection de motifs** : Un seuil de 3 actions identiques en 1 minute déclenche une alerte.
- **Blocage automatique** : Les utilisateurs sont bloqués après 3 actions suspectes.
- **Déblocage automatique** : Les utilisateurs sont déblocés après 30 minutes (configurable).
- **Notifications** : Permet d'enregistrer des callbacks pour notifier les administrateurs.
- **Déterminisme** : Le module est déterministe, ce qui facilite les tests.

---

## Ressources
- [PRD Almanéa - FR-66](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-8](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
