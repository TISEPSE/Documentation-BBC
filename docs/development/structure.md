# Structure du projet

## Vue d'ensemble

```text
Projet-BTS-Alan-Maxime-Tisba/
├── Backend/                # API Flask
├── src/                    # Application React
├── Docs/                   # Documentation
├── package.json            # Config npm
└── vite.config.js         # Config Vite
```

## Frontend (src/)

```text
src/
├── App.jsx                         # Composant racine + routing
├── main.jsx                        # Point d'entrée
│
├── pages/                          # Pages de l'application
│   ├── Home.jsx                   # Page d'accueil
│   ├── Login.jsx                  # Connexion
│   ├── Recap.jsx                  # Récapitulatif
│   ├── Forgot_password.jsx        # Mot de passe oublié
│   ├── Choice_Register.jsx        # Choix type inscription
│   ├── Register_User.jsx          # Inscription étudiant
│   ├── Register_Pro.jsx           # Inscription entreprise
│   ├── Teste_form.jsx             # Test formulaire
│   │
│   └── dashboard_entreprise/      # Dashboard entreprise
│       ├── Dashboard_Entreprise.jsx
│       └── sections/
│           ├── Dashboard_Content.jsx    # Vue d'ensemble
│           ├── Calendar_Content.jsx     # Calendrier
│           ├── Clients_Content.jsx      # Gestion clients
│           ├── Statistics_Content.jsx   # Statistiques
│           └── Reservations_Content.jsx # Gestion réservations
│
├── components/                     # Composants réutilisables
│   └── Header.jsx                 # En-tête navigation
│
├── Hook/                           # Custom hooks
│   └── useForm.jsx                # Hook formulaire
│
├── assets/                         # Images, fonts
└── static/                         # Ressources publiques
```

### Organisation recommandée

Pour une meilleure scalabilité :

```text
src/
├── features/              # Organisation par feature
│   ├── auth/
│   │   ├── Login.jsx
│   │   ├── Register.jsx
│   │   └── hooks/
│   │       └── useAuth.js
│   │
│   ├── reservations/
│   │   ├── ReservationList.jsx
│   │   ├── ReservationForm.jsx
│   │   └── hooks/
│   │       └── useReservations.js
│   │
│   └── dashboard/
│       └── ...
│
├── shared/                # Code partagé
│   ├── components/
│   ├── hooks/
│   ├── utils/
│   └── api/
│
└── styles/               # Styles globaux
```

## Backend (Backend/)

### Structure actuelle

```text
Backend/
├── app.py              # Application Flask
├── db/                 # Base de données
│   └── docker-compose.yml
└── __pycache__/        # Cache Python
```

### Structure recommandée

```text
Backend/
├── app.py              # Point d'entrée
├── config.py           # Configuration
├── requirements.txt    # Dépendances
├── .env               # Variables d'environnement
│
├── routes/            # Routes API
│   ├── __init__.py
│   ├── auth.py       # /api/auth/*
│   ├── users.py      # /api/users/*
│   ├── companies.py  # /api/companies/*
│   └── reservations.py # /api/reservations/*
│
├── models/            # Modèles de données
│   ├── __init__.py
│   ├── user.py
│   ├── company.py
│   └── reservation.py
│
├── services/          # Logique métier
│   ├── __init__.py
│   ├── auth_service.py
│   ├── reservation_service.py
│   └── email_service.py
│
├── utils/             # Utilitaires
│   ├── __init__.py
│   ├── database.py   # Connexion DB
│   ├── validators.py # Validations
│   └── decorators.py # Decorators
│
├── tests/             # Tests
│   ├── test_auth.py
│   └── test_reservations.py
│
└── db/
    ├── docker-compose.yml
    ├── init.sql       # Script d'initialisation
    └── migrations/    # Migrations DB
```

## Configuration

### package.json

```json
{
  "name": "react-with-flask",
  "version": "0.0.0",
  "scripts": {
    "dev": "vite",
    "dev:lin": "concurrently ...",
    "dev:win": "concurrently ...",
    "dev:all": "concurrently ...",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^19.2.0",
    "react-router-dom": "^7.9.6",
    "tailwindcss": "^4.1.17",
    "@heroicons/react": "^2.2.0"
  }
}
```

### vite.config.js

```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [react(), tailwindcss()],
  server: {
    port: 5173,
    proxy: {
      '/api': 'http://localhost:5000'
    }
  }
})
```

## Patterns

### Composants

```jsx
// Composant fonctionnel simple
export default function MyComponent({ prop1, prop2 }) {
  return <div>{prop1}</div>
}

// Avec hooks
export default function MyComponent() {
  const [state, setState] = useState(null)

  useEffect(() => {
    // Side effect
  }, [])

  return <div>{state}</div>
}
```

### Services

```python
# Service pattern
class UserService:
    @staticmethod
    def get_user(user_id):
        # Logic
        pass

    @staticmethod
    def create_user(data):
        # Logic
        pass
```

### Routes

```python
# Blueprint pattern
auth_bp = Blueprint('auth', __name__)

@auth_bp.route('/login', methods=['POST'])
def login():
    # Logic
    pass

# Dans app.py
app.register_blueprint(auth_bp, url_prefix='/api/auth')
```

## Modules tiers

### Frontend

- `react` - Framework UI
- `react-router-dom` - Routing
- `tailwindcss` - CSS framework
- `@heroicons/react` - Icônes

### Backend

- `flask` - Framework web
- `flask-cors` - CORS
- `mysql-connector-python` - MySQL driver
- `python-dotenv` - Variables d'env
- `pyjwt` - JWT tokens
- `werkzeug` - Security utilities

## Prochaines étapes

- [Guide de démarrage](getting-started.md)
- [Contribution](contributing.md)
