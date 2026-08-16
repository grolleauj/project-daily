# Epic 1 Context: Context Engine

<!-- Compiled from planning artifacts. Edit freely. Regenerate with compile-epic-context if planning docs change. -->

## Goal

Collecter, normaliser, mettre en cache, et enrichir les données des providers (Atmo France, RTE, Hub'Eau, OpenStreetMap) pour fournir un contexte unifié au Recommendation Engine. Cet épic est **critique** pour le PoC, car toutes les autres fonctionnalités (recommandations, notifications, UI) dépendent des données collectées et normalisées.

## Stories

- Story 1.1: Implémenter `ProviderAdapter` pour Atmo France
- Story 1.2: Implémenter `ProviderAdapter` pour RTE
- Story 1.3: Implémenter `ProviderAdapter` pour Hub'Eau
- Story 1.4: Implémenter `ProviderAdapter` pour OpenStreetMap
- Story 1.5: Implémenter le cache Redis
- Story 1.6: Construire `UnifiedContext`
- Story 1.7: Gérer les erreurs des Providers

## Requirements & Constraints

### Functional Requirements
- Collecte des données externes depuis les Providers via `ProviderAdapter` (FR-1).
- Normalisation des données brutes en format `Observation` (provider, observedAt, retrievedAt, expiresAt, quality, value) (FR-2).
- Mise en cache des données normalisées dans Redis (TTL: 5 min pour le temps réel, 1h pour les données statiques) (FR-3).
- Enrichissement du contexte avec des données dérivées (ex: indice de qualité de l'air global) (FR-4).
- Gestion des erreurs des Providers (fallback vers le cache, alertes pour les pannes critiques) (FR-5).

### Non-Functional Requirements
- Déterminisme : Les données collectées doivent être reproductibles et validées.
- Pas de vendor lock-in : Les adapters doivent être interchangeables sans impact sur le reste du système.
- Sécurité : Les secrets (API keys) ne doivent **jamais** être exposés côté frontend.

### Architecture Decisions
- **AD-2 (Provider-Agnostic Adapters)** : Chaque provider a un adapter dédié implémentant `ProviderAdapter` avec `fetch()`, `normalize()`, `validate()`.
- **AD-4 (Unified Context Contract)** : `UnifiedContext` inclut `observations` (list of `Observation`), `user_profile` (optionnel), `timestamp`.
- **AD-7 (Caching Strategy)** : Cache TTL: 5 minutes pour les données temps réel (ex: qualité de l'air), 1 heure pour les données statiques (ex: phases lunaires).

## Technical Decisions

- **Provider-Agnostic Design** : Chaque provider doit avoir un adapter dédié implémentant l'interface `ProviderAdapter` avec les méthodes `fetch()`, `normalize()`, et `validate()`.
- **Unified Context** : Les données normalisées doivent être agrégées dans un objet `UnifiedContext` standardisé pour une consommation cohérente par le Recommendation Engine.
- **Caching Strategy** : Utiliser Redis pour mettre en cache les données normalisées. En cas d'échec d'un provider, utiliser la dernière valeur valide en cache ou une valeur par défaut.
- **Error Handling** : Générer une alerte si >50% des providers critiques (Atmo, RTE, Hub'Eau) sont indisponibles.

## Cross-Story Dependencies

- **Story 1.1 à 1.4** : Doivent être implémentées avant **Story 1.6** (UnifiedContext dépend des adapters).
- **Story 1.5** (Cache Redis) : Doit être implémentée avant **Story 1.7** (Gestion des erreurs utilise le cache).
- **Story 1.6** (UnifiedContext) : Blocante pour **Épic 2 (Recommendation Engine)**.
