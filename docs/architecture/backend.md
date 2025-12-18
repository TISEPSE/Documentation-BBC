# Architecture Backend

## Vue d'ensemble

Le backend est une API REST construite avec **Flask** (Python) qui gère la logique métier et la communication avec la base de données **PostgreSQL 15** via **SQLAlchemy ORM**.

## Stack Technique

- **Framework**: Flask
- **ORM**: SQLAlchemy (Flask-SQLAlchemy)
- **Base de données**: PostgreSQL 15
- **CORS**: Flask-CORS
- **Email**: SMTP via conteneur Docker
- **Conteneurisation**: Docker Compose (DB + Mail)

## Structure du projet

```text
Backend/
└── src/
    ├── app.py                # Point d'entrée avec factory pattern
    ├── models.py             # Modèles SQLAlchemy
    ├── pages.py              # Blueprint avec les routes
    ├── extension.py          # Initialisation db et cors
    ├── mailer.py             # Gestion des emails
    ├── seed.py               # Données de test
    ├── docker-compose.yml    # PostgreSQL + Serveur Mail
    ├── data/                 # Fichiers JSON
    │   ├── services.json
    │   └── communes-france-avec-polygon-2025.json
    └── mail/                 # Configuration serveur mail
```

## Point d'entrée : app.py

Utilise le **factory pattern** pour créer l'application Flask avec une configuration flexible.

```python
from flask import Flask
from pages import pages_blueprint
import os
from extension import db, cors

def create_app():
    app = Flask(__name__)

    # Configuration de la base de données
    app.config["SQLALCHEMY_DATABASE_URI"] = os.getenv(
        "SQLALCHEMY_DATABASE_URI",
        "postgresql://appuser:apppassword@localhost:5432/appdb"
    )
    app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

    # Enregistrement du blueprint
    app.register_blueprint(pages_blueprint)

    # Initialisation des extensions
    db.init_app(app)
    cors(app)

    # Création des tables et seed de données
    try:
        with app.app_context():
            db.create_all()
            import seed
    except Exception as e:
        print(f"Avertissement: Impossible de se connecter à la base de données: {e}")

    return app

if __name__ == "__main__":
    app = create_app()
    app.run(debug=True, port=5000)
```

## Extensions : extension.py

Initialise les extensions Flask utilisées dans l'application.

```python
from flask_sqlalchemy import SQLAlchemy
from flask_cors import CORS

db = SQLAlchemy()
cors = CORS
```

## Configuration

### Variables d'environnement

```env
SQLALCHEMY_DATABASE_URI=postgresql://appuser:apppassword@localhost:5432/appdb
```

### Connexion à la base de données

- **SGBD**: PostgreSQL 15
- **URL**: `postgresql://appuser:apppassword@localhost:5432/appdb`
- **Port**: 5432
- **Utilisateur**: appuser
- **Mot de passe**: apppassword
- **Base**: appdb

## Modèles de données : models.py

Définit tous les modèles SQLAlchemy représentant les tables de la base de données.

### Modèles disponibles

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
    idTypeUtilisateur = db.Column(db.Integer, db.ForeignKey('typeutilisateur.idType'))

    entreprises = db.relationship('Entreprise', backref='gerant')
    reservations = db.relationship('Reservation', backref='client')

class Entreprise(db.Model):
    __tablename__ = "entreprise"
    idPro = db.Column(db.Integer, primary_key=True)
    nomEntreprise = db.Column(db.String(), nullable=False)
    nomSecteur = db.Column(db.String(), nullable=False)
    idGerant = db.Column(db.Integer, db.ForeignKey('utilisateur.idClient'))
    slugPublic = db.Column(db.String(), nullable=False)
    adresse = db.Column(db.String(), nullable=False)
    codePostal = db.Column(db.String(), nullable=False)
    ville = db.Column(db.String(), nullable=False)
    pays = db.Column(db.String(), nullable=False)

    creneaus = db.relationship('Creneau', backref='entreprise')
    reservations = db.relationship('Reservation', backref='entreprise')
    prestations = db.relationship('Prestation', backref='entreprise')
    evenements = db.relationship('Evenement', backref='entreprise')
    semainestype = db.relationship('SemaineType', backref='entreprise')

class Prestation(db.Model):
    __tablename__ = "prestation"
    idPrestation = db.Column(db.Integer, primary_key=True)
    idPro = db.Column(db.Integer, db.ForeignKey('entreprise.idPro'))
    libelle = db.Column(db.String)
    dureeMinutes = db.Column(db.Integer, nullable=False)
    tarif = db.Column(db.Numeric(10,2))
    reservations = db.relationship('Reservation', backref="prestation")

class Creneau(db.Model):
    __tablename__ = "creneau"
    idCreneau = db.Column(db.Integer, primary_key=True)
    idPro = db.Column(db.Integer, db.ForeignKey('entreprise.idPro'))
    dateHeureDebut = db.Column(db.DateTime, nullable=False)
    dateHeureFin = db.Column(db.DateTime, nullable=False)
    statut = db.Column(db.Boolean, nullable=False)

class Reservation(db.Model):
    __tablename__ = "reservation"
    idReservation = db.Column(db.Integer, primary_key=True)
    idPro = db.Column(db.Integer, db.ForeignKey('entreprise.idPro'))
    idClient = db.Column(db.Integer, db.ForeignKey('utilisateur.idClient'))
    idPrestation = db.Column(db.Integer, db.ForeignKey('prestation.idPrestation'))
    commentaireClient = db.Column(db.String())
    statut = db.Column(db.Boolean, nullable=False)
    dateCreation = db.Column(db.DateTime, nullable=False)
    EventEmails = db.relationship('EventEmail', backref='reservation')

class Evenement(db.Model):
    __tablename__ = "evenement"
    idEvenement = db.Column(db.Integer, primary_key=True)
    idPro = db.Column(db.Integer, db.ForeignKey('entreprise.idPro'))
    titre = db.Column(db.String(), nullable=False)
    description = db.Column(db.String(), nullable=False)
    dateDebut = db.Column(db.DateTime, nullable=False)
    dateFin = db.Column(db.DateTime, nullable=False)
    typeEvenement = db.Column(db.String(), nullable=False)

class SemaineType(db.Model):
    __tablename__ = "semainetype"
    idSemaineType = db.Column(db.Integer, primary_key=True)
    idPro = db.Column(db.Integer, db.ForeignKey('entreprise.idPro'))
    libelle = db.Column(db.String())
    description = db.Column(db.String())
    joursPattern = db.Column(db.String())

class EventEmail(db.Model):
    __tablename__ = "eventemail"
    idLog = db.Column(db.Integer, primary_key=True)
    idReservation = db.Column(db.Integer, db.ForeignKey('reservation.idReservation'))
    email = db.Column(db.String(), nullable=False)
    dateEnvoi = db.Column(db.DateTime, nullable=False)
    statutEnvoi = db.Column(db.Boolean, nullable=False)
```

## Routes API : pages.py

Le blueprint `pages_blueprint` contient toutes les routes de l'API.

### Routes disponibles

```python
from flask import Blueprint, request, jsonify
from extension import cors, db
from models import Utilisateur, Entreprise, Reservation
from mailer import send_contact_email

pages_blueprint = Blueprint("pages", __name__)

# Inscription
@pages_blueprint.route("/register_form", methods=["POST"])
def register_form():
    email = request.form.get("email")
    password = request.form.get("password")
    nom = request.form.get("nom")
    # Logique d'inscription
    return "OK"

# Connexion
@pages_blueprint.route("/login_form", methods=["POST"])
def login_form():
    email = request.form.get("email")
    password = request.form.get("password")
    return jsonify({"email": email, "password": password})

# Test récupération utilisateur
@pages_blueprint.route("/teste", methods=["POST"])
def recap():
    username = request.form.get("username")
    user = Utilisateur.query.filter(Utilisateur.nom == username).first()
    return jsonify({
        "id": user.idClient,
        "nom": user.nom,
        "prenom": user.prenom,
        "email": user.email
    })

# Formulaire de contact
@pages_blueprint.route("/contact", methods=["POST"])
def contact():
    data = request.json
    name = data.get("name")
    email = data.get("email")
    phone = data.get("phone")
    message = data.get("message")

    success = send_contact_email(name, email, phone, message)

    if success:
        return jsonify({"success": True}), 200
    else:
        return jsonify({"success": False}), 500

# Autocomplétion services
@pages_blueprint.route("/api/services", methods=["GET"])
def get_services():
    # Charge services.json et retourne les services filtrés
    query = request.args.get('q', '').lower()
    # ... logique de filtrage
    return jsonify(services)

# Autocomplétion villes
@pages_blueprint.route("/api/villes", methods=["GET"])
def get_villes():
    # Charge communes-france-avec-polygon-2025.json
    query = request.args.get('q', '').lower()
    # ... logique de filtrage
    return jsonify(villes[:15])
```

## Gestion des emails : mailer.py

Envoie les emails via le serveur SMTP conteneurisé.

```python
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

def send_contact_email(name, email, phone, message):
    try:
        msg = MIMEMultipart()
        msg['From'] = 'Book By Click <noreply@bookbyclick.com>'
        msg['To'] = 'delacroixarthur016@gmail.com'
        msg['Subject'] = f'Nouveau message de contact - {name}'

        body = f"""
Vous avez reçu un nouveau message via le formulaire de contact :

Nom: {name}
Email: {email}
Téléphone: {phone}

Message:
{message}

---
Vous pouvez répondre directement à : {email}
"""
        msg.attach(MIMEText(body, 'plain'))

        # Connexion au serveur SMTP local (conteneur mail)
        server = smtplib.SMTP('localhost', 25, timeout=10)
        server.send_message(msg)
        server.quit()

        return True
    except Exception as e:
        print(f"Erreur lors de l'envoi de l'email: {e}")
        return False
```

## Données de test : seed.py

Script pour initialiser la base avec des données de test.

```python
from app import create_app
from models import db, Utilisateur, TypeUtilisateur, Entreprise
from datetime import datetime, date

app = create_app()

with app.app_context():
    db.drop_all()
    db.create_all()

    # Types d'utilisateur
    admin = TypeUtilisateur(role="admin", description="Administrateur du système")
    client = TypeUtilisateur(role="client", description="Client de base")
    db.session.add_all([admin, client])
    db.session.commit()

    # Utilisateurs
    u1 = Utilisateur(
        nom="Dupont",
        prenom="Jean",
        dateNaissance=date(1990, 5, 20),
        email="jean.dupont@example.com",
        motDePasseHash="hash123",
        telephone="0601020304",
        dateInscription=datetime.now(),
        idTypeUtilisateur=admin.idType
    )
    db.session.add(u1)
    db.session.commit()

    # Entreprise
    e1 = Entreprise(
        nomEntreprise="Salon Beauté Zen",
        nomSecteur="Bien-être",
        idGerant=u1.idClient,
        slugPublic="beaute-zen",
        adresse="12 rue de la Paix",
        codePostal="75001",
        ville="Paris",
        pays="France"
    )
    db.session.add(e1)
    db.session.commit()
```

## Infrastructure Docker

### docker-compose.yml

Contient PostgreSQL 15 et un serveur mail SMTP.

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

  mail:
    build:
      context: ./mail
    container_name: mail
    restart: unless-stopped
    dns:
      - 8.8.8.8
      - 8.8.4.4
    environment:
      MAIL_DOMAIN: "gmail.com"
      RELAY_HOST: "smtp.gmail.com"
      RELAY_PORT: "587"
      RELAY_USER: "arthuralternance.pro@gmail.com"
      RELAY_PASS: "sfaj vcqj pxkh wmzs"
    ports:
      - "25:25"

volumes:
  db_data:
```

### Lancer l'infrastructure

```bash
cd Backend/src
docker-compose up -d
```

## Lancement du serveur

### Développement

```bash
cd Backend/src

# Lancer le serveur Flask (port 5000)
python app.py
```

Le serveur démarre avec les fonctionnalités suivantes:
- Création automatique des tables via `db.create_all()`
- Import automatique du fichier `seed.py` pour les données de test
- Mode debug activé
- CORS configuré pour accepter les requêtes du frontend

### Vérifier que tout fonctionne

```bash
# Tester une route
curl http://localhost:5000/api/services
```

## Récapitulatif des endpoints

| Méthode | Route | Description |
|---------|-------|-------------|
| POST | `/register_form` | Inscription utilisateur |
| POST | `/login_form` | Connexion utilisateur |
| POST | `/teste` | Test récupération utilisateur par nom |
| POST | `/contact` | Envoi formulaire de contact |
| GET | `/api/services` | Autocomplétion des services |
| GET | `/api/villes` | Autocomplétion des villes françaises |

## Architecture actuelle vs Architecture recommandée

### État actuel
L'architecture actuelle est simple et fonctionne pour un prototype:
- Un seul blueprint `pages.py` avec toutes les routes
- Pas de couche service (logique directement dans les routes)
- Pas de validation des données
- Pas d'authentification JWT
- Pas de middleware de sécurité

### Améliorations futures recommandées

1. **Authentification sécurisée**
   - Implémenter JWT avec Flask-JWT-Extended
   - Hasher les mots de passe avec Werkzeug ou bcrypt
   - Ajouter des decorators `@login_required`

2. **Organisation du code**
   - Créer plusieurs blueprints (auth, reservations, companies, etc.)
   - Séparer la logique métier dans des services
   - Créer des validators pour les données entrantes

3. **Sécurité**
   - Ajouter rate limiting avec Flask-Limiter
   - Valider et sanitiser toutes les entrées utilisateur
   - Utiliser des variables d'environnement pour les secrets (pas de mots de passe en dur)
   - Implémenter HTTPS en production

4. **Performance**
   - Ajouter du cache avec Redis
   - Optimiser les requêtes SQLAlchemy
   - Implémenter la pagination pour les endpoints

5. **Tests**
   - Ajouter pytest et des tests unitaires
   - Tests d'intégration pour les routes
   - Tests des modèles SQLAlchemy

## Prochaines étapes

- [Base de données](database.md) - Schéma détaillé PostgreSQL
- [API Endpoints](../api/endpoints.md) - Documentation complète des routes
- [Guide de développement](../development/getting-started.md) - Configuration et déploiement
