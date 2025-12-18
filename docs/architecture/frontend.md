# Architecture Frontend

## Vue d'ensemble

Le frontend est une application **React 19.2.0** construite avec **Vite (Rolldown-Vite 7.2.5)** et stylisée avec **Tailwind CSS 4.1.17**.

## Structure du projet

```text
src/
├── App.jsx                 # Composant racine avec routing (11 routes)
├── main.jsx                # Point d'entrée React
├── pages/                  # Pages de l'application (11 pages)
│   ├── Home.jsx
│   ├── Login.jsx
│   ├── Register_User.jsx
│   ├── Register_Pro.jsx
│   ├── Choice_Register.jsx
│   ├── Forgot_password.jsx
│   ├── Contact.jsx
│   ├── PageResultatRecherche.jsx
│   ├── Teste_form.jsx
│   ├── Recap.jsx
│   └── dashboard/
│       ├── Dashboard_Entreprise.jsx
│       └── sections/
│           ├── Dashboard_Content.jsx
│           ├── Calendar_Content.jsx
│           ├── Clients_Content.jsx
│           └── Statistics_Content.jsx
├── components/             # Composants réutilisables (9 composants)
│   ├── Header.jsx
│   ├── Calendar.jsx
│   ├── MonthView.jsx
│   ├── WeekView.jsx
│   ├── ViewDropdown.jsx
│   ├── NavigationButtons.jsx
│   ├── UserAvatar.jsx
│   ├── reservationClient.jsx
│   └── forms.jsx
├── Hook/                   # Custom hooks
│   └── useCalendar.jsx
└── static/                 # Fichiers CSS
    ├── App.css
    └── index.css
```

## Technologies

### React 19.2.0

Dernière version de React avec :

- Concurrent features
- Automatic batching
- Server Components ready
- StrictMode activé

### Vite (Rolldown-Vite 7.2.5)

Build tool moderne offrant :

- Hot Module Replacement (HMR) ultra-rapide
- Build optimisé avec Rolldown
- Support ES modules natif
- CSS code splitting

### Tailwind CSS 4.1.17

Framework CSS utility-first :

- Configuration via `@tailwindcss/vite` plugin
- JIT compiler
- Composants personnalisables
- Design system cohérent

### Bibliothèques tierces

#### Icônes
- **@heroicons/react 2.2.0** : Icônes Heroicons (utilisées dans le Header)
- **lucide-react 0.555.0** : Icônes Lucide (utilisées dans Statistics, UserAvatar)

#### Date et Calendrier
- **date-fns 4.1.0** : Manipulation de dates
- **react-big-calendar 1.19.4** : Composant calendrier (non encore intégré)

#### Graphiques
- **recharts 3.5.1** : Bibliothèque de graphiques pour Statistics_Content
  - LineChart : Évolution du CA
  - BarChart : Réservations par service
  - PieChart : Répartition CA par service
  - AreaChart : Taux de remplissage
  - RadarChart : Satisfaction clients

#### Styles
- **styled-components 6.1.19** : CSS-in-JS (optionnel)

## Routing

### React Router DOM 7.10.1

Configuration dans `/home/baptiste/VsCode/Book-By-Click-Source/src/App.jsx` :

```jsx
import {BrowserRouter, Routes, Route} from "react-router-dom"

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/login_form" element={<Login />} />
        <Route path="/register_choice" element={<Choice_Register />} />
        <Route path="/register_form_user" element={<Resgister_user />} />
        <Route path="/register_form_pro" element={<Resgister_pro />} />
        <Route path="/fogot_password" element={<Forgot_password />} />
        <Route path="/dashboard_entreprise" element={<Dashboard_Entreprise />}/>
        <Route path="/contact" element={<Contact />} />
        <Route path="/dashboard" element={<Dashboard_Entreprise />} />
        <Route path="/result" element={<ResultatRecherhce />} />
        <Route path="/teste" element={<Test />} />
        <Route path="/recap" element={<Recap />} />
      </Routes>
    </BrowserRouter>
  )
}
```

### Routes définies

| Route | Composant | Description |
|-------|-----------|-------------|
| `/` | Home | Page d'accueil |
| `/login_form` | Login | Connexion utilisateur |
| `/register_choice` | Choice_Register | Choix du type d'inscription |
| `/register_form_user` | Register_User | Inscription étudiant |
| `/register_form_pro` | Register_Pro | Inscription entreprise |
| `/fogot_password` | Forgot_password | Récupération mot de passe |
| `/contact` | Contact | Page de contact |
| `/dashboard` | Dashboard_Entreprise | Dashboard entreprise (alias) |
| `/dashboard_entreprise` | Dashboard_Entreprise | Dashboard entreprise |
| `/result` | PageResultatRecherche | Résultats de recherche |
| `/teste` | Teste_form | Page de test |
| `/recap` | Recap | Récapitulatif |

## Composants

### Pages principales

#### Home.jsx

Page d'accueil de l'application (34KB - page complexe).

#### Login.jsx

Formulaire de connexion avec :
- Champs email et mot de passe
- Validation côté client
- Lien vers inscription et récupération mot de passe

#### Register_User.jsx / Register_Pro.jsx

Formulaires d'inscription avec :
- **Register_User** : Inscription pour les étudiants/utilisateurs
- **Register_Pro** : Inscription pour les entreprises/professionnels
- Intégration OAuth Google
- Validation des champs

#### Choice_Register.jsx

Page de choix du type d'inscription (7KB - interface de sélection).

#### Contact.jsx

Page de contact pour communiquer avec l'équipe.

#### PageResultatRecherche.jsx

Affichage des résultats de recherche (18KB - page complexe avec filtres et résultats).

#### Dashboard Entreprise

Le dashboard entreprise utilise un système de navigation par sections avec état local.

**Structure** (`/home/baptiste/VsCode/Book-By-Click-Source/src/pages/dashboard/Dashboard_Entreprise.jsx`) :

```jsx
import { useState } from 'react'
import Header from '../../components/Header'
import DashboardContent from './sections/Dashboard_Content'
import CalendarContent from './sections/Calendar_Content'
import ClientsContent from './sections/Clients_Content'
import StatisticsContent from './sections/Statistics_Content'

const Dashboard_entreprises = () => {
  const [activeSection, setActiveSection] = useState('dashboard')

  const renderContent = () => {
    switch(activeSection) {
      case 'dashboard': return <DashboardContent />
      case 'calendar': return <CalendarContent />
      case 'clients': return <ClientsContent />
      case 'statistics': return <StatisticsContent />
      default: return <DashboardContent />
    }
  }

  return (
    <div className="min-h-screen bg-gray-50">
      <Header
        activeSection={activeSection}
        setActiveSection={setActiveSection}
      />
      <main className="w-full mx-auto px-6 py-8">
        <div className="max-w-7xl mx-auto">
          {renderContent()}
        </div>
      </main>
    </div>
  )
}
```

**Sections disponibles** :

- **Dashboard_Content** : Vue d'ensemble avec KPIs
- **Calendar_Content** : Calendrier des réservations
- **Clients_Content** : Liste et gestion des clients
- **Statistics_Content** : Statistiques et graphiques détaillés (22KB)
  - Évolution du CA (LineChart)
  - Répartition par service (PieChart)
  - Réservations par service (BarChart)
  - Taux de remplissage (AreaChart)
  - Satisfaction clients (RadarChart)
  - Objectifs vs Réalisation

### Composants réutilisables

#### Header.jsx

Header de navigation avec :
- Logo et branding "Book By Click"
- Icône calendrier (@heroicons/react)
- Barre de recherche
- Navigation responsive
- Menu burger pour mobile
- Avatar utilisateur (UserAvatar)
- Onglets de navigation pour le dashboard :
  - Tableau de bord
  - Calendrier
  - Réservations
  - Clients
  - Statistiques

**Props** :
- `activeSection` : Section active actuelle
- `setActiveSection` : Fonction pour changer de section

#### UserAvatar.jsx

Avatar utilisateur avec dropdown menu :

```jsx
export default function UserAvatarSimple({
  user = {name: "Utilisateur", email: "user@bookbyclick.com"}
}) {
  // Génération des initiales
  // Menu déroulant avec :
  // - Mon profil
  // - Paramètres
  // - Aide
  // - Déconnexion
}
```

**Fonctionnalités** :
- Affichage des initiales
- Indicateur de présence (point vert)
- Menu déroulant responsive
- Fermeture automatique au clic extérieur
- Icônes Lucide React

#### Calendar.jsx

Composant calendrier principal utilisant le hook `useCalendar` :

```jsx
export default function Calendar() {
  const {
    date,
    view,
    events,
    setDate,
    setView,
    handlePrevious,
    handleNext,
    handleToday,
  } = useCalendar()

  return (
    <>
      <div className="flex items-center justify-between mb-6">
        <div>
          <h2>{format(date, "MMMM yyyy", {locale: fr})}</h2>
          <p>Semaine {format(date, "w", {locale: fr})}</p>
        </div>
        <div className="flex items-center gap-3">
          <NavigationButtons />
          <ViewDropdown view={view} onViewChange={setView} />
        </div>
      </div>
      {view === "week" ? <WeekView /> : <MonthView />}
    </>
  )
}
```

**Sous-composants** :
- **MonthView** : Vue mensuelle du calendrier
- **WeekView** : Vue hebdomadaire du calendrier
- **NavigationButtons** : Boutons Précédent/Aujourd'hui/Suivant
- **ViewDropdown** : Sélecteur de vue (mois/semaine)

#### forms.jsx

Composant de formulaire générique avec :
- Champs texte, email, password
- Validation des champs
- Intégration OAuth Google
- Soumission vers API Flask

## Custom Hooks

### useCalendar.jsx

Hook personnalisé pour la gestion du calendrier :

```jsx
export function useCalendar() {
  const [date, setDate] = useState(new Date())
  const [view, setView] = useState("month")
  const events = []

  const handlePrevious = () => {
    if (view === "month") {
      setDate(subMonths(date, 1))
    } else {
      setDate(subWeeks(date, 1))
    }
  }

  const handleNext = () => {
    if (view === "month") {
      setDate(addMonths(date, 1))
    } else {
      setDate(addWeeks(date, 1))
    }
  }

  const handleToday = () => {
    setDate(new Date())
  }

  return {
    date,
    view,
    events,
    setDate,
    setView,
    handlePrevious,
    handleNext,
    handleToday,
  }
}
```

**Fonctionnalités** :
- Gestion de la date courante
- Changement de vue (mois/semaine)
- Navigation temporelle
- Gestion des événements (à implémenter)

## Gestion de l'état

### Local State

Utilisation de `useState` pour l'état local des composants :

```jsx
const [activeSection, setActiveSection] = useState('dashboard')
const [dateRange, setDateRange] = useState("30jours")
const [isOpen, setIsOpen] = useState(false)
```

### Effects

Utilisation de `useEffect` pour les side effects :

```jsx
useEffect(() => {
  const handleClickOutside = e => {
    if (dropdownRef.current && !dropdownRef.current.contains(e.target)) {
      setIsOpen(false)
    }
  }
  document.addEventListener("mousedown", handleClickOutside)
  return () => document.removeEventListener("mousedown", handleClickOutside)
}, [])
```

## Styling avec Tailwind

### Classes utilitaires

Le projet utilise massivement Tailwind CSS avec :

```jsx
<div className="min-h-screen bg-gray-50">
  <header className="text-base lg:text-sm bg-white border-b border-gray-200">
    <button className="w-full px-4 py-2 text-white font-medium bg-indigo-600 hover:bg-indigo-500 active:bg-indigo-600 rounded-lg duration-150">
      Action
    </button>
  </header>
</div>
```

### Responsive Design

Breakpoints utilisés :
- `sm:` - Small (640px+)
- `md:` - Medium (768px+)
- `lg:` - Large (1024px+)
- `xl:` - Extra Large (1280px+)

Exemple :

```jsx
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
  <div className="hidden lg:flex lg:flex-1 lg:items-center lg:gap-6">
    {/* Contenu desktop uniquement */}
  </div>
</div>
```

### Palette de couleurs

Couleurs principales utilisées :
- **Indigo** (`indigo-600`, `indigo-700`) : Couleur principale de marque
- **Gray** : Backgrounds et textes (`gray-50`, `gray-100`, `gray-200`, etc.)
- **Emerald** : Succès et positif (`emerald-600`, `emerald-100`)
- **Red** : Erreurs et danger (`red-600`, `red-100`)
- **Blue** : Informations (`blue-600`, `blue-100`)
- **Amber/Yellow** : Warnings et évaluations (`amber-600`, `yellow-100`)

## Scripts npm

### Développement

```bash
# Frontend uniquement
npm run dev

# Frontend + API (Linux)
npm run dev:lin

# Frontend + API (Windows)
npm run dev:win

# Database + API + Frontend
npm run dev:all
```

**Script `dev:lin`** :
```json
"concurrently --kill-others-on-fail false \"npm run api\" \"npm run web\""
```

**Script `dev:win`** :
```json
"concurrently --kill-others-on-fail false \"npm run api:win\" \"npm run web\""
```

### Production

```bash
# Build
npm run build

# Preview
npm run preview
```

### Linting

```bash
npm run lint
```

Configuration ESLint :
- `@eslint/js 9.39.1`
- `eslint-plugin-react-hooks 7.0.1`
- `eslint-plugin-react-refresh 0.4.24`

## Build & Optimisation

### Développement

Le serveur de développement Vite démarre sur le port **5173** (par défaut) avec :
- Hot Module Replacement (HMR)
- Fast Refresh pour React
- CSS HMR

### Production

```bash
npm run build
```

Génère les fichiers optimisés dans `dist/` :
- Minification JavaScript et CSS
- Tree shaking automatique
- Code splitting
- Asset optimization
- Hash des fichiers pour cache busting

### Configuration Vite

**Plugins utilisés** :
- `@vitejs/plugin-react 5.1.1` : Support React avec Fast Refresh
- `@tailwindcss/vite 4.1.17` : Intégration Tailwind CSS

**Build tool** :
- Vite utilise `rolldown-vite@7.2.5` (override dans package.json)

## Intégration API

### Configuration

L'application communique avec le backend Flask :

```javascript
// Action de formulaire (forms.jsx)
<form action="http://localhost:5000/register_form" method="POST">
```

**Note** : L'intégration API utilise actuellement des formulaires HTML standards. Une refonte avec `fetch` ou `axios` est recommandée.

### Variables d'environnement

Configuration suggérée dans `.env` :

```bash
VITE_API_URL=http://localhost:5000
```

Utilisation :

```javascript
const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:5000'
```

## Performance

### Optimisations implémentées

- **Composants fonctionnels** : Tous les composants utilisent les hooks
- **Code splitting** : Lazy loading possible (à implémenter)
- **CSS optimisé** : Tailwind JIT pour CSS minimal
- **Asset optimization** : Vite optimise automatiquement les assets

### Optimisations suggérées

1. **Lazy loading des routes** :

```jsx
import { lazy, Suspense } from 'react'

const Dashboard = lazy(() => import('./pages/dashboard/Dashboard_Entreprise'))

<Suspense fallback={<div>Loading...</div>}>
  <Route path="/dashboard" element={<Dashboard />} />
</Suspense>
```

2. **Mémoïsation** :

```jsx
import { useMemo, useCallback } from 'react'

const filteredData = useMemo(() => filterData(data), [data])
const handleClick = useCallback(() => doSomething(), [])
```

3. **React.memo pour composants purs** :

```jsx
export default React.memo(MonthView)
```

## Bonnes pratiques

1. **Composants fonctionnels** avec hooks uniquement
2. **Structure** : Un composant par fichier
3. **Nomenclature** : PascalCase pour composants, camelCase pour fonctions
4. **Imports** : Imports absolus depuis `/src`
5. **Tailwind** : Classes utilitaires plutôt que CSS custom
6. **Accessibilité** : Utilisation de balises sémantiques HTML
7. **Responsive** : Mobile-first avec breakpoints Tailwind

## Points d'amélioration

### Sécurité et bonnes pratiques

1. **Validation des formulaires** : Ajouter une validation côté client robuste
2. **Gestion d'erreurs** : Implémenter un système de gestion d'erreurs global
3. **Loading states** : Ajouter des indicateurs de chargement
4. **Error boundaries** : Implémenter des error boundaries React
5. **TypeScript** : Migrer vers TypeScript pour la type safety

### Architecture

1. **State management** : Considérer Context API ou Zustand pour l'état global
2. **API client** : Créer un client API centralisé avec `fetch` ou `axios`
3. **Authentification** : Implémenter un système d'auth avec tokens JWT
4. **Tests** : Ajouter des tests unitaires et d'intégration (Vitest + Testing Library)
5. **Routes protégées** : Implémenter la protection des routes authentifiées

### Performance

1. **Lazy loading** : Charger les routes à la demande
2. **Image optimization** : Optimiser les images
3. **Bundle analysis** : Analyser et réduire la taille du bundle

## Prochaines étapes

- [Backend Flask](backend.md)
- [Base de données](database.md)
- [Guide de développement](../development/getting-started.md)
