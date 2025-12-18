# Base de données

## Vue d'ensemble

La base de données **PostgreSQL 15** stocke toutes les données du système de réservation Book By Click. Elle comprend 9 tables principales gérant les utilisateurs, les entreprises, les réservations et les événements.

## Configuration Docker

Le fichier `Backend/src/docker-compose.yml` configure PostgreSQL :

```yaml
services:
  db:
    image: postgres:15
    container_name: db
    restart: unless-stopped
    environment:
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppassword
      POSTGRES_DB: appdb
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

**Informations de connexion** :
- Host : `localhost`
- Port : `5432`
- Database : `appdb`
- User : `appuser`
- Password : `apppassword`

## Schéma de la base

### Diagramme ERD

```mermaid
erDiagram
    TYPEUTILISATEUR ||--o{ UTILISATEUR : "définit le rôle"
    UTILISATEUR ||--o{ ENTREPRISE : "gère"
    UTILISATEUR ||--o{ RESERVATION : "effectue"
    ENTREPRISE ||--o{ PRESTATION : "propose"
    ENTREPRISE ||--o{ CRENEAU : "configure"
    ENTREPRISE ||--o{ RESERVATION : "reçoit"
    ENTREPRISE ||--o{ EVENEMENT : "organise"
    ENTREPRISE ||--o{ SEMAINETYPE : "définit"
    PRESTATION ||--o{ RESERVATION : "concerne"
    RESERVATION ||--o{ EVENTEMAIL : "génère"

    TYPEUTILISATEUR {
        int idType PK
        string role
        string description
    }

    UTILISATEUR {
        int idClient PK
        string nom
        string prenom
        date dateNaissance
        string email
        string motDePasseHash
        string telephone
        datetime dateInscription
        int idTypeUtilisateur FK
    }

    ENTREPRISE {
        int idPro PK
        string nomEntreprise
        string nomSecteur
        int idGerant FK
        string slugPublic
        string adresse
        string codePostal
        string ville
        string pays
    }

    PRESTATION {
        int idPrestation PK
        int idPro FK
        string libelle
        int dureeMinutes
        decimal tarif
    }

    CRENEAU {
        int idCreneau PK
        int idPro FK
        datetime dateHeureDebut
        datetime dateHeureFin
        boolean statut
    }

    RESERVATION {
        int idReservation PK
        int idPro FK
        int idClient FK
        int idPrestation FK
        string commentaireClient
        boolean statut
        datetime dateCreation
    }

    EVENEMENT {
        int idEvenement PK
        int idPro FK
        string titre
        string description
        datetime dateDebut
        datetime dateFin
        string typeEvenement
    }

    SEMAINETYPE {
        int idSemaineType PK
        int idPro FK
        string libelle
        string description
        string joursPattern
    }

    EVENTEMAIL {
        int idLog PK
        int idReservation FK
        string email
        datetime dateEnvoi
        boolean statutEnvoi
    }
```

## Tables

### typeutilisateur

Définit les types/rôles d'utilisateurs dans le système (admin, client, professionnel, etc.).

**Modèle SQLAlchemy** : `TypeUtilisateur`

| Colonne | Type | Contraintes | Description |
|---------|------|-------------|-------------|
| idType | INTEGER | PRIMARY KEY | Identifiant unique du type |
| role | VARCHAR | NOT NULL | Nom du rôle (ex: "admin", "client", "pro") |
| description | VARCHAR | NOT NULL | Description du rôle |

**Relations** :
- `utilisateurs` : Liste des utilisateurs de ce type

```sql
CREATE TABLE typeutilisateur (
    idType SERIAL PRIMARY KEY,
    role VARCHAR NOT NULL,
    description VARCHAR NOT NULL
);
```

### utilisateur

Stocke tous les utilisateurs du système (clients et gérants d'entreprise).

**Modèle SQLAlchemy** : `Utilisateur`

| Colonne | Type | Contraintes | Description |
|---------|------|-------------|-------------|
| idClient | INTEGER | PRIMARY KEY | Identifiant unique de l'utilisateur |
| nom | VARCHAR | NOT NULL | Nom de famille |
| prenom | VARCHAR | NOT NULL | Prénom |
| dateNaissance | DATE | NOT NULL | Date de naissance |
| email | VARCHAR | NOT NULL | Adresse email (unique) |
| motDePasseHash | VARCHAR | NOT NULL | Hash du mot de passe (bcrypt) |
| telephone | VARCHAR | NOT NULL | Numéro de téléphone |
| dateInscription | TIMESTAMP | NOT NULL | Date d'inscription |
| idTypeUtilisateur | INTEGER | FK NOT NULL | Référence vers typeutilisateur |

**Relations** :
- `type` : TypeUtilisateur associé
- `entreprises` : Entreprises gérées par cet utilisateur
- `reservations` : Réservations effectuées par cet utilisateur

```sql
CREATE TABLE utilisateur (
    idClient SERIAL PRIMARY KEY,
    nom VARCHAR NOT NULL,
    prenom VARCHAR NOT NULL,
    dateNaissance DATE NOT NULL,
    email VARCHAR NOT NULL UNIQUE,
    motDePasseHash VARCHAR NOT NULL,
    telephone VARCHAR NOT NULL,
    dateInscription TIMESTAMP NOT NULL,
    idTypeUtilisateur INTEGER NOT NULL REFERENCES typeutilisateur(idType)
);

CREATE INDEX idx_utilisateur_email ON utilisateur(email);
CREATE INDEX idx_utilisateur_type ON utilisateur(idTypeUtilisateur);
```

### entreprise

Représente les entreprises qui proposent des services de réservation.

**Modèle SQLAlchemy** : `Entreprise`

| Colonne | Type | Contraintes | Description |
|---------|------|-------------|-------------|
| idPro | INTEGER | PRIMARY KEY | Identifiant unique de l'entreprise |
| nomEntreprise | VARCHAR | NOT NULL | Nom commercial |
| nomSecteur | VARCHAR | NOT NULL | Secteur d'activité |
| idGerant | INTEGER | FK NOT NULL | Référence vers utilisateur (gérant) |
| slugPublic | VARCHAR | NOT NULL | URL slug pour page publique |
| adresse | VARCHAR | NOT NULL | Adresse complète |
| codePostal | VARCHAR | NOT NULL | Code postal |
| ville | VARCHAR | NOT NULL | Ville |
| pays | VARCHAR | NOT NULL | Pays |

**Relations** :
- `gerant` : Utilisateur propriétaire/gérant
- `creneaus` : Créneaux horaires disponibles
- `reservations` : Réservations reçues
- `prestations` : Services proposés
- `evenements` : Événements organisés
- `semainestype` : Modèles de semaines configurés

```sql
CREATE TABLE entreprise (
    idPro SERIAL PRIMARY KEY,
    nomEntreprise VARCHAR NOT NULL,
    nomSecteur VARCHAR NOT NULL,
    idGerant INTEGER NOT NULL REFERENCES utilisateur(idClient),
    slugPublic VARCHAR NOT NULL UNIQUE,
    adresse VARCHAR NOT NULL,
    codePostal VARCHAR NOT NULL,
    ville VARCHAR NOT NULL,
    pays VARCHAR NOT NULL
);

CREATE INDEX idx_entreprise_gerant ON entreprise(idGerant);
CREATE INDEX idx_entreprise_slug ON entreprise(slugPublic);
CREATE INDEX idx_entreprise_ville ON entreprise(ville);
```

### prestation

Définit les services proposés par chaque entreprise.

**Modèle SQLAlchemy** : `Prestation`

| Colonne | Type | Contraintes | Description |
|---------|------|-------------|-------------|
| idPrestation | INTEGER | PRIMARY KEY | Identifiant unique de la prestation |
| idPro | INTEGER | FK NOT NULL | Référence vers entreprise |
| libelle | VARCHAR | NULL | Nom de la prestation |
| dureeMinutes | INTEGER | NOT NULL | Durée en minutes |
| tarif | NUMERIC(10,2) | NULL | Prix en euros |

**Relations** :
- `entreprise` : Entreprise proposant cette prestation
- `reservations` : Réservations pour cette prestation

```sql
CREATE TABLE prestation (
    idPrestation SERIAL PRIMARY KEY,
    idPro INTEGER NOT NULL REFERENCES entreprise(idPro) ON DELETE CASCADE,
    libelle VARCHAR,
    dureeMinutes INTEGER NOT NULL,
    tarif NUMERIC(10,2)
);

CREATE INDEX idx_prestation_pro ON prestation(idPro);
```

### creneau

Gère les plages horaires disponibles pour chaque entreprise.

**Modèle SQLAlchemy** : `Creneau`

| Colonne | Type | Contraintes | Description |
|---------|------|-------------|-------------|
| idCreneau | INTEGER | PRIMARY KEY | Identifiant unique du créneau |
| idPro | INTEGER | FK NOT NULL | Référence vers entreprise |
| dateHeureDebut | TIMESTAMP | NOT NULL | Date et heure de début |
| dateHeureFin | TIMESTAMP | NOT NULL | Date et heure de fin |
| statut | BOOLEAN | NOT NULL | Disponibilité (true=disponible, false=occupé) |

**Relations** :
- `entreprise` : Entreprise propriétaire du créneau

```sql
CREATE TABLE creneau (
    idCreneau SERIAL PRIMARY KEY,
    idPro INTEGER NOT NULL REFERENCES entreprise(idPro) ON DELETE CASCADE,
    dateHeureDebut TIMESTAMP NOT NULL,
    dateHeureFin TIMESTAMP NOT NULL,
    statut BOOLEAN NOT NULL
);

CREATE INDEX idx_creneau_pro ON creneau(idPro);
CREATE INDEX idx_creneau_date ON creneau(dateHeureDebut, dateHeureFin);
CREATE INDEX idx_creneau_statut ON creneau(statut);
```

### reservation

Enregistre les réservations effectuées par les clients.

**Modèle SQLAlchemy** : `Reservation`

| Colonne | Type | Contraintes | Description |
|---------|------|-------------|-------------|
| idReservation | INTEGER | PRIMARY KEY | Identifiant unique de la réservation |
| idPro | INTEGER | FK NOT NULL | Référence vers entreprise |
| idClient | INTEGER | FK NOT NULL | Référence vers utilisateur (client) |
| idPrestation | INTEGER | FK NOT NULL | Référence vers prestation |
| commentaireClient | VARCHAR | NULL | Commentaire optionnel du client |
| statut | BOOLEAN | NOT NULL | État de la réservation (true=confirmée, false=annulée) |
| dateCreation | TIMESTAMP | NOT NULL | Date de création de la réservation |

**Relations** :
- `entreprise` : Entreprise concernée
- `client` : Client ayant effectué la réservation
- `prestation` : Service réservé
- `EventEmails` : Emails envoyés liés à cette réservation

```sql
CREATE TABLE reservation (
    idReservation SERIAL PRIMARY KEY,
    idPro INTEGER NOT NULL REFERENCES entreprise(idPro) ON DELETE CASCADE,
    idClient INTEGER NOT NULL REFERENCES utilisateur(idClient) ON DELETE CASCADE,
    idPrestation INTEGER NOT NULL REFERENCES prestation(idPrestation) ON DELETE CASCADE,
    commentaireClient VARCHAR,
    statut BOOLEAN NOT NULL,
    dateCreation TIMESTAMP NOT NULL
);

CREATE INDEX idx_reservation_pro ON reservation(idPro);
CREATE INDEX idx_reservation_client ON reservation(idClient);
CREATE INDEX idx_reservation_prestation ON reservation(idPrestation);
CREATE INDEX idx_reservation_date ON reservation(dateCreation);
CREATE INDEX idx_reservation_statut ON reservation(statut);
```

### evenement

Stocke les événements spéciaux des entreprises (fermetures, promotions, etc.).

**Modèle SQLAlchemy** : `Evenement`

| Colonne | Type | Contraintes | Description |
|---------|------|-------------|-------------|
| idEvenement | INTEGER | PRIMARY KEY | Identifiant unique de l'événement |
| idPro | INTEGER | FK NOT NULL | Référence vers entreprise |
| titre | VARCHAR | NOT NULL | Titre de l'événement |
| description | VARCHAR | NOT NULL | Description détaillée |
| dateDebut | TIMESTAMP | NOT NULL | Date et heure de début |
| dateFin | TIMESTAMP | NOT NULL | Date et heure de fin |
| typeEvenement | VARCHAR | NOT NULL | Type (ex: "fermeture", "promotion", "vacances") |

**Relations** :
- `entreprise` : Entreprise organisatrice

```sql
CREATE TABLE evenement (
    idEvenement SERIAL PRIMARY KEY,
    idPro INTEGER NOT NULL REFERENCES entreprise(idPro) ON DELETE CASCADE,
    titre VARCHAR NOT NULL,
    description VARCHAR NOT NULL,
    dateDebut TIMESTAMP NOT NULL,
    dateFin TIMESTAMP NOT NULL,
    typeEvenement VARCHAR NOT NULL
);

CREATE INDEX idx_evenement_pro ON evenement(idPro);
CREATE INDEX idx_evenement_dates ON evenement(dateDebut, dateFin);
CREATE INDEX idx_evenement_type ON evenement(typeEvenement);
```

### semainetype

Définit des modèles de semaine réutilisables pour les horaires d'entreprise.

**Modèle SQLAlchemy** : `SemaineType`

| Colonne | Type | Contraintes | Description |
|---------|------|-------------|-------------|
| idSemaineType | INTEGER | PRIMARY KEY | Identifiant unique du modèle |
| idPro | INTEGER | FK NOT NULL | Référence vers entreprise |
| libelle | VARCHAR | NULL | Nom du modèle |
| description | VARCHAR | NULL | Description |
| joursPattern | VARCHAR | NULL | Pattern JSON des jours et horaires |

**Relations** :
- `entreprise` : Entreprise propriétaire du modèle

```sql
CREATE TABLE semainetype (
    idSemaineType SERIAL PRIMARY KEY,
    idPro INTEGER NOT NULL REFERENCES entreprise(idPro) ON DELETE CASCADE,
    libelle VARCHAR,
    description VARCHAR,
    joursPattern VARCHAR
);

CREATE INDEX idx_semainetype_pro ON semainetype(idPro);
```

### eventemail

Journal des emails envoyés (confirmations, rappels, etc.).

**Modèle SQLAlchemy** : `EventEmail`

| Colonne | Type | Contraintes | Description |
|---------|------|-------------|-------------|
| idLog | INTEGER | PRIMARY KEY | Identifiant unique du log |
| idReservation | INTEGER | FK NOT NULL | Référence vers reservation |
| email | VARCHAR | NOT NULL | Adresse email destinataire |
| dateEnvoi | TIMESTAMP | NOT NULL | Date et heure d'envoi |
| statutEnvoi | BOOLEAN | NOT NULL | Succès de l'envoi (true=envoyé, false=échec) |

**Relations** :
- `reservation` : Réservation associée à l'email

```sql
CREATE TABLE eventemail (
    idLog SERIAL PRIMARY KEY,
    idReservation INTEGER NOT NULL REFERENCES reservation(idReservation) ON DELETE CASCADE,
    email VARCHAR NOT NULL,
    dateEnvoi TIMESTAMP NOT NULL,
    statutEnvoi BOOLEAN NOT NULL
);

CREATE INDEX idx_eventemail_reservation ON eventemail(idReservation);
CREATE INDEX idx_eventemail_date ON eventemail(dateEnvoi);
CREATE INDEX idx_eventemail_statut ON eventemail(statutEnvoi);
```

## Requêtes utiles

### Récupérer les créneaux disponibles d'une entreprise

```sql
SELECT c.*
FROM creneau c
WHERE c.idPro = 1
  AND c.statut = TRUE
  AND c.dateHeureDebut >= NOW()
ORDER BY c.dateHeureDebut;
```

### Lister les réservations d'un client avec détails

```sql
SELECT
    r.idReservation,
    r.dateCreation,
    r.statut,
    r.commentaireClient,
    e.nomEntreprise,
    e.ville,
    p.libelle AS prestation,
    p.dureeMinutes,
    p.tarif
FROM reservation r
INNER JOIN entreprise e ON r.idPro = e.idPro
INNER JOIN prestation p ON r.idPrestation = p.idPrestation
WHERE r.idClient = 1
ORDER BY r.dateCreation DESC;
```

### Statistiques d'une entreprise

```sql
SELECT
    e.nomEntreprise,
    COUNT(r.idReservation) AS total_reservations,
    SUM(CASE WHEN r.statut = TRUE THEN 1 ELSE 0 END) AS confirmees,
    SUM(CASE WHEN r.statut = FALSE THEN 1 ELSE 0 END) AS annulees,
    COUNT(DISTINCT r.idClient) AS clients_uniques,
    SUM(p.tarif) AS chiffre_affaires
FROM entreprise e
LEFT JOIN reservation r ON e.idPro = r.idPro
LEFT JOIN prestation p ON r.idPrestation = p.idPrestation
WHERE e.idPro = 1
GROUP BY e.idPro, e.nomEntreprise;
```

### Événements à venir d'une entreprise

```sql
SELECT *
FROM evenement
WHERE idPro = 1
  AND dateFin >= NOW()
ORDER BY dateDebut;
```

### Vérifier les emails envoyés pour une réservation

```sql
SELECT
    ee.email,
    ee.dateEnvoi,
    ee.statutEnvoi,
    r.idReservation,
    r.dateCreation
FROM eventemail ee
INNER JOIN reservation r ON ee.idReservation = r.idReservation
WHERE r.idReservation = 1
ORDER BY ee.dateEnvoi DESC;
```

### Trouver les entreprises par ville

```sql
SELECT
    e.nomEntreprise,
    e.nomSecteur,
    e.adresse,
    e.ville,
    u.nom || ' ' || u.prenom AS gerant,
    u.email AS contact_email
FROM entreprise e
INNER JOIN utilisateur u ON e.idGerant = u.idClient
WHERE e.ville ILIKE '%Paris%'
ORDER BY e.nomEntreprise;
```

## Données initiales

Exemple de données de test pour initialiser la base :

```sql
-- Types d'utilisateurs
INSERT INTO typeutilisateur (role, description) VALUES
('admin', 'Administrateur système avec tous les droits'),
('client', 'Utilisateur client pouvant effectuer des réservations'),
('professionnel', 'Gérant d''entreprise pouvant gérer ses services');

-- Utilisateur admin
INSERT INTO utilisateur (nom, prenom, dateNaissance, email, motDePasseHash, telephone, dateInscription, idTypeUtilisateur) VALUES
('Dupont', 'Jean', '1985-05-15', 'admin@bookbyclick.com', '$2b$12$...', '0601020304', NOW(), 1);

-- Utilisateur professionnel
INSERT INTO utilisateur (nom, prenom, dateNaissance, email, motDePasseHash, telephone, dateInscription, idTypeUtilisateur) VALUES
('Martin', 'Sophie', '1990-08-22', 'sophie.martin@example.com', '$2b$12$...', '0602030405', NOW(), 3);

-- Entreprise exemple
INSERT INTO entreprise (nomEntreprise, nomSecteur, idGerant, slugPublic, adresse, codePostal, ville, pays) VALUES
('Salon Belle Allure', 'Coiffure & Esthétique', 2, 'salon-belle-allure', '45 Rue de la Beauté', '75001', 'Paris', 'France');

-- Prestations
INSERT INTO prestation (idPro, libelle, dureeMinutes, tarif) VALUES
(1, 'Coupe homme', 30, 25.00),
(1, 'Coupe femme', 45, 35.00),
(1, 'Coloration', 90, 65.00);

-- Créneaux disponibles (exemple pour une journée)
INSERT INTO creneau (idPro, dateHeureDebut, dateHeureFin, statut) VALUES
(1, '2025-01-15 09:00:00', '2025-01-15 09:30:00', TRUE),
(1, '2025-01-15 09:30:00', '2025-01-15 10:00:00', TRUE),
(1, '2025-01-15 10:00:00', '2025-01-15 10:30:00', TRUE);
```

## Configuration SQLAlchemy

Le fichier `Backend/src/models.py` définit tous les modèles avec SQLAlchemy :

```python
from extension import db

class TypeUtilisateur(db.Model):
    __tablename__ = "typeutilisateur"
    idType = db.Column(db.Integer, primary_key=True)
    role = db.Column(db.String(), nullable=False)
    description = db.Column(db.String(), nullable=False)
    utilisateurs = db.relationship('Utilisateur', backref='type')

class Utilisateur(db.Model):
    __tablename__ = "utilisateur"
    idClient = db.Column(db.Integer, primary_key=True)
    nom = db.Column(db.String(), nullable=False)
    prenom = db.Column(db.String(), nullable=False)
    dateNaissance = db.Column(db.Date, nullable=False)
    email = db.Column(db.String(), nullable=False)
    motDePasseHash = db.Column(db.String(), nullable=False)
    telephone = db.Column(db.String(), nullable=False)
    dateInscription = db.Column(db.DateTime, nullable=False)
    idTypeUtilisateur = db.Column(db.Integer, db.ForeignKey('typeutilisateur.idType'), nullable=False)

    entreprises = db.relationship('Entreprise', backref='gerant')
    reservations = db.relationship('Reservation', backref='client')

# ... (autres modèles)
```

**Connexion à PostgreSQL** :

```python
# Configuration dans config.py ou .env
SQLALCHEMY_DATABASE_URI = 'postgresql://appuser:apppassword@localhost:5432/appdb'
SQLALCHEMY_TRACK_MODIFICATIONS = False
```

## Sauvegarde et restauration

### Export de la base

```bash
# Export complet
docker exec -t db pg_dump -U appuser appdb > backup_$(date +%Y%m%d_%H%M%S).sql

# Export avec compression
docker exec -t db pg_dump -U appuser appdb | gzip > backup_$(date +%Y%m%d_%H%M%S).sql.gz

# Export structure seulement
docker exec -t db pg_dump -U appuser --schema-only appdb > structure.sql
```

### Import/Restauration

```bash
# Import SQL
docker exec -i db psql -U appuser -d appdb < backup.sql

# Import SQL compressé
gunzip -c backup.sql.gz | docker exec -i db psql -U appuser -d appdb

# Recréer la base avant import
docker exec -i db psql -U appuser -c "DROP DATABASE IF EXISTS appdb;"
docker exec -i db psql -U appuser -c "CREATE DATABASE appdb;"
docker exec -i db psql -U appuser -d appdb < backup.sql
```

## Performance et optimisation

### Analyser les requêtes lentes

```sql
-- Activer le logging des requêtes lentes
ALTER SYSTEM SET log_min_duration_statement = 1000; -- 1 seconde

-- Analyser une requête
EXPLAIN ANALYZE
SELECT * FROM reservation r
INNER JOIN entreprise e ON r.idPro = e.idPro
WHERE r.idClient = 1;
```

### Maintenance régulière

```sql
-- Analyser les tables pour mettre à jour les statistiques
ANALYZE reservation;
ANALYZE entreprise;

-- Nettoyer et optimiser
VACUUM ANALYZE reservation;

-- Reconstruire les index
REINDEX TABLE reservation;
```

### Index additionnels recommandés

```sql
-- Index composés pour les recherches fréquentes
CREATE INDEX idx_reservation_client_date ON reservation(idClient, dateCreation);
CREATE INDEX idx_reservation_pro_statut ON reservation(idPro, statut);

-- Index partiel pour les créneaux disponibles
CREATE INDEX idx_creneau_dispo ON creneau(idPro, dateHeureDebut) WHERE statut = TRUE;

-- Index pour la recherche full-text (si nécessaire)
CREATE INDEX idx_entreprise_nom_gin ON entreprise USING gin(to_tsvector('french', nomEntreprise));
```

## Migration et évolution du schéma

Pour gérer les migrations de schéma, utiliser Alembic avec SQLAlchemy :

```bash
# Installer Alembic
pip install alembic

# Initialiser Alembic
alembic init migrations

# Créer une migration
alembic revision --autogenerate -m "Description de la migration"

# Appliquer les migrations
alembic upgrade head

# Revenir en arrière
alembic downgrade -1
```

## Prochaines étapes

- [Architecture Backend](backend.md)
- [API Endpoints](../api/endpoints.md)
- [Guide de développement](../development/getting-started.md)
