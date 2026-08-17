---
name: Stocker les preuves scientifiques
id: 3-1-stocker-les-preuves-scientifiques
epic: epic-3
story_type: backend
priority: high
estimation: M
dependencies: []
status: ready-for-dev
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 3.1: Stocker les preuves scientifiques

## Contexte
Cette story fait partie de **l'Épic 3 (Knowledge Engine)**. Son objectif est de **stocker et récupérer des preuves scientifiques (Evidence)** pour expliquer les recommandations, conformément à **FR-11 (Gestion des preuves scientifiques)**. Les preuves scientifiques permettent de **renforcer la crédibilité** des recommandations en les associant à des sources fiables et vérifiables.

Le `KnowledgeEngine` gère un **catalogue de preuves scientifiques** qui peuvent être associées aux recommandations pour expliquer pourquoi une action est pertinente (ex: "La qualité de l'air a un impact direct sur la santé respiratoire — Source : OMS, 2023").

---

## Exigences Fonctionnelles
- **FR-11**: Gestion des preuves scientifiques (Evidence: id, claim, source, evidenceLevel, reviewStatus, topics).

---

## Critères d'Acceptation
1. **Stockage des preuves scientifiques** :
   - [ ] Une preuve scientifique contient les champs suivants :
     - `id`: Identifiant unique.
     - `claim`: Affirmation ou conclusion (ex: "La qualité de l'air a un impact sur la santé").
     - `source`: Source de la preuve (titre, éditeur, URL).
     - `evidenceLevel`: Niveau de confiance (`HIGH`, `MEDIUM`, `LOW`).
     - `reviewStatus`: Statut de révision (`PEER_REVIEWED`, `PREPRINT`, `UNREVIEWED`).
     - `topics`: Liste de sujets associés (ex: ["air_quality", "health"]).

2. **Classement par niveau de confiance** :
   - [ ] Les preuves sont **classées par niveau de confiance** (`HIGH` > `MEDIUM` > `LOW`).
   - [ ] Les preuves `HIGH` sont **prioritaires** pour l'association aux recommandations.

3. **Récupération des preuves** :
   - [ ] Les preuves peuvent être **recherchées par** :
     - `id`.
     - `topics` (ex: toutes les preuves sur "air_quality").
     - `evidenceLevel` (ex: toutes les preuves `HIGH`).
     - `reviewStatus` (ex: toutes les preuves `PEER_REVIEWED`).

4. **Validation des preuves** :
   - [ ] Les preuves sont **validées** avant d'être stockées (ex: URL valide, champs obligatoires présents).
   - [ ] Une **erreur** est générée si une preuve invalide est ajoutée.

5. **Intégration avec le Knowledge Engine** :
   - [ ] Le `KnowledgeEngine` permet de **stocker, récupérer et mettre à jour** les preuves scientifiques.
   - [ ] Les preuves sont **accessibles** pour l'association aux recommandations (voir **Story 3.2**).

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/knowledge_engine/models.py` | Modèle `Evidence` | À créer |
| `backend/engines/knowledge_engine/storage.py` | Module `EvidenceStorage` | À créer |
| `backend/engines/knowledge_engine/__init__.py` | Export du `KnowledgeEngine` | À créer |
| `tests/unit/test_knowledge_engine.py` | Tests unitaires | À créer |

### Modèle `Evidence` (`models.py`)
```python
from typing import List, Optional
from enum import Enum
from pydantic import BaseModel, Field, HttpUrl
from datetime import datetime


class EvidenceLevel(Enum):
    """Niveau de confiance de la preuve scientifique."""
    HIGH = "high"       # Preuves solides (ex: méta-analyses, études randomisées)
    MEDIUM = "medium"   # Preuves modérées (ex: études observationnelles)
    LOW = "low"         # Preuves limitées (ex: études préliminaires)


class ReviewStatus(Enum):
    """Statut de révision de la preuve scientifique."""
    PEER_REVIEWED = "peer_reviewed"  # Publié dans une revue à comité de lecture
    PREPRINT = "preprint"            # Prépublication (non révisée)
    UNREVIEWED = "unreviewed"      # Non révisée


class Source(BaseModel):
    """Source de la preuve scientifique."""
    title: str = Field(..., description="Titre de la source (ex: 'Rapport de l'OMS sur la qualité de l'air')")
    publisher: str = Field(..., description="Éditeur ou organisation (ex: 'OMS')")
    url: HttpUrl = Field(..., description="URL de la source")
    publication_date: Optional[datetime] = Field(None, description="Date de publication")


class Evidence(BaseModel):
    """
    Preuve scientifique pour expliquer les recommandations.
    
    Exemple:
    ```python
    evidence = Evidence(
        id="evidence_001",
        claim="La qualité de l'air a un impact direct sur la santé respiratoire.",
        source=Source(
            title="Rapport mondial sur la qualité de l'air",
            publisher="OMS",
            url="https://www.who.int/fr/news-room/fact-sheets/detail/ambient-(outdoor)-air-quality-and-health",
            publication_date=datetime(2023, 1, 1)
        ),
        evidenceLevel=EvidenceLevel.HIGH,
        reviewStatus=ReviewStatus.PEER_REVIEWED,
        topics=["air_quality", "health", "respiratory"]
    )
    ```
    """
    id: str = Field(..., description="ID unique de la preuve")
    claim: str = Field(..., description="Affirmation ou conclusion de la preuve")
    source: Source = Field(..., description="Source de la preuve")
    evidenceLevel: EvidenceLevel = Field(..., description="Niveau de confiance de la preuve")
    reviewStatus: ReviewStatus = Field(..., description="Statut de révision de la preuve")
    topics: List[str] = Field(default_factory=list, description="Liste de sujets associés")
    created_at: datetime = Field(default_factory=datetime.now, description="Date de création")
    updated_at: Optional[datetime] = Field(None, description="Date de dernière mise à jour")
```

### Module `EvidenceStorage` (`storage.py`)
```python
from typing import List, Optional, Dict
from .models import Evidence, EvidenceLevel, ReviewStatus, Source
from datetime import datetime
import uuid


class EvidenceStorage:
    """
    Module de stockage pour les preuves scientifiques.
    - Stocke les preuves en mémoire (pour la phase PoC).
    - Permet de récupérer les preuves par ID, topics, evidenceLevel ou reviewStatus.
    - Valide les preuves avant stockage.
    """

    def __init__(self):
        """Initialise le stockage des preuves."""
        self._evidences: Dict[str, Evidence] = {}
        self._topics_index: Dict[str, List[str]] = {}  # topic -> list[evidence_id]
        self._level_index: Dict[EvidenceLevel, List[str]] = {}  # evidenceLevel -> list[evidence_id]
        self._review_index: Dict[ReviewStatus, List[str]] = {}  # reviewStatus -> list[evidence_id]

    def add_evidence(self, evidence: Evidence) -> Evidence:
        """
        Ajoute une preuve scientifique au stockage.

        Args:
            evidence: La preuve à ajouter.

        Returns:
            Evidence: La preuve ajoutée (avec ID généré si nécessaire).

        Raises:
            ValueError: Si la preuve est invalide.
        """
        self._validate_evidence(evidence)

        # Générer un ID si non fourni
        if not evidence.id:
            evidence.id = f"evidence_{uuid.uuid4().hex[:8]}"

        # Ajouter au stockage principal
        self._evidences[evidence.id] = evidence

        # Mettre à jour les index
        for topic in evidence.topics:
            if topic not in self._topics_index:
                self._topics_index[topic] = []
            self._topics_index[topic].append(evidence.id)

        if evidence.evidenceLevel not in self._level_index:
            self._level_index[evidence.evidenceLevel] = []
        self._level_index[evidence.evidenceLevel].append(evidence.id)

        if evidence.reviewStatus not in self._review_index:
            self._review_index[evidence.reviewStatus] = []
        self._review_index[evidence.reviewStatus].append(evidence.id)

        return evidence

    def get_evidence(self, evidence_id: str) -> Optional[Evidence]:
        """
        Récupère une preuve par son ID.

        Args:
            evidence_id: L'ID de la preuve.

        Returns:
            Optional[Evidence]: La preuve si trouvée, sinon None.
        """
        return self._evidences.get(evidence_id)

    def get_evidences_by_topic(self, topic: str) -> List[Evidence]:
        """
        Récupère toutes les preuves associées à un sujet.

        Args:
            topic: Le sujet à rechercher.

        Returns:
            List[Evidence]: Liste des preuves associées au sujet.
        """
        evidence_ids = self._topics_index.get(topic, [])
        return [self._evidences[evidence_id] for evidence_id in evidence_ids]

    def get_evidences_by_level(self, level: EvidenceLevel) -> List[Evidence]:
        """
        Récupère toutes les preuves d'un niveau de confiance donné.

        Args:
            level: Le niveau de confiance.

        Returns:
            List[Evidence]: Liste des preuves du niveau spécifié.
        """
        evidence_ids = self._level_index.get(level, [])
        return [self._evidences[evidence_id] for evidence_id in evidence_ids]

    def get_evidences_by_review_status(self, status: ReviewStatus) -> List[Evidence]:
        """
        Récupère toutes les preuves d'un statut de révision donné.

        Args:
            status: Le statut de révision.

        Returns:
            List[Evidence]: Liste des preuves du statut spécifié.
        """
        evidence_ids = self._review_index.get(status, [])
        return [self._evidences[evidence_id] for evidence_id in evidence_ids]

    def get_all_evidences(self) -> List[Evidence]:
        """
        Récupère toutes les preuves stockées.

        Returns:
            List[Evidence]: Liste de toutes les preuves.
        """
        return list(self._evidences.values())

    def update_evidence(self, evidence_id: str, **kwargs) -> Optional[Evidence]:
        """
        Met à jour une preuve existante.

        Args:
            evidence_id: L'ID de la preuve à mettre à jour.
            **kwargs: Champs à mettre à jour (ex: claim="Nouvelle affirmation").

        Returns:
            Optional[Evidence]: La preuve mise à jour, ou None si non trouvée.
        """
        if evidence_id not in self._evidences:
            return None

        evidence = self._evidences[evidence_id]
        for key, value in kwargs.items():
            if hasattr(evidence, key):
                setattr(evidence, key, value)

        # Mettre à jour l'index des topics si nécessaire
        if "topics" in kwargs:
            # Supprimer l'ancien index
            for topic in getattr(evidence, "topics", []):
                if topic in self._topics_index:
                    self._topics_index[topic].remove(evidence_id)
            # Ajouter le nouvel index
            for topic in kwargs["topics"]:
                if topic not in self._topics_index:
                    self._topics_index[topic] = []
                self._topics_index[topic].append(evidence_id)

        evidence.updated_at = datetime.now()
        return evidence

    def delete_evidence(self, evidence_id: str) -> bool:
        """
        Supprime une preuve du stockage.

        Args:
            evidence_id: L'ID de la preuve à supprimer.

        Returns:
            bool: True si la preuve a été supprimée, False sinon.
        """
        if evidence_id not in self._evidences:
            return False

        evidence = self._evidences[evidence_id]

        # Supprimer des index
        for topic in evidence.topics:
            if topic in self._topics_index:
                self._topics_index[topic].remove(evidence_id)

        if evidence.evidenceLevel in self._level_index:
            self._level_index[evidence.evidenceLevel].remove(evidence_id)

        if evidence.reviewStatus in self._review_index:
            self._review_index[evidence.reviewStatus].remove(evidence_id)

        del self._evidences[evidence_id]
        return True

    def _validate_evidence(self, evidence: Evidence) -> None:
        """
        Valide une preuve avant stockage.

        Args:
            evidence: La preuve à valider.

        Raises:
            ValueError: Si la preuve est invalide.
        """
        if not evidence.claim:
            raise ValueError("Le champ 'claim' est obligatoire")

        if not evidence.source:
            raise ValueError("Le champ 'source' est obligatoire")

        if not evidence.source.title:
            raise ValueError("Le champ 'source.title' est obligatoire")

        if not evidence.source.publisher:
            raise ValueError("Le champ 'source.publisher' est obligatoire")

        if not evidence.source.url:
            raise ValueError("Le champ 'source.url' est obligatoire")

        if not evidence.evidenceLevel:
            raise ValueError("Le champ 'evidenceLevel' est obligatoire")

        if not evidence.reviewStatus:
            raise ValueError("Le champ 'reviewStatus' est obligatoire")

    def clear(self) -> None:
        """Supprime toutes les preuves du stockage."""
        self._evidences.clear()
        self._topics_index.clear()
        self._level_index.clear()
        self._review_index.clear()


### Module `KnowledgeEngine` (`__init__.py`)
```python
from .models import Evidence, EvidenceLevel, ReviewStatus, Source
from .storage import EvidenceStorage


class KnowledgeEngine:
    """
    Moteur de gestion des preuves scientifiques.
    - Stocke et récupère les preuves scientifiques.
    - Permet de rechercher des preuves par sujet, niveau de confiance ou statut de révision.
    """

    def __init__(self):
        """Initialise le KnowledgeEngine avec un stockage en mémoire."""
        self.storage = EvidenceStorage()

    def add_evidence(self, evidence: Evidence) -> Evidence:
        """
        Ajoute une preuve scientifique.

        Args:
            evidence: La preuve à ajouter.

        Returns:
            Evidence: La preuve ajoutée.
        """
        return self.storage.add_evidence(evidence)

    def get_evidence(self, evidence_id: str) -> Optional[Evidence]:
        """
        Récupère une preuve par son ID.

        Args:
            evidence_id: L'ID de la preuve.

        Returns:
            Optional[Evidence]: La preuve si trouvée, sinon None.
        """
        return self.storage.get_evidence(evidence_id)

    def get_evidences_by_topic(self, topic: str) -> List[Evidence]:
        """
        Récupère toutes les preuves associées à un sujet.

        Args:
            topic: Le sujet à rechercher.

        Returns:
            List[Evidence]: Liste des preuves associées au sujet.
        """
        return self.storage.get_evidences_by_topic(topic)

    def get_evidences_by_level(self, level: EvidenceLevel) -> List[Evidence]:
        """
        Récupère toutes les preuves d'un niveau de confiance donné.

        Args:
            level: Le niveau de confiance.

        Returns:
            List[Evidence]: Liste des preuves du niveau spécifié.
        """
        return self.storage.get_evidences_by_level(level)

    def get_evidences_by_review_status(self, status: ReviewStatus) -> List[Evidence]:
        """
        Récupère toutes les preuves d'un statut de révision donné.

        Args:
            status: Le statut de révision.

        Returns:
            List[Evidence]: Liste des preuves du statut spécifié.
        """
        return self.storage.get_evidences_by_review_status(status)

    def get_all_evidences(self) -> List[Evidence]:
        """
        Récupère toutes les preuves stockées.

        Returns:
            List[Evidence]: Liste de toutes les preuves.
        """
        return self.storage.get_all_evidences()

    def update_evidence(self, evidence_id: str, **kwargs) -> Optional[Evidence]:
        """
        Met à jour une preuve existante.

        Args:
            evidence_id: L'ID de la preuve à mettre à jour.
            **kwargs: Champs à mettre à jour.

        Returns:
            Optional[Evidence]: La preuve mise à jour, ou None si non trouvée.
        """
        return self.storage.update_evidence(evidence_id, **kwargs)

    def delete_evidence(self, evidence_id: str) -> bool:
        """
        Supprime une preuve.

        Args:
            evidence_id: L'ID de la preuve à supprimer.

        Returns:
            bool: True si la preuve a été supprimée, False sinon.
        """
        return self.storage.delete_evidence(evidence_id)

    def clear(self) -> None:
        """Supprime toutes les preuves."""
        self.storage.clear()


# Instance globale
knowledge_engine = KnowledgeEngine()
```

---

## Tests Unitaires
Créer un fichier `tests/unit/test_knowledge_engine.py` avec les tests suivants :

```python
import pytest
from datetime import datetime
from unittest.mock import AsyncMock, patch, MagicMock
import sys
import os

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '../../backend'))

from backend.engines.knowledge_engine.models import Evidence, EvidenceLevel, ReviewStatus, Source
from backend.engines.knowledge_engine import knowledge_engine


@pytest.fixture
def mock_evidence():
    """Crée une preuve scientifique valide pour les tests."""
    return Evidence(
        id="evidence_001",
        claim="La qualité de l'air a un impact direct sur la santé respiratoire.",
        source=Source(
            title="Rapport mondial sur la qualité de l'air",
            publisher="OMS",
            url="https://www.who.int/fr/news-room/fact-sheets/detail/ambient-(outdoor)-air-quality-and-health",
            publication_date=datetime(2023, 1, 1)
        ),
        evidenceLevel=EvidenceLevel.HIGH,
        reviewStatus=ReviewStatus.PEER_REVIEWED,
        topics=["air_quality", "health", "respiratory"]
    )


@pytest.fixture
def mock_evidence_medium():
    """Crée une preuve scientifique de niveau MEDIUM."""
    return Evidence(
        id="evidence_002",
        claim="Les activités en plein air améliorent le bien-être mental.",
        source=Source(
            title="Étude sur le bien-être et la nature",
            publisher="Université de Stanford",
            url="https://news.stanford.edu/stories/2015/06/30/nature-experience-mental-health-063015",
            publication_date=datetime(2025, 6, 30)
        ),
        evidenceLevel=EvidenceLevel.MEDIUM,
        reviewStatus=ReviewStatus.PREPRINT,
        topics=["wellbeing", "nature", "mental_health"]
    )


@pytest.fixture
def mock_evidence_low():
    """Crée une preuve scientifique de niveau LOW."""
    return Evidence(
        id="evidence_003",
        claim="Le vélo est une activité écologique.",
        source=Source(
            title="Blog sur l'écologie",
            publisher="ÉcoBlog",
            url="https://ecoblog.fr/velo-ecologique",
            publication_date=datetime(2024, 3, 15)
        ),
        evidenceLevel=EvidenceLevel.LOW,
        reviewStatus=ReviewStatus.UNREVIEWED,
        topics=["mobility", "ecology"]
    )


# Test d'ajout de preuve
def test_add_evidence(knowledge_engine, mock_evidence):
    """Test l'ajout d'une preuve scientifique."""
    knowledge_engine.clear()  # Nettoyer avant le test
    added_evidence = knowledge_engine.add_evidence(mock_evidence)
    assert added_evidence.id == "evidence_001"
    assert len(knowledge_engine.get_all_evidences()) == 1


# Test de récupération de preuve par ID
def test_get_evidence(knowledge_engine, mock_evidence):
    """Test la récupération d'une preuve par son ID."""
    knowledge_engine.clear()
    knowledge_engine.add_evidence(mock_evidence)
    retrieved_evidence = knowledge_engine.get_evidence("evidence_001")
    assert retrieved_evidence is not None
    assert retrieved_evidence.claim == mock_evidence.claim


# Test de récupération de preuve inexistante
def test_get_nonexistent_evidence(knowledge_engine):
    """Test la récupération d'une preuve inexistante."""
    knowledge_engine.clear()
    retrieved_evidence = knowledge_engine.get_evidence("nonexistent")
    assert retrieved_evidence is None


# Test de récupération par sujet
def test_get_evidences_by_topic(knowledge_engine, mock_evidence, mock_evidence_medium):
    """Test la récupération des preuves par sujet."""
    knowledge_engine.clear()
    knowledge_engine.add_evidence(mock_evidence)
    knowledge_engine.add_evidence(mock_evidence_medium)

    air_quality_evidences = knowledge_engine.get_evidences_by_topic("air_quality")
    assert len(air_quality_evidences) == 1
    assert air_quality_evidences[0].id == "evidence_001"

    wellbeing_evidences = knowledge_engine.get_evidences_by_topic("wellbeing")
    assert len(wellbeing_evidences) == 1
    assert wellbeing_evidences[0].id == "evidence_002"


# Test de récupération par niveau de confiance
def test_get_evidences_by_level(knowledge_engine, mock_evidence, mock_evidence_medium, mock_evidence_low):
    """Test la récupération des preuves par niveau de confiance."""
    knowledge_engine.clear()
    knowledge_engine.add_evidence(mock_evidence)
    knowledge_engine.add_evidence(mock_evidence_medium)
    knowledge_engine.add_evidence(mock_evidence_low)

    high_evidences = knowledge_engine.get_evidences_by_level(EvidenceLevel.HIGH)
    assert len(high_evidences) == 1
    assert high_evidences[0].id == "evidence_001"

    medium_evidences = knowledge_engine.get_evidences_by_level(EvidenceLevel.MEDIUM)
    assert len(medium_evidences) == 1
    assert medium_evidences[0].id == "evidence_002"


# Test de récupération par statut de révision
def test_get_evidences_by_review_status(knowledge_engine, mock_evidence, mock_evidence_medium):
    """Test la récupération des preuves par statut de révision."""
    knowledge_engine.clear()
    knowledge_engine.add_evidence(mock_evidence)
    knowledge_engine.add_evidence(mock_evidence_medium)

    peer_reviewed_evidences = knowledge_engine.get_evidences_by_review_status(ReviewStatus.PEER_REVIEWED)
    assert len(peer_reviewed_evidences) == 1
    assert peer_reviewed_evidences[0].id == "evidence_001"

    preprint_evidences = knowledge_engine.get_evidences_by_review_status(ReviewStatus.PREPRINT)
    assert len(preprint_evidences) == 1
    assert preprint_evidences[0].id == "evidence_002"


# Test de mise à jour de preuve
def test_update_evidence(knowledge_engine, mock_evidence):
    """Test la mise à jour d'une preuve."""
    knowledge_engine.clear()
    knowledge_engine.add_evidence(mock_evidence)

    updated_evidence = knowledge_engine.update_evidence("evidence_001", claim="Nouvelle affirmation mise à jour")
    assert updated_evidence is not None
    assert updated_evidence.claim == "Nouvelle affirmation mise à jour"


# Test de mise à jour de preuve inexistante
def test_update_nonexistent_evidence(knowledge_engine):
    """Test la mise à jour d'une preuve inexistante."""
    knowledge_engine.clear()
    updated_evidence = knowledge_engine.update_evidence("nonexistent", claim="Nouvelle affirmation")
    assert updated_evidence is None


# Test de suppression de preuve
def test_delete_evidence(knowledge_engine, mock_evidence):
    """Test la suppression d'une preuve."""
    knowledge_engine.clear()
    knowledge_engine.add_evidence(mock_evidence)

    assert knowledge_engine.delete_evidence("evidence_001") is True
    assert knowledge_engine.get_evidence("evidence_001") is None


# Test de suppression de preuve inexistante
def test_delete_nonexistent_evidence(knowledge_engine):
    """Test la suppression d'une preuve inexistante."""
    knowledge_engine.clear()
    assert knowledge_engine.delete_evidence("nonexistent") is False


# Test de validation des preuves
def test_validate_evidence(knowledge_engine):
    """Test la validation des preuves avant stockage."""
    knowledge_engine.clear()

    # Preuve invalide (champ 'claim' manquant)
    with pytest.raises(ValueError, match="Le champ 'claim' est obligatoire"):
        knowledge_engine.add_evidence(Evidence(
            id="invalid_evidence",
            claim="",  # Vide
            source=Source(
                title="Titre",
                publisher="Éditeur",
                url="https://example.com"
            ),
            evidenceLevel=EvidenceLevel.HIGH,
            reviewStatus=ReviewStatus.PEER_REVIEWED
        ))

    # Preuve invalide (source manquante)
    with pytest.raises(ValueError, match="Le champ 'source' est obligatoire"):
        knowledge_engine.add_evidence(Evidence(
            id="invalid_evidence",
            claim="Affirmation valide",
            source=None,  # Manquant
            evidenceLevel=EvidenceLevel.HIGH,
            reviewStatus=ReviewStatus.PEER_REVIEWED
        ))


# Test de récupération de toutes les preuves
def test_get_all_evidences(knowledge_engine, mock_evidence, mock_evidence_medium, mock_evidence_low):
    """Test la récupération de toutes les preuves."""
    knowledge_engine.clear()
    knowledge_engine.add_evidence(mock_evidence)
    knowledge_engine.add_evidence(mock_evidence_medium)
    knowledge_engine.add_evidence(mock_evidence_low)

    all_evidences = knowledge_engine.get_all_evidences()
    assert len(all_evidences) == 3


# Test de nettoyage du stockage
def test_clear_evidences(knowledge_engine, mock_evidence):
    """Test le nettoyage de toutes les preuves."""
    knowledge_engine.clear()
    knowledge_engine.add_evidence(mock_evidence)
    assert len(knowledge_engine.get_all_evidences()) == 1

    knowledge_engine.clear()
    assert len(knowledge_engine.get_all_evidences()) == 0


# Test d'ajout de preuve sans ID
def test_add_evidence_without_id(knowledge_engine):
    """Test l'ajout d'une preuve sans ID (généré automatiquement)."""
    knowledge_engine.clear()
    evidence = Evidence(
        id="",  # Vide
        claim="Affirmation sans ID",
        source=Source(
            title="Titre",
            publisher="Éditeur",
            url="https://example.com"
        ),
        evidenceLevel=EvidenceLevel.HIGH,
        reviewStatus=ReviewStatus.PEER_REVIEWED
    )

    added_evidence = knowledge_engine.add_evidence(evidence)
    assert added_evidence.id.startswith("evidence_")
    assert len(added_evidence.id) > 8  # UUID généré


# Test de déterminisme du KnowledgeEngine
def test_knowledge_engine_determinism(knowledge_engine, mock_evidence):
    """Test que le KnowledgeEngine est déterministe."""
    knowledge_engine.clear()

    # Ajouter la même preuve deux fois (avec des IDs différents)
    evidence1 = knowledge_engine.add_evidence(mock_evidence)
    knowledge_engine.clear()
    evidence2 = knowledge_engine.add_evidence(mock_evidence)

    # Les IDs peuvent être différents, mais les autres champs doivent être identiques
    assert evidence1.claim == evidence2.claim
    assert evidence1.source.title == evidence2.source.title
    assert evidence1.evidenceLevel == evidence2.evidenceLevel
```

---

## Configuration Requise
Aucune configuration supplémentaire nécessaire. Utilise `pydantic` pour la validation des données.

---

## Dépendances
- **Python 3.11+**
- **Librairies** :
  - `pydantic` (pour la validation des données).

---

## Notes
- **Preuves scientifiques** : Les preuves sont **stockées en mémoire** pour la phase PoC. Une intégration avec une base de données (ex: PostgreSQL) sera nécessaire pour la phase GA.
- **Niveaux de confiance** : Les preuves sont classées par niveau de confiance (`HIGH` > `MEDIUM` > `LOW`) pour prioriser les sources les plus fiables.
- **Statuts de révision** : Les preuves peuvent être `PEER_REVIEWED`, `PREPRINT` ou `UNREVIEWED` pour indiquer leur niveau de validation.
- **Indexation** : Les preuves sont **indexées par sujet, niveau de confiance et statut de révision** pour une recherche efficace.
- **Validation** : Les preuves sont **validées** avant d'être stockées pour garantir leur intégrité.

---

## Exemple d'Utilisation
```python
from backend.engines.knowledge_engine import knowledge_engine
from backend.engines.knowledge_engine.models import Evidence, EvidenceLevel, ReviewStatus, Source
from datetime import datetime

# Ajouter une preuve scientifique
knowledge_engine.add_evidence(Evidence(
    id="evidence_001",
    claim="La qualité de l'air a un impact direct sur la santé respiratoire.",
    source=Source(
        title="Rapport mondial sur la qualité de l'air",
        publisher="OMS",
        url="https://www.who.int/fr/news-room/fact-sheets/detail/ambient-(outdoor)-air-quality-and-health",
        publication_date=datetime(2023, 1, 1)
    ),
    evidenceLevel=EvidenceLevel.HIGH,
    reviewStatus=ReviewStatus.PEER_REVIEWED,
    topics=["air_quality", "health", "respiratory"]
))

# Récupérer une preuve par son ID
evidence = knowledge_engine.get_evidence("evidence_001")
print(f"Preuve: {evidence.claim} (Source: {evidence.source.publisher})")

# Récupérer toutes les preuves sur la qualité de l'air
air_quality_evidences = knowledge_engine.get_evidences_by_topic("air_quality")
for evidence in air_quality_evidences:
    print(f"- {evidence.claim} (Niveau: {evidence.evidenceLevel.value})")

# Récupérer toutes les preuves de niveau HIGH
high_evidences = knowledge_engine.get_evidences_by_level(EvidenceLevel.HIGH)
print(f"Nombre de preuves HIGH: {len(high_evidences)}")
```

---

## Ressources
- [PRD Almanéa - FR-11](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-5](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
