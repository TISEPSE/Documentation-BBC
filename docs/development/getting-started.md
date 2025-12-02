# Guide de démarrage - Développement

## Configuration de l'environnement

### 1. Cloner le projet

```bash
git clone <repository-url>
cd Projet-BTS-Alan-Maxime-Tisba
```

### 2. Frontend

```bash
# Installer les dépendances
npm install

# Variables d'environnement (optionnel)
cp .env.example .env
```

### 3. Backend

```bash
cd Backend

# Créer environnement virtuel
python -m venv ../venv

# Activer
source ../venv/bin/activate  # Linux/Mac
../venv/Scripts/activate  # Windows

# Installer dépendances
pip install flask flask-cors mysql-connector-python python-dotenv pyjwt werkzeug

# Configuration
cp .env.example .env
# Éditez .env avec vos paramètres
```

### 4. Base de données

```bash
cd Backend/db
docker-compose up -d
```

## Lancement

### Option 1 : Commande combinée

```bash
# Linux/Mac
npm run dev:lin

# Windows
npm run dev:win

# Avec Docker
npm run dev:all
```

### Option 2 : Séparé

```bash
# Terminal 1 - Backend
cd Backend
source ../venv/bin/activate
flask --app app run --no-debugger

# Terminal 2 - Frontend
npm run dev
```

## Structure du code

### Frontend

```text
src/
├── pages/          # Routes principales
├── components/     # Composants réutilisables
├── Hook/          # Custom hooks
└── assets/        # Static files
```

### Backend

```text
Backend/
├── app.py         # Point d'entrée
├── routes/        # API routes
├── models/        # Data models
├── services/      # Business logic
└── utils/         # Utilities
```

## Conventions

### Nommage

- **Composants** : PascalCase (`MyComponent.jsx`)
- **Fichiers** : snake_case (`my_file.py`)
- **Variables** : camelCase (JS) / snake_case (Python)

### Git

```bash
# Branches
git checkout -b feature/ma-fonctionnalite
git checkout -b fix/mon-bug

# Commits
git commit -m "feat: ajouter fonctionnalité X"
git commit -m "fix: corriger bug Y"
```

## Tests

### Frontend

```bash
npm test
```

### Backend

```bash
pytest
```

## Debugging

### Frontend

Utilisez React DevTools dans le navigateur.

### Backend

```python
import pdb; pdb.set_trace()  # Breakpoint
```

Ou avec VS Code, configurez `.vscode/launch.json` :

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: Flask",
      "type": "python",
      "request": "launch",
      "module": "flask",
      "env": {
        "FLASK_APP": "app.py",
        "FLASK_ENV": "development"
      },
      "args": ["run", "--no-debugger"],
      "jinja": true
    }
  ]
}
```

## Prochaines étapes

- [Structure du code](structure.md)
- [Contribution](contributing.md)
- [Architecture](../architecture/overview.md)
