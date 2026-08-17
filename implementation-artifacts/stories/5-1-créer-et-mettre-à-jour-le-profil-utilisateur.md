---
name: Créer et mettre à jour le profil utilisateur
id: 5-1-créer-et-mettre-à-jour-le-profil-utilisateur
epic: epic-5
story_type: backend
priority: high
estimation: M
dependances: []
status: done
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 5.1: Créer et mettre à jour le profil utilisateur

## Contexte
Cette story fait partie de **l'Épic 5 (Gestion du profil utilisateur)**. Son objectif est de **créer, mettre à jour et gérer les profils utilisateurs** pour personnaliser les recommandations.

Le `UserProfileManager` permet de :
- **Stocker les profils utilisateurs** (ID, email, fuseau horaire, localisations, préférences).
- **Effectuer des opérations CRUD** (Créer, Lire, Mettre à jour, Supprimer).
- **Rechercher des profils** par email ou localisation.

---

## Exigences Fonctionnelles
- **FR-30**: Créer et mettre à jour un profil utilisateur.
- **FR-31**: Stocker les préférences utilisateur (ex: activités préférées, contraintes).
- **FR-32**: Permettre la recherche de profils par email ou localisation.

---

## Critères d'Acceptation
1. **Création de profil** :
   - [x] Créer un profil avec les champs : `user_id`, `email`, `timezone`, `home_location`, `work_location`, `preferences`.
   - [x] Valider que le `user_id` est unique.

2. **Mise à jour de profil** :
   - [x] Mettre à jour un ou plusieurs champs d'un profil existant.
   - [x] Conserver les champs non modifiés.

3. **Récupération de profil** :
   - [x] Récupérer un profil par son `user_id`.
   - [x] Récupérer tous les profils.
   - [x] Récupérer un profil par son `email`.

4. **Recherche par localisation** :
   - [x] Récupérer tous les profils associés à une localisation (domicile ou travail).

5. **Suppression de profil** :
   - [x] Supprimer un profil par son `user_id`.
   - [x] Supprimer tous les profils.

6. **Intégration avec le RecommendationEngine** :
   - [x] Le `RecommendationEngine` utilise le `UserProfileManager` pour gérer les profils.

---

## Implémentation Technique

### Fichiers créés/modifiés
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/user_profile_manager.py` | Module `UserProfileManager` | ✅ Créé |
| `backend/engines/recommendation_engine/__init__.py` | Intégration du `UserProfileManager` | ✅ Modifié |
| `tests/unit/test_user_profile_manager.py` | Tests unitaires | ✅ Créé |

### Module `UserProfileManager`
Ce module gère les profils utilisateurs en mémoire (pour la phase PoC) :

```python
from typing import Dict, Optional, List
from .models import UserProfile
import uuid

class UserProfileManager:
    def __init__(self):
        self._profiles: Dict[str, UserProfile] = {}

    def create_profile(
        self,
        user_id: str,
        email: Optional[str] = None,
        timezone: Optional[str] = None,
        home_location: Optional[str] = None,
        work_location: Optional[str] = None,
        preferences: Optional[Dict] = None
    ) -> UserProfile:
        """Crée un nouveau profil utilisateur."""
        if user_id in self._profiles:
            raise ValueError(f"Un profil avec l'ID '{user_id}' existe déjà")
        profile = UserProfile(
            user_id=user_id,
            email=email,
            timezone=timezone,
            home_location=home_location,
            work_location=work_location,
            preferences=preferences or {}
        )
        self._profiles[user_id] = profile
        return profile

    def get_profile(self, user_id: str) -> Optional[UserProfile]:
        """Récupère un profil par son ID."""
        return self._profiles.get(user_id)

    def get_all_profiles(self) -> List[UserProfile]:
        """Récupère tous les profils."""
        return list(self._profiles.values())

    def update_profile(
        self,
        user_id: str,
        email: Optional[str] = None,
        timezone: Optional[str] = None,
        home_location: Optional[str] = None,
        work_location: Optional[str] = None,
        preferences: Optional[Dict] = None
    ) -> Optional[UserProfile]:
        """Met à jour un profil existant."""
        if user_id not in self._profiles:
            return None
        profile = self._profiles[user_id]
        if email is not None:
            profile.email = email
        if timezone is not None:
            profile.timezone = timezone
        if home_location is not None:
            profile.home_location = home_location
        if work_location is not None:
            profile.work_location = work_location
        if preferences is not None:
            profile.preferences.update(preferences)
        return profile

    def delete_profile(self, user_id: str) -> bool:
        """Supprime un profil."""
        if user_id not in self._profiles:
            return False
        del self._profiles[user_id]
        return True

    def clear(self) -> None:
        """Supprime tous les profils."""
        self._profiles.clear()

    def get_profile_by_email(self, email: str) -> Optional[UserProfile]:
        """Récupère un profil par son email."""
        for profile in self._profiles.values():
            if profile.email == email:
                return profile
        return None

    def get_profiles_by_location(self, location: str) -> List[UserProfile]:
        """Récupère tous les profils associés à une localisation."""
        return [
            profile for profile in self._profiles.values()
            if profile.home_location == location or profile.work_location == location
        ]
```

### Modèle `UserProfile`
Le modèle `UserProfile` est défini dans `models.py` :
```python
from datetime import datetime
from typing import Dict, Any, Optional
from pydantic import BaseModel, Field

class UserProfile(BaseModel):
    user_id: str = Field(..., description="ID unique de l'utilisateur")
    email: Optional[str] = Field(None, description="Email de l'utilisateur")
    timezone: Optional[str] = Field(None, description="Fuseau horaire de l'utilisateur")
    home_location: Optional[str] = Field(None, description="Localisation domicile (code postal)")
    work_location: Optional[str] = Field(None, description="Localisation travail (code postal)")
    preferences: Dict[str, Any] = Field(
        default_factory=dict,
        description="Préférences utilisateur (ex: {'likes_cycling': True, 'max_duration': 60})"
    )
```

---

## Tests Unitaires
16 tests ont été créés dans `tests/unit/test_user_profile_manager.py` pour valider :
- La création de profils.
- La récupération de profils par ID, email ou localisation.
- La mise à jour de profils.
- La suppression de profils.
- La gestion des préférences.
- Le déterminisme du module.

---

## Configuration Requise
- **Dépendances** : `pydantic` pour la validation des données.
- **Stockage** : En mémoire pour la phase PoC (à remplacer par une base de données pour la phase GA).

---

## Dépendances
Aucune dépendance spécifique. Utilise le modèle `UserProfile` existant.

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine.user_profile_manager import user_profile_manager

# Créer un profil
profile = user_profile_manager.create_profile(
    user_id="user_001",
    email="user@example.com",
    timezone="Europe/Paris",
    home_location="69003",
    work_location="69001",
    preferences={"likes_cycling": True, "max_duration": 60}
)

# Récupérer un profil
retrieved_profile = user_profile_manager.get_profile("user_001")
print(f"Profil : {retrieved_profile.email}, Localisation : {retrieved_profile.home_location}")

# Mettre à jour un profil
updated_profile = user_profile_manager.update_profile(
    user_id="user_001",
    timezone="Europe/London"
)

# Récupérer des profils par localisation
profiles_in_lyon = user_profile_manager.get_profiles_by_location("69003")

# Supprimer un profil
user_profile_manager.delete_profile("user_001")
```

---

## Notes
- **Stockage en mémoire** : Les profils sont stockés en mémoire pour la phase PoC. Une intégration avec une base de données (ex: PostgreSQL) sera nécessaire pour la phase GA.
- **Préférences utilisateur** : Les préférences sont stockées sous forme de dictionnaire pour une flexibilité maximale.
- **Recherche flexible** : Permet de rechercher des profils par email ou localisation.
- **Déterminisme** : Le module est déterministe, ce qui facilite les tests.

---

## Ressources
- [PRD Almanéa - FR-30](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-7](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
