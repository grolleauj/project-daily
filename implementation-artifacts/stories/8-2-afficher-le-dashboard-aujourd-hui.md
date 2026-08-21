---
name: Afficher le dashboard Aujourd'hui
id: 8-2-afficher-le-dashboard-aujourd-hui
epic: epic-8
story_type: frontend
priority: high
estimation: M
dependencies: ["8-1-créer-le-layout-mobile-first", "epic-1", "epic-2"]
status: backlog
created: 2026-08-20
updated: 2026-08-20
---

# Story 8.2: Afficher le dashboard "Aujourd'hui"

## Contexte
Cette story fait partie de **l'Épic 8 (Web UI)**. Son objectif est de créer le **dashboard principal** qui affiche le **contexte environnemental** (météo, qualité de l'air, précipitations) et la **section "Aujourd'hui pour vous"** avec les recommandations personnalisées. Cela répond aux exigences **FR-34 (Dashboard principal)** et **UX-DR-2 (Affichage du contexte environnemental)**.

---

## Exigences Fonctionnelles
- **FR-34**: Dashboard principal avec header (météo, qualité de l'air) et section "Aujourd'hui pour vous".
- **UX-DR-2**: Affichage du contexte environnemental (météo, AQI, précipitations, variations).

---

## Critères d'Acceptation
1. **Header** :
   - [ ] Affiche l'**heure locale** (ex: 9:41).
   - [ ] Affiche la **localisation** (ex: Lyon 69003).
   - [ ] Affiche la **météo** :
     - Température (ex: 18° Ensoleillé).
     - Précipitations (ex: Pluie faible 10%).
     - Variations (ex: ↑ 21° • ↓ 12°).
   - [ ] Affiche la **qualité de l'air** :
     - AQI (ex: 42).
     - Label (ex: Air bon).

2. **Section "Aujourd'hui pour vous"** :
   - [ ] Affiche une **liste de cartes de recommandations** avec :
     - Icône (ex: 🚲, 💧, 🔧).
     - Titre (ex: "Une belle journée pour sortir à vélo").
     - Description (ex: "Itinéraire de 35 min à 1.8 km de chez vous").

3. **Intégration des données** :
   - [ ] Les données du **header** proviennent du **Context Engine** (Epic 1).
   - [ ] Les **recommandations** proviennent du **Recommendation Engine** (Epic 2).

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `frontend/app/aujourdhui/page.tsx` | Page du dashboard "Aujourd'hui" | À créer |
| `frontend/components/dashboard/TodayDashboard.tsx` | Composant principal du dashboard | À créer |
| `frontend/components/dashboard/RecommendationCard.tsx` | Carte de recommandation | À créer |
| `frontend/components/dashboard/EnvironmentalContext.tsx` | Composant pour le contexte environnemental | À créer |
| `frontend/lib/api.ts` | Fonctions pour récupérer les données du backend | À créer |

---

### Implémentation de la Page (`page.tsx`)
```tsx
import TodayDashboard from '@/components/dashboard/TodayDashboard';

export default function AujourdHuiPage() {
  return (
    <div className="p-4">
      <TodayDashboard />
    </div>
  );
}
```

---

### Implémentation du Dashboard (`TodayDashboard.tsx`)
```tsx
'use client';

import { useEffect, useState } from 'react';
import EnvironmentalContext from './EnvironmentalContext';
import RecommendationCard from './RecommendationCard';

interface WeatherData {
  temperature: string;
  condition: string;
  precipitation: string;
  variations: string;
}

interface AirQualityData {
  aqi: number;
  label: string;
}

interface Recommendation {
  id: string;
  icon: string;
  title: string;
  description: string;
  score: number;
}

interface UnifiedContext {
  weather: WeatherData;
  airQuality: AirQualityData;
  location: string;
}

export default function TodayDashboard() {
  const [context, setContext] = useState<UnifiedContext | null>(null);
  const [recommendations, setRecommendations] = useState<Recommendation[]>([]);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        
        // Récupérer le contexte environnemental
        const contextResponse = await fetch('http://localhost:8000/api/context/unified');
        if (!contextResponse.ok) {
          throw new Error('Échec de la récupération du contexte');
        }
        const contextData = await contextResponse.json();
        setContext(contextData);

        // Récupérer les recommandations
        const recommendationsResponse = await fetch('http://localhost:8000/api/recommendations');
        if (!recommendationsResponse.ok) {
          throw new Error('Échec de la récupération des recommandations');
        }
        const recommendationsData = await recommendationsResponse.json();
        setRecommendations(recommendationsData);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Une erreur est survenue');
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, []);

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
      {/* Contexte environnemental */}
      {context && (
        <EnvironmentalContext
          time={new Date().toLocaleTimeString('fr-FR', { hour: '2-digit', minute: '2-digit' })}
          location={context.location}
          weather={context.weather}
          airQuality={context.airQuality}
        />
      )}

      {/* Section "Aujourd'hui pour vous" */}
      <section className="bg-white rounded-lg shadow-md p-4">
        <h2 className="text-xl font-bold text-gray-800 mb-4">Aujourd'hui pour vous</h2>
        
        {recommendations.length > 0 ? (
          <div className="space-y-4">
            {recommendations.map((recommendation) => (
              <RecommendationCard
                key={recommendation.id}
                icon={recommendation.icon}
                title={recommendation.title}
                description={recommendation.description}
                score={recommendation.score}
              />
            ))}
          </div>
        ) : (
          <p className="text-gray-500">Aucune recommandation disponible pour le moment.</p>
        )}
      </section>
    </div>
  );
}
```

---

### Implémentation du Contexte Environnemental (`EnvironmentalContext.tsx`)
```tsx
interface WeatherData {
  temperature: string;
  condition: string;
  precipitation: string;
  variations: string;
}

interface AirQualityData {
  aqi: number;
  label: string;
}

interface EnvironmentalContextProps {
  time: string;
  location: string;
  weather: WeatherData;
  airQuality: AirQualityData;
}

export default function EnvironmentalContext({
  time,
  location,
  weather,
  airQuality,
}: EnvironmentalContextProps) {
  return (
    <header className="bg-gradient-to-b from-green-100 to-white rounded-lg p-4 shadow-sm">
      <div className="flex justify-between items-center mb-2">
        <span className="text-2xl font-bold">{time}</span>
        <span className="text-lg text-gray-600">{location}</span>
      </div>

      <div className="flex justify-between items-center">
        <div className="flex items-center space-x-4">
          {/* Météo */}
          <div className="flex items-center space-x-2">
            <span className="text-2xl">🌤️</span>
            <div>
              <span className="font-semibold">{weather.temperature}</span>
              <span className="text-sm text-gray-600"> {weather.condition}</span>
            </div>
          </div>

          {/* Qualité de l'air */}
          <div className="flex items-center space-x-2">
            <span className="text-2xl">💨</span>
            <div>
              <span className="font-semibold">{airQuality.aqi} AQI</span>
              <span className="text-sm text-gray-600"> {airQuality.label}</span>
            </div>
          </div>
        </div>

        {/* Précipitations et variations */}
        <div className="text-sm text-gray-600">
          <div>{weather.precipitation}</div>
          <div>{weather.variations}</div>
        </div>
      </div>
    </header>
  );
}
```

---

### Implémentation de la Carte de Recommandation (`RecommendationCard.tsx`)
```tsx
interface RecommendationCardProps {
  icon: string;
  title: string;
  description: string;
  score: number;
}

export default function RecommendationCard({
  icon,
  title,
  description,
  score,
}: RecommendationCardProps) {
  // Déterminer la couleur du score
  const getScoreColor = (score: number) => {
    if (score >= 80) return 'bg-green-500';
    if (score >= 60) return 'bg-blue-500';
    if (score >= 40) return 'bg-yellow-500';
    return 'bg-gray-500';
  };

  return (
    <div className="bg-white border border-gray-200 rounded-lg p-4 shadow-sm hover:shadow-md transition-shadow">
      <div className="flex items-start space-x-4">
        {/* Icône */}
        <div className="text-3xl">{icon}</div>

        {/* Contenu */}
        <div className="flex-1">
          <div className="flex justify-between items-start">
            <h3 className="font-semibold text-lg">{title}</h3>
            <div className={`text-xs text-white px-2 py-1 rounded-full ${getScoreColor(score)}`}>
              Score: {score}
            </div>
          </div>
          <p className="text-gray-600 mt-1">{description}</p>
        </div>
      </div>

      {/* Bouton Valider */}
      <div className="mt-4 flex justify-end">
        <button className="bg-green-600 text-white px-4 py-2 rounded-lg text-sm hover:bg-green-700 transition-colors">
          Valider
        </button>
      </div>
    </div>
  );
}
```

---

### Fonctions API (`api.ts`)
```tsx
// Fonction pour récupérer le contexte unifié
export async function fetchUnifiedContext(location: string) {
  const response = await fetch(`http://localhost:8000/api/context/unified?location=${location}`);
  if (!response.ok) {
    throw new Error('Échec de la récupération du contexte');
  }
  return response.json();
}

// Fonction pour récupérer les recommandations
export async function fetchRecommendations(userId: string) {
  const response = await fetch(`http://localhost:8000/api/recommendations?user_id=${userId}`);
  if (!response.ok) {
    throw new Error('Échec de la récupération des recommandations');
  }
  return response.json();
}
```

---

## Tests

### Tests d'Intégration
1. **Vérifier l'affichage du header** :
   - Vérifier que l'heure, la localisation, la météo et la qualité de l'air s'affichent correctement.

2. **Vérifier l'affichage des recommandations** :
   - Vérifier que les cartes de recommandations s'affichent avec les bonnes informations (icône, titre, description, score).

3. **Vérifier l'intégration des données** :
   - Vérifier que les données proviennent bien du backend (Context Engine et Recommendation Engine).

---

## Configuration Requise

### Backend
Assurez-vous que les endpoints suivants sont disponibles :
- `GET /api/context/unified` : Retourne le contexte environnemental unifié.
- `GET /api/recommendations` : Retourne les recommandations personnalisées pour l'utilisateur.

Exemple de réponse pour `/api/context/unified` :
```json
{
  "weather": {
    "temperature": "18°",
    "condition": "Ensoleillé",
    "precipitation": "Pluie faible 10%",
    "variations": "↑ 21° • ↓ 12°"
  },
  "airQuality": {
    "aqi": 42,
    "label": "Air bon"
  },
  "location": "Lyon 69003"
}
```

Exemple de réponse pour `/api/recommendations` :
```json
[
  {
    "id": "rec-001",
    "icon": "🚲",
    "title": "Une belle journée pour sortir à vélo",
    "description": "Itinéraire de 35 min à 1.8 km de chez vous",
    "score": 85
  },
  {
    "id": "rec-002",
    "icon": "🌿",
    "title": "Balade en forêt avec vos enfants",
    "description": "Qualité de l'air excellente et sentiers accessibles à 5 km",
    "score": 78
  }
]
```

---

## Dépendances
- **Story 8.1** : Le layout mobile-first doit être implémenté.
- **Epic 1 (Context Engine)** : Pour récupérer les données de contexte (météo, qualité de l'air).
- **Epic 2 (Recommendation Engine)** : Pour générer et récupérer les recommandations.

---

## Ressources
- [Documentation Next.js](https://nextjs.org/docs)
- [Documentation Tailwind CSS](https://tailwindcss.com/docs)
- [PRD Almanéa - FR-34](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - UX-DR-2](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
