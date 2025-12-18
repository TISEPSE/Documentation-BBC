# Exemples d'utilisation de l'API

Cette page fournit des exemples pratiques d'utilisation de l'API Book By Click dans différents langages et frameworks.

## Exemples Python

### Configuration de base

```python
import requests
import json

class BookByClickAPI:
    def __init__(self, base_url="http://localhost:5000/api"):
        self.base_url = base_url
        self.token = None

    def login(self, email, password):
        """Connexion et récupération du token"""
        response = requests.post(
            f"{self.base_url}/auth/login",
            json={"email": email, "password": password}
        )
        response.raise_for_status()
        data = response.json()
        self.token = data["token"]
        return data["user"]

    def get_headers(self):
        """Headers avec authentification"""
        headers = {"Content-Type": "application/json"}
        if self.token:
            headers["Authorization"] = f"Bearer {self.token}"
        return headers

    def get_companies(self, city=None, sector=None):
        """Récupérer la liste des entreprises"""
        params = {}
        if city:
            params["city"] = city
        if sector:
            params["sector"] = sector

        response = requests.get(
            f"{self.base_url}/companies",
            params=params,
            headers=self.get_headers()
        )
        response.raise_for_status()
        return response.json()

    def get_timeslots(self, company_id, from_date, to_date):
        """Récupérer les créneaux disponibles"""
        response = requests.get(
            f"{self.base_url}/companies/{company_id}/timeslots",
            params={"from_date": from_date, "to_date": to_date},
            headers=self.get_headers()
        )
        response.raise_for_status()
        return response.json()

    def create_reservation(self, company_id, date, time_start, time_end, notes=""):
        """Créer une réservation"""
        response = requests.post(
            f"{self.base_url}/reservations",
            json={
                "company_id": company_id,
                "date": date,
                "time_start": time_start,
                "time_end": time_end,
                "notes": notes
            },
            headers=self.get_headers()
        )
        response.raise_for_status()
        return response.json()

    def get_my_reservations(self, status=None):
        """Récupérer mes réservations"""
        params = {}
        if status:
            params["status"] = status

        response = requests.get(
            f"{self.base_url}/reservations",
            params=params,
            headers=self.get_headers()
        )
        response.raise_for_status()
        return response.json()
```

### Utilisation complète

```python
# Initialiser l'API
api = BookByClickAPI()

try:
    # 1. Connexion
    user = api.login("etudiant@exemple.fr", "MotDePasse123!")
    print(f"Connecté en tant que: {user['first_name']} {user['last_name']}")

    # 2. Rechercher des entreprises
    companies = api.get_companies(city="Paris", sector="Informatique")
    print(f"Trouvé {companies['total']} entreprises")

    # 3. Voir les créneaux disponibles
    if companies['companies']:
        company = companies['companies'][0]
        timeslots = api.get_timeslots(
            company['id'],
            "2024-12-20",
            "2024-12-27"
        )
        print(f"Créneaux disponibles: {len(timeslots['timeslots'])}")

        # 4. Créer une réservation
        if timeslots['timeslots']:
            slot = timeslots['timeslots'][0]
            if slot['available']:
                reservation = api.create_reservation(
                    company_id=company['id'],
                    date=slot['date'],
                    time_start=slot['start_time'][:5],
                    time_end=slot['end_time'][:5],
                    notes="Recherche de stage en développement web"
                )
                print(f"Réservation créée: ID {reservation['id']}")

    # 5. Voir mes réservations
    my_reservations = api.get_my_reservations(status="pending")
    print(f"Réservations en attente: {my_reservations['total']}")

except requests.exceptions.HTTPError as e:
    print(f"Erreur HTTP: {e}")
except Exception as e:
    print(f"Erreur: {e}")
```

---

## Exemples React/JavaScript

### Service API complet

```javascript
// services/api.js
class APIService {
  constructor(baseURL = 'http://localhost:5000/api') {
    this.baseURL = baseURL;
    this.token = localStorage.getItem('authToken');
  }

  async request(endpoint, options = {}) {
    const headers = {
      'Content-Type': 'application/json',
      ...options.headers,
    };

    if (this.token) {
      headers['Authorization'] = `Bearer ${this.token}`;
    }

    const config = {
      ...options,
      headers,
    };

    try {
      const response = await fetch(`${this.baseURL}${endpoint}`, config);

      if (!response.ok) {
        const error = await response.json();
        throw new Error(error.error || `HTTP ${response.status}`);
      }

      return await response.json();
    } catch (error) {
      console.error('API Error:', error);
      throw error;
    }
  }

  // Authentification
  async login(email, password) {
    const data = await this.request('/auth/login', {
      method: 'POST',
      body: JSON.stringify({ email, password }),
    });

    this.token = data.token;
    localStorage.setItem('authToken', data.token);
    return data;
  }

  async register(userData) {
    return await this.request('/auth/register', {
      method: 'POST',
      body: JSON.stringify(userData),
    });
  }

  logout() {
    this.token = null;
    localStorage.removeItem('authToken');
  }

  // Entreprises
  async getCompanies(filters = {}) {
    const params = new URLSearchParams(filters);
    return await this.request(`/companies?${params}`);
  }

  async getCompanyDetails(companyId) {
    return await this.request(`/companies/${companyId}`);
  }

  // Créneaux horaires
  async getTimeslots(companyId, filters = {}) {
    const params = new URLSearchParams(filters);
    return await this.request(`/companies/${companyId}/timeslots?${params}`);
  }

  // Réservations
  async getMyReservations(filters = {}) {
    const params = new URLSearchParams(filters);
    return await this.request(`/reservations?${params}`);
  }

  async createReservation(reservationData) {
    return await this.request('/reservations', {
      method: 'POST',
      body: JSON.stringify(reservationData),
    });
  }

  async updateReservation(reservationId, updates) {
    return await this.request(`/reservations/${reservationId}`, {
      method: 'PUT',
      body: JSON.stringify(updates),
    });
  }

  async cancelReservation(reservationId, reason) {
    return await this.request(
      `/reservations/${reservationId}?reason=${encodeURIComponent(reason)}`,
      { method: 'DELETE' }
    );
  }

  // Profil utilisateur
  async getProfile() {
    return await this.request('/users/me');
  }

  async updateProfile(updates) {
    return await this.request('/users/me', {
      method: 'PUT',
      body: JSON.stringify(updates),
    });
  }
}

export default new APIService();
```

### Hook React personnalisé

```javascript
// hooks/useAPI.js
import { useState, useEffect } from 'react';
import api from '../services/api';

export function useCompanies(filters = {}) {
  const [companies, setCompanies] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function fetchCompanies() {
      try {
        setLoading(true);
        const data = await api.getCompanies(filters);
        setCompanies(data.companies || []);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    fetchCompanies();
  }, [JSON.stringify(filters)]);

  return { companies, loading, error };
}

export function useReservations(filters = {}) {
  const [reservations, setReservations] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const refresh = async () => {
    try {
      setLoading(true);
      const data = await api.getMyReservations(filters);
      setReservations(data.reservations || []);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    refresh();
  }, [JSON.stringify(filters)]);

  return { reservations, loading, error, refresh };
}
```

### Composant React exemple

```javascript
// components/CompanyList.jsx
import React from 'react';
import { useCompanies } from '../hooks/useAPI';

export default function CompanyList({ city, sector }) {
  const { companies, loading, error } = useCompanies({ city, sector });

  if (loading) {
    return <div className="text-center p-8">Chargement...</div>;
  }

  if (error) {
    return (
      <div className="bg-red-100 text-red-700 p-4 rounded">
        Erreur: {error}
      </div>
    );
  }

  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      {companies.map(company => (
        <div key={company.id} className="bg-white shadow rounded-lg p-6">
          <h3 className="text-xl font-bold">{company.name}</h3>
          <p className="text-gray-600">{company.city}</p>
          <p className="mt-2">{company.description}</p>
          <button
            className="mt-4 bg-blue-500 text-white px-4 py-2 rounded"
            onClick={() => window.location.href = `/companies/${company.id}`}
          >
            Voir les créneaux
          </button>
        </div>
      ))}
    </div>
  );
}
```

---

## Gestion des erreurs

### Python

```python
try:
    reservation = api.create_reservation(...)
except requests.exceptions.HTTPError as e:
    if e.response.status_code == 401:
        print("Non authentifié - veuillez vous reconnecter")
    elif e.response.status_code == 409:
        print("Créneau déjà réservé")
    else:
        print(f"Erreur: {e.response.json().get('error')}")
except Exception as e:
    print(f"Erreur inattendue: {e}")
```

### JavaScript

```javascript
try {
  const reservation = await api.createReservation(data);
  console.log('Réservation créée:', reservation);
} catch (error) {
  if (error.message.includes('401')) {
    // Rediriger vers login
    window.location.href = '/login';
  } else if (error.message.includes('409')) {
    alert('Ce créneau est déjà réservé');
  } else {
    alert('Erreur: ' + error.message);
  }
}
```

---

## Tests unitaires

### Python avec pytest

```python
# tests/test_api.py
import pytest
from api_client import BookByClickAPI

@pytest.fixture
def api():
    return BookByClickAPI(base_url="http://localhost:5000/api")

def test_login(api):
    user = api.login("test@example.com", "password")
    assert user["email"] == "test@example.com"
    assert api.token is not None

def test_get_companies(api):
    api.login("test@example.com", "password")
    companies = api.get_companies(city="Paris")
    assert "companies" in companies
    assert isinstance(companies["companies"], list)

def test_create_reservation(api):
    api.login("test@example.com", "password")
    reservation = api.create_reservation(
        company_id=1,
        date="2024-12-20",
        time_start="10:00",
        time_end="11:00"
    )
    assert "id" in reservation
```

### JavaScript avec Jest

```javascript
// __tests__/api.test.js
import APIService from '../services/api';

describe('API Service', () => {
  let api;

  beforeEach(() => {
    api = new APIService();
  });

  test('login returns user and token', async () => {
    const result = await api.login('test@example.com', 'password');
    expect(result).toHaveProperty('token');
    expect(result).toHaveProperty('user');
  });

  test('getCompanies returns list', async () => {
    await api.login('test@example.com', 'password');
    const companies = await api.getCompanies({ city: 'Paris' });
    expect(companies).toHaveProperty('companies');
    expect(Array.isArray(companies.companies)).toBe(true);
  });
});
```

---

## Ressources supplémentaires

- [Documentation API complète](endpoints.md)
- [Guide d'architecture](../architecture/overview.md)
- [Guide de développement](../development/getting-started.md)
