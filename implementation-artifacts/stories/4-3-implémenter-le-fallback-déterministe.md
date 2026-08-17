---
name: Implémenter le fallback déterministe
id: 4-3-implémenter-le-fallback-déterministe
epic: epic-4
story_type: backend
priority: high
estimation: S
dependencies: [4-1-implémenter-llmprovider]
status: done
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 4.3: Implémenter le fallback déterministe

## Contexte
Cette story fait partie de **l'Épic 4 (Génération d'explications dynamiques)**. Son objectif est de **garantir que le système peut toujours générer des explications**, même si le **LLM principal échoue** (ex: API indisponible, erreur réseau, quota dépassé).

Le `FallbackLLMProvider` permet de :
- **Bascule automatiquement** sur un fournisseur de fallback (ex: `MockLLMProvider`) en cas d'échec.
- **Garantir un fonctionnement déterministe** en mode dégradé.
- **Maintenir la disponibilité** du service même en cas de panne du LLM principal.

---

## Exigences Fonctionnelles
- **FR-26**: Garantir la disponibilité des explications même en cas d'échec du LLM principal.
- **FR-27**: Basculer automatiquement sur un fournisseur de fallback en cas d'erreur.
- **FR-28**: Maintenir un comportement déterministe en mode dégradé.

---

## Critères d'Acceptation
1. **Mécanisme de fallback** :
   - [x] Le `FallbackLLMProvider` tente d'abord d'utiliser le fournisseur principal.
   - [x] Si le fournisseur principal échoue, il bascule automatiquement sur le fournisseur de fallback.

2. **Configuration flexible** :
   - [x] Permet de définir un fournisseur principal personnalisé.
   - [x] Permet de définir un fournisseur de fallback personnalisé.

3. **Transparence** :
   - [x] Le basculement sur le fallback est transparent pour l'utilisateur.
   - [x] Les explications générées en mode fallback sont cohérentes.

4. **Tests de résilience** :
   - [x] Vérifier que le système reste fonctionnel même si le LLM principal échoue.
   - [x] Vérifier que le fallback est utilisé correctement.

---

## Implémentation Technique

### Fichiers créés/modifiés
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/llm/fallback_llm.py` | Module `FallbackLLMProvider` | ✅ Créé |
| `backend/engines/recommendation_engine/llm/__init__.py` | Mise à jour de l'export | ✅ Modifié |
| `tests/unit/test_fallback_llm.py` | Tests unitaires | ✅ Créé |

### Module `FallbackLLMProvider`
Ce module implémente un **fournisseur LLM avec fallback** :
- **Fournisseur principal** : Par défaut, utilise `MockLLMProvider`.
- **Fournisseur de fallback** : Utilise `MockLLMProvider` en cas d'échec.
- **Mécanisme de basculement** : Si le fournisseur principal lève une exception, le fallback est utilisé.

```python
from typing import Optional
from backend.engines.context_engine.models import UnifiedContext
from backend.engines.recommendation_engine.models import Recommendation
from .base import LLMProvider
from .mock_llm import MockLLMProvider

class FallbackLLMProvider(LLMProvider):
    def __init__(self, primary_provider: Optional[LLMProvider] = None):
        """Initialise avec un fournisseur principal et un fallback."""
        self.primary_provider = primary_provider if primary_provider else MockLLMProvider()
        self.fallback_provider = MockLLMProvider()

    def generate_explanation(
        self,
        recommendation: Recommendation,
        context: Optional[UnifiedContext] = None
    ) -> str:
        """Génère une explication avec fallback."""
        try:
            return self.primary_provider.generate_explanation(recommendation, context)
        except Exception:
            return self.fallback_provider.generate_explanation(recommendation, context)

    def generate_custom_prompt(
        self,
        prompt: str,
        context: Optional[UnifiedContext] = None
    ) -> str:
        """Génère une réponse à un prompt avec fallback."""
        try:
            return self.primary_provider.generate_custom_prompt(prompt, context)
        except Exception:
            return self.fallback_provider.generate_custom_prompt(prompt, context)

    def set_primary_provider(self, provider: LLMProvider) -> None:
        """Définit le fournisseur principal."""
        self.primary_provider = provider

    def set_fallback_provider(self, provider: LLMProvider) -> None:
        """Définit le fournisseur de fallback."""
        self.fallback_provider = provider
```

### Mise à jour de l'export dans `llm/__init__.py`
L'instance globale `llm_provider` utilise maintenant le `FallbackLLMProvider` :
```python
from .base import LLMProvider
from .mock_llm import MockLLMProvider
from .fallback_llm import FallbackLLMProvider

# Instance globale (par défaut : FallbackLLMProvider avec MockLLMProvider comme primary et fallback)
llm_provider = FallbackLLMProvider()
```

---

## Tests Unitaires
9 tests ont été créés dans `tests/unit/test_fallback_llm.py` pour valider :
- La génération d'explications avec le fournisseur principal.
- Le basculement sur le fallback en cas d'échec.
- La génération de réponses à des prompts personnalisés avec fallback.
- Le changement dynamique des fournisseurs principal et de fallback.
- Le déterminisme du module.
- Le fonctionnement après plusieurs échecs.

---

## Configuration Requise
- **Dépendances** : Aucune dépendance supplémentaire (utilise `typing` et `enum`).
- **Intégration** : Remplace l'instance globale `llm_provider` dans `llm/__init__.py`.

---

## Dépendances
- **Story 4-1** : `implémenter-llmprovider` (doit être implémentée avant).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine.llm import FallbackLLMProvider, MockLLMProvider
from backend.engines.recommendation_engine.models import Recommendation, RecommendationCategory

# Créer un fournisseur principal qui échoue
class FailingLLMProvider(LLMProvider):
    def generate_explanation(self, recommendation, context=None):
        raise Exception("LLM API Error")
    def generate_custom_prompt(self, prompt, context=None):
        raise Exception("LLM API Error")

# Initialiser le FallbackLLMProvider avec le fournisseur principal qui échoue
fallback_provider = FallbackLLMProvider(primary_provider=FailingLLMProvider())

# Générer une explication (utilisera le fallback)
recommendation = Recommendation(
    id="rec_001",
    title="Éviter les sorties en vélo",
    description="La qualité de l'air est mauvaise aujourd'hui.",
    category=RecommendationCategory.MOBILITY,
    action="Éviter les sorties en vélo",
    conditions={"aqi": 80}
)

explanation = fallback_provider.generate_explanation(recommendation)
print(explanation)
# Output: "En fonction des conditions actuelles (aqi=80), il est recommandé de éviter les sorties en vélo pour optimiser votre mobilité."
```

---

## Notes
- **Résilience** : Le `FallbackLLMProvider` garantit que le système reste fonctionnel même en cas de panne du LLM principal.
- **Extensibilité** : Permet de remplacer facilement le fournisseur principal ou de fallback.
- **Déterminisme** : En mode fallback, le comportement est déterministe (grâce au `MockLLMProvider`).
- **Transparence** : Le basculement sur le fallback est transparent pour l'utilisateur final.

---

## Ressources
- [PRD Almanéa - FR-26](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-6](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
