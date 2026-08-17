---
name: Donner un feedback sur les recommandations
id: 5-4-donner-un-feedback-sur-les-recommandations
epic: epic-5
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

# Story 5.4: Donner un feedback sur les recommandations

## Contexte
Cette story fait partie de **l'Épic 5 (Gestion du profil utilisateur)**. Son objectif est de **permettre aux utilisateurs de donner un feedback** sur les recommandations (ex: "J'ai aimé", "Je n'ai pas aimé", "C'était utile", etc.) pour :
- **Améliorer la pertinence** des futures recommandations.
- **Analyser les retours utilisateurs** (ex: quelles recommandations sont les plus appréciées ?).
- **Personnaliser les suggestions** en fonction des feedbacks.

Le `FeedbackManager` permet de :
- **Stocker les feedbacks** des utilisateurs sur les recommandations.
- **Récupérer les feedbacks** par utilisateur, recommandation ou type.
- **Calculer des statistiques** (ex: note moyenne d'une recommandation).

---

## Exigences Fonctionnelles
- **FR-40**: Permettre aux utilisateurs de donner un feedback sur une recommandation.
- **FR-41**: Stocker les feedbacks avec un type (ex: LIKED, DISLIKED, USEFUL).
- **FR-42**: Calculer des statistiques à partir des feedbacks (ex: note moyenne, nombre de feedbacks par type).

---

## Critères d'Acceptation
1. **Ajout de feedback** :
   - [x] Ajouter un feedback pour une recommandation avec un type (ex: `LIKED`, `DISLIKED`).
   - [x] Associer un commentaire et une note (1-5) optionnels au feedback.

2. **Récupération des feedbacks** :
   - [x] Récupérer un feedback par son ID.
   - [x] Récupérer tous les feedbacks d'un utilisateur.
   - [x] Récupérer tous les feedbacks pour une recommandation.
   - [x] Récupérer tous les feedbacks d'un type spécifique (ex: `LIKED`).

3. **Statistiques** :
   - [x] Calculer la note moyenne pour une recommandation.
   - [x] Générer des statistiques globales ou par utilisateur (ex: nombre de `LIKED`, `DISLIKED`).

4. **Gestion des feedbacks** :
   - [x] Mettre à jour un feedback (commentaire ou note).
   - [x] Supprimer un feedback.
   - [x] Supprimer tous les feedbacks d'un utilisateur.
   - [x] Supprimer tous les feedbacks.

5. **Intégration avec le RecommendationEngine** :
   - [x] Le `RecommendationEngine` utilise le `FeedbackManager` pour gérer les feedbacks.

---

## Implémentation Technique

### Fichiers créés/modifiés
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/feedback_manager.py` | Module `FeedbackManager` | ✅ Créé |
| `backend/engines/recommendation_engine/__init__.py` | Intégration du `FeedbackManager` | ✅ Modifié |
| `tests/unit/test_feedback_manager.py` | Tests unitaires | ✅ Créé |

### Modèle `Feedback`
Un modèle Pydantic pour représenter un feedback :

```python
from typing import Optional
from enum import Enum
from datetime import datetime
from pydantic import BaseModel, Field

class FeedbackType(Enum):
    LIKED = "liked"
    DISLIKED = "disliked"
    USEFUL = "useful"
    NOT_USEFUL = "not_useful"
    FOLLOWED = "followed"
    NOT_FOLLOWED = "not_followed"

class Feedback(BaseModel):
    id: str = Field(..., description="ID unique du feedback")
    user_id: str = Field(..., description="ID de l'utilisateur")
    recommendation_id: str = Field(..., description="ID de la recommandation")
    feedback_type: FeedbackType = Field(..., description="Type de feedback")
    comment: Optional[str] = Field(None, description="Commentaire optionnel")
    rating: Optional[int] = Field(None, description="Note (1-5)", ge=1, le=5)
    timestamp: datetime = Field(default_factory=datetime.now, description="Date et heure du feedback")
```

### Module `FeedbackManager`
Ce module gère les feedbacks en mémoire :

```python
from typing import Dict, List, Optional
from .models import Feedback, FeedbackType
import uuid

class FeedbackManager:
    def __init__(self):
        self._feedbacks: Dict[str, Feedback] = {}
        self._user_feedback_index: Dict[str, List[str]] = {}
        self._recommendation_feedback_index: Dict[str, List[str]] = {}
        self._feedback_type_index: Dict[FeedbackType, List[str]] = {}

    def add_feedback(
        self,
        user_id: str,
        recommendation_id: str,
        feedback_type: FeedbackType,
        comment: Optional[str] = None,
        rating: Optional[int] = None
    ) -> Feedback:
        """Ajoute un feedback pour une recommandation."""
        feedback_id = f"feedback_{uuid.uuid4().hex[:8]}"
        feedback = Feedback(
            id=feedback_id,
            user_id=user_id,
            recommendation_id=recommendation_id,
            feedback_type=feedback_type,
            comment=comment,
            rating=rating
        )
        self._feedbacks[feedback_id] = feedback
        # Mettre à jour les index
        self._user_feedback_index.setdefault(user_id, []).append(feedback_id)
        self._recommendation_feedback_index.setdefault(recommendation_id, []).append(feedback_id)
        self._feedback_type_index.setdefault(feedback_type, []).append(feedback_id)
        return feedback

    def get_feedback(self, feedback_id: str) -> Optional[Feedback]:
        """Récupère un feedback par son ID."""
        return self._feedbacks.get(feedback_id)

    def get_feedbacks_by_user(self, user_id: str) -> List[Feedback]:
        """Récupère tous les feedbacks d'un utilisateur."""
        return [self._feedbacks[fb_id] for fb_id in self._user_feedback_index.get(user_id, [])]

    def get_feedbacks_by_recommendation(self, recommendation_id: str) -> List[Feedback]:
        """Récupère tous les feedbacks pour une recommandation."""
        return [self._feedbacks[fb_id] for fb_id in self._recommendation_feedback_index.get(recommendation_id, [])]

    def get_feedbacks_by_type(self, feedback_type: FeedbackType) -> List[Feedback]:
        """Récupère tous les feedbacks d'un type spécifique."""
        return [self._feedbacks[fb_id] for fb_id in self._feedback_type_index.get(feedback_type, [])]

    def get_average_rating(self, recommendation_id: str) -> Optional[float]:
        """Calcule la note moyenne pour une recommandation."""
        feedbacks = self.get_feedbacks_by_recommendation(recommendation_id)
        ratings = [fb.rating for fb in feedbacks if fb.rating is not None]
        return sum(ratings) / len(ratings) if ratings else None

    def get_feedback_stats(self, user_id: Optional[str] = None) -> Dict[str, int]:
        """Récupère les statistiques des feedbacks."""
        feedbacks = self.get_feedbacks_by_user(user_id) if user_id else self.get_all_feedbacks()
        stats = {"total": len(feedbacks)}
        for feedback in feedbacks:
            stats[feedback.feedback_type.value] = stats.get(feedback.feedback_type.value, 0) + 1
        return stats

    def update_feedback(self, feedback_id: str, comment: Optional[str] = None, rating: Optional[int] = None) -> Optional[Feedback]:
        """Met à jour un feedback."""
        if feedback_id not in self._feedbacks:
            return None
        feedback = self._feedbacks[feedback_id]
        if comment is not None:
            feedback.comment = comment
        if rating is not None:
            feedback.rating = rating
        return feedback

    def delete_feedback(self, feedback_id: str) -> bool:
        """Supprime un feedback."""
        if feedback_id not in self._feedbacks:
            return False
        feedback = self._feedbacks[feedback_id]
        # Supprimer des index
        self._user_feedback_index[feedback.user_id].remove(feedback_id)
        self._recommendation_feedback_index[feedback.recommendation_id].remove(feedback_id)
        self._feedback_type_index[feedback.feedback_type].remove(feedback_id)
        del self._feedbacks[feedback_id]
        return True

    def clear_user_feedbacks(self, user_id: str) -> bool:
        """Supprime tous les feedbacks d'un utilisateur."""
        if user_id not in self._user_feedback_index:
            return False
        for fb_id in list(self._user_feedback_index[user_id]):
            self.delete_feedback(fb_id)
        return True

    def clear_all_feedbacks(self) -> None:
        """Supprime tous les feedbacks."""
        self._feedbacks.clear()
        self._user_feedback_index.clear()
        self._recommendation_feedback_index.clear()
        self._feedback_type_index.clear()
```

---

## Tests Unitaires
20 tests ont été créés dans `tests/unit/test_feedback_manager.py` pour valider :
- L'ajout de feedbacks.
- La récupération de feedbacks par ID, utilisateur, recommandation ou type.
- La mise à jour et la suppression de feedbacks.
- Le calcul de la note moyenne.
- La génération de statistiques.
- Le déterminisme du module.

---

## Configuration Requise
- **Dépendances** : `uuid` pour générer des IDs uniques, `pydantic` pour la validation.
- **Stockage** : En mémoire pour la phase PoC (à remplacer par une base de données pour la phase GA).

---

## Dépendances
- **Story 5-1** : `créer-et-mettre-à-jour-le-profil-utilisateur` (pour l'identification des utilisateurs).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine.feedback_manager import feedback_manager, FeedbackType

# Ajouter un feedback
feedback = feedback_manager.add_feedback(
    user_id="user_001",
    recommendation_id="rec_001",
    feedback_type=FeedbackType.LIKED,
    comment="J'ai aimé cette recommandation !",
    rating=5
)

# Récupérer un feedback
retrieved_feedback = feedback_manager.get_feedback(feedback.id)
print(f"Feedback : {retrieved_feedback.feedback_type.value}, Note : {retrieved_feedback.rating}")

# Récupérer les feedbacks d'un utilisateur
user_feedbacks = feedback_manager.get_feedbacks_by_user("user_001")
print(f"Nombre de feedbacks pour user_001 : {len(user_feedbacks)}")

# Récupérer les feedbacks pour une recommandation
rec_feedbacks = feedback_manager.get_feedbacks_by_recommendation("rec_001")
print(f"Feedbacks pour rec_001 : {len(rec_feedbacks)}")

# Calculer la note moyenne
average_rating = feedback_manager.get_average_rating("rec_001")
print(f"Note moyenne pour rec_001 : {average_rating}")

# Récupérer les statistiques
stats = feedback_manager.get_feedback_stats(user_id="user_001")
print(f"Statistiques : {stats}")

# Mettre à jour un feedback
updated_feedback = feedback_manager.update_feedback(
    feedback_id=feedback.id,
    comment="Nouveau commentaire",
    rating=4
)

# Supprimer un feedback
feedback_manager.delete_feedback(feedback.id)
```

---

## Notes
- **Stockage en mémoire** : Les feedbacks sont stockés en mémoire pour la phase PoC.
- **Indexation** : Les feedbacks sont indexés par utilisateur, recommandation et type pour une récupération efficace.
- **Statistiques** : Permet d'analyser les retours utilisateurs pour améliorer les recommandations.
- **Déterminisme** : Le module est déterministe, ce qui facilite les tests.

---

## Ressources
- [PRD Almanéa - FR-40](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-7](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
