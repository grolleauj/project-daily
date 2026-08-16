# Memory - Almanéa
*Dernière mise à jour : 2026-08-15*

---

## 📌 Décisions Clés
- **Nom du projet** : **Almanéa** (nom officiel, validé via le brief produit).
- **Slogan** : *"Chaque jour a quelque chose à offrir."*
- **Positionnement** : Almanach vivant et personnalisé du quotidien.
- **Public cible** : 
  - **Primaire** : Adultes souhaitant mieux profiter de leur environnement, découvrir des activités locales, prendre soin de leur maison/jardin, apprendre à réparer, réduire le gaspillage.
  - **Secondaire** : Familles, collectivités territoriales, entreprises, associations/acteurs de l'économie circulaire.
- **Objectif principal** : Transformer des données publiques, locales et contextuelles en **actions, activités, conseils et découvertes pertinents** pour chaque utilisateur, avec une approche **positive first** (pas de culpabilisation).
- **Phases** :
  - **PoC (1 mois)** : Validation du core produit (Context Engine, Recommendation Engine, Web UI, Gamification basique).
  - **EA (Early Access)** : Notifications via applis natives + fonctionnalités P1 (Explorer, Fiches pratiques, Défis).
  - **GA (General Availability)** : Packaging produit + intégration données partenaires clients + MCP (Model Context Protocol) pour l'interopérabilité.

---

## ⚙️ Contraintes et Principes

| **Catégorie**          | **Détails**                                                                                     |
|------------------------|---------------------------------------------------------------------------------------------|
| **Légales**            | RGPD (anonymisation, consentement utilisateur, droit à l'oubli).                          |
| **Délai**              | 1 mois pour le PoC (Proof of Concept).                                                        |
| **Temps de dev**       | Développement uniquement le soir.                                                           |
| **Technologies**       | Utilisation maximale de librairies open-source existantes, compatibles avec un produit commercialisé (licences MIT, Apache 2.0 à valider pour GA). |
| **Optimisation**       | Éviter les appels répétés aux APIs : privilégier les bases de données (PostgreSQL) et les caches (Redis). |
| **LLM**                | **Le LLM n'est pas utilisé pour les traitements déterministes** (règles, scores, calculs, validations). Il est réservé à : la compréhension de requêtes naturelles, la génération/adaptation de contenus, les explications, la personnalisation rédactionnelle, et la synthèse de connaissances. |
| **Réduction des coûts**| Limiter l'usage des LLMs en phase de dev et d'exploitation via des caches agressifs et des bases de données. |
| **Tests**              | Outils de tests intégrés pour permettre une maintenance avec un minimum d'appels aux APIs/LLMs. |
| **Gamification**       | **Approche positive first** : Pas de classement culpabilisant, pas de comparaison excessive entre utilisateurs. Privilégier la progression personnelle et les défis collectifs. |

---

## 🔧 Décisions Techniques (APIs et Providers)
- **APIs pour le PoC** (toutes **publiques et gratuites**) :
  - **Météo** : Météo France (`api.meteofrance.fr`, 100 requêtes/jour) → **Pas de clé nécessaire**.
  - **Qualité de l'air** : Atmo France (`api.atmo-france.org`, 100 requêtes/minute) → **Pas de clé nécessaire**.
  - **Mix énergétique** : RTE Eco2Mix (`api.rte-france.com`) → **Pas de clé nécessaire** (accès public).
  - **Niveaux d'eau** : Hub'Eau (`hubeau.eaufrance.fr`, 1000 requêtes/jour) → **Pas de clé nécessaire**.
  - **Géocodage** : Photon (OSM, `photon.komoot.io`, 10 requêtes/seconde) ou Overpass API (`overpass-api.de`, 2 requêtes/seconde) → **Pas de clé nécessaire**.
  - **Phases lunaires** : Données statiques (calendrier lunaire) → **Pas d'API nécessaire**.

- **APIs pour un usage intensif (optionnel)** :
  - **Météo** : OpenWeatherMap (clé gratuite, 60 appels/minute).
  - **Géocodage** : LocationIQ (clé gratuite, 5000 requêtes/jour).

- **Configuration des clés API** :
  - Fichier `.env` à la racine du projet pour stocker les clés (ex : `OPENWEATHER_API_KEY`).
  - **Ne jamais commiter les clés** dans le dépôt (utiliser `.gitignore`).

- **Fallbacks** :
  - Si une API publique atteint ses limites, utiliser un **cache agressif** (ex : 1h pour la météo, 24h pour Hub'Eau).

---

## 🎯 Fonctionnalités Prioritaires
*Organisées selon les **domaines principaux** inspirés du brief Almanéa : Cultiver, Préserver, Découvrir, Bouger, Réparer, Réemployer, Habiter, Vivre.*

### Backend
- **Agrégation de données** :
  - **Environnement** : Météo (API Météo France, 100 requêtes/jour), qualité de l'air (API Atmo France, 100 requêtes/minute), mix énergétique (API RTE Eco2Mix, accès public), nappes fluviales (API Hub'Eau, 1000 requêtes/jour).
  - **Localisation** : Géocodage via Photon (OSM) ou LocationIQ.
  - **Contexte utilisateur** : Rythmes de vie (stockés dans le User Profile), phases lunaires (données statiques).
  - **Données locales** : data.gouv.fr, données territoriales (collectivités).
  - **Connaissances** : arXiv (pour alimenter les contenus pédagogiques et explications).
- **Système de gamification** :
  - **Points d'impact positif** : +120 pts (vélo), +30 pts (marche), +150 pts (réparation), +100 pts (réemploi), +40 pts (économie d'eau), +80 pts (plantation), +60 pts (compostage).
  - Récompenses personnalisables par les collectivités (accès à la bibliothèque, événements municipaux, cours MJC).
  - Badges (ex : "Éco-citoyen", "Expert en jardinage", "Maître du réemploi", "Répare tout").
  - Journal écologique (suivi des actions).
  - **Suivi des progrès** (remplace le leaderboard) : Graphiques d'évolution, comparaison avec la moyenne anonyme de la collectivité.
  - Défis (challenges) individuels ou collectifs (ex : "Réparer plutôt que jeter", "7 jours de mobilité douce").
- **Base de données** :
  - Stockage des données (PostgreSQL + PostGIS) pour éviter les appels répétés aux APIs.
  - Cache (Redis) pour optimiser les performances (TTL : 1h pour la météo, 24h pour Hub'Eau).

### Frontend
- **Dashboard web** (Next.js) :
  - **Vue "Aujourd'hui"** : Recommandations personnalisées (météo, qualité de l'air, eau, énergie, mobilité, nature, jardinage, réparation, réemploi).
  - **Module de connaissance** : Journal écologique, saisons, phases lunaires, jardin, fiches pratiques (réparation, réemploi).
  - Affichage des données agrégées (météo, RTE, qualité de l'air, etc.).
  - **Carte interactive** (OpenStreetMap) avec points d'intérêt (parcs, marchés, ressourceries, Repair Cafés, événements).
- **Vues supplémentaires (P1 - EA)** :
  - "Explorer" : Recherche et découverte par catégories (Cultiver, Préserver, Découvrir, Bouger, Réparer, Réemployer, Habiter, Vivre).
  - "Impact" : Points, évolution, répartition par thème, badges, défis.
  - "Missions" : Défis en cours, progression, récompenses.
  - "Agenda" : Événements, activités, recommandations adaptées aux créneaux.
  - "Maison & Énergie" : Conseils personnalisés pour l'habitat.
  - "Assistant" : Réponses aux questions utilisateurs (ex : "Quand planter mes tomates ?").
- **Notifications** :
  - Notifications basiques (email ou in-app) pour le PoC.
  - Notifications natives (phase EA).

### Packaging et Configuration (Phase GA)
- **Packaging produit** : Pour vente aux collectivités.
- **Système de configuration** : Intégration de données spécifiques des partenaires clients.
- **MCP (Model Context Protocol)** : Pour l'interopérabilité avec des agents externes (ex : assistants IA des collectivités).

---

## 📂 Artifacts Générés et à Générer

| **Artifact**               | **Chemin**                                                                                     | **Statut**       |
|---------------------------|---------------------------------------------------------------------------------------------|------------------|
| **PRD**                   | `/Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-almane-2026-08-15/prd.md` | ✅ **Mis à jour** (nom : Almanéa) |
| **Almanéa Product Brief** | Document de référence pour la vision produit (intégré dans le PRD).                          | ✅ **Validé**    |
| **API_KEYS_GUIDE.md**     | `/Users/julie/Projects/daily-opportunities/documentation/API_KEYS_GUIDE.md`                | ✅ **Validé**    |
| **README.md**             | `/Users/julie/Projects/daily-opportunities/documentation/README.md`                       | ✅ **Validé**    |
| **Memory.md**             | `/Users/julie/Projects/_bmad-output/Memory.md`                                               | ✅ **Mis à jour** |

---

## 🚀 Prochaines Étapes
1. **Finaliser le PRD** :
    - ✅ **Nom du projet** : Mis à jour vers **Almanéa**.
    - ✅ **Vision et positionnement** : Alignés sur le brief (approche positive first, local first, context aware, actionable, learn & do, no ecological guilt).
    - ✅ **Domaines fonctionnels** : Ajoutés (Réparer, Réemployer, Habiter, Vivre).
    - ✅ **Contraintes techniques** : LLM déterministe uniquement, pas de leaderboard culpabilisant.
    - ✅ **Priorités MVP** : Ajoutées (P0, P1, P2).
    - ⚠️ **À valider** : Résoudre les **hypothèses restantes** (A-8, A-10, A-11, A-12, A-20, A-25, A-27, A-28, A-30, A-31).
2. **Explorer la documentation existante** :
    - ✅ **API_KEYS_GUIDE.md** : Validé et intégré.
    - ✅ **README.md** : Validé et intégré.
    - ✅ **Almanéa Product Brief** : Intégré dans le PRD.
    - À explorer : Autres fichiers dans `/Users/julie/Projects/daily-opportunities/documentation/`.
3. **Lancer `bmad-architecture`** : Après validation finale du PRD.
4. **Créer les épopées et stories** avec `bmad-create-epics-and-stories`.

---

## ✅ Résolutions Récentes
- **OQ-1, OQ-2, OQ-3, OQ-4, OQ-8, OQ-9, OQ-10, OQ-14, OQ-18, OQ-19** : Résolues grâce au guide `API_KEYS_GUIDE.md`.
- **A-3, A-4, A-6, A-7, A-9, A-16, A-18, A-19, A-23, A-24, A-29** : Validées (alignées avec le brief Almanéa ou les contraintes techniques).
- **Nom du projet** : Mis à jour vers **Almanéa** (aligné avec le brief produit).
- **Approche LLM** : Validée (utilisation limitée au contenu, pas pour les traitements déterministes).
- **Gamification** : Validée (approche positive first, pas de leaderboard culpabilisant).

---

## 🔄 Historique des Sessions
- **2026-08-13** :
    - Brain Dump initial.
    - Calibration des enjeux (délai, RGPD, optimisation).
    - Choix du mode d'entrée pour le PRD : **Vision + Fonctionnalités** (validé).
    - Création de `Memory.md`.
    - Exploration de `/Users/julie/Projects/daily-opportunities/documentation/` :
      - ✅ **README.md** : Intégré dans le PRD (Vision, Architecture, MVP Scope).
      - ✅ **API_KEYS_GUIDE.md** : Résolu les OQ-1 à OQ-4 et validé A-9, A-23.
    - Mise à jour de `Memory.md` avec les décisions techniques (APIs, Providers).
    - **Création du PRD** : `/Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-daily-opportunities-2026-08-13/prd.md` (sections 1 à 9).
    - **Finalisation du PRD en cours** : Étapes 1 à 8 du workflow `bmad-prd` (Memlog Audit, Input Reconciliation, Reviewer Gate, etc.).
- **2026-08-15** :
    - **Changement de nom** : Passage de **Daily Opportunities** à **Almanéa** (aligné avec le brief produit).
    - **Mise à jour du PRD** :
      - ✅ Nom, vision, positionnement, différenciateurs.
      - ✅ Ajout des domaines (Réparer, Réemployer, Habiter, Vivre).
      - ✅ Contraintes techniques (LLM déterministe uniquement).
      - ✅ Priorités MVP (P0, P1, P2).
      - ✅ Gamification (points d'impact positif, pas de leaderboard culpabilisant).
    - **Mise à jour de Memory.md** : Alignement avec le brief Almanéa.
    - **Renommage du fichier PRD** : `prd-daily-opportunities-2026-08-13` → `prd-almane-2026-08-15`.
