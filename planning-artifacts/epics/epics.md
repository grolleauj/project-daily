---
name: Almanéa
type: epics-and-stories
inputDocuments:
  - /Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md
  - /Users/julie/Projects/_bmad-output/planning-artifacts/architecture/architecture-Almanéa-2026-08-16/ARCHITECTURE-SPINE.md
  - daily-opportunities/documentation/Almanea1.png
  - daily-opportunities/documentation/Almanea2.png
stepsCompleted: [step-01, step-02, step-03, step-04]
status: final
created: 2026-08-16
updated: 2026-08-16
finalized: true
---

# Almanéa — Épics et Stories

## Overview
Ce document fournit la décomposition complète des épics et stories pour **Almanéa**, en transformant les exigences du PRD, de l'Architecture, et des maquettes UX en tâches implémentables.

---

## Requirements Inventory

### Functional Requirements (FRs)
FR-1: Collecte des données externes depuis les Providers (Atmo France, RTE, Hub'Eau, OSM) via ProviderAdapter.
FR-2: Normalisation des données brutes en format Observation (provider, observedAt, retrievedAt, expiresAt, quality, value).
FR-3: Mise en cache des données normalisées dans Redis (TTL: 5 min pour le temps réel, 1h pour les données statiques).
FR-4: Enrichissement du contexte avec des données dérivées (ex: indice de qualité de l'air global).
FR-5: Gestion des erreurs des Providers (fallback vers le cache, alertes pour les pannes critiques).
FR-6: Génération de candidats de recommandations via Rule Engine (règles déterministes).
FR-7: Calcul du score des recommandations (formule: 25% ContextFit + 20% TimeFit + 15% UserPreference + 15% Accessibility + 10% EnvironmentalOpportunity + 10% WellbeingOpportunity + 5% Novelty).
FR-8: Classement des recommandations par score décroissant.
FR-9: Filtrage des recommandations invalides ou contradictoires (priorité: sécurité > restrictions légales > contraintes techniques > préférences utilisateur).
FR-10: Génération de recommandations déterministes (sans LLM pour le core).
FR-11: Gestion des preuves scientifiques (Evidence: id, claim, source, evidenceLevel, reviewStatus, topics).
FR-12: Association des preuves scientifiques aux recommandations.
FR-13: Mise à jour des preuves scientifiques avec de nouvelles sources.
FR-14: Génération d'explications textuelles pour une recommandation en utilisant un LLM.
FR-15: Abstraction LLM-agnostique via l'interface LLMProvider.
FR-16: Fallback déterministe si le LLM est indisponible.
FR-17: Création et mise à jour du profil utilisateur (id, email, timezone, home_location, work_location, preferences).
FR-18: Gestion des préférences utilisateur (activités, transport, durée max).
FR-19: Historique des recommandations (recommendation_id, type, score, date, feedback).
FR-20: Feedback sur les recommandations (USEFUL, NOT_RELEVANT, DONE, DISMISSED).
FR-21: Système de points et suivi des progrès (ex: +120 points pour une balade à vélo).
FR-22: Récompenses personnalisables par les collectivités (accès à la bibliothèque, places pour des événements).
FR-23: Échange de points contre des récompenses.
FR-24: Journal Écologique (action, date, durée, points gagnés, catégorie).
FR-25: Mécanismes anti-triche et seuils pour les badges.
FR-26: Débloquer des badges (ex: "Réparateur" après 5 actions de réparation).
FR-27: Suivi des progrès (graphiques, comparaison à la moyenne de la collectivité).
FR-28: Personnalisation des règles de points par les collectivités.
FR-29: Gestion des récompenses par les collectivités (ajout/suppression, modification des points requis).
FR-30: Notifications pour les récompenses (nouvelle récompense disponible, points suffisants).
FR-31: Notifications pour les opportunités temporaires (ex: "Balade à vélo cet après-midi").
FR-32: Alertes pour les conditions critiques (ex: "Qualité de l'air mauvaise") avec SLA <5 min.
FR-33: Gestion des préférences de notification (types d'alertes, fréquence, canaux).
FR-34: Dashboard principal (Écran "Aujourd’hui") avec header (météo, qualité de l'air) et section "Aujourd’hui pour vous".
FR-35: Navigation Principale (5 onglets + bouton central flottant pour déclarer une action).
FR-36: Affichage des cartes de recommandations (icône, titre, description, explication scientifique).

### Non-Functional Requirements (NFRs)
NFR-1: Déterminisme — Les recommandations et scores doivent être reproductibles (sans LLM pour le core).
NFR-2: Pas de vendor lock-in — Abstraction pour les providers (APIs) et LLMs.
NFR-3: SLA des alertes — <5 min pour les alertes critiques, <15 min pour les non-critiques.
NFR-4: Approche positive — Aucune culpabilisation de l'utilisateur (UX Tone).

### Additional Requirements (Architecture)
- AR-1: Modular Monolith — Tous les engines (Context, Recommendation, Knowledge, Generation) sont des modules dans une seule unité déployable.
- AR-2: Stack technique — Python 3.11.8, FastAPI 0.109.0, Next.js 14.1.0, PostgreSQL 15.7, Redis 7.2.4, RabbitMQ 3.12.0, Celery 5.3.4.
- AR-3: Provider Adapters — Chaque provider a un adapter dédié implémentant ProviderAdapter (fetch(), normalize(), validate()).
- AR-4: Unified Context — Contrat standardisé entre Context Engine et Recommendation Engine (observations, user_profile, timestamp).
- AR-5: LLMProvider — Interface pour abstraire les LLMs (generate(prompt: str, context: dict) -> str).

### UX Design Requirements
UX-DR-1: Design mobile-first — 5 onglets en bas (Aujourd’hui, Explorer, + Action, Impact, Profil) + bouton central flottant pour les actions.
UX-DR-2: Affichage du contexte environnemental — Header avec météo (température, précipitations), qualité de l'air (AQI), et variations.
UX-DR-3: Cartes de recommandations — Icône, titre, description, explication scientifique, bouton "Valider" pour ajouter au journal.

### Requirements Coverage Map
| Exigence | Épic | Story | Statut |
|----------|------|-------|--------|
| FR-1     | E-1  | US-1.1, US-1.2, US-1.3, US-1.4 | ⬜ |
| FR-2     | E-1  | US-1.1, US-1.2, US-1.3, US-1.4 | ⬜ |
| FR-3     | E-1  | US-1.5 | ⬜ |
| FR-4     | E-1  | US-1.6 | ⬜ |
| FR-5     | E-1  | US-1.7 | ⬜ |
| FR-6     | E-2  | US-2.1 | ⬜ |
| FR-7     | E-2  | US-2.2 | ⬜ |
| FR-8     | E-2  | US-2.3 | ⬜ |
| FR-9     | E-2  | US-2.4 | ⬜ |
| FR-10    | E-2  | US-2.5 | ⬜ |
| FR-11    | E-3  | US-3.1 | ⬜ |
| FR-12    | E-3  | US-3.2 | ⬜ |
| FR-13    | E-3  | US-3.3 | ⬜ |
| FR-14    | E-4  | US-4.2 | ⬜ |
| FR-15    | E-4  | US-4.1 | ⬜ |
| FR-16    | E-4  | US-4.3 | ⬜ |
| FR-17    | E-5  | US-5.1 | ⬜ |
| FR-18    | E-5  | US-5.2 | ⬜ |
| FR-19    | E-5  | US-5.3 | ⬜ |
| FR-20    | E-5  | US-5.4 | ⬜ |
| FR-21    | E-6  | US-6.1 | ⬜ |
| FR-22    | E-6  | US-6.5 | ⬜ |
| FR-23    | E-6  | US-6.6 | ⬜ |
| FR-24    | E-6  | US-6.3 | ⬜ |
| FR-25    | E-6  | US-6.4, US-6.7 | ⬜ |
| FR-26    | E-6  | US-6.4 | ⬜ |
| FR-27    | E-6  | US-6.2 | ⬜ |
| FR-28    | E-6  | US-6.8 | ⬜ |
| FR-29    | E-6  | US-6.9 | ⬜ |
| FR-30    | E-6  | US-6.10 | ⬜ |
| FR-31    | E-7  | US-7.1 | ⬜ |
| FR-32    | E-7  | US-7.2 | ⬜ |
| FR-33    | E-7  | US-7.3 | ⬜ |
| FR-34    | E-8  | US-8.1, US-8.2 | ⬜ |
| FR-35    | E-8  | US-8.3, US-8.4 | ⬜ |
| FR-36    | E-8  | US-8.5, US-8.6 | ⬜ |
| NFR-1    | Tous | Tous | ⬜ |
| NFR-2    | E-1, E-4 | US-1.1 à US-1.4, US-4.1 | ⬜ |
| NFR-3    | E-7  | US-7.2 | ⬜ |
| NFR-4    | E-8  | US-8.1 à US-8.6 | ⬜ |

## Epic List

### E-1: Context Engine
**Goal**: Collecter, normaliser, mettre en cache, et enrichir les données des providers (Atmo France, RTE, Hub'Eau, OSM) pour fournir un contexte unifié au Recommendation Engine.
**Priority**: ⭐⭐⭐⭐⭐ (PoC)
**Dependencies**: None

### E-2: Recommendation Engine
**Goal**: Générer, scorer, classer, et filtrer les recommandations à partir du Unified Context et du User Profile.
**Priority**: ⭐⭐⭐⭐⭐ (PoC)
**Dependencies**: E-1 (UnifiedContext requis)

### E-3: Knowledge Engine
**Goal**: Gérer les preuves scientifiques pour expliquer les recommandations.
**Priority**: ⭐⭐⭐ (Early Access)
**Dependencies**: None

### E-4: Generation Engine
**Goal**: Générer des explications textuelles pour les recommandations via LLM (avec fallback déterministe).
**Priority**: ⭐⭐⭐ (Early Access)
**Dependencies**: E-2 (Recommendation Engine), E-3 (Knowledge Engine)

### E-5: User Management
**Goal**: Gérer les profils, préférences, historique, et feedback des utilisateurs.
**Priority**: ⭐⭐⭐⭐ (PoC)
**Dependencies**: None

### E-6: Gamification
**Goal**: Implémenter le système de points, badges, défis, journal écologique, et récompenses localisées.
**Priority**: ⭐⭐⭐ (Early Access)
**Dependencies**: E-5 (User Management)

### E-7: Notifications
**Goal**: Envoyer des notifications et alertes en temps réel pour les opportunités et conditions critiques.
**Priority**: ⭐⭐⭐ (PoC)
**Dependencies**: E-1 (Context Engine), AR-2 (RabbitMQ/Celery)

### E-8: Web UI
**Goal**: Développer l'interface utilisateur responsive (Next.js) avec 5 onglets et bouton central.
**Priority**: ⭐⭐⭐⭐⭐ (PoC)
**Dependencies**: E-1 (UnifiedContext), E-2 (Recommendations), E-5 (User Profile)

---

## Epic Details

## Epic 1: Context Engine
**Goal**: Collecter, normaliser, mettre en cache, et enrichir les données des providers pour fournir un contexte unifié.

### Story 1.1: Implémenter `ProviderAdapter` pour Atmo France
As a **Context Engine**,
I want **un adapter pour récupérer et normaliser les données de qualité de l'air depuis Atmo France**,
So that **le Recommendation Engine peut utiliser ces données pour générer des recommandations basées sur la qualité de l'air**.

**Acceptance Criteria:**
- **Given** une requête HTTP valide à l'API Atmo France,
- **When** `fetch()` est appelé,
- **Then** les données brutes sont retournées (HTTP 200).
- **And** `normalize()` convertit les données en `Observation` avec les champs : `provider`, `observedAt`, `retrievedAt`, `expiresAt`, `quality`, `value`.
- **And** `validate()` rejette les valeurs hors plage (AQI 0-500).
- **And** les secrets (API keys) ne sont **jamais exposés** côté frontend.

**Related Requirements**: FR-1, FR-2, AR-3
**Estimation**: M
**Priority**: High

---

### Story 1.2: Implémenter `ProviderAdapter` pour RTE
As a **Context Engine**,
I want **un adapter pour récupérer et normaliser les données du mix énergétique depuis RTE**,
So that **le Recommendation Engine peut utiliser ces données pour générer des recommandations basées sur l'énergie**.

**Acceptance Criteria:**
- **Given** une requête HTTP valide à l'API RTE,
- **When** `fetch()` est appelé,
- **Then** les données brutes sont retournées (HTTP 200).
- **And** `normalize()` convertit les données en `Observation` avec `provider="RTE"`.
- **And** `validate()` rejette les valeurs hors plage (0 ≤ production ≤ 100 GW par type de filière).

**Related Requirements**: FR-1, FR-2, AR-3
**Estimation**: M
**Priority**: High

---

### Story 1.3: Implémenter `ProviderAdapter` pour Hub'Eau
As a **Context Engine**,
I want **un adapter pour récupérer et normaliser les données des niveaux d'eau depuis Hub'Eau**,
So that **le Recommendation Engine peut utiliser ces données pour générer des recommandations liées à l'eau**.

**Acceptance Criteria:**
- **Given** une requête HTTP valide à l'API Hub'Eau,
- **When** `fetch()` est appelé,
- **Then** les données brutes sont retournées (HTTP 200).
- **And** `normalize()` convertit les données en `Observation` avec `provider="HubEau"`.
- **And** `validate()` rejette les valeurs hors plage (0 ≤ niveau ≤ 1000 cm).

**Related Requirements**: FR-1, FR-2, AR-3
**Estimation**: M
**Priority**: High

---

### Story 1.4: Implémenter `ProviderAdapter` pour OpenStreetMap
As a **Context Engine**,
I want **un adapter pour récupérer et normaliser les données géographiques depuis OpenStreetMap (Photon et Overpass API)**,
So that **le Recommendation Engine peut utiliser ces données pour générer des recommandations liées à la localisation**.

**Acceptance Criteria:**
- **Given** une requête HTTP valide à Photon (géocodage) ou Overpass API (données géographiques),
- **When** `fetch()` est appelé,
- **Then** les données brutes sont retournées (HTTP 200).
- **And** `normalize()` convertit les données en `Observation` avec `provider="OSM"`.
- **And** `validate()` rejette les valeurs invalides (ex: coordonnées hors plage).

**Related Requirements**: FR-1, FR-2, AR-3
**Estimation**: M
**Priority**: High

---

### Story 1.5: Implémenter le cache Redis
As a **Context Engine**,
I want **mettre en cache les données normalisées dans Redis**,
So that **les appels répétés aux APIs des providers sont évités et les performances sont améliorées**.

**Acceptance Criteria:**
- **Given** une `Observation` normalisée,
- **When** elle est stockée dans Redis,
- **Then** elle est servie en **<500ms** pour les requêtes complexes.
- **And** les données **simples** (ex: météo seule) sont servies en **<100ms**.
- **And** les données expirées sont **rafraîchies automatiquement**.
- **And** TTL = **5 minutes** pour les données temps réel (AQI, météo).
- **And** TTL = **1 heure** pour les données statiques (phases lunaires).
- **And** en cas d'échec du provider, le système **utilise la dernière valeur valide en cache**. 

**Related Requirements**: FR-3, AR-2
**Estimation**: S
**Priority**: High

---

### Story 1.6: Construire `UnifiedContext`
As a **Context Engine**,
I want **agréger les `Observation` en un `UnifiedContext` standardisé**,
So that **le Recommendation Engine peut consommer un format cohérent pour générer des recommandations**.

**Acceptance Criteria:**
- **Given** une liste d'`Observation` valides,
- **When** `UnifiedContext` est construit,
- **Then** il contient : `observations: list[Observation]`, `user_profile` (optionnel), `timestamp`.
- **And** les données enrichies sont **cohérentes** (ex: pas de conflit entre météo et qualité de l'air).

**Related Requirements**: FR-4, AR-4
**Estimation**: M
**Priority**: High

---

### Story 1.7: Gérer les erreurs des Providers
As a **Context Engine**,
I want **gérer les erreurs des providers (API indisponible, rate limit, timeout)**,
So that **le système reste résilient et dégrade gracieusement en cas de panne**.

**Acceptance Criteria:**
- **Given** une erreur de provider (ex: API indisponible),
- **When** le système détecte l'erreur,
- **Then** il **utilise des données en cache** si disponibles (TTL étendu temporairement).
- **And** une **alerte** est générée pour l'administrateur si un provider critique est indisponible.
- **And** en cas de **pannes partielles**, le système **dégrade gracieusement** :
  - Les recommandations dépendant du provider défaillant sont **masquées ou remplacées** par des alternatives.
  - Les données des providers disponibles sont **toujours utilisées**. 
- **And** une alerte est déclenchée si **>50% des providers critiques** (Atmo, RTE, Hub'Eau) sont indisponibles.

**Related Requirements**: FR-5, AR-2
**Estimation**: S
**Priority**: High

---

## Epic 2: Recommendation Engine
**Goal**: Générer, scorer, classer, et filtrer les recommandations à partir du Unified Context et du User Profile.

### Story 2.1: Implémenter `Rule Engine`
As a **Recommendation Engine**,
I want **un moteur de règles pour générer des candidats de recommandations**,
So that **les recommandations sont basées sur des règles déterministes et reproductibles**.

**Acceptance Criteria:**
- **Given** un `UnifiedContext` valide et un `UserProfile`,
- **When** `Rule Engine` est exécuté,
- **Then** il génère des **candidats de recommandations** valides (ex: pas de recommandation de vélo si la qualité de l'air est mauvaise).
- **And** les candidats sont **testables indépendamment du LLM**.
- **And** les règles sont **déterministes** (ex: `IF air_quality = GOOD AND rain_probability < 30% THEN candidate = BIKE_ACTIVITY`).

**Example Rule:**
```
IF
  air_quality = GOOD
  AND rain_probability < 30%
  AND wind_speed < 20 km/h
  AND user.likes_cycling = true
  AND available_time >= estimated_duration
THEN
  candidate = BIKE_ACTIVITY
```

**Related Requirements**: FR-6, FR-10, AR-5
**Estimation**: L
**Priority**: High
**Dependencies**: E-1 (UnifiedContext)

---

### Story 2.2: Implémenter le scoring
As a **Recommendation Engine**,
I want **calculer un score pour chaque candidat de recommandation**,
So that **les recommandations peuvent être classées par pertinence**.

**Acceptance Criteria:**
- **Given** un candidat de recommandation et son contexte,
- **When** le score est calculé,
- **Then** la formule utilisée est :
  `Score = 25% ContextFit + 20% TimeFit + 15% UserPreference + 15% Accessibility + 10% EnvironmentalOpportunity + 10% WellbeingOpportunity + 5% Novelty`.
- **And** le score est **déterministe** (mêmes entrées → même score).
- **And** le score est **normalisé entre 0 et 1**. 
- **And** les **valeurs nulles** pour un composant (ex: `ContextFit=0`) sont **remplacées par une valeur par défaut (0.5)**.
- **And** les **poids** (25%, 20%, etc.) sont **validés** pour s'assurer qu'ils totalisent 100%.
- **And** une **erreur** est générée si un composant retourne une valeur **hors plage** (ex: `ContextFit > 1`).

**Related Requirements**: FR-7, AR-5
**Estimation**: M
**Priority**: High
**Dependencies**: US-2.1

---

### Story 2.3: Classer les recommandations
As a **Recommendation Engine**,
I want **classer les recommandations par score décroissant**,
So that **les recommandations les plus pertinentes sont affichées en premier**.

**Acceptance Criteria:**
- **Given** une liste de recommandations avec des scores,
- **When** le classement est appliqué,
- **Then** les recommandations sont **triées par pertinence** (score décroissant).
- **And** les recommandations **diverses** (ex: pas que du vélo) sont **priorisées**. 

**Related Requirements**: FR-8
**Estimation**: S
**Priority**: High
**Dependencies**: US-2.2

---

### Story 2.4: Filtrer les recommandations invalides
As a **Recommendation Engine**,
I want **supprimer ou ajuster les recommandations invalides ou contradictoires**,
So that **les utilisateurs ne reçoivent que des recommandations sûres et pertinentes**.

**Acceptance Criteria:**
- **Given** une liste de recommandations,
- **When** le filtrage est appliqué,
- **Then** une recommandation est **supprimée** si :
  - Conditions météo dangereuses.
  - Qualité de l'air incompatible.
  - Restriction locale incompatible (ex: restriction d'eau).
  - Données critiques indisponibles.
  - Temps insuffisant.
- **And** en cas de **contradictions**, la **règle critique prime** (priorité : 1) Sécurité, 2) Restrictions légales, 3) Contraintes techniques, 4) Préférences utilisateur).
- **And** les recommandations filtrées sont **remplacées par un message alternatif** (ex: "Arrosage interdit aujourd'hui. Essayez une activité sans eau.").

**Related Requirements**: FR-9
**Estimation**: M
**Priority**: High
**Dependencies**: US-2.1

---

### Story 2.5: Générer des recommandations déterministes
As a **Recommendation Engine**,
I want **valider que les recommandations sont générées de manière déterministe (sans LLM)**,
So that **les tests unitaires peuvent être exécutés sans dépendre d'un LLM**.

**Acceptance Criteria:**
- **Given** les mêmes entrées (UnifiedContext, UserProfile),
- **When** le Recommendation Engine est exécuté,
- **Then** les mêmes recommandations sont générées.
- **And** les tests unitaires du **Recommendation Engine** fonctionnent **sans LLM**. 

**Related Requirements**: FR-10
**Estimation**: S
**Priority**: High
**Dependencies**: US-2.1, US-2.2, US-2.4

---

## Epic 3: Knowledge Engine
**Goal**: Gérer les preuves scientifiques pour expliquer les recommandations.

### Story 3.1: Stocker les preuves scientifiques
As a **Knowledge Engine**,
I want **stocker et récupérer des preuves scientifiques (Evidence)**,
So that **les recommandations peuvent être expliquées avec des sources fiables**.

**Acceptance Criteria:**
- **Given** une preuve scientifique valide,
- **When** elle est stockée,
- **Then** elle contient : `id`, `claim`, `source` (titre, éditeur, URL), `evidenceLevel`, `reviewStatus`, `topics`.
- **And** les preuves sont **classées par niveau de confiance** (ex: `HIGH`, `MEDIUM`, `LOW`).

**Related Requirements**: FR-11
**Estimation**: M
**Priority**: Medium

---

### Story 3.2: Associer les preuves aux recommandations
As a **Knowledge Engine**,
I want **associer des preuves scientifiques à une recommandation**,
So that **les utilisateurs comprennent pourquoi une recommandation est pertinente**.

**Acceptance Criteria:**
- **Given** une recommandation et une liste de preuves,
- **When** l'association est faite,
- **Then** chaque recommandation a **au moins une Evidence** associée.
- **And** les preuves sont **pertinentes** (ex: une preuve sur la qualité de l'air pour une recommandation de balade).

**Related Requirements**: FR-12
**Estimation**: S
**Priority**: Medium

---

### Story 3.3: Mettre à jour les preuves scientifiques
As a **Knowledge Engine**,
I want **mettre à jour les preuves scientifiques avec de nouvelles sources**,
So that **les recommandations restent basées sur des informations à jour**.

**Acceptance Criteria:**
- **Given** de nouvelles preuves scientifiques,
- **When** elles sont ajoutées,
- **Then** les preuves sont **rafraîchies périodiquement** (ex: toutes les semaines).
- **And** les nouvelles preuves sont **validées** avant d'être ajoutées.

**Related Requirements**: FR-13
**Estimation**: S
**Priority**: Low

---

## Epic 4: Generation Engine
**Goal**: Générer des explications textuelles pour les recommandations via LLM (avec fallback déterministe).

### Story 4.1: Implémenter `LLMProvider`
As a **Generation Engine**,
I want **une interface `LLMProvider` pour abstraire les interactions avec les LLMs**,
So that **le système peut utiliser n'importe quel LLM sans dépendre d'un fournisseur spécifique**.

**Acceptance Criteria:**
- **Given** un LLM compatible,
- **When** `LLMProvider.generate(prompt: str, context: dict)` est appelé,
- **Then** il retourne une **explication textuelle**.
- **And** le code ne dépend **pas d'un fournisseur LLM spécifique**.
- **And** le **routing** entre différents LLMs est possible (ex: LLM local pour les tests, LLM cloud pour la production).

**Interface `LLMProvider`:**
- **Méthodes**: `generate(prompt: str, context: dict) -> str`.
- **Gestion des erreurs**: Retourne un message standardisé (ex: `LLMError: "Le service LLM est temporairement indisponible"`).

**Related Requirements**: FR-15, AR-5
**Estimation**: M
**Priority**: Medium
**Dependencies**: None

---

### Story 4.2: Générer des explications textuelles
As a **Generation Engine**,
I want **générer des explications textuelles pour une recommandation en utilisant un LLM**,
So that **les utilisateurs comprennent clairement pourquoi une recommandation est pertinente**.

**Acceptance Criteria:**
- **Given** une recommandation et son contexte,
- **When** une explication est générée,
- **Then** l'explication est **claire et utile** pour l'utilisateur.
- **And** l'explication contient **au moins une raison** (ex: "Qualité de l'air excellente").

**Related Requirements**: FR-14
**Estimation**: M
**Priority**: Medium
**Dependencies**: US-4.1

---

### Story 4.3: Implémenter le fallback déterministe
As a **Generation Engine**,
I want **fournir une explication déterministe minimale si le LLM est indisponible**,
So that **les utilisateurs reçoivent toujours une explication, même sans LLM**.

**Acceptance Criteria:**
- **Given** un LLM indisponible,
- **When** une explication est demandée,
- **Then** le système retourne un **fallback déterministe**.
- **And** le fallback inclut :
  - **Contexte** (ex: "Qualité de l'air excellente").
  - **Preuve scientifique** (si disponible, ex: "Source : Atmo France, AQI=42").
  - **Action recommandée** (ex: "Balade de 30 minutes").
  - **Bénéfice** (ex: "Bonne pour la santé").
- **And** les fallbacks sont **testés automatiquement** pour s'assurer qu'ils ne contiennent pas d'erreurs factuelles.

**Example Fallback:**
```
"Bonne qualité de l'air + météo favorable : une balade de 30 minutes est une bonne option aujourd'hui."
```

**Related Requirements**: FR-16
**Estimation**: S
**Priority**: Medium
**Dependencies**: US-4.1

---

## Epic 5: User Management
**Goal**: Gérer les profils, préférences, historique, et feedback des utilisateurs.

### Story 5.1: Créer et mettre à jour le profil utilisateur
As a **User**,
I want **créer et mettre à jour mon profil**,
So that **mes préférences et informations personnelles sont enregistrées pour des recommandations personnalisées**.

**Acceptance Criteria:**
- **Given** un utilisateur non authentifié ou authentifié,
- **When** il crée ou met à jour son profil,
- **Then** le profil contient : `id`, `email`, `timezone`, `home_location`, `work_location`, `preferences`.
- **And** les données sont **validées** (ex: email valide, localisation existante).

**Related Requirements**: FR-17
**Estimation**: M
**Priority**: High

---

### Story 5.2: Gérer les préférences utilisateur
As a **User**,
I want **configurer mes préférences (activités, transport, durée max)**,
So that **les recommandations sont adaptées à mes centres d'intérêt et contraintes**.

**Acceptance Criteria:**
- **Given** un utilisateur authentifié,
- **When** il configure ses préférences,
- **Then** les préférences sont **prises en compte** dans les recommandations.
- **And** les préférences sont **stockées et récupérables**. 

**Related Requirements**: FR-18
**Estimation**: S
**Priority**: High

---

### Story 5.3: Consulter l'historique des recommandations
As a **User**,
I want **consulter mon historique de recommandations**,
So that **je peux suivre mes actions passées et mes progrès**.

**Acceptance Criteria:**
- **Given** un utilisateur authentifié,
- **When** il consulte son historique,
- **Then** l'historique contient : `recommendation_id`, `type`, `score`, `date`, `feedback`.
- **And** l'historique est **filtrable** (ex: par date, par type).

**Related Requirements**: FR-19
**Estimation**: M
**Priority**: Medium

---

### Story 5.4: Donner un feedback sur les recommandations
As a **User**,
I want **donner un feedback sur une recommandation (`USEFUL`, `NOT_RELEVANT`, `DONE`, `DISMISSED`)**,
So that **le système peut améliorer les recommandations futures**.

**Acceptance Criteria:**
- **Given** une recommandation affichée,
- **When** l'utilisateur donne un feedback,
- **Then** le feedback est **enregistré** et utilisé pour améliorer les recommandations futures.
- **And** le feedback est **anonyme** (pas de données personnelles stockées).

**Related Requirements**: FR-20
**Estimation**: S
**Priority**: Medium

---

## Epic 6: Gamification
**Goal**: Implémenter le système de points, badges, défis, journal écologique, et récompenses localisées.

### Story 6.1: Système de points
As a **User**,
I want **gagner des points en suivant les recommandations**,
So that **je peux mesurer mon impact positif et débloquer des récompenses**.

**Acceptance Criteria:**
- **Given** une recommandation réalisée,
- **When** l'utilisateur valide l'action,
- **Then** il gagne des points selon les règles :
  - **+120 points** pour une balade à vélo.
  - **+30 points** pour une marche.
  - **+20 points** pour une balade en forêt.
- **And** les points sont **affichés en temps réel** dans le profil utilisateur.
- **And** les points sont **cumulables et non périssables** (sauf configuration spécifique par la collectivité).

**Related Requirements**: FR-21
**Estimation**: M
**Priority**: Medium
**Dependencies**: E-5 (User Management)

---

### Story 6.2: Suivi des progrès
As a **User**,
I want **voir mes progrès sous forme de graphiques**,
So that **je peux suivre mon évolution et me comparer à la moyenne de ma collectivité**.

**Acceptance Criteria:**
- **Given** un utilisateur avec un historique d'actions,
- **When** il consulte ses progrès,
- **Then** les progrès sont **affichés sous forme de graphiques** (ex: évolution des points sur 30 jours).
- **And** les progrès sont **comparables à la moyenne anonyme** de la collectivité (ex: "Vous êtes au-dessus de la moyenne cette semaine").
- **And** les progrès peuvent être **filtrés** (ex: par semaine, par mois, par catégorie d'action).
- **And** **pas de classement culpabilisant** (aucune comparaison directe entre utilisateurs).

**Related Requirements**: FR-27
**Estimation**: M
**Priority**: Medium
**Dependencies**: US-6.1

---

### Story 6.3: Journal Écologique
As a **User**,
I want **enregistrer mes actions dans un journal écologique**,
So that **je peux suivre mes activités et leurs impacts**.

**Acceptance Criteria:**
- **Given** une action réalisée,
- **When** l'utilisateur l'enregistre,
- **Then** le journal contient : `action`, `date`, `durée`, `points gagnés`, `catégorie` (ex: nature, énergie, bien-être).
- **And** le journal est **exportable** (ex: CSV, PDF).
- **And** le journal est **filtrable** (ex: par catégorie, par date).

**Related Requirements**: FR-24
**Estimation**: M
**Priority**: Medium
**Dependencies**: US-6.1

---

### Story 6.4: Débloquer des badges
As a **User**,
I want **débloquer des badges en accumulant des points ou en réalisant des actions spécifiques**,
So that **je suis motivé à participer davantage**.

**Acceptance Criteria:**
- **Given** un utilisateur avec des actions enregistrées,
- **When** il atteint un seuil,
- **Then** il débloque un badge selon les règles :
  - **Réparateur** (🏆) : 5 actions de réparation.
  - **Explorateur urbain** (🌿) : 10 découvertes locales.
  - **Zéro gaspi niveau 1** (♻️) : 5 actions de réemploi.
  - **Éco-citoyen** (🌍) : 20 actions écoresponsables.
  - **Maître du vélo** (🚲) : 15 trajets à vélo.
  - **Jardinier expert** (🌱) : 10 actions de jardinage.
- **And** les badges sont **affichés dans le profil utilisateur**. 
- **And** les badges peuvent être **partagés** (ex: sur les réseaux sociaux ou via un lien public).

**Related Requirements**: FR-26
**Estimation**: S
**Priority**: Medium
**Dependencies**: US-6.1, US-6.3

---

### Story 6.5: Mécanismes anti-triche
As a **System**,
I want **détecter et prévenir les abus du système de points**,
So that **l'intégrité de la gamification est garantie**.

**Acceptance Criteria:**
- **Given** une action déclarée par un utilisateur,
- **When** le système vérifie l'action,
- **Then** les mécanismes suivants sont appliqués :
  - **Validation manuelle** : Les récompenses à haute valeur (ex: >500 points) nécessitent une **validation manuelle** par un administrateur.
  - **Limites de fréquence** : Un utilisateur ne peut pas déclarer la même action plus d'une fois par jour (ex: "J'ai réparé" → 1x/jour max).
  - **Vérification contexte** : Les actions déclarées sont **validées par rapport au contexte** (ex: vérifier que l'utilisateur était bien dans la zone géolocalisée pour une balade).
  - **Alerte pour comportements suspects** : Détection des schémas anormaux (ex: 20 actions déclarées en 1 heure) et **blocage temporaire** du compte.
  - **Journal d'audit** : Toutes les actions déclarées sont **enregistrées avec horodatage et métadonnées** (IP, user agent) pour analyse.

**Related Requirements**: FR-25
**Estimation**: S
**Priority**: Medium
**Dependencies**: US-6.1

---

### Story 6.6: Récompenses personnalisables par les collectivités
As an **Admin (Collectivité)**,
I want **configurer des récompenses locales pour les utilisateurs de ma collectivité**,
So that **je peux motiver les citoyens à adopter des comportements écoresponsables**.

**Acceptance Criteria:**
- **Given** un administrateur de collectivité,
- **When** il configure les récompenses,
- **Then** les récompenses peuvent être :
  - **Accès à des services municipaux** (ex: 100 points = 1 heure d'accès à la bibliothèque).
  - **Places pour des événements** (ex: 50 points = 1 place pour un concert municipal).
  - **Cours ou ateliers offerts** (ex: 200 points = 1 cours de jardinage à la MJC).
  - **Autres avantages** (ex: réductions dans les commerces locaux partenaires).
- **And** les récompenses sont **affichées dans un catalogue** accessible aux utilisateurs.
- **And** les récompenses sont **modifiables** par l'administrateur (ajout/suppression, modification des points requis).

**Related Requirements**: FR-22, FR-29
**Estimation**: M
**Priority**: Medium
**Dependencies**: E-5 (User Management)

---

### Story 6.7: Échange de points contre des récompenses
As a **User**,
I want **échanger mes points contre des récompenses disponibles dans le catalogue de ma collectivité**,
So that **je peux bénéficier d'avantages concrets**.

**Acceptance Criteria:**
- **Given** un utilisateur avec suffisamment de points,
- **When** il échange ses points,
- **Then** il voit **les récompenses disponibles** et le **nombre de points requis** pour chacune.
- **And** l'échange est **validé automatiquement** si l'utilisateur a suffisamment de points.
- **And** une **confirmation** est envoyée à l'utilisateur (ex: email ou notification avec un code ou un bon à présenter).
- **And** les points sont **déduits du solde** de l'utilisateur après échange.

**Related Requirements**: FR-23
**Estimation**: M
**Priority**: Medium
**Dependencies**: US-6.6

---

### Story 6.8: Personnalisation des règles de points
As an **Admin (Collectivité)**,
I want **définir les règles d'attribution des points pour ma collectivité**,
So that **je peux adapter le système de gamification à mes objectifs locaux**.

**Acceptance Criteria:**
- **Given** un administrateur de collectivité,
- **When** il définit les règles,
- **Then** il peut **configurer le nombre de points** attribués pour chaque type d'action (ex: +20 points pour une balade en forêt, +5 points pour une action simple).
- **And** les règles peuvent être **modifiées à tout moment** et s'appliquent **immédiatement** aux nouveaux utilisateurs.
- **And** les règles sont **affichées clairement** pour les utilisateurs (ex: "Dans cette collectivité, une balade rapporte 20 points").

**Related Requirements**: FR-28
**Estimation**: S
**Priority**: Medium
**Dependencies**: E-5 (User Management)

---

### Story 6.9: Gestion des récompenses
As an **Admin (Collectivité)**,
I want **gérer le catalogue de récompenses pour ma collectivité**,
So that **je peux ajouter, supprimer ou modifier les récompenses disponibles**.

**Acceptance Criteria:**
- **Given** un administrateur de collectivité,
- **When** il gère le catalogue,
- **Then** il peut :
  - **Ajouter/supprimer des récompenses**. 
  - **Modifier le nombre de points requis** pour une récompense.
  - **Définir des limites** (ex: 1 récompense par utilisateur et par mois).
  - **Suivre les échanges** (ex: nombre de récompenses attribuées, points dépensés).
- **And** les récompenses sont **affichées dans un catalogue** avec :
  - Une **description** (ex: "1 heure d'accès à la bibliothèque").
  - Le **nombre de points requis**. 
  - Les **disponibilités** (ex: "5 places restantes").

**Related Requirements**: FR-29
**Estimation**: S
**Priority**: Medium
**Dependencies**: US-6.6

---

### Story 6.10: Notifications pour les récompenses
As a **User**,
I want **recevoir des notifications lorsqu'une nouvelle récompense est disponible ou lorsque j'ai suffisamment de points pour en bénéficier**,
So that **je ne manque pas les opportunités de récompenses**.

**Acceptance Criteria:**
- **Given** un utilisateur avec des préférences de notification activées,
- **When** une nouvelle récompense est disponible ou l'utilisateur a suffisamment de points,
- **Then** une notification est **envoyée via les canaux configurés** (email, appli native).
- **And** la notification contient :
  - Le **nom de la récompense**. 
  - Le **nombre de points requis**. 
  - Un **lien direct** pour échanger les points.

**Related Requirements**: FR-30
**Estimation**: S
**Priority**: Low
**Dependencies**: US-6.6, US-6.7

---

## Epic 7: Notifications
**Goal**: Envoyer des notifications et alertes en temps réel pour les opportunités et conditions critiques.

### Story 7.1: Notifications pour les opportunités temporaires
As a **User**,
I want **recevoir des notifications pour les opportunités temporaires (ex: "Balade à vélo cet après-midi : qualité de l'air excellente")**,
So that **je peux profiter des meilleures conditions en temps réel**.

**Acceptance Criteria:**
- **Given** une opportunité temporaire détectée,
- **When** une notification est envoyée,
- **Then** elle est **envoyée via email** (phase PoC) ou **appli native** (phase EA).
- **And** les notifications sont **personnalisées** (ex: basées sur les préférences utilisateur).

**Related Requirements**: FR-31
**Estimation**: M
**Priority**: Medium
**Dependencies**: E-1 (Context Engine)

---

### Story 7.2: Alertes pour les conditions critiques
As a **User**,
I want **recevoir des alertes pour les conditions critiques (ex: "Qualité de l'air mauvaise : évitez les activités extérieures intenses")**,
So that **je peux éviter les situations dangereuses**.

**Acceptance Criteria:**
- **Given** une condition critique détectée (ex: qualité de l'air mauvaise),
- **When** une alerte est envoyée,
- **Then** elle est **prioritaire** (affichée en haut du dashboard).
- **And** elle est **envoyée en temps réel** (SLA : **<5 minutes**).
- **And** l'architecture utilise :
  - **File de messages asynchrone** (RabbitMQ).
  - **Workers dédiés** (Celery) pour traiter les alertes en arrière-plan.
  - **Cache des alertes** (Redis) pour éviter les doublons.
  - **Priorisation** : Les alertes critiques sont **traitées avant** les alertes non critiques.

**Related Requirements**: FR-32, AR-2
**Estimation**: L
**Priority**: High
**Dependencies**: E-1 (Context Engine), AR-2 (RabbitMQ/Celery)

---

### Story 7.3: Gérer les préférences de notification
As a **User**,
I want **configurer mes préférences de notification (types d'alertes, fréquence, canaux)**,
So that **je ne reçoive que les notifications qui m'intéressent**.

**Acceptance Criteria:**
- **Given** un utilisateur authentifié,
- **When** il configure ses préférences,
- **Then** les préférences sont **stockées et appliquées**. 
- **And** l'utilisateur peut **désactiver les notifications** à tout moment.

**Related Requirements**: FR-33
**Estimation**: S
**Priority**: Medium

---

## Epic 8: Web UI
**Goal**: Développer l'interface utilisateur responsive (Next.js) avec 5 onglets et bouton central.

### Story 8.1: Créer le layout mobile-first
As a **User**,
I want **une interface responsive avec 5 onglets en bas et un bouton central flottant**,
So that **je peux naviguer facilement sur mobile et desktop**.

**Acceptance Criteria:**
- **Given** un utilisateur sur mobile ou desktop,
- **When** il accède à l'application,
- **Then** l'interface est **responsive** (mobile, tablette, desktop).
- **And** la navigation comprend :
  - 5 onglets en bas : **Aujourd’hui**, **Explorer**, **+ Action**, **Impact**, **Profil**. 
  - Un **bouton central flottant** pour déclarer une action.
- **And** l'interface est **touch-friendly** (boutons assez grands pour les doigts).

**Related Requirements**: FR-34, FR-35, UX-DR-1
**Estimation**: L
**Priority**: High
**Dependencies**: None

---

### Story 8.2: Afficher le dashboard "Aujourd’hui"
As a **User**,
I want **voir un dashboard avec le contexte environnemental et les recommandations du jour**,
So that **je peux rapidement comprendre les opportunités disponibles**.

**Acceptance Criteria:**
- **Given** un utilisateur authentifié,
- **When** il accède à l'onglet "Aujourd’hui",
- **Then** le dashboard affiche :
  - **Header** :
    - Heure locale (ex: 9:41).
    - Localisation (ex: Lyon 69003).
    - **Météo et contexte environnemental** :
      - Température (ex: 18° Ensoleillé).
      - Qualité de l'air (ex: Air bon, 42 AQI).
      - Précipitations (ex: Pluie faible 10%).
      - Variations (ex: ↑ 21° • ↓ 12°).
  - **Section "Aujourd’hui pour vous"** : Liste de **cartes de recommandations** avec :
    - Icône (ex: 🚲, 💧, 🔧).
    - Titre (ex: "Une belle journée pour sortir à vélo").
    - Description (ex: "Itinéraire de 35 min à 1.8 km de chez vous").

**Related Requirements**: FR-34, UX-DR-2
**Estimation**: M
**Priority**: High
**Dependencies**: E-1 (UnifiedContext), E-2 (Recommendations)

---

### Story 8.3: Afficher les cartes de recommandations
As a **User**,
I want **voir des cartes de recommandations détaillées avec explications scientifiques**,
So that **je peux comprendre pourquoi une recommandation est pertinente et agir en conséquence**.

**Acceptance Criteria:**
- **Given** une recommandation générée,
- **When** elle est affichée,
- **Then** la carte contient :
  - Icône (ex: 🚲 pour vélo, 🌿 pour nature).
  - Titre (ex: "Balade en forêt avec vos enfants").
  - Description (ex: "Qualité de l'air excellente et sentiers accessibles à 5 km").
  - **Explication scientifique** (ex: "Source : Atmo France, AQI=42. Les balades en forêt améliorent la santé mentale.").
  - Bouton **"Valider"** pour ajouter l'action à son historique.

**Related Requirements**: FR-35, FR-36, UX-DR-3
**Estimation**: M
**Priority**: High
**Dependencies**: E-2 (Recommendations), E-3 (Knowledge Engine)

---

### Story 8.4: Afficher l'onglet "Explorer"
As a **User**,
I want **explorer les opportunités par catégories (nature, énergie, mobilité, etc.)**,
So that **je peux découvrir des activités adaptées à mes centres d'intérêt**.

**Acceptance Criteria:**
- **Given** un utilisateur sur l'onglet "Explorer",
- **When** il utilise les filtres,
- **Then** il peut filtrer par :
  - **Catégories** : Nature, Énergie, Mobilité, Jardinage, Réparation, Réemploi, Bien-être.
  - **Localisation** (si activée).
  - **Disponibilité** (ex: aujourd'hui, cette semaine).
- **And** une **liste des opportunités disponibles** est affichée.

**Related Requirements**: FR-35
**Estimation**: M
**Priority**: Medium
**Dependencies**: E-2 (Recommendations)

---

### Story 8.5: Afficher l'onglet "Impact"
As a **User**,
I want **voir mes points, badges, défis, et historique dans l'onglet "Impact"**,
So that **je peux suivre mes progrès et mes accomplissements**.

**Acceptance Criteria:**
- **Given** un utilisateur authentifié,
- **When** il accède à l'onglet "Impact",
- **Then** il voit :
  - **Points cumulés** (ex: 1 250 points).
  - **Liste des badges débloqués** (ex: 🏆 Réparateur, 🌿 Explorateur urbain).
  - **Graphiques de progrès** (ex: évolution des points sur 30 jours).
  - **Journal écologique** (ex: liste des actions réalisées avec date, durée, points gagnés).
  - **Défis en cours** (ex: "7 jours de mobilité douce").

**Related Requirements**: FR-36, UX-DR-3
**Estimation**: M
**Priority**: Medium
**Dependencies**: E-6 (Gamification)

---

### Story 8.6: Afficher l'onglet "Profil"
As a **User**,
I want **gérer mon profil, mes préférences, et mes statistiques dans l'onglet "Profil"**,
So that **je peux personnaliser mon expérience et suivre mes données**.

**Acceptance Criteria:**
- **Given** un utilisateur authentifié,
- **When** il accède à l'onglet "Profil",
- **Then** il peut :
  - **Modifier son profil** (email, timezone, localisation, etc.).
  - **Configurer ses préférences** (activités, transport, durée max).
  - **Voir ses statistiques personnelles** (ex: nombre de recommandations suivies, points gagnés).
  - **Consulter son historique** (ex: liste des actions réalisées).

**Related Requirements**: FR-36
**Estimation**: S
**Priority**: Medium
**Dependencies**: E-5 (User Management)

---

## Dependencies Map

| Story | Dépendances | Statut |
|-------|--------------|--------|
| US-2.1 | US-1.6 (UnifiedContext) | ⚠️ Bloquant |
| US-2.2 | US-2.1 (Rule Engine) | ⚠️ Bloquant |
| US-2.3 | US-2.2 (Scoring) | ⚠️ Bloquant |
| US-2.4 | US-2.1 (Rule Engine) | ⚠️ Bloquant |
| US-2.5 | US-2.1, US-2.2, US-2.4 | ⚠️ Bloquant |
| US-4.1 | None | ✅ |
| US-4.2 | US-4.1 (LLMProvider) | ⚠️ Bloquant |
| US-4.3 | US-4.1 (LLMProvider) | ⚠️ Bloquant |
| US-6.1 | E-5 (User Management) | ⚠️ Bloquant |
| US-6.2 | US-6.1 (Points) | ⚠️ Bloquant |
| US-6.3 | US-6.1 (Points) | ⚠️ Bloquant |
| US-6.4 | US-6.1, US-6.3 | ⚠️ Bloquant |
| US-6.5 | US-6.1 | ⚠️ Bloquant |
| US-6.6 | E-5 (User Management) | ⚠️ Bloquant |
| US-6.7 | US-6.6 (Récompenses) | ⚠️ Bloquant |
| US-6.8 | E-5 (User Management) | ⚠️ Bloquant |
| US-6.9 | US-6.6 | ⚠️ Bloquant |
| US-6.10 | US-6.6, US-6.7 | ⚠️ Bloquant |
| US-7.1 | E-1 (Context Engine) | ⚠️ Bloquant |
| US-7.2 | E-1 (Context Engine), AR-2 (RabbitMQ/Celery) | ⚠️ Bloquant |
| US-7.3 | None | ✅ |
| US-8.1 | None | ✅ |
| US-8.2 | E-1 (UnifiedContext), E-2 (Recommendations) | ⚠️ Bloquant |
| US-8.3 | E-2 (Recommendations), E-3 (Knowledge Engine) | ⚠️ Bloquant |
| US-8.4 | E-2 (Recommendations) | ⚠️ Bloquant |
| US-8.5 | E-6 (Gamification) | ⚠️ Bloquant |
| US-8.6 | E-5 (User Management) | ⚠️ Bloquant |

---

## Phases et Priorisation

### Phase 1: PoC (Proof of Concept)
**Objectif**: Valider le core produit (Context Engine, Recommendation Engine, Web UI, Gamification basique).
**Épics**: E-1, E-2, E-5, E-7, E-8
**Durée estimée**: 1 mois

### Phase 2: Early Access (EA)
**Objectif**: Ajouter les fonctionnalités avancées (Knowledge Engine, Generation Engine, Gamification complète).
**Épics**: E-3, E-4, E-6
**Durée estimée**: 1-2 mois

### Phase 3: General Availability (GA)
**Objectif**: Packaging produit + intégration données partenaires + MCP.
**Épics**: Intégration MCP, scaling, monitoring avancé
**Durée estimée**: 1 mois

---

## Estimations Globales
| Épic | Stories | Estimation Totale |
|------|---------|-------------------|
| E-1  | 7       | 5M + 2S = **~5.5M** |
| E-2  | 5       | 1L + 2M + 2S = **~3.5M** |
| E-3  | 3       | 1M + 2S = **~1.5M** |
| E-4  | 3       | 1M + 2S = **~1.5M** |
| E-5  | 4       | 2M + 2S = **~2.5M** |
| E-6  | 10      | 4M + 4S + 2L = **~8M** |
| E-7  | 3       | 1L + 1M + 1S = **~2.5M** |
| E-8  | 6       | 1L + 4M + 1S = **~5.5M** |
| **Total** | **41** | **~30M** (≈ 30-40 jours de développement) |

---

## Prochaines Étapes
1. **Valider ce document** avec les parties prenantes.
2. **Affiner les estimations** si nécessaire.
3. **Lancer `bmad-sprint-planning`** pour planifier le premier sprint (PoC).
4. **Implémenter les stories** en suivant les dépendances.
