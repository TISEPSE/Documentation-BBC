# Architecture Frontend

## Vue d'ensemble

Le frontend est une application **React 19** construite avec **Vite** et stylisée avec **Tailwind CSS 4**.

## Structure du projet

```text
src/
├── App.jsx                 # Composant racine avec routing
├── main.jsx                # Point d'entrée
├── pages/                  # Pages de l'application
│   ├── Home.jsx
│   ├── Login.jsx
│   ├── Recap.jsx
│   ├── Forgot_password.jsx
│   ├── Choice_Register.jsx
│   ├── Register_User.jsx      # Inscription étudiant
│   ├── Register_Pro.jsx       # Inscription entreprise
│   ├── Teste_form.jsx
│   └── dashboard_entreprise/
│       ├── Dashboard_Entreprise.jsx
│       └── sections/
│           ├── Dashboard_Content.jsx
│           ├── Calendar_Content.jsx
│           ├── Clients_Content.jsx
│           ├── Statistics_Content.jsx
│           └── Reservations_Content.jsx
├── components/             # Composants réutilisables
│   └── Header.jsx
├── Hook/                   # Custom hooks
│   └── useForm.jsx
├── assets/                 # Fichiers statiques
└── static/                 # Resources publiques
```

## Technologies

### React 19

Dernière version de React avec :

- Concurrent features
- Automatic batching
- Transitions API
- Server Components ready

### Vite

Build tool moderne offrant :

- Hot Module Replacement (HMR) ultra-rapide
- Build optimisé avec Rollup
- Support TypeScript natif
- CSS code splitting

### Tailwind CSS 4

Framework CSS utility-first :

- Configuration via Vite plugin
- JIT compiler
- Composants personnalisables
- Dark mode support

## Routing

### React Router v7

Configuration dans `App.jsx` :

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom'
import Home from './pages/Home'
import Login from './pages/Login'
import Dashboard from './pages/dashboard_entreprise/Dashboard_Entreprise'

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/login" element={<Login />} />
        <Route path="/register/user" element={<Register_User />} />
        <Route path="/register/pro" element={<Register_Pro />} />
        <Route path="/dashboard" element={<Dashboard />} />
        {/* ... autres routes */}
      </Routes>
    </BrowserRouter>
  )
}
```

### Routes principales

| Route | Composant | Description |
|-------|-----------|-------------|
| `/` | Home | Page d'accueil |
| `/login` | Login | Connexion |
| `/register/choice` | Choice_Register | Choix type inscription |
| `/register/user` | Register_User | Inscription étudiant |
| `/register/pro` | Register_Pro | Inscription entreprise |
| `/dashboard` | Dashboard_Entreprise | Dashboard entreprise |
| `/forgot-password` | Forgot_password | Mot de passe oublié |

## Composants

### Pages

#### Home.jsx

Page d'accueil de l'application.

```jsx
export default function Home() {
  return (
    <div className="min-h-screen bg-gray-50">
      <Header />
      <main>
        {/* Contenu */}
      </main>
    </div>
  )
}
```

#### Login.jsx

Formulaire de connexion avec validation.

#### Register_User.jsx / Register_Pro.jsx

Formulaires d'inscription pour :

- **User** : Étudiants
- **Pro** : Entreprises

#### Dashboard Entreprise

Dashboard principal avec navigation par sections :

- **Dashboard_Content** : Vue d'ensemble
- **Calendar_Content** : Calendrier des réservations
- **Clients_Content** : Liste des clients
- **Statistics_Content** : Statistiques
- **Reservations_Content** : Gestion des réservations

### Composants réutilisables

#### Header.jsx

Header de navigation avec menu.

```jsx
export default function Header() {
  return (
    <header className="bg-white shadow">
      <nav className="container mx-auto px-4">
        {/* Navigation */}
      </nav>
    </header>
  )
}
```

## Custom Hooks

### useForm.jsx

Hook personnalisé pour la gestion des formulaires.

**Fonctionnalités** :

- Gestion de l'état des champs
- Validation
- Gestion de la soumission
- Reset du formulaire

**Utilisation** :

```jsx
import { useForm } from './Hook/useForm'

function LoginForm() {
  const { values, errors, handleChange, handleSubmit } = useForm({
    initialValues: {
      email: '',
      password: ''
    },
    onSubmit: async (values) => {
      // Logique de soumission
    },
    validate: (values) => {
      const errors = {}
      if (!values.email) errors.email = 'Email requis'
      if (!values.password) errors.password = 'Mot de passe requis'
      return errors
    }
  })

  return (
    <form onSubmit={handleSubmit}>
      {/* Champs du formulaire */}
    </form>
  )
}
```

## Gestion de l'état

### Local State

Utilisation de `useState` pour l'état local des composants.

```jsx
const [isOpen, setIsOpen] = useState(false)
const [data, setData] = useState([])
```

### Effects

Utilisation de `useEffect` pour les side effects.

```jsx
useEffect(() => {
  // Fetch data au montage
  fetchData()
}, [])
```

### Context API (potentiel)

Pour l'état global (authentification, thème) :

```jsx
// AuthContext.jsx
const AuthContext = createContext()

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null)

  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {children}
    </AuthContext.Provider>
  )
}
```

## API Integration

### Configuration

Base URL de l'API :

```javascript
const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:5000'
```

### Fetch wrapper

```javascript
async function api(endpoint, options = {}) {
  const token = localStorage.getItem('token')

  const config = {
    headers: {
      'Content-Type': 'application/json',
      ...(token && { 'Authorization': `Bearer ${token}` })
    },
    ...options
  }

  const response = await fetch(`${API_URL}${endpoint}`, config)

  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`)
  }

  return response.json()
}
```

### Utilisation

```jsx
// Dans un composant
useEffect(() => {
  api('/api/reservations')
    .then(data => setReservations(data))
    .catch(error => console.error(error))
}, [])
```

## Styling avec Tailwind

### Configuration

Dans `tailwind.config.js` :

```javascript
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,jsx}"
  ],
  theme: {
    extend: {
      colors: {
        primary: '#3b82f6',
        secondary: '#8b5cf6'
      }
    }
  }
}
```

### Classes utilitaires

```jsx
<div className="flex items-center justify-between p-4 bg-white shadow-md rounded-lg">
  <h1 className="text-2xl font-bold text-gray-900">Titre</h1>
  <button className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">
    Action
  </button>
</div>
```

### Responsive

```jsx
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {/* Grid responsive */}
</div>
```

## Build & Optimisation

### Développement

```bash
npm run dev
```

Vite démarre un serveur de développement avec HMR sur le port 5173.

### Production

```bash
npm run build
```

Génère les fichiers optimisés dans `dist/` :

- Minification
- Tree shaking
- Code splitting
- Asset optimization

### Preview

```bash
npm run preview
```

Prévisualise le build de production localement.

## Performance

### Code Splitting

```jsx
import { lazy, Suspense } from 'react'

const Dashboard = lazy(() => import('./pages/dashboard_entreprise/Dashboard_Entreprise'))

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Dashboard />
    </Suspense>
  )
}
```

### Mémoïsation

```jsx
import { useMemo, useCallback } from 'react'

const expensiveValue = useMemo(() => computeExpensive(data), [data])
const handleClick = useCallback(() => doSomething(), [])
```

## Tests

### Configuration

```bash
npm install --save-dev vitest @testing-library/react @testing-library/jest-dom
```

### Exemple de test

```jsx
import { render, screen } from '@testing-library/react'
import { describe, it, expect } from 'vitest'
import Header from './components/Header'

describe('Header', () => {
  it('renders navigation', () => {
    render(<Header />)
    expect(screen.getByRole('navigation')).toBeInTheDocument()
  })
})
```

## Bonnes pratiques

1. **Composants fonctionnels** avec hooks
2. **PropTypes** ou TypeScript pour la validation
3. **Nomenclature** : PascalCase pour composants
4. **Structure** : Un composant par fichier
5. **Séparation** : Logique vs présentation
6. **Accessibilité** : Attributs ARIA, semantic HTML
7. **Performance** : Mémoïsation, code splitting

## Prochaines étapes

- [Backend Flask](backend.md)
- [Base de données](database.md)
- [Guide de développement](../development/getting-started.md)
