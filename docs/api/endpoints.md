# API Endpoints

## Base URL

```text
http://localhost:5000/api
```

## Authentification

Tous les endpoints protégés nécessitent un header `Authorization` :

```http
Authorization: Bearer {token}
```

## Endpoints

### Auth

#### POST /api/auth/register

Créer un nouveau compte.

**Body:**
```json
{
  "email": "user@example.com",
  "password": "securepass123",
  "first_name": "John",
  "last_name": "Doe",
  "role": "student",
  "school": "Mon École"
}
```

**Response:** `201 Created`
```json
{
  "message": "User created successfully",
  "user_id": 123
}
```

#### POST /api/auth/login

Connexion.

**Body:**
```json
{
  "email": "user@example.com",
  "password": "securepass123"
}
```

**Response:** `200 OK`
```json
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": 123,
    "email": "user@example.com",
    "role": "student"
  }
}
```

### Users

#### GET /api/users/me

Obtenir le profil utilisateur connecté.

**Headers:** Authorization required

**Response:** `200 OK`
```json
{
  "id": 123,
  "email": "user@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "role": "student",
  "school": "Mon École"
}
```

#### PUT /api/users/me

Mettre à jour le profil.

**Headers:** Authorization required

**Body:**
```json
{
  "first_name": "John",
  "last_name": "Smith",
  "phone": "0612345678"
}
```

**Response:** `200 OK`

### Companies

#### GET /api/companies

Lister toutes les entreprises actives.

**Query params:**
- `city` : Filtrer par ville
- `sector` : Filtrer par secteur

**Response:** `200 OK`
```json
[
  {
    "id": 1,
    "name": "Tech Company",
    "sector": "Informatique",
    "city": "Paris",
    "description": "...",
    "website": "https://example.com"
  }
]
```

#### GET /api/companies/{id}

Détails d'une entreprise.

**Response:** `200 OK`
```json
{
  "id": 1,
  "name": "Tech Company",
  "siret": "12345678901234",
  "address": "123 Rue Example",
  "city": "Paris",
  "sector": "Informatique",
  "description": "...",
  "timeslots": [
    {
      "day_of_week": "monday",
      "start_time": "09:00",
      "end_time": "12:00"
    }
  ]
}
```

### Reservations

#### GET /api/reservations

Lister les réservations de l'utilisateur.

**Headers:** Authorization required

**Query params:**
- `status` : pending, confirmed, cancelled, completed

**Response:** `200 OK`
```json
[
  {
    "id": 1,
    "company": {
      "id": 1,
      "name": "Tech Company"
    },
    "date": "2024-12-15",
    "time_start": "10:00",
    "time_end": "11:00",
    "status": "confirmed",
    "notes": "Intéressé par un stage"
  }
]
```

#### POST /api/reservations

Créer une réservation.

**Headers:** Authorization required

**Body:**
```json
{
  "company_id": 1,
  "date": "2024-12-15",
  "time_start": "10:00",
  "time_end": "11:00",
  "notes": "Intéressé par un stage en développement"
}
```

**Response:** `201 Created`
```json
{
  "id": 123,
  "message": "Reservation created successfully"
}
```

#### PUT /api/reservations/{id}

Modifier une réservation (si pending).

**Headers:** Authorization required

**Body:**
```json
{
  "date": "2024-12-16",
  "notes": "Modification de la demande"
}
```

**Response:** `200 OK`

#### DELETE /api/reservations/{id}

Annuler une réservation.

**Headers:** Authorization required

**Query params:**
- `reason` : Raison d'annulation

**Response:** `200 OK`
```json
{
  "message": "Reservation cancelled"
}
```

### Timeslots

#### GET /api/companies/{id}/timeslots

Créneaux disponibles d'une entreprise.

**Query params:**
- `date` : Date spécifique (YYYY-MM-DD)
- `week` : Semaine (numéro)

**Response:** `200 OK`
```json
[
  {
    "date": "2024-12-15",
    "day_of_week": "monday",
    "start_time": "10:00",
    "end_time": "11:00",
    "available": true
  }
]
```

## Codes de réponse

| Code | Description |
|------|-------------|
| 200 | OK |
| 201 | Created |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Internal Server Error |

## Exemples

### Flux complet : Créer une réservation

```javascript
// 1. Login
const loginRes = await fetch('http://localhost:5000/api/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    email: 'student@example.com',
    password: 'password123'
  })
})
const { token } = await loginRes.json()

// 2. Voir les entreprises
const companiesRes = await fetch('http://localhost:5000/api/companies')
const companies = await companiesRes.json()

// 3. Voir les créneaux disponibles
const slotsRes = await fetch(`http://localhost:5000/api/companies/1/timeslots?date=2024-12-15`)
const slots = await slotsRes.json()

// 4. Créer la réservation
const reservRes = await fetch('http://localhost:5000/api/reservations', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  },
  body: JSON.stringify({
    company_id: 1,
    date: '2024-12-15',
    time_start: '10:00',
    time_end: '11:00',
    notes: 'Intéressé par un stage'
  })
})
```
