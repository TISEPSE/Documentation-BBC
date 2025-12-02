# Architecture Backend

## Vue d'ensemble

Le backend est une API REST construite avec **Flask** (Python) qui gère la logique métier et la communication avec la base de données MySQL.

## Structure actuelle

```text
Backend/
├── app.py              # Application Flask principale
├── db/                 # Configuration base de données
│   └── docker-compose.yml
└── __pycache__/        # Cache Python
```

## Flask Application

### app.py

Point d'entrée de l'application Flask.

```python
from flask import Flask, jsonify
from flask_cors import CORS

app = Flask(__name__)
CORS(app)  # Autoriser les requêtes depuis le frontend

@app.route('/api/health', methods=['GET'])
def health():
    return jsonify({
        'status': 'ok',
        'message': 'API is running'
    })

if __name__ == '__main__':
    app.run(debug=True, port=5000)
```

## Configuration

### Variables d'environnement

Créer un fichier `.env` dans `Backend/` :

```env
FLASK_APP=app.py
FLASK_ENV=development
SECRET_KEY=votre_cle_secrete

DATABASE_HOST=localhost
DATABASE_USER=reserv_user
DATABASE_PASSWORD=votre_mot_de_passe
DATABASE_NAME=reservations_db

JWT_SECRET_KEY=cle_jwt_secrete
JWT_ACCESS_TOKEN_EXPIRES=3600
```

### Chargement de la config

```python
from dotenv import load_dotenv
import os

load_dotenv()

class Config:
    SECRET_KEY = os.getenv('SECRET_KEY')
    DATABASE_URI = f"mysql://{os.getenv('DATABASE_USER')}:{os.getenv('DATABASE_PASSWORD')}@{os.getenv('DATABASE_HOST')}/{os.getenv('DATABASE_NAME')}"

app.config.from_object(Config)
```

## Structure recommandée

Pour un backend plus organisé :

```text
Backend/
├── app.py                  # Point d'entrée
├── config.py               # Configuration
├── requirements.txt        # Dépendances
├── .env                    # Variables d'environnement
├── routes/                 # Routes API
│   ├── __init__.py
│   ├── auth.py            # Routes authentification
│   ├── users.py           # Routes utilisateurs
│   ├── companies.py       # Routes entreprises
│   └── reservations.py    # Routes réservations
├── models/                 # Modèles de données
│   ├── __init__.py
│   ├── user.py
│   ├── company.py
│   └── reservation.py
├── services/               # Logique métier
│   ├── __init__.py
│   ├── auth_service.py
│   └── reservation_service.py
├── utils/                  # Utilitaires
│   ├── __init__.py
│   ├── database.py        # Connexion DB
│   ├── validators.py      # Validations
│   └── decorators.py      # Decorators (auth, etc.)
└── db/
    └── docker-compose.yml
```

## Routes API

### Authentification

```python
# routes/auth.py
from flask import Blueprint, request, jsonify
from services.auth_service import AuthService

auth_bp = Blueprint('auth', __name__)

@auth_bp.route('/register', methods=['POST'])
def register():
    data = request.get_json()
    result = AuthService.register(data)
    return jsonify(result), 201

@auth_bp.route('/login', methods=['POST'])
def login():
    data = request.get_json()
    result = AuthService.login(data['email'], data['password'])
    return jsonify(result), 200
```

### Réservations

```python
# routes/reservations.py
from flask import Blueprint, request, jsonify
from utils.decorators import login_required

reservations_bp = Blueprint('reservations', __name__)

@reservations_bp.route('/', methods=['GET'])
@login_required
def get_reservations():
    # Récupérer les réservations
    pass

@reservations_bp.route('/', methods=['POST'])
@login_required
def create_reservation():
    data = request.get_json()
    # Créer une réservation
    pass
```

### Enregistrement des blueprints

```python
# app.py
from routes.auth import auth_bp
from routes.reservations import reservations_bp

app.register_blueprint(auth_bp, url_prefix='/api/auth')
app.register_blueprint(reservations_bp, url_prefix='/api/reservations')
```

## Base de données

### Connexion MySQL

```python
# utils/database.py
import mysql.connector
from mysql.connector import pooling
import os

db_config = {
    'host': os.getenv('DATABASE_HOST'),
    'user': os.getenv('DATABASE_USER'),
    'password': os.getenv('DATABASE_PASSWORD'),
    'database': os.getenv('DATABASE_NAME')
}

connection_pool = pooling.MySQLConnectionPool(
    pool_name="reservation_pool",
    pool_size=5,
    **db_config
)

def get_db_connection():
    return connection_pool.get_connection()
```

### Modèles

```python
# models/user.py
from utils.database import get_db_connection

class User:
    @staticmethod
    def find_by_email(email):
        conn = get_db_connection()
        cursor = conn.cursor(dictionary=True)
        cursor.execute("SELECT * FROM users WHERE email = %s", (email,))
        user = cursor.fetchone()
        cursor.close()
        conn.close()
        return user

    @staticmethod
    def create(user_data):
        conn = get_db_connection()
        cursor = conn.cursor()
        query = """
            INSERT INTO users (email, password, first_name, last_name, role)
            VALUES (%s, %s, %s, %s, %s)
        """
        cursor.execute(query, (
            user_data['email'],
            user_data['password'],
            user_data['first_name'],
            user_data['last_name'],
            user_data['role']
        ))
        conn.commit()
        user_id = cursor.lastrowid
        cursor.close()
        conn.close()
        return user_id
```

## Services

### Auth Service

```python
# services/auth_service.py
from models.user import User
from werkzeug.security import generate_password_hash, check_password_hash
import jwt
import os
from datetime import datetime, timedelta

class AuthService:
    @staticmethod
    def register(user_data):
        # Vérifier si l'email existe déjà
        existing_user = User.find_by_email(user_data['email'])
        if existing_user:
            return {'error': 'Email already exists'}, 400

        # Hasher le mot de passe
        user_data['password'] = generate_password_hash(user_data['password'])

        # Créer l'utilisateur
        user_id = User.create(user_data)

        return {'message': 'User created successfully', 'user_id': user_id}

    @staticmethod
    def login(email, password):
        user = User.find_by_email(email)

        if not user or not check_password_hash(user['password'], password):
            return {'error': 'Invalid credentials'}, 401

        # Générer JWT token
        token = jwt.encode({
            'user_id': user['id'],
            'email': user['email'],
            'role': user['role'],
            'exp': datetime.utcnow() + timedelta(hours=24)
        }, os.getenv('JWT_SECRET_KEY'), algorithm='HS256')

        return {
            'token': token,
            'user': {
                'id': user['id'],
                'email': user['email'],
                'role': user['role']
            }
        }
```

## Middleware & Decorators

### Authentification

```python
# utils/decorators.py
from functools import wraps
from flask import request, jsonify
import jwt
import os

def login_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        token = request.headers.get('Authorization')

        if not token:
            return jsonify({'error': 'Token is missing'}), 401

        try:
            # Enlever "Bearer " si présent
            token = token.replace('Bearer ', '')
            data = jwt.decode(token, os.getenv('JWT_SECRET_KEY'), algorithms=['HS256'])
            request.user_id = data['user_id']
            request.user_role = data['role']
        except jwt.ExpiredSignatureError:
            return jsonify({'error': 'Token expired'}), 401
        except jwt.InvalidTokenError:
            return jsonify({'error': 'Invalid token'}), 401

        return f(*args, **kwargs)

    return decorated_function
```

### CORS

```python
from flask_cors import CORS

CORS(app, resources={
    r"/api/*": {
        "origins": ["http://localhost:5173"],
        "methods": ["GET", "POST", "PUT", "DELETE"],
        "allow_headers": ["Content-Type", "Authorization"]
    }
})
```

## Gestion des erreurs

```python
# app.py
@app.errorhandler(404)
def not_found(error):
    return jsonify({'error': 'Not found'}), 404

@app.errorhandler(500)
def internal_error(error):
    return jsonify({'error': 'Internal server error'}), 500

@app.errorhandler(Exception)
def handle_exception(error):
    return jsonify({'error': str(error)}), 500
```

## Validation

```python
# utils/validators.py
import re

def validate_email(email):
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return re.match(pattern, email) is not None

def validate_password(password):
    return len(password) >= 8

def validate_user_data(data):
    errors = []

    if 'email' not in data or not validate_email(data['email']):
        errors.append('Invalid email')

    if 'password' not in data or not validate_password(data['password']):
        errors.append('Password must be at least 8 characters')

    if 'first_name' not in data or not data['first_name']:
        errors.append('First name is required')

    return errors
```

## Dépendances

### requirements.txt

```txt
Flask==3.0.0
Flask-CORS==4.0.0
mysql-connector-python==8.2.0
PyJWT==2.8.0
python-dotenv==1.0.0
Werkzeug==3.0.0
gunicorn==21.2.0
```

### Installation

```bash
pip install -r requirements.txt
```

## Docker Database

### docker-compose.yml

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: reservations_db
      MYSQL_USER: reserv_user
      MYSQL_PASSWORD: reserv_password
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql

volumes:
  mysql_data:
```

## Lancement

### Développement

```bash
# Activer l'environnement virtuel
source ../venv/bin/activate

# Lancer Flask
python -m flask --app app run --no-debugger
```

### Production

```bash
gunicorn -w 4 -b 0.0.0.0:5000 app:app
```

## Tests

```python
# tests/test_auth.py
import pytest
from app import app

@pytest.fixture
def client():
    app.config['TESTING'] = True
    with app.test_client() as client:
        yield client

def test_health(client):
    response = client.get('/api/health')
    assert response.status_code == 200
    assert response.json['status'] == 'ok'

def test_register(client):
    response = client.post('/api/auth/register', json={
        'email': 'test@example.com',
        'password': 'password123',
        'first_name': 'John',
        'last_name': 'Doe',
        'role': 'student'
    })
    assert response.status_code == 201
```

## Bonnes pratiques

1. Utiliser des **blueprints** pour organiser les routes
2. **Séparer** la logique métier dans des services
3. **Valider** toutes les entrées utilisateur
4. **Hasher** les mots de passe avec bcrypt/werkzeug
5. Utiliser des **variables d'environnement** pour les secrets
6. **Logger** les erreurs et actions importantes
7. Implémenter un **rate limiting**
8. Utiliser des **transactions** pour les opérations critiques

## Prochaines étapes

- [Base de données](database.md)
- [API Endpoints](../api/endpoints.md)
- [Guide de développement](../development/getting-started.md)
