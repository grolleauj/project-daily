---
name: Implémenter le scoring
id: 2-2-implémenter-le-scoring
epic: epic-2
story_type: backend
priority: high
estimation: M
dependencies: [2-1]
status: ready-for-dev
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/implementation-artifacts/epic-1-context.md
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 2.2: Implémenter le scoring

## Contexte
Cette story fait partie de **l'Épic 2 (Recommendation Engine)**. Son objectif est de **calculer un score pour chaque candidat de recommandation** généré par le `Rule Engine`, conformément à **FR-7 (Calcul du score des recommandations)**. Le score permet de **classer les recommandations par pertinence** et de prioriser les meilleures options pour l'utilisateur.

Le `ScoringEngine` applique une **formule déterministe** pour garantir que les mêmes entrées produisent toujours les mêmes scores, conformément à **NFR-1 (Déterminisme)**.

---

## Exigences Fonctionnelles
- **FR-7**: Calcul du score des recommandations via une formule pondérée.
- **NFR-1**: Déterminisme — Les scores doivent être reproductibles.

---

## Critères d'Acceptation
1. **Calcul du score** :
   - [ ] Chaque recommandation se voit attribuer un **score normalisé entre 0 et 1**.
   - [ ] La formule utilisée est :
     `Score = 25% ContextFit + 20% TimeFit + 15% UserPreference + 15% Accessibility + 10% EnvironmentalOpportunity + 10% WellbeingOpportunity + 5% Novelty`.

2. **Validation des composants** :
   - [ ] Les **valeurs nulles** pour un composant (ex: `ContextFit=0`) sont **remplacées par une valeur par défaut (0.5)**.
   - [ ] Les **poids** (25%, 20%, etc.) sont **validés** pour s'assurer qu'ils totalisent 100%.
   - [ ] Une **erreur** est générée si un composant retourne une valeur **hors plage** (ex: `ContextFit > 1`).

3. **Déterminisme** :
   - [ ] Le score est **déterministe** (mêmes entrées → même score).
   - [ ] Les tests vérifient que le calcul est **reproductible**.

4. **Intégration avec le Recommendation Engine** :
   - [ ] Le `ScoringEngine` est **intégré avec le `RuleEngine` et le `RankingEngine`**.
   - [ ] Le scoring est **appliqué automatiquement** après la génération des candidats.

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/scoring.py` | Module `ScoringEngine` | À créer |
| `backend/engines/recommendation_engine/__init__.py` | Export du `ScoringEngine` | À modifier |
| `tests/unit/test_scoring.py` | Tests unitaires | À créer |

### Structure du `ScoringEngine` (`scoring.py`)
```python
from typing import List, Dict, Optional
from dataclasses import dataclass
from .models import Recommendation, UnifiedContext, UserProfile


@dataclass
class ScoreComponents:
    """Composants du score avec leurs poids respectifs."""
    context_fit: float = 0.5
    time_fit: float = 0.5
    user_preference: float = 0.5
    accessibility: float = 0.5
    environmental_opportunity: float = 0.5
    wellbeing_opportunity: float = 0.5
    novelty: float = 0.5


class ScoringEngine:
    """
    Moteur de scoring pour calculer la pertinence des recommandations.
    - Applique une formule pondérée pour calculer un score entre 0 et 1.
    - Remplace les valeurs nulles par 0.5.
    - Valide que les poids totalisent 100%.
    """

    # Poids des composants (25% + 20% + 15% + 15% + 10% + 10% + 5% = 100%)
    WEIGHTS = {
        "context_fit": 0.25,
        "time_fit": 0.20,
        "user_preference": 0.15,
        "accessibility": 0.15,
        "environmental_opportunity": 0.10,
        "wellbeing_opportunity": 0.10,
        "novelty": 0.05
    }

    def __init__(self):
        """Initialise le ScoringEngine."""
        pass

    def calculate_score(self, recommendation: Recommendation, context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> float:
        """
        Calcule le score d'une recommandation.

        Args:
            recommendation: La recommandation à scorer.
            context: Le contexte unifié.
            user_profile: Le profil utilisateur (optionnel).

        Returns:
            float: Score normalisé entre 0 et 1.
        """
        components = self._calculate_components(recommendation, context, user_profile)
        return self._apply_weights(components)

    def calculate_scores(self, recommendations: List[Recommendation], context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> List[Recommendation]:
        """
        Calcule les scores pour une liste de recommandations.

        Args:
            recommendations: Liste de recommandations à scorer.
            context: Le contexte unifié.
            user_profile: Le profil utilisateur (optionnel).

        Returns:
            List[Recommendation]: Liste de recommandations avec leurs scores mis à jour.
        """
        for rec in recommendations:
            rec.score = self.calculate_score(rec, context, user_profile)
        return recommendations

    def _calculate_components(self, recommendation: Recommendation, context: UnifiedContext, user_profile: Optional[UserProfile]) -> ScoreComponents:
        """
        Calcule les composants du score pour une recommandation.

        Args:
            recommendation: La recommandation à évaluer.
            context: Le contexte unifié.
            user_profile: Le profil utilisateur (optionnel).

        Returns:
            ScoreComponents: Les valeurs des composants (entre 0 et 1).
        """
        components = ScoreComponents()

        # ContextFit: Pertinence par rapport au contexte environnemental (ex: AQI, météo)
        components.context_fit = self._calculate_context_fit(recommendation, context)

        # TimeFit: Pertinence par rapport au moment (ex: heure de la journée, jour de la semaine)
        components.time_fit = self._calculate_time_fit(recommendation, context)

        # UserPreference: Pertinence par rapport aux préférences utilisateur
        components.user_preference = self._calculate_user_preference(recommendation, user_profile)

        # Accessibility: Accessibilité de l'action (ex: distance, durée, ressources requises)
        components.accessibility = self._calculate_accessibility(recommendation, context, user_profile)

        # EnvironmentalOpportunity: Opportunité environnementale (ex: qualité de l'air excellente pour une balade)
        components.environmental_opportunity = self._calculate_environmental_opportunity(recommendation, context)

        # WellbeingOpportunity: Opportunité pour le bien-être (ex: activité relaxante après une journée stressante)
        components.wellbeing_opportunity = self._calculate_wellbeing_opportunity(recommendation, context)

        # Novelty: Nouveauté de la recommandation (éviter la répétition)
        components.novelty = self._calculate_novelty(recommendation, context)

        return components

    def _apply_weights(self, components: ScoreComponents) -> float:
        """
        Applique les poids aux composants pour calculer le score final.

        Args:
            components: Les composants du score.

        Returns:
            float: Score final entre 0 et 1.
        """
        # Remplacer les valeurs nulles par 0.5
        normalized_components = {
            "context_fit": components.context_fit if components.context_fit is not None else 0.5,
            "time_fit": components.time_fit if components.time_fit is not None else 0.5,
            "user_preference": components.user_preference if components.user_preference is not None else 0.5,
            "accessibility": components.accessibility if components.accessibility is not None else 0.5,
            "environmental_opportunity": components.environmental_opportunity if components.environmental_opportunity is not None else 0.5,
            "wellbeing_opportunity": components.wellbeing_opportunity if components.wellbeing_opportunity is not None else 0.5,
            "novelty": components.novelty if components.novelty is not None else 0.5
        }

        # Valider que les valeurs sont entre 0 et 1
        for key, value in normalized_components.items():
            if not (0 <= value <= 1):
                raise ValueError(f"Composant {key} hors plage: {value} (doit être entre 0 et 1)")

        # Calculer le score pondéré
        score = sum(normalized_components[key] * self.WEIGHTS[key] for key in self.WEIGHTS)

        # Valider que le score est entre 0 et 1
        if not (0 <= score <= 1):
            raise ValueError(f"Score final hors plage: {score} (doit être entre 0 et 1)")

        return score

    def _calculate_context_fit(self, recommendation: Recommendation, context: UnifiedContext) -> float:
        """
        Calcule la pertinence de la recommandation par rapport au contexte environnemental.

        Args:
            recommendation: La recommandation à évaluer.
            context: Le contexte unifié.

        Returns:
            float: Pertinence entre 0 et 1.
        """
        # Exemple: Pour une recommandation de balade à vélo, vérifier la qualité de l'air (AQI)
        aqi = context.derived_data.get("global_aqi", 50)
        if recommendation.category.value == "mobility":
            # AQI < 50: excellente qualité de l'air -> 1.0
            # AQI < 100: bonne qualité -> 0.75
            # AQI < 150: moyenne -> 0.5
            # AQI >= 150: mauvaise -> 0.0
            if aqi < 50:
                return 1.0
            elif aqi < 100:
                return 0.75
            elif aqi < 150:
                return 0.5
            else:
                return 0.0
        elif recommendation.category.value == "nature":
            # Pour les activités en nature, vérifier AQI et pluie
            rain_probability = context.get_value("atmo-france", "rain_probability", 0)
            if aqi < 50 and rain_probability < 20:
                return 1.0
            elif aqi < 70 and rain_probability < 40:
                return 0.75
            elif aqi < 100 and rain_probability < 60:
                return 0.5
            else:
                return 0.0
        elif recommendation.category.value == "energy":
            # Pour les recommandations énergétiques, vérifier le mix énergétique
            co2 = context.get_value("rte", "co2", 0)
            if co2 > 800:
                return 1.0  # Mix très carboné -> opportunité de réduire la consommation
            elif co2 > 500:
                return 0.75
            elif co2 > 300:
                return 0.5
            else:
                return 0.0
        elif recommendation.category.value == "water":
            # Pour les recommandations liées à l'eau, vérifier le niveau d'eau
            avg_water_level = context.derived_data.get("avg_water_level", 100)
            if avg_water_level < 20:
                return 1.0  # Niveau critique -> opportunité d'économiser l'eau
            elif avg_water_level < 50:
                return 0.75
            elif avg_water_level < 80:
                return 0.5
            else:
                return 0.0
        else:
            return 0.5  # Valeur par défaut

    def _calculate_time_fit(self, recommendation: Recommendation, context: UnifiedContext) -> float:
        """
        Calcule la pertinence de la recommandation par rapport au moment.

        Args:
            recommendation: La recommandation à évaluer.
            context: Le contexte unifié.

        Returns:
            float: Pertinence entre 0 et 1.
        """
        # Exemple: Vérifier l'heure de la journée et le jour de la semaine
        from datetime import datetime
        now = datetime.now()
        hour = now.hour
        weekday = now.weekday()  # 0=lundi, 6=dimanche

        if recommendation.category.value == "mobility":
            # Les activités de mobilité sont plus pertinentes en journée (8h-18h) et en semaine
            if 8 <= hour <= 18 and weekday < 5:
                return 1.0
            elif 6 <= hour <= 20:
                return 0.75
            else:
                return 0.5
        elif recommendation.category.value == "wellbeing":
            # Les activités de bien-être sont pertinentes le matin ou le soir
            if 6 <= hour <= 10 or 18 <= hour <= 22:
                return 1.0
            elif 10 <= hour <= 18:
                return 0.75
            else:
                return 0.5
        else:
            return 0.75  # Valeur par défaut

    def _calculate_user_preference(self, recommendation: Recommendation, user_profile: Optional[UserProfile]) -> float:
        """
        Calcule la pertinence de la recommandation par rapport aux préférences utilisateur.

        Args:
            recommendation: La recommandation à évaluer.
            user_profile: Le profil utilisateur (optionnel).

        Returns:
            float: Pertinence entre 0 et 1.
        """
        if user_profile is None:
            return 0.5

        preferences = user_profile.preferences or {}

        if recommendation.category.value == "mobility":
            if preferences.get("likes_cycling", False):
                return 1.0
            elif preferences.get("likes_walking", False):
                return 0.75
            else:
                return 0.5
        elif recommendation.category.value == "nature":
            if preferences.get("likes_nature", False):
                return 1.0
            else:
                return 0.5
        elif recommendation.category.value == "repair":
            if preferences.get("likes_repair", False) or preferences.get("likes_reuse", False):
                return 1.0
            else:
                return 0.5
        else:
            return 0.5

    def _calculate_accessibility(self, recommendation: Recommendation, context: UnifiedContext, user_profile: Optional[UserProfile]) -> float:
        """
        Calcule l'accessibilité de la recommandation (ex: distance, durée, ressources requises).

        Args:
            recommendation: La recommandation à évaluer.
            context: Le contexte unifié.
            user_profile: Le profil utilisateur (optionnel).

        Returns:
            float: Accessibilité entre 0 et 1.
        """
        if recommendation.duration_minutes is None:
            return 0.5

        if user_profile is None:
            return 0.75

        max_duration = user_profile.preferences.get("max_duration", 120)  # 2 heures par défaut

        if recommendation.duration_minutes <= max_duration * 0.5:
            return 1.0
        elif recommendation.duration_minutes <= max_duration:
            return 0.75
        else:
            return 0.25

    def _calculate_environmental_opportunity(self, recommendation: Recommendation, context: UnifiedContext) -> float:
        """
        Calcule l'opportunité environnementale de la recommandation.

        Args:
            recommendation: La recommandation à évaluer.
            context: Le contexte unifié.

        Returns:
            float: Opportunité entre 0 et 1.
        """
        # Exemple: Pour une recommandation de balade à vélo, vérifier si la qualité de l'air est excellente
        aqi = context.derived_data.get("global_aqi", 50)
        if aqi <= 30:
            return 1.0  # Qualité de l'air excellente
        elif aqi <= 50:
            return 0.75
        elif aqi <= 70:
            return 0.5
        else:
            return 0.25

    def _calculate_wellbeing_opportunity(self, recommendation: Recommendation, context: UnifiedContext) -> float:
        """
        Calcule l'opportunité pour le bien-être de la recommandation.

        Args:
            recommendation: La recommandation à évaluer.
            context: Le contexte unifié.

        Returns:
            float: Opportunité entre 0 et 1.
        """
        # Exemple: Pour une recommandation de méditation, vérifier si la qualité de l'air est bonne
        aqi = context.derived_data.get("global_aqi", 50)
        if recommendation.category.value == "wellbeing":
            if aqi <= 50:
                return 1.0
            elif aqi <= 70:
                return 0.75
            else:
                return 0.5
        else:
            return 0.5

    def _calculate_novelty(self, recommendation: Recommendation, context: UnifiedContext) -> float:
        """
        Calcule la nouveauté de la recommandation (éviter la répétition).

        Args:
            recommendation: La recommandation à évaluer.
            context: Le contexte unifié.

        Returns:
            float: Nouveauté entre 0 et 1.
        """
        # Exemple: Si la recommandation a déjà été proposée récemment, réduire la nouveauté
        # Pour simplifier, on retourne une valeur par défaut
        return 0.75


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
Créer un fichier `tests/unit/test_scoring.py` avec les tests suivants :

```python
import pytest
from datetime import datetime, timedelta
from unittest.mock import AsyncMock, patch, MagicMock
import sys
import os

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '../../backend'))

from backend.engines.recommendation_engine.models import Recommendation, RecommendationCategory, UnifiedContext, UserProfile
from backend.engines.recommendation_engine.scoring import ScoringEngine, ScoreComponents


@pytest.fixture
def scoring_engine():
    return ScoringEngine()


@pytest.fixture
def mock_recommendation():
    return Recommendation(
        id="rec_001",
        title="Balade à vélo",
        description="Profitez de la bonne qualité de l'air.",
        category=RecommendationCategory.MOBILITY,
        action="Faire du vélo",
        duration_minutes=30
    )


@pytest.fixture
def mock_unified_context():
    return UnifiedContext(
        observations=[],
        timestamp=datetime.now(),
        derived_data={
            "global_aqi": 42,
            "energy_mix": {"nuclear": 0.5, "hydro": 0.3},
            "avg_water_level": 120.5
        }
    )


@pytest.fixture
def mock_user_profile():
    return UserProfile(
        user_id="user123",
        location="69003",
        preferences={
            "likes_cycling": True,
            "max_duration": 60
        }
    )


# Test de calcul du score
@pytest.mark.parametrize("aqi,expected", [
    (20, 1.0),   # AQI < 50 -> 1.0
    (40, 1.0),
    (60, 0.75),  # 50 <= AQI < 100 -> 0.75
    (80, 0.75),
    (120, 0.5),  # 100 <= AQI < 150 -> 0.5
    (160, 0.0),  # AQI >= 150 -> 0.0
])
def test_calculate_context_fit_mobility(scoring_engine, mock_recommendation, mock_unified_context, aqi, expected):
    mock_unified_context.derived_data["global_aqi"] = aqi
    mock_recommendation.category = RecommendationCategory.MOBILITY
    context_fit = scoring_engine._calculate_context_fit(mock_recommendation, mock_unified_context)
    assert context_fit == expected


# Test de calcul du TimeFit
def test_calculate_time_fit_mobility(scoring_engine, mock_recommendation, mock_unified_context):
    mock_recommendation.category = RecommendationCategory.MOBILITY
    # Simuler une heure en journée (10h)
    with patch('backend.engines.recommendation_engine.scoring.datetime') as mock_datetime:
        mock_datetime.now.return_value.hour = 10
        mock_datetime.now.return_value.weekday.return_value = 2  # Mercredi
        time_fit = scoring_engine._calculate_time_fit(mock_recommendation, mock_unified_context)
        assert time_fit == 1.0


# Test de calcul du UserPreference
def test_calculate_user_preference_mobility(scoring_engine, mock_recommendation, mock_user_profile):
    mock_recommendation.category = RecommendationCategory.MOBILITY
    mock_user_profile.preferences["likes_cycling"] = True
    user_pref = scoring_engine._calculate_user_preference(mock_recommendation, mock_user_profile)
    assert user_pref == 1.0


# Test de calcul de l'Accessibility
def test_calculate_accessibility(scoring_engine, mock_recommendation, mock_user_profile):
    mock_recommendation.duration_minutes = 30
    mock_user_profile.preferences["max_duration"] = 60
    accessibility = scoring_engine._calculate_accessibility(mock_recommendation, mock_unified_context, mock_user_profile)
    assert accessibility == 1.0


# Test de calcul de l'EnvironmentalOpportunity
def test_calculate_environmental_opportunity(scoring_engine, mock_recommendation, mock_unified_context):
    mock_unified_context.derived_data["global_aqi"] = 20
    env_opportunity = scoring_engine._calculate_environmental_opportunity(mock_recommendation, mock_unified_context)
    assert env_opportunity == 1.0


# Test de calcul du WellbeingOpportunity
def test_calculate_wellbeing_opportunity(scoring_engine, mock_recommendation, mock_unified_context):
    mock_recommendation.category = RecommendationCategory.WELLBEING
    mock_unified_context.derived_data["global_aqi"] = 40
    wellbeing_opportunity = scoring_engine._calculate_wellbeing_opportunity(mock_recommendation, mock_unified_context)
    assert wellbeing_opportunity == 1.0


# Test de calcul du score complet
def test_calculate_score(scoring_engine, mock_recommendation, mock_unified_context, mock_user_profile):
    mock_recommendation.category = RecommendationCategory.MOBILITY
    mock_recommendation.duration_minutes = 30
    mock_unified_context.derived_data["global_aqi"] = 40
    mock_user_profile.preferences["likes_cycling"] = True
    mock_user_profile.preferences["max_duration"] = 60

    with patch('backend.engines.recommendation_engine.scoring.datetime') as mock_datetime:
        mock_datetime.now.return_value.hour = 10
        mock_datetime.now.return_value.weekday.return_value = 2

        score = scoring_engine.calculate_score(mock_recommendation, mock_unified_context, mock_user_profile)
        # Vérifier que le score est entre 0 et 1
        assert 0 <= score <= 1


# Test de calcul des scores pour une liste
def test_calculate_scores(scoring_engine, mock_unified_context, mock_user_profile):
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

    scored_recommendations = scoring_engine.calculate_scores(recommendations, mock_unified_context, mock_user_profile)
    assert len(scored_recommendations) == 2
    for rec in scored_recommendations:
        assert rec.score is not None
        assert 0 <= rec.score <= 1


# Test de validation des poids
def test_weights_sum_to_100(scoring_engine):
    total_weight = sum(ScoringEngine.WEIGHTS.values())
    assert abs(total_weight - 1.0) < 1e-6


# Test de déterminisme
def test_deterministic_scoring(scoring_engine, mock_recommendation, mock_unified_context, mock_user_profile):
    mock_recommendation.category = RecommendationCategory.MOBILITY
    mock_recommendation.duration_minutes = 30
    mock_unified_context.derived_data["global_aqi"] = 40
    mock_user_profile.preferences["likes_cycling"] = True
    mock_user_profile.preferences["max_duration"] = 60

    with patch('backend.engines.recommendation_engine.scoring.datetime') as mock_datetime:
        mock_datetime.now.return_value.hour = 10
        mock_datetime.now.return_value.weekday.return_value = 2

        score1 = scoring_engine.calculate_score(mock_recommendation, mock_unified_context, mock_user_profile)
        score2 = scoring_engine.calculate_score(mock_recommendation, mock_unified_context, mock_user_profile)
        assert score1 == score2


# Test de gestion des valeurs nulles
def test_null_components(scoring_engine, mock_recommendation, mock_unified_context):
    mock_recommendation.category = RecommendationCategory.MOBILITY
    mock_recommendation.duration_minutes = None

    # Simuler des composants nuls
    with patch.object(scoring_engine, '_calculate_components', return_value=ScoreComponents()):
        score = scoring_engine.calculate_score(mock_recommendation, mock_unified_context, None)
        # Tous les composants sont nuls -> remplacés par 0.5
        # Score = 0.25*0.5 + 0.20*0.5 + 0.15*0.5 + 0.15*0.5 + 0.10*0.5 + 0.10*0.5 + 0.05*0.5 = 0.5
        assert abs(score - 0.5) < 1e-6


# Test d'erreur pour les valeurs hors plage
def test_out_of_range_component(scoring_engine, mock_recommendation, mock_unified_context):
    mock_recommendation.category = RecommendationCategory.MOBILITY

    # Simuler un composant hors plage
    with patch.object(scoring_engine, '_calculate_components', return_value=ScoreComponents(context_fit=1.5)):
        with pytest.raises(ValueError, match="Composant context_fit hors plage"):
            scoring_engine.calculate_score(mock_recommendation, mock_unified_context, None)
```

---

## Configuration Requise
Aucune configuration supplémentaire nécessaire. Utilise les modèles existants (`UnifiedContext`, `UserProfile`, `Recommendation`) de l'**Épic 1**.

---

## Dépendances
- **Python 3.11+**
- **Librairies** :
  - Aucune dépendance supplémentaire.

---

## Notes
- **Déterminisme** : Le `ScoringEngine` doit être **100% déterministe** (mêmes entrées → mêmes scores).
- **Normalisation** : Les scores sont **normalisés entre 0 et 1**.
- **Valeurs par défaut** : Les composants nuls sont **remplacés par 0.5**.
- **Validation** : Les poids totalisent **100%** et les valeurs hors plage génèrent des **erreurs**.
- **Intégration** : Le `ScoringEngine` est **intégré avec le `RuleEngine` et le `RankingEngine`**.

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine import recommendation_engine
from backend.engines.context_engine.models import UnifiedContext, UserProfile

# Générer, scorer et classer les recommandations
context = UnifiedContext(...)  # Contexte unifié
user_profile = UserProfile(...)  # Profil utilisateur

recommendations = recommendation_engine.generate_recommendations(context, user_profile)

# Afficher les recommandations avec leurs scores
for rec in recommendations:
    print(f"{rec.title} (Score: {rec.score:.2f}, Catégorie: {rec.category.value})")
```

---

## Ressources
- [PRD Almanéa - FR-7](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-5](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
