---
name: Générer des explications textuelles
id: 4-2-générer-des-explications-textuelles
epic: epic-4
story_type: backend
priority: high
estimation: M
dependencies: [4-1-implémenter-llmprovider, 3-2-associer-les-preuves-aux-recommandations]
status: done
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 4.2: Générer des explications textuelles

## Contexte
Cette story fait partie de **l'Épic 4 (Génération d'explications dynamiques)**. Son objectif est de **générer des explications textuelles complètes** pour les recommandations, en combinant :
- Les données du **contexte unifié** (ex: AQI, mix énergétique).
- Les **preuves scientifiques** associées (via `RecommendationEvidenceBuilder`).
- Les **préférences utilisateur** (si disponibles).

Le `ExplanationGenerator` centralise cette logique et permet de générer des explications **dynamiques, personnalisées et riches en informations**.

---

## Exigences Fonctionnelles
- **FR-23**: Générer des explications textuelles pour chaque recommandation.
- **FR-24**: Combiner les données du contexte, les preuves scientifiques et les préférences utilisateur.
- **FR-25**: Permettre des explications personnalisées via des prompts spécifiques.

---

## Critères d'Acceptation
1. **Génération d'explications dynamiques** :
   - [x] Le `ExplanationGenerator` génère des explications en combinant le LLM et les preuves scientifiques.
   - [x] Les explications incluent les `claims` des preuves, leurs sources et niveaux de confiance.

2. **Intégration avec le LLMProvider** :
   - [x] Utilise le `LLMProvider` pour générer une base d'explication.
   - [x] Enrichit l'explication avec les preuves scientifiques associées.

3. **Personnalisation des explications** :
   - [x] Permet de générer des explications personnalisées via `generate_custom_explanation`.
   - [x] Supporte le passage de contexte unifié pour enrichir les explications.

4. **Intégration avec le RecommendationEngine** :
   - [x] Le `RecommendationEngine` utilise le `ExplanationGenerator` pour générer des explications.
   - [x] Les explications sont générées automatiquement pour chaque recommandation.

5. **Fallback déterministe** :
   - [x] Si le LLM échoue, le `MockLLMProvider` garantit une explication statique.

---

## Implémentation Technique

### Fichiers créés/modifiés
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/explanation_generator.py` | Module `ExplanationGenerator` | ✅ Créé |
| `backend/engines/recommendation_engine/__init__.py` | Intégration du `ExplanationGenerator` | ✅ Modifié |
| `tests/unit/test_explanation_generator.py` | Tests unitaires | ✅ Créé |

### Module `ExplanationGenerator`
Ce module permet de :
1. **Générer des explications complètes** pour les recommandations.
2. **Combiner le LLM et les preuves scientifiques**.
3. **Personnaliser les explications** via des prompts.

```python
from typing import Optional, List
from backend.engines.context_engine.models import UnifiedContext
from backend.engines.recommendation_engine.models import Recommendation
from backend.engines.knowledge_engine import knowledge_engine
from backend.engines.knowledge_engine.models import Evidence
from .llm import llm_provider

class ExplanationGenerator:
    def __init__(self):
        self.llm_provider = llm_provider
        self.knowledge_engine = knowledge_engine

    def generate_explanation(
        self,
        recommendation: Recommendation,
        context: Optional[UnifiedContext] = None
    ) -> str:
        """Génère une explication complète pour une recommandation."""
        if recommendation.explanation:
            return recommendation.explanation
        return self._generate_dynamic_explanation(recommendation, context)

    def _generate_dynamic_explanation(
        self,
        recommendation: Recommendation,
        context: Optional[UnifiedContext] = None
    ) -> str:
        """Génère une explication dynamique en combinant LLM et preuves."""
        llm_explanation = self.llm_provider.generate_explanation(recommendation, context)
        evidences = self._get_evidences_for_recommendation(recommendation)
        if evidences:
            evidence_summaries = self._summarize_evidences(evidences)
            llm_explanation += f" {evidence_summaries}"
        return llm_explanation

    def generate_custom_explanation(
        self,
        recommendation: Recommendation,
        context: Optional[UnifiedContext] = None,
        custom_prompt: Optional[str] = None
    ) -> str:
        """Génère une explication personnalisée à partir d'un prompt."""
        if not custom_prompt:
            custom_prompt = f"Explique pourquoi {recommendation.action} est pertinent."
        return self.llm_provider.generate_custom_prompt(custom_prompt, context)

    def _get_evidences_for_recommendation(self, recommendation: Recommendation) -> List[Evidence]:
        """Récupère les preuves associées à une recommandation."""
        return [
            self.knowledge_engine.get_evidence(evidence_id)
            for evidence_id in recommendation.evidence_ids
            if self.knowledge_engine.get_evidence(evidence_id)
        ]

    def _summarize_evidences(self, evidences: List[Evidence]) -> str:
        """Résume les preuves pour les inclure dans l'explication."""
        return " \n".join(
            f"Preuve : {evidence.claim} (Source : {evidence.source.publisher}, Niveau : {evidence.evidenceLevel.value})"
            for evidence in evidences
        )
```

### Intégration dans `RecommendationEngine`
Le `ExplanationGenerator` est intégré dans le `RecommendationEngine` :
```python
from .explanation_generator import explanation_generator

class RecommendationEngine:
    def __init__(self):
        self.explanation_generator = explanation_generator

    def generate_recommendations(
        self,
        context: UnifiedContext,
        user_profile: Optional[UserProfile] = None,
        generate_explanations: bool = True
    ) -> List[Recommendation]:
        # ... (génération des recommandations)
        if generate_explanations:
            for recommendation in ranked_recommendations:
                recommendation.explanation = self.explanation_generator.generate_explanation(
                    recommendation, context
                )
        return ranked_recommendations
```

---

## Tests Unitaires
11 tests ont été créés dans `tests/unit/test_explanation_generator.py` pour valider :
- La génération d'explications de base.
- La génération d'explications avec preuves associées.
- La génération d'explications personnalisées.
- La récupération des preuves pour une recommandation.
- Le résumé des preuves.
- Le déterminisme du module.

---

## Configuration Requise
- **Dépendances** : Utilise le `LLMProvider` et le `KnowledgeEngine`.
- **Intégration** : Doit être utilisé après l'association des preuves aux recommandations (voir **Story 3-2**).

---

## Dépendances
- **Story 3-2** : `associer-les-preuves-aux-recommandations` (pour l'association des preuves).
- **Story 4-1** : `implémenter-llmprovider` (pour la génération d'explications via LLM).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine import recommendation_engine
from backend.engines.context_engine.models import UnifiedContext
from backend.engines.recommendation_engine.models import Recommendation, RecommendationCategory

# Créer un contexte unifié
context = UnifiedContext(
    observations=[...],
    timestamp=datetime.now()
)

# Générer des recommandations avec explications
recommendations = recommendation_engine.generate_recommendations(
    context=context,
    generate_explanations=True
)

for recommendation in recommendations:
    print(f"Recommandation : {recommendation.title}")
    print(f"Explication : {recommendation.explanation}")
    # Output: "Recommandation : Éviter les sorties en vélo"
    #         "Explication : En fonction des conditions actuelles (aqi=80, polluant=PM2.5), il est recommandé de éviter les sorties en vélo pour optimiser votre mobilité. Preuve : La qualité de l'air a un impact direct sur la santé respiratoire. (Source : OMS, Niveau : high)"
```

---

## Notes
- **Explications dynamiques** : Les explications sont générées en combinant le LLM et les preuves scientifiques.
- **Personnalisation** : Les explications peuvent être personnalisées via des prompts spécifiques.
- **Fallback** : Si le LLM échoue, le `MockLLMProvider` garantit une explication statique.
- **Déterminisme** : Le module est déterministe en mode mock, ce qui facilite les tests.

---

## Ressources
- [PRD Almanéa - FR-23](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-6](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
