---
name: Associer les preuves aux recommandations
id: 3-2-associer-les-preuves-aux-recommandations
epic: epic-3
story_type: backend
priority: high
estimation: M
dependencies: [3-1-stocker-les-preuves-scientifiques]
status: done
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 3.2: Associer les preuves scientifiques aux recommandations

## Contexte
Cette story fait partie de **l'Épic 3 (Knowledge Engine)**. Son objectif est d'**associer les preuves scientifiques (Evidence)** aux recommandations générées par le `RuleEngine`, afin d'**expliquer pourquoi une recommandation est pertinente** (ex: "Pourquoi éviter de sortir aujourd'hui ? → Parce que la qualité de l'air est mauvaise, selon l'OMS").

Le `RecommendationEvidenceBuilder` permet de lier des preuves scientifiques aux recommandations, et de générer des **explications textuelles dynamiques** en combinant les données des preuves et du contexte.

---

## Exigences Fonctionnelles
- **FR-12**: Associer des preuves scientifiques aux recommandations pour renforcer leur crédibilité.
- **FR-13**: Générer des explications textuelles basées sur les preuves associées.

---

## Critères d'Acceptation
1. **Association des preuves aux recommandations** :
   - [x] Une recommandation peut être associée à une ou plusieurs preuves scientifiques via `evidence_ids`.
   - [x] Les preuves sont validées avant association (ex: vérifier qu'elles existent dans le `KnowledgeEngine`).

2. **Génération d'explications** :
   - [x] Une explication textuelle est générée automatiquement à partir des preuves associées.
   - [x] L'explication inclut le `claim` de la preuve, la source, et le niveau de confiance.

3. **Recherche de preuves par sujet** :
   - [x] Les preuves peuvent être associées automatiquement en fonction des `topics` de la recommandation.
   - [x] Priorisation des preuves de niveau `HIGH` pour les explications.

4. **Intégration avec le RecommendationEngine** :
   - [x] Le `RecommendationEngine` utilise le `RecommendationEvidenceBuilder` pour associer les preuves.
   - [x] Les explications sont générées pour chaque recommandation.

---

## Implémentation Technique

### Fichiers créés/modifiés
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/models.py` | Ajout de `evidence_ids` et `explanation` au modèle `Recommendation` | ✅ Modifié |
| `backend/engines/recommendation_engine/recommendation_builder.py` | Module `RecommendationEvidenceBuilder` | ✅ Créé |
| `tests/unit/test_recommendation_evidence.py` | Tests unitaires | ✅ Créé |

### Modèle `Recommendation` (modifié)
Le modèle `Recommendation` a été étendu pour inclure :
- `evidence_ids`: Liste des IDs des preuves scientifiques associées.
- `explanation`: Explication textuelle générée à partir des preuves.

```python
class Recommendation(BaseModel):
    # ... (champs existants)
    evidence_ids: List[str] = Field(
        default_factory=list,
        description="Liste des IDs des preuves scientifiques associées"
    )
    explanation: Optional[str] = Field(
        None,
        description="Explication textuelle générée à partir des preuves"
    )
```

### Module `RecommendationEvidenceBuilder`
Ce module permet de :
1. **Associer des preuves** à une recommandation.
2. **Générer des explications** à partir des preuves.
3. **Rechercher des preuves par sujet** ou niveau de confiance.

```python
class RecommendationEvidenceBuilder:
    def associate_evidence_to_recommendation(self, recommendation: Recommendation, evidence_ids: List[str]) -> Recommendation:
        """Associe des preuves à une recommandation et génère une explication."""
        pass

    def get_evidences_for_recommendation(self, recommendation: Recommendation) -> List[Evidence]:
        """Récupère les preuves associées à une recommandation."""
        pass

    def associate_evidence_by_topic(self, recommendation: Recommendation, topics: List[str], max_evidences: int = 3) -> Recommendation:
        """Associe automatiquement des preuves par sujet."""
        pass

    def associate_high_confidence_evidence(self, recommendation: Recommendation, topics: List[str], max_evidences: int = 2) -> Recommendation:
        """Associe automatiquement des preuves de niveau HIGH."""
        pass
```

---

## Tests Unitaires
11 tests ont été créés dans `tests/unit/test_recommendation_evidence.py` pour valider :
- L'association de preuves à une recommandation.
- La génération d'explications.
- La recherche de preuves par sujet ou niveau de confiance.
- La validation des preuves avant association.
- Le déterminisme du module.

---

## Configuration Requise
- **Dépendances** : `pydantic`, `typing`.
- **Intégration** : Utilise le `KnowledgeEngine` pour récupérer les preuves.

---

## Dépendances
- **Story 3-1** : `stocker-les-preuves-scientifiques` (doit être implémentée avant).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine.recommendation_builder import recommendation_evidence_builder
from backend.engines.recommendation_engine.models import Recommendation, RecommendationCategory

# Créer une recommandation
recommendation = Recommendation(
    id="rec_001",
    title="Éviter les sorties en vélo",
    description="La qualité de l'air est mauvaise aujourd'hui.",
    category=RecommendationCategory.MOBILITY,
    action="Éviter les sorties en vélo",
    conditions={"aqi": 80}
)

# Associer des preuves par sujet
updated_recommendation = recommendation_evidence_builder.associate_evidence_by_topic(
    recommendation,
    topics=["air_quality", "health"],
    max_evidences=2
)

print(updated_recommendation.explanation)
# Output: "Pourquoi ? La qualité de l'air a un impact direct sur la santé respiratoire. — Source : OMS (Rapport mondial sur la qualité de l'air)."
```

---

## Notes
- **Preuves scientifiques** : Les preuves sont récupérées depuis le `KnowledgeEngine`.
- **Explications dynamiques** : Les explications sont générées en combinant les `claims` des preuves et leurs sources.
- **Priorisation** : Les preuves de niveau `HIGH` sont prioritaires pour les explications.
- **Validation** : Les preuves sont validées avant d'être associées à une recommandation.

---

## Ressources
- [PRD Almanéa - FR-12](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-5](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
