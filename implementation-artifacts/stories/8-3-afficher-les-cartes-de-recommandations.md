---
name: Afficher les cartes de recommandations
id: 8-3-afficher-les-cartes-de-recommandations
epic: epic-8
story_type: frontend
priority: high
estimation: M
dependencies: ["8-1-créer-le-layout-mobile-first", "8-2-afficher-le-dashboard-aujourd-hui", "epic-2", "epic-3"]
status: backlog
created: 2026-08-20
updated: 2026-08-20
---

# Story 8.3: Afficher les cartes de recommandations

## Contexte
Cette story fait partie de **l'Épic 8 (Web UI)**. Son objectif est d'afficher des **cartes de recommandations détaillées** avec des **explications scientifiques** pour chaque recommandation. Cela répond aux exigences **FR-35 (Affichage des cartes de recommandations)**, **FR-36 (Explications scientifiques)**, et **UX-DR-3 (Cartes de recommandations)**.

---

## Exigences Fonctionnelles
- **FR-35**: Affichage des cartes de recommandations (icône, titre, description, explication scientifique).
- **FR-36**: Explications scientifiques pour chaque recommandation.
- **UX-DR-3**: Cartes de recommandations avec icône, titre, description, explication scientifique, et bouton "Valider".

---

## Critères d'Acceptation
1. **Contenu de la carte** :
   - [ ] Affiche une **icône** (ex: 🚲 pour vélo, 🌿 pour nature).
   - [ ] Affiche un **titre** (ex: "Balade en forêt avec vos enfants").
   - [ ] Affiche une **description** (ex: "Qualité de l'air excellente et sentiers accessibles à 5 km").
   - [ ] Affiche une **explication scientifique** (ex: "Source : Atmo France, AQI=42. Les balades en forêt améliorent la santé mentale.").
   - [ ] Affiche un **bouton "Valider"** pour ajouter l'action à son historique.

2. **Intégration des données** :
   - [ ] Les **recommandations** proviennent du **Recommendation Engine** (Epic 2).
   - [ ] Les **explications scientifiques** proviennent du **Knowledge Engine** (Epic 3).

3. **Expérience utilisateur** :
   - [ ] Les cartes sont **claires et lisibles** sur mobile et desktop.
   - [ ] Les cartes sont **cliquables** pour plus de détails.

---

## Implémentation Technique

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `frontend/components/recommendations/RecommendationCard.tsx` | Carte de recommandation améliorée | À créer |
| `frontend/components/recommendations/RecommendationList.tsx` | Liste des recommandations | À créer |
| `frontend/components/recommendations/ScientificExplanation.tsx` | Composant pour l'explication scientifique | À créer |
| `frontend/lib/api.ts` | Mise à jour des fonctions API | À modifier |

---

### Implémentation de la Carte de Recommandation (`RecommendationCard.tsx`)
```tsx
'use client';

import { useState } from 'react';
import ScientificExplanation from './ScientificExplanation';

interface Recommendation {
  id: string;
  icon: string;
  title: string;
  description: string;
  scientificExplanation: string;
  score: number;
  evidence?: {
    id: string;
    claim: string;
    source: string;
    evidenceLevel: string;
  };
}

interface RecommendationCardProps {
  recommendation: Recommendation;
  onValidate: (recommendationId: string) => void;
}

export default function RecommendationCard({ recommendation, onValidate }: RecommendationCardProps) {
  const [isExpanded, setIsExpanded] = useState<boolean>(false);

  // Déterminer la couleur du score
  const getScoreColor = (score: number) => {
    if (score >= 80) return 'bg-green-500';
    if (score >= 60) return 'bg-blue-500';
    if (score >= 40) return 'bg-yellow-500';
    return 'bg-gray-500';
  };

  // Déterminer la couleur du niveau de preuve
  const getEvidenceLevelColor = (level: string) => {
    switch (level) {
      case 'HIGH':
        return 'bg-green-100 text-green-800';
      case 'MEDIUM':
        return 'bg-yellow-100 text-yellow-800';
      case 'LOW':
        return 'bg-red-100 text-red-800';
      default:
        return 'bg-gray-100 text-gray-800';
    }
  };

  return (
    <div className="bg-white border border-gray-200 rounded-lg p-4 shadow-sm hover:shadow-md transition-shadow">
      <div className="flex items-start space-x-4">
        {/* Icône */}
        <div className="text-3xl">{recommendation.icon}</div>

        {/* Contenu principal */}
        <div className="flex-1">
          <div className="flex justify-between items-start">
            <div>
              <h3 className="font-semibold text-lg">{recommendation.title}</h3>
              <p className="text-gray-600 mt-1">{recommendation.description}</p>
            </div>
            <div className={`text-xs text-white px-2 py-1 rounded-full ${getScoreColor(recommendation.score)}`}>
              Score: {recommendation.score}
            </div>
          </div>

          {/* Explication scientifique (développée au clic) */}
          {isExpanded && (
            <div className="mt-4">
              <ScientificExplanation
                explanation={recommendation.scientificExplanation}
                evidence={recommendation.evidence}
              />
            </div>
          )}

          {/* Boutons */}
          <div className="mt-4 flex justify-between items-center">
            <button
              onClick={() => setIsExpanded(!isExpanded)}
              className="text-sm text-green-600 hover:text-green-800"
            >
              {isExpanded ? 'Masquer les détails' : 'Voir les détails'}
            </button>
            <button
              onClick={() => onValidate(recommendation.id)}
              className="bg-green-600 text-white px-4 py-2 rounded-lg text-sm hover:bg-green-700 transition-colors"
            >
              Valider
            </button>
          </div>
        </div>
      </div>
    </div>
  );
}
```

---

### Implémentation de l'Explication Scientifique (`ScientificExplanation.tsx`)
```tsx
interface Evidence {
  id: string;
  claim: string;
  source: string;
  evidenceLevel: string;
}

interface ScientificExplanationProps {
  explanation: string;
  evidence?: Evidence;
}

export default function ScientificExplanation({ explanation, evidence }: ScientificExplanationProps) {
  // Déterminer la couleur du niveau de preuve
  const getEvidenceLevelColor = (level: string) => {
    switch (level) {
      case 'HIGH':
        return 'bg-green-100 text-green-800';
      case 'MEDIUM':
        return 'bg-yellow-100 text-yellow-800';
      case 'LOW':
        return 'bg-red-100 text-red-800';
      default:
        return 'bg-gray-100 text-gray-800';
    }
  };

  return (
    <div className="bg-gray-50 rounded-lg p-3 mt-2">
      <p className="text-sm text-gray-700">{explanation}</p>
      
      {evidence && (
        <div className="mt-3 pt-3 border-t border-gray-200">
          <div className="flex items-center space-x-2">
            <span className="text-sm font-medium">Source:</span>
            <span className="text-sm text-blue-600">{evidence.source}</span>
          </div>
          <div className="flex items-center space-x-2 mt-1">
            <span className="text-sm font-medium">Niveau de preuve:</span>
            <span className={`text-xs px-2 py-1 rounded-full ${getEvidenceLevelColor(evidence.evidenceLevel)}`}>
              {evidence.evidenceLevel}
            </span>
          </div>
        </div>
      )}
    </div>
  );
}
```

---

### Implémentation de la Liste de Recommandations (`RecommendationList.tsx`)
```tsx
'use client';

import { useEffect, useState } from 'react';
import RecommendationCard from './RecommendationCard';

interface Recommendation {
  id: string;
  icon: string;
  title: string;
  description: string;
  scientificExplanation: string;
  score: number;
  evidence?: {
    id: string;
    claim: string;
    source: string;
    evidenceLevel: string;
  };
}

interface RecommendationListProps {
  userId: string;
}

export default function RecommendationList({ userId }: RecommendationListProps) {
  const [recommendations, setRecommendations] = useState<Recommendation[]>([]);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchRecommendations = async () => {
      try {
        setLoading(true);
        const response = await fetch(`http://localhost:8000/api/recommendations?user_id=${userId}`);
        if (!response.ok) {
          throw new Error('Échec de la récupération des recommandations');
        }
        const data = await response.json();
        setRecommendations(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Une erreur est survenue');
      } finally {
        setLoading(false);
      }
    };

    fetchRecommendations();
  }, [userId]);

  const handleValidate = async (recommendationId: string) => {
    try {
      const response = await fetch('http://localhost:8000/api/recommendations/validate', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ user_id: userId, recommendation_id: recommendationId }),
      });

      if (!response.ok) {
        throw new Error('Échec de la validation de la recommandation');
      }

      // Rafraîchir les recommandations après validation
      const updatedResponse = await fetch(`http://localhost:8000/api/recommendations?user_id=${userId}`);
      if (updatedResponse.ok) {
        const updatedData = await updatedResponse.json();
        setRecommendations(updatedData);
      }
    } catch (err) {
      console.error('Erreur lors de la validation:', err);
    }
  };

  if (loading) {
    return (
      <div className="flex items-center justify-center h-64">
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

  if (recommendations.length === 0) {
    return (
      <div className="bg-gray-100 border border-gray-300 text-gray-700 px-4 py-3 rounded">
        Aucune recommandation disponible pour le moment.
      </div>
    );
  }

  return (
    <div className="space-y-4">
      {recommendations.map((recommendation) => (
        <RecommendationCard
          key={recommendation.id}
          recommendation={recommendation}
          onValidate={handleValidate}
        />
      ))}
    </div>
  );
}
```

---

### Mise à jour des Fonctions API (`api.ts`)
```tsx
// Fonction pour récupérer les recommandations avec explications scientifiques
export async function fetchRecommendationsWithExplanations(userId: string) {
  const response = await fetch(`http://localhost:8000/api/recommendations?user_id=${userId}&include_explanations=true`);
  if (!response.ok) {
    throw new Error('Échec de la récupération des recommandations');
  }
  return response.json();
}

// Fonction pour valider une recommandation
export async function validateRecommendation(userId: string, recommendationId: string) {
  const response = await fetch('http://localhost:8000/api/recommendations/validate', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ user_id: userId, recommendation_id: recommendationId }),
  });
  if (!response.ok) {
    throw new Error('Échec de la validation de la recommandation');
  }
  return response.json();
}
```

---

## Tests

### Tests Unitaires
1. **Test du composant `RecommendationCard`** :
   - Vérifier que la carte affiche correctement les informations (icône, titre, description, score).
   - Vérifier que l'explication scientifique s'affiche au clic sur "Voir les détails".
   - Vérifier que le bouton "Valider" déclenche la fonction `onValidate`.

2. **Test du composant `ScientificExplanation`** :
   - Vérifier que l'explication scientifique et les informations sur la preuve s'affichent correctement.

3. **Test du composant `RecommendationList`** :
   - Vérifier que la liste affiche toutes les recommandations.
   - Vérifier que la validation d'une recommandation fonctionne correctement.

---

## Configuration Requise

### Backend
Assurez-vous que les endpoints suivants sont disponibles :
- `GET /api/recommendations?user_id={user_id}&include_explanations=true` : Retourne les recommandations avec leurs explications scientifiques.
- `POST /api/recommendations/validate` : Valide une recommandation et l'ajoute à l'historique de l'utilisateur.

Exemple de réponse pour `/api/recommendations` :
```json
[
  {
    "id": "rec-001",
    "icon": "🚲",
    "title": "Balade en forêt avec vos enfants",
    "description": "Qualité de l'air excellente et sentiers accessibles à 5 km",
    "scientificExplanation": "Source : Atmo France, AQI=42. Les balades en forêt améliorent la santé mentale et réduisent le stress. Une étude de l'Université de Stanford a montré que les balades en nature réduisent l'anxiété de 20%.",
    "score": 85,
    "evidence": {
      "id": "ev-001",
      "claim": "Les balades en forêt améliorent la santé mentale",
      "source": "Université de Stanford, 2020",
      "evidenceLevel": "HIGH"
    }
  }
]
```

---

## Dépendances
- **Story 8.1** : Le layout mobile-first doit être implémenté.
- **Story 8.2** : Le dashboard "Aujourd'hui" doit être implémenté.
- **Epic 2 (Recommendation Engine)** : Pour générer et récupérer les recommandations.
- **Epic 3 (Knowledge Engine)** : Pour récupérer les explications scientifiques et les preuves associées.

---

## Ressources
- [Documentation Next.js](https://nextjs.org/docs)
- [Documentation Tailwind CSS](https://tailwindcss.com/docs)
- [PRD Almanéa - FR-35, FR-36](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - UX-DR-3](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
