---
name: Consulter l'historique des recommandations
id: 5-3-consulter-l-historique-des-recommandations
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

# Story 5.3: Consulter l'historique des recommandations

## Contexte
Cette story fait partie de **l'Épic 5 (Gestion du profil utilisateur)**. Son objectif est de **stocker et consulter l'historique des recommandations** générées pour chaque utilisateur, afin de :
- Permettre aux utilisateurs de **revoir leurs recommandations passées**.
- **Analyser les tendances** (ex: quelles recommandations sont les plus suivies ?).
- **Améliorer la personnalisation** en fonction de l'historique.

Le `RecommendationHistoryManager` permet de :
- **Stocker les recommandations** générées pour chaque utilisateur.
- **Récupérer l'historique** par utilisateur, date, catégorie, etc.
- **Générer des statistiques** (ex: nombre de recommandations par catégorie).

---

## Exigences Fonctionnelles
- **FR-37**: Stocker l'historique des recommandations générées pour chaque utilisateur.
- **FR-38**: Récupérer l'historique des recommandations par utilisateur, date ou catégorie.
- **FR-39**: Générer des statistiques à partir de l'historique.

---

## Critères d'Acceptation
1. **Stockage des recommandations** :
   - [x] Ajouter une recommandation à l'historique d'un utilisateur.
   - [x] Associer un timestamp à chaque recommandation stockée.

2. **Récupération de l'historique** :
   - [x] Récupérer l'historique complet d'un utilisateur.
   - [x] Récupérer une recommandation spécifique par son ID.
   - [x] Récupérer l'historique par catégorie (ex: toutes les recommandations de type "mobility").
   - [x] Récupérer l'historique par plage de dates.

3. **Statistiques** :
   - [x] Générer des statistiques par utilisateur (ex: nombre total de recommandations, répartition par catégorie).
   - [x] Récupérer les recommandations récentes.

4. **Gestion de l'historique** :
   - [x] Supprimer l'historique d'un utilisateur.
   - [x] Supprimer tout l'historique.

5. **Intégration avec le RecommendationEngine** :
   - [x] Le `RecommendationEngine` utilise le `RecommendationHistoryManager` pour sauvegarder les recommandations.

---

## Implémentation Technique

### Fichiers créés/modifiés
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/recommendation_history_manager.py` | Module `RecommendationHistoryManager` | ✅ Créé |
| `backend/engines/recommendation_engine/__init__.py` | Intégration du `RecommendationHistoryManager` | ✅ Modifié |
| `tests/unit/test_recommendation_history_manager.py` | Tests unitaires | ✅ Créé |

### Module `RecommendationHistoryManager`
Ce module gère l'historique des recommandations en mémoire :

```python
from typing import Dict, List, Optional
from datetime import datetime
from .models import Recommendation
import uuid

class RecommendationHistoryManager:
    def __init__(self):
        # Structure : {user_id: [{"id": str, "recommendation": Recommendation, "timestamp": datetime}]}
        self._history: Dict[str, List[Dict]] = {}

    def add_recommendation_to_history(self, user_id: str, recommendation: Recommendation) -> str:
        """Ajoute une recommandation à l'historique d'un utilisateur."""
        entry_id = f"history_{uuid.uuid4().hex[:8]}"
        if user_id not in self._history:
            self._history[user_id] = []
        self._history[user_id].append({
            "id": entry_id,
            "recommendation": recommendation,
            "timestamp": datetime.now()
        })
        return entry_id

    def get_user_history(self, user_id: str, limit: Optional[int] = None) -> List[Dict]:
        """Récupère l'historique des recommandations pour un utilisateur."""
        if user_id not in self._history:
            return []
        history = self._history[user_id]
        return history[-limit:] if limit else history

    def get_recommendation_by_id(self, user_id: str, entry_id: str) -> Optional[Dict]:
        """Récupère une recommandation spécifique de l'historique."""
        if user_id not in self._history:
            return None
        for entry in self._history[user_id]:
            if entry["id"] == entry_id:
                return entry
        return None

    def get_history_by_category(self, user_id: str, category: str, limit: Optional[int] = None) -> List[Dict]:
        """Récupère l'historique des recommandations pour une catégorie spécifique."""
        history = self.get_user_history(user_id)
        filtered_history = [
            entry for entry in history
            if entry["recommendation"].category.value == category
        ]
        return filtered_history[-limit:] if limit else filtered_history

    def get_history_by_date(self, user_id: str, start_date: Optional[datetime] = None, end_date: Optional[datetime] = None) -> List[Dict]:
        """Récupère l'historique des recommandations dans une plage de dates."""
        history = self.get_user_history(user_id)
        return [
            entry for entry in history
            if (start_date is None or entry["timestamp"] >= start_date) and
               (end_date is None or entry["timestamp"] <= end_date)
        ]

    def get_recent_recommendations(self, user_id: str, limit: int = 5) -> List[Dict]:
        """Récupère les recommandations récentes pour un utilisateur."""
        history = self.get_user_history(user_id, limit=limit)
        return history[::-1]  # Retourne dans l'ordre chronologique

    def get_recommendation_stats(self, user_id: str) -> Dict[str, int]:
        """Récupère les statistiques de l'historique des recommandations."""
        history = self.get_user_history(user_id)
        stats = {"total": len(history)}
        for entry in history:
            category = entry["recommendation"].category.value
            stats[category] = stats.get(category, 0) + 1
        return stats

    def clear_user_history(self, user_id: str) -> bool:
        """Supprime l'historique des recommandations pour un utilisateur."""
        if user_id not in self._history:
            return False
        del self._history[user_id]
        return True

    def clear_all_history(self) -> None:
        """Supprime tout l'historique des recommandations."""
        self._history.clear()
```

---

## Tests Unitaires
14 tests ont été créés dans `tests/unit/test_recommendation_history_manager.py` pour valider :
- L'ajout de recommandations à l'historique.
- La récupération de l'historique par utilisateur, ID, catégorie ou date.
- La génération de statistiques.
- La suppression de l'historique.
- Le déterminisme du module.

---

## Configuration Requise
- **Dépendances** : `uuid` pour générer des IDs uniques, `datetime` pour les timestamps.
- **Stockage** : En mémoire pour la phase PoC (à remplacer par une base de données pour la phase GA).

---

## Dépendances
- **Story 5-1** : `créer-et-mettre-à-jour-le-profil-utilisateur` (pour l'identification des utilisateurs).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine.recommendation_history_manager import recommendation_history_manager
from backend.engines.recommendation_engine.models import Recommendation, RecommendationCategory

# Ajouter une recommandation à l'historique
recommendation = Recommendation(
    id="rec_001",
    title="Éviter les sorties en vélo",
    description="La qualité de l'air est mauvaise aujourd'hui.",
    category=RecommendationCategory.MOBILITY,
    action="Éviter les sorties en vélo",
    conditions={"aqi": 80}
)

entry_id = recommendation_history_manager.add_recommendation_to_history(
    user_id="user_001",
    recommendation=recommendation
)

# Récupérer l'historique de l'utilisateur
history = recommendation_history_manager.get_user_history("user_001")
print(f"Historique : {len(history)} recommandations")

# Récupérer les statistiques
stats = recommendation_history_manager.get_recommendation_stats("user_001")
print(f"Statistiques : {stats}")

# Récupérer les recommandations récentes
recent = recommendation_history_manager.get_recent_recommendations("user_001", limit=3)
print(f"Recommandations récentes : {len(recent)}")

# Récupérer l'historique par catégorie
mobility_history = recommendation_history_manager.get_history_by_category(
    user_id="user_001",
    category="mobility"
)
print(f"Recommandations de mobilité : {len(mobility_history)}")
```

---

## Notes
- **Stockage en mémoire** : L'historique est stocké en mémoire pour la phase PoC.
- **Statistiques** : Permet de générer des rapports pour analyser les tendances d'utilisation.
- **Recommandations récentes** : Permet d'afficher les dernières recommandations à l'utilisateur.
- **Déterminisme** : Le module est déterministe, ce qui facilite les tests.

---

## Ressources
- [PRD Almanéa - FR-37](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-7](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
