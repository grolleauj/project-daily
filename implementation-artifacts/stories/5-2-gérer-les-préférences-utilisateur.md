---
name: Gérer les préférences utilisateur
id: 5-2-gérer-les-préférences-utilisateur
epic: epic-5
story_type: backend
priority: high
estimation: S
dependencies: [5-1-créer-et-mettre-à-jour-le-profil-utilisateur]
status: done
created: 2026-08-17
updated: 2026-08-17
baseline_commit: NO_VCS
context:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/epics/epics.md
---

# Story 5.2: Gérer les préférences utilisateur

## Contexte
Cette story fait partie de **l'Épic 5 (Gestion du profil utilisateur)**. Son objectif est de **permettre aux utilisateurs de définir, mettre à jour et consulter leurs préférences** (ex: activités préférées, contraintes de temps, localisations à éviter).

Le `UserPreferencesManager` étend le `UserProfileManager` pour :
- **Gérer les préférences** de manière granulaire (ex: `likes_cycling`, `max_duration_minutes`).
- **Valider les préférences** avant de les stocker.
- **Rechercher des utilisateurs par préférence** (ex: tous les utilisateurs qui aiment le vélo).

---

## Exigences Fonctionnelles
- **FR-33**: Définir et mettre à jour des préférences utilisateur.
- **FR-34**: Valider les préférences avant de les stocker.
- **FR-35**: Récupérer les préférences d'un utilisateur.
- **FR-36**: Rechercher des utilisateurs par préférence.

---

## Critères d'Acceptation
1. **Définition de préférences** :
   - [x] Définir une préférence pour un utilisateur (ex: `likes_cycling=True`).
   - [x] Mettre à jour plusieurs préférences en une seule opération.

2. **Récupération de préférences** :
   - [x] Récupérer une préférence spécifique pour un utilisateur.
   - [x] Récupérer toutes les préférences d'un utilisateur.

3. **Validation des préférences** :
   - [x] Valider qu'une préférence est valide avant de la stocker.
   - [x] Utiliser le modèle `UserPreferences` pour la validation.

4. **Recherche par préférence** :
   - [x] Récupérer tous les utilisateurs ayant une préférence spécifique (ex: `likes_cycling=True`).

5. **Suppression de préférences** :
   - [x] Supprimer une préférence spécifique pour un utilisateur.
   - [x] Supprimer toutes les préférences d'un utilisateur.

6. **Intégration avec le UserProfileManager** :
   - [x] Le `UserPreferencesManager` utilise le `UserProfileManager` pour stocker les préférences.

---

## Implémentation Technique

### Fichiers créés/modifiés
| Fichier | Rôle | Statut |
|---------|------|--------|
| `backend/engines/recommendation_engine/user_preferences_manager.py` | Module `UserPreferencesManager` | ✅ Créé |
| `tests/unit/test_user_preferences_manager.py` | Tests unitaires | ✅ Créé |

### Modèle `UserPreferences`
Un modèle Pydantic pour valider les préférences utilisateur :

```python
from typing import Dict, Any, Optional, List
from enum import Enum
from pydantic import BaseModel, Field

class PreferenceCategory(Enum):
    MOBILITY = "mobility"
    NATURE = "nature"
    ENERGY = "energy"
    WATER = "water"
    REPAIR = "repair"
    WELLBEING = "wellbeing"
    HEALTH = "health"
    TIME = "time"
    LOCATION = "location"

class UserPreferences(BaseModel):
    likes_cycling: Optional[bool] = Field(None, description="Aime faire du vélo")
    likes_walking: Optional[bool] = Field(None, description="Aime marcher")
    likes_public_transport: Optional[bool] = Field(None, description="Aime les transports en commun")
    likes_carpooling: Optional[bool] = Field(None, description="Aime le covoiturage")
    likes_nature_activities: Optional[bool] = Field(None, description="Aime les activités en nature")
    likes_diy: Optional[bool] = Field(None, description="Aime le bricolage")
    likes_gardening: Optional[bool] = Field(None, description="Aime le jardinage")
    avoid_pollution: Optional[bool] = Field(None, description="Éviter les zones polluées")
    avoid_rain: Optional[bool] = Field(None, description="Éviter la pluie")
    avoid_high_temperature: Optional[bool] = Field(None, description="Éviter les températures élevées")
    avoid_low_temperature: Optional[bool] = Field(None, description="Éviter les températures basses")
    max_duration_minutes: Optional[int] = Field(None, description="Durée maximale pour une activité (en minutes)")
    min_duration_minutes: Optional[int] = Field(None, description="Durée minimale pour une activité (en minutes)")
    preferred_locations: Optional[List[str]] = Field(None, description="Localisations préférées")
    avoided_locations: Optional[List[str]] = Field(None, description="Localisations à éviter")
    preferred_times: Optional[List[str]] = Field(None, description="Créneaux horaires préférés")
    avoided_times: Optional[List[str]] = Field(None, description="Créneaux horaires à éviter")
```

### Module `UserPreferencesManager`
Ce module gère les préférences utilisateur :

```python
from typing import Dict, Any, Optional, List
from .user_profile_manager import user_profile_manager
from .models import UserPreferences

class UserPreferencesManager:
    def __init__(self):
        self.user_profile_manager = user_profile_manager

    def set_preference(
        self,
        user_id: str,
        preference_key: str,
        preference_value: Any
    ) -> bool:
        """Définit une préférence pour un utilisateur."""
        profile = self.user_profile_manager.get_profile(user_id)
        if not profile:
            return False
        profile.preferences[preference_key] = preference_value
        return True

    def get_preference(
        self,
        user_id: str,
        preference_key: str,
        default: Any = None
    ) -> Any:
        """Récupère une préférence pour un utilisateur."""
        profile = self.user_profile_manager.get_profile(user_id)
        if not profile:
            return default
        return profile.preferences.get(preference_key, default)

    def get_all_preferences(self, user_id: str) -> Optional[Dict[str, Any]]:
        """Récupère toutes les préférences d'un utilisateur."""
        profile = self.user_profile_manager.get_profile(user_id)
        if not profile:
            return None
        return profile.preferences

    def update_preferences(self, user_id: str, new_preferences: Dict[str, Any]) -> bool:
        """Met à jour plusieurs préférences pour un utilisateur."""
        profile = self.user_profile_manager.get_profile(user_id)
        if not profile:
            return False
        profile.preferences.update(new_preferences)
        return True

    def clear_preferences(self, user_id: str) -> bool:
        """Supprime toutes les préférences d'un utilisateur."""
        profile = self.user_profile_manager.get_profile(user_id)
        if not profile:
            return False
        profile.preferences.clear()
        return True

    def clear_preference(self, user_id: str, preference_key: str) -> bool:
        """Supprime une préférence spécifique pour un utilisateur."""
        profile = self.user_profile_manager.get_profile(user_id)
        if not profile or preference_key not in profile.preferences:
            return False
        del profile.preferences[preference_key]
        return True

    def validate_preferences(self, preferences: Dict[str, Any]) -> bool:
        """Valide un dictionnaire de préférences."""
        try:
            UserPreferences(**preferences)
            return True
        except Exception:
            return False

    def get_users_by_preference(self, preference_key: str, preference_value: Any) -> List[str]:
        """Récupère la liste des utilisateurs ayant une préférence spécifique."""
        user_ids = []
        for profile in self.user_profile_manager.get_all_profiles():
            if profile.preferences.get(preference_key) == preference_value:
                user_ids.append(profile.user_id)
        return user_ids
```

---

## Tests Unitaires
17 tests ont été créés dans `tests/unit/test_user_preferences_manager.py` pour valider :
- La définition et la récupération de préférences.
- La mise à jour de plusieurs préférences.
- La suppression de préférences.
- La validation des préférences.
- La recherche d'utilisateurs par préférence.
- Le déterminisme du module.

---

## Configuration Requise
- **Dépendances** : `pydantic` pour la validation des préférences.
- **Intégration** : Utilise le `UserProfileManager` pour stocker les préférences.

---

## Dépendances
- **Story 5-1** : `créer-et-mettre-à-jour-le-profil-utilisateur` (doit être implémentée avant).

---

## Exemple d'Utilisation
```python
from backend.engines.recommendation_engine.user_preferences_manager import user_preferences_manager

# Définir une préférence
user_preferences_manager.set_preference(
    user_id="user_001",
    preference_key="likes_cycling",
    preference_value=True
)

# Récupérer une préférence
likes_cycling = user_preferences_manager.get_preference(
    user_id="user_001",
    preference_key="likes_cycling"
)
print(f"Aime le vélo : {likes_cycling}")

# Mettre à jour plusieurs préférences
user_preferences_manager.update_preferences(
    user_id="user_001",
    new_preferences={
        "max_duration_minutes": 60,
        "avoid_pollution": True
    }
)

# Récupérer toutes les préférences
preferences = user_preferences_manager.get_all_preferences("user_001")
print(f"Préférences : {preferences}")

# Récupérer les utilisateurs qui aiment le vélo
cycling_lovers = user_preferences_manager.get_users_by_preference(
    preference_key="likes_cycling",
    preference_value=True
)
print(f"Utilisateurs qui aiment le vélo : {cycling_lovers}")

# Valider des préférences
is_valid = user_preferences_manager.validate_preferences({
    "likes_cycling": True,
    "max_duration_minutes": 60
})
print(f"Préférences valides : {is_valid}")
```

---

## Notes
- **Flexibilité** : Les préférences sont stockées sous forme de dictionnaire pour permettre des extensions futures.
- **Validation** : Le modèle `UserPreferences` permet de valider les préférences avant de les stocker.
- **Recherche** : Permet de rechercher des utilisateurs par préférence pour des analyses ou des recommandations ciblées.
- **Déterminisme** : Le module est déterministe, ce qui facilite les tests.

---

## Ressources
- [PRD Almanéa - FR-33](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - AR-7](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
