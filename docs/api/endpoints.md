# Référence API

Documentation complète de l'API REST du système de réservation Book By Click.

## URL de base

**Développement:**
```
http://localhost:5000/api
```

**Production:**
```
https://api.book-by-click.example.com/api
```

## Authentification

L'API utilise **JWT (JSON Web Tokens)** pour l'authentification.

### Obtenir un token

Utilisez l'endpoint `/auth/login` pour obtenir un token JWT valide.

### Utiliser le token

Tous les endpoints protégés nécessitent le header `Authorization` avec le format Bearer:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

### Expiration

Les tokens expirent après **24 heures**. Vous devrez vous reconnecter pour obtenir un nouveau token.

## Format des réponses

Toutes les réponses sont au format **JSON**.

### Réponse réussie

```json
{
  "data": { ... },
  "message": "Success"
}
```

### Réponse d'erreur

```json
{
  "error": "Description de l'erreur",
  "code": "ERROR_CODE"
}
```

## Endpoints

### Authentification

#### POST /auth/register

Créer un nouveau compte utilisateur dans le système.

**Authentification:** Non requise

**Paramètres du corps (JSON):**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `email` | string | Oui | Adresse email de l'utilisateur (doit être unique) |
| `password` | string | Oui | Mot de passe (minimum 8 caractères) |
| `first_name` | string | Oui | Prénom |
| `last_name` | string | Oui | Nom de famille |
| `role` | string | Oui | Rôle: `student`, `company` ou `admin` |
| `school` | string | Non | Nom de l'établissement scolaire (uniquement pour les étudiants) |
| `phone` | string | Non | Numéro de téléphone |

**Exemple de requête:**
```json
{
  "email": "john.doe@example.com",
  "password": "SecurePass123!",
  "first_name": "John",
  "last_name": "Doe",
  "role": "student",
  "school": "Université Paris-Saclay",
  "phone": "+33612345678"
}
```

**Réponse:** `201 Created`
```json
{
  "message": "User created successfully",
  "user_id": 123
}
```

**Erreurs possibles:**

| Code | Description |
|------|-------------|
| 400 | Email déjà utilisé ou données invalides |
| 400 | Mot de passe trop court (< 8 caractères) |
| 400 | Champs requis manquants |

---

#### POST /auth/login

Connexion au système et récupération d'un token JWT.

**Authentification:** Non requise

**Paramètres du corps (JSON):**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `email` | string | Oui | Adresse email du compte |
| `password` | string | Oui | Mot de passe |

**Exemple de requête:**
```json
{
  "email": "john.doe@example.com",
  "password": "SecurePass123!"
}
```

**Réponse:** `200 OK`
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 123,
    "email": "john.doe@example.com",
    "first_name": "John",
    "last_name": "Doe",
    "role": "student",
    "school": "Université Paris-Saclay"
  }
}
```

**Erreurs possibles:**

| Code | Description |
|------|-------------|
| 401 | Email ou mot de passe incorrect |
| 400 | Champs requis manquants |
| 403 | Compte désactivé |

---

### Utilisateurs

#### GET /users/me

Récupérer les informations du profil de l'utilisateur connecté.

**Authentification:** Requise (Bearer Token)

**Paramètres:** Aucun

**Réponse:** `200 OK`
```json
{
  "id": 123,
  "email": "john.doe@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "role": "student",
  "school": "Université Paris-Saclay",
  "phone": "+33612345678",
  "is_active": true,
  "email_verified": true,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-12-01T14:22:00Z"
}
```

**Erreurs possibles:**

| Code | Description |
|------|-------------|
| 401 | Token manquant ou invalide |
| 403 | Token expiré |

---

#### PUT /users/me

Mettre à jour les informations du profil utilisateur.

**Authentification:** Requise (Bearer Token)

**Paramètres du corps (JSON):**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `first_name` | string | Non | Nouveau prénom |
| `last_name` | string | Non | Nouveau nom de famille |
| `phone` | string | Non | Nouveau numéro de téléphone |
| `school` | string | Non | Nouvel établissement (pour les étudiants) |

**Exemple de requête:**
```json
{
  "first_name": "John",
  "last_name": "Smith",
  "phone": "+33612345679",
  "school": "Sorbonne Université"
}
```

**Réponse:** `200 OK`
```json
{
  "message": "Profile updated successfully",
  "user": {
    "id": 123,
    "email": "john.doe@example.com",
    "first_name": "John",
    "last_name": "Smith",
    "phone": "+33612345679",
    "school": "Sorbonne Université"
  }
}
```

**Erreurs possibles:**

| Code | Description |
|------|-------------|
| 401 | Token manquant ou invalide |
| 400 | Données invalides |

---

### Entreprises

#### GET /companies

Récupérer la liste de toutes les entreprises actives disponibles pour les réservations.

**Authentification:** Non requise

**Paramètres de requête (Query):**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `city` | string | Non | Filtrer par ville (ex: "Paris") |
| `sector` | string | Non | Filtrer par secteur d'activité (ex: "Informatique") |
| `page` | integer | Non | Numéro de page (défaut: 1) |
| `limit` | integer | Non | Nombre d'entreprises par page (défaut: 20, max: 100) |

**Exemple de requête:**
```
GET /companies?city=Paris&sector=Informatique&page=1&limit=10
```

**Réponse:** `200 OK`
```json
{
  "total": 45,
  "page": 1,
  "limit": 10,
  "companies": [
    {
      "id": 1,
      "name": "Tech Innovations SAS",
      "sector": "Informatique",
      "city": "Paris",
      "postal_code": "75001",
      "description": "Entreprise spécialisée dans le développement web et mobile",
      "website": "https://tech-innovations.example.com",
      "logo_url": "https://example.com/logos/tech-innov.png",
      "is_active": true
    },
    {
      "id": 2,
      "name": "Digital Services Pro",
      "sector": "Informatique",
      "city": "Paris",
      "postal_code": "75008",
      "description": "Conseil et services numériques",
      "website": "https://digital-services.example.com",
      "logo_url": null,
      "is_active": true
    }
  ]
}
```

**Erreurs possibles:**

| Code | Description |
|------|-------------|
| 400 | Paramètres de pagination invalides |

---

#### GET /companies/{id}

Récupérer les détails complets d'une entreprise spécifique, incluant ses créneaux horaires.

**Authentification:** Non requise

**Paramètres d'URL:**

| Paramètre | Type | Description |
|-----------|------|-------------|
| `id` | integer | ID de l'entreprise |

**Exemple de requête:**
```
GET /companies/1
```

**Réponse:** `200 OK`
```json
{
  "id": 1,
  "name": "Tech Innovations SAS",
  "siret": "12345678901234",
  "address": "123 Avenue des Champs-Élysées",
  "city": "Paris",
  "postal_code": "75008",
  "phone": "+33142123456",
  "email": "contact@tech-innovations.example.com",
  "website": "https://tech-innovations.example.com",
  "sector": "Informatique",
  "description": "Leader dans le développement d'applications web et mobile innovantes. Nous recherchons des stagiaires motivés pour rejoindre nos équipes.",
  "logo_url": "https://example.com/logos/tech-innov.png",
  "is_active": true,
  "created_at": "2024-01-10T09:00:00Z",
  "timeslots": [
    {
      "id": 1,
      "day_of_week": "monday",
      "start_time": "09:00:00",
      "end_time": "12:00:00",
      "max_reservations": 3,
      "is_active": true
    },
    {
      "id": 2,
      "day_of_week": "monday",
      "start_time": "14:00:00",
      "end_time": "17:00:00",
      "max_reservations": 3,
      "is_active": true
    },
    {
      "id": 3,
      "day_of_week": "wednesday",
      "start_time": "09:00:00",
      "end_time": "12:00:00",
      "max_reservations": 2,
      "is_active": true
    }
  ]
}
```

**Erreurs possibles:**

| Code | Description |
|------|-------------|
| 404 | Entreprise non trouvée |

---

### Réservations

#### GET /reservations

Récupérer la liste des réservations de l'utilisateur connecté.

**Authentification:** Requise (Bearer Token)

**Paramètres de requête (Query):**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `status` | string | Non | Filtrer par statut: `pending`, `confirmed`, `cancelled`, `completed` |
| `from_date` | date | Non | Date de début (format: YYYY-MM-DD) |
| `to_date` | date | Non | Date de fin (format: YYYY-MM-DD) |
| `company_id` | integer | Non | Filtrer par entreprise |

**Exemple de requête:**
```
GET /reservations?status=confirmed&from_date=2024-12-01
```

**Réponse:** `200 OK`
```json
{
  "total": 5,
  "reservations": [
    {
      "id": 1,
      "company": {
        "id": 1,
        "name": "Tech Innovations SAS",
        "city": "Paris",
        "logo_url": "https://example.com/logos/tech-innov.png"
      },
      "date": "2024-12-15",
      "time_start": "10:00:00",
      "time_end": "11:00:00",
      "status": "confirmed",
      "notes": "Intéressé par un stage en développement web",
      "confirmed_at": "2024-12-10T14:30:00Z",
      "created_at": "2024-12-08T09:15:00Z"
    },
    {
      "id": 2,
      "company": {
        "id": 2,
        "name": "Digital Services Pro",
        "city": "Lyon",
        "logo_url": null
      },
      "date": "2024-12-20",
      "time_start": "14:00:00",
      "time_end": "15:30:00",
      "status": "pending",
      "notes": "Demande d'informations sur les opportunités de stage",
      "confirmed_at": null,
      "created_at": "2024-12-12T16:20:00Z"
    }
  ]
}
```

**Erreurs possibles:**

| Code | Description |
|------|-------------|
| 401 | Token manquant ou invalide |
| 400 | Paramètres de date invalides |

---

#### POST /reservations

Créer une nouvelle réservation pour un créneau disponible.

**Authentification:** Requise (Bearer Token)

**Paramètres du corps (JSON):**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `company_id` | integer | Oui | ID de l'entreprise |
| `date` | date | Oui | Date de la réservation (format: YYYY-MM-DD) |
| `time_start` | time | Oui | Heure de début (format: HH:MM) |
| `time_end` | time | Oui | Heure de fin (format: HH:MM) |
| `notes` | string | Non | Notes ou message pour l'entreprise (max: 500 caractères) |

**Exemple de requête:**
```json
{
  "company_id": 1,
  "date": "2024-12-15",
  "time_start": "10:00",
  "time_end": "11:00",
  "notes": "Je suis étudiant en BTS SIO et je recherche un stage en développement web. Je serais intéressé pour discuter des opportunités disponibles."
}
```

**Réponse:** `201 Created`
```json
{
  "id": 123,
  "message": "Reservation created successfully",
  "reservation": {
    "id": 123,
    "company_id": 1,
    "date": "2024-12-15",
    "time_start": "10:00:00",
    "time_end": "11:00:00",
    "status": "pending",
    "created_at": "2024-12-18T10:30:00Z"
  }
}
```

**Erreurs possibles:**

| Code | Description |
|------|-------------|
| 401 | Token manquant ou invalide |
| 400 | Données invalides ou manquantes |
| 404 | Entreprise non trouvée |
| 409 | Créneau déjà réservé ou indisponible |
| 409 | L'utilisateur a déjà une réservation à cette date/heure |

---

#### PUT /reservations/{id}

Modifier une réservation existante (uniquement si le statut est `pending`).

**Authentification:** Requise (Bearer Token)

**Paramètres d'URL:**

| Paramètre | Type | Description |
|-----------|------|-------------|
| `id` | integer | ID de la réservation |

**Paramètres du corps (JSON):**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `date` | date | Non | Nouvelle date (format: YYYY-MM-DD) |
| `time_start` | time | Non | Nouvelle heure de début (format: HH:MM) |
| `time_end` | time | Non | Nouvelle heure de fin (format: HH:MM) |
| `notes` | string | Non | Nouvelles notes |

**Exemple de requête:**
```json
{
  "date": "2024-12-16",
  "notes": "Modification : je préférerais discuter des opportunités en développement mobile."
}
```

**Réponse:** `200 OK`
```json
{
  "message": "Reservation updated successfully",
  "reservation": {
    "id": 123,
    "date": "2024-12-16",
    "time_start": "10:00:00",
    "time_end": "11:00:00",
    "notes": "Modification : je préférerais discuter des opportunités en développement mobile.",
    "updated_at": "2024-12-18T11:00:00Z"
  }
}
```

**Erreurs possibles:**

| Code | Description |
|------|-------------|
| 401 | Token manquant ou invalide |
| 403 | Modification non autorisée (réservation confirmée/annulée) |
| 404 | Réservation non trouvée |
| 409 | Nouveau créneau déjà réservé |

---

#### DELETE /reservations/{id}

Annuler une réservation existante.

**Authentification:** Requise (Bearer Token)

**Paramètres d'URL:**

| Paramètre | Type | Description |
|-----------|------|-------------|
| `id` | integer | ID de la réservation |

**Paramètres de requête (Query):**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `reason` | string | Non | Raison de l'annulation (max: 200 caractères) |

**Exemple de requête:**
```
DELETE /reservations/123?reason=Conflit d'emploi du temps
```

**Réponse:** `200 OK`
```json
{
  "message": "Reservation cancelled successfully",
  "reservation": {
    "id": 123,
    "status": "cancelled",
    "cancellation_reason": "Conflit d'emploi du temps",
    "cancelled_at": "2024-12-18T11:30:00Z"
  }
}
```

**Erreurs possibles:**

| Code | Description |
|------|-------------|
| 401 | Token manquant ou invalide |
| 404 | Réservation non trouvée |
| 403 | Annulation non autorisée (réservation déjà terminée) |

---

### Créneaux horaires

#### GET /companies/{id}/timeslots

Récupérer les créneaux horaires disponibles d'une entreprise pour une période donnée.

**Authentification:** Non requise

**Paramètres d'URL:**

| Paramètre | Type | Description |
|-----------|------|-------------|
| `id` | integer | ID de l'entreprise |

**Paramètres de requête (Query):**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `date` | date | Non | Date spécifique (format: YYYY-MM-DD) |
| `week` | integer | Non | Numéro de semaine (1-53) |
| `from_date` | date | Non | Date de début de période (format: YYYY-MM-DD) |
| `to_date` | date | Non | Date de fin de période (format: YYYY-MM-DD) |

!!! note
    Si aucun paramètre n'est fourni, retourne les créneaux de la semaine courante.

**Exemple de requête:**
```
GET /companies/1/timeslots?from_date=2024-12-15&to_date=2024-12-22
```

**Réponse:** `200 OK`
```json
{
  "company_id": 1,
  "company_name": "Tech Innovations SAS",
  "period": {
    "from": "2024-12-15",
    "to": "2024-12-22"
  },
  "timeslots": [
    {
      "date": "2024-12-16",
      "day_of_week": "monday",
      "start_time": "09:00:00",
      "end_time": "10:00:00",
      "available": true,
      "remaining_slots": 3
    },
    {
      "date": "2024-12-16",
      "day_of_week": "monday",
      "start_time": "10:00:00",
      "end_time": "11:00:00",
      "available": true,
      "remaining_slots": 2
    },
    {
      "date": "2024-12-16",
      "day_of_week": "monday",
      "start_time": "14:00:00",
      "end_time": "15:00:00",
      "available": false,
      "remaining_slots": 0
    },
    {
      "date": "2024-12-18",
      "day_of_week": "wednesday",
      "start_time": "09:00:00",
      "end_time": "10:00:00",
      "available": true,
      "remaining_slots": 1
    }
  ]
}
```

**Erreurs possibles:**

| Code | Description |
|------|-------------|
| 404 | Entreprise non trouvée |
| 400 | Paramètres de date invalides |

---

## Codes de réponse HTTP

| Code | Nom | Description | Utilisation |
|------|-----|-------------|-------------|
| 200 | OK | Requête réussie | GET, PUT, DELETE réussis |
| 201 | Created | Ressource créée | POST réussi (création) |
| 400 | Bad Request | Données invalides ou manquantes | Validation échouée |
| 401 | Unauthorized | Non authentifié | Token manquant ou invalide |
| 403 | Forbidden | Non autorisé | Action interdite pour ce rôle/statut |
| 404 | Not Found | Ressource introuvable | ID inexistant |
| 409 | Conflict | Conflit de données | Réservation en double, email existant |
| 500 | Internal Server Error | Erreur serveur | Erreur interne de l'API |

---

## Exemples d'utilisation

### Scénario 1 : Créer un compte et se connecter

#### Étape 1 : Inscription

```bash
curl -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "etudiant@exemple.fr",
    "password": "MotDePasse123!",
    "first_name": "Marie",
    "last_name": "Dupont",
    "role": "student",
    "school": "Lycée Technique Paris"
  }'
```

**Réponse:**
```json
{
  "message": "User created successfully",
  "user_id": 42
}
```

#### Étape 2 : Connexion

```bash
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "etudiant@exemple.fr",
    "password": "MotDePasse123!"
  }'
```

**Réponse:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 42,
    "email": "etudiant@exemple.fr",
    "first_name": "Marie",
    "last_name": "Dupont",
    "role": "student"
  }
}
```

---

### Scénario 2 : Rechercher une entreprise et réserver un créneau

#### Étape 1 : Lister les entreprises par secteur

```bash
curl -X GET "http://localhost:5000/api/companies?sector=Informatique&city=Paris"
```

**Réponse:**
```json
{
  "total": 12,
  "companies": [
    {
      "id": 5,
      "name": "Dev Solutions",
      "sector": "Informatique",
      "city": "Paris",
      "description": "Agence de développement web"
    }
  ]
}
```

#### Étape 2 : Voir les créneaux disponibles

```bash
curl -X GET "http://localhost:5000/api/companies/5/timeslots?from_date=2024-12-20&to_date=2024-12-27"
```

**Réponse:**
```json
{
  "company_id": 5,
  "timeslots": [
    {
      "date": "2024-12-20",
      "start_time": "14:00:00",
      "end_time": "15:00:00",
      "available": true,
      "remaining_slots": 2
    }
  ]
}
```

#### Étape 3 : Créer une réservation

```bash
curl -X POST http://localhost:5000/api/reservations \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." \
  -d '{
    "company_id": 5,
    "date": "2024-12-20",
    "time_start": "14:00",
    "time_end": "15:00",
    "notes": "Intéressé par un stage de développeur web"
  }'
```

**Réponse:**
```json
{
  "id": 78,
  "message": "Reservation created successfully",
  "reservation": {
    "id": 78,
    "status": "pending",
    "created_at": "2024-12-18T15:30:00Z"
  }
}
```

---

### Scénario 3 : Gérer ses réservations

#### Lister mes réservations

```bash
curl -X GET "http://localhost:5000/api/reservations?status=pending" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### Modifier une réservation

```bash
curl -X PUT http://localhost:5000/api/reservations/78 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." \
  -d '{
    "notes": "Je souhaiterais également discuter des opportunités de CDI"
  }'
```

#### Annuler une réservation

```bash
curl -X DELETE "http://localhost:5000/api/reservations/78?reason=Imprévu personnel" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

---

## Exemple JavaScript complet

```javascript
// Configuration
const API_BASE_URL = 'http://localhost:5000/api';
let authToken = null;

// 1. Inscription et connexion
async function login(email, password) {
  const response = await fetch(`${API_BASE_URL}/auth/login`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password })
  });

  if (!response.ok) {
    throw new Error('Login failed');
  }

  const data = await response.json();
  authToken = data.token;
  return data.user;
}

// 2. Récupérer les entreprises
async function getCompanies(filters = {}) {
  const params = new URLSearchParams(filters);
  const response = await fetch(`${API_BASE_URL}/companies?${params}`);

  if (!response.ok) {
    throw new Error('Failed to fetch companies');
  }

  return await response.json();
}

// 3. Récupérer les créneaux disponibles
async function getTimeslots(companyId, fromDate, toDate) {
  const params = new URLSearchParams({ from_date: fromDate, to_date: toDate });
  const response = await fetch(
    `${API_BASE_URL}/companies/${companyId}/timeslots?${params}`
  );

  if (!response.ok) {
    throw new Error('Failed to fetch timeslots');
  }

  return await response.json();
}

// 4. Créer une réservation
async function createReservation(reservationData) {
  const response = await fetch(`${API_BASE_URL}/reservations`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${authToken}`
    },
    body: JSON.stringify(reservationData)
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error || 'Failed to create reservation');
  }

  return await response.json();
}

// 5. Récupérer mes réservations
async function getMyReservations(status = null) {
  const params = status ? `?status=${status}` : '';
  const response = await fetch(`${API_BASE_URL}/reservations${params}`, {
    headers: {
      'Authorization': `Bearer ${authToken}`
    }
  });

  if (!response.ok) {
    throw new Error('Failed to fetch reservations');
  }

  return await response.json();
}

// Utilisation complète
async function completeBookingFlow() {
  try {
    // Connexion
    const user = await login('etudiant@exemple.fr', 'MotDePasse123!');
    console.log('Connecté:', user);

    // Rechercher des entreprises
    const { companies } = await getCompanies({
      sector: 'Informatique',
      city: 'Paris'
    });
    console.log(`${companies.length} entreprises trouvées`);

    // Voir les créneaux de la première entreprise
    const { timeslots } = await getTimeslots(
      companies[0].id,
      '2024-12-20',
      '2024-12-27'
    );
    console.log(`${timeslots.length} créneaux disponibles`);

    // Réserver le premier créneau disponible
    const availableSlot = timeslots.find(slot => slot.available);
    if (availableSlot) {
      const reservation = await createReservation({
        company_id: companies[0].id,
        date: availableSlot.date,
        time_start: availableSlot.start_time.slice(0, 5),
        time_end: availableSlot.end_time.slice(0, 5),
        notes: 'Recherche de stage en développement'
      });
      console.log('Réservation créée:', reservation);
    }

    // Afficher mes réservations
    const { reservations } = await getMyReservations();
    console.log(`J'ai ${reservations.length} réservation(s)`);

  } catch (error) {
    console.error('Erreur:', error.message);
  }
}

// Lancer le flux
completeBookingFlow();
```

---

## Rate Limiting

!!! warning "Limites de requêtes"
    L'API implémente un rate limiting pour éviter les abus:

    - **Authentification** : 5 tentatives par minute
    - **Endpoints publics** : 100 requêtes par minute
    - **Endpoints authentifiés** : 200 requêtes par minute

    En cas de dépassement, vous recevrez une erreur **429 Too Many Requests**.

---

## Support et contact

Pour toute question sur l'API :

- Consultez la [documentation complète](../index.md)
- Ouvrez une issue sur [GitHub](https://github.com/TISEPSE/Book-By-Click/issues)
- Contactez l'équipe technique
