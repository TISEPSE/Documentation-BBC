# Guide d'installation

Guide d'installation complet de Book By Click.

**Projet GitHub :** [https://github.com/TISEPSE/Book-By-Click.git](https://github.com/TISEPSE/Book-By-Click.git)

## Prérequis

- **Node.js** 18+
- **Python** 3.8+
- **Git**
- **Docker** (recommandé pour PostgreSQL)

```bash
# Vérifier les versions
node --version
python --version
git --version
docker --version
```

## Installation

### 1. Cloner le projet

```bash
git clone https://github.com/TISEPSE/Book-By-Click.git
cd Book-By-Click
```

### 2. Frontend

```bash
npm install
```

**Dépendances principales :**
- React 19.2.0
- Vite (rolldown-vite 7.2.5)
- Tailwind CSS 4.1.17
- React Router DOM 7.10.1

### 3. Backend

```bash
cd Backend
python -m venv ../venv
source ../venv/bin/activate  # Linux/Mac
# ou ..\venv\Scripts\activate  # Windows

pip install flask flask-sqlalchemy flask-cors psycopg2-binary python-dotenv
```

### 4. Base de données PostgreSQL

#### Avec Docker (recommandé)

```bash
cd Backend/src
docker compose up -d
```

Configuration automatique :
- PostgreSQL 15
- User: `appuser`
- Password: `apppassword`
- Database: `appdb`
- Port: 5432

#### Sans Docker

```bash
# Ubuntu/Debian
sudo apt install postgresql-15

# macOS
brew install postgresql@15

# Créer la base
sudo -u postgres psql
CREATE DATABASE appdb;
CREATE USER appuser WITH PASSWORD 'apppassword';
GRANT ALL PRIVILEGES ON DATABASE appdb TO appuser;
\q
```

### 5. Configuration

La configuration par défaut fonctionne avec Docker. Pour personnaliser, créez un fichier `.env` :

```bash
# Backend/src/.env
SQLALCHEMY_DATABASE_URI=postgresql://appuser:apppassword@localhost:5432/appdb
```

## Lancement

```bash
# Linux/Mac
npm run dev:lin

# Windows
npm run dev:win

# Avec Docker (lance automatiquement la base de données)
npm run dev:all
```

**Services lancés :**
- Frontend : http://localhost:5173
- Backend : http://localhost:5000

### Commandes séparées

```bash
# Base de données seulement
npm run db

# Backend seulement
npm run api          # Linux/Mac
npm run api:win      # Windows

# Frontend seulement
npm run web
```

## Vérification

### Test API

```bash
curl http://localhost:5000/api/services
```

## Troubleshooting

### Port déjà utilisé

```bash
# Trouver le processus
lsof -i :5173  # Linux/Mac
netstat -ano | findstr :5173  # Windows

# Tuer le processus
kill -9 <PID>
```

### Erreur de connexion PostgreSQL

```bash
# Vérifier que Docker est lancé
docker ps

# Vérifier les logs
docker logs db

# Redémarrer PostgreSQL
cd Backend/src
docker compose restart db
```

### Erreur de dépendances

```bash
# Frontend
npm install

# Backend
source venv/bin/activate
pip install flask flask-sqlalchemy flask-cors psycopg2-binary python-dotenv
```

## Production

```bash
# Build frontend
npm run build

# Backend avec Gunicorn
pip install gunicorn
gunicorn -w 4 -b 0.0.0.0:5000 app:app
```
