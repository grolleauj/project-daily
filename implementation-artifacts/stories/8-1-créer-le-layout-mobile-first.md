---
name: Créer le layout mobile-first
id: 8-1-créer-le-layout-mobile-first
epic: epic-8
story_type: frontend
priority: high
estimation: L
dependencies: []
status: backlog
created: 2026-08-20
updated: 2026-08-20
---

# Story 8.1: Créer le layout mobile-first

## Contexte
Cette story fait partie de **l'Épic 8 (Web UI)**. Son objectif est de créer une **interface responsive** avec une navigation optimisée pour le mobile, incluant **5 onglets en bas** et un **bouton central flottant** pour déclarer une action. Cela répond aux exigences **FR-34 (Dashboard principal)**, **FR-35 (Navigation Principale)**, et **UX-DR-1 (Design mobile-first)**.

---

## Exigences Fonctionnelles
- **FR-34**: Dashboard principal avec header (météo, qualité de l'air) et section "Aujourd'hui pour vous".
- **FR-35**: Navigation Principale (5 onglets + bouton central flottant pour déclarer une action).
- **UX-DR-1**: Design mobile-first avec 5 onglets en bas et bouton central flottant.

---

## Critères d'Acceptation
1. **Responsivité** :
   - [ ] L'interface est **responsive** (mobile, tablette, desktop).
   - [ ] Les styles s'adaptent correctement à toutes les tailles d'écran.

2. **Navigation** :
   - [ ] 5 onglets en bas : **Aujourd'hui**, **Explorer**, **+ Action**, **Impact**, **Profil**. 
   - [ ] Un **bouton central flottant** pour déclarer une action.

3. **Expérience utilisateur** :
   - [ ] L'interface est **touch-friendly** (boutons assez grands pour les doigts).
   - [ ] La navigation entre les onglets est fluide et intuitive.

---

## Implémentation Technique

### Structure du projet Next.js
```
frontend/
├── app/
│   ├── layout.tsx          # Layout principal avec navigation
│   ├── page.tsx            # Page d'accueil (redirige vers /aujourdhui)
│   ├── aujourdhui/
│   │   └── page.tsx        # Dashboard "Aujourd'hui"
│   ├── explorer/
│   │   └── page.tsx        # Onglet "Explorer"
│   ├── action/
│   │   └── page.tsx        # Page pour déclarer une action
│   ├── impact/
│   │   └── page.tsx        # Onglet "Impact"
│   └── profil/
│       └── page.tsx        # Onglet "Profil"
├── components/
│   ├── layout/
│   │   ├── BottomNavigation.tsx  # Barre de navigation en bas
│   │   ├── FloatingActionButton.tsx  # Bouton flottant central
│   │   └── Header.tsx       # Header avec contexte environnemental
│   └── ui/                 # Composants UI réutilisables
├── styles/
│   └── globals.css         # Styles globaux
└── package.json
```

### Fichiers à créer/modifier
| Fichier | Rôle | Statut |
|---------|------|--------|
| `frontend/app/layout.tsx` | Layout principal avec navigation | À créer |
| `frontend/components/layout/BottomNavigation.tsx` | Barre de navigation en bas | À créer |
| `frontend/components/layout/FloatingActionButton.tsx` | Bouton flottant central | À créer |
| `frontend/components/layout/Header.tsx` | Header avec contexte environnemental | À créer |
| `frontend/styles/globals.css` | Styles globaux | À créer |

---

### Implémentation du Layout (`layout.tsx`)
```tsx
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import './globals.css';
import BottomNavigation from '@/components/layout/BottomNavigation';
import FloatingActionButton from '@/components/layout/FloatingActionButton';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'Almanéa - Votre compagnon écologique',
  description: 'Découvrez des recommandations personnalisées pour agir positivement sur l\'environnement.',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="fr">
      <body className={inter.className}>
        <div className="min-h-screen flex flex-col">
          {/* Contenu principal */}
          <main className="flex-grow relative pb-20">
            {children}
          </main>
          
          {/* Bouton flottant central */}
          <FloatingActionButton />
          
          {/* Barre de navigation en bas */}
          <BottomNavigation />
        </div>
      </body>
    </html>
  );
}
```

---

### Implémentation de la Barre de Navigation (`BottomNavigation.tsx`)
```tsx
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';

const navItems = [
  { name: 'Aujourd\'hui', href: '/aujourdhui', icon: '📅' },
  { name: 'Explorer', href: '/explorer', icon: '🔍' },
  { name: 'Action', href: '/action', icon: '➕' },
  { name: 'Impact', href: '/impact', icon: '🌱' },
  { name: 'Profil', href: '/profil', icon: '👤' },
];

export default function BottomNavigation() {
  const pathname = usePathname();

  return (
    <nav className="fixed bottom-0 left-0 right-0 bg-white border-t border-gray-200 z-50">
      <div className="flex justify-around items-center h-16">
        {navItems.map((item) => (
          <Link
            key={item.href}
            href={item.href}
            className={`flex flex-col items-center justify-center h-full px-4 ${
              pathname === item.href ? 'text-green-600' : 'text-gray-500'
            }`}
          >
            <span className="text-xl">{item.icon}</span>
            <span className="text-xs mt-1">{item.name}</span>
          </Link>
        ))}
      </div>
    </nav>
  );
}
```

---

### Implémentation du Bouton Flottant (`FloatingActionButton.tsx`)
```tsx
'use client';

import Link from 'next/link';

export default function FloatingActionButton() {
  return (
    <Link
      href="/action"
      className="fixed bottom-20 left-1/2 transform -translate-x-1/2 w-16 h-16 bg-green-600 rounded-full flex items-center justify-center text-white text-2xl shadow-lg z-40 hover:bg-green-700 transition-colors"
    >
      ➕
    </Link>
  );
}
```

---

### Implémentation du Header (`Header.tsx`)
```tsx
'use client';

import { useEffect, useState } from 'react';

export default function Header() {
  const [time, setTime] = useState<string>('');
  const [location, setLocation] = useState<string>('Lyon 69003');
  const [weather, setWeather] = useState<{
    temperature: string;
    condition: string;
    precipitation: string;
  }>({
    temperature: '18°',
    condition: 'Ensoleillé',
    precipitation: 'Pluie faible 10%',
  });
  const [airQuality, setAirQuality] = useState<{
    aqi: number;
    label: string;
  }>({ aqi: 42, label: 'Air bon' });

  useEffect(() => {
    // Mettre à jour l'heure toutes les minutes
    const updateTime = () => {
      const now = new Date();
      setTime(now.toLocaleTimeString('fr-FR', { hour: '2-digit', minute: '2-digit' }));
    };
    updateTime();
    const interval = setInterval(updateTime, 60000);
    return () => clearInterval(interval);
  }, []);

  return (
    <header className="bg-gradient-to-b from-green-100 to-white p-4 shadow-sm">
      <div className="max-w-4xl mx-auto">
        {/* Heure et localisation */}
        <div className="flex justify-between items-center mb-2">
          <span className="text-2xl font-bold">{time}</span>
          <span className="text-lg text-gray-600">{location}</span>
        </div>

        {/* Météo et qualité de l'air */}
        <div className="flex justify-between items-center">
          <div className="flex items-center space-x-4">
            <div className="flex items-center space-x-2">
              <span className="text-2xl">🌤️</span>
              <div>
                <span className="font-semibold">{weather.temperature}</span>
                <span className="text-sm text-gray-600"> {weather.condition}</span>
              </div>
            </div>
            <div className="flex items-center space-x-2">
              <span className="text-2xl">💨</span>
              <div>
                <span className="font-semibold">{airQuality.aqi} AQI</span>
                <span className="text-sm text-gray-600"> {airQuality.label}</span>
              </div>
            </div>
          </div>
          <div className="text-sm text-gray-600">
            {weather.precipitation}
          </div>
        </div>
      </div>
    </header>
  );
}
```

---

### Styles Globaux (`globals.css`)
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Styles personnalisés */
body {
  margin: 0;
  padding: 0;
  font-family: 'Inter', sans-serif;
}

/* Assurer que le contenu ne passe pas sous la navigation */
.main-content {
  padding-bottom: 80px; /* Hauteur de la barre de navigation + marge */
}

/* Styles pour les onglets actifs */
.nav-item-active {
  color: #16a34a; /* Vert principal */
  border-top: 2px solid #16a34a;
}

/* Animation pour le bouton flottant */
@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
}

.floating-button {
  animation: pulse 2s infinite;
}
```

---

## Tests

### Tests d'Intégration
1. **Vérifier la responsivité** :
   - Ouvrir l'application sur mobile, tablette et desktop.
   - Vérifier que la navigation et le bouton flottant s'affichent correctement.

2. **Vérifier la navigation** :
   - Cliquer sur chaque onglet et vérifier que la page correspondante s'affiche.
   - Vérifier que le bouton flottant redirige vers `/action`.

3. **Vérifier le header** :
   - Vérifier que l'heure, la localisation, la météo et la qualité de l'air s'affichent correctement.

---

## Configuration Requise

### Dépendances
- **Next.js 14.1.0**
- **React 18+**
- **TypeScript**
- **Tailwind CSS** (pour les styles)

Installer avec :
```bash
npx create-next-app@latest frontend --typescript --tailwind --eslint
cd frontend
npm install
```

---

## Intégration avec le Backend

### Récupération du Contexte Environnemental
Pour afficher les données réelles (météo, qualité de l'air), intégrer les endpoints du backend :

```tsx
// Exemple dans Header.tsx
useEffect(() => {
  const fetchContextData = async () => {
    try {
      const response = await fetch('http://localhost:8000/api/context/unified');
      const data = await response.json();
      
      // Mettre à jour les états avec les données réelles
      setWeather({
        temperature: `${data.weather.temperature}°`,
        condition: data.weather.condition,
        precipitation: data.weather.precipitation,
      });
      setAirQuality({
        aqi: data.air_quality.aqi,
        label: data.air_quality.label,
      });
    } catch (error) {
      console.error('Erreur lors de la récupération du contexte:', error);
    }
  };
  
  fetchContextData();
}, []);
```

---

## Dépendances
- **Epic 1 (Context Engine)** : Pour récupérer les données de contexte (météo, qualité de l'air).
- **Epic 2 (Recommendation Engine)** : Pour afficher les recommandations dans le dashboard.

---

## Ressources
- [Documentation Next.js](https://nextjs.org/docs)
- [Documentation Tailwind CSS](https://tailwindcss.com/docs)
- [PRD Almanéa - FR-34, FR-35](file:///Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md)
- [Architecture Spine - UX-DR-1](file:///Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md)
