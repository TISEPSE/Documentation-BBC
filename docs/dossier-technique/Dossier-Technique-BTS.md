# ANNEXE 7-1-B : Fiche descriptive de réalisation professionnelle (Option SLAM)

## 1. Informations Générales
*   **Nom, prénom :** [Votre Nom]
*   **N° candidat :** [Votre Numéro]
*   **Épreuve :** E5 - Conception et développement d’applications
*   **Date :** [Date du jour]
*   **Modalité :** Seul(e) ou En équipe

---

## 2. Présentation de la Réalisation
*   **Organisation support :** [Nom de votre organisation ou projet académique]
*   **Intitulé :** **Book-By-Click (BBC)**
*   **Période de réalisation :** [Dates]
*   **Lieu :** [Établissement ou Entreprise]

---

## 3. Compétences Travaillées (Bloc 2 - SLAM)
*   **[X] Concevoir et développer une solution applicative** (Utilisation de frameworks modernes et services Web)
*   **[X] Assurer la maintenance corrective ou évolutive** (Gestion des versions et corrections)
*   **[X] Gérer les données** (Persistance avec PostgreSQL et SQLAlchemy)

---

## 4. Conditions de Réalisation
*   **Ressources fournies :** Cahier des charges du système de réservation en ligne, accès au dépôt GitHub.
*   **Résultats attendus :** Une application fonctionnelle permettant aux clients de prendre rendez-vous avec des entreprises partenaires, stable dans l'environnement de production.

---

## 5. Description des Ressources Utilisées
*   **Ressources documentaires :** Documentation technique BBC (Installation, API, Architecture).
*   **Ressources matérielles :** Poste de travail (PC), Serveur de test/déploiement.
*   **Ressources logicielles :**
    *   **Frontend :** React 19, Vite, Tailwind CSS 4, React Router 7.
    *   **Backend :** Flask (Python), SQLAlchemy (ORM).
    *   **Base de données :** PostgreSQL 15.
    *   **Infrastructure :** Docker, Docker Compose pour la conteneurisation.
    *   **Outils :** Git/GitHub (153 commits), Scripts d'installation (`.sh`, `.ps1`).

---

## 6. Modalités d’Accès aux Productions
*   **Dépôt distant :** [https://github.com/TISEPSE/Book-By-Click.git](https://github.com/TISEPSE/Book-By-Click.git).
*   **Documentation en ligne :** [tisepse.github.io/Documentation-BBC/](https://tisepse.github.io/Documentation-BBC/).
*   **Accès local :**
    *   **Frontend :** `http://localhost:5173`
    *   **Backend (API) :** `http://localhost:5000`

---

## 7. Descriptif de la Réalisation (Verso)

### Architecture Technique
La solution repose sur une architecture découplée. Le **Frontend** utilise **React 19** et **Vite** pour une interface utilisateur réactive. Le **Backend** est une API **Flask** qui communique avec une base de données **PostgreSQL** via l'ORM **SQLAlchemy**, garantissant une gestion structurée des données de réservation.

### Productions Réalisées
1.  **Interface de Réservation :** Composants React pour la prise de rendez-vous et la navigation via React Router 7.
2.  **API REST :** Endpoints pour la gestion des utilisateurs, des entreprises et des créneaux.
3.  **Gestion des Données :** Modélisation relationnelle et gestion de la persistance avec SQLAlchemy.
4.  **Automatisation & Déploiement :** Création de scripts d'installation automatisés (`install_Linux.sh` et `install_Windows.ps1`) et configuration Docker.

### Démarche de Maintenance et Qualité
*   **Gestion des versions :** Suivi rigoureux du développement via GitHub (plus de 150 commits).
*   **Tests :** Validation des endpoints de l'API et de la stabilité du système.
*   **Documentation :** Rédaction d'un guide d'installation complet et d'une référence API pour faciliter l'évolution future du projet.

---

**Note pour l'examen :** Ce dossier doit être accompagné des schémas explicatifs (diagrammes de classes, MCD) et de la documentation utilisateur complète lors de la présentation orale.

**Analogie :** Concevoir ce dossier technique, c'est comme fournir le **manuel d'entretien complet d'un véhicule** : vous décrivez non seulement les pièces utilisées (les technologies), mais aussi comment les assembler (l'installation) et comment s'assurer que le moteur continue de tourner sans panne (la maintenance et les tests).