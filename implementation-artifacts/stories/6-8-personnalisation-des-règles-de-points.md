---
name: Personnalisation des règles de points
id: 6-8-personnalisation-des-règles-de-points
story_type: backend
epic: epic-6
priority: high
estimation: M
dependencies: [6-7-échange-de-points-contre-des-récompenses, 6-1-système-de-points]
status: backlog
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 6.8: Personnalisation des règles de points

## Contexte
Cette story fait partie de **l'Épic 6 (Gamification)**. Son objectif est de **permettre aux administrateurs de personnaliser les règles d'attribution des points** pour adapter le système de récompenses aux besoins spécifiques de leur collectivité ou de leur programme. Le `PointsRuleCustomizationSystem` permet de :
- **Modifier les règles de points** existantes (ex: changer le nombre de points pour une action).
- **Ajouter de nouvelles actions** avec des règles de points personnalisées.
- **Supprimer des actions** existantes.
- **Réinitialiser les règles** aux valeurs par défaut.
- **Appliquer des règles spécifiques** par collectivité ou par utilisateur.

---

## Exigences Fonctionnelles
- **FR-80**: Permettre aux administrateurs de modifier les règles de points existantes.
- **FR-81**: Permettre aux administrateurs d'ajouter de nouvelles actions avec des règles de points.
- **FR-82**: Permettre aux administrateurs de supprimer des actions existantes.
- **FR-83**: Réinitialiser les règles de points aux valeurs par défaut.
- **FR-84**: Appliquer des règles de points spécifiques par collectivité.

---

## Critères d'Acceptation
1. **Modification des règles** :
   - [ ] Modifier le nombre de points attribués pour une action existante.
   - [ ] Vérifier que les modifications sont appliquées immédiatement.

2. **Ajout de nouvelles actions** :
   - [ ] Ajouter une nouvelle action avec un nombre de points personnalisé.
   - [ ] Vérifier que la nouvelle action est disponible pour l'attribution de points.

3. **Suppression d'actions** :
   - [ ] Supprimer une action existante.
   - [ ] Vérifier que l'action supprimée n'est plus disponible.

4. **Réinitialisation des règles** :
   - [ ] Réinitialiser toutes les règles aux valeurs par défaut.
   - [ ] Réinitialiser les règles pour une action spécifique.

5. **Règles par collectivité** :
   - [ ] Définir des règles de points spécifiques pour une collectivité.
   - [ ] Récupérer les règles de points pour une collectivité.

6. **Intégration avec le RecommendationEngine** :
   - [ ] Le `RecommendationEngine` utilise le `PointsRuleCustomizationSystem` pour gérer les règles.

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/points_rule_customization_system.py` | Module `PointsRuleCustomizationSystem` | ⏳ À créer |
| `backend/engines/recommendation_engine/__init__.py` | Intégration du `PointsRuleCustomizationSystem` | ⏳ À modifier |
| `tests/unit/test_points_rule_customization_system.py` | Tests unitaires | ⏳ À créer |

### Module `PointsRuleCustomizationSystem`
Ce module gère la personnalisation des règles de points :

```python
from typing import Dict, Optional, List
from enum import Enum
from dataclasses import dataclass, field
from .points_system import PointsAction

@dataclass
class CustomPointsRule:
    """
    Règle de points personnalisée.
    
    Attributes:
        action: Action pour laquelle la règle s'applique.
        points: Nombre de points attribués.
        community_id: ID de la collectivité (optionnel, pour les règles spécifiques).
    """
    action: PointsAction
    points: int
    community_id: Optional[str] = None

class PointsRuleCustomizationSystem:
    def __init__(self, points_system):
        self.points_system = points_system
        # Règles personnalisées globales : {PointsAction: int}
        self._custom_rules: Dict[PointsAction, int] = {}
        # Règles personnalisées par collectivité : {community_id: {PointsAction: int}}
        self._community_rules: Dict[str, Dict[PointsAction, int]] = {}

    def set_rule(self, action: PointsAction, points: int, community_id: Optional[str] = None) -> bool:
        """Définit une règle de points personnalisée."""
        if community_id:
            if community_id not in self._community_rules:
                self._community_rules[community_id] = {}
            self._community_rules[community_id][action] = points
        else:
            self._custom_rules[action] = points
        return True

    def get_rule(self, action: PointsAction, community_id: Optional[str] = None) -> int:
        """Récupère le nombre de points pour une action."""
        if community_id and community_id in self._community_rules:
            if action in self._community_rules[community_id]:
                return self._community_rules[community_id][action]
        if action in self._custom_rules:
            return self._custom_rules[action]
        return self.points_system.get_points_rule(action)

    def add_custom_action(self, action_name: str, points: int, community_id: Optional[str] = None) -> bool:
        """Ajoute une nouvelle action personnalisée."""
        try:
            custom_action = PointsAction(action_name)
            return self.set_rule(custom_action, points, community_id)
        except ValueError:
            return False

    def remove_rule(self, action: PointsAction, community_id: Optional[str] = None) -> bool:
        """Supprime une règle personnalisée."""
        if community_id and community_id in self._community_rules:
            if action in self._community_rules[community_id]:
                del self._community_rules[community_id][action]
                return True
        elif action in self._custom_rules:
            del self._custom_rules[action]
            return True
        return False

    def reset_rule(self, action: PointsAction, community_id: Optional[str] = None) -> bool:
        """Réinitialise une règle aux valeurs par défaut."""
        return self.remove_rule(action, community_id)

    def reset_all_rules(self, community_id: Optional[str] = None) -> None:
        """Réinitialise toutes les règles aux valeurs par défaut."""
        if community_id:
            if community_id in self._community_rules:
                del self._community_rules[community_id]
        else:
            self._custom_rules.clear()

    def get_all_rules(self, community_id: Optional[str] = None) -> Dict[PointsAction, int]:
        """Récupère toutes les règles de points."""
        if community_id and community_id in self._community_rules:
            return self._community_rules[community_id]
        return {**self.points_system.get_all_points_rules(), **self._custom_rules}

    def apply_community_rules(self, community_id: str, rules: Dict[PointsAction, int]) -> bool:
        """Applique un ensemble de règles pour une collectivité."""
        if community_id not in self._community_rules:
            self._community_rules[community_id] = {}
        self._community_rules[community_id].update(rules)
        return True

    def clear_all_custom_rules(self) -> None:
        """Efface toutes les règles personnalisées."""
        self._custom_rules.clear()
        self._community_rules.clear()
```

---

## Tests Unitaires
Les tests suivants seront créés dans `tests/unit/test_points_rule_customization_system.py` :
- Définition et récupération de règles personnalisées.
- Ajout et suppression d'actions personnalisées.
- Réinitialisation des règles.
- Application de règles par collectivité.
- Vérification de l'intégration avec `PointsSystem`.

---

## Configuration Requise
- **Dépendances** : `enum` pour les actions, `dataclasses` pour les structures de données.
- **Intégration** : Dépend du `PointsSystem` pour les règles par défaut.

---

## Dépendances
- **Story 6-7** : `échange-de-points-contre-des-récompenses` (pour appliquer les règles personnalisées).
- **Story 6-1** : `système-de-points` (pour les règles de base).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine.points_rule_customization_system import (
    points_rule_customization_system,
    PointsAction
)

# Définir une règle personnalisée
points_rule_customization_system.set_rule(PointsAction.FOLLOW_RECOMMENDATION, 20)

# Récupérer une règle
points = points_rule_customization_system.get_rule(PointsAction.FOLLOW_RECOMMENDATION)
print(f"Points pour FOLLOW_RECOMMENDATION : {points}")

# Ajouter une nouvelle action
points_rule_customization_system.add_custom_action("custom_action_1", 100)

# Définir une règle pour une collectivité
points_rule_customization_system.set_rule(
    PointsAction.FOLLOW_RECOMMENDATION, 30, community_id="community_001"
)

# Récupérer les règles pour une collectivité
rules = points_rule_customization_system.get_all_rules(community_id="community_001")
print(f"Règles pour community_001 : {rules}")

# Réinitialiser une règle
points_rule_customization_system.reset_rule(PointsAction.FOLLOW_RECOMMENDATION)

# Réinitialiser toutes les règles
points_rule_customization_system.reset_all_rules()
```

---

## Notes
- **Règles globales vs. locales** : Les règles personnalisées globales s'appliquent à tous les utilisateurs, sauf si une règle spécifique à une collectivité est définie.
- **Priorité** : Les règles par collectivité ont la priorité sur les règles globales, qui ont elles-mêmes la priorité sur les règles par défaut.
- **Flexibilité** : Permet d'ajouter des actions personnalisées pour des cas d'usage spécifiques.
- **Réinitialisation** : Permet de revenir aux règles par défaut à tout moment.

---

## Ressources
- [PRD Almanéa - FR-80](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-8](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
