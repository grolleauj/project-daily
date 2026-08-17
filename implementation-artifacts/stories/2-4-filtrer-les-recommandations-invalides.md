---
name: Filtrer les recommandations invalides
id: 2-4-filtrer-les-recommandations-invalides
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

# Story 2.4: Filtrer les recommandations invalides

## Contexte
Cette story fait partie de **l'Épic 2 (Recommendation Engine)**. Son objectif est de **supprimer ou ajuster les recommandations invalides ou contradictoires**, conformément à **FR-9 (Filtrage des recommandations invalides ou contradictoires)**. Cela garantit que les utilisateurs ne reçoivent que des recommandations **sûres, pertinentes et applicables** dans leur contexte.

Le `FilterEngine` applique des **règles de filtrage** pour éliminer les recommandations qui :
- Présentent des **risques pour la sécurité** (ex: conditions météo dangereuses).
- Sont **incompatibles avec des restrictions légales** (ex: restriction d'eau).
- Violent des **contraintes techniques** (ex: données critiques indisponibles).
- Ne respectent pas les **préférences utilisateur** (ex: durée maximale dépassée).

---

## Exigences Fonctionnelles
- **FR-9**: Filtrage des recommandations invalides ou contradictoires.
- **NFR-1**: Déterminisme — Le filtrage doit être reproductible.

---

## Critères d'Acceptation
1. **Filtrage des recommandations invalides** :
   - [ ] Une recommandation est **supprimée** si elle présente :
     - Des **conditions météo dangereuses** (ex: AQI > 150, pluie > 80%).
     - Une **qualité de l'air incompatible** avec l'activité (ex: balade à vélo avec AQI > 100).
     - Une **restriction locale incompatible** (ex: restriction d'eau active mais recommandation d'arrosage).
     - Des **données critiques indisponibles** (ex: AQI ou météo manquants).
     - Un **temps insuffisant** (ex: durée de l'activité > temps disponible utilisateur).

2. **Gestion des contradictions** :
   - [ ] En cas de **contradictions entre recommandations**, la **règle critique prime** selon la priorité :
     1. **Sécurité** (ex: éviter les activités dangereuses).
     2. **Restrictions légales** (ex: respecter les interdictions locales).
     3. **Contraintes techniques** (ex: données manquantes).
     4. **Préférences utilisateur** (ex: durée maximale).

3. **Remplacement par des messages alternatifs** :
   - [ ] Les recommandations filtrées sont **remplacées par un message alternatif** expliquant pourquoi elles ne sont pas applicables (ex: "Arrosage interdit aujourd'hui. Essayez une activité sans eau.").

4. **Déterminisme** :
   - [ ] Le filtrage est **déterministe** (mêmes entrées → mêmes résultats).
   - [ ] Les tests vérifient que le filtrage est **reproductible**.

5. **Intégration avec le Recommendation Engine** :
   - [ ] Le `FilterEngine` est **intégré avec le `RuleEngine`, `ScoringEngine` et `RankingEngine`**.
   - [ ] Le filtrage est **appliqué automatiquement** après le scoring et avant le classement.

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/filtering.py` | Module `FilterEngine` | À créer |
| `backend/engines/recommendation_engine/__init__.py` | Export du `FilterEngine` | À modifier |
| `tests/unit/test_filtering.py` | Tests unitaires | À créer |

### Structure du `FilterEngine` (`filtering.py`)
```python
from typing import List, Dict, Optional, Tuple
from enum import Enum
from .models import Recommendation, UnifiedContext, UserProfile, RecommendationCategory


class FilterPriority(Enum):
    """Priorité des règles de filtrage."""
    SAFETY = 1          # Sécurité (ex: conditions météo dangereuses)
    LEGAL = 2          # Restrictions légales (ex: restriction d'eau)
    TECHNICAL = 3      # Contraintes techniques (ex: données manquantes)
    USER_PREFERENCE = 4  # Préférences utilisateur (ex: durée maximale)


class FilterRule:
    """Règle de filtrage pour les recommandations."""

    def __init__(self, name: str, priority: FilterPriority, condition_func, message: str):
        """
        Initialise une règle de filtrage.

        Args:
            name: Nom de la règle (ex: "aqi_too_high").
            priority: Priorité de la règle (FilterPriority).
            condition_func: Fonction qui prend (recommendation, context, user_profile) et retourne un booléen.
            message: Message alternatif si la recommandation est filtrée.
        """
        self.name = name
        self.priority = priority
        self.condition_func = condition_func
        self.message = message

    def apply(self, recommendation: Recommendation, context: UnifiedContext, user_profile: Optional[UserProfile]) -> Optional[str]:
        """
        Applique la règle de filtrage à une recommandation.

        Args:
            recommendation: La recommandation à filtrer.
            context: Le contexte unifié.
            user_profile: Le profil utilisateur (optionnel).

        Returns:
            Optional[str]: Message alternatif si la recommandation est filtrée, sinon None.
        """
        if self.condition_func(recommendation, context, user_profile):
            return self.message
        return None


class FilterEngine:
    """
    Moteur de filtrage pour supprimer ou ajuster les recommandations invalides.
    - Applique des règles de filtrage par priorité.
    - Remplace les recommandations filtrées par des messages alternatifs.
    """

    def __init__(self):
        """Initialise le FilterEngine avec les règles par défaut."""
        self.rules: List[FilterRule] = []
        self._initialize_default_rules()

    def _initialize_default_rules(self):
        """Initialise les règles de filtrage par défaut."""
        # Règles de sécurité (priorité 1)
        self.rules.append(FilterRule(
            name="aqi_too_high_for_mobility",
            priority=FilterPriority.SAFETY,
            condition_func=self._is_aqi_too_high_for_mobility,
            message="Qualité de l'air trop mauvaise pour une activité en extérieur. Essayez une activité en intérieur."
        ))

        self.rules.append(FilterRule(
            name="rain_too_high_for_outdoor",
            priority=FilterPriority.SAFETY,
            condition_func=self._is_rain_too_high_for_outdoor,
            message="Risque de pluie trop élevé pour une activité en extérieur. Essayez une activité en intérieur."
        ))

        self.rules.append(FilterRule(
            name="wind_too_high_for_mobility",
            priority=FilterPriority.SAFETY,
            condition_func=self._is_wind_too_high_for_mobility,
            message="Vent trop fort pour une activité en extérieur. Essayez une activité en intérieur."
        ))

        # Règles légales (priorité 2)
        self.rules.append(FilterRule(
            name="water_restriction_active",
            priority=FilterPriority.LEGAL,
            condition_func=self._is_water_restriction_active,
            message="Restriction d'eau active. Évitez les activités nécessitant de l'eau."
        ))

        # Règles techniques (priorité 3)
        self.rules.append(FilterRule(
            name="missing_critical_data",
            priority=FilterPriority.TECHNICAL,
            condition_func=self._is_missing_critical_data,
            message="Données critiques manquantes. Cette recommandation ne peut pas être validée."
        ))

        # Règles de préférences utilisateur (priorité 4)
        self.rules.append(FilterRule(
            name="duration_exceeds_max",
            priority=FilterPriority.USER_PREFERENCE,
            condition_func=self._is_duration_exceeds_max,
            message="Cette activité dépasse votre durée maximale. Essayez une activité plus courte."
        ))

    def filter_recommendations(
        self,
        recommendations: List[Recommendation],
        context: UnifiedContext,
        user_profile: Optional[UserProfile] = None
    ) -> Tuple[List[Recommendation], List[Dict]]:
        """
        Filtre une liste de recommandations et retourne les recommandations valides et les messages alternatifs.

        Args:
            recommendations: Liste de recommandations à filtrer.
            context: Le contexte unifié.
            user_profile: Le profil utilisateur (optionnel).

        Returns:
            Tuple[List[Recommendation], List[Dict]]: 
                - Liste des recommandations valides.
                - Liste des messages alternatifs (chaque message est un dict avec "original_id", "message", "priority").
        """
        valid_recommendations = []
        alternative_messages = []

        for rec in recommendations:
            filtered = False
            for rule in sorted(self.rules, key=lambda r: r.priority.value):
                message = rule.apply(rec, context, user_profile)
                if message:
                    alternative_messages.append({
                        "original_id": rec.id,
                        "message": message,
                        "priority": rule.priority.name,
                        "category": rec.category.value
                    })
                    filtered = True
                    break
            if not filtered:
                valid_recommendations.append(rec)

        return valid_recommendations, alternative_messages

    # --- Règles de sécurité ---

    def _is_aqi_too_high_for_mobility(self, recommendation: Recommendation, context: UnifiedContext, user_profile: Optional[UserProfile]) -> bool:
        """Vérifie si l'AQI est trop élevé pour une activité de mobilité."""
        aqi = context.derived_data.get("global_aqi", 50)
        if recommendation.category.value in ["mobility", "nature", "wellbeing"]:
            return aqi > 150  # AQI > 150: mauvaise qualité de l'air
        return False

    def _is_rain_too_high_for_outdoor(self, recommendation: Recommendation, context: UnifiedContext, user_profile: Optional[UserProfile]) -> bool:
        """Vérifie si la pluie est trop élevée pour une activité en extérieur."""
        rain_probability = context.get_value("atmo-france", "rain_probability", 0)
        if recommendation.category.value in ["mobility", "nature", "wellbeing"]:
            return rain_probability > 80  # Pluie > 80%
        return False

    def _is_wind_too_high_for_mobility(self, recommendation: Recommendation, context: UnifiedContext, user_profile: Optional[UserProfile]) -> bool:
        """Vérifie si le vent est trop fort pour une activité de mobilité."""
        wind_speed = context.get_value("atmo-france", "wind_speed", 0)
        if recommendation.category.value == "mobility":
            return wind_speed > 30  # Vent > 30 km/h
        return False

    # --- Règles légales ---

    def _is_water_restriction_active(self, recommendation: Recommendation, context: UnifiedContext, user_profile: Optional[UserProfile]) -> bool:
        """Vérifie si une restriction d'eau est active."""
        avg_water_level = context.derived_data.get("avg_water_level", 100)
        if recommendation.category.value == "water" and avg_water_level < 20:
            return True  # Niveau d'eau critique
        return False

    # --- Règles techniques ---

    def _is_missing_critical_data(self, recommendation: Recommendation, context: UnifiedContext, user_profile: Optional[UserProfile]) -> bool:
        """Vérifie si des données critiques sont manquantes."""
        required_data = ["global_aqi"]
        for data in required_data:
            if data not in context.derived_data:
                return True
        return False

    # --- Règles de préférences utilisateur ---

    def _is_duration_exceeds_max(self, recommendation: Recommendation, context: UnifiedContext, user_profile: Optional[UserProfile]) -> bool:
        """Vérifie si la durée de la recommandation dépasse la durée maximale de l'utilisateur."""
        if user_profile is None or recommendation.duration_minutes is None:
            return False
        max_duration = user_profile.preferences.get("max_duration", 120)  # 2 heures par défaut
        return recommendation.duration_minutes > max_duration

    # --- Méthodes utilitaires ---

    def add_rule(self, rule: FilterRule) -> None:
        """Ajoute une règle de filtrage."""
        self.rules.append(rule)

    def remove_rule(self, name: str) -> bool:
        """Supprime une règle de filtrage par son nom."""
        for i, rule in enumerate(self.rules):
            if rule.name == name:
                self.rules.pop(i)
                return True
        return False

    def clear_rules(self) -> None:
        """Supprime toutes les règles de filtrage."""
        self.rules.clear()

    def get_rules_by_priority(self, priority: FilterPriority) -> List[FilterRule]:
        """Retourne les règles de filtrage d'une priorité spécifique."""
        return [rule for rule in self.rules if rule.priority == priority]


### Intégration avec le Recommendation Engine
Dans `backend/engines/recommendation_engine/__init__.py` :
```python
from .rules import rule_engine
from .models import Recommendation
from .scoring import ScoringEngine
from .ranking import RankingEngine
from .filtering import FilterEngine


# Créer une instance globale du RecommendationEngine
class RecommendationEngine:
    """
    Moteur principal pour générer, scorer, filtrer et classer les recommandations.
    """

    def __init__(self):
        self.rule_engine = rule_engine
        self.scoring_engine = ScoringEngine()
        self.filter_engine = FilterEngine()
        self.ranking_engine = RankingEngine(enable_diversity=True)

    def generate_recommendations(
        self,
        context: UnifiedContext,
        user_profile: Optional[UserProfile] = None
    ) -> List[Recommendation]:
        """
        Génère, score, filtre et classe les recommandations.

        Args:
            context: Le contexte unifié.
            user_profile: Le profil utilisateur (optionnel).

        Returns:
            List[Recommendation]: Liste de recommandations générées, scorées, filtrées et classées.
        """
        # 1. Générer les candidats avec le RuleEngine
        candidates = self.rule_engine.evaluate_all(context, user_profile)

        # 2. Calculer les scores avec le ScoringEngine
        scored_candidates = self.scoring_engine.calculate_scores(candidates, context, user_profile)

        # 3. Filtrer les recommandations invalides avec le FilterEngine
        valid_recommendations, alternative_messages = self.filter_engine.filter_recommendations(
            scored_candidates, context, user_profile
        )

        # 4. Classer les recommandations avec le RankingEngine
        ranked_recommendations = self.ranking_engine.rank_recommendations(valid_recommendations)

        return ranked_recommendations

    def get_alternative_messages(
        self,
        context: UnifiedContext,
        user_profile: Optional[UserProfile] = None
    ) -> List[Dict]:
        """
        Retourne les messages alternatifs pour les recommandations filtrées.

        Args:
            context: Le contexte unifié.
            user_profile: Le profil utilisateur (optionnel).

        Returns:
            List[Dict]: Liste des messages alternatifs.
        """
        candidates = self.rule_engine.evaluate_all(context, user_profile)
        scored_candidates = self.scoring_engine.calculate_scores(candidates, context, user_profile)
        _, alternative_messages = self.filter_engine.filter_recommendations(
            scored_candidates, context, user_profile
        )
        return alternative_messages


# Instance globale
recommendation_engine = RecommendationEngine()
```

---

## Tests Unitaires
Créer un fichier `tests/unit/test_filtering.py` avec les tests suivants :

```python
import pytest
from datetime import datetime, timedelta
from unittest.mock import AsyncMock, patch, MagicMock
import sys
import os

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '../../backend'))

from backend.engines.recommendation_engine.models import Recommendation, RecommendationCategory, UnifiedContext, UserProfile
from backend.engines.recommendation_engine.filtering import FilterEngine, FilterRule, FilterPriority


@pytest.fixture
def filter_engine():
    return FilterEngine()


@pytest.fixture
def mock_recommendation_mobility():
    return Recommendation(
        id="rec_001",
        title="Balade à vélo",
        description="Profitez de la bonne qualité de l'air.",
        category=RecommendationCategory.MOBILITY,
        action="Faire du vélo",
        duration_minutes=30
    )


@pytest.fixture
def mock_recommendation_water():
    return Recommendation(
        id="rec_002",
        title="Arroser les plantes",
        description="Niveau d'eau bas.",
        category=RecommendationCategory.WATER,
        action="Arroser les plantes",
        duration_minutes=15
    )


@pytest.fixture
def mock_unified_context():
    return UnifiedContext(
        observations=[],
        timestamp=datetime.now(),
        derived_data={
            "global_aqi": 42,
            "avg_water_level": 120.5
        }
    )


@pytest.fixture
def mock_unified_context_bad_aqi():
    return UnifiedContext(
        observations=[],
        timestamp=datetime.now(),
        derived_data={
            "global_aqi": 200,  # AQI très élevé
            "avg_water_level": 120.5
        }
    )


@pytest.fixture
def mock_unified_context_low_water():
    return UnifiedContext(
        observations=[],
        timestamp=datetime.now(),
        derived_data={
            "global_aqi": 42,
            "avg_water_level": 10  # Niveau d'eau critique
        }
    )


@pytest.fixture
def mock_user_profile():
    return UserProfile(
        user_id="user123",
        location="69003",
        preferences={
            "max_duration": 60
        }
    )


# Test de filtrage pour AQI trop élevé
def test_filter_aqi_too_high(filter_engine, mock_recommendation_mobility, mock_unified_context_bad_aqi):
    valid, alternatives = filter_engine.filter_recommendations(
        [mock_recommendation_mobility], mock_unified_context_bad_aqi, None
    )
    assert len(valid) == 0
    assert len(alternatives) == 1
    assert alternatives[0]["original_id"] == "rec_001"
    assert "Qualité de l'air trop mauvaise" in alternatives[0]["message"]


# Test de filtrage pour restriction d'eau
def test_filter_water_restriction(filter_engine, mock_recommendation_water, mock_unified_context_low_water):
    valid, alternatives = filter_engine.filter_recommendations(
        [mock_recommendation_water], mock_unified_context_low_water, None
    )
    assert len(valid) == 0
    assert len(alternatives) == 1
    assert alternatives[0]["original_id"] == "rec_002"
    assert "Restriction d'eau active" in alternatives[0]["message"]


# Test de filtrage pour durée dépassée
def test_filter_duration_exceeds_max(filter_engine, mock_recommendation_mobility, mock_unified_context, mock_user_profile):
    mock_recommendation_mobility.duration_minutes = 120  # Durée > max_duration (60)
    valid, alternatives = filter_engine.filter_recommendations(
        [mock_recommendation_mobility], mock_unified_context, mock_user_profile
    )
    assert len(valid) == 0
    assert len(alternatives) == 1
    assert alternatives[0]["original_id"] == "rec_001"
    assert "dépasse votre durée maximale" in alternatives[0]["message"]


# Test de filtrage pour recommandation valide
def test_filter_valid_recommendation(filter_engine, mock_recommendation_mobility, mock_unified_context, mock_user_profile):
    valid, alternatives = filter_engine.filter_recommendations(
        [mock_recommendation_mobility], mock_unified_context, mock_user_profile
    )
    assert len(valid) == 1
    assert len(alternatives) == 0
    assert valid[0].id == "rec_001"


# Test de filtrage pour plusieurs recommandations
def test_filter_multiple_recommendations(filter_engine, mock_unified_context, mock_user_profile):
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
        Recommendation(
            id="rec_003",
            title="Balade en forêt",
            description="Explorez les sentiers locaux.",
            category=RecommendationCategory.NATURE,
            action="Balade en forêt",
            duration_minutes=45
        ),
    ]

    valid, alternatives = filter_engine.filter_recommendations(recommendations, mock_unified_context, mock_user_profile)
    assert len(valid) == 3
    assert len(alternatives) == 0


# Test de priorité des règles
def test_rule_priority(filter_engine, mock_recommendation_mobility, mock_unified_context_bad_aqi):
    # Ajouter une règle de priorité inférieure (ex: préférence utilisateur)
    filter_engine.add_rule(FilterRule(
        name="test_rule",
        priority=FilterPriority.USER_PREFERENCE,
        condition_func=lambda rec, ctx, profile: True,  # Toujours vrai
        message="Règle de test"
    ))

    # L'AQI trop élevé (priorité SAFETY) doit être appliqué en premier
    valid, alternatives = filter_engine.filter_recommendations(
        [mock_recommendation_mobility], mock_unified_context_bad_aqi, None
    )
    assert len(valid) == 0
    assert len(alternatives) == 1
    assert "Qualité de l'air trop mauvaise" in alternatives[0]["message"]


# Test de déterminisme
def test_deterministic_filtering(filter_engine, mock_recommendation_mobility, mock_unified_context, mock_user_profile):
    recommendations = [mock_recommendation_mobility]

    valid1, alternatives1 = filter_engine.filter_recommendations(recommendations, mock_unified_context, mock_user_profile)
    valid2, alternatives2 = filter_engine.filter_recommendations(recommendations, mock_unified_context, mock_user_profile)

    assert len(valid1) == len(valid2)
    assert len(alternatives1) == len(alternatives2)
    for v1, v2 in zip(valid1, valid2):
        assert v1.id == v2.id


# Test de suppression de règle
def test_remove_rule(filter_engine):
    initial_count = len(filter_engine.rules)
    assert filter_engine.remove_rule("aqi_too_high_for_mobility") is True
    assert len(filter_engine.rules) == initial_count - 1


# Test de suppression de règle inexistante
def test_remove_nonexistent_rule(filter_engine):
    assert filter_engine.remove_rule("nonexistent_rule") is False


# Test de récupération des règles par priorité
def test_get_rules_by_priority(filter_engine):
    safety_rules = filter_engine.get_rules_by_priority(FilterPriority.SAFETY)
    assert len(safety_rules) > 0
    for rule in safety_rules:
        assert rule.priority == FilterPriority.SAFETY


# Test de vidage des règles
def test_clear_rules(filter_engine):
    filter_engine.clear_rules()
    assert len(filter_engine.rules) == 0
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
- **Sécurité d'abord** : Les règles de **sécurité** (ex: conditions météo dangereuses) sont appliquées en **premier** et ont la priorité la plus élevée.
- **Restrictions légales** : Les règles légales (ex: restriction d'eau) sont appliquées après les règles de sécurité.
- **Contraintes techniques** : Les règles techniques (ex: données manquantes) sont appliquées après les règles légales.
- **Préférences utilisateur** : Les règles de préférences utilisateur sont appliquées en dernier.
- **Déterminisme** : Le filtrage est **100% déterministe** (mêmes entrées → mêmes résultats).
- **Messages alternatifs** : Les recommandations filtrées sont **remplacées par des messages alternatifs** pour informer l'utilisateur.

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine import recommendation_engine
from backend.engines.context_engine.models import UnifiedContext, UserProfile

# Générer, scorer, filtrer et classer les recommandations
context = UnifiedContext(...)  # Contexte unifié
user_profile = UserProfile(...)  # Profil utilisateur

recommendations = recommendation_engine.generate_recommendations(context, user_profile)

# Afficher les recommandations valides
for rec in recommendations:
    print(f"{rec.title} (Score: {rec.score:.2f}, Catégorie: {rec.category.value})")

# Afficher les messages alternatifs pour les recommandations filtrées
alternative_messages = recommendation_engine.get_alternative_messages(context, user_profile)
for msg in alternative_messages:
    print(f"Alternative: {msg['message']} (Priorité: {msg['priority']})")
```

---

## Ressources
- [PRD Almanéa - FR-9](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-5](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
