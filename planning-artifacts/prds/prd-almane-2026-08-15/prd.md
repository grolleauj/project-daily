---
title: Almanéa
date: 2026-08-15
status: draft
created: 2026-08-13
updated: 2026-08-15
author: Julie
---

# PRD: Almanéa
*L'almanach vivant de votre quotidien*

**Logo** : Arbre stylisé avec feuille verte.
**Slogan** : *"L'almanach vivant de votre quotidien"*
**Description** : *"Des idées, des conseils et des ressources près de chez vous pour profiter, apprendre, réparer et prendre soin de votre environnement."*

---

## 0. Document Purpose
Ce document définit les **exigences produit** pour **Almanéa**, un almanach vivant et personnalisé du quotidien conçu pour aider les **citoyens** et les **collectivités territoriales** à découvrir des opportunités positives basées sur des **données locales et contextuelles**. 

- **Public cible** : Product Managers, développeurs, designers, et parties prenantes des collectivités.
- **Structure** : Ce PRD suit une approche **Vision + Fonctionnalités**, avec des **FR (Functional Requirements)** numérotés globalement, des **User Journeys (UJ)** référencés, et un **Glossary** pour les termes clés.
- **Artifacts associés** : 
  - `README.md` : Architecture globale et détails techniques.
  - `API_KEYS_GUIDE.md` : Guide pour les clés API des Providers.
  - **Almanéa Product Brief** : Document de référence pour la vision produit.

### 0.1 Version History
| **Version** | **Date**       | **Auteur** | **Changements**                                                                                     |
|-------------|----------------|------------|---------------------------------------------------------------------------------------------------|
| 1.0         | 2026-08-13     | Julie      | Création initiale (Daily Opportunities → Almanéa).                                               |
| 1.1         | 2026-08-15     | Julie      | Mise à jour du nom, vision, domaines fonctionnels, contraintes LLM, priorités MVP, UX Tone.       |
| 1.2         | 2026-08-15     | Julie      | Intégration des maquettes ([Almanea1.png](../daily-opportunities/documentation/Almanea1.png), [Almanea2.png](../daily-opportunities/documentation/Almanea2.png)) : branding, navigation, écrans, fonctionnalités clés, badges/niveaux. |

---

## 1. Vision

**Almanéa** est un **almanach vivant et personnalisé du quotidien** qui transforme des **données publiques, locales et contextuelles** (météo, qualité de l'air, eau, énergie, etc.) en **actions, activités et conseils pertinents** pour chaque utilisateur.

Positionnement :
> "Chaque jour a quelque chose à offrir."

Inspiré de **Rustica**, Almanéa est **personnalisé, local, temps réel, interactif, orienté action et positif** : l'objectif est de montrer à l'utilisateur **ce qu'il peut faire aujourd'hui**, pas ce qu'il "doit" faire.

### **Exemples d'opportunités proposées** :
- **Mobilité** : "Une belle journée pour aller au travail à vélo."
- **Nature** : "Une balade nature de 30 min est idéale aujourd'hui."
- **Jardin** : "Pas besoin d'arroser ce soir : de la pluie est prévue."
- **Énergie** : "Le mix électrique est actuellement peu carboné."
- **Réparation** : "Votre vélo ? Profitez de 20 minutes pour apprendre à réparer une crevaison."
- **Réemploi** : "Une ressourcerie près de chez vous accepte les petits appareils."

### **Pourquoi ce produit ?**
Pour les **citoyens**, **Almanéa** offre un outil **utile avant d'être écologique**, compatible avec leur **vie quotidienne** (travail, école, famille). Il permet de **découvrir des activités adaptées à leur contexte** (météo, qualité de l'air, disponibilités) sans alourdir leur routine.

Pour les **collectivités territoriales**, il fournit un **levier de sensibilisation** et d'engagement citoyen, en valorisant les **données locales** (ex : restrictions d'eau, événements) et en encourageant les **bonnes pratiques** via un système de **gamification positive** (points, badges, défis).

### **Différenciateurs**
- **Approche positive first** : Ne jamais culpabiliser l'utilisateur. Formuler des opportunités plutôt que des interdictions.
- **Local first** : Les recommandations dépendent du territoire de l'utilisateur. Utiliser les données locales lorsque disponibles.
- **Context aware** : Météo, qualité de l'air, eau, saison, horaires, localisation, agenda et préférences influencent les recommandations.
- **Actionable** : Chaque recommandation doit pouvoir conduire à une action concrète.
- **Learn & do** : L'application ne se limite pas à recommander. Elle permet également d'apprendre à réparer, entretenir, cultiver, réutiliser et mieux comprendre son environnement.
- **No ecological guilt** : L'écologie est un résultat possible, pas le discours principal.
- **Données agrégées** : Météo (APIs), qualité de l'air (Atmo France), mix énergétique (RTE Eco2Mix), eau (Hub'Eau), géographie (OpenStreetMap), et **rythmes de vie** (saisons, lune, journal écologique).
- **Personnalisation** : Recommandations adaptées au **profil utilisateur** (localisation, disponibilités, préférences) et au **contexte temps réel**.
- **Explicabilité** : Chaque recommandation est accompagnée de **sources scientifiques** et d'une **explication claire**.
- **Technologie** : **LLM-agnostique** (abstraction pour éviter le vendor lock-in).
- **Provider-agnostique** : Les **APIs externes** (Atmo, RTE, Hub'Eau, OSM) sont intégrées via des **adapters standardisés** pour éviter le vendor lock-in. Chaque provider peut être remplacé sans impact sur le reste du système.
- **Cache et bases de données** : Utilisation de **Redis** (cache) et **PostgreSQL** (stockage) pour limiter les coûts et les appels aux APIs.

### **Phases du Projet**
- **PoC (1 mois)** : Validation du core produit (Context Engine, Recommendation Engine, Web UI, Gamification basique).
- **Early Access (EA)** : Notifications via applis natives + fonctionnalités P1 (Explorer, Fiches pratiques, Défis).
- **General Availability (GA)** : Packaging produit + intégration données partenaires clients + MCP (Model Context Protocol) pour l'interopérabilité.

### **Branding**
- **Logo** : Arbre stylisé avec feuille verte.
- **Couleurs principales** : Vert (#2E7D32 ou similaire), blanc cassé, gris clair.
- **Typographie** : Moderne et lisible, titres gras, texte aéré.

### **Mission**
Démocratiser l'accès aux données locales et contextuelles pour que chaque citoyen et chaque collectivité puisse **découvrir des opportunités positives** chaque jour, **sans complexité ni culpabilisation**.

---

## 2. Target User

### 2.1 Jobs To Be Done
Les utilisateurs de **Almanéa** cherchent à :
- Découvrir des **opportunités quotidiennes positives** adaptées à leur contexte (météo, qualité de l'air, énergie, eau, localisation).
- Recevoir des **recommandations personnalisées** sans culpabilisation (approche "positive by design").
- Comprendre **pourquoi une recommandation est proposée** (explications scientifiques, sources fiables).
- **Personnaliser leur expérience** (profil, préférences, disponibilités).
- **Suivre leurs actions** via un **journal écologique** et un **système de gamification** (points, récompenses).
- Être **alertés en temps réel** pour les opportunités temporaires ou urgentes.

**Mapping JTBD → User Journeys** :
| **Job To Be Done**                          | **User Journey**               |
|--------------------------------------------|--------------------------------|
| Découvrir des opportunités positives        | UJ-1, UJ-2                     |
| Recevoir des recommandations personnalisées | UJ-1, UJ-2, UJ-3               |
| Comprendre pourquoi une recommandation est proposée | UJ-1, UJ-2, UJ-3 |
| Personnaliser leur expérience               | UJ-1, UJ-2, UJ-3               |
| Suivre leurs actions                        | UJ-1, UJ-2, UJ-3               |
| Être alertés en temps réel                  | UJ-1, UJ-2, UJ-3, UJ-4         |

### 2.2 Personas
| **Persona**       | **Contexte**                                                                 | **Besoins**                                                                 | **Détails concrets**                                                                 |
|-------------------|-----------------------------------------------------------------------------|-----------------------------------------------------------------------------|------------------------------------------------------------------------------------|
| Marie, 35 ans     | Mère de 2 enfants (5 et 8 ans), emploi du temps chargé (30h/semaine).       | Activités familiales et écologiques adaptées à sa routine.                | Budget : 2 500€/mois. Objectif : 2h/semaine d'activités en plein air avec ses enfants. |
| Pierre, 50 ans    | Retraité, passionné de jardinage, potager de 200m².                        | Optimiser son jardinage (saisons, lune, restrictions d'eau).              | Objectif : Réduire sa consommation d'eau de 30% et augmenter ses récoltes.          |
| Sophie, 28 ans    | Jeune active, télétravail 3j/semaine, budget serré (1 200€/mois).          | Réduire sa facture d'électricité avec des gestes simples.                  | Facture actuelle : 80€/mois. Objectif : Réduire à 60€/mois.                              |
| Jean, 45 ans      | Maire d'une commune de 5 000 habitants.                                      | Sensibiliser ses administrés aux enjeux locaux (eau, air, énergie).        | Budget municipal : 10 000€/an pour les initiatives écologiques.                     |
| Lucie, 22 ans     | Étudiante, colocation (300€/mois), temps libre limité.                       | Trouver des activités gratuites et écoresponsables.                       | Objectif : Participer à 1 activité écoresponsable/semaine.                              |

### 2.3 Key User Journeys
- **UJ-1 : Marie découvre une activité familiale adaptée à la météo**
  - **Persona + contexte** : Marie, 35 ans, mère de 2 enfants, cherche une activité pour occuper ses enfants cet après-midi.
  - **Entry state** : Marie ouvre l'application **Almanéa** sur son téléphone (Web UI responsive). Elle est authentifiée et sa localisation est activée.
  - **Path** :
    1. Marie voit le **dashboard** avec la météo du jour (ensoleillé, 22°C) et la qualité de l'air (bonne).
    2. Elle consulte la section **"Vos opportunités du jour"** et voit une recommandation : *"Balade en forêt avec vos enfants : qualité de l'air excellente et sentiers accessibles à 5 km."*
    3. Elle clique sur la recommandation pour voir les **détails** : durée (2h), bénéfices (santé, découverte de la nature), et **explications scientifiques** (lien vers une étude sur les bienfaits des balades en forêt).
    4. Elle valide la recommandation et **ajoute l'activité à son historique**.
  - **Climax** : Marie reçoit une **notification** 30 minutes avant l'activité pour lui rappeler de préparer les affaires des enfants.
  - **Resolution** : Marie et ses enfants passent un **moment agréable en forêt**. L'application enregistre l'activité dans son **journal écologique** et lui attribue des **points** pour sa participation.
  - **Edge case** : Si la météo change soudainement (ex : pluie), Marie reçoit une **alerte en temps réel** (SLA : <5 minutes) avec une alternative : *"La balade est annulée. Essayez plutôt une activité en intérieur : atelier de jardinage en pot."*
  - **SLA pour les alertes** : Les alertes critiques (ex : qualité de l'air mauvaise) doivent être **livrées en moins de 5 minutes** après détection. Les alertes non critiques (ex : opportunités temporaires) doivent être livrées en **moins de 15 minutes**.

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
  - **Edge case : Recommandations contradictoires** : Si une recommandation de jardinage (ex: "Semer des carottes") entre en conflit avec une **restriction critique** (ex: restriction d'eau), le **Recommendation Engine** priorise la restriction et **filtre la recommandation**. Une alerte est affichée : *"Impossible de semer aujourd'hui en raison des restrictions d'eau. Essayez demain."

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
  - **Entry state** : Jean accède au **portail administrateur** de Almanéa.
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

- **Almanéa** : Almanach vivant et personnalisé du quotidien qui transforme des **données publiques, locales et contextuelles** en **actions, activités, conseils et découvertes pertinents** pour chaque utilisateur.

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

- **Modular Monolith** : Architecture logicielle où tous les composants (Context Engine, Recommendation Engine, etc.) sont **modulaires mais déployés ensemble** (vs. microservices). *Exemple : Comme un couteau suisse—tous les outils sont dans un seul appareil, mais chacun peut être utilisé indépendamment.*

- **Open Data Providers** : Fournisseurs de données ouvertes utilisés par le projet : Atmo France, RTE Eco2Mix, Hub'Eau, OpenStreetMap, Photon (OSM), data.gouv.fr, arXiv (connaissances scientifiques).

- **Récompenses Localisées** : Récompenses **configurées par les collectivités** (ex : accès à la bibliothèque, places pour des événements, cours offerts) que les utilisateurs peuvent obtenir en échange de leurs **points d'impact positif**. Les récompenses évitent les classements culpabilisants.

- **Catalogue de Récompenses** : Liste des **récompenses disponibles** pour les utilisateurs d'une collectivité, avec le **nombre de points requis** pour chacune.

- **Règles de Points Personnalisables** : Règles **définies par les collectivités** pour attribuer des **points d'impact positif** aux actions des utilisateurs (ex : +120 points pour une balade à vélo, +30 points pour une marche).

- **Points d'Impact Positif** : Système de motivation basé sur des points gagnés pour des actions concrètes (ex : vélo, réparation, réemploi). Les points servent à visualiser la progression, débloquer des badges, et participer à des défis.

- **Badges** : Récompenses virtuelles pour accomplissements spécifiques (ex : "Éco-citoyen", "Expert en jardinage", "Maître du réemploi").

- **Défis (Challenges)** : Missions individuelles ou collectives proposées aux utilisateurs (ex : "Réparer plutôt que jeter", "7 jours de mobilité douce").

---

## 4. Features
*Chaque sous-section décrit une fonctionnalité cohérente. Les FR (Functional Requirements) sont numérotés globalement et référencent les User Journeys (UJ) et le Glossary. Les fonctionnalités sont organisées selon les **domaines principaux** : Cultiver, Préserver, Découvrir, Bouger, Réparer, Réemployer, Habiter, Vivre.*

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
[Actor: Context Engine] **MUST fetch data** depuis les **Providers** (Atmo France, RTE, Hub'Eau, OSM, etc.) via leurs **Provider Adapters**.
**Consequences (testable)** :
- Les données sont récupérées **sans erreur** (ex : HTTP 200 pour les APIs).
- Les **secrets** (API keys, tokens) ne sont **jamais exposés** côté frontend.
- **Validation des données** :
  - **Seuils par provider** :
    - **Atmo France (AQI)** : 0 ≤ AQI ≤ 500 (rejeter si > 500).
    - **Météo France (Température)** : -50°C ≤ T ≤ 60°C (rejeter si hors plage).
    - **RTE (Mix énergétique)** : 0 ≤ production ≤ 100 GW (par type de filière).
    - **Hub'Eau (Niveau d'eau)** : 0 ≤ niveau ≤ 1000 cm (rejeter si négatif).
    - **Phases lunaires** : 0 ≤ phase ≤ 29.5 (cycle lunaire moyen).
  - Les **champs critiques** (ex: `provider`, `observedAt`, `value`) sont **validés** (non nuls, format correct).
  - Une **alerte** est générée si une donnée est rejetée (ex: *"Valeur AQI invalide : 1000 (max: 500). Utilisation de la dernière valeur valide en cache."*).
  - **Fallback** : Si une donnée est rejetée, utiliser la **dernière valeur valide en cache** ou une **valeur par défaut** (ex: AQI = 50 pour "moyen").
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
- Les données en cache sont **servies en moins de 500ms** (cible réaliste pour les requêtes complexes).
- Les données **simples** (ex: météo seule) sont servies en **moins de 100ms**.
- Les données expirées sont **rafraîchies automatiquement**.
- **Benchmark** : Valider les performances avec un prototype avant le PoC.
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
[Actor: Context Engine] peut **gérer les erreurs** des Providers (ex : API indisponible, rate limit, timeout).
**Consequences (testable)** :
- En cas d'erreur, le système **utilise des données en cache** si disponibles (TTL étendu temporairement).
- Une **alerte** est générée pour l'administrateur si un Provider critique est indisponible.
- **Comportement pour les pannes partielles** : Si un Provider est indisponible mais que d'autres fonctionnent, le système **dégrade gracieusement** :
  - Les recommandations dépendant du Provider défaillant sont **masquées ou remplacées** par des alternatives.
  - Les données des Providers disponibles sont **toujours utilisées**.
  - Exemple : Si Atmo France est down, les recommandations basées sur la qualité de l'air sont **suspendues**, mais celles basées sur la météo (Météo France) restent actives.
- **Seuils critiques** : Une alerte est déclenchée si >50% des Providers critiques (Atmo, RTE, Hub'Eau) sont indisponibles.
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
- **Gestion des edge cases** :
  - Les **valeurs nulles** pour un composant (ex: `ContextFit=0`) sont **remplacées par une valeur par défaut** (ex: 0.5 pour éviter les biais).
  - Les **poids** (25%, 20%, etc.) sont **validés** pour s'assurer qu'ils totalisent 100%.
  - Une **erreur** est générée si un composant retourne une valeur **hors plage** (ex: `ContextFit > 1`).
- **Exemple de test unitaire** (Python) :
```python
# Test de déterminisme du scoring
def test_scoring_deterministic():
    context = {
        "ContextFit": 0.8,
        "TimeFit": 0.9,
        "UserPreference": 0.7,
        "Accessibility": 0.6,
        "EnvironmentalOpportunity": 0.5,
        "WellbeingOpportunity": 0.4,
        "Novelty": 0.3
    }
    score1 = calculate_score(context)
    score2 = calculate_score(context)  # Même entrée
    assert score1 == score2, "Le score doit être déterministe"

# Test de normalisation
def test_scoring_normalized():
    context = {
        "ContextFit": 1.0,
        "TimeFit": 1.0,
        "UserPreference": 1.0,
        "Accessibility": 1.0,
        "EnvironmentalOpportunity": 1.0,
        "WellbeingOpportunity": 1.0,
        "Novelty": 1.0
    }
    score = calculate_score(context)
    assert 0 <= score <= 1, "Le score doit être normalisé entre 0 et 1"

# Test des valeurs par défaut
def test_scoring_default_values():
    context = {
        "ContextFit": 0,  # Valeur nulle
        "TimeFit": 0.5,
        "UserPreference": 0.5,
        "Accessibility": 0.5,
        "EnvironmentalOpportunity": 0.5,
        "WellbeingOpportunity": 0.5,
        "Novelty": 0.5
    }
    score = calculate_score(context)
    assert score == 0.55, "Le score doit utiliser la valeur par défaut (0.5) pour ContextFit"
```
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-8 : Classement des recommandations
[Actor: Recommendation Engine] peut **classer les recommandations** par score décroissant.
**Consequences (testable)** :
- Les recommandations sont **triées par pertinence**.
- Les recommandations **diverses** (ex : pas que du vélo) sont priorisées.
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-9 : Filtrage des recommandations invalides ou contradictoires
[Actor: Recommendation Engine] peut **supprimer ou ajuster les recommandations invalides ou contradictoires** (ex : conditions météo dangereuses, qualité de l'air incompatible, conflits entre règles).
**Consequences (testable)** :
- Une recommandation est **supprimée** si :
  - Conditions météo dangereuses.
  - Qualité de l'air incompatible.
  - Restriction locale incompatible (ex : restriction d'eau).
  - Données critiques indisponibles.
  - Temps insuffisant.
- **Gestion des contradictions** :
  - Si une recommandation entre en conflit avec une **règle critique** (ex: restriction d'eau vs. conseil de jardinage), la **règle critique prime** et la recommandation est **supprimée ou ajustée**.
  - Exemple : Si une restriction d'eau est active, les recommandations d'arrosage sont **remplacées** par un message : *"Arrosage interdit aujourd'hui. Essayez une activité sans eau."*
  - **Priorité des règles** :
    1. **Sécurité** (ex: qualité de l'air dangereuse).
    2. **Restrictions légales** (ex: restriction d'eau).
    3. **Contraintes techniques** (ex: données manquantes).
    4. **Préférences utilisateur** (ex: centres d'intérêt).
- **Utilisateur sans localisation** : Si l'utilisateur n'a pas activé la géolocalisation ou n'a pas saisi sa localisation, le système **génère des recommandations génériques** basées sur :
  - La **saison actuelle** (ex: "C'est la saison des tomates, essayez de planter !").
  - Le **contexte temporel** (ex: "C'est le moment idéal pour une pause bien-être en intérieur").
  - Les **préférences utilisateur** (ex: "Vous aimez le jardinage ? Voici des conseils généraux pour cette période").
  - Un **message d'encouragement** est affiché : *"Activez la géolocalisation pour des recommandations plus précises !"*
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
- **Interface `LLMProvider`** :
  - **Méthodes** : `generate(prompt: str, context: dict) -> str` (génère une explication textuelle).
  - **Schémas d'entrée/sortie** :
    - `prompt` : Chaîne de caractères (ex: "Explique pourquoi cette recommandation est pertinente").
    - `context` : Dictionnaire avec les données nécessaires (ex: `{user_id: str, recommendation: dict, evidence: list}`).
    - Retour : Explication textuelle (ex: "La qualité de l'air est excellente aujourd'hui...").
  - **Gestion des erreurs** :
    - Si le LLM échoue, retourner un **message d'erreur standardisé** (ex: `LLMError: "Le service LLM est temporairement indisponible"`).
    - Utiliser le **fallback déterministe** (FR-16) si le LLM est indisponible.
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-16 : Fallback déterministe
[Actor: Generation Engine] peut **fournir une explication déterministe minimale** si le LLM est indisponible.
**Consequences (testable)** :
- Exemple : *"Bonne qualité de l'air + météo favorable : une balade de 30 minutes est une bonne option aujourd'hui."*
- L'explication est **toujours disponible**, même sans LLM.
- **Standards de qualité minimaux** :
  - Chaque fallback doit inclure :
    - **Contexte** (ex: "Qualité de l'air excellente").
    - **Preuve scientifique** (si disponible, ex: "Source : Atmo France, AQI=42").
    - **Action recommandée** (ex: "Balade de 30 minutes").
    - **Bénéfice** (ex: "Bonne pour la santé").
  - Les fallbacks sont **testés automatiquement** pour s'assurer qu'ils ne contiennent pas d'erreurs factuelles.
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
Système de **motivation positive** basé sur des **points d'impact positif**, des **badges**, des **défis**, et un **journal écologique** pour encourager les utilisateurs à suivre les recommandations. Les **collectivités** peuvent **configurer leurs propres récompenses locales** (ex : accès à la bibliothèque, places pour des événements, cours offerts) et **définir les règles d'attribution des points**. 

**Principes clés** :
- Éviter les classements culpabilisants (pas de comparaison excessive entre utilisateurs).
- Le LLM n'est **pas utilisé** pour les calculs de points ou les règles de gamification (tout est déterministe).
- Les points servent principalement à visualiser la progression, débloquer des badges, et participer à des défis.

**Functional Requirements** :

---

#### FR-21 : Système de points et suivi des progrès
[Actor: User] peut **gagner des points** en suivant les recommandations et **suivre ses progrès**.
**Consequences (testable)** :
- **Système de points** :
  - Chaque recommandation **réalisée** rapporte des points (ex : +120 points pour une balade à vélo, +30 points pour une marche).
  - Les points sont **affichés en temps réel** dans le profil utilisateur.
  - Les points sont **cumulables** et **non périssables** (sauf configuration spécifique par la collectivité).
- **Suivi des progrès** :
  - Les progrès sont **affichés sous forme de graphiques** (ex : évolution des points sur 30 jours).
  - Les progrès sont **comparables à la moyenne anonyme** de la collectivité (pas de classement culpabilisant).
- **Journal écologique** :
  - Le journal contient : `action`, `date`, `durée`, `points gagnés`, `catégorie` (ex : nature, énergie, bien-être).
  - Le journal est **exportable** (ex : CSV, PDF) et **filtrable** (ex : par catégorie, par date).
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


[Actor: User] peut **débloquer des badges** en accumulant des points ou en réalisant des actions spécifiques.
**Consequences (testable)** :
- Les badges sont **associés à des accomplissements** (ex : "Expert en jardinage" après 10 actions de jardinage, "Éco-citoyen" après 50 recommandations suivies).
- Les badges sont **affichés dans le profil utilisateur**.
- Les badges peuvent être **partagés** (ex : sur les réseaux sociaux ou via un lien public).
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-25 : Mécanismes anti-triche et seuils pour les badges
[Actor: System] peut **détecter et prévenir les abus du système de points** pour garantir l'intégrité de la gamification.
**Consequences (testable)** :
- **Validation manuelle** : Les récompenses à haute valeur (ex: >500 points) nécessitent une **validation manuelle** par un administrateur.
- **Limites de fréquence** : Un utilisateur ne peut pas déclarer la même action plus d'une fois par jour (ex: "J'ai réparé" → 1x/jour max).
- **Vérification contexte** : Les actions déclarées sont **validées par rapport au contexte** (ex: vérifier que l'utilisateur était bien dans la zone géolocalisée pour une balade).
- **Alerte pour comportements suspects** : Détection des schémas anormaux (ex: 20 actions déclarées en 1 heure) et **blocage temporaire** du compte.
- **Journal d'audit** : Toutes les actions déclarées sont **enregistrées avec horodatage et métadonnées** (IP, user agent) pour analyse.
- **Seuils pour les badges** :
  - **Réparateur** (🏆) : 5 actions de réparation.
  - **Explorateur urbain** (🌿) : 10 découvertes locales.
  - **Zéro gaspi niveau 1** (♻️) : 5 actions de réemploi.
  - **Éco-citoyen** (🌍) : 20 actions écoresponsables.
  - **Maître du vélo** (🚲) : 15 trajets à vélo.
  - **Jardinier expert** (🌱) : 10 actions de jardinage.
**Realizes** : UJ-1, UJ-2, UJ-3, UJ-4.

---

#### FR-27 : Suivi des progrès (remplace le Leaderboard)
[Actor: User] peut **voir ses progrès** par rapport à ses propres objectifs et à la moyenne de sa collectivité.
**Consequences (testable)** :
- Les progrès sont **affichés sous forme de graphiques** (ex : évolution des points sur 30 jours).
- Les progrès sont **comparables à la moyenne anonyme** de la collectivité (ex : "Vous êtes au-dessus de la moyenne cette semaine").
- Les progrès peuvent être **filtrés** (ex : par semaine, par mois, par catégorie d'action).
- **Pas de classement culpabilisant** : Aucune comparaison directe entre utilisateurs.
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
- Les alertes sont **envoyées en temps réel** (SLA : <5 minutes).
- **Exigences techniques pour garantir le SLA** :
  - **Architecture asynchrone** : Utiliser une **file de messages** (ex: RabbitMQ, AWS SQS) pour découpler la génération des alertes de leur livraison.
  - **Workers dédiés** : Des **workers séparés** (ex: Celery) pour traiter les alertes en arrière-plan.
  - **Cache des alertes** : Les alertes sont **mises en cache** (Redis) pour éviter les doublons.
  - **Priorisation** : Les alertes critiques (ex: qualité de l'air mauvaise) sont **traitées avant** les alertes non critiques.
  - **Monitoring** : Un **dashboard de monitoring** (ex: Prometheus + Grafana) pour suivre le temps de livraison des alertes.
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
Interface utilisateur **responsive** (Next.js) pour afficher les recommandations, le contexte, et les explications. **Priorité mobile-first** (maquettes validées pour smartphone).

#### Navigation Principale
**Description** :
Barre de navigation en bas de l'écran avec 5 onglets + un bouton central flottant pour déclarer une action.

- **Onglets** :
  1. **Aujourd’hui** (🏠) : Recommandations du jour et contexte local.
  2. **Explorer** (🔍) : Recherche et découverte par catégories.
  3. **+ Action** (➕) : **Bouton central flottant** pour déclarer une action réalisée (ex: "J’ai réparé", "J’ai réemployé").
  4. **Impact** (❤️) : Points, badges, défis, et historique.
  5. **Profil** (👤) : Préférences, favoris, et statistiques.

**Functional Requirements** :

---

#### FR-34 : Dashboard principal (Écran "Aujourd’hui")
[Actor: User] peut **consulter le dashboard principal** avec :
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
  - Contexte (ex: "Pluie prévue cette nuit").
- **Bouton "Voir tout"** : Accès à la liste complète des recommandations.
**Consequences (testable)** :
- Le dashboard est **affiché en moins de 2 secondes** (en mode en ligne).
- Le dashboard est **responsive** (mobile, tablette, desktop).
- Les cartes sont **classées par pertinence** (score du Recommendation Engine).
- **Mode hors ligne** :
  - Le dashboard affiche les **dernières données en cache** (ex: météo, qualité de l'air) avec un **message clair** : *"Mode hors ligne : les données peuvent être obsolètes."*
  - Les **recommandations basées sur le cache** sont marquées comme *"Basé sur des données en cache"*.
  - Les fonctionnalités nécessitant une connexion (ex: mise à jour des points) sont **désactivées** avec un message : *"Connectez-vous pour mettre à jour vos points."*
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-35 : Cartes de recommandations
[Actor: User] peut **voir les détails d'une recommandation** (catégorie, action, durée, raisons, niveau de confiance, bénéfices).
**Consequences (testable)** :
- Chaque carte contient :
  - **Icône** (ex: 🚲 pour vélo, 🔧 pour réparation).
  - **Titre** (ex: "Une belle journée pour sortir à vélo").
  - **Description** (ex: "Itinéraire de 35 min à 1.8 km de chez vous").
  - **Contexte** (ex: "Pluie prévue cette nuit").
- Chaque carte contient un **bouton "Pourquoi cette suggestion ?"** qui affiche les **Evidence** associées.
- Les cartes sont **classées par score** (pertinence).
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-36 : Écran "Explorer"
[Actor: User] peut **explorer les opportunités par catégorie** avec :
- **Barre de recherche** : "Rechercher une idée, un lieu, une fiche..."
- **8 catégories principales** (avec icônes) :
  1. **Cultiver** (🌱) : Jardinage, potager, plantes.
  2. **Réparer** (🔧) : Tutoriels, DIY, entretien.
  3. **Réemployer** (♻️) : Ressourceries, dons, seconde main.
  4. **Bouger** (🚲) : Vélo, marche, mobilité douce.
  5. **Habiter** (🏡) : Énergie, eau, confort domestique.
  6. **Découvrir** (🌳) : Balades, nature, biodiversité.
  7. **Préserver** (💧) : Eau, restrictions, économies.
  8. **Vivre** (❤️) : Bien-être, loisirs, activités familiales.
- **Carte interactive** (OpenStreetMap) :
  - **Points d’intérêt géolocalisés** (ex: Ressourcerie Le Rebond, Atelier vélo participatif, Distribution de compost).
  - **Filtres par catégorie** (ex: afficher uniquement les ressourceries).
  - **Distance** (ex: 1,1 km, 1,4 km, 2,2 km).
  - **Événements à venir** (ex: "Samedi 11 mai • 10h-12h").
- **Bouton "Voir la carte"** : Bascule entre liste et vue carte.
**Consequences (testable)** :
- La carte est **centrée sur la localisation de l'utilisateur**.
- Les points d'intérêt sont **filtrables par catégorie**.
- Les résultats sont **mis à jour en temps réel**.
**Realizes** : UJ-1, UJ-2.

---

#### FR-38 : Écran "Impact"
[Actor: User] peut **consulter son impact positif** avec :
- **Points cumulés** : Total de points (ex: 1 240 points cette semaine).
- **Message d’encouragement** : "Vous avez réalisé 8 actions positives".
- **Répartition par thèmes** (graphique ou liste) :
  - 🚲 **Mobilité** : +420 pts
  - 🔧 **Faire durer** : +280 pts
  - ♻️ **Réemploi** : +210 pts
  - 🌳 **Nature** : +190 pts
  - 💧 **Eau** : +140 pts
  - 🏡 **Habitat** : +100 pts
- **Badges récents** :
  - **Réparateur** (🏆) : Pour les actions de réparation.
  - **Explorateur urbain** (🌿) : Pour les découvertes locales.
  - **Zéro gaspi niveau 1** (♻️) : Pour les actions de réemploi.
- **Bouton "Voir tout"** : Accès à l’historique complet.
**Consequences (testable)** :
- Les points sont **mis à jour en temps réel**.
- Les badges sont **débloqués automatiquement** selon les actions réalisées.
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-38 : Écran "Idée du jour" (Détail d’une recommandation)
[Actor: User] peut **consulter les détails d’une recommandation** avec :
- **Titre** (ex: "Réparer une crevaison de vélo").
- **Catégorie** (ex: Réparer).
- **Métadonnées** :
  - **Difficulté** : Facile / Moyen / Difficile.
  - **Durée** : 20 min.
  - **Nombre d’étapes** : 8 étapes.
- **Description** : "Une crevaison ? Pas de panique ! Suivez le guide pas à pas pour réparer sur la route."
- **Outils nécessaires** :
  - Clés démonte-pneu, pompe, rustines, chiffon.
- **Matériel** :
  - Chambre à air, rustine.
- **Bouton CTA** : "Voir les étapes" (vert, #2E7D32).
**Consequences (testable)** :
- Les détails sont **affichés en moins de 1 seconde**.
- Le bouton CTA redirige vers le **guide étape par étape**.
**Realizes** : UJ-1, UJ-2.

---

#### FR-39 : Fiche pratique (Détail d’un tutoriel)
[Actor: User] peut **consulter une fiche pratique** avec :
- **Titre** (ex: "Recoller et renforcer une chaise en bois").
- **Catégorie** (ex: Maison).
- **Métadonnées** :
  - **Difficulté** : Facile / Moyen / Difficile.
  - **Durée** : 40 min.
  - **Nombre d’étapes** : 6 étapes.
- **Onglets** :
  - **Étapes** : Guide détaillé étape par étape.
  - **Matériel** : Liste des outils et matériaux nécessaires.
  - **Conseils** : Astuces et bonnes pratiques.
- **Description** : "Préparer la chaise : Nettoyez les surfaces à coller et vérifiez la solidité des pièces."
**Consequences (testable)** :
- Les fiches sont **organisées par catégorie** (Réparer, Réemployer, etc.).
- Les onglets permettent une **navigation fluide**.
**Realizes** : UJ-1, UJ-2.

---

#### FR-40 : Écran "Réparer" (Liste des tutoriels)
[Actor: User] peut **consulter la liste des tutoriels de réparation** avec :
- **Filtres** : Tout, Vélo, Maison, Électroménager.
- **Cartes de tutoriels** :
  - "Réparer une crevaison de vélo" (Facile • 20 min).
  - "Changer une chambre à air" (Facile • 15 min).
  - "Réparer une chaise qui bouge" (Facile • 25 min).
  - "Recoudre un vêtement" (Facile • 30 min).
- **Recherche** : Filtrer par mot-clé (ex: "vélo", "chaise").
**Consequences (testable)** :
- Les tutoriels sont **classés par pertinence** (score ou popularité).
- Les filtres sont **appliqués en temps réel**.
**Realizes** : UJ-1, UJ-2.

---

#### FR-41 : Écran "Réemployer" (Liste des opportunités)
[Actor: User] peut **consulter la liste des opportunités de réemploi** avec :
- **Filtres** : Tout, Donner, Récupérer, Ateliers.
- **Cartes d’opportunités** :
  - "Donner un objet" : "Trouvez une association ou une donnerie près de chez vous."
  - "Récupérer près de chez vous" : Liste des objets disponibles (ex: meubles, matériaux, plantes).
  - "Matériauthèques" : "Matériaux de réemploi pour vos projets."
- **Carte interactive** : Localisation des ressourceries, Repair Cafés, etc.
**Consequences (testable)** :
- Les opportunités sont **géolocalisées**.
- Les filtres permettent un **accès rapide** aux ressources.
**Realizes** : UJ-1, UJ-2.

---

#### FR-39 : Écran "Profil"
[Actor: User] peut **consulter et gérer son profil** avec :
- **Header** :
  - Photo de profil (ex: Julie Martin).
  - Statut : "Membre depuis avril 2024".
- **Points cumulés** : 1 240 pts (Total cumulé).
- **Niveau** : Niveau 3 • "Explorateur engagé" (1 240 / 2 000 pts).
- **Favoris** (onglets) :
  - Idées
  - Fiches
  - Lieux
  - Événements
- **Bouton "Voir tout"** : Accès à la liste complète des favoris.
**Consequences (testable)** :
- Le profil est **personnalisable** (photo, préférences).
- Les favoris sont **accessibles rapidement**.
**Realizes** : UJ-1, UJ-2, UJ-3.

---


---

### 4.9 Admin Portal
**Description** :
Portail pour les **administrateurs** (collectivités) afin de configurer des **alertes locales**, gérer les **récompenses**, et consulter des **statistiques**.

**Exigences de sécurité** :
- **Authentification** : Les admins doivent s'authentifier via **email + mot de passe** (PoC) ou **SSO/OAuth2.0** (GA).
- **RBAC (Role-Based Access Control)** :
  - **Rôle Admin** : Peut configurer les alertes, récompenses, et règles de points.
  - **Rôle Super-Admin** : Peut gérer les utilisateurs et les collectivités.
- **Audit Log** : Toutes les actions admin (ex: modification des récompenses) sont **enregistrées** avec horodatage et utilisateur.

**Functional Requirements** :

---

#### FR-41 : Configuration des alertes locales
[Actor: Admin] peut **configurer des alertes locales** (ex : restrictions d'eau, événements spéciaux).
**Consequences (testable)** :
- Les alertes sont **envoyées aux utilisateurs** de la collectivité.
- Les alertes sont **prioritaires** (ex : affichées en haut du dashboard).
**Realizes** : UJ-4.

---

#### FR-42 : Statistiques d'engagement
[Actor: Admin] peut **consulter les statistiques d'engagement** de sa collectivité (ex : nombre de recommandations suivies, points accumulés).
**Consequences (testable)** :
- Les statistiques sont **affichées sous forme de graphiques**.
- Les statistiques sont **exportables** (ex : CSV, PDF).
**Realizes** : UJ-4.

---

#### FR-43 : Gestion des utilisateurs
[Actor: Admin] peut **gérer les utilisateurs** de sa collectivité (ex : ajouter/supprimer des utilisateurs, configurer des groupes).
**Consequences (testable)** :
- Les utilisateurs sont **associés à une collectivité**.
- Les groupes permettent de **cibler des alertes** (ex : seulement pour les habitants d'un quartier).
- **Validation des entrées admin** :
  - Les **règles de points** doivent être des **entiers positifs** (ex: interdire les valeurs négatives ou nulles).
  - Les **récompenses** doivent avoir un **nombre de points requis valide** (ex: >0).
  - Les **alertes locales** doivent avoir une **date de début/fin cohérente** (ex: date de fin > date de début).
  - Les **champs obligatoires** (ex: titre, description) sont **validés avant sauvegarde**.
  - Une **erreur claire** est affichée si une entrée est invalide (ex: *"Le nombre de points doit être supérieur à 0."*).
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

---

### 4.11 Fonctionnalités Clés (Issues des Maquettes)
**Description** :
Fonctionnalités spécifiques identifiées dans les maquettes :
- [Almanea1.png](../daily-opportunities/documentation/Almanea1.png) : Écran "Aujourd'hui", navigation, branding.
- [Almanea2.png](../daily-opportunities/documentation/Almanea2.png) : Écran "Explorer", "Impact", "Profil", fiches pratiques.

**Functional Requirements** :

---

#### FR-42 : Filtres par catégorie
[Actor: User] peut **filtrer les opportunités par catégorie** dans les écrans "Explorer", "Réparer", et "Réemployer".
**Consequences (testable)** :
- **Explorer** : Filtres pour les 8 catégories (Cultiver, Réparer, Réemployer, Bouger, Habiter, Découvrir, Préserver, Vivre).
- **Réparer** : Filtres par type (Tout, Vélo, Maison, Électroménager).
- **Réemployer** : Filtres par type (Tout, Donner, Récupérer, Ateliers).
- Les filtres sont **appliqués en temps réel** (sans rechargement de page).
**Realizes** : UJ-1, UJ-2.

---

#### FR-43 : Carte interactive avec points d’intérêt
[Actor: User] peut **interagir avec une carte** (OpenStreetMap) affichant :
- **Points d’intérêt géolocalisés** :
  - Ressourceries (ex: "Ressourcerie Le Rebond", 1,1 km).
  - Ateliers (ex: "Atelier vélo participatif", 1,4 km).
  - Événements (ex: "Distribution de compost", 2,2 km).
- **Distance depuis l’utilisateur** (ex: 1,1 km, 1,4 km).
- **Horaires** (ex: "Tous les mercredis • 18h-20h").
- **Bouton "Voir la carte"** : Bascule entre vue liste et vue carte.
**Consequences (testable)** :
- La carte est **centrée sur la localisation de l’utilisateur**.
- Les points d’intérêt sont **cliquables** pour afficher plus de détails.
- La carte est **responsive** (mobile et desktop).
**Realizes** : UJ-1, UJ-2.

---

#### FR-44 : Fiches pratiques détaillées
[Actor: User] peut **consulter des fiches pratiques** avec :
- **Structure en onglets** : Étapes, Matériel, Conseils.
- **Métadonnées** :
  - Difficulté (Facile / Moyen / Difficile).
  - Durée (ex: 20 min, 40 min).
  - Nombre d’étapes (ex: 6 étapes, 8 étapes).
- **Contenu** :
  - **Étapes** : Guide détaillé pas à pas.
  - **Matériel** : Liste des outils et matériaux nécessaires.
  - **Conseils** : Astuces et bonnes pratiques.
- **Exemples** :
  - "Réparer une crevaison de vélo" (Facile • 20 min • 8 étapes).
  - "Recoller et renforcer une chaise en bois" (Moyen • 40 min • 6 étapes).
**Consequences (testable)** :
- Les fiches sont **accessibles hors ligne** (cache local).
- Les onglets permettent une **navigation fluide**.
**Realizes** : UJ-1, UJ-2.

---

#### FR-44 : Système de badges et niveaux
[Actor: User] peut **débloquer des badges et monter de niveau** en fonction de ses actions.
**Consequences (testable)** :
- **Badges** :
  - **Réparateur** (🏆) : Pour les actions de réparation.
  - **Explorateur urbain** (🌿) : Pour les découvertes locales.
  - **Zéro gaspi niveau 1** (♻️) : Pour les actions de réemploi.
- **Niveaux** :
  - Exemple : Niveau 3 • "Explorateur engagé" (1 240 / 2 000 pts).
  - Barre de progression visuelle.
- **Affichage** :
  - Dans l’écran "Impact".
  - Dans le profil utilisateur.
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-45 : Bouton "+ Action" (Déclaration d’action)
[Actor: User] peut **déclarer une action réalisée** via le bouton central flottant.
**Consequences (testable)** :
- **Menu contextuel** avec options :
  - J’ai réparé
  - J’ai réemployé
  - J’ai donné
  - J’ai marché
  - J’ai utilisé le vélo
  - J’ai économisé de l’eau
  - J’ai composté
  - J’ai planté
- **Validation** :
  - L’action est **enregistrée dans le journal écologique**.
  - Les **points sont attribués automatiquement**.
  - Une **notification de confirmation** est affichée.
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-46 : Personnalisation du profil
[Actor: User] peut **personnaliser son profil** avec :
- **Photo de profil**.
- **Prénom** (utilisé pour les messages personnalisés).
- **Localisation** (ville ou code postal).
- **Centres d’intérêt** (ex: Jardinage, Réparation, Vélo).
- **Préférences de notification** (types d’alertes, fréquence).
- **Statut** : "Membre depuis [mois/année]".
**Consequences (testable)** :
- Les préférences sont **sauvegardées automatiquement**.
- Les recommandations sont **adaptées aux centres d’intérêt**.
**Realizes** : UJ-1, UJ-2, UJ-3.

---

#### FR-47 : Historique des actions et favoris
[Actor: User] peut **consulter son historique et ses favoris** avec :
- **Historique** :
  - Liste des actions réalisées (ex: "Réparer une crevaison", "Balade à vélo").
  - Date, durée, points gagnés.
  - **Exportable** (CSV, PDF).
- **Favoris** (onglets) :
  - Idées
  - Fiches
  - Lieux
  - Événements
- **Bouton "Voir tout"** : Accès à la liste complète.
**Consequences (testable)** :
- L’historique est **filtrable par date ou catégorie**.
- Les favoris sont **accessibles rapidement**.
**Realizes** : UJ-1, UJ-2, UJ-3.
[Actor: Context Engine] peut **intégrer des données saisonnières** (ex : dates des saisons, températures moyennes).
**Consequences (testable)** :
- Les recommandations de jardinage sont **adaptées à la saison** (ex : "Semer des tomates en été").
- Les données saisonnières sont **mises à jour automatiquement**.
- **Intégration avec le Recommendation Engine** : Les données saisonnières influencent les scores des recommandations (ex: +10% pour les activités adaptées à la saison).
- **Sources de données** : Utiliser des calendriers saisonniers statiques ou des APIs météo (ex: Météo France).
**Realizes** : UJ-2.

---

#### FR-43 : Intégration des phases lunaires
[Actor: Context Engine] peut **intégrer des données sur les phases lunaires** (ex : pleine lune, nouvelle lune).
**Consequences (testable)** :
- Les recommandations de jardinage sont **adaptées aux phases lunaires** (ex : "Semer en lune croissante").
- Les données lunaires sont **mises à jour quotidiennement**.
- **Intégration avec le Recommendation Engine** : Les phases lunaires influencent les recommandations (ex: priorité aux activités de jardinage pendant les phases favorables).
- **Sources de données** : Utiliser des calendriers lunaires statiques (ex: algorithme de calcul des phases lunaires).
**Realizes** : UJ-2.

---

#### FR-44 : Recommandations basées sur les rythmes de vie
[Actor: Recommendation Engine] peut **générer des recommandations** basées sur les **rythmes de vie** de l'utilisateur (ex : horaires de travail, temps libre).
**Consequences (testable)** :
- Les recommandations sont **adaptées aux disponibilités** de l'utilisateur.
- Les rythmes de vie sont **configurables** dans le profil utilisateur.
**Realizes** : UJ-1, UJ-2, UJ-3.

---
--- 

## 10. Non-Goals et Out of Scope
*Ce que **Almanéa** ne fera **pas** en v1 (PoC). Ces éléments pourront être ajoutés dans des versions ultérieures (EA ou GA).*

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

## 11. MVP Scope
*Priorités inspirées du brief Almanéa : P0 (core), P1 (étendues), P2 (avancées).*

### 11.1 In Scope
*Ce qui sera **implémenté** dans le PoC (1 mois). Priorités alignées sur le brief Almanéa.*

#### P0 (Core - PoC)
**Objectif** : Valider le core produit avec un **sous-ensemble minimal** pour respecter le délai d'1 mois.
**Priorités** :
- **Frontend** : Dashboard principal + Vue "Aujourd'hui" + Cartes de recommandations.
- **Backend** : Context Engine + Recommendation Engine (règles de base).
- **Gamification** : Système de points basique (sans récompenses locales).

###### Frontend
- **Web UI responsive** (Next.js/TypeScript) :
  - Dashboard principal (météo, qualité de l'air, mix énergétique, eau, contexte local).
  - Cartes de recommandations (catégorie, action, durée, raisons, bénéfices, points d'impact).
  - Vue "Aujourd'hui" (recommandations du jour + opportunités locales).
  - **Carte interactive basique** (OpenStreetMap) avec points d'intérêt (parcs, pistes vélo).
  - **Journal écologique simplifié** (liste des actions, sans export).
  - **Suivi des progrès basique** (points cumulés, sans graphiques).
  - Profil utilisateur (localisation, préférences de base).

###### Backend
- **API** (FastAPI) avec endpoints pour :
  - Récupération du contexte (`/context/current`, `/context/air`, `/context/energy`, `/context/water`, `/context/local`).
  - Récupération des recommandations (`/recommendations/today`, `/recommendations/{id}`).
  - Gestion des utilisateurs (`/me`, `/me/preferences`, `/me/profile`).
  - Feedback (`/recommendations/{id}/feedback`).
  - Evidence (`/evidence/{id}`).
  - Carte (`/map/opportunities`).
  - Gamification (`/points/history`, `/badges`, `/challenges`).

#### P1 (Étendues - Phase EA)
**Objectif** : Ajouter des fonctionnalités pour enrichir l'expérience utilisateur.

##### Frontend
- Vue "Explorer" (recherche et découverte par catégories : Jardin, Réparer, Réemployer, Bouger, Habiter, Découvrir, Préserver, Vivre).
- Fiches pratiques (ex : "Réparer une crevaison de vélo").
- Vue "Impact" (points, évolution, répartition par thème, badges, défis).
- Vue "Missions" (défis en cours, progression, récompenses).
- Vue "Agenda" (événements, activités, recommandations adaptées aux créneaux).
- Vue "Maison & Énergie" (conseils personnalisés pour l'habitat).

##### Backend
- Intégration des données de **réparation** (tutoriels, fiches pratiques).
- Intégration des données de **réemploi** (ressourceries, Repair Cafés, dons).
- Intégration des données **habitat** (conseils énergie, eau, confort).
- Système de **défis (challenges)** individuels et collectifs.
- **Assistant LLM** pour répondre aux questions utilisateurs (ex : "Quand planter mes tomates ?").

#### Base de Données
- **PostgreSQL + PostGIS** :
  - Stockage des utilisateurs, observations, recommandations, evidence, feedback.
  - Schéma optimisé pour les requêtes du **Recommendation Engine**.
  - Stockage des **fiches pratiques** (réparation, réemploi, jardinage).
  - Stockage des **défis** et **récompenses locales**.

#### Cache
- **Redis** :
  - Mise en cache des données des **Providers** pour éviter les appels répétés.
  - Gestion des **Context Snapshots**.

#### Moteurs
- **Context Engine** :
  - Collecte et normalisation des données des **Providers** (Atmo France, RTE Eco2Mix, Hub'Eau, OpenStreetMap, Météo France, data.gouv.fr).
  - Mise en cache et enrichissement du contexte.
  - **Contrainte** : Le LLM n'est **pas utilisé** dans ce moteur (traitements déterministes uniquement).
- **Recommendation Engine** :
  - Génération de candidats via le **Rule Engine** (règles pour tous les domaines : Cultiver, Préserver, Découvrir, Bouger, Réparer, Réemployer, Habiter, Vivre).
  - Calcul du **score** (formule définie).
  - Classement et filtrage des recommandations.
  - **Contrainte** : Le LLM n'est **pas utilisé** dans ce moteur.
- **Knowledge Engine** :
  - Gestion des **Evidence** (sources scientifiques : ADEME, Santé publique France, OFB, EEA, OMS, PubMed, OpenAlex, arXiv).
- **Generation Engine** :
  - Génération d'explications textuelles via **LLM-agnostique** (`LLMProvider`).
  - Fallback déterministe si LLM indisponible.
  - **Utilisation limitée** : Uniquement pour la compréhension de requêtes naturelles, la génération/adaptation de contenus, et les explications.
- **Explanation Engine** :
  - Combinaison de recommandation, contexte, preuve, et LLM pour des explications claires.

#### Gamification
- **Système de points d'impact positif** :
  - Points attribués pour chaque action concrète (ex : +120 points pour une balade à vélo, +30 points pour une marche, +150 points pour une réparation).
  - Points **personnalisables par les collectivités** (règles configurables via le portail administrateur).
- **Badges** :
  - Badges pour accomplissements (ex : "Expert en jardinage", "Éco-citoyen", "Maître du réemploi", "Répare tout").
- **Journal Écologique** :
  - Enregistrement des actions (balade, jardinage, réparation, réemploi, etc.).
  - Exportable (CSV, PDF).
- **Suivi des progrès** :
  - Graphiques d'évolution des points et actions.
  - Comparaison avec la moyenne anonyme de la collectivité.
- **Défis (Challenges)** :
  - Défis individuels ou collectifs (ex : "Réparer plutôt que jeter", "7 jours de mobilité douce").
  - Défis **configurables par les collectivités**.

#### Notifications
- **Notifications basiques** (email ou in-app) :
  - Alertes pour opportunités temporaires (ex : "Balade à vélo cet après-midi").
  - Alertes pour conditions critiques (ex : "Qualité de l'air mauvaise").

#### Accessibilité
**Exigences WCAG 2.1 AA** :
- **Contraste des couleurs** : Ratio de contraste minimum de 4.5:1 pour le texte.
- **Navigation au clavier** : Toutes les fonctionnalités doivent être **accessibles via clavier** (ex: onglets, boutons).
- **Textes alternatifs** : Toutes les images et icônes doivent avoir un **texte alternatif** (ex: `alt="Icône vélo"`).
- **Taille du texte** : Le texte doit être **redimensionnable jusqu'à 200%** sans perte de fonctionnalité.
- **Messages d'erreur** : Les erreurs doivent être **claires et accessibles** (ex: associées à un champ via `aria-describedby`).
- **Focus visible** : Les éléments interactifs doivent avoir un **focus visible** (ex: bordure ou ombre).
- **Tests** : Utiliser des outils comme **axe-core** ou **Lighthouse** pour valider l'accessibilité.

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
- **Réparation & Réemploi** :
  - Intégration des données locales (ressourceries, Repair Cafés, ateliers participatifs).
- **Habitat** :
  - Conseils personnalisés pour l'énergie, l'eau, et le confort domestique.

#### Documentation
- **README.md** :
  - Description du projet, architecture, déploiement.
- **OpenAPI** :
  - Documentation automatique de l'API.
- **Schémas d'architecture** :
  - Diagrammes Mermaid pour Context Engine, Recommendation Engine, etc.

#### Dependencies
**Frontend** :
- Next.js (v14+)
- TypeScript (v5+)
- React (v18+)
- OpenStreetMap/Leaflet (pour la carte interactive)
- Tailwind CSS ou Material-UI (pour le styling)

**Backend** :
- FastAPI (Python 3.10+)
- PostgreSQL (v15+) + PostGIS (pour les données géospatiales)
- Redis (v7+) (pour le cache)
- Docker + Docker Compose (pour le déploiement local)

**APIs Externes** :
- Atmo France (qualité de l'air)
- RTE Eco2Mix (mix énergétique)
- Hub'Eau (niveaux d'eau)
- Météo France (météo)
- OpenStreetMap/Photon (géocodage)

**Outils** :
- Git (pour le versioning)
- pytest (pour les tests)
- Black/Flake8 (pour le linting)

**Licences** :
- Toutes les dépendances open-source doivent être **compatibles avec une utilisation commerciale** (ex: MIT, Apache 2.0).
- Vérifier les licences avant la phase GA.

---

### 11.2 Out of Scope for MVP
*Ce qui **ne sera pas implémenté** dans le PoC (reporté à EA ou GA). Voir aussi la [Section 10: Non-Goals](#10-non-goals-explicit) pour une liste complète.*

- **Classements culpabilisants** : Éviter toute comparaison directe entre utilisateurs (remplacé par un suivi des progrès personnel).

---

## 12. UX Tone
L'expérience doit être :
- **Positive** : Toujours formuler des opportunités, jamais des reproches.
- **Chaleureuse** : Ton amical et encourageant.
- **Optimiste** : Mettre en avant ce qui est possible, pas ce qui est interdit.
- **Pédagogique** : Expliquer clairement les bénéfices et les étapes.
- **Simple** : Langage accessible, sans jargon technique.
- **Moderne** : Design épuré et intuitif.
- **Locale** : Mettre en avant les ressources et opportunités proches de l'utilisateur.
- **Non moralisatrice** : Éviter toute culpabilisation.

### **Exemples concrets de formulations (issus des maquettes)**
**À éviter** :
- "Vous devez..."
- "Vous polluez..."
- "Mauvais comportement..."
- "Votre score écologique est mauvais."

**À privilégier** :
- **Encouragement** :
  - "Une belle journée pour sortir à vélo."
  - "Une balade nature de 30 min est idéale aujourd'hui."
  - "Pas besoin d'arroser ce soir : de la pluie est prévue."
- **Apprentissage** :
  - "Une crevaison ? **Pas de panique !** Suivez le guide pas à pas pour réparer sur la route."
  - "Apprenez à réparer une crevaison de vélo en 20 min."
- **Découverte** :
  - "Près de chez vous : **Ressourcerie Le Rebond** (1,1 km)."
  - "Atelier vélo participatif à 1,4 km."
  - "Distribution de compost ce samedi (10h-12h)."
- **Action** :
  - "Vous avez réalisé **8 actions positives** cette semaine !"
  - "Votre niveau : **Explorateur engagé** (1 240 / 2 000 pts)."
- **Conseils** :
  - "Nettoyez les surfaces à coller et vérifiez la solidité des pièces."
  - "Trouvez une association ou une donnerie près de chez vous."

### **Ton des messages système**
- **Notifications** :
  - "Balade à vélo cet après-midi : qualité de l'air excellente."
  - "Qualité de l'air mauvaise : évitez les activités extérieures intenses."
- **Feedback** :
  - "Vous avez fait la différence !"
  - "Envie d'essayer ?"
- **Messages d'erreur** :
  - **API indisponible** : *"Désolé, nous n'avons pas pu charger les données météo. Voici une recommandation basée sur les dernières données disponibles."*
  - **Localisation manquante** : *"Activez la géolocalisation pour des recommandations plus précises. En attendant, voici des idées générales."*
  - **Réseau lent** : *"Connexion lente détectée. Les données peuvent être obsolètes."*
  - **Action impossible** : *"Cette action n'est pas disponible dans votre zone. Essayez une autre recommandation."*
- **Messages de confirmation** :
  - "Votre action a été enregistrée ! +120 points."
  - "Votre badge 'Réparateur' a été débloqué !"

### **Règles de rédaction**
1. **Toujours proposer une alternative** : Si une action n'est pas possible (ex: pluie), suggérer une autre activité.
2. **Expliquer le "pourquoi"** : Chaque recommandation doit inclure une raison (ex: "Qualité de l'air excellente").
3. **Utiliser des verbes d'action** : "Découvrez", "Apprenez", "Réparez", "Réemployez".
4. **Personnaliser** : Utiliser le prénom de l'utilisateur (ex: "Bonjour Julie").
5. **Rester concret** : Toujours lier les recommandations à des actions réalisables (ex: "20 min", "1,8 km de chez vous").

---

## 13. Success Metrics

*Les métriques de succès pour **Almanéa** se concentrent sur **l'utilité perçue**, **l'engagement des utilisateurs**, et **la pertinence des recommandations**. Elles évitent les métriques complexes (ex : calcul d'empreinte carbone) au profit de mesures **concrètes et actionnables**.*

---

### 13.1 Product Metrics *(Métriques Produit)*
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

### 13.2 Quality Metrics *(Métriques de Qualité)*
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
  - **Mesure** :
    - **Validation automatique** :
      - **Comparaison avec les données sources** : Vérifier que les explications mentionnent des **valeurs réelles** (ex: si l'explication dit *"la température est de 25°C"*, vérifier que la donnée Atmo France ou Météo France correspond bien à 25°C ± 2°C).
      - **Regex pour motifs suspects** : Utiliser des expressions régulières pour détecter des **valeurs impossibles** (ex: `r"température de [5-9][0-9]°C"` pour >50°C, `r"AQI de [6-9][0-9][0-9]"` pour AQI > 500).
      - **Vérification des entités** : Utiliser une **liste d'entités valides** (ex: noms de parcs, rues, événements) pour détecter les inventions (ex: *"Parc de la Tête d'Or à Lyon"* est valide, *"Parc Imaginaire"* ne l'est pas).
    - **Audit manuel** : Échantillonnage aléatoire de **10% des explications générées**, revu par un humain.
    - **Outils** : Utiliser **Llama Guard** (Meta) ou **NeMo Guardrails** (NVIDIA) pour filtrer les sorties LLM.
  - **Processus de correction** :
    - Les hallucinations détectées sont **signalées** à l'équipe via un **tableau de bord** (ex: Grafana).
    - Les **prompts LLM** sont ajustés pour réduire les risques (ex: ajouter *"Ne pas inventer de données. Utiliser uniquement les informations du contexte fourni."*).
    - Les **fallbacks déterministes** (FR-16) sont utilisés si le taux d'hallucinations dépasse **5%**.
    - **Seuil d'alerte** : Si >10 hallucinations sont détectées en 24h, **désactiver temporairement le LLM** et utiliser uniquement les fallbacks.
  - **Validates** : FR-15 (Abstraction LLM-agnostique), FR-16 (Fallback déterministe).

- **SM-10 : Percentage of Recommendations with Sufficient Evidence**
  - **Définition** : Pourcentage de recommandations qui ont **au moins une Evidence scientifique** associée.
  - **Cible** : 100% en phase EA et GA.
  - **Mesure** : `(Nombre de recommandations avec Evidence / Nombre total de recommandations) * 100`.
  - **Validates** : FR-11 (Gestion des preuves scientifiques), FR-12 (Association des preuves aux recommandations).

---

### 13.3 Principle Metric *(Métrique Principale)*
*La métrique la plus importante, qui résume la valeur du produit pour l'utilisateur.*

- **SM-P1 : "Chaque jour a quelque chose à offrir"**
  - **Définition** : Pourcentage d'utilisateurs qui **déclarent** que **Almanéa** les a aidés à découvrir **au moins une opportunité utile, agréable ou significative** qui s'intégrait naturellement dans leur journée.
  - **Cible** : 70% en phase EA, 85% en phase GA.
  - **Mesure** :
    - **Enquête utilisateur** : Question posée après 1 semaine d'utilisation : *"Almanéa vous a-t-il aidé à découvrir au moins une opportunité utile, agréable ou significative cette semaine ?"* (Réponse : Oui/Non).
    - **Feedback implicite** : Combinaison de SM-3 (Open Rate), SM-4 (Completion Rate), et SM-6 (Relevance).
    - **Validation croisée** : SM-P1 est **validé par** :
      - Un **taux de corrélation > 0.8** avec SM-3 et SM-4 (si les utilisateurs ouvrent et complètent des recommandations, ils devraient aussi répondre positivement à l'enquête).
      - Un **échantillonnage aléatoire** de 10% des réponses à l'enquête, vérifié manuellement.
  - **Validates** : **Toutes les fonctionnalités** (Context Engine, Recommendation Engine, Gamification, etc.).

---

### 13.4 Counter-Metrics *(Métriques à Ne Pas Optimiser)*
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

## 14. Open Questions
*Liste des questions non résolues qui devront être adressées avant ou pendant le développement. Mises à jour avec les décisions du brief Almanéa.*

---

### 14.1 Données et Providers
| **ID** | **Question** | **Impact** | **Propriétaire** | **Échéance** | **Statut** |
|--------|--------------|------------|------------------|--------------|------------|
| OQ-1   | Quelles APIs météo utiliser pour le PoC ? | Choix impacte la qualité des données et les coûts. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Utiliser Météo France (publique, 100 requêtes/jour). |
| OQ-2   | Comment accéder aux données RTE Eco2Mix ? | Nécessaire pour intégrer le mix énergétique. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Utiliser l'API publique sans clé. |
| OQ-3   | Comment gérer les restrictions d'eau (Hub'Eau) ? | Intégration des données locales. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Utiliser l'API publique sans clé (1000 requêtes/jour). |
| OQ-4   | Quelles données astronomiques utiliser pour les phases lunaires ? | Nécessaire pour le jardinage. | Julie/Équipe Dev | Phase EA | ✅ **Résolue** : Utiliser des données statiques (calendrier lunaire). |
| OQ-5   | Comment gérer les données de rythmes de vie (ex : horaires de travail) ? | Personnalisation des recommandations. | Julie/Équipe Dev | Avant le PoC | ⚠️ **À résoudre** : Ajouter un champ `work_schedule` dans le **User Profile** (FR-17). |

---

### 14.2 Architecture et Technique
| **ID** | **Question** | **Impact** | **Propriétaire** | **Échéance** | **Statut** |
|--------|--------------|------------|------------------|--------------|------------|
| OQ-6   | Comment structurer le modular monolith pour faciliter l'ajout de MCP en phase GA ? | Architecture doit être modulaire. | Winston | Avant le PoC | ✅ **Résolue** : **Structure proposée** :
- **Dossier `services/`** :
  - `context_engine/` : Collecte, normalisation, cache des données.
  - `recommendation_engine/` : Génération, scoring, classement des recommandations.
  - `knowledge_engine/` : Gestion des preuves scientifiques.
  - `generation_engine/` : Génération des explications (LLM-agnostique).
  - `user_management/` : Gestion des profils et préférences.
  - `gamification/` : Système de points, badges, défis.
- **Dossier `models/`** : Schémas de base de données (ex: `User`, `Recommendation`, `Evidence`).
- **Dossier `adapters/`** : Adapters pour chaque Provider (ex: `atmo_adapter.py`, `rte_adapter.py`).
- **Fichier `main.py`** : Point d'entrée FastAPI.
- **Diagramme Mermaid** :
```mermaid
graph TD
    A[API] --> B[Context Engine]
    A --> C[Recommendation Engine]
    A --> D[User Management]
    A --> E[Gamification]
    B --> F[Provider Adapters]
    C --> B
    C --> G[Knowledge Engine]
    C --> H[Generation Engine]
    D --> I[PostgreSQL]
    F --> J[Atmo France]
    F --> K[RTE]
    F --> L[Hub'Eau]
``` |
| OQ-7   | Quel LLM utiliser pour le Generation Engine ? | Impacte coûts, latence, qualité. | Julie/Équipe Dev | Avant le PoC | ⚠️ **À résoudre** : Tester Mistral (local), OpenAI (cloud), ou Llama (open-source). |
| OQ-8   | Comment gérer le cache des données des Providers ? | Optimisation des appels aux APIs. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Utiliser Redis avec des TTL (ex : 1h pour la météo, 24h pour Hub'Eau). |
| OQ-9   | Comment gérer les erreurs des Providers (ex : API indisponible) ? | Fiabilité du Context Engine. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Utiliser les données en cache si disponibles, sinon afficher une alerte (FR-5). |
| OQ-10  | Comment sécuriser les secrets (API keys, tokens) ? | Sécurité des données. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Utiliser des variables d'environnement (`.env`) et `.gitignore`. |

---

### 14.3 Gamification et Récompenses
| **ID** | **Question** | **Impact** | **Propriétaire** | **Échéance** | **Statut** |
|--------|--------------|------------|------------------|--------------|------------|
| OQ-11  | Comment les collectivités vont-elles configurer leurs récompenses ? | Personnalisation pour les clients B2B. | Julie/Équipe Dev | Phase GA | ⚠️ **À résoudre** : Créer un portail administrateur (FR-38 à FR-40). |
| OQ-12  | Comment gérer l'échange de points contre des récompenses ? | Éviter les abus. | Julie/Équipe Dev | Phase GA | ⚠️ **À résoudre** : Utiliser un système de transactions (FR-23). |
| OQ-13  | Comment éviter que les utilisateurs "trichent" pour gagner des points ? | Intégrité du système de gamification. | Julie/Équipe Dev | Phase GA | ✅ **Résolue** : Voir **FR-25** (mécanismes anti-triche : validation manuelle, limites de fréquence, vérification contexte, alerte pour comportements suspects). |
| OQ-14  | Quelles récompenses proposer par défaut pour le PoC ? | Simulation pour le PoC. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Utiliser des badges virtuels (ex : "Éco-citoyen", "Expert en jardinage"). |

---

### 14.4 RGPD et Sécurité
| **ID** | **Question** | **Impact** | **Propriétaire** | **Échéance** | **Statut** |
|--------|--------------|------------|------------------|--------------|------------|
| OQ-15  | Comment gérer le consentement des utilisateurs pour le traitement des données ? | Conformité RGPD. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Ajouter une **page de consentement** à l'inscription avec :
- **Opt-in explicite** pour chaque type de donnée (localisation, historique, préférences).
- **Lien vers la politique de confidentialité** (ex: `/privacy-policy`).
- **Possibilité de refuser** le consentement (avec impact clair : ex: "Sans localisation, les recommandations seront génériques"). |
| OQ-16  | Quelles données utilisateurs sont considérées comme personnelles ? | Conformité RGPD. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : **Anonymisation des données** :
- **Localisation** : Stocker uniquement la **commune** (pas l'adresse exacte).
- **Historique** : Agrégation des données (ex: "10 actions en août") sans détails personnels.
- **Identifiants** : Utiliser des **UUID** au lieu d'emails ou noms pour les logs.
- **Données sensibles** : Chiffrement des données (ex: API keys, tokens). |
| OQ-17  | Comment permettre aux utilisateurs de supprimer leurs données ? | Droit à l'oubli (RGPD). | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Ajouter un **bouton "Supprimer mon compte"** dans le profil utilisateur avec :
- **Confirmation en 2 étapes** (ex: "Êtes-vous sûr ?" + "Entrez votre mot de passe").
- **Suppression cascade** : Toutes les données utilisateur (profil, historique, feedback) sont **supprimées irréversiblement**.
- **Délai** : Suppression sous **30 jours** (conformément au RGPD).
- **Notification** : Email de confirmation après suppression. |
| OQ-18  | Comment sécuriser les données des Providers (ex : API keys) ? | Sécurité des secrets. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Stocker les API keys dans des variables d'environnement côté backend. |

---

### 14.5 Déploiement et Infrastructure
| **ID** | **Question** | **Impact** | **Propriétaire** | **Échéance** | **Statut** |
|--------|--------------|------------|------------------|--------------|------------|
| OQ-19  | Comment déployer le PoC sur un iMac localement ? | Déploiement local pour les tests. | Julie/Équipe Dev | Avant le PoC | ✅ **Résolue** : Utiliser `docker compose up -d` avec les services définis dans le `docker-compose.yml`. |
| OQ-20  | Comment intégrer les données de réparation et réemploi (ressourceries, Repair Cafés) ? | Nouveaux domaines du brief Almanéa. | Julie/Équipe Dev | Phase EA | ⚠️ **À résoudre** : Intégrer des APIs ou bases de données locales (ex : data.gouv.fr). |
| OQ-21  | Comment intégrer les données d'habitat (énergie, eau) pour les conseils personnalisés ? | Nouveaux domaines du brief Almanéa. | Julie/Équipe Dev | Phase EA | ⚠️ **À résoudre** : Utiliser les données RTE et Hub'Eau existantes + données locales. |
| OQ-22  | Quelle infrastructure utiliser pour la phase GA ? | Scalabilité et neutralité carbone. | Winston | Phase GA | ⚠️ **À résoudre** : Évaluer AWS, GCP, ou un hébergement vert. |
| OQ-23  | Comment gérer les logs sans exposer de secrets ? | Sécurité des logs. | Julie/Équipe Dev | Avant le PoC | ⚠️ **À résoudre** : Utiliser un filtre de logs pour supprimer les secrets avant écriture. |
| OQ-24  | Comment intégrer les données de réparation et réemploi (ressourceries, Repair Cafés) ? | Nouveaux domaines du brief Almanéa. | Julie/Équipe Dev | Phase EA | ⚠️ **À résoudre** : Intégrer des APIs ou bases de données locales (ex : data.gouv.fr). |
| OQ-25  | Comment intégrer les données d'habitat (énergie, eau) pour les conseils personnalisés ? | Nouveaux domaines du brief Almanéa. | Julie/Équipe Dev | Phase EA | ⚠️ **À résoudre** : Utiliser les données RTE et Hub'Eau existantes + données locales. |

---

### 14.6 Tests et Validation
| **ID** | **Question** | **Impact** | **Propriétaire** | **Échéance** | **Statut** |
|--------|--------------|------------|------------------|--------------|------------|
| OQ-22  | Comment tester le Recommendation Engine sans LLM ? | Tests unitaires déterministes. | Amelia | Avant le PoC | ⚠️ **À résoudre** : Écrire des tests avec des données mockées. |
| OQ-23  | Comment valider la pertinence des recommandations ? | Qualité des recommandations. | Julie/Équipe Dev | Phase EA | ⚠️ **À résoudre** : Utiliser le feedback utilisateur (FR-20) et des enquêtes. |

---

## 15. Assumptions Index
*Liste de toutes les hypothèses faites dans ce PRD. Chaque hypothèse doit être validée explicitement avant le développement. Mises à jour avec les principes du brief Almanéa.*

---

### 15.1 Vision et Objectifs
| **ID** | **Hypothèse** | **Statut** | **Validation Requise** |
|--------|---------------|------------|------------------------|
| A-1    | Les collectivités territoriales seront intéressées par un outil de sensibilisation écologique gamifié. | ⚠️ **À valider** | Confirmer avec des collectivités pilotes. |
| A-2    | Les citoyens seront motivés par un système de points et récompenses pour adopter des bonnes pratiques. | ⚠️ **À valider** | Tester avec des utilisateurs pilotes. |
| A-3    | Le produit sera commercialisé aux collectivités après la phase GA. | ✅ **Validée** | Aligné avec votre objectif. |
| A-4    | Les opportunités positives (ex : balade, jardinage) seront plus engageantes que des messages punitifs. | ✅ **Validée** | Aligné avec le principe "Positive by design". |

---

### 15.2 Target User
| **ID** | **Hypothèse** | **Statut** | **Validation Requise** |
|--------|---------------|------------|------------------------|
| A-5    | Les personas identifiés (Marie, Pierre, Sophie, Jean, Lucie) couvrent tous les cas d’usage principaux. | ⚠️ **À valider** | Affiner avec des interviews utilisateurs. |
| A-6    | Les utilisateurs finaux utiliseront principalement le produit via un navigateur web (pas d’appli native en PoC). | ✅ **Validée** | Aligné avec le scope du PoC. |
| A-7    | Les administrateurs (collectivités) auront besoin d’un portail dédié pour configurer les alertes et récompenses. | ✅ **Validée** | Aligné avec la phase GA. |

---

### 15.3 Glossary
| **ID** | **Hypothèse** | **Statut** | **Validation Requise** |
|--------|---------------|------------|------------------------|
| A-8    | Le Unified Context sera suffisant pour générer des recommandations pertinentes sans données supplémentaires. | ✅ **Résolue** | **Plan de validation** :
- **Test avec données réelles** : Utiliser des données des Providers (Atmo, RTE, Hub'Eau) pour générer des recommandations.
- **Critère de succès** : 95% des recommandations générées doivent être **pertinentes** (évalué via feedback utilisateur ou audit manuel).
- **Cas de test** :
  - Scénario 1 : Données complètes (tous les Providers disponibles) → recommandations précises.
  - Scénario 2 : Données partielles (1 Provider down) → recommandations dégradées mais utiles.
  - Scénario 3 : Données manquantes (ex: pas de localisation) → recommandations génériques.
- **Outils** : Utiliser des **jeux de données mockées** pour simuler différents contextes. |
| A-9    | Les Providers (Atmo France, RTE, Hub'Eau, OSM) fourniront des données fiables et à jour pour le PoC. | ✅ **Validée** | Votre guide `API_KEYS_GUIDE.md` confirme que toutes les APIs sont publiques et gratuites. |
| A-10   | Le scoring (25% ContextFit + 20% TimeFit + ...) sera pertinent pour classer les recommandations. | ⚠️ **À valider** | Tester avec des utilisateurs réels. |

---

### 15.4 Features
| **ID** | **Hypothèse** | **Statut** | **Validation Requise** |
|--------|---------------|------------|------------------------|
| A-11   | Le Context Engine pourra normaliser les données de tous les Providers sans perte d’information. | ⚠️ **À valider** | Tester l’intégration avec chaque Provider. |
| A-12   | Le Recommendation Engine générera des recommandations suffisamment pertinentes avec les règles déterministes (sans LLM). | ⚠️ **À valider** | Tester avec des données réelles. |
| A-13   | Les règles du Rule Engine seront suffisantes pour couvrir tous les cas d’usage. | ⚠️ **À valider** | Affiner les règles avec des tests utilisateurs. |
| A-14   | Le système de gamification (points, badges, récompenses) motivera les utilisateurs à suivre les recommandations. | ⚠️ **À valider** | Tester avec des utilisateurs pilotes. |
| A-15   | Les récompenses personnalisables par les collectivités seront faciles à configurer via le portail administrateur. | ⚠️ **À valider** | Tester avec des administrateurs de collectivités. |
| A-16   | Les notifications basiques (email ou in-app) seront suffisantes pour alerter les utilisateurs en phase PoC. | ✅ **Validée** | Aligné avec le scope du PoC. |

---

### 15.5 MVP Scope
| **ID** | **Hypothèse** | **Statut** | **Validation Requise** |
|--------|---------------|------------|------------------------|
| A-17   | Le modular monolith sera suffisant pour le PoC et facile à étendre vers MCP en phase GA. | ⚠️ **À valider** | Valider avec l’architecte (Winston). |
| A-18   | Docker Compose sera suffisant pour déployer le PoC localement sur un iMac. | ✅ **Validée** | Aligné avec le README.md. |
| A-19   | Les tests unitaires et d’intégration seront suffisants pour valider le PoC. | ✅ **Validée** | Aligné avec le README.md. |
| A-20   | Le PoC pourra être développé en 1 mois avec les ressources disponibles (développement le soir). | ⚠️ **À valider** | Évaluer la charge de travail réelle. |

---

### 15.6 Success Metrics
| **ID** | **Hypothèse** | **Statut** | **Validation Requise** |
|--------|---------------|------------|------------------------|
| A-21   | Les métriques de produit (DAU, Retention Rate, etc.) seront mesurables avec les outils disponibles. | ✅ **Validée** | Aligné avec les outils de monitoring. |
| A-22   | La métrique principale (SM-P1) reflétera fidèlement l’utilité perçue par les utilisateurs. | ⚠️ **À valider** | Tester avec des utilisateurs réels. |

---

### 15.7 Open Questions
| **ID** | **Hypothèse** | **Statut** | **Validation Requise** |
|--------|---------------|------------|------------------------|
| A-23   | Les APIs météo, RTE, Hub'Eau, et OSM seront accessibles et gratuites pour le PoC. | ✅ **Validée** | Votre guide `API_KEYS_GUIDE.md` confirme que toutes les APIs sont publiques et gratuites. |
| A-24   | Le LLM choisi sera suffisamment performant pour générer des explications claires. | ✅ **Validée** | Le LLM est utilisé uniquement pour la génération de contenu (explications, personnalisation rédactionnelle). Les règles, calculs et scores sont déterministes (voir brief Almanéa). |
| A-25   | Les utilisateurs ne tricheront pas pour gagner des points. | ⚠️ **À valider** | Mettre en place des mécanismes de validation manuelle pour les récompenses à haute valeur. |
| A-26   | Les nouveaux domaines (Réparer, Réemployer, Habiter, Vivre) seront pertinents pour les utilisateurs. | ⚠️ **À valider** | Tester avec des utilisateurs pilotes. |

---

### 15.8 Contraintes et Principes
| **ID** | **Hypothèse** | **Statut** | **Validation Requise** |
|--------|---------------|------------|------------------------|
| A-27   | Les librairies open-source existantes seront compatibles avec un produit commercialisé. | ⚠️ **À valider pour GA** | Vérifier les licences (ex : MIT, Apache 2.0) pour la phase GA. |
| A-28   | Le développement le soir uniquement permettra de livrer le PoC en 1 mois. | ✅ **Résolue** | **Plan de mitigation** :
- **Priorisation stricte** : Se concentrer sur le **scope P0 minimal** (Context Engine + Recommendation Engine + UI basique).
- **Répartition des tâches** : Chaque membre de l'équipe se concentre sur un **composant clé** (ex: Julie = Context Engine, Winston = Architecture).
- **Réunions quotidiennes** (15 min) pour synchroniser les progrès et bloquants.
- **Fallback** : Si le PoC n'est pas livré en 1 mois, **reporter les features non-core** (ex: gamification avancée, Admin Portal) à la phase EA.
- **Outils** : Utiliser des **templates de code** et des **librairies existantes** pour accélérer le développement.
- **Tests automatisés** : Écrire des tests unitaires pour éviter les régressions et réduire le temps de debug.
|
| A-29   | L’optimisation des tokens (caches, bases de données) sera suffisante pour limiter les coûts. | ✅ **Validée** | Voir **Annexe A : Brief Almanéa** pour les détails. Les caches (Redis) et bases de données (PostgreSQL) limiteront les coûts. |
| A-30   | Le RGPD sera respecté avec les mesures proposées (consentement, anonymisation, droit à l’oubli). | ⚠️ **À valider** | Valider avec un expert juridique. |
| A-31   | L'approche "Positive First" (pas de culpabilisation) sera bien reçue par les utilisateurs. | ⚠️ **À valider** | Tester avec des utilisateurs pilotes et mesurer SM-P1. |

---

## Annexe A : Brief Almanéa
*Résumé du brief produit utilisé pour les décisions du PRD.*

### 1. Product Vision
**Almanéa** est un **almanach vivant et personnalisé du quotidien** qui transforme des **données publiques, locales et contextuelles** en **actions, activités, conseils et découvertes pertinents** pour chaque utilisateur.

**Positionnement** : *"Chaque jour a quelque chose à offrir."*

### 2. Product Principles
1. **Positive first** : Ne jamais culpabiliser l'utilisateur. Formuler des opportunités plutôt que des interdictions.
2. **Local first** : Les recommandations dépendent du territoire de l'utilisateur.
3. **Context aware** : Météo, qualité de l'air, eau, saison, horaires, localisation, agenda et préférences influencent les recommandations.
4. **Actionable** : Chaque recommandation doit pouvoir conduire à une action concrète.
5. **Learn & do** : L'application permet d'apprendre à réparer, entretenir, cultiver, réutiliser.
6. **No ecological guilt** : L'écologie est un résultat possible, pas le discours principal.

### 3. Domaines Principaux
- 🌱 **Cultiver** : Jardin, potager, plantes, semis.
- 💧 **Préserver** : Eau, restrictions, arrosage, récupération.
- 🌳 **Découvrir** : Balades, nature, biodiversité, parcs.
- 🚲 **Bouger** : Vélo, marche, mobilité douce.
- 🔧 **Réparer** : Tutoriels, DIY, entretien.
- ♻️ **Réemployer** : Ressourceries, Repair Cafés, dons.
- 🏡 **Habiter** : Énergie, eau, confort domestique.
- ❤️ **Vivre** : Bien-être, activités familiales, loisirs.

### 4. Contraintes Techniques
- **LLM** : Utilisé **uniquement pour le contenu** (explications, génération de texte, personnalisation rédactionnelle). **Pas pour les traitements déterministes** (règles, scores, calculs).
- **Provider-agnostique** : Les APIs externes (Atmo, RTE, Hub'Eau) sont intégrées via des **adapters standardisés** pour éviter le vendor lock-in.
- **Cache et bases de données** : Utilisation de **Redis** (cache) et **PostgreSQL** (stockage) pour limiter les coûts et les appels aux APIs.

### 5. Priorités MVP
- **P0** : Context Engine + Recommendation Engine + Web UI basique.
- **P1** : Explorer, Fiches pratiques, Défis, Assistant LLM.
- **P2** : Notifications push, packaging produit, MCP.
