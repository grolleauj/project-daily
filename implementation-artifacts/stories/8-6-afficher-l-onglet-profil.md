---
name: Afficher l'onglet Profil
id: 8-6-afficher-l-onglet-profil
epic: epic-8
story_type: frontend
priority: medium
estimation: S
dependencies: ["8-1-créer-le-layout-mobile-first", "epic-5"]
status: backlog
created: 2026-08-20
updated: 2026-08-20
---

# Story 8.6: Afficher l'onglet "Profil"

## Contexte
Cette story fait partie de **l'Épic 8 (Web UI)**. Son objectif est de créer l'**onglet "Profil"** qui permet aux utilisateurs de **gérer leur profil, leurs préférences et consulter leurs statistiques personnelles**. Cela répond à l'exigence **FR-36 (Gestion du profil, préférences et statistiques)**.

---

## Exigences Fonctionnelles
- **FR-36**: Gestion du profil, des préférences et des statistiques personnelles dans l'onglet "Profil".

---

## Critères d'Acceptation
1. **Gestion du profil** :
   - [ ] L'utilisateur peut **modifier son profil** (email, timezone, localisation, etc.).

2. **Gestion des préférences** :
   - [ ] L'utilisateur peut **configurer ses préférences** (activités, transport, durée max).

3. **Affichage des statistiques personnelles** :
   - [ ] L'utilisateur peut **voir ses statistiques personnelles** (ex: nombre de recommandations suivies, points gagnés).

4. **Consultation de l'historique** :
   - [ ] L'utilisateur peut **consulter son historique** (ex: liste des actions réalisées).

5. **Intégration des données** :
   - [ ] Les données proviennent du **User Management Engine** (Epic 5).

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `frontend/app/profil/page.tsx` | Page de l'onglet "Profil" | À créer |
| `frontend/components/profil/ProfileView.tsx` | Composant principal de l'onglet "Profil" | À créer |
| `frontend/components/profil/ProfileForm.tsx` | Formulaire de modification du profil | À créer |
| `frontend/components/profil/PreferencesForm.tsx` | Formulaire de configuration des préférences | À créer |
| `frontend/components/profil/StatsDisplay.tsx` | Affichage des statistiques personnelles | À créer |
| `frontend/components/profil/HistoryList.tsx` | Liste de l'historique des actions | À créer |

---

### Implémentation de la Page (`page.tsx`)
```tsx
import ProfileView from '@/components/profil/ProfileView';

export default function ProfilPage() {
  return (
    <div className="p-4">
      <ProfileView />
    </div>
  );
}
```

---

### Implémentation de la Vue Profil (`ProfileView.tsx`)
```tsx
'use client';

import { useEffect, useState } from 'react';
import ProfileForm from './ProfileForm';
import PreferencesForm from './PreferencesForm';
import StatsDisplay from './StatsDisplay';
import HistoryList from './HistoryList';

interface UserProfile {
  id: string;
  email: string;
  timezone: string;
  homeLocation: string;
  workLocation: string;
}

interface UserPreferences {
  activities: string[];
  transport: string[];
  maxDuration: number;
}

interface UserStats {
  totalRecommendationsFollowed: number;
  totalPoints: number;
  averageScore: number;
}

interface HistoryEntry {
  id: string;
  action: string;
  date: string;
  points: number;
}

interface ProfileData {
  profile: UserProfile;
  preferences: UserPreferences;
  stats: UserStats;
  history: HistoryEntry[];
}

export default function ProfileView() {
  const [data, setData] = useState<ProfileData | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);
  const [activeTab, setActiveTab] = useState<'profile' | 'preferences' | 'stats' | 'history'>('profile');

  useEffect(() => {
    const fetchProfileData = async () => {
      try {
        setLoading(true);
        const response = await fetch('http://localhost:8000/api/users/me');
        if (!response.ok) {
          throw new Error('Échec de la récupération du profil');
        }
        const profileData = await response.json();
        setData(profileData);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Une erreur est survenue');
      } finally {
        setLoading(false);
      }
    };

    fetchProfileData();
  }, []);

  const handleUpdateProfile = async (updatedProfile: UserProfile) => {
    try {
      const response = await fetch('http://localhost:8000/api/users/me', {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(updatedProfile),
      });
      if (!response.ok) {
        throw new Error('Échec de la mise à jour du profil');
      }
      const updatedData = await response.json();
      setData((prev) => prev ? { ...prev, profile: updatedData.profile } : null);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Une erreur est survenue');
    }
  };

  const handleUpdatePreferences = async (updatedPreferences: UserPreferences) => {
    try {
      const response = await fetch('http://localhost:8000/api/users/me/preferences', {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(updatedPreferences),
      });
      if (!response.ok) {
        throw new Error('Échec de la mise à jour des préférences');
      }
      const updatedData = await response.json();
      setData((prev) => prev ? { ...prev, preferences: updatedData.preferences } : null);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Une erreur est survenue');
    }
  };

  if (loading) {
    return (
      <div className="flex items-center justify-center h-screen">
        <div className="animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-green-500"></div>
      </div>
    );
  }

  if (error) {
    return (
      <div className="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded">
        {error}
      </div>
    );
  }

  if (!data) {
    return (
      <div className="bg-gray-100 border border-gray-300 text-gray-700 px-4 py-3 rounded">
        Aucune donnée disponible.
      </div>
    );
  }

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold text-gray-800">Profil</h1>

      {/* Onglets */}
      <div className="flex space-x-2 border-b border-gray-200">
        <button
          onClick={() => setActiveTab('profile')}
          className={`px-4 py-2 text-sm font-medium ${
            activeTab === 'profile'
              ? 'text-green-600 border-b-2 border-green-600'
              : 'text-gray-500 hover:text-gray-700'
          }`}
        >
          Mon Profil
        </button>
        <button
          onClick={() => setActiveTab('preferences')}
          className={`px-4 py-2 text-sm font-medium ${
            activeTab === 'preferences'
              ? 'text-green-600 border-b-2 border-green-600'
              : 'text-gray-500 hover:text-gray-700'
          }`}
        >
          Préférences
        </button>
        <button
          onClick={() => setActiveTab('stats')}
          className={`px-4 py-2 text-sm font-medium ${
            activeTab === 'stats'
              ? 'text-green-600 border-b-2 border-green-600'
              : 'text-gray-500 hover:text-gray-700'
          }`}
        >
          Statistiques
        </button>
        <button
          onClick={() => setActiveTab('history')}
          className={`px-4 py-2 text-sm font-medium ${
            activeTab === 'history'
              ? 'text-green-600 border-b-2 border-green-600'
              : 'text-gray-500 hover:text-gray-700'
          }`}
        >
          Historique
        </button>
      </div>

      {/* Contenu des onglets */}
      <div className="bg-white rounded-lg shadow-md p-4">
        {activeTab === 'profile' && (
          <ProfileForm profile={data.profile} onUpdate={handleUpdateProfile} />
        )}
        {activeTab === 'preferences' && (
          <PreferencesForm preferences={data.preferences} onUpdate={handleUpdatePreferences} />
        )}
        {activeTab === 'stats' && (
          <StatsDisplay stats={data.stats} />
        )}
        {activeTab === 'history' && (
          <HistoryList history={data.history} />
        )}
      </div>
    </div>
  );
}
```

---

### Implémentation du Formulaire de Profil (`ProfileForm.tsx`)
```tsx
'use client';

import { useState } from 'react';

interface UserProfile {
  id: string;
  email: string;
  timezone: string;
  homeLocation: string;
  workLocation: string;
}

interface ProfileFormProps {
  profile: UserProfile;
  onUpdate: (updatedProfile: UserProfile) => void;
}

export default function ProfileForm({ profile, onUpdate }: ProfileFormProps) {
  const [formData, setFormData] = useState<UserProfile>(profile);
  const [isSubmitting, setIsSubmitting] = useState<boolean>(false);

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData((prev) => ({ ...prev, [name]: value }));
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsSubmitting(true);
    try {
      await onUpdate(formData);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <h2 className="text-xl font-semibold text-gray-700 mb-4">Mon Profil</h2>

      <div className="space-y-4">
        <div>
          <label htmlFor="email" className="block text-sm font-medium text-gray-700 mb-1">
            Email
          </label>
          <input
            type="email"
            id="email"
            name="email"
            value={formData.email}
            onChange={handleChange}
            className="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-green-500"
            required
          />
        </div>

        <div>
          <label htmlFor="timezone" className="block text-sm font-medium text-gray-700 mb-1">
            Fuseau horaire
          </label>
          <select
            id="timezone"
            name="timezone"
            value={formData.timezone}
            onChange={(e) => setFormData((prev) => ({ ...prev, timezone: e.target.value }))}
            className="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-green-500"
            required
          >
            <option value="Europe/Paris">Europe/Paris</option>
            <option value="Europe/London">Europe/London</option>
            <option value="America/New_York">America/New_York</option>
            {/* Ajouter d'autres fuseaux horaires si nécessaire */}
          </select>
        </div>

        <div>
          <label htmlFor="homeLocation" className="block text-sm font-medium text-gray-700 mb-1">
            Localisation à la maison
          </label>
          <input
            type="text"
            id="homeLocation"
            name="homeLocation"
            value={formData.homeLocation}
            onChange={handleChange}
            placeholder="Ex: Lyon 69003"
            className="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-green-500"
            required
          />
        </div>

        <div>
          <label htmlFor="workLocation" className="block text-sm font-medium text-gray-700 mb-1">
            Localisation au travail
          </label>
          <input
            type="text"
            id="workLocation"
            name="workLocation"
            value={formData.workLocation}
            onChange={handleChange}
            placeholder="Ex: Lyon 69002"
            className="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-green-500"
          />
        </div>
      </div>

      <button
        type="submit"
        disabled={isSubmitting}
        className="bg-green-600 text-white px-4 py-2 rounded-lg hover:bg-green-700 transition-colors disabled:opacity-50"
      >
        {isSubmitting ? 'Enregistrement...' : 'Enregistrer'}
      </button>
    </form>
  );
}
```

---

### Implémentation du Formulaire de Préférences (`PreferencesForm.tsx`)
```tsx
'use client';

import { useState } from 'react';

interface UserPreferences {
  activities: string[];
  transport: string[];
  maxDuration: number;
}

interface PreferencesFormProps {
  preferences: UserPreferences;
  onUpdate: (updatedPreferences: UserPreferences) => void;
}

// Options pour les activités
const activityOptions = [
  { id: 'nature', label: 'Nature' },
  { id: 'jardinage', label: 'Jardinage' },
  { id: 'reparation', label: 'Réparation' },
  { id: 'reemploi', label: 'Réemploi' },
  { id: 'bien-etre', label: 'Bien-être' },
];

// Options pour les transports
const transportOptions = [
  { id: 'velo', label: 'Vélo' },
  { id: 'marche', label: 'Marche' },
  { id: 'transports-en-commun', label: 'Transports en commun' },
  { id: 'covoiturage', label: 'Covoiturage' },
];

export default function PreferencesForm({ preferences, onUpdate }: PreferencesFormProps) {
  const [formData, setFormData] = useState<UserPreferences>(preferences);
  const [isSubmitting, setIsSubmitting] = useState<boolean>(false);

  const handleActivityChange = (activityId: string) => {
    setFormData((prev) => {
      const newActivities = prev.activities.includes(activityId)
        ? prev.activities.filter((id) => id !== activityId)
        : [...prev.activities, activityId];
      return { ...prev, activities: newActivities };
    });
  };

  const handleTransportChange = (transportId: string) => {
    setFormData((prev) => {
      const newTransport = prev.transport.includes(transportId)
        ? prev.transport.filter((id) => id !== transportId)
        : [...prev.transport, transportId];
      return { ...prev, transport: newTransport };
    });
  };

  const handleDurationChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setFormData((prev) => ({ ...prev, maxDuration: parseInt(e.target.value) || 0 }));
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsSubmitting(true);
    try {
      await onUpdate(formData);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <h2 className="text-xl font-semibold text-gray-700 mb-4">Mes Préférences</h2>

      <div className="space-y-6">
        {/* Activités */}
        <div>
          <h3 className="text-lg font-medium text-gray-700 mb-3">Activités préférées</h3>
          <div className="flex flex-wrap gap-2">
            {activityOptions.map((option) => (
              <button
                key={option.id}
                type="button"
                onClick={() => handleActivityChange(option.id)}
                className={`px-3 py-2 rounded-full text-sm transition-colors ${
                  formData.activities.includes(option.id)
                    ? 'bg-green-600 text-white'
                    : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
                }`}
              >
                {option.label}
              </button>
            ))}
          </div>
        </div>

        {/* Transports */}
        <div>
          <h3 className="text-lg font-medium text-gray-700 mb-3">Moyens de transport</h3>
          <div className="flex flex-wrap gap-2">
            {transportOptions.map((option) => (
              <button
                key={option.id}
                type="button"
                onClick={() => handleTransportChange(option.id)}
                className={`px-3 py-2 rounded-full text-sm transition-colors ${
                  formData.transport.includes(option.id)
                    ? 'bg-green-600 text-white'
                    : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
                }`}
              >
                {option.label}
              </button>
            ))}
          </div>
        </div>

        {/* Durée maximale */}
        <div>
          <h3 className="text-lg font-medium text-gray-700 mb-3">Durée maximale par activité</h3>
          <div className="flex items-center space-x-4">
            <input
              type="number"
              id="maxDuration"
              value={formData.maxDuration}
              onChange={handleDurationChange}
              min="0"
              className="w-20 px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-green-500"
            />
            <span className="text-gray-700">minutes</span>
          </div>
        </div>
      </div>

      <button
        type="submit"
        disabled={isSubmitting}
        className="bg-green-600 text-white px-4 py-2 rounded-lg hover:bg-green-700 transition-colors disabled:opacity-50"
      >
        {isSubmitting ? 'Enregistrement...' : 'Enregistrer'}
      </button>
    </form>
  );
}
```

---

### Implémentation de l'Affichage des Statistiques (`StatsDisplay.tsx`)
```tsx
interface UserStats {
  totalRecommendationsFollowed: number;
  totalPoints: number;
  averageScore: number;
}

interface StatsDisplayProps {
  stats: UserStats;
}

export default function StatsDisplay({ stats }: StatsDisplayProps) {
  return (
    <div className="space-y-4">
      <h2 className="text-xl font-semibold text-gray-700 mb-4">Mes Statistiques</h2>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        {/* Recommandations suivies */}
        <div className="bg-gray-50 rounded-lg p-4 text-center">
          <h3 className="text-3xl font-bold text-green-600">{stats.totalRecommendationsFollowed}</h3>
          <p className="text-gray-600 mt-1">Recommandations suivies</p>
        </div>

        {/* Points totaux */}
        <div className="bg-gray-50 rounded-lg p-4 text-center">
          <h3 className="text-3xl font-bold text-green-600">{stats.totalPoints}</h3>
          <p className="text-gray-600 mt-1">Points totaux</p>
        </div>

        {/* Score moyen */}
        <div className="bg-gray-50 rounded-lg p-4 text-center">
          <h3 className="text-3xl font-bold text-green-600">{stats.averageScore.toFixed(1)}</h3>
          <p className="text-gray-600 mt-1">Score moyen</p>
        </div>
      </div>
    </div>
  );
}
```

---

### Implémentation de la Liste de l'Historique (`HistoryList.tsx`)
```tsx
interface HistoryEntry {
  id: string;
  action: string;
  date: string;
  points: number;
}

interface HistoryListProps {
  history: HistoryEntry[];
}

export default function HistoryList({ history }: HistoryListProps) {
  if (history.length === 0) {
    return (
      <div className="space-y-4">
        <h2 className="text-xl font-semibold text-gray-700 mb-4">Mon Historique</h2>
        <p className="text-gray-500">Aucune action enregistrée pour le moment.</p>
      </div>
    );
  }

  return (
    <div className="space-y-4">
      <h2 className="text-xl font-semibold text-gray-700 mb-4">Mon Historique</h2>
      <div className="space-y-3">
        {history.map((entry) => (
          <div key={entry.id} className="bg-gray-50 rounded-lg p-3 flex justify-between items-center">
            <div>
              <h3 className="font-medium text-gray-800">{entry.action}</h3>
              <p className="text-sm text-gray-500">{entry.date}</p>
            </div>
            <div className="text-right">
              <p className="font-semibold text-green-600">+{entry.points} pts</p>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## Tests

### Tests d'Intégration
1. **Vérifier la gestion du profil** :
   - Vérifier que l'utilisateur peut modifier son profil (email, timezone, localisations).

2. **Vérifier la gestion des préférences** :
   - Vérifier que l'utilisateur peut configurer ses préférences (activités, transports, durée max).

3. **Vérifier l'affichage des statistiques** :
   - Vérifier que les statistiques personnelles s'affichent correctement.

4. **Vérifier l'affichage de l'historique** :
   - Vérifier que l'historique des actions s'affiche correctement.

---

## Configuration Requise

### Backend
Assurez-vous que les endpoints suivants sont disponibles :
- `GET /api/users/me` : Retourne les informations du profil, les préférences, les statistiques et l'historique de l'utilisateur.
- `PUT /api/users/me` : Met à jour le profil de l'utilisateur.
- `PUT /api/users/me/preferences` : Met à jour les préférences de l'utilisateur.

Exemple de réponse pour `GET /api/users/me` :
```json
{
  "profile": {
    "id": "user-001",
    "email": "julie@example.com",
    "timezone": "Europe/Paris",
    "homeLocation": "Lyon 69003",
    "workLocation": "Lyon 69002"
  },
  "preferences": {
    "activities": ["nature", "jardinage"],
    "transport": ["velo", "marche"],
    "maxDuration": 60
  },
  "stats": {
    "totalRecommendationsFollowed": 15,
    "totalPoints": 1250,
    "averageScore": 82.5
  },
  "history": [
    {
      "id": "hist-001",
      "action": "Balade à vélo",
      "date": "2026-08-15",
      "points": 120
    },
    {
      "id": "hist-002",
      "action": "Réparation de vélo",
      "date": "2026-08-16",
      "points": 50
    }
  ]
}
```

---

## Dépendances
- **Story 8.1** : Le layout mobile-first doit être implémenté.
- **Epic 5 (User Management)** : Pour gérer le profil, les préférences et l'historique de l'utilisateur.

---

## Ressources
- [Documentation Next.js](https://nextjs.org/docs)
- [Documentation Tailwind CSS](https://tailwindcss.com/docs)
- [PRD Almanéa - FR-36](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
