---
name: Implémenter LLMProvider
id: 4-1-implémenter-llmprovider
epic: epic-4
story_type: backend
priority: high
estimation: M
dependencies: []
status: done
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 4.1: Implémenter LLMProvider

## Contexte
Cette story fait partie de **l'Épic 4 (Génération d'explications dynamiques)**. Son objectif est de **créer une interface abstraite pour les fournisseurs de modèles de langage (LLM)**, afin de générer des **explications textuelles dynamiques** pour les recommandations.

Le `LLMProvider` permet de :
- Générer des explications personnalisées pour les recommandations.
- Intégrer différents fournisseurs de LLM (ex: OpenAI, Mistral, local).
- **Fournir un fallback déterministe** en cas d'échec du LLM (via `MockLLMProvider`).

---

## Exigences Fonctionnelles
- **FR-20**: Générer des explications textuelles dynamiques pour les recommandations.
- **FR-21**: Supporter plusieurs fournisseurs de LLM (extensibilité).
- **FR-22**: Garantir un fonctionnement déterministe en mode dégradé (fallback).

---

## Critères d'Acceptation
1. **Interface abstraite `LLMProvider`** :
   - [x] Définir une interface abstraite avec les méthodes `generate_explanation` et `generate_custom_prompt`.
   - [x] Permettre l'intégration de différents fournisseurs de LLM.

2. **Fournisseur LLM mock** :
   - [x] Implémenter un `MockLLMProvider` pour la phase PoC.
   - [x] Générer des explications basées sur des templates prédéfinis.

3. **Intégration avec le RecommendationEngine** :
   - [x] Le `RecommendationEngine` utilise le `LLMProvider` pour générer des explications.
   - [x] Le `LLMProvider` est accessible via une instance globale.

4. **Extensibilité** :
   - [x] Permettre l'ajout de nouveaux fournisseurs de LLM (ex: OpenAI, Mistral).
   - [x] Supporter le passage de contexte unifié pour enrichir les explications.

---

## Implémentation Technique

### Fichiers créés/modifiés
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/llm/__init__.py` | Export du `LLMProvider` et instance globale | ✅ Créé |
| `backend/engines/recommendation_engine/llm/base.py` | Interface abstraite `LLMProvider` | ✅ Créé |
| `backend/engines/recommendation_engine/llm/mock_llm.py` | Fournisseur LLM mock | ✅ Créé |
| `tests/unit/test_llm_provider.py` | Tests unitaires | ✅ Créé |

### Interface `LLMProvider` (`base.py`)
```python
from abc import ABC, abstractmethod
from typing import Optional
from backend.engines.context_engine.models import UnifiedContext
from backend.engines.recommendation_engine.models import Recommendation

class LLMProvider(ABC):
    @abstractmethod
    def generate_explanation(
        self,
        recommendation: Recommendation,
        context: Optional[UnifiedContext] = None
    ) -> str:
        """Génère une explication textuelle pour une recommandation."""
        pass

    @abstractmethod
    def generate_custom_prompt(
        self,
        prompt: str,
        context: Optional[UnifiedContext] = None
    ) -> str:
        """Génère une réponse à partir d'un prompt personnalisé."""
        pass
```

### Fournisseur `MockLLMProvider` (`mock_llm.py`)
```python
from typing import Optional
from backend.engines.context_engine.models import UnifiedContext
from backend.engines.recommendation_engine.models import Recommendation
from .base import LLMProvider

class MockLLMProvider(LLMProvider):
    def __init__(self):
        self.templates = {
            "default": "Cette recommandation est basée sur les données actuelles : {conditions}.",
            "mobility": "En fonction des conditions actuelles ({conditions}), il est recommandé de {action} pour optimiser votre mobilité.",
            "health": "Pour préserver votre santé, {action} en raison des conditions suivantes : {conditions}.",
            "energy": "Pour réduire votre empreinte énergétique, {action}. Cela est particulièrement pertinent aujourd'hui en raison de : {conditions}.",
        }

    def generate_explanation(self, recommendation: Recommendation, context: Optional[UnifiedContext] = None) -> str:
        category = recommendation.category.value
        template = self.templates.get(category, self.templates["default"])
        conditions_str = ", ".join(f"{k}={v}" for k, v in recommendation.conditions.items())
        return template.format(action=recommendation.action.lower(), conditions=conditions_str)

    def generate_custom_prompt(self, prompt: str, context: Optional[UnifiedContext] = None) -> str:
        return f"Réponse générée pour le prompt : '{prompt}' (Mode mock - à remplacer par un vrai LLM)."
```

### Intégration dans `RecommendationEngine`
Le `LLMProvider` est intégré dans le `RecommendationEngine` pour générer des explications dynamiques :
```python
from .llm import llm_provider

class RecommendationEngine:
    def __init__(self):
        self.llm_provider = llm_provider

    def generate_recommendations(self, context: UnifiedContext, user_profile: Optional[UserProfile] = None) -> List[Recommendation]:
        # ... (génération des recommandations)
        for recommendation in ranked_recommendations:
            recommendation.explanation = self.llm_provider.generate_explanation(recommendation, context)
        return ranked_recommendations
```

---

## Tests Unitaires
10 tests ont été créés dans `tests/unit/test_llm_provider.py` pour valider :
- La génération d'explications pour différentes catégories de recommandations.
- La génération de réponses à partir de prompts personnalisés.
- Le déterminisme du `MockLLMProvider`.
- L'initialisation et la compatibilité avec le contexte unifié.

---

## Configuration Requise
- **Dépendances** : Aucune dépendance supplémentaire (utilise `typing` et `enum`).
- **Extensibilité** : Pour ajouter un nouveau fournisseur de LLM, il suffit de créer une classe qui hérite de `LLMProvider`.

---

## Dépendances
Aucune dépendance spécifique. Utilise les modules existants (`Recommendation`, `UnifiedContext`).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine.llm import llm_provider
from backend.engines.recommendation_engine.models import Recommendation, RecommendationCategory

# Créer une recommandation
recommendation = Recommendation(
    id="rec_001",
    title="Éviter les sorties en vélo",
    description="La qualité de l'air est mauvaise aujourd'hui.",
    category=RecommendationCategory.MOBILITY,
    action="Éviter les sorties en vélo",
    conditions={"aqi": 80, "polluant": "PM2.5"}
)

# Générer une explication
explanation = llm_provider.generate_explanation(recommendation)
print(explanation)
# Output: "En fonction des conditions actuelles (aqi=80, polluant=PM2.5), il est recommandé de éviter les sorties en vélo pour optimiser votre mobilité."

# Générer une réponse à un prompt personnalisé
response = llm_provider.generate_custom_prompt("Pourquoi devrais-je éviter de sortir aujourd'hui ?")
print(response)
# Output: "Réponse générée pour le prompt : 'Pourquoi devrais-je éviter de sortir aujourd'hui ?' (Mode mock - à remplacer par un vrai LLM)."
```

---

## Notes
- **Phase PoC** : Le `MockLLMProvider` est utilisé par défaut pour la phase de preuve de concept.
- **Extensibilité** : Pour la phase GA, il faudra implémenter des fournisseurs concrets (ex: `OpenAIProvider`, `MistralProvider`).
- **Fallback** : Un mécanisme de fallback est implémenté dans le `FallbackLLMProvider` (voir **Story 4-3**).
- **Déterminisme** : Le `MockLLMProvider` est déterministe, ce qui garantit des tests reproductibles.

---

## Ressources
- [PRD Almanéa - FR-20](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-6](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
