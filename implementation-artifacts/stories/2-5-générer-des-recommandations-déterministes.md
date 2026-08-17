---
name: Générer des recommandations déterministes
id: 2-5-générer-des-recommandations-déterministes
epic: epic-2
story_type: backend
priority: high
estimation: S
dependencies: [2-1, 2-2, 2-4]
status: ready-for-dev
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/implementation-artifacts/epic-1-context.md
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 2.5: Générer des recommandations déterministes

## Contexte
Cette story fait partie de **l'Épic 2 (Recommendation Engine)**. Son objectif est de **valider que les recommandations sont générées de manière déterministe (sans LLM)**, conformément à **FR-10 (Génération de recommandations déterministes)**. Cela garantit que les **tests unitaires** peuvent être exécutés sans dépendre d'un LLM, et que les mêmes entrées produisent toujours les mêmes sorties.

Le **Recommendation Engine** doit être **100% déterministe** pour les fonctionnalités core (génération, scoring, filtrage, classement). Les LLMs ne sont utilisés que pour des fonctionnalités optionnelles comme la génération d'explications textuelles (voir **Épic 4**).

---

## Exigences Fonctionnelles
- **FR-10**: Génération de recommandations déterministes (sans LLM pour le core).
- **NFR-1**: Déterminisme — Les recommandations doivent être reproductibles.

---

## Critères d'Acceptation
1. **Déterminisme du Recommendation Engine** :
   - [ ] Les mêmes entrées (`UnifiedContext`, `UserProfile`) produisent **les mêmes recommandations** (mêmes IDs, titres, scores, ordre).
   - [ ] Les tests unitaires du **Recommendation Engine** fonctionnent **sans LLM**.

2. **Validation des composants** :
   - [ ] Le `RuleEngine` est **déterministe** (mêmes règles → mêmes candidats).
   - [ ] Le `ScoringEngine` est **déterministe** (mêmes candidats + contexte → mêmes scores).
   - [ ] Le `FilterEngine` est **déterministe** (mêmes candidats + contexte → mêmes filtrages).
   - [ ] Le `RankingEngine` est **déterministe** (mêmes candidats scorés → même ordre).

3. **Tests automatiques** :
   - [ ] Un test valide que le `RecommendationEngine.generate_recommendations()` retourne les **mêmes résultats** pour les mêmes entrées.
   - [ ] Un test valide que chaque composant (`RuleEngine`, `ScoringEngine`, `FilterEngine`, `RankingEngine`) est **déterministe**.

4. **Documentation** :
   - [ ] La documentation explique clairement que le **core du Recommendation Engine est déterministe**.
   - [ ] Les dépendances au LLM (pour les explications textuelles) sont **isolées** dans l'**Épic 4**.

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `tests/unit/test_recommendation_engine_determinism.py` | Tests de déterminisme | À créer |

### Tests de déterminisme (`test_recommendation_engine_determinism.py`)
```python
import pytest
from datetime import datetime, timedelta
from unittest.mock import AsyncMock, patch, MagicMock
import sys
import os

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '../../backend'))

from backend.engines.recommendation_engine import recommendation_engine
from backend.engines.recommendation_engine.models import Recommendation, RecommendationCategory, UnifiedContext, UserProfile, Observation
from backend.engines.recommendation_engine.rules.rule_engine import RuleEngine
from backend.engines.recommendation_engine.scoring import ScoringEngine
from backend.engines.recommendation_engine.filtering import FilterEngine
from backend.engines.recommendation_engine.ranking import RankingEngine


@pytest.fixture
def mock_unified_context():
    """Contexte unifié déterministe pour les tests."""
    observations = [
        Observation(
            provider="atmo-france",
            observedAt=datetime(2026, 8, 17, 12, 0),
            retrievedAt=datetime(2026, 8, 17, 12, 0),
            expiresAt=datetime(2026, 8, 17, 13, 0),
            quality=0.95,
            value={"aqi": 42, "polluant": "PM2.5", "rain_probability": 10, "wind_speed": 5}
        ),
        Observation(
            provider="rte",
            observedAt=datetime(2026, 8, 17, 12, 0),
            retrievedAt=datetime(2026, 8, 17, 12, 0),
            expiresAt=datetime(2026, 8, 17, 13, 0),
            quality=0.95,
            value={"mix": {"nuclear": 0.5, "hydro": 0.3}, "co2": 100}
        ),
        Observation(
            provider="hubeau",
            observedAt=datetime(2026, 8, 17, 12, 0),
            retrievedAt=datetime(2026, 8, 17, 12, 0),
            expiresAt=datetime(2026, 8, 17, 13, 0),
            quality=0.95,
            value={"niveau": 120.5, "unite": "cm"}
        ),
    ]
    return UnifiedContext(
        observations=observations,
        timestamp=datetime(2026, 8, 17, 12, 0),
        derived_data={
            "global_aqi": 42,
            "energy_mix": {"nuclear": 0.5, "hydro": 0.3},
            "avg_water_level": 120.5
        }
    )


@pytest.fixture
def mock_user_profile():
    """Profil utilisateur déterministe pour les tests."""
    return UserProfile(
        user_id="user123",
        location="69003",
        preferences={
            "likes_cycling": True,
            "likes_walking": True,
            "likes_nature": True,
            "likes_repair": False,
            "likes_reuse": False,
            "max_duration": 60
        }
    )


# Test de déterminisme du RecommendationEngine complet
def test_recommendation_engine_determinism(mock_unified_context, mock_user_profile):
    """Test que le RecommendationEngine produit les mêmes résultats pour les mêmes entrées."""
    # Exécuter le RecommendationEngine deux fois avec les mêmes entrées
    recommendations1 = recommendation_engine.generate_recommendations(mock_unified_context, mock_user_profile)
    recommendations2 = recommendation_engine.generate_recommendations(mock_unified_context, mock_user_profile)

    # Vérifier que les mêmes recommandations sont générées
    assert len(recommendations1) == len(recommendations2)
    for rec1, rec2 in zip(recommendations1, recommendations2):
        assert rec1.id == rec2.id
        assert rec1.title == rec2.title
        assert rec1.category == rec2.category
        assert rec1.score == rec2.score  # Scores identiques


# Test de déterminisme du RuleEngine
def test_rule_engine_determinism(mock_unified_context, mock_user_profile):
    """Test que le RuleEngine produit les mêmes candidats pour les mêmes entrées."""
    rule_engine = RuleEngine()
    # Ajouter les règles par défaut (mêmes règles que dans le RecommendationEngine)
    from backend.engines.recommendation_engine.rules.mobility import BikeActivityRule, WalkActivityRule, CarpoolingRule
    from backend.engines.recommendation_engine.rules.nature import ForestWalkRule, GardeningRule
    from backend.engines.recommendation_engine.rules.energy import ReduceEnergyConsumptionRule, UseRenewableEnergyRule
    from backend.engines.recommendation_engine.rules.water import WateringRule, SaveWaterRule
    from backend.engines.recommendation_engine.rules.repair import RepairRule, ReuseRule
    from backend.engines.recommendation_engine.rules.wellbeing import MeditationRule, YogaRule

    rule_engine.add_rule(BikeActivityRule())
    rule_engine.add_rule(WalkActivityRule())
    rule_engine.add_rule(CarpoolingRule())
    rule_engine.add_rule(ForestWalkRule())
    rule_engine.add_rule(GardeningRule())
    rule_engine.add_rule(ReduceEnergyConsumptionRule())
    rule_engine.add_rule(UseRenewableEnergyRule())
    rule_engine.add_rule(WateringRule())
    rule_engine.add_rule(SaveWaterRule())
    rule_engine.add_rule(RepairRule())
    rule_engine.add_rule(ReuseRule())
    rule_engine.add_rule(MeditationRule())
    rule_engine.add_rule(YogaRule())

    # Exécuter le RuleEngine deux fois
    candidates1 = rule_engine.evaluate_all(mock_unified_context, mock_user_profile)
    candidates2 = rule_engine.evaluate_all(mock_unified_context, mock_user_profile)

    # Vérifier que les mêmes candidats sont générés
    assert len(candidates1) == len(candidates2)
    for rec1, rec2 in zip(candidates1, candidates2):
        assert rec1.id == rec2.id
        assert rec1.title == rec2.title
        assert rec1.category == rec2.category


# Test de déterminisme du ScoringEngine
def test_scoring_engine_determinism(mock_unified_context, mock_user_profile):
    """Test que le ScoringEngine produit les mêmes scores pour les mêmes entrées."""
    scoring_engine = ScoringEngine()

    # Créer des recommandations déterministes
    recommendations = [
        Recommendation(
            id="rec_001",
            title="Balade à vélo",
            description="Profitez de la bonne qualité de l'air.",
            category=RecommendationCategory.MOBILITY,
            action="Faire du vélo",
            duration_minutes=30
        ),
        Recommendation(
            id="rec_002",
            title="Balade en forêt",
            description="Explorez les sentiers locaux.",
            category=RecommendationCategory.NATURE,
            action="Balade en forêt",
            duration_minutes=45
        ),
    ]

    # Calculer les scores deux fois
    scored1 = scoring_engine.calculate_scores(recommendations, mock_unified_context, mock_user_profile)
    scored2 = scoring_engine.calculate_scores(recommendations, mock_unified_context, mock_user_profile)

    # Vérifier que les mêmes scores sont calculés
    assert len(scored1) == len(scored2)
    for rec1, rec2 in zip(scored1, scored2):
        assert rec1.score == rec2.score


# Test de déterminisme du FilterEngine
def test_filter_engine_determinism(mock_unified_context, mock_user_profile):
    """Test que le FilterEngine produit les mêmes résultats pour les mêmes entrées."""
    filter_engine = FilterEngine()

    # Créer des recommandations déterministes
    recommendations = [
        Recommendation(
            id="rec_001",
            title="Balade à vélo",
            description="Profitez de la bonne qualité de l'air.",
            category=RecommendationCategory.MOBILITY,
            action="Faire du vélo",
            duration_minutes=30
        ),
        Recommendation(
            id="rec_002",
            title="Arroser les plantes",
            description="Niveau d'eau bas.",
            category=RecommendationCategory.WATER,
            action="Arroser les plantes",
            duration_minutes=15
        ),
    ]

    # Filtrer les recommandations deux fois
    valid1, alternatives1 = filter_engine.filter_recommendations(recommendations, mock_unified_context, mock_user_profile)
    valid2, alternatives2 = filter_engine.filter_recommendations(recommendations, mock_unified_context, mock_user_profile)

    # Vérifier que les mêmes résultats sont obtenus
    assert len(valid1) == len(valid2)
    assert len(alternatives1) == len(alternatives2)
    for v1, v2 in zip(valid1, valid2):
        assert v1.id == v2.id


# Test de déterminisme du RankingEngine
def test_ranking_engine_determinism(mock_unified_context, mock_user_profile):
    """Test que le RankingEngine produit les mêmes résultats pour les mêmes entrées."""
    ranking_engine = RankingEngine(enable_diversity=False)

    # Créer des recommandations déterministes avec des scores
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
            title="Balade en forêt",
            description="Explorez les sentiers locaux.",
            category=RecommendationCategory.NATURE,
            action="Balade en forêt",
            score=0.85
        ),
        Recommendation(
            id="rec_003",
            title="Réduire la consommation",
            description="Éteignez les appareils inutiles.",
            category=RecommendationCategory.ENERGY,
            action="Réduire la consommation",
            score=0.80
        ),
    ]

    # Classer les recommandations deux fois
    ranked1 = ranking_engine.rank_recommendations(recommendations)
    ranked2 = ranking_engine.rank_recommendations(recommendations)

    # Vérifier que les mêmes résultats sont obtenus
    assert len(ranked1) == len(ranked2)
    for rec1, rec2 in zip(ranked1, ranked2):
        assert rec1.id == rec2.id


# Test de déterminisme avec diversité activée
def test_ranking_engine_determinism_with_diversity(mock_unified_context, mock_user_profile):
    """Test que le RankingEngine avec diversité est déterministe."""
    ranking_engine = RankingEngine(enable_diversity=True)

    # Créer des recommandations déterministes avec des scores identiques
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

    # Classer les recommandations deux fois
    ranked1 = ranking_engine.rank_recommendations(recommendations)
    ranked2 = ranking_engine.rank_recommendations(recommendations)

    # Vérifier que les mêmes résultats sont obtenus
    assert len(ranked1) == len(ranked2)
    for rec1, rec2 in zip(ranked1, ranked2):
        assert rec1.id == rec2.id


# Test de déterminisme avec plusieurs exécutions
def test_multiple_executions_determinism(mock_unified_context, mock_user_profile):
    """Test que le RecommendationEngine est déterministe sur plusieurs exécutions."""
    results = []
    for _ in range(5):  # Exécuter 5 fois
        recommendations = recommendation_engine.generate_recommendations(mock_unified_context, mock_user_profile)
        results.append([(rec.id, rec.score) for rec in recommendations])

    # Vérifier que tous les résultats sont identiques
    for i in range(1, len(results)):
        assert results[0] == results[i]
```

---

## Configuration Requise
Aucune configuration supplémentaire nécessaire. Utilise les composants existants (`RuleEngine`, `ScoringEngine`, `FilterEngine`, `RankingEngine`) de l'**Épic 2**.

---

## Dépendances
- **Python 3.11+**
- **Librairies** :
  - `pytest` (pour les tests unitaires).

---

## Notes
- **Déterminisme absolu** : Le **Recommendation Engine** doit être **100% déterministe** pour le core (génération, scoring, filtrage, classement).
- **Pas de LLM dans le core** : Les LLMs ne sont **pas utilisés** dans le core du Recommendation Engine. Ils sont réservés à des fonctionnalités optionnelles comme la génération d'explications textuelles (voir **Épic 4**).
- **Tests exhaustifs** : Les tests couvrent **tous les composants** (`RuleEngine`, `ScoringEngine`, `FilterEngine`, `RankingEngine`) et le **RecommendationEngine complet**.
- **Reproductibilité** : Les mêmes entrées (`UnifiedContext`, `UserProfile`) doivent **toujours** produire les mêmes sorties.

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine import recommendation_engine
from backend.engines.context_engine.models import UnifiedContext, UserProfile

# Créer un contexte et un profil utilisateur déterministes
context = UnifiedContext(...)  # Contexte unifié
user_profile = UserProfile(...)  # Profil utilisateur

# Générer des recommandations (déterministe)
recommendations1 = recommendation_engine.generate_recommendations(context, user_profile)
recommendations2 = recommendation_engine.generate_recommendations(context, user_profile)

# Vérifier que les résultats sont identiques
assert recommendations1 == recommendations2  # Toujours vrai
```

---

## Ressources
- [PRD Almanéa - FR-10](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-5](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
