---
name: Classer les recommandations
id: 2-3-classer-les-recommandations
epic: epic-2
story_type: backend
priority: high
estimation: S
dependencies: [2-1, 2-2]
status: ready-for-dev
created: 2026-08-16
updated: 2026-08-16
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/implementation-artifacts/epic-1-context.md
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 2.3: Classer les recommandations

## Contexte
Cette story fait partie de **l'Épic 2 (Recommendation Engine)**. Son objectif est de **classer les recommandations par score décroissant** pour que les recommandations les plus pertinentes soient affichées en premier, conformément à **FR-8 (Classement des recommandations par score décroissant)**.

Le classement est une étape clé pour garantir que l'utilisateur voit **les meilleures recommandations en premier**, tout en assurant une **diversité** dans les résultats.

---

## Exigences Fonctionnelles
- **FR-8**: Classement des recommandations par score décroissant.
- **NFR-1**: Déterminisme — Le classement doit être **reproductible** (mêmes entrées → même ordre).

---

## Critères d'Acceptation
1. **Classement par score** :
   - [ ] Les recommandations sont **triées par score décroissant** (du plus pertinent au moins pertinent).
   - [ ] Si deux recommandations ont le **même score**, elles sont **classées par catégorie** (ex: MOBILITY avant NATURE).

2. **Diversité des recommandations** :
   - [ ] Les recommandations **diverses** (ex: pas que du vélo) sont **priorisées** pour éviter une liste trop homogène.
   - [ ] Un **algorithme de diversité** est appliqué pour garantir un mélange de catégories.

3. **Stabilité du classement** :
   - [ ] Le classement est **déterministe** (mêmes entrées → même ordre).
   - [ ] Les tests vérifient que l'ordre est **reproductible**.

4. **Intégration avec le Recommendation Engine** :
   - [ ] Le classement est **intégré avec le `RuleEngine` et le `ScoringEngine`**.
   - [ ] Le classement est **appliqué automatiquement** après le scoring.

5. **Gestion des ex-aequo** :
   - [ ] Si deux recommandations ont le **même score**, elles sont **classées par** :
     1. **Catégorie** (ordre prédéfini : MOBILITY > NATURE > ENERGY > WATER > REPAIR > WELLBEING).
     2. **ID de recommandation** (ordre alphabétique).

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/ranking.py` | Module `RankingEngine` | À créer |
| `backend/engines/recommendation_engine/__init__.py` | Export du `RankingEngine` | À modifier |
| `tests/unit/test_ranking.py` | Tests unitaires | À créer |

### Structure du `RankingEngine` (`ranking.py`)
```python
from typing import List, Dict
from enum import Enum
from .models import Recommendation, RecommendationCategory


class CategoryPriority(Enum):
    """Priorité des catégories pour le classement."""
    MOBILITY = 1
    NATURE = 2
    ENERGY = 3
    WATER = 4
    REPAIR = 5
    WELLBEING = 6


class RankingEngine:
    """
    Moteur de classement pour trier les recommandations.
    - Trie par score décroissant.
    - Applique un algorithme de diversité pour éviter les doublons de catégorie.
    """

    # Priorité des catégories (pour le classement des ex-aequo)
    CATEGORY_PRIORITY = {
        RecommendationCategory.MOBILITY: 1,
        RecommendationCategory.NATURE: 2,
        RecommendationCategory.ENERGY: 3,
        RecommendationCategory.WATER: 4,
        RecommendationCategory.REPAIR: 5,
        RecommendationCategory.WELLBEING: 6
    }

    def __init__(self, enable_diversity: bool = True):
        """
        Initialise le RankingEngine.
        
        Args:
            enable_diversity: Si True, applique un algorithme de diversité pour éviter les doublons de catégorie.
        """
        self.enable_diversity = enable_diversity

    def rank_recommendations(self, recommendations: List[Recommendation]) -> List[Recommendation]:
        """
        Classe une liste de recommandations par score décroissant.
        
        Args:
            recommendations: Liste de recommandations à classer.
        
        Returns:
            List[Recommendation]: Liste de recommandations classées.
        """
        if not recommendations:
            return []

        # Trier par score décroissant, puis par catégorie, puis par ID
        sorted_recommendations = sorted(
            recommendations,
            key=lambda rec: (
                -rec.score if rec.score is not None else 0,  # Score décroissant
                self.CATEGORY_PRIORITY.get(rec.category, 999),  # Catégorie (priorité)
                rec.id  # ID (ordre alphabétique)
            )
        )

        # Appliquer l'algorithme de diversité si activé
        if self.enable_diversity:
            sorted_recommendations = self._apply_diversity(sorted_recommendations)

        return sorted_recommendations

    def _apply_diversity(self, recommendations: List[Recommendation]) -> List[Recommendation]:
        """
        Applique un algorithme de diversité pour éviter les doublons de catégorie.
        
        Exemple : Si les 3 premières recommandations sont toutes de la catégorie MOBILITY,
        on peut insérer une recommandation d'une autre catégorie entre elles.
        
        Args:
            recommendations: Liste de recommandations déjà triées par score.
        
        Returns:
            List[Recommendation]: Liste avec une meilleure diversité de catégories.
        """
        if len(recommendations) <= 1:
            return recommendations

        ranked = []
        used_categories = set()

        for rec in recommendations:
            if rec.category not in used_categories:
                ranked.append(rec)
                used_categories.add(rec.category)
            else:
                # Trouver la position optimale pour insérer cette recommandation
                # sans créer de doublon de catégorie consécutif
                inserted = False
                for i in range(len(ranked)):
                    if ranked[i].category != rec.category:
                        # Vérifier que l'insertion ne crée pas de doublon avec le suivant
                        if i + 1 < len(ranked):
                            if ranked[i + 1].category != rec.category:
                                ranked.insert(i + 1, rec)
                                inserted = True
                                break
                        else:
                            ranked.append(rec)
                            inserted = True
                            break
                if not inserted:
                    ranked.append(rec)

        return ranked

    def get_top_n(self, recommendations: List[Recommendation], n: int = 5) -> List[Recommendation]:
        """
        Retourne les N meilleures recommandations.
        
        Args:
            recommendations: Liste de recommandations à classer.
            n: Nombre de recommandations à retourner.
        
        Returns:
            List[Recommendation]: Top N recommandations classées.
        """
        ranked = self.rank_recommendations(recommendations)
        return ranked[:n]
```

### Intégration avec le Recommendation Engine
Dans `backend/engines/recommendation_engine/__init__.py` :
```python
from .rules import rule_engine
from .models import Recommendation
from .scoring import ScoringEngine
from .ranking import RankingEngine

# Créer une instance globale du RecommendationEngine
class RecommendationEngine:
    """
    Moteur principal pour générer, scorer et classer les recommandations.
    """

    def __init__(self):
        self.rule_engine = rule_engine
        self.scoring_engine = ScoringEngine()
        self.ranking_engine = RankingEngine(enable_diversity=True)

    def generate_recommendations(
        self,
        context: UnifiedContext,
        user_profile: Optional[UserProfile] = None
    ) -> List[Recommendation]:
        """
        Génère, score et classe les recommandations.
        
        Args:
            context: Le contexte unifié.
            user_profile: Le profil utilisateur (optionnel).
        
        Returns:
            List[Recommendation]: Liste de recommandations générées, scorées et classées.
        """
        # 1. Générer les candidats avec le RuleEngine
        candidates = self.rule_engine.evaluate_all(context, user_profile)

        # 2. Calculer les scores avec le ScoringEngine
        scored_candidates = self.scoring_engine.calculate_scores(candidates, context, user_profile)

        # 3. Classer les recommandations avec le RankingEngine
        ranked_recommendations = self.ranking_engine.rank_recommendations(scored_candidates)

        return ranked_recommendations


# Instance globale
recommendation_engine = RecommendationEngine()
```

---

## Tests Unitaires
Créer un fichier `tests/unit/test_ranking.py` avec les tests suivants :

```python
import pytest
from datetime import datetime, timedelta
from unittest.mock import AsyncMock, patch, MagicMock
import sys
import os

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '../../backend'))

from backend.engines.recommendation_engine.models import Recommendation, RecommendationCategory
from backend.engines.recommendation_engine.ranking import RankingEngine


@pytest.fixture
def ranking_engine():
    return RankingEngine(enable_diversity=False)


@pytest.fixture
def mock_recommendations():
    return [
        Recommendation(
            id="rec_001",
            title="Balade à vélo",
            description="Profitez de la bonne qualité de l'air.",
            category=RecommendationCategory.MOBILITY,
            action="Faire du vélo",
            score=0.95
        ),
        Recommendation(
            id="rec_002",
            title="Promenade à pied",
            description="Une marche est idéale.",
            category=RecommendationCategory.MOBILITY,
            action="Marcher",
            score=0.90
        ),
        Recommendation(
            id="rec_003",
            title="Balade en forêt",
            description="Explorez les sentiers locaux.",
            category=RecommendationCategory.NATURE,
            action="Balade en forêt",
            score=0.85
        ),
        Recommendation(
            id="rec_004",
            title="Réduire la consommation",
            description="Éteignez les appareils inutiles.",
            category=RecommendationCategory.ENERGY,
            action="Réduire la consommation",
            score=0.80
        ),
    ]


# Test de classement par score décroissant
def test_rank_by_score(ranking_engine, mock_recommendations):
    ranked = ranking_engine.rank_recommendations(mock_recommendations)
    assert len(ranked) == 4
    assert ranked[0].id == "rec_001"  # Score 0.95
    assert ranked[1].id == "rec_002"  # Score 0.90
    assert ranked[2].id == "rec_003"  # Score 0.85
    assert ranked[3].id == "rec_004"  # Score 0.80


# Test de classement avec même score (ex-aequo)
def test_rank_with_tie(ranking_engine):
    recommendations = [
        Recommendation(
            id="rec_001",
            title="Balade à vélo",
            description="Profitez de la bonne qualité de l'air.",
            category=RecommendationCategory.MOBILITY,
            action="Faire du vélo",
            score=0.90
        ),
        Recommendation(
            id="rec_002",
            title="Balade en forêt",
            description="Explorez les sentiers locaux.",
            category=RecommendationCategory.NATURE,
            action="Balade en forêt",
            score=0.90
        ),
        Recommendation(
            id="rec_003",
            title="Réduire la consommation",
            description="Éteignez les appareils inutiles.",
            category=RecommendationCategory.ENERGY,
            action="Réduire la consommation",
            score=0.90
        ),
    ]
    ranked = ranking_engine.rank_recommendations(recommendations)
    # MOBILITY (1) > NATURE (2) > ENERGY (3)
    assert ranked[0].id == "rec_001"
    assert ranked[1].id == "rec_002"
    assert ranked[2].id == "rec_003"


# Test de classement avec même score et même catégorie
def test_rank_with_tie_same_category(ranking_engine):
    recommendations = [
        Recommendation(
            id="rec_001",
            title="Balade à vélo",
            description="Profitez de la bonne qualité de l'air.",
            category=RecommendationCategory.MOBILITY,
            action="Faire du vélo",
            score=0.90
        ),
        Recommendation(
            id="rec_002",
            title="Promenade à pied",
            description="Une marche est idéale.",
            category=RecommendationCategory.MOBILITY,
            action="Marcher",
            score=0.90
        ),
    ]
    ranked = ranking_engine.rank_recommendations(recommendations)
    # Même catégorie, classement par ID
    assert ranked[0].id == "rec_001"
    assert ranked[1].id == "rec_002"


# Test de classement avec diversité activée
def test_rank_with_diversity():
    ranking_engine = RankingEngine(enable_diversity=True)
    recommendations = [
        Recommendation(
            id="rec_001",
            title="Balade à vélo",
            description="Profitez de la bonne qualité de l'air.",
            category=RecommendationCategory.MOBILITY,
            action="Faire du vélo",
            score=0.95
        ),
        Recommendation(
            id="rec_002",
            title="Promenade à pied",
            description="Une marche est idéale.",
            category=RecommendationCategory.MOBILITY,
            action="Marcher",
            score=0.90
        ),
        Recommendation(
            id="rec_003",
            title="Balade en forêt",
            description="Explorez les sentiers locaux.",
            category=RecommendationCategory.NATURE,
            action="Balade en forêt",
            score=0.85
        ),
        Recommendation(
            id="rec_004",
            title="Réduire la consommation",
            description="Éteignez les appareils inutiles.",
            category=RecommendationCategory.ENERGY,
            action="Réduire la consommation",
            score=0.80
        ),
    ]
    ranked = ranking_engine.rank_recommendations(recommendations)
    # Avec diversité, on devrait avoir une alternance de catégories
    assert len(ranked) == 4
    # Vérifier que les scores sont toujours décroissants
    for i in range(len(ranked) - 1):
        assert ranked[i].score >= ranked[i + 1].score


# Test de classement avec diversité et ex-aequo
def test_rank_with_diversity_and_tie():
    ranking_engine = RankingEngine(enable_diversity=True)
    recommendations = [
        Recommendation(
            id="rec_001",
            title="Balade à vélo",
            description="Profitez de la bonne qualité de l'air.",
            category=RecommendationCategory.MOBILITY,
            action="Faire du vélo",
            score=0.90
        ),
        Recommendation(
            id="rec_002",
            title="Promenade à pied",
            description="Une marche est idéale.",
            category=RecommendationCategory.MOBILITY,
            action="Marcher",
            score=0.90
        ),
        Recommendation(
            id="rec_003",
            title="Balade en forêt",
            description="Explorez les sentiers locaux.",
            category=RecommendationCategory.NATURE,
            action="Balade en forêt",
            score=0.90
        ),
    ]
    ranked = ranking_engine.rank_recommendations(recommendations)
    # Avec diversité, on devrait avoir au moins une catégorie différente dans les 2 premières
    categories = [rec.category for rec in ranked[:2]]
    assert len(set(categories)) > 1


# Test de top N
def test_get_top_n(ranking_engine, mock_recommendations):
    top_2 = ranking_engine.get_top_n(mock_recommendations, 2)
    assert len(top_2) == 2
    assert top_2[0].id == "rec_001"
    assert top_2[1].id == "rec_002"


# Test de top N avec N > len(recommendations)
def test_get_top_n_larger_than_list(ranking_engine, mock_recommendations):
    top_10 = ranking_engine.get_top_n(mock_recommendations, 10)
    assert len(top_10) == 4  # Toutes les recommandations


# Test de classement vide
def test_rank_empty_list(ranking_engine):
    ranked = ranking_engine.rank_recommendations([])
    assert ranked == []


# Test de déterminisme
def test_deterministic_ranking(ranking_engine, mock_recommendations):
    ranked1 = ranking_engine.rank_recommendations(mock_recommendations)
    ranked2 = ranking_engine.rank_recommendations(mock_recommendations)
    assert len(ranked1) == len(ranked2)
    for rec1, rec2 in zip(ranked1, ranked2):
        assert rec1.id == rec2.id
```

---

## Configuration Requise
Aucune configuration supplémentaire nécessaire.

---

## Dépendances
- **Python 3.11+**
- **Librairies** :
  - Aucune dépendance supplémentaire.

---

## Notes
- **Classement par score** : Les recommandations sont triées par score décroissant.
- **Ex-aequo** : En cas d'égalité de score, les recommandations sont classées par catégorie (priorité prédéfini) puis par ID.
- **Diversité** : L'algorithme de diversité évite les doublons de catégorie consécutifs pour une meilleure expérience utilisateur.
- **Déterminisme** : Le classement est 100% déterministe (mêmes entrées → même ordre).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine import recommendation_engine
from backend.engines.context_engine.models import UnifiedContext, UserProfile

# Générer, scorer et classer les recommandations
context = UnifiedContext(...)  # Contexte unifié
user_profile = UserProfile(...)  # Profil utilisateur

recommendations = recommendation_engine.generate_recommendations(context, user_profile)

# Afficher les 5 meilleures recommandations
for rec in recommendations[:5]:
    print(f"{rec.title} (Score: {rec.score:.2f}, Catégorie: {rec.category.value})")
```

---

## Ressources
- [PRD Almanéa - FR-8](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-5](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
