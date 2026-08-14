---
title: Daily Opportunities
date: 2026-08-13
status: draft
created: 2026-08-13
updated: 2026-08-13
author: Julie
---

# PRD: Daily Opportunities
*Plateforme web qui transforme des données environnementales ouvertes, le contexte local, les contraintes de vie et des connaissances scientifiques en recommandations quotidiennes positives, réalistes et personnalisées.*

---

## 0. Document Purpose
Ce document définit les **exigences produit** pour **Daily Opportunities**, une plateforme conçue pour aider les **citoyens** et les **collectivités territoriales** à prendre des décisions éclairées basées sur des **données environnementales locales**. 

- **Public cible** : Product Managers, développeurs, designers, et parties prenantes des collectivités.
- **Structure** : Ce PRD suit une approche **Vision + Fonctionnalités**, avec des **FR (Functional Requirements)** numérotés globalement, des **User Journeys (UJ)** référencés, et un **Glossary** pour les termes clés.
- **Artifacts associés** : 
  - `README.md` : Architecture globale et détails techniques.
  - `API_KEYS_GUIDE.md` : Guide pour les clés API des Providers.

---

## 1. Vision

**Daily Opportunities** est une **plateforme web** qui transforme des **données environnementales ouvertes**, le **contexte local**, les **contraintes de vie** et des **connaissances scientifiques** en **recommandations quotidiennes positives, réalistes et personnalisées**. 

Contrairement aux applications punitives de réduction d'empreinte carbone, **Daily Opportunities** cherche des **opportunités positives** pour répondre à une question simple :
> *« Compte tenu de qui je suis, de l'endroit où je suis, de mon emploi du temps et des conditions du moment, quelle est une bonne chose que je pourrais faire aujourd'hui ? »*

### **Exemples d'opportunités proposées** :
- Balade à pied ou à vélo ;
- Pause bien-être en extérieur ;
- Visite d'un marché local ;
- Activité familiale (parc, forêt) ;
- Meilleur moment pour utiliser de l'électricité (heures creuses) ;
- Jardinage (avec conseils basés sur les saisons et les phases lunaires) ;
- Découverte du territoire (événements, points d'intérêt).

### **Pourquoi ce produit ?**
Pour les **citoyens**, **Daily Opportunities** offre un outil **utile avant d'être écologique**, compatible avec leur **vie quotidienne** (travail, école, famille). Il permet de **découvrir des activités adaptées à leur contexte** (météo, qualité de l'air, disponibilités) sans alourdir leur routine.

Pour les **collectivités territoriales**, il fournit un **levier de sensibilisation** et d'engagement citoyen, en valorisant les **données locales** (ex : restrictions d'eau, événements) et en encourageant les **bonnes pratiques** via un système de **scoring et de gamification** (points, récompenses, historique).

### **Différenciateurs**
- **Approche positive** : Pas de "malus" ou de culpabilisation, uniquement des **opportunités**.
- **Données agrégées** : Météo (APIs), qualité de l'air (Atmo France), mix énergétique (RTE Eco2Mix), eau (Hub'Eau), géographie (OpenStreetMap), et **rythmes de vie** (saisons, lune, journal écologique).
- **Personnalisation** : Recommandations adaptées au **profil utilisateur** (localisation, disponibilités, préférences) et au **contexte temps réel**.
- **Explicabilité** : Chaque recommandation est accompagnée de **sources scientifiques** et d'une **explication claire**.
- **Technologie** : **LLM-agnostique** (abstraction pour éviter le vendor lock-in), **cache et bases de données** pour limiter les coûts et les appels aux APIs.

### **Phases du Projet**
- **PoC (1 mois)** : Version minimale avec **Web UI (Next.js), Backend (FastAPI), Providers (Atmo, RTE, Hub'Eau, OSM), scoring, et explications**.
- **Early Access (EA)** : Ajout des **notifications natives** et des **premiers utilisateurs tests**.
- **General Availability (GA)** : **Packaging produit** pour les collectivités, **intégration de données partenaires**, et **MCP (Model Context Protocol)** pour l'interopérabilité.

### **Mission**
Démocratiser l'accès aux données environnementales pour que chaque citoyen et chaque collectivité puisse **agir concrètement** en faveur de la transition écologique, **sans complexité ni surcoût**.

---

## 2. Target User

### 2.1 Jobs To Be Done
Les utilisateurs de **Daily Opportunities** cherchent à :
- Découvrir des **opportunités quotidiennes positives** adaptées à leur contexte (météo, qualité de l'air, énergie, eau, localisation).
- Recevoir des **recommandations personnalisées** sans culpabilisation (approche "positive by design").
- Comprendre **pourquoi une recommandation est proposée** (explications scientifiques, sources fiables).
- **Personnaliser leur expérience** (profil, préférences, disponibilités).
- **Suivre leurs actions** via un **journal écologique** et un **système de gamification** (points, récompenses).
- Être **alertés en temps réel** pour les opportunités temporaires ou urgentes.

### 2.2 Personas
| **Persona**       | **Contexte**                                                                 | **Besoins**                                                                 |
|-------------------|-----------------------------------------------------------------------------|-----------------------------------------------------------------------------|
| Marie, 35 ans     | Mère de 2 enfants, emploi du temps chargé.                                | Activités familiales et écologiques adaptées à sa routine.                |
| Pierre, 50 ans    | Retraité, passionné de jardinage.                                           | Optimiser son jardinage (saisons, lune, restrictions d'eau).              |
| Sophie, 28 ans    | Jeune active, télétravail, budget serré.                                   | Réduire sa facture d'électricité avec des gestes simples.                  |
| Jean, 45 ans      | Maire d'une commune.                                                        | Sensibiliser ses administrés aux enjeux locaux (eau, air, énergie).        |
| Lucie, 22 ans     | Étudiante, colocation.                                                       | Trouver des activités gratuites et écoresponsables.                       |

### 2.3 Key User Journeys
- **UJ-1 : Marie découvre une activité familiale adaptée à la météo**
  - **Persona + contexte** : Marie, 35 ans, mère de 2 enfants, cherche une activité pour occuper ses enfants cet après-midi.
  - **Entry state** : Marie ouvre l'application **Daily Opportunities** sur son téléphone (Web UI responsive). Elle est authentifiée et sa localisation est activée.
  - **Path** :
    1. Marie voit le **dashboard** avec la météo du jour (ensoleillé, 22°C) et la qualité de l'air (bonne).
    2. Elle consulte la section **"Vos opportunités du jour"** et voit une recommandation : *"Balade en forêt avec vos enfants : qualité de l'air excellente et sentiers accessibles à 5 km."*
    3. Elle clique sur la recommandation pour voir les **détails** : durée (2h), bénéfices (santé, découverte de la nature), et **explications scientifiques** (lien vers une étude sur les bienfaits des balades en forêt).
    4. Elle valide la recommandation et **ajoute l'activité à son historique**.
  - **Climax** : Marie reçoit une **notification** 30 minutes avant l'activité pour lui rappeler de préparer les affaires des enfants.
  - **Resolution** : Marie et ses enfants passent un **moment agréable en forêt**. L'application enregistre l'activité dans son **journal écologique** et lui attribue des **points** pour sa participation.
  - **Edge case** : Si la météo change soudainement (ex : pluie), Marie reçoit une **alerte** avec une alternative : *"La balade est annulée. Essayez plutôt une activité en intérieur : atelier de jardinage en pot."*

- **UJ-2 : Pierre optimise son jardinage grâce aux données locales**
  - **Persona + contexte** : Pierre, 50 ans, veut semer des carottes dans son potager.
  - **Entry state** : Pierre ouvre l'application et consulte la section **"Jardinage"**.
  - **Path** :
    1. Il voit une recommandation : *"Aujourd'hui est un jour idéal pour semer des carottes : phase lunaire favorable, pas de restriction d'arrosage, et sol humide."*
    2. Il clique sur la recommandation pour voir les **sources scientifiques** (ex : calendrier lunaire, données Hub'Eau sur les nappes phréatiques).
    3. Il confirme qu'il va suivre la recommandation.
  - **Climax** : Pierre sème ses carottes et **enregistre l'action dans son journal écologique**.
  - **Resolution** : L'application lui attribue des **points** et lui suggère de **noter l'évolution de ses plantes** dans les jours suivants.
  - **Edge case** : Si une **restriction d'eau** est annoncée, Pierre reçoit une alerte : *"Arrosage interdit aujourd'hui. Reporté à demain."*

- **UJ-3 : Sophie réduit sa consommation d'énergie**
  - **Persona + contexte** : Sophie, 28 ans, veut réduire sa facture d'électricité.
  - **Entry state** : Sophie consulte le **dashboard** et voit le **mix énergétique du jour** (faible carbone).
  - **Path** :
    1. Elle reçoit une recommandation : *"Lave-linge : utilisez-le entre 14h et 16h pour profiter des heures creuses et d'un mix énergétique vert."*
    2. Elle programme son lave-linge pour 15h et **valide la recommandation**.
  - **Climax** : Sophie reçoit une **notification** à 14h30 pour lui rappeler de lancer son lave-linge.
  - **Resolution** : Elle gagne des **points** pour son action et voit son **score énergétique** s'améliorer dans l'application.

- **UJ-4 : Jean (Maire) configure des récompenses pour sa collectivité**
  - **Persona + contexte** : Jean, maire, veut **motiver ses citoyens** à adopter des comportements écoresponsables en leur offrant des **récompenses locales**.
  - **Entry state** : Jean accède au **portail administrateur** de Daily Opportunities.
  - **Path** :
    1. Il **configure le catalogue de récompenses** :
       - Ajoute une récompense : *"1 heure d'accès à la bibliothèque"* (100 points).
       - Ajoute une récompense : *"1 place pour un concert municipal"* (50 points).
       - Ajoute une récompense : *"1 cours de jardinage à la MJC"* (200 points).
    2. Il **définir les règles de points** :
       - +20 points pour une balade en forêt.
       - +10 points pour une action simple (ex : utiliser son lave-linge aux heures creuses).
    3. Il **envoie une notification collective** : *"Nouveautés : Échangez vos points contre des récompenses locales !"*
    4. Il **consulte les statistiques** :
       - Nombre de récompenses attribuées.
       - Points cumulés par les utilisateurs.
  - **Climax** : Les citoyens **échangent leurs points** contre des récompenses et **participent davantage** aux actions écoresponsables.
  - **Resolution** : Jean voit une **amélioration de l'engagement** dans son tableau de bord et peut **ajuster les récompenses** en fonction des retours.

### 2.4 Non-Users (v1)
- Utilisateurs sans accès à un navigateur web.
- Collectivités sans données ouvertes (Hub'Eau, RTE, etc.).
- Utilisateurs cherchant un calcul exhaustif d'empreinte carbone.
- Applications natives iOS/Android (seulement Web UI pour le PoC).

---

## 3. Glossary
*Les termes suivants doivent être utilisés **exactement** dans le reste du document. Aucune synonymie n'est autorisée.*

- **Daily Opportunities** : Plateforme web qui transforme des **données environnementales ouvertes**, le **contexte local**, et les **contraintes de vie** en **recommandations quotidiennes positives, réalistes et personnalisées**.

- **Unified Context** : Contexte normalisé qui agrège toutes les données externes (météo, qualité de l'air, énergie, eau, localisation, etc.) dans un **format standardisé** pour être utilisé par le **Recommendation Engine**.

- **Context Engine** : Moteur responsable de la **collecte, normalisation, mise en cache et enrichissement** des données externes (ex : APIs Atmo France, RTE, Hub'Eau). Il est le **seul composant** à connaître les formats spécifiques des fournisseurs.

- **Recommendation Engine** : Moteur qui **transforme le contexte unifié** en **opportunités concrètes** (ex : "Balade à vélo cet après-midi"). Il utilise des **règles déterministes** et un **système de scoring** pour classer les recommandations.

- **Rule Engine** : Composant du **Recommendation Engine** qui applique des **règles déterministes** pour générer des **candidats de recommandations**. Exemple : `IF air_quality = GOOD AND rain_probability < 30% THEN candidate = BIKE_ACTIVITY`.

- **Scoring** : Système de notation pour classer les recommandations. Le score est calculé selon : 25% ContextFit + 20% TimeFit + 15% UserPreference + 15% Accessibility + 10% EnvironmentalOpportunity + 10% WellbeingOpportunity + 5% Novelty.

- **Knowledge Engine** : Moteur qui **fournit les connaissances scientifiques** nécessaires pour expliquer les recommandations. Il s'appuie sur des **sources institutionnelles** (ADEME, Santé publique France, OFB, EEA, OMS, PubMed, OpenAlex).

- **Generation Engine** : Moteur qui **génère des explications textuelles** à partir des recommandations, du contexte, et des preuves scientifiques. Il utilise une **abstraction LLM-agnostique**.

- **Explanation Engine** : Moteur qui **combine recommandation, contexte, preuve scientifique, et LLM** pour fournir une explication claire et utile à l'utilisateur.

- **LLM-agnostique** : Approche qui permet d'utiliser **n'importe quel modèle de langage (LLM)** via une **interface standardisée** (`LLMProvider`). Cela évite la dépendance à un fournisseur spécifique.

- **MCP (Model Context Protocol)** : Protocole pour **exposer des capacités de haut niveau** (ex : `get_context()`, `get_today_recommendations()`) et permettre l'**interopérabilité** avec des agents externes.

- **Provider** : Fournisseur de données externe (ex : Atmo France, RTE Eco2Mix, Hub'Eau, OpenStreetMap). Chaque provider a un **adapter** pour normaliser ses données.

- **Provider Adapter** : Composant qui **encapsule la logique spécifique** à un fournisseur de données (ex : authentification, format des données).

- **Observation** : Donnée brute collectée par un provider (ex : qualité de l'air à 14h à Paris). Chaque observation contient : `provider`, `observedAt`, `retrievedAt`, `expiresAt`, `quality`.

- **Evidence** : Preuve scientifique qui soutient une recommandation. Chaque evidence contient : `claim`, `source` (titre, éditeur, URL), `evidenceLevel`, `reviewStatus`, `topics`.

- **User Profile** : Profil utilisateur contenant : localisation, horaires, préférences (activités, transport, durée max), et notifications.

- **Context Snapshot** : Instantané du contexte à un moment donné pour un utilisateur. Contient : `user_id`, `location`, `created_at`, `data_json` (météo, air, énergie, etc.).

- **Recommendation** : Objet recommandation contenant : `id`, `user_id`, `type`, `score`, `payload_json`, `context_snapshot_id`, `created_at`, `expires_at`, `status`.

- **Feedback** : Retour utilisateur sur une recommandation. Valeurs possibles : `USEFUL`, `NOT_RELEVANT`, `DONE`, `DISMISSED`.

- **Modular Monolith** : Architecture logicielle où tous les composants (Context Engine, Recommendation Engine, etc.) sont **modulaires mais déployés ensemble** (vs. microservices).

- **Open Data Providers** : Fournisseurs de données ouvertes utilisés par le projet : Atmo France, RTE Eco2Mix, Hub'Eau, OpenStreetMap, Photon (OSM).

- **Récompenses Localisées** : Récompenses **configurées par les collectivités** (ex : accès à la bibliothèque, places pour des événements municipaux, cours offerts) que les utilisateurs peuvent obtenir en échange de leurs points.

- **Catalogue de Récompenses** : Liste des **récompenses disponibles** pour les utilisateurs d'une collectivité, avec le **nombre de points requis** pour chacune.

- **Règles de Points Personnalisables** : Règles **définies par les collectivités** pour attribuer des points aux actions des utilisateurs (ex : 10 points pour une balade, 5 points pour une action simple).

---

## 4. Features
*Chaque sous-section décrit une fonctionnalité cohérente. Les FR (Functional Requirements) sont numérotés globalement et référencent les User Journeys (UJ) et le Glossary.*

---

### 4.1 Context Engine
**Description** :
Le **Context Engine** est responsable de la **collecte, normalisation, mise en cache et enrichissement** des données externes (météo, qualité de l'air, énergie, eau, etc.). Il est le **seul composant** à connaître les formats spécifiques des **Providers** (Atmo France, RTE, Hub'Eau, OpenStreetMap). Les données sont ensuite exposées sous forme de **Unified Context** pour être consommées par le **Recommendation Engine**.

**Providers utilisés pour le PoC** (toutes les APIs sont **publiques et gratuites**) :
- **Atmo France** (`api.atmo-france.org`) : Qualité de l'air (100 requêtes/minute).
- **Météo France** (`api.meteofrance.fr`) : Météo (100 requêtes/jour).
- **RTE** (`api.rte-france.com`) : Mix énergétique (accès public).
- **Hub'Eau** (`hubeau.eaufrance.fr`) : Niveaux d'eau (1000 requêtes/jour).
- **OpenStreetMap** :
  - **Photon** (`photon.komoot.io`) : Géocodage (10 requêtes/seconde).
  - **Overpass API** (`overpass-api.de`) : Données géographiques (2 requêtes/seconde).
- **Phases Lunaires** : Données statiques (calendrier lunaire).

**Pour un usage intensif ou en production** :
- **Météo** : OpenWeatherMap (clé gratuite, 60 appels/minute) ou Météo France Pro (payant).
- **Géocodage** : LocationIQ (clé gratuite, 5000 requêtes/jour) ou une **instance Nominatim locale** (Docker).

**Functional Requirements** :

#### FR-1 : Collecte des données externes
[Actor: Context Engine] peut **récupérer les données** depuis les **Providers** (Atmo France, RTE, Hub'Eau, OSM, etc.) via leurs **Provider Adapters**.
**Consequences (testable)** :
- Les données sont récupérées **sans erreur** (ex : HTTP 200 pour les APIs).
- Les **secrets** (API keys, tokens) ne sont **jamais exposés** côté frontend.
**Realizes** : UJ-1, UJ-2, UJ-3.
**Out of Scope** : La logique métier de génération de recommandations (voir **Recommendation Engine**).

---

#### FR-2 : Normalisation des données
[Actor: Context Engine] peut **normaliser les données brutes** des Providers en un format standardisé (**Observation**).
**Consequences (testable)** :
- Chaque **Observation** contient : `provider`, `observedAt`, `retrievedAt`, `expiresAt`, `quality`.
- Les données normalisées sont **validées** (ex : pas de valeurs nulles pour les champs critiques).
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-3 : Mise en cache des données
[Actor: Context Engine] peut **mettre en cache** les données normalisées pour éviter les appels répétés aux APIs.
**Consequences (testable)** :
- Les données en cache sont **servies en moins de 100ms**.
- Les données expirées sont **rafraîchies automatiquement**.
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-4 : Enrichissement du contexte
[Actor: Context Engine] peut **enrichir le contexte** avec des données dérivées (ex : calculer un indice de qualité de l'air global à partir des données Atmo France).
**Consequences (testable)** :
- Le **Unified Context** contient toutes les données nécessaires pour le **Recommendation Engine**.
- Les données enrichies sont **cohérentes** (ex : pas de conflit entre météo et qualité de l'air).
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-5 : Gestion des erreurs des Providers
[Actor: Context Engine] peut **gérer les erreurs** des Providers (ex : API indisponible, rate limit).
**Consequences (testable)** :
- En cas d'erreur, le système **utilise des données en cache** si disponibles.
- Une **alerte** est générée pour l'administrateur si un Provider critique est indisponible.
**Realizes** : UJ-1, UJ-2, UJ-3.

---
---

### 4.2 Recommendation Engine
**Description** :
Le **Recommendation Engine** transforme le **Unified Context** en **opportunités concrètes** (ex : "Balade à vélo cet après-midi"). Il utilise un **Rule Engine** pour générer des candidats, un **système de scoring** pour les classer, et un **système de diversité** pour éviter les répétitions.

**Functional Requirements** :

---

#### FR-6 : Génération de candidats de recommandations
[Actor: Recommendation Engine] peut **générer des candidats de recommandations** à partir du **Unified Context** et du **User Profile** via le **Rule Engine**.
**Consequences (testable)** :
- Chaque candidat est **valide** (ex : pas de recommandation de vélo si la qualité de l'air est mauvaise).
- Les candidats sont **testables indépendamment** du LLM (voir **ADR-003**).
**Realizes** : UJ-1, UJ-2, UJ-3.
**Example Rule** :
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

---

#### FR-7 : Calcul du score des recommandations
[Actor: Recommendation Engine] peut **calculer un score** pour chaque candidat en utilisant la formule :
`Score = 25% ContextFit + 20% TimeFit + 15% UserPreference + 15% Accessibility + 10% EnvironmentalOpportunity + 10% WellbeingOpportunity + 5% Novelty`.
**Consequences (testable)** :
- Le score est **déterministe** (mêmes entrées → même score).
- Le score est **normalisé entre 0 et 1**.
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-8 : Classement des recommandations
[Actor: Recommendation Engine] peut **classer les recommandations** par score décroissant.
**Consequences (testable)** :
- Les recommandations sont **triées par pertinence**.
- Les recommandations **diverses** (ex : pas que du vélo) sont priorisées.
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-9 : Filtrage des recommandations invalides
[Actor: Recommendation Engine] peut **supprimer les recommandations invalides** (ex : conditions météo dangereuses, qualité de l'air incompatible).
**Consequences (testable)** :
- Une recommandation est supprimée si :
  - Conditions météo dangereuses.
  - Qualité de l'air incompatible.
  - Restriction locale incompatible (ex : restriction d'eau).
  - Données critiques indisponibles.
  - Temps insuffisant.
  - Itinéraire non disponible.
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-10 : Génération de recommandations déterministes
[Actor: Recommendation Engine] peut **générer des recommandations sans utiliser de LLM** (pour les tests et la reproductibilité).
**Consequences (testable)** :
- Les recommandations sont **reproductibles** (mêmes entrées → mêmes sorties).
- Les tests unitaires du **Recommendation Engine** fonctionnent **sans LLM**.
**Realizes** : UJ-1, UJ-2, UJ-3.

---
---

### 4.3 Knowledge Engine
**Description** :
Le **Knowledge Engine** fournit les **connaissances scientifiques** nécessaires pour expliquer les recommandations. Il s'appuie sur des **sources institutionnelles** (ADEME, Santé publique France, OFB, EEA, OMS, PubMed, OpenAlex).

**Functional Requirements** :

---

#### FR-11 : Gestion des preuves scientifiques
[Actor: Knowledge Engine] peut **stocker et récupérer des preuves scientifiques** (Evidence) pour soutenir les recommandations.
**Consequences (testable)** :
- Chaque **Evidence** contient : `id`, `claim`, `source` (titre, éditeur, URL), `evidenceLevel`, `reviewStatus`, `topics`.
- Les preuves sont **classées par niveau de confiance** (ex : `HIGH`, `MEDIUM`, `LOW`).
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-12 : Association des preuves aux recommandations
[Actor: Knowledge Engine] peut **associer des preuves scientifiques** à une recommandation.
**Consequences (testable)** :
- Chaque recommandation a au moins **une Evidence** associée.
- Les preuves sont **pertinentes** (ex : une preuve sur la qualité de l'air pour une recommandation de balade).
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-13 : Mise à jour des preuves scientifiques
[Actor: Knowledge Engine] peut **mettre à jour les preuves scientifiques** avec de nouvelles sources (ex : études récentes).
**Consequences (testable)** :
- Les preuves sont **rafraîchies périodiquement** (ex : toutes les semaines).
- Les nouvelles preuves sont **validées** avant d'être ajoutées.
**Realizes** : UJ-1, UJ-2.

---
---

### 4.4 Generation Engine
**Description** :
Le **Generation Engine** génère des **explications textuelles** à partir des recommandations, du contexte, et des preuves scientifiques. Il utilise une **abstraction LLM-agnostique** pour éviter le vendor lock-in.

**Functional Requirements** :

---

#### FR-14 : Génération d'explications textuelles
[Actor: Generation Engine] peut **générer des explications textuelles** pour une recommandation en utilisant un **LLM**.
**Consequences (testable)** :
- L'explication est **claire et utile** pour l'utilisateur.
- L'explication contient **au moins une raison** (ex : "Qualité de l'air excellente").
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-15 : Abstraction LLM-agnostique
[Actor: Generation Engine] peut **utiliser n'importe quel LLM** via l'interface `LLMProvider`.
**Consequences (testable)** :
- Le code ne dépend **pas d'un fournisseur LLM spécifique**.
- Le **routing** entre différents LLM est possible (ex : utiliser un LLM local pour les tests, un LLM cloud pour la production).
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-16 : Fallback déterministe
[Actor: Generation Engine] peut **fournir une explication déterministe minimale** si le LLM est indisponible.
**Consequences (testable)** :
- Exemple : *"Bonne qualité de l'air + météo favorable : une balade de 30 minutes est une bonne option aujourd'hui."*
- L'explication est **toujours disponible**, même sans LLM.
**Realizes** : UJ-1, UJ-2, UJ-3.

---
---

### 4.5 User Management
**Description** :
Gestion des **profils utilisateurs**, de leurs **préférences**, et de leur **historique**.

**Functional Requirements** :

---

#### FR-17 : Création et mise à jour du profil utilisateur
[Actor: User] peut **créer et mettre à jour son profil** (localisation, horaires, préférences).
**Consequences (testable)** :
- Le profil contient : `id`, `email`, `timezone`, `home_location`, `work_location`, `preferences` (activités, transport, durée max).
- Les données sont **validées** (ex : email valide, localisation existante).
**Realizes** : UJ-1, UJ-2, UJ-3, UJ-4.

---

#### FR-18 : Gestion des préférences utilisateur
[Actor: User] peut **configurer ses préférences** (ex : activités préférées, modes de transport, durée max des activités).
**Consequences (testable)** :
- Les préférences sont **prises en compte** dans les recommandations.
- Les préférences sont **stockées et récupérables**.
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-19 : Historique des recommandations
[Actor: User] peut **consulter son historique** de recommandations (ex : activités réalisées, points gagnés).
**Consequences (testable)** :
- L'historique contient : `recommendation_id`, `type`, `score`, `date`, `feedback`.
- L'historique est **filtrable** (ex : par date, par type).
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-20 : Feedback sur les recommandations
[Actor: User] peut **donner un feedback** sur une recommandation (`USEFUL`, `NOT_RELEVANT`, `DONE`, `DISMISSED`).
**Consequences (testable)** :
- Le feedback est **enregistré** et utilisé pour améliorer les recommandations futures.
- Le feedback est **anonyme** (pas de données personnelles stockées).
**Realizes** : UJ-1, UJ-2, UJ-3.

---
---

### 4.6 Gamification
**Description** :
Système de **motivation** basé sur des **points**, des **récompenses personnalisables**, et un **journal écologique** pour encourager les utilisateurs à suivre les recommandations. Les **collectivités** peuvent **configurer leurs propres récompenses** (ex : accès à la bibliothèque, places pour des événements, cours offerts) et **définir les règles d'attribution des points**.

**Functional Requirements** :

---

#### FR-21 : Système de points
[Actor: User] peut **gagner des points** en suivant les recommandations.
**Consequences (testable)** :
- Chaque recommandation **réalisée** rapporte des points (ex : +10 points pour une balade, +5 points pour une action simple comme utiliser son lave-linge aux heures creuses).
- Les points sont **affichés en temps réel** dans le profil utilisateur.
- Les points sont **cumulables** et **non périssables** (sauf configuration spécifique par la collectivité).
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-22 : Récompenses personnalisables par les collectivités
[Actor: Admin] peut **configurer des récompenses locales** pour les utilisateurs de sa collectivité.
**Consequences (testable)** :
- Les récompenses peuvent être :
  - **Accès à des services municipaux** (ex : 100 points = 1 heure d'accès à la bibliothèque).
  - **Places pour des événements** (ex : 50 points = 1 place pour un concert municipal).
  - **Cours ou ateliers offerts** (ex : 200 points = 1 cours de jardinage à la MJC).
  - **Autres avantages** (ex : réductions dans les commerces locaux partenaires).
- Les récompenses sont **affichées dans un catalogue** accessible aux utilisateurs.
- Les récompenses sont **modifiables** par l'administrateur (ajout/suppression, modification des points requis).
**Realizes** : UJ-4.

---

#### FR-23 : Échange de points contre des récompenses
[Actor: User] peut **échanger ses points contre des récompenses** disponibles dans le catalogue de sa collectivité.
**Consequences (testable)** :
- L'utilisateur voit **les récompenses disponibles** et le **nombre de points requis** pour chacune.
- L'échange est **validé automatiquement** si l'utilisateur a suffisamment de points.
- Une **confirmation** est envoyée à l'utilisateur (ex : email ou notification avec un code ou un bon à présenter).
- Les points sont **déduits du solde** de l'utilisateur après échange.
**Realizes** : UJ-1, UJ-2, UJ-3, UJ-4.

---

#### FR-24 : Journal Écologique
[Actor: User] peut **enregistrer ses actions** dans un **journal écologique** (ex : balade, jardinage, utilisation d'énergie aux heures creuses).
**Consequences (testable)** :
- Le journal contient : `action`, `date`, `durée`, `points gagnés`, `catégorie` (ex : nature, énergie, bien-être).
- Le journal est **exportable** (ex : CSV, PDF).
- Le journal est **filtrable** (ex : par catégorie, par date).
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-25 : Suivi des progrès
[Actor: User] peut **suivre ses progrès** (ex : nombre de recommandations suivies, points accumulés, badges débloqués).
**Consequences (testable)** :
- Les progrès sont **affichés sous forme de graphiques** (ex : évolution des points sur 30 jours).
- Les progrès sont **comparables** (ex : comparaison avec la moyenne des utilisateurs de sa collectivité).
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-26 : Badges
[Actor: User] peut **débloquer des badges** en accumulant des points ou en réalisant des actions spécifiques.
**Consequences (testable)** :
- Les badges sont **associés à des accomplissements** (ex : "Expert en jardinage" après 10 actions de jardinage, "Éco-citoyen" après 50 recommandations suivies).
- Les badges sont **affichés dans le profil utilisateur**.
- Les badges peuvent être **partagés** (ex : sur les réseaux sociaux ou via un lien public).
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-27 : Classement (Leaderboard)
[Actor: User] peut **voir son classement** par rapport aux autres utilisateurs de sa collectivité.
**Consequences (testable)** :
- Le classement est **anonyme** (ex : "Vous êtes 3ème sur 100 utilisateurs cette semaine").
- Le classement est **mis à jour en temps réel**.
- Le classement peut être **filtré** (ex : par semaine, par mois, par catégorie d'action).
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-28 : Personnalisation des règles de points par les collectivités
[Actor: Admin] peut **définir les règles d'attribution des points** pour sa collectivité.
**Consequences (testable)** :
- L'administrateur peut **configurer le nombre de points** attribués pour chaque type d'action (ex : +20 points pour une balade en forêt, +5 points pour une action simple).
- Les règles peuvent être **modifiées à tout moment** et s'appliquent **immédiatement** aux nouveaux utilisateurs.
- Les règles sont **affichées clairement** pour les utilisateurs (ex : "Dans cette collectivité, une balade rapporte 20 points").
**Realizes** : UJ-4.

---

#### FR-29 : Gestion des récompenses par les collectivités
[Actor: Admin] peut **gérer le catalogue de récompenses** pour sa collectivité.
**Consequences (testable)** :
- L'administrateur peut :
  - **Ajouter/supprimer des récompenses**.
  - **Modifier le nombre de points requis** pour une récompense.
  - **Définir des limites** (ex : 1 récompense par utilisateur et par mois).
  - **Suivre les échanges** (ex : nombre de récompenses attribuées, points dépensés).
- Les récompenses sont **affichées dans un catalogue** avec :
  - Une **description** (ex : "1 heure d'accès à la bibliothèque").
  - Le **nombre de points requis**.
  - Les **disponibilités** (ex : "5 places restantes").
**Realizes** : UJ-4.

---

#### FR-30 : Notifications pour les récompenses
[Actor: User] peut **recevoir des notifications** lorsqu'une nouvelle récompense est disponible ou lorsqu'il a suffisamment de points pour en bénéficier.
**Consequences (testable)** :
- Les notifications sont **envoyées via les canaux configurés** (email, appli native).
- Les notifications contiennent :
  - Le **nom de la récompense**.
  - Le **nombre de points requis**.
  - Un **lien direct** pour échanger les points.
**Realizes** : UJ-1, UJ-2, UJ-3, UJ-4.

---
---

### 4.7 Notifications
**Description** :
Système d'**alertes en temps réel** pour les opportunités temporaires ou urgentes (ex : météo, qualité de l'air).

**Functional Requirements** :

---

#### FR-31 : Notifications pour les opportunités temporaires
[Actor: User] peut **recevoir des notifications** pour les opportunités temporaires (ex : "Balade à vélo cet après-midi : qualité de l'air excellente").
**Consequences (testable)** :
- Les notifications sont **envoyées via les applis natives** (phase EA) ou **email** (phase PoC).
- Les notifications sont **personnalisées** (ex : basées sur les préférences utilisateur).
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-32 : Alertes pour les conditions critiques
[Actor: User] peut **recevoir des alertes** pour les conditions critiques (ex : "Qualité de l'air mauvaise : évitez les activités extérieures intenses").
**Consequences (testable)** :
- Les alertes sont **prioritaires** (ex : affichées en haut du dashboard).
- Les alertes sont **envoyées en temps réel**.
**Realizes** : UJ-1, UJ-2, UJ-3, UJ-4.

---

#### FR-33 : Gestion des préférences de notification
[Actor: User] peut **configurer ses préférences de notification** (ex : types d'alertes, fréquence, canaux).
**Consequences (testable)** :
- Les préférences sont **stockées et appliquées**.
- L'utilisateur peut **désactiver les notifications** à tout moment.
**Realizes** : UJ-1, UJ-2, UJ-3.

---
---

### 4.8 Web UI
**Description** :
Interface utilisateur **responsive** (Next.js) pour afficher les recommandations, le contexte, et les explications.

**Functional Requirements** :

---

#### FR-34 : Dashboard principal
[Actor: User] peut **consulter le dashboard principal** avec :
- **Header** : Bonjour + météo + qualité de l'air + coucher du soleil + notifications.
- **Context Cards** : Qualité de l'air, mix énergétique, eau, météo.
- **Main Section** : "Vos opportunités du jour" (liste de recommandations).
**Consequences (testable)** :
- Le dashboard est **affiché en moins de 2 secondes**.
- Le dashboard est **responsive** (mobile, tablette, desktop).
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-35 : Cartes de recommandations
[Actor: User] peut **voir les détails d'une recommandation** (catégorie, action, durée, raisons, niveau de confiance, bénéfices).
**Consequences (testable)** :
- Chaque carte contient un **bouton "Pourquoi cette suggestion ?"** qui affiche les **Evidence** associées.
- Les cartes sont **classées par score**.
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-36 : Carte interactive
[Actor: User] peut **consulter une carte interactive** (OpenStreetMap) avec :
- Position de l'utilisateur.
- Parcs, pistes vélo, marchés, événements, activités.
**Consequences (testable)** :
- La carte est **centrée sur la localisation de l'utilisateur**.
- Les points d'intérêt sont **filtrables** (ex : par catégorie).
**Realizes** : UJ-1, UJ-2.

---

#### FR-37 : Historique et journal écologique
[Actor: User] peut **consulter son historique** et son **journal écologique** (actions enregistrées, points gagnés, badges).
**Consequences (testable)** :
- L'historique est **affiché sous forme de liste ou de calendrier**.
- Le journal écologique est **exportable**.
**Realizes** : UJ-1, UJ-2, UJ-3.

---
---

### 4.9 Admin Portal
**Description** :
Portail pour les **administrateurs** (collectivités) afin de configurer des **alertes locales** et consulter des **statistiques**.

**Functional Requirements** :

---

#### FR-38 : Configuration des alertes locales
[Actor: Admin] peut **configurer des alertes locales** (ex : restrictions d'eau, événements spéciaux).
**Consequences (testable)** :
- Les alertes sont **envoyées aux utilisateurs** de la collectivité.
- Les alertes sont **prioritaires** (ex : affichées en haut du dashboard).
**Realizes** : UJ-4.

---

#### FR-39 : Statistiques d'engagement
[Actor: Admin] peut **consulter les statistiques d'engagement** de sa collectivité (ex : nombre de recommandations suivies, points accumulés).
**Consequences (testable)** :
- Les statistiques sont **affichées sous forme de graphiques**.
- Les statistiques sont **exportables** (ex : CSV, PDF).
**Realizes** : UJ-4.

---

#### FR-40 : Gestion des utilisateurs
[Actor: Admin] peut **gérer les utilisateurs** de sa collectivité (ex : ajouter/supprimer des utilisateurs, configurer des groupes).
**Consequences (testable)** :
- Les utilisateurs sont **associés à une collectivité**.
- Les groupes permettent de **cibler des alertes** (ex : seulement pour les habitants d'un quartier).
**Realizes** : UJ-4.

---
---

### 4.10 Saisons & Phases Lunaires
**Description** :
Intégration des **données saisonnières et lunaires** pour adapter les recommandations (ex : conseils de jardinage).

**Functional Requirements** :

---

#### FR-41 : Intégration des données saisonnières
[Actor: Context Engine] peut **intégrer des données saisonnières** (ex : dates des saisons, températures moyennes).
**Consequences (testable)** :
- Les recommandations de jardinage sont **adaptées à la saison** (ex : "Semer des tomates en été").
- Les données saisonnières sont **mises à jour automatiquement**.
**Realizes** : UJ-2.

---

#### FR-42 : Intégration des phases lunaires
[Actor: Context Engine] peut **intégrer des données sur les phases lunaires** (ex : pleine lune, nouvelle lune).
**Consequences (testable)** :
- Les recommandations de jardinage sont **adaptées aux phases lunaires** (ex : "Semer en lune croissante").
- Les données lunaires sont **mises à jour quotidiennement**.
**Realizes** : UJ-2.

---

#### FR-43 : Recommandations basées sur les rythmes de vie
[Actor: Recommendation Engine] peut **générer des recommandations** basées sur les **rythmes de vie** de l'utilisateur (ex : horaires de travail, temps libre).
**Consequences (testable)** :
- Les recommandations sont **adaptées aux disponibilités** de l'utilisateur.
- Les rythmes de vie sont **configurables** dans le profil utilisateur.
**Realizes** : UJ-1, UJ-2, UJ-3.

---
---

## 5. Non-Goals (Explicit)
*Ce que **Daily Opportunities** ne fera **pas** en v1 (PoC). Ces éléments pourront être ajoutés dans des versions ultérieures (EA ou GA).*

- **Applications natives iOS/Android** : Reporté à la phase **Early Access (EA)**.
- **Réseau social** : Pas de feed, likes, commentaires, ou partage public.
- **Publicité** : Aucune publicité ne sera intégrée.
- **Calcul exhaustif d'empreinte carbone** : Le produit se concentre sur les **opportunités positives**.
- **Diagnostic médical** : Les recommandations ne concernent pas la santé ou le diagnostic médical.
- **Contrôle automatique d'appareils** : Pas de contrôle d'appareils connectés (ex : lave-linge).
- **Infrastructure Kubernetes** : Reporté à une phase ultérieure (GA).
- **Microservices** : Le PoC utilisera un **modular monolith**.
- **MCP (Model Context Protocol)** : Reporté à la phase **GA**.
- **Packaging produit pour les collectivités** : Reporté à la phase **GA**.
- **Intégration de données partenaires clients** : Reporté à la phase **GA**.
- **Authentification avancée (SSO, OAuth2.0)** : Le PoC utilisera une authentification basique (email/mot de passe).
- **Multi-tenancy avancé** : Le PoC supportera une **seule collectivité**.
- **Capteurs IoT** : Pas d'intégration avec des capteurs physiques.
- **Traduction automatique** : Le PoC sera en **français uniquement**.
- **Notifications natives (push)** : Reporté à la phase **EA**.

---

## 6. MVP Scope

### 6.1 In Scope
*Ce qui sera **implémenté** dans le PoC (1 mois).*

#### Frontend
- **Web UI responsive** (Next.js/TypeScript) :
  - Dashboard principal (météo, qualité de l'air, mix énergétique, eau).
  - Cartes de recommandations (catégorie, action, durée, raisons, bénéfices).
  - Carte interactive (OpenStreetMap) avec points d'intérêt (parcs, pistes vélo, marchés).
  - Historique des recommandations et **journal écologique**.
  - Classements (leaderboard) et badges.
  - Profil utilisateur (localisation, préférences, disponibilités).

#### Backend
- **API** (FastAPI) avec endpoints pour :
  - Récupération du contexte (`/context/current`, `/context/air`, `/context/energy`, `/context/water`).
  - Récupération des recommandations (`/recommendations/today`, `/recommendations/{id}`).
  - Gestion des utilisateurs (`/me`, `/me/preferences`).
  - Feedback (`/recommendations/{id}/feedback`).
  - Evidence (`/evidence/{id}`).
  - Carte (`/map/opportunities`).

#### Base de Données
- **PostgreSQL + PostGIS** :
  - Stockage des utilisateurs, observations, recommandations, evidence, feedback.
  - Schéma optimisé pour les requêtes du **Recommendation Engine**.

#### Cache
- **Redis** :
  - Mise en cache des données des **Providers** pour éviter les appels répétés.
  - Gestion des **Context Snapshots**.

#### Moteurs
- **Context Engine** :
  - Collecte et normalisation des données des **Providers** (Atmo France, RTE Eco2Mix, Hub'Eau, OpenStreetMap, Météo France).
  - Mise en cache et enrichissement du contexte.
- **Recommendation Engine** :
  - Génération de candidats via le **Rule Engine**.
  - Calcul du **score** (formule définie).
  - Classement et filtrage des recommandations.
- **Knowledge Engine** :
  - Gestion des **Evidence** (sources scientifiques : ADEME, Santé publique France, OFB, etc.).
- **Generation Engine** :
  - Génération d'explications textuelles via **LLM-agnostique** (`LLMProvider`).
  - Fallback déterministe si LLM indisponible.
- **Explanation Engine** :
  - Combinaison de recommandation, contexte, preuve, et LLM pour des explications claires.

#### Gamification
- **Système de points** :
  - Points attribués pour chaque recommandation suivie.
  - Points **personnalisables manuellement** pour le PoC (configuration basique).
- **Badges** :
  - Badges pour accomplissements (ex : "Expert en jardinage", "Éco-citoyen").
- **Journal Écologique** :
  - Enregistrement des actions (balade, jardinage, etc.).
  - Exportable (CSV, PDF).
- **Classement (Leaderboard)** :
  - Classement des utilisateurs par points.

#### Notifications
- **Notifications basiques** (email ou in-app) :
  - Alertes pour opportunités temporaires (ex : "Balade à vélo cet après-midi").
  - Alertes pour conditions critiques (ex : "Qualité de l'air mauvaise").

#### Tests
- **Tests unitaires** :
  - Context Engine, Recommendation Engine, Rule Engine, Scoring.
- **Tests d'intégration** :
  - Provider → Context → Recommendation → Generation → API.
- **Tests end-to-end** :
  - Création de profil → consultation du dashboard → suivi d'une recommandation → feedback.

#### Déploiement
- **Docker Compose** :
  - Conteneurs pour : Web (Next.js), API (FastAPI), Worker, PostgreSQL, Redis.
  - Exécution locale sur iMac.

#### Données Supplémentaires
- **Saisons & Phases Lunaires** :
  - Intégration basique pour adapter les recommandations (ex : jardinage).
- **Rythmes de Vie** :
  - Prise en compte des horaires de travail et disponibilités dans le profil utilisateur.

#### Documentation
- **README.md** :
  - Description du projet, architecture, déploiement.
- **OpenAPI** :
  - Documentation automatique de l'API.
- **Schémas d'architecture** :
  - Diagrammes Mermaid pour Context Engine, Recommendation Engine, etc.

---

### 6.2 Out of Scope for MVP
*Ce qui **ne sera pas implémenté** dans le PoC (reporté à EA ou GA).*

- **Applications natives iOS/Android** : Reporté à la phase **Early Access (EA)**.
- **Réseau social** : Pas de feed, likes, commentaires, ou partage public.
- **Publicité** : Aucune publicité ne sera intégrée.
- **Calcul exhaustif d'empreinte carbone** : Le produit se concentre sur les **opportunités positives**.
- **Diagnostic médical** : Les recommandations ne concernent pas la santé ou le diagnostic médical.
- **Contrôle automatique d'appareils** : Pas de contrôle d'appareils connectés (ex : lave-linge).
- **Infrastructure Kubernetes** : Reporté à une phase ultérieure (GA).
- **Microservices** : Le PoC utilisera un **modular monolith**.
- **MCP (Model Context Protocol)** : Reporté à la phase **GA**.
- **Packaging produit pour les collectivités** : Reporté à la phase **GA**.
- **Intégration de données partenaires clients** : Reporté à la phase **GA**.
- **Authentification avancée (SSO, OAuth2.0)** : Le PoC utilisera une authentification basique (email/mot de passe).
- **Multi-tenancy avancé** : Le PoC supportera une **seule collectivité**.
- **Capteurs IoT** : Pas d'intégration avec des capteurs physiques.
- **Traduction automatique** : Le PoC sera en **français uniquement**.
- **Notifications natives (push)** : Reporté à la phase **EA**.

---

## 7. Success Metrics

*Les métriques de succès pour **Daily Opportunities** se concentrent sur **l'utilité perçue**, **l'engagement des utilisateurs**, et **la pertinence des recommandations**. Elles évitent les métriques complexes (ex : calcul d'empreinte carbone) au profit de mesures **concrètes et actionnables**.*

---

### 7.1 Product Metrics *(Métriques Produit)*
*Mesurent l'adoption et l'utilisation du produit par les utilisateurs.*

- **SM-1 : Daily Active Users (DAU)**
  - **Définition** : Nombre d'**utilisateurs uniques** qui ouvrent l'application ou consultent le dashboard **au moins une fois par jour**.
  - **Cible** : 100 DAU en phase **Early Access** (EA), 1 000 DAU en phase **GA** (pour une collectivité moyenne).
  - **Mesure** : Comptage des sessions uniques via les logs ou la base de données.
  - **Validates** : FR-17 (Création de profil utilisateur), FR-34 (Dashboard principal).

- **SM-2 : Weekly Retention Rate**
  - **Définition** : Pourcentage d'utilisateurs qui **reviennent utiliser l'application au moins une fois par semaine**.
  - **Cible** : 50% en phase EA, 70% en phase GA.
  - **Mesure** : `(Nombre d'utilisateurs actifs cette semaine / Nombre d'utilisateurs actifs la semaine précédente) * 100`.
  - **Validates** : FR-17 (Profil utilisateur), FR-34 (Dashboard).

- **SM-3 : Recommendation Open Rate**
  - **Définition** : Pourcentage de recommandations **affichées** qui sont **ouvertes** (cliquées) par l'utilisateur.
  - **Cible** : 60% en phase EA, 75% en phase GA.
  - **Mesure** : `(Nombre de recommandations cliquées / Nombre de recommandations affichées) * 100`.
  - **Validates** : FR-35 (Cartes de recommandations), FR-6 (Génération de candidats).

- **SM-4 : Recommendation Completion Rate**
  - **Définition** : Pourcentage de recommandations **marquées comme "DONE"** par l'utilisateur (via feedback).
  - **Cible** : 30% en phase EA, 50% en phase GA.
  - **Mesure** : `(Nombre de recommandations marquées "DONE" / Nombre de recommandations affichées) * 100`.
  - **Validates** : FR-20 (Feedback sur les recommandations), FR-6 (Génération de candidats).

- **SM-5 : Repeat Usage**
  - **Définition** : Nombre moyen de **sessions par utilisateur et par jour**.
  - **Cible** : 2 sessions/jour en phase EA, 3 sessions/jour en phase GA.
  - **Mesure** : `(Nombre total de sessions / Nombre d'utilisateurs actifs) / Nombre de jours`.
  - **Validates** : FR-34 (Dashboard), FR-17 (Profil utilisateur).

---

### 7.2 Quality Metrics *(Métriques de Qualité)*
*Mesurent la **pertinence**, **l'exactitude**, et **l'utilité** des recommandations et explications.*

- **SM-6 : Recommendation Relevance**
  - **Définition** : Pourcentage de recommandations **jugées "USEFUL"** par les utilisateurs (via feedback).
  - **Cible** : 80% en phase EA, 90% en phase GA.
  - **Mesure** : `(Nombre de feedbacks "USEFUL" / Nombre total de feedbacks) * 100`.
  - **Validates** : FR-20 (Feedback), FR-8 (Classement des recommandations).

- **SM-7 : Explanation Accuracy**
  - **Définition** : Pourcentage d'**explications** (générées par le Generation Engine) **jugées claires et utiles** par les utilisateurs.
  - **Cible** : 85% en phase EA, 95% en phase GA.
  - **Mesure** : Évaluation manuelle ou via feedback utilisateur (ex : "Cette explication était-elle utile ?").
  - **Validates** : FR-14 (Génération d'explications), FR-16 (Fallback déterministe).

- **SM-8 : Provider Freshness**
  - **Définition** : Pourcentage de données des **Providers** (Atmo, RTE, Hub'Eau) qui sont **à jour** (non expirées).
  - **Cible** : 95% en phase EA et GA.
  - **Mesure** : `(Nombre d'observations non expirées / Nombre total d'observations) * 100`.
  - **Validates** : FR-3 (Mise en cache des données), FR-5 (Gestion des erreurs des Providers).

- **SM-9 : LLM Hallucination Rate**
  - **Définition** : Pourcentage d'**explications générées par LLM** qui contiennent des **informations incorrectes ou inventées** (ex : météo inventée, trajet inexistant).
  - **Cible** : < 5% en phase EA et GA.
  - **Mesure** : Audit manuel ou via **Output Validator** (FR-656 : "Le LLM ne doit pas inventer une météo").
  - **Validates** : FR-15 (Abstraction LLM-agnostique), FR-656 (Règles pour le LLM).

- **SM-10 : Percentage of Recommendations with Sufficient Evidence**
  - **Définition** : Pourcentage de recommandations qui ont **au moins une Evidence scientifique** associée.
  - **Cible** : 100% en phase EA et GA.
  - **Mesure** : `(Nombre de recommandations avec Evidence / Nombre total de recommandations) * 100`.
  - **Validates** : FR-11 (Gestion des preuves scientifiques), FR-12 (Association des preuves aux recommandations).

---

### 7.3 Principle Metric *(Métrique Principale)*
*La métrique la plus importante, qui résume la valeur du produit pour l'utilisateur.*

- **SM-P1 : "How often did the application help the user discover something useful, enjoyable or meaningful that fit naturally into their day?"**
  - **Définition** : Pourcentage d'utilisateurs qui **déclarent** que l'application les a aidés à découvrir **au moins une opportunité utile, agréable ou significative** qui s'intégrait naturellement dans leur journée.
  - **Cible** : 70% en phase EA, 85% en phase GA.
  - **Mesure** :
    - **Enquête utilisateur** : Question posée après 1 semaine d'utilisation : *"L'application vous a-t-elle aidé à découvrir au moins une opportunité utile, agréable ou significative cette semaine ?"* (Réponse : Oui/Non).
    - **Feedback implicite** : Combinaison de SM-3 (Open Rate), SM-4 (Completion Rate), et SM-6 (Relevance).
  - **Validates** : **Toutes les fonctionnalités** (Context Engine, Recommendation Engine, Gamification, etc.).

---

### 7.4 Counter-Metrics *(Métriques à Ne Pas Optimiser)*
*Ces métriques ne doivent **pas** être optimisées au détriment des métriques principales.*

- **SM-C1 : Nombre de recommandations générées par jour**
  - **Pourquoi ne pas optimiser ?** : Générer plus de recommandations **ne garantit pas** qu'elles seront utiles ou pertinentes. Mieux vaut **quelques recommandations très pertinentes** que beaucoup de recommandations peu utiles.
  - **Lien avec SM-P1** : Optimiser SM-C1 pourrait **diminuer SM-P1** (utilité perçue).

- **SM-C2 : Temps passé sur l'application**
  - **Pourquoi ne pas optimiser ?** : Un temps passé élevé peut indiquer que l'utilisateur **ne trouve pas ce qu'il cherche** (mauvaise UX) ou qu'il est **perdu**.
  - **Lien avec SM-P1** : Optimiser SM-C2 pourrait **diminuer SM-5** (Repeat Usage) si l'expérience est frustrante.

- **SM-C3 : Nombre de points accumulés par utilisateur**
  - **Pourquoi ne pas optimiser ?** : Accumuler des points **ne signifie pas** que l'utilisateur trouve l'application utile. Cela pourrait encourager des **comportements artificiels** (ex : suivre des recommandations sans intérêt juste pour les points).
  - **Lien avec SM-P1** : Optimiser SM-C3 pourrait **diminuer SM-P1** si les utilisateurs se concentrent sur les points plutôt que sur la valeur réelle.

---

## 8. Open Questions
*Liste des questions non résolues qui devront être adressées avant ou pendant le développement.*

---

### 8.1 Données et Providers
| **ID** | **Question** | **Impact** | **Propriétaire** | **Échéance** | **Statut** |
|--------|--------------|------------|------------------|--------------|------------|
| OQ-1   | Quelles APIs météo utiliser pour le PoC ? | Choix impacte la qualité des données et les coûts. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Utiliser Météo France (publique, 100 requêtes/jour). |
| OQ-2   | Comment accéder aux données RTE Eco2Mix ? | Nécessaire pour intégrer le mix énergétique. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Utiliser l'API publique sans clé. |
| OQ-3   | Comment gérer les restrictions d'eau (Hub'Eau) ? | Intégration des données locales. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Utiliser l'API publique sans clé (1000 requêtes/jour). |
| OQ-4   | Quelles données astronomiques utiliser pour les phases lunaires ? | Nécessaire pour le jardinage. | Julie/Équipe Dev | Phase EA | ✅ **Résolue** : Utiliser des données statiques (calendrier lunaire). |
| OQ-5   | Comment gérer les données de rythmes de vie (ex : horaires de travail) ? | Personnalisation des recommandations. | Julie/Équipe Dev | Avant le PoC | ⚠️ **À résoudre** : Ajouter un champ `work_schedule` dans le **User Profile** (FR-17). |

---

### 8.2 Architecture et Technique
| **ID** | **Question** | **Impact** | **Propriétaire** | **Échéance** | **Statut** |
|--------|--------------|------------|------------------|--------------|------------|
| OQ-6   | Comment structurer le modular monolith pour faciliter l'ajout de MCP en phase GA ? | Architecture doit être modulaire. | Winston | Avant le PoC | ⚠️ **À résoudre** : Voir la structure proposée (ex : `services/context_service.py`). |
| OQ-7   | Quel LLM utiliser pour le Generation Engine ? | Impacte coûts, latence, qualité. | Julie/Équipe Dev | Avant le PoC | ⚠️ **À résoudre** : Tester Mistral (local), OpenAI (cloud), ou Llama (open-source). |
| OQ-8   | Comment gérer le cache des données des Providers ? | Optimisation des appels aux APIs. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Utiliser Redis avec des TTL (ex : 1h pour la météo, 24h pour Hub'Eau). |
| OQ-9   | Comment gérer les erreurs des Providers (ex : API indisponible) ? | Fiabilité du Context Engine. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Utiliser les données en cache si disponibles, sinon afficher une alerte (FR-5). |
| OQ-10  | Comment sécuriser les secrets (API keys, tokens) ? | Sécurité des données. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Utiliser des variables d'environnement (`.env`) et `.gitignore`. |

---

### 8.3 Gamification et Récompenses
| **ID** | **Question** | **Impact** | **Propriétaire** | **Échéance** | **Statut** |
|--------|--------------|------------|------------------|--------------|------------|
| OQ-11  | Comment les collectivités vont-elles configurer leurs récompenses ? | Personnalisation pour les clients B2B. | Julie/Équipe Dev | Phase GA | ⚠️ **À résoudre** : Créer un portail administrateur (FR-38 à FR-40). |
| OQ-12  | Comment gérer l'échange de points contre des récompenses ? | Éviter les abus. | Julie/Équipe Dev | Phase GA | ⚠️ **À résoudre** : Utiliser un système de transactions (FR-23). |
| OQ-13  | Comment éviter que les utilisateurs "trichent" pour gagner des points ? | Intégrité du système de gamification. | Julie/Équipe Dev | Phase GA | ⚠️ **À résoudre** : Ajouter une validation manuelle pour les récompenses à haute valeur. |
| OQ-14  | Quelles récompenses proposer par défaut pour le PoC ? | Simulation pour le PoC. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Utiliser des badges virtuels (ex : "Éco-citoyen", "Expert en jardinage"). |

---

### 8.4 RGPD et Sécurité
| **ID** | **Question** | **Impact** | **Propriétaire** | **Échéance** | **Statut** |
|--------|--------------|------------|------------------|--------------|------------|
| OQ-15  | Comment gérer le consentement des utilisateurs pour le traitement des données ? | Conformité RGPD. | Julie/Équipe Dev | Avant le PoC | ⚠️ **À résoudre** : Ajouter une page de consentement à l'inscription. |
| OQ-16  | Quelles données utilisateurs sont considérées comme personnelles ? | Conformité RGPD. | Julie/Équipe Dev | Avant le PoC | ⚠️ **À résoudre** : Anonymiser les données de localisation (ex : stocker la commune, pas l'adresse exacte). |
| OQ-17  | Comment permettre aux utilisateurs de supprimer leurs données ? | Droit à l'oubli (RGPD). | Julie/Équipe Dev | Avant le PoC | ⚠️ **À résoudre** : Ajouter un bouton "Supprimer mon compte" dans le profil utilisateur. |
| OQ-18  | Comment sécuriser les données des Providers (ex : API keys) ? | Sécurité des secrets. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Stocker les API keys dans des variables d'environnement côté backend. |

---

### 8.5 Déploiement et Infrastructure
| **ID** | **Question** | **Impact** | **Propriétaire** | **Échéance** | **Statut** |
|--------|--------------|------------|------------------|--------------|------------|
| OQ-19  | Comment déployer le PoC sur un iMac localement ? | Déploiement local pour les tests. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Utiliser `docker compose up -d` avec les services définis dans le `docker-compose.yml`. |
| OQ-20  | Quelle infrastructure utiliser pour la phase GA ? | Scalabilité et neutralité carbone. | Winston | Phase GA | ⚠️ **À résoudre** : Évaluer AWS, GCP, ou un hébergement vert. |
| OQ-21  | Comment gérer les logs sans exposer de secrets ? | Sécurité des logs. | Julie/Équipe Dev | Avant le PoC | ⚠️ **À résoudre** : Utiliser un filtre de logs pour supprimer les secrets avant écriture. |

---

### 8.6 Tests et Validation
| **ID** | **Question** | **Impact** | **Propriétaire** | **Échéance** | **Statut** |
|--------|--------------|------------|------------------|--------------|------------|
| OQ-22  | Comment tester le Recommendation Engine sans LLM ? | Tests unitaires déterministes. | Amelia | Avant le PoC | ⚠️ **À résoudre** : Écrire des tests avec des données mockées. |
| OQ-23  | Comment valider la pertinence des recommandations ? | Qualité des recommandations. | Julie/Équipe Dev | Phase EA | ⚠️ **À résoudre** : Utiliser le feedback utilisateur (FR-20) et des enquêtes. |

---

## 9. Assumptions Index
*Liste de toutes les hypothèses faites dans ce PRD. Chaque hypothèse doit être validée explicitement avant le développement.*

---

### 9.1 Vision et Objectifs
| **ID** | **Hypothèse** | **Statut** | **Validation Requise** |
|--------|---------------|------------|------------------------|
| A-1    | Les collectivités territoriales seront intéressées par un outil de sensibilisation écologique gamifié. | ⚠️ **À valider** | Confirmer avec des collectivités pilotes. |
| A-2    | Les citoyens seront motivés par un système de points et récompenses pour adopter des bonnes pratiques. | ⚠️ **À valider** | Tester avec des utilisateurs pilotes. |
| A-3    | Le produit sera commercialisé aux collectivités après la phase GA. | ✅ **Validée** | Aligné avec votre objectif. |
| A-4    | Les opportunités positives (ex : balade, jardinage) seront plus engageantes que des messages punitifs. | ✅ **Validée** | Aligné avec le principe "Positive by design". |

---

### 9.2 Target User
| **ID** | **Hypothèse** | **Statut** | **Validation Requise** |
|--------|---------------|------------|------------------------|
| A-5    | Les personas identifiés (Marie, Pierre, Sophie, Jean, Lucie) couvrent tous les cas d’usage principaux. | ⚠️ **À valider** | Affiner avec des interviews utilisateurs. |
| A-6    | Les utilisateurs finaux utiliseront principalement le produit via un navigateur web (pas d’appli native en PoC). | ✅ **Validée** | Aligné avec le scope du PoC. |
| A-7    | Les administrateurs (collectivités) auront besoin d’un portail dédié pour configurer les alertes et récompenses. | ✅ **Validée** | Aligné avec la phase GA. |

---

### 9.3 Glossary
| **ID** | **Hypothèse** | **Statut** | **Validation Requise** |
|--------|---------------|------------|------------------------|
| A-8    | Le Unified Context sera suffisant pour générer des recommandations pertinentes sans données supplémentaires. | ⚠️ **À valider** | Tester avec des données réelles. |
| A-9    | Les Providers (Atmo France, RTE, Hub'Eau, OSM) fourniront des données fiables et à jour pour le PoC. | ✅ **Validée** | Votre guide `API_KEYS_GUIDE.md` confirme que toutes les APIs sont publiques et gratuites. |
| A-10   | Le scoring (25% ContextFit + 20% TimeFit + ...) sera pertinent pour classer les recommandations. | ⚠️ **À valider** | Tester avec des utilisateurs réels. |

---

### 9.4 Features
| **ID** | **Hypothèse** | **Statut** | **Validation Requise** |
|--------|---------------|------------|------------------------|
| A-11   | Le Context Engine pourra normaliser les données de tous les Providers sans perte d’information. | ⚠️ **À valider** | Tester l’intégration avec chaque Provider. |
| A-12   | Le Recommendation Engine générera des recommandations suffisamment pertinentes avec les règles déterministes (sans LLM). | ⚠️ **À valider** | Tester avec des données réelles. |
| A-13   | Les règles du Rule Engine seront suffisantes pour couvrir tous les cas d’usage. | ⚠️ **À valider** | Affiner les règles avec des tests utilisateurs. |
| A-14   | Le système de gamification (points, badges, récompenses) motivera les utilisateurs à suivre les recommandations. | ⚠️ **À valider** | Tester avec des utilisateurs pilotes. |
| A-15   | Les récompenses personnalisables par les collectivités seront faciles à configurer via le portail administrateur. | ⚠️ **À valider** | Tester avec des administrateurs de collectivités. |
| A-16   | Les notifications basiques (email ou in-app) seront suffisantes pour alerter les utilisateurs en phase PoC. | ✅ **Validée** | Aligné avec le scope du PoC. |

---

### 9.5 MVP Scope
| **ID** | **Hypothèse** | **Statut** | **Validation Requise** |
|--------|---------------|------------|------------------------|
| A-17   | Le modular monolith sera suffisant pour le PoC et facile à étendre vers MCP en phase GA. | ⚠️ **À valider** | Valider avec l’architecte (Winston). |
| A-18   | Docker Compose sera suffisant pour déployer le PoC localement sur un iMac. | ✅ **Validée** | Aligné avec le README.md. |
| A-19   | Les tests unitaires et d’intégration seront suffisants pour valider le PoC. | ✅ **Validée** | Aligné avec le README.md. |
| A-20   | Le PoC pourra être développé en 1 mois avec les ressources disponibles (développement le soir). | ⚠️ **À valider** | Évaluer la charge de travail réelle. |

---

### 9.6 Success Metrics
| **ID** | **Hypothèse** | **Statut** | **Validation Requise** |
|--------|---------------|------------|------------------------|
| A-21   | Les métriques de produit (DAU, Retention Rate, etc.) seront mesurables avec les outils disponibles. | ✅ **Validée** | Aligné avec les outils de monitoring. |
| A-22   | La métrique principale (SM-P1) reflétera fidèlement l’utilité perçue par les utilisateurs. | ⚠️ **À valider** | Tester avec des utilisateurs réels. |

---

### 9.7 Open Questions
| **ID** | **Hypothèse** | **Statut** | **Validation Requise** |
|--------|---------------|------------|------------------------|
| A-23   | Les APIs météo, RTE, Hub'Eau, et OSM seront accessibles et gratuites pour le PoC. | ✅ **Validée** | Votre guide `API_KEYS_GUIDE.md` confirme que toutes les APIs sont publiques et gratuites. |
| A-24   | Le LLM choisi sera suffisamment performant pour générer des explications claires. | ⚠️ **À valider** | Tester avec plusieurs LLM. |
| A-25   | Les utilisateurs ne tricheront pas pour gagner des points. | ⚠️ **À valider** | Mettre en place des mécanismes de validation. |

---

### 9.8 Contraintes et Principes
| **ID** | **Hypothèse** | **Statut** | **Validation Requise** |
|--------|---------------|------------|------------------------|
| A-26   | Les librairies open-source existantes seront compatibles avec un produit commercialisé. | ⚠️ **À valider pour GA** | Vérifier les licences (ex : MIT, Apache 2.0) pour la phase GA. |
| A-27   | Le développement le soir uniquement permettra de livrer le PoC en 1 mois. | ⚠️ **À valider** | Évaluer la charge de travail. |
| A-28   | L’optimisation des tokens (caches, bases de données) sera suffisante pour limiter les coûts. | ⚠️ **À valider** | Monitorer les coûts pendant le PoC. |
| A-29   | Le RGPD sera respecté avec les mesures proposées (consentement, anonymisation, droit à l’oubli). | ⚠️ **À valider** | Valider avec un expert juridique. |
