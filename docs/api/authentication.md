# Guide d'authentification

Ce guide explique en détail comment fonctionne l'authentification dans l'API Book By Click.

## Vue d'ensemble

L'API utilise **JSON Web Tokens (JWT)** pour l'authentification. C'est un standard ouvert (RFC 7519) qui définit une méthode compacte et autonome pour transmettre des informations de manière sécurisée entre les parties.

## Processus d'authentification

```mermaid
sequenceDiagram
    participant Client
    participant API
    participant Database

    Client->>API: POST /auth/login (email, password)
    API->>Database: Vérifier les credentials
    Database-->>API: Utilisateur trouvé
    API->>API: Générer JWT token
    API-->>Client: Token + données utilisateur
    Client->>Client: Stocker le token
    Client->>API: GET /reservations (avec token)
    API->>API: Valider le token
    API-->>Client: Données des réservations
```

## Inscription

### Créer un nouveau compte

**Endpoint:** `POST /auth/register`

**Corps de la requête:**

```json
{
  "email": "utilisateur@example.com",
  "password": "MotDePasseSecurise123!",
  "first_name": "Jean",
  "last_name": "Dupont",
  "role": "student",
  "school": "Lycée Technique de Paris",
  "phone": "+33612345678"
}
```

**Validation:**

- **Email:** Doit être une adresse email valide et unique
- **Mot de passe:** Minimum 8 caractères
- **Rôle:** Doit être `student`, `company` ou `admin`
- **Prénom et nom:** Obligatoires

**Réponse réussie (201 Created):**

```json
{
  "message": "User created successfully",
  "user_id": 42
}
```

**Erreurs possibles:**

| Code | Message | Signification |
|------|---------|---------------|
| 400 | Email already exists | Un compte existe déjà avec cet email |
| 400 | Password must be at least 8 characters | Mot de passe trop court |
| 400 | Invalid email format | Format d'email invalide |
| 400 | Missing required field: {field} | Champ obligatoire manquant |

---

## Connexion

### Obtenir un token JWT

**Endpoint:** `POST /auth/login`

**Corps de la requête:**

```json
{
  "email": "utilisateur@example.com",
  "password": "MotDePasseSecurise123!"
}
```

**Réponse réussie (200 OK):**

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo0MiwiZW1haWwiOiJ1dGlsaXNhdGV1ckBleGFtcGxlLmNvbSIsInJvbGUiOiJzdHVkZW50IiwiZXhwIjoxNzM0NTQ1NDAwfQ.signature",
  "user": {
    "id": 42,
    "email": "utilisateur@example.com",
    "first_name": "Jean",
    "last_name": "Dupont",
    "role": "student",
    "school": "Lycée Technique de Paris"
  }
}
```

**Erreurs possibles:**

| Code | Message | Signification |
|------|---------|---------------|
| 401 | Invalid credentials | Email ou mot de passe incorrect |
| 403 | Account disabled | Le compte a été désactivé |
| 400 | Missing email or password | Champs obligatoires manquants |

---

## Structure du JWT

### Anatomie d'un token

Un JWT est composé de trois parties séparées par des points (`.`):

```
Header.Payload.Signature
```

#### 1. Header (En-tête)

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

- `alg`: Algorithme de signature (HS256 = HMAC SHA-256)
- `typ`: Type de token (JWT)

#### 2. Payload (Données)

```json
{
  "user_id": 42,
  "email": "utilisateur@example.com",
  "role": "student",
  "exp": 1734545400,
  "iat": 1734459000
}
```

- `user_id`: Identifiant unique de l'utilisateur
- `email`: Email de l'utilisateur
- `role`: Rôle (student, company, admin)
- `exp`: Date d'expiration (timestamp Unix)
- `iat`: Date d'émission (timestamp Unix)

#### 3. Signature

La signature garantit l'intégrité du token et est calculée avec:

```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret_key
)
```

---

## Utilisation du token

### Dans les requêtes HTTP

Le token doit être inclus dans le header `Authorization` avec le préfixe `Bearer`:

```http
GET /api/reservations HTTP/1.1
Host: localhost:5000
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
```

### Exemples par langage

#### JavaScript (Fetch API)

```javascript
const token = localStorage.getItem('authToken');

fetch('http://localhost:5000/api/reservations', {
  method: 'GET',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  }
})
.then(response => response.json())
.then(data => console.log(data));
```

#### Python (requests)

```python
import requests

token = "eyJhbGciOiJIUzI1NiIs..."

headers = {
    'Authorization': f'Bearer {token}',
    'Content-Type': 'application/json'
}

response = requests.get(
    'http://localhost:5000/api/reservations',
    headers=headers
)
data = response.json()
```

#### cURL

```bash
curl -X GET http://localhost:5000/api/reservations \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." \
  -H "Content-Type: application/json"
```

---

## Stockage du token

### Frontend (Navigateur)

#### localStorage (Recommandé pour SPA)

```javascript
// Sauvegarder
localStorage.setItem('authToken', token);

// Récupérer
const token = localStorage.getItem('authToken');

// Supprimer (déconnexion)
localStorage.removeItem('authToken');
```

**Avantages:**
- Persiste entre les sessions
- Simple à utiliser
- Accessible depuis tout JavaScript

**Inconvénients:**
- Vulnérable aux attaques XSS
- Pas de protection CSRF intégrée

#### sessionStorage (Session temporaire)

```javascript
// Similaire à localStorage, mais expire à la fermeture de l'onglet
sessionStorage.setItem('authToken', token);
```

#### Cookies HTTP-Only (Plus sécurisé)

```javascript
// Côté serveur (Flask)
from flask import make_response

response = make_response(jsonify({'user': user_data}))
response.set_cookie(
    'authToken',
    token,
    httponly=True,
    secure=True,  # HTTPS uniquement
    samesite='Strict'
)
```

**Avantages:**
- Protection contre XSS (httponly)
- Protection CSRF avec samesite
- Plus sécurisé

**Inconvénients:**
- Plus complexe à implémenter
- Nécessite configuration CORS

---

## Expiration et renouvellement

### Durée de vie

Par défaut, les tokens expirent après **24 heures**.

### Vérifier l'expiration

```javascript
function isTokenExpired(token) {
  try {
    const payload = JSON.parse(atob(token.split('.')[1]));
    const expiration = payload.exp * 1000; // Convertir en millisecondes
    return Date.now() > expiration;
  } catch (error) {
    return true;
  }
}

const token = localStorage.getItem('authToken');
if (isTokenExpired(token)) {
  // Rediriger vers login
  window.location.href = '/login';
}
```

### Renouvellement automatique

Pour éviter de demander à l'utilisateur de se reconnecter fréquemment, vous pouvez implémenter un système de refresh token:

```javascript
async function refreshToken() {
  const response = await fetch('http://localhost:5000/api/auth/refresh', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${localStorage.getItem('authToken')}`
    }
  });

  if (response.ok) {
    const data = await response.json();
    localStorage.setItem('authToken', data.token);
    return data.token;
  } else {
    // Token invalide, rediriger vers login
    window.location.href = '/login';
  }
}
```

---

## Gestion des erreurs d'authentification

### Codes d'erreur

| Code | Description | Action recommandée |
|------|-------------|-------------------|
| 401 | Unauthorized - Token manquant ou invalide | Rediriger vers /login |
| 403 | Forbidden - Token valide mais permissions insuffisantes | Afficher erreur "Accès refusé" |
| 419 | Authentication Timeout - Token expiré | Redemander la connexion |

### Intercepteur de requêtes

#### Axios (JavaScript)

```javascript
import axios from 'axios';

axios.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 401) {
      // Token invalide, rediriger vers login
      localStorage.removeItem('authToken');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

#### Fetch API (JavaScript)

```javascript
async function fetchWithAuth(url, options = {}) {
  const token = localStorage.getItem('authToken');

  const response = await fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'Authorization': `Bearer ${token}`
    }
  });

  if (response.status === 401) {
    localStorage.removeItem('authToken');
    window.location.href = '/login';
    throw new Error('Unauthorized');
  }

  return response;
}
```

---

## Sécurité

### Bonnes pratiques

1. **HTTPS uniquement en production**
   - Ne jamais transmettre de tokens sur HTTP non sécurisé

2. **Stocker de manière sécurisée**
   - Utiliser httpOnly cookies quand possible
   - Éviter de stocker dans l'URL ou query params

3. **Valider côté serveur**
   - Ne jamais faire confiance au client
   - Toujours vérifier le token sur le serveur

4. **Expiration courte**
   - Tokens de courte durée (24h max)
   - Implémenter un refresh token pour les sessions longues

5. **Protection XSS**
   - Sanitizer toutes les entrées utilisateur
   - Utiliser Content Security Policy (CSP)

6. **Logout approprié**
   - Supprimer le token côté client
   - Implémenter une blacklist de tokens côté serveur (optionnel)

### Vérification du token côté serveur

```python
# Backend Flask
from functools import wraps
import jwt
from flask import request, jsonify
import os

def token_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        token = request.headers.get('Authorization')

        if not token:
            return jsonify({'error': 'Token is missing'}), 401

        try:
            # Enlever "Bearer " du token
            token = token.replace('Bearer ', '')

            # Décoder et valider
            data = jwt.decode(
                token,
                os.getenv('JWT_SECRET_KEY'),
                algorithms=['HS256']
            )

            # Ajouter les infos utilisateur à la requête
            request.user_id = data['user_id']
            request.user_role = data['role']

        except jwt.ExpiredSignatureError:
            return jsonify({'error': 'Token has expired'}), 401
        except jwt.InvalidTokenError:
            return jsonify({'error': 'Invalid token'}), 401

        return f(*args, **kwargs)

    return decorated
```

---

## Ressources

- [Documentation endpoints](endpoints.md)
- [Exemples d'utilisation](examples.md)
- [Spécification JWT (RFC 7519)](https://tools.ietf.org/html/rfc7519)
- [jwt.io - Décodeur de JWT](https://jwt.io/)
