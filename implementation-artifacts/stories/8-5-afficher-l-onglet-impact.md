---
name: Afficher l'onglet Impact
id: 8-5-afficher-l-onglet-impact
epic: epic-8
story_type: frontend
priority: medium
estimation: M
dependencies: ["8-1-créer-le-layout-mobile-first", "epic-6"]
status: backlog
created: 2026-08-20
updated: 2026-08-20
---

# Story 8.5: Afficher l'onglet "Impact"

## Contexte
Cette story fait partie de **l'Épic 8 (Web UI)**. Son objectif est de créer l'**onglet "Impact"** qui permet aux utilisateurs de **suivre leurs points, badges, défis, journal écologique et progrès**. Cela répond aux exigences **FR-36 (Affichage des points, badges, défis, et historique)**.

---

## Exigences Fonctionnelles
- **FR-36**: Affichage des points, badges, défis, journal écologique, et historique dans l'onglet "Impact".

---

## Critères d'Acceptation
1. **Affichage des points** :
   - [ ] Affiche le **total des points cumulés** (ex: 1 250 points).

2. **Affichage des badges** :
   - [ ] Affiche la **liste des badges débloqués** (ex: 🏆 Réparateur, 🌿 Explorateur urbain).

3. **Affichage des graphiques de progrès** :
   - [ ] Affiche des **graphiques** (ex: évolution des points sur 30 jours).

4. **Affichage du journal écologique** :
   - [ ] Affiche le **journal écologique** (ex: liste des actions réalisées avec date, durée, points gagnés, catégorie).

5. **Affichage des défis en cours** :
   - [ ] Affiche les **défis en cours** (ex: "7 jours de mobilité douce").

6. **Intégration des données** :
   - [ ] Les données proviennent du **Gamification Engine** (Epic 6).

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `frontend/app/impact/page.tsx` | Page de l'onglet "Impact" | À créer |
| `frontend/components/impact/ImpactView.tsx` | Composant principal de l'onglet "Impact" | À créer |
| `frontend/components/impact/PointsDisplay.tsx` | Affichage des points | À créer |
| `frontend/components/impact/BadgesList.tsx` | Liste des badges | À créer |
| `frontend/components/impact/ProgressChart.tsx` | Graphique de progrès | À créer |
| `frontend/components/impact/EcoJournal.tsx` | Journal écologique | À créer |
| `frontend/components/impact/ChallengesList.tsx` | Liste des défis | À créer |

---

### Implémentation de la Page (`page.tsx`)
```tsx
import ImpactView from '@/components/impact/ImpactView';

export default function ImpactPage() {
  return (
    <div className="p-4">
      <ImpactView />
    </div>
  );
}
```

---

### Implémentation de la Vue Impact (`ImpactView.tsx`)
```tsx
'use client';

import { useEffect, useState } from 'react';
import PointsDisplay from './PointsDisplay';
import BadgesList from './BadgesList';
import ProgressChart from './ProgressChart';
import EcoJournal from './EcoJournal';
import ChallengesList from './ChallengesList';

interface Badge {
  id: string;
  name: string;
  emoji: string;
  description: string;
  unlockedAt: string;
}

interface JournalEntry {
  id: string;
  action: string;
  date: string;
  duration: string;
  points: number;
  category: string;
}

interface Challenge {
  id: string;
  name: string;
  description: string;
  progress: number;
  target: number;
}

interface UserStats {
  totalPoints: number;
  badges: Badge[];
  journal: JournalEntry[];
  challenges: Challenge[];
  dailyPoints: { date: string; points: number }[];
}

export default function ImpactView() {
  const [stats, setStats] = useState<UserStats | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchStats = async () => {
      try {
        setLoading(true);
        const response = await fetch('http://localhost:8000/api/users/me/stats');
        if (!response.ok) {
          throw new Error('Échec de la récupération des statistiques');
        }
        const data = await response.json();
        setStats(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Une erreur est survenue');
      } finally {
        setLoading(false);
      }
    };

    fetchStats();
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

  if (!stats) {
    return (
      <div className="bg-gray-100 border border-gray-300 text-gray-700 px-4 py-3 rounded">
        Aucune donnée disponible.
      </div>
    );
  }

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold text-gray-800">Impact</h1>

      {/* Points totaux */}
      <PointsDisplay totalPoints={stats.totalPoints} />

      {/* Badges */}
      <BadgesList badges={stats.badges} />

      {/* Graphique de progrès */}
      <ProgressChart dailyPoints={stats.dailyPoints} />

      {/* Journal écologique */}
      <EcoJournal journal={stats.journal} />

      {/* Défis en cours */}
      <ChallengesList challenges={stats.challenges} />
    </div>
  );
}
```

---

### Implémentation de l'Affichage des Points (`PointsDisplay.tsx`)
```tsx
interface PointsDisplayProps {
  totalPoints: number;
}

export default function PointsDisplay({ totalPoints }: PointsDisplayProps) {
  return (
    <div className="bg-gradient-to-r from-green-500 to-green-600 rounded-lg p-6 text-white shadow-md">
      <div className="flex items-center justify-between">
        <div>
          <h2 className="text-2xl font-bold">Points totaux</h2>
          <p className="text-green-100 mt-1">Cumulés depuis votre inscription</p>
        </div>
        <div className="text-4xl font-bold">{totalPoints}</div>
      </div>
    </div>
  );
}
```

---

### Implémentation de la Liste des Badges (`BadgesList.tsx`)
```tsx
interface Badge {
  id: string;
  name: string;
  emoji: string;
  description: string;
  unlockedAt: string;
}

interface BadgesListProps {
  badges: Badge[];
}

export default function BadgesList({ badges }: BadgesListProps) {
  if (badges.length === 0) {
    return (
      <div className="bg-white rounded-lg shadow-md p-4">
        <h2 className="text-xl font-semibold text-gray-700 mb-4">Badges</h2>
        <p className="text-gray-500">Aucun badge débloqué pour le moment.</p>
      </div>
    );
  }

  return (
    <div className="bg-white rounded-lg shadow-md p-4">
      <h2 className="text-xl font-semibold text-gray-700 mb-4">Badges</h2>
      <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
        {badges.map((badge) => (
          <div key={badge.id} className="bg-gray-50 rounded-lg p-4 text-center">
            <div className="text-3xl mb-2">{badge.emoji}</div>
            <h3 className="font-semibold text-gray-800">{badge.name}</h3>
            <p className="text-sm text-gray-600 mt-1">{badge.description}</p>
            <p className="text-xs text-gray-500 mt-2">Débloqué le {badge.unlockedAt}</p>
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

### Implémentation du Graphique de Progrès (`ProgressChart.tsx`)
```tsx
'use client';

import { Line } from 'react-chartjs-2';
import { Chart as ChartJS, CategoryScale, LinearScale, PointElement, LineElement, Title, Tooltip, Legend } from 'chart.js';

ChartJS.register(CategoryScale, LinearScale, PointElement, LineElement, Title, Tooltip, Legend);

interface DailyPoint {
  date: string;
  points: number;
}

interface ProgressChartProps {
  dailyPoints: DailyPoint[];
}

export default function ProgressChart({ dailyPoints }: ProgressChartProps) {
  // Préparer les données pour le graphique
  const dates = dailyPoints.map((day) => day.date);
  const points = dailyPoints.map((day) => day.points);

  const data = {
    labels: dates,
    datasets: [
      {
        label: 'Points quotidiens',
        data: points,
        borderColor: '#16a34a', // Vert principal
        backgroundColor: 'rgba(22, 163, 74, 0.1)',
        tension: 0.4,
        fill: true,
      },
    ],
  };

  const options = {
    responsive: true,
    plugins: {
      legend: {
        position: 'top' as const,
      },
      title: {
        display: true,
        text: 'Évolution des points sur 30 jours',
      },
    },
    scales: {
      y: {
        beginAtZero: true,
      },
    },
  };

  return (
    <div className="bg-white rounded-lg shadow-md p-4">
      <h2 className="text-xl font-semibold text-gray-700 mb-4">Progrès</h2>
      <div className="h-64">
        <Line data={data} options={options} />
      </div>
    </div>
  );
}
```

---

### Implémentation du Journal Écologique (`EcoJournal.tsx`)
```tsx
interface JournalEntry {
  id: string;
  action: string;
  date: string;
  duration: string;
  points: number;
  category: string;
}

interface EcoJournalProps {
  journal: JournalEntry[];
}

// Mapper les catégories aux icônes
const categoryIcons: Record<string, string> = {
  nature: '🌿',
  energie: '⚡',
  mobilite: '🚲',
  jardinage: '🌱',
  reparation: '🔧',
  reemploi: '♻️',
  'bien-etre': '🧘',
};

export default function EcoJournal({ journal }: EcoJournalProps) {
  if (journal.length === 0) {
    return (
      <div className="bg-white rounded-lg shadow-md p-4">
        <h2 className="text-xl font-semibold text-gray-700 mb-4">Journal Écologique</h2>
        <p className="text-gray-500">Aucune action enregistrée pour le moment.</p>
      </div>
    );
  }

  return (
    <div className="bg-white rounded-lg shadow-md p-4">
      <h2 className="text-xl font-semibold text-gray-700 mb-4">Journal Écologique</h2>
      <div className="space-y-4">
        {journal.map((entry) => (
          <div key={entry.id} className="bg-gray-50 rounded-lg p-3 flex items-center space-x-4">
            <div className="text-2xl">{categoryIcons[entry.category] || '❓'}</div>
            <div className="flex-1">
              <h3 className="font-semibold text-gray-800">{entry.action}</h3>
              <p className="text-sm text-gray-600">
                {entry.date} • {entry.duration}
              </p>
            </div>
            <div className="text-right">
              <p className="font-semibold text-green-600">+{entry.points} pts</p>
              <p className="text-xs text-gray-500">{entry.category}</p>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

### Implémentation de la Liste des Défis (`ChallengesList.tsx`)
```tsx
interface Challenge {
  id: string;
  name: string;
  description: string;
  progress: number;
  target: number;
}

interface ChallengesListProps {
  challenges: Challenge[];
}

export default function ChallengesList({ challenges }: ChallengesListProps) {
  if (challenges.length === 0) {
    return (
      <div className="bg-white rounded-lg shadow-md p-4">
        <h2 className="text-xl font-semibold text-gray-700 mb-4">Défis en cours</h2>
        <p className="text-gray-500">Aucun défi en cours pour le moment.</p>
      </div>
    );
  }

  return (
    <div className="bg-white rounded-lg shadow-md p-4">
      <h2 className="text-xl font-semibold text-gray-700 mb-4">Défis en cours</h2>
      <div className="space-y-4">
        {challenges.map((challenge) => {
          const progressPercentage = Math.min((challenge.progress / challenge.target) * 100, 100);
          return (
            <div key={challenge.id} className="bg-gray-50 rounded-lg p-4">
              <div className="flex justify-between items-start mb-2">
                <div>
                  <h3 className="font-semibold text-gray-800">{challenge.name}</h3>
                  <p className="text-sm text-gray-600">{challenge.description}</p>
                </div>
                <span className="text-sm font-medium text-gray-700">
                  {challenge.progress}/{challenge.target}
                </span>
              </div>
              <div className="w-full bg-gray-200 rounded-full h-2">
                <div
                  className="bg-green-600 h-2 rounded-full"
                  style={{ width: `${progressPercentage}%` }}
                ></div>
              </div>
            </div>
          );
        })}
      </div>
    </div>
  );
}
```

---

## Tests

### Tests d'Intégration
1. **Vérifier l'affichage des points** :
   - Vérifier que le total des points s'affiche correctement.

2. **Vérifier l'affichage des badges** :
   - Vérifier que les badges débloqués s'affichent avec leurs icônes, noms et descriptions.

3. **Vérifier l'affichage du graphique de progrès** :
   - Vérifier que le graphique affiche correctement l'évolution des points.

4. **Vérifier l'affichage du journal écologique** :
   - Vérifier que les actions enregistrées s'affichent avec leurs détails (date, durée, points, catégorie).

5. **Vérifier l'affichage des défis** :
   - Vérifier que les défis en cours s'affichent avec leur progression.

---

## Configuration Requise

### Backend
Assurez-vous que l'endpoint suivant est disponible :
- `GET /api/users/me/stats` : Retourne les statistiques de l'utilisateur (points, badges, journal, défis, progrès quotidiens).

Exemple de réponse :
```json
{
  "totalPoints": 1250,
  "badges": [
    {
      "id": "badge-001",
      "name": "Réparateur",
      "emoji": "🏆",
      "description": "5 actions de réparation",
      "unlockedAt": "2026-08-15"
    },
    {
      "id": "badge-002",
      "name": "Explorateur urbain",
      "emoji": "🌿",
      "description": "10 découvertes locales",
      "unlockedAt": "2026-08-18"
    }
  ],
  "journal": [
    {
      "id": "entry-001",
      "action": "Balade à vélo",
      "date": "2026-08-15",
      "duration": "30 min",
      "points": 120,
      "category": "mobilite"
    },
    {
      "id": "entry-002",
      "action": "Réparation de vélo",
      "date": "2026-08-16",
      "duration": "1h",
      "points": 50,
      "category": "reparation"
    }
  ],
  "challenges": [
    {
      "id": "challenge-001",
      "name": "7 jours de mobilité douce",
      "description": "Utilisez des moyens de transport doux pendant 7 jours consécutifs",
      "progress": 3,
      "target": 7
    }
  ],
  "dailyPoints": [
    { "date": "2026-08-01", "points": 50 },
    { "date": "2026-08-02", "points": 120 },
    { "date": "2026-08-03", "points": 80 },
    { "date": "2026-08-04", "points": 200 }
  ]
}
```

---

### Dépendances Frontend
- **chart.js** et **react-chartjs-2** pour les graphiques.

Installer avec :
```bash
npm install chart.js react-chartjs-2
```

---

## Dépendances
- **Story 8.1** : Le layout mobile-first doit être implémenté.
- **Epic 6 (Gamification)** : Pour récupérer les données de points, badges, journal et défis.

---

## Ressources
- [Documentation Next.js](https://nextjs.org/docs)
- [Documentation Tailwind CSS](https://tailwindcss.com/docs)
- [Documentation Chart.js](https://www.chartjs.org/docs/latest/)
- [PRD Almanéa - FR-36](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
