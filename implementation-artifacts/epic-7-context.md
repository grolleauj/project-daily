# Epic 7 Context: Notifications

<!-- Compiled from planning artifacts. Edit freely. Regenerate with compile-epic-context if planning docs change. -->

## Goal

Permettre aux utilisateurs de recevoir des notifications et alertes en temps réel pour les opportunités temporaires (ex: "Balade à vélo cet après-midi : qualité de l'air excellente") et les conditions critiques (ex: "Qualité de l'air mauvaise : évitez les activités extérieures intenses"). Ce système utilise une architecture asynchrone (RabbitMQ + Celery) pour garantir un SLA de livraison inférieur à 5 minutes pour les alertes critiques, tout en permettant une personnalisation fine des préférences de notification par utilisateur.

## Stories

- Story 7.1: Notifications pour les opportunités temporaires
- Story 7.2: Alertes pour les conditions critiques
- Story 7.3: Gérer les préférences de notification

## Requirements & Constraints

- Les notifications doivent être envoyées via email (phase PoC) ou appli native (phase EA), avec une personnalisation basée sur les préférences utilisateur.
- Les alertes critiques doivent être prioritaires, affichées en haut du dashboard, et livrées en temps réel avec un SLA strict de **<5 minutes**.
- Les alertes doivent utiliser une file de messages asynchrone (RabbitMQ) et des workers dédiés (Celery) pour un traitement en arrière-plan.
- Un cache des alertes (Redis) est requis pour éviter les doublons.
- Les alertes critiques doivent être traitées avant les alertes non critiques.
- Les utilisateurs doivent pouvoir configurer leurs préférences de notification (types d'alertes, fréquence, canaux) et les désactiver à tout moment.

## Technical Decisions

- **Architecture asynchrone** : Utilisation de RabbitMQ pour la file de messages et Celery pour les workers dédiés, conformément à AD-8.
- **Cache des alertes** : Redis est utilisé pour éviter les doublons et optimiser les performances, en alignement avec AD-7.
- **Priorisation** : Les alertes critiques sont traitées en priorité par rapport aux notifications non critiques.
- **Intégration avec Context Engine** : Les notifications dépendent des données contextuelles (météo, qualité de l'air, etc.) fournies par E-1, via le contrat `UnifiedContext`.
- **Stack technique** : Python 3.11.8, FastAPI 0.109.0, RabbitMQ 3.12.0, Celery 5.3.4, Redis 7.2.4.

## UX & Interaction Patterns

- Les alertes critiques sont affichées en haut du dashboard pour une visibilité maximale.
- Les notifications sont personnalisées en fonction des préférences utilisateur (ex: types d'alertes, canaux de livraison).
- Les utilisateurs peuvent désactiver les notifications à tout moment via leur profil.

## Cross-Story Dependencies

- **7.1 et 7.2 dépendent de E-1 (Context Engine)** : Les notifications et alertes reposent sur les données contextuelles unifiées (météo, qualité de l'air, etc.).
- **7.2 dépend de AR-2 (RabbitMQ/Celery)** : L'architecture asynchrone est requise pour garantir le SLA de livraison des alertes critiques.
- **7.3 n'a pas de dépendance bloquante** : La gestion des préférences est autonome mais s'intègre avec 7.1 et 7.2 pour la personnalisation.
