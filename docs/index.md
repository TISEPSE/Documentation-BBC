# Book By Click

**Book By Click** est un système de réservation en ligne permettant aux clients de prendre rendez-vous avec des entreprises partenaires.

**Projet GitHub :** [https://github.com/TISEPSE/Book-By-Click.git](https://github.com/TISEPSE/Book-By-Click.git)

## Technologies

- **Frontend** : React 19, Vite, Tailwind CSS 4, React Router 7
- **Backend** : Flask, PostgreSQL 15, SQLAlchemy
- **Infrastructure** : Docker, Docker Compose

## Installation

```bash
# Cloner le projet
git clone https://github.com/TISEPSE/Book-By-Click.git
cd Book-By-Click

# Installer les dépendances frontend
npm install

# Installer les dépendances backend
cd Backend
python -m venv ../venv
source ../venv/bin/activate  # Linux/Mac
pip install -r requirements.txt

# Lancer l'application
cd ..
npm run dev:lin  # Linux/Mac
# ou npm run dev:win  # Windows
```

**URLs :**
- Frontend : http://localhost:5173
- Backend : http://localhost:5000

## Documentation

- **[Installation](installation.md)** - Guide d'installation complet
- **[API](api/endpoints.md)** - Endpoints et exemples
- **[Architecture](architecture/overview.md)** - Structure technique du projet
- **[Développement](development/getting-started.md)** - Guide pour développeurs
- **[Déploiement](deployment.md)** - Mise en production
