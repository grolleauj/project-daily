---
name: Afficher l'onglet Explorer
id: 8-4-afficher-l-onglet-explorer
epic: epic-8
story_type: frontend
priority: medium
estimation: M
dependencies: ["8-1-créer-le-layout-mobile-first", "epic-2"]
status: backlog
created: 2026-08-20
updated: 2026-08-20
---

# Story 8.4: Afficher l'onglet "Explorer"

## Contexte
Cette story fait partie de **l'Épic 8 (Web UI)**. Son objectif est de créer l'**onglet "Explorer"** qui permet aux utilisateurs de **découvrir des opportunités par catégories** (nature, énergie, mobilité, etc.). Cela répond à l'exigence **FR-35 (Navigation Principale)**.

---

## Exigences Fonctionnelles
- **FR-35**: Navigation Principale (5 onglets + bouton central flottant pour déclarer une action).
- **FR-36**: Affichage des opportunités par catégories.

---

## Critères d'Acceptation
1. **Fonctionnalité de filtrage** :
   - [ ] L'utilisateur peut filtrer les opportunités par **catégories** :
     - Nature
     - Énergie
     - Mobilité
     - Jardinage
     - Réparation
     - Réemploi
     - Bien-être
   - [ ] L'utilisateur peut filtrer par **localisation** (si activée).
   - [ ] L'utilisateur peut filtrer par **disponibilité** (ex: aujourd'hui, cette semaine).

2. **Affichage des résultats** :
   - [ ] Une **liste des opportunités disponibles** est affichée après application des filtres.
   - [ ] Chaque opportunité est affichée sous forme de **carte** avec :
     - Icône
     - Titre
     - Description
     - Catégorie

3. **Expérience utilisateur** :
   - [ ] Les filtres sont **faciles à utiliser** sur mobile et desktop.
   - [ ] Les résultats sont **mis à jour en temps réel** lors de la sélection des filtres.

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `frontend/app/explorer/page.tsx` | Page de l'onglet "Explorer" | À créer |
| `frontend/components/explorer/ExplorerView.tsx` | Composant principal de l'onglet "Explorer" | À créer |
| `frontend/components/explorer/CategoryFilter.tsx` | Filtre par catégories | À créer |
| `frontend/components/explorer/OpportunityCard.tsx` | Carte d'opportunité | À créer |
| `frontend/components/explorer/LocationFilter.tsx` | Filtre par localisation | À créer |
| `frontend/components/explorer/AvailabilityFilter.tsx` | Filtre par disponibilité | À créer |

---

### Implémentation de la Page (`page.tsx`)
```tsx
import ExplorerView from '@/components/explorer/ExplorerView';

export default function ExplorerPage() {
  return (
    <div className="p-4">
      <ExplorerView />
    </div>
  );
}
```

---

### Implémentation de la Vue Explorer (`ExplorerView.tsx`)
```tsx
'use client';

import { useEffect, useState } from 'react';
import CategoryFilter from './CategoryFilter';
import LocationFilter from './LocationFilter';
import AvailabilityFilter from './AvailabilityFilter';
import OpportunityCard from './OpportunityCard';

// Catégories disponibles
const categories = [
  { id: 'nature', label: 'Nature', icon: '🌿' },
  { id: 'energie', label: 'Énergie', icon: '⚡' },
  { id: 'mobilite', label: 'Mobilité', icon: '🚲' },
  { id: 'jardinage', label: 'Jardinage', icon: '🌱' },
  { id: 'reparation', label: 'Réparation', icon: '🔧' },
  { id: 'reemploi', label: 'Réemploi', icon: '♻️' },
  { id: 'bien-etre', label: 'Bien-être', icon: '🧘' },
];

// Options de disponibilité
const availabilityOptions = [
  { id: 'aujourdhui', label: "Aujourd'hui" },
  { id: 'cette-semaine', label: 'Cette semaine' },
  { id: 'ce-weekend', label: 'Ce week-end' },
];

interface Opportunity {
  id: string;
  title: string;
  description: string;
  category: string;
  location?: string;
  availability: string;
  icon: string;
}

export default function ExplorerView() {
  const [selectedCategories, setSelectedCategories] = useState<string[]>([]);
  const [selectedLocation, setSelectedLocation] = useState<string>('');
  const [selectedAvailability, setSelectedAvailability] = useState<string>('');
  const [opportunities, setOpportunities] = useState<Opportunity[]>([]);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchOpportunities = async () => {
      try {
        setLoading(true);
        
        // Construire les paramètres de requête en fonction des filtres
        const params = new URLSearchParams();
        if (selectedCategories.length > 0) {
          params.append('categories', selectedCategories.join(','));
        }
        if (selectedLocation) {
          params.append('location', selectedLocation);
        }
        if (selectedAvailability) {
          params.append('availability', selectedAvailability);
        }

        const response = await fetch(`http://localhost:8000/api/opportunities?${params.toString()}`);
        if (!response.ok) {
          throw new Error('Échec de la récupération des opportunités');
        }
        const data = await response.json();
        setOpportunities(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Une erreur est survenue');
      } finally {
        setLoading(false);
      }
    };

    fetchOpportunities();
  }, [selectedCategories, selectedLocation, selectedAvailability]);

  const handleCategoryChange = (categoryId: string) => {
    setSelectedCategories((prev) =>
      prev.includes(categoryId) ? prev.filter((id) => id !== categoryId) : [...prev, categoryId]
    );
  };

  const handleLocationChange = (location: string) => {
    setSelectedLocation(location);
  };

  const handleAvailabilityChange = (availability: string) => {
    setSelectedAvailability(availability);
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

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold text-gray-800">Explorer</h1>

      {/* Filtres */}
      <div className="space-y-4">
        {/* Filtre par catégories */}
        <CategoryFilter
          categories={categories}
          selectedCategories={selectedCategories}
          onCategoryChange={handleCategoryChange}
        />

        {/* Filtre par localisation */}
        <LocationFilter
          value={selectedLocation}
          onChange={handleLocationChange}
        />

        {/* Filtre par disponibilité */}
        <AvailabilityFilter
          options={availabilityOptions}
          selectedOption={selectedAvailability}
          onChange={handleAvailabilityChange}
        />
      </div>

      {/* Résultats */}
      <div className="bg-white rounded-lg shadow-md p-4">
        <h2 className="text-xl font-semibold text-gray-700 mb-4">
          Opportunités disponibles
        </h2>

        {opportunities.length > 0 ? (
          <div className="space-y-4">
            {opportunities.map((opportunity) => (
              <OpportunityCard
                key={opportunity.id}
                opportunity={opportunity}
              />
            ))}
          </div>
        ) : (
          <p className="text-gray-500">Aucune opportunité trouvée avec les filtres actuels.</p>
        )}
      </div>
    </div>
  );
}
```

---

### Implémentation du Filtre par Catégories (`CategoryFilter.tsx`)
```tsx
interface Category {
  id: string;
  label: string;
  icon: string;
}

interface CategoryFilterProps {
  categories: Category[];
  selectedCategories: string[];
  onCategoryChange: (categoryId: string) => void;
}

export default function CategoryFilter({
  categories,
  selectedCategories,
  onCategoryChange,
}: CategoryFilterProps) {
  return (
    <div className="bg-white rounded-lg shadow-sm p-4">
      <h3 className="font-semibold text-gray-800 mb-3">Catégories</h3>
      <div className="flex flex-wrap gap-2">
        {categories.map((category) => (
          <button
            key={category.id}
            onClick={() => onCategoryChange(category.id)}
            className={`flex items-center space-x-1 px-3 py-2 rounded-full text-sm transition-colors ${
              selectedCategories.includes(category.id)
                ? 'bg-green-600 text-white'
                : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
            }`}
          >
            <span>{category.icon}</span>
            <span>{category.label}</span>
          </button>
        ))}
      </div>
    </div>
  );
}
```

---

### Implémentation du Filtre par Localisation (`LocationFilter.tsx`)
```tsx
interface LocationFilterProps {
  value: string;
  onChange: (location: string) => void;
}

export default function LocationFilter({ value, onChange }: LocationFilterProps) {
  return (
    <div className="bg-white rounded-lg shadow-sm p-4">
      <h3 className="font-semibold text-gray-800 mb-3">Localisation</h3>
      <input
        type="text"
        value={value}
        onChange={(e) => onChange(e.target.value)}
        placeholder="Ex: Lyon, Paris, 69003..."
        className="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-green-500"
      />
    </div>
  );
}
```

---

### Implémentation du Filtre par Disponibilité (`AvailabilityFilter.tsx`)
```tsx
interface Option {
  id: string;
  label: string;
}

interface AvailabilityFilterProps {
  options: Option[];
  selectedOption: string;
  onChange: (optionId: string) => void;
}

export default function AvailabilityFilter({
  options,
  selectedOption,
  onChange,
}: AvailabilityFilterProps) {
  return (
    <div className="bg-white rounded-lg shadow-sm p-4">
      <h3 className="font-semibold text-gray-800 mb-3">Disponibilité</h3>
      <div className="flex flex-wrap gap-2">
        {options.map((option) => (
          <button
            key={option.id}
            onClick={() => onChange(option.id)}
            className={`px-3 py-2 rounded-lg text-sm transition-colors ${
              selectedOption === option.id
                ? 'bg-green-600 text-white'
                : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
            }`}
          >
            {option.label}
          </button>
        ))}
      </div>
    </div>
  );
}
```

---

### Implémentation de la Carte d'Opportunité (`OpportunityCard.tsx`)
```tsx
interface Opportunity {
  id: string;
  title: string;
  description: string;
  category: string;
  location?: string;
  availability: string;
  icon: string;
}

interface OpportunityCardProps {
  opportunity: Opportunity;
}

// Mapper les IDs de catégorie aux labels et icônes
const categoryMap: Record<string, { label: string; icon: string }> = {
  nature: { label: 'Nature', icon: '🌿' },
  energie: { label: 'Énergie', icon: '⚡' },
  mobilite: { label: 'Mobilité', icon: '🚲' },
  jardinage: { label: 'Jardinage', icon: '🌱' },
  reparation: { label: 'Réparation', icon: '🔧' },
  reemploi: { label: 'Réemploi', icon: '♻️' },
  'bien-etre': { label: 'Bien-être', icon: '🧘' },
};

// Mapper les IDs de disponibilité aux labels
const availabilityMap: Record<string, string> = {
  aujourdhui: "Aujourd'hui",
  'cette-semaine': 'Cette semaine',
  'ce-weekend': 'Ce week-end',
};

export default function OpportunityCard({ opportunity }: OpportunityCardProps) {
  const category = categoryMap[opportunity.category] || { label: opportunity.category, icon: '❓' };
  const availability = availabilityMap[opportunity.availability] || opportunity.availability;

  return (
    <div className="bg-gray-50 border border-gray-200 rounded-lg p-4 hover:shadow-md transition-shadow">
      <div className="flex items-start space-x-4">
        {/* Icône */}
        <div className="text-3xl">{category.icon}</div>

        {/* Contenu */}
        <div className="flex-1">
          <div className="flex justify-between items-start">
            <div>
              <h3 className="font-semibold text-lg">{opportunity.title}</h3>
              <p className="text-gray-600 mt-1">{opportunity.description}</p>
            </div>
            <span className={`text-xs px-2 py-1 rounded-full bg-green-100 text-green-800`}>
              {category.label}
            </span>
          </div>

          {/* Localisation et disponibilité */}
          <div className="mt-3 flex justify-between items-center text-sm text-gray-500">
            <div className="flex items-center space-x-1">
              <span>📍</span>
              <span>{opportunity.location || 'Non spécifiée'}</span>
            </div>
            <div className="flex items-center space-x-1">
              <span>📅</span>
              <span>{availability}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
}
```

---

## Tests

### Tests d'Intégration
1. **Vérifier les filtres** :
   - Vérifier que les filtres par catégories, localisation et disponibilité fonctionnent correctement.
   - Vérifier que les résultats sont mis à jour en temps réel lors de la sélection des filtres.

2. **Vérifier l'affichage des opportunités** :
   - Vérifier que les cartes d'opportunités s'affichent avec les bonnes informations (icône, titre, description, catégorie, localisation, disponibilité).

3. **Vérifier l'expérience utilisateur** :
   - Vérifier que l'interface est intuitive et facile à utiliser sur mobile et desktop.

---

## Configuration Requise

### Backend
Assurez-vous que l'endpoint suivant est disponible :
- `GET /api/opportunities` : Retourne les opportunités filtrées par catégories, localisation et disponibilité.

Exemple de requête :
```
GET /api/opportunities?categories=nature,mobilite&location=Lyon&availability=aujourdhui
```

Exemple de réponse :
```json
[
  {
    "id": "opp-001",
    "title": "Balade en forêt de Saint-Germain",
    "description": "Parcours de 5 km dans une forêt préservée, idéal pour une balade en famille.",
    "category": "nature",
    "location": "Lyon 69003",
    "availability": "aujourdhui",
    "icon": "🌿"
  },
  {
    "id": "opp-002",
    "title": "Atelier de réparation de vélos",
    "description": "Apprenez à réparer votre vélo avec des experts locaux.",
    "category": "reparation",
    "location": "Lyon 69001",
    "availability": "cette-semaine",
    "icon": "🔧"
  }
]
```

---

## Dépendances
- **Story 8.1** : Le layout mobile-first doit être implémenté.
- **Epic 2 (Recommendation Engine)** : Pour récupérer les opportunités disponibles.

---

## Ressources
- [Documentation Next.js](https://nextjs.org/docs)
- [Documentation Tailwind CSS](https://tailwindcss.com/docs)
- [PRD Almanéa - FR-35](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
