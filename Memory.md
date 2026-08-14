# Memory - Almanach Intelligent Territorial
*Dernière mise à jour : 2026-08-13*

---

## 📌 Décisions Clés
- **Nom du projet** : **Daily Opportunities** (nom officiel, validé via README.md).
- **Public cible** : Collectivités territoriales (mairies, régions) + citoyens.
- **Objectif principal** : Sensibiliser les citoyens et gamifier les bonnes pratiques pour améliorer leur résilience (climat, gestion des ressources).
- **Phases** :
  - **PoC (1 mois)** : Validation du core produit (Context Engine, Recommendation Engine, Web UI, Gamification basique).
  - **EA (Early Access)** : Notifications via applis natives.
  - **GA (General Availability)** : Packaging produit + intégration données partenaires clients + MCP.

---

## ⚙️ Contraintes et Principes

| **Catégorie**          | **Détails**                                                                                     |
|------------------------|---------------------------------------------------------------------------------------------|
| **Légales**            | RGPD (anonymisation, consentement utilisateur).                                              |
| **Délai**              | 1 mois pour le PoC (Proof of Concept).                                                        |
| **Temps de dev**       | Développement uniquement le soir.                                                           |
| **Technologies**       | Utilisation maximale de librairies open-source existantes, compatibles avec un produit commercialisé. |
| **Optimisation**       | Éviter les appels répétés aux LLMs : privilégier les bases de données et les caches (Redis).   |
| **Réduction des coûts**| Limiter l'usage des LLMs en phase de dev et d'exploitation.                                  |
| **Tests**              | Outils de tests intégrés pour permettre une maintenance avec un minimum d'appels aux LLMs. |

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

### Backend
- **Agrégation de données** :
  - Météo (API Météo France, 100 requêtes/jour).
  - Qualité de l'air (API Atmo France, 100 requêtes/minute).
  - Mix énergétique (API RTE Eco2Mix, accès public).
  - Nappes fluviales (API Hub'Eau, 1000 requêtes/jour).
  - Localisation utilisateur (géocodage via Photon ou LocationIQ).
  - Rythmes de vie (stockés dans le User Profile).
  - Phases lunaires (données statiques).
- **Système de gamification** :
  - Points pour les actions écologiques (configurables par les collectivités en phase GA).
  - Récompenses personnalisables (accès à la bibliothèque, événements municipaux, cours MJC).
  - Badges (ex : "Éco-citoyen", "Expert en jardinage").
  - Journal écologique (suivi des actions).
  - Classement (leaderboard) par collectivité.
- **Base de données** :
  - Stockage des données (PostgreSQL + PostGIS) pour éviter les appels répétés aux APIs.
  - Cache (Redis) pour optimiser les performances.

### Frontend
- **Dashboard web** (Next.js) :
  - Recommandations personnalisées (jardinage, bien-être, gestion des ressources).
  - Module de connaissance : journal écologique, saisons, phases lunaires, jardin.
  - Affichage des données agrégées (météo, RTE, qualité de l'air, etc.).
  - Carte interactive (OpenStreetMap) avec points d'intérêt (parcs, marchés, etc.).
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
| **PRD**                   | `/Users/julie/Projects/_bmad-output/planning-artifacts/prds/prd-daily-opportunities-{date}/prd.md` | **En cours**     |
| **API_KEYS_GUIDE.md**     | `/Users/julie/Projects/daily-opportunities/documentation/API_KEYS_GUIDE.md`                | ✅ **Validé**    |
| **README.md**             | `/Users/julie/Projects/daily-opportunities/documentation/README.md`                       | ✅ **Validé**    |
| **Memory.md**             | `/Users/julie/Projects/_bmad-output/Memory.md`                                               | ✅ **Mis à jour** |

---

## 🚀 Prochaines Étapes
1. **Finaliser le PRD** :
    - Valider les sections **Vision**, **Target User**, **Glossary**, **Features**, **Non-Goals**, **MVP Scope**, **Success Metrics**, **Open Questions**, **Assumptions Index**.
    - Résoudre les **hypothèses restantes** (A-8, A-10, A-11, A-12, A-20, A-24, A-27, A-28, A-29).
2. **Explorer la documentation existante** :
    - ✅ **API_KEYS_GUIDE.md** : Validé et intégré.
    - ✅ **README.md** : Validé et intégré.
    - À explorer : Autres fichiers dans `/Users/julie/Projects/daily-opportunities/documentation/`.
3. **Lancer `bmad-architecture`** : Après validation du PRD.
4. **Créer les épopées et stories** avec `bmad-create-epics-and-stories`.

---

## ✅ Résolutions Récentes
- **OQ-1, OQ-2, OQ-3, OQ-4, OQ-19** : Résolues grâce au guide `API_KEYS_GUIDE.md`.
- **A-9, A-23** : Validées grâce au guide `API_KEYS_GUIDE.md`.
- **Nom du projet** : Confirmé comme **Daily Opportunities** (aligné avec le README.md).

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
