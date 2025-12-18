# Référence API

Documentation complète de l'API REST du système de réservation Book By Click.

## URL de base

**Développement:**
```
http://localhost:5000
```

**Production:**
```
https://api.book-by-click.example.com
```

## Format des données

- **Content-Type:** `application/x-www-form-urlencoded` ou `application/json` selon l'endpoint
- **Charset:** UTF-8

## Endpoints

### Authentification

#### POST /register_form

Inscription d'un nouvel utilisateur.

**Content-Type:** `application/x-www-form-urlencoded`

**Paramètres (form-data):**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `email` | string | Oui | Adresse email de l'utilisateur |
| `password` | string | Oui | Mot de passe |
| `nom` | string | Oui | Nom de famille |

**Exemple avec cURL:**
```bash
curl -X POST http://localhost:5000/register_form \
  -d "email=utilisateur@example.com" \
  -d "password=motdepasse123" \
  -d "nom=Dupont"
```

**Exemple avec JavaScript (Fetch):**
```javascript
const formData = new FormData();
formData.append('email', 'utilisateur@example.com');
formData.append('password', 'motdepasse123');
formData.append('nom', 'Dupont');

fetch('http://localhost:5000/register_form', {
  method: 'POST',
  body: formData
})
```

**Réponse:** `200 OK`
```
OK
```

---

#### POST /login_form

Connexion d'un utilisateur existant.

**Content-Type:** `application/x-www-form-urlencoded`

**Paramètres (form-data):**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `email` | string | Oui | Adresse email du compte |
| `password` | string | Oui | Mot de passe |

**Exemple avec cURL:**
```bash
curl -X POST http://localhost:5000/login_form \
  -d "email=utilisateur@example.com" \
  -d "password=motdepasse123"
```

**Exemple avec JavaScript:**
```javascript
const formData = new FormData();
formData.append('email', 'utilisateur@example.com');
formData.append('password', 'motdepasse123');

const response = await fetch('http://localhost:5000/login_form', {
  method: 'POST',
  body: formData
});

const data = await response.json();
console.log(data);
```

**Réponse:** `200 OK`
```json
{
  "email": "utilisateur@example.com",
  "password": "motdepasse123"
}
```

!!! warning "Attention"
    Cette route retourne actuellement le mot de passe en clair. Elle est en cours de développement et ne doit **pas être utilisée en production** sans sécurisation appropriée.

---

### Utilisateurs

#### POST /teste

Récupérer les informations d'un utilisateur par son nom.

**Content-Type:** `application/x-www-form-urlencoded`

**Paramètres (form-data):**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `username` | string | Oui | Nom de l'utilisateur à rechercher |

**Exemple avec cURL:**
```bash
curl -X POST http://localhost:5000/teste \
  -d "username=Dupont"
```

**Réponse:** `200 OK`
```json
{
  "id": 1,
  "nom": "Dupont",
  "prenom": "Jean",
  "dateNaissance": "1990-05-15",
  "email": "jean.dupont@example.com",
  "motDePasseHash": "$2b$12$...",
  "telephone": "0612345678"
}
```

**Erreurs possibles:**

| Code | Description |
|------|-------------|
| 404 | Utilisateur non trouvé |
| 500 | Erreur serveur (base de données indisponible) |

---

### Contact

#### POST /contact

Envoyer un message de contact par email.

**Content-Type:** `application/json`

**Paramètres (JSON):**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `name` | string | Oui | Nom de la personne |
| `email` | string | Oui | Email de contact |
| `phone` | string | Non | Numéro de téléphone (optionnel) |
| `message` | string | Oui | Message à envoyer |

**Exemple avec cURL:**
```bash
curl -X POST http://localhost:5000/contact \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Jean Dupont",
    "email": "jean.dupont@example.com",
    "phone": "0612345678",
    "message": "Je souhaite obtenir plus d'\''informations sur vos services."
  }'
```

**Exemple avec JavaScript:**
```javascript
fetch('http://localhost:5000/contact', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: 'Jean Dupont',
    email: 'jean.dupont@example.com',
    phone: '0612345678',
    message: 'Je souhaite obtenir plus d\'informations sur vos services.'
  })
})
.then(response => response.json())
.then(data => console.log(data));
```

**Réponse:** `200 OK`
```json
{
  "success": true,
  "message": "Votre message a été envoyé avec succès !"
}
```

**Erreur:** `400 Bad Request`
```json
{
  "success": false,
  "error": "Tous les champs sont requis"
}
```

**Erreur:** `500 Internal Server Error`
```json
{
  "success": false,
  "error": "Erreur lors de l'envoi du message"
}
```

---

### Autocomplétion

#### GET /api/services

Récupérer la liste des services disponibles avec autocomplétion.

**Paramètres de requête (Query):**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `q` | string | Non | Terme de recherche pour filtrer les services |

**Exemple sans filtre:**
```bash
curl http://localhost:5000/api/services
```

**Exemple avec recherche:**
```bash
curl "http://localhost:5000/api/services?q=dev"
```

**Exemple avec JavaScript:**
```javascript
// Sans filtre
fetch('http://localhost:5000/api/services')
  .then(response => response.json())
  .then(data => console.log(data));

// Avec recherche
fetch('http://localhost:5000/api/services?q=dev')
  .then(response => response.json())
  .then(data => console.log(data));
```

**Réponse:** `200 OK`
```json
[
  "Développement Web",
  "Développement Mobile",
  "Développement Logiciel",
  ...
]
```

!!! note
    Retourne maximum 20 résultats quand un filtre est appliqué. La liste est chargée depuis `/Backend/src/data/services.json`.

---

#### GET /api/villes

Récupérer la liste des villes de France avec autocomplétion.

**Paramètres de requête (Query):**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `q` | string | Non | Terme de recherche pour filtrer les villes |

**Exemple sans filtre:**
```bash
curl http://localhost:5000/api/villes
```

**Exemple avec recherche:**
```bash
curl "http://localhost:5000/api/villes?q=par"
```

**Exemple avec JavaScript:**
```javascript
// Recherche dynamique (debounced)
let timeout;
const searchInput = document.getElementById('ville');

searchInput.addEventListener('input', (e) => {
  clearTimeout(timeout);
  timeout = setTimeout(() => {
    const query = e.target.value;

    fetch(`http://localhost:5000/api/villes?q=${query}`)
      .then(response => response.json())
      .then(villes => {
        // Afficher les suggestions
        console.log(villes);
      });
  }, 300);
});
```

**Réponse sans filtre:** `200 OK` (100 premières villes)
```json
[
  "Paris",
  "Marseille",
  "Lyon",
  "Toulouse",
  ...
]
```

**Réponse avec filtre `q=par`:** `200 OK` (max 15 résultats)
```json
[
  "Paris",
  "Paray-le-Monial",
  "Paray-Vieille-Poste",
  "Parcay-Meslay",
  ...
]
```

!!! info "Performance"
    - Les villes sont chargées depuis `/Backend/src/data/communes-france-avec-polygon-2025.json`
    - Système de cache global pour optimiser les performances
    - Filtrage côté serveur avec `startswith()` pour des résultats pertinents

---

### Réservations

!!! warning "En cours de développement"
    Les endpoints suivants sont à implémenter selon le schéma de base de données.


Selon le modèle de base de données (voir [Base de données](../architecture/database.md)), les endpoints de réservation utiliseront les tables:

- `reservation`: Réservations des clients
- `creneau`: Créneaux horaires disponibles
- `prestation`: Services offerts par les entreprises

---

## Résumé des endpoints

| Méthode | Endpoint | Description | Auth |
|---------|----------|-------------|------|
| POST | `/register_form` | Inscription utilisateur | Non |
| POST | `/login_form` | Connexion utilisateur | Non |
| POST | `/teste` | Récupérer un utilisateur par nom | Non |
| POST | `/contact` | Envoyer un message de contact | Non |
| GET | `/api/services` | Autocomplétion des services | Non |
| GET | `/api/villes` | Autocomplétion des villes de France | Non |

## Notes techniques

### Base de données

L'API utilise **PostgreSQL 15** avec SQLAlchemy comme ORM.

Configuration par défaut:
```
postgresql://appuser:apppassword@localhost:5432/appdb
```

Voir le fichier `Backend/src/docker-compose.yml` pour la configuration complète.

### CORS

CORS est activé sur tous les endpoints via Flask-CORS.

### Email

Le système d'envoi d'emails utilise un serveur SMTP configuré via Docker (voir `docker-compose.yml`).

Configuration Gmail relay:
- Host: smtp.gmail.com
- Port: 587
- TLS activé

---

## Développement futur

### Endpoints à implémenter

Basé sur le schéma de base de données, voici les endpoints qui devraient être implémentés:

#### Entreprises
- `GET /api/entreprises` - Liste des entreprises
- `GET /api/entreprises/{id}` - Détails d'une entreprise
- `POST /api/entreprises` - Créer une entreprise (pour les gérants)
- `PUT /api/entreprises/{id}` - Modifier une entreprise
- `GET /api/entreprises/{id}/creneaux` - Créneaux d'une entreprise

#### Réservations
- `GET /api/reservations` - Liste des réservations
- `POST /api/reservations` - Créer une réservation  
- `PUT /api/reservations/{id}` - Modifier une réservation
- `DELETE /api/reservations/{id}` - Annuler une réservation

#### Prestations
- `GET /api/prestations` - Liste des prestations
- `GET /api/entreprises/{id}/prestations` - Prestations d'une entreprise

#### Événements
- `GET /api/evenements` - Liste des événements
- `GET /api/entreprises/{id}/evenements` - Événements d'une entreprise

---

## Ressources

- [Guide d'authentification](authentication.md)
- [Exemples d'utilisation](examples.md)
- [Modèles de données](../architecture/database.md)
- [Architecture Backend](../architecture/backend.md)
