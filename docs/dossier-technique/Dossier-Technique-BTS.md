# Dossier Projets - Épreuve E5 : Book-By-Click (BBC)

## 1. Cahier des charges
*   **Organisation support** : Entreprises partenaires souhaitant digitaliser leur prise de rendez-vous.
*   **Problème métier** : Difficulté de gestion des réservations hors des heures d'ouverture et lourdeur administrative du planning manuel.
*   **Utilisateurs** : Clients (utilisateurs finaux) et entreprises partenaires (gestionnaires de services).
*   **Objectifs** : Automatisation de la prise de rendez-vous et centralisation des disponibilités.
*   **Contraintes techniques** : Utilisation d'une stack moderne incluant **React 19**, **Flask**, et **PostgreSQL 15**.
*   **Contraintes d'infrastructure** : Conteneurisation obligatoire via **Docker** et **Docker Compose** pour assurer la portabilité.
*   **Périmètre** : Annuaire des partenaires, affichage des créneaux, système de validation des réservations et authentification.

## 2. Modélisation
*   **Architecture applicative** : Architecture découplée Client/Serveur.
    *   **Frontend** : Développé en **React 19** avec **Vite** comme outil de build et **Tailwind CSS 4** pour le stylage.
    *   **Backend** : API RESTful conçue avec **Flask** (Python).
    *   **Base de données** : Modèle relationnel sous **PostgreSQL 15**.
*   **Modèle de données** : La persistance est gérée par l'ORM **SQLAlchemy**, permettant de structurer les tables (Utilisateurs, Entreprises, Rendez-vous) via des objets Python.
*   **Flux** : Le frontend (port 5173) consomme les ressources du backend (port 5000) via des services web standardisés.

## 3. Description de la solution
*   **Fonctionnalités principales** : Recherche de prestataires, visualisation interactive des créneaux et gestion du profil utilisateur.
*   **Rôle des composants** :
    *   **React Router 7** : Gère la navigation fluide côté client sans rechargement de page.
    *   **Flask API** : Assure la logique métier et la sécurité des accès aux données.
*   **Services Web** : Exposition de points de terminaison (endpoints) pour la synchronisation en temps réel des réservations.

## 4. Preuve de développement professionnel
*   **Gestion de versions** : Utilisation intensive de **Git** avec un historique de **153 commits**.
*   **Collaboration** : Projet multi-contributeur impliquant **5 développeurs**.
*   **Cycle de vie** : Évolution technologique marquée par l'intégration de versions très récentes des frameworks (React 19, Tailwind 4), prouvant une maintenance évolutive rigoureuse.

## 5. Maintenance corrective ou évolutive
*   **Maintenance adaptative** : Développement de scripts de déploiement spécifiques pour chaque système d'exploitation (**install_Linux.sh** et **install_Windows.ps1**).
*   **Qualité du code** : Mise en place d'**ESLint** pour l'analyse statique du code afin de prévenir les bugs de syntaxe et d'assurer l'homogénéité du code source.

## 6. Base de données
*   **Sécurisation** : Utilisation de **SQLAlchemy** pour prévenir les injections SQL et isolation de la base de données PostgreSQL dans un conteneur Docker dédié.
*   **Environnement** : Exploitation de **PostgreSQL 15** pour sa gestion avancée de l'intégrité référentielle.
*   **Sauvegardes** : Simulées via la gestion de volumes persistants dans la configuration **Docker Compose**.

## 7. Tests
*   **Tests de qualité** : Configuration d'un environnement de linting avec **ESLint** pour garantir que le code respecte les standards de production.
*   **Tests d'intégration** : Vérification de l'interopérabilité entre le Frontend (JavaScript) et le Backend (Python) via les scripts de démarrage automatisés.
*   **Recommandations** : Pour une mise en production, le projet suggère l'utilisation de **TypeScript** pour renforcer la fiabilité via le typage statique.

## 8. Documentation technique
*   **Installation rapide** :
    1.  Clonage du dépôt GitHub.
    2.  Frontend : `npm install`.
    3.  Backend : `pip install -r requirements.txt` dans un environnement virtuel Python.
*   **Exécution** : Commandes `npm run dev:lin` (Linux) ou `npm run dev:win` (Windows) pour lancer simultanément les services.
*   **Déploiement** : Support complet pour **Docker**, facilitant l'orchestration des services via un fichier `docker-compose.yml`.

## 9. Documentation utilisateur
*   **Prise en main** : Accès via `http://localhost:5173`.
*   **Interface** : Navigation intuitive pour sélectionner une entreprise et "cliquer" pour réserver (Book-By-Click).
*   **Support** : Une documentation détaillée est hébergée sur **GitHub Pages** pour guider les utilisateurs et les futurs développeurs.

***

**Analogie pour retenir la structure :**
Le projet est comme une **chaîne de montage automobile**. Le **Cahier des charges** est le plan du véhicule. La **Modélisation** est le châssis. Le **Développement (Git)** représente les 153 étapes de montage. Les **Tests (ESLint)** sont le contrôle qualité en fin de ligne, et **Docker** est le conteneur maritime qui permet de livrer la voiture n'importe où dans le monde sans qu'elle ne soit abîmée.