# Guide d'installation

Ce guide vous accompagne dans l'installation du système de réservation sur votre machine.

## Prérequis

Avant de commencer, assurez-vous d'avoir :

- [x] **Node.js** 18.x ou supérieur
- [x] **Python** 3.8 ou supérieur
- [x] **Git** pour cloner le repository
- [x] **Docker** (optionnel, pour la base de données)

### Vérifier les installations

```bash
node --version  # v18.0.0 ou supérieur
python --version  # Python 3.8 ou supérieur
git --version
docker --version  # (optionnel)
```

## Installation

### 1. Cloner le repository

```bash
git clone <url-du-repository>
cd Projet-BTS-Alan-Maxime-Tisba
```

### 2. Installation du Frontend

```bash
# Installer les dépendances
npm install
```

#### Dépendances installées

- `react` & `react-dom` (v19.2.0)
- `react-router-dom` (v7.9.6)
- `tailwindcss` (v4.1.17)
- `vite` (rolldown-vite v7.2.5)
- `@heroicons/react` (v2.2.0)

### 3. Installation du Backend

```bash
cd Backend

# Créer un environnement virtuel Python
python -m venv ../venv

# Activer l'environnement virtuel
source ../venv/bin/activate  # Linux/Mac
# ou
..\venv\Scripts\activate  # Windows

# Installer les dépendances
pip install flask flask-cors mysql-connector-python python-dotenv
```

### 4. Configuration de la base de données

#### Option A : Avec Docker (Recommandé)

```bash
cd Backend/db
docker-compose up -d
```

Le fichier `docker-compose.yml` configure automatiquement MySQL.

#### Option B : Installation manuelle MySQL

```bash
# Ubuntu/Debian
sudo apt install mysql-server

# macOS avec Homebrew
brew install mysql

# Démarrer MySQL
sudo service mysql start  # Linux
brew services start mysql  # macOS
```

Créez la base de données :

```sql
CREATE DATABASE reservations_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'reserv_user'@'localhost' IDENTIFIED BY 'votre_mot_de_passe';
GRANT ALL PRIVILEGES ON reservations_db.* TO 'reserv_user'@'localhost';
FLUSH PRIVILEGES;
```

### 5. Configuration de l'environnement

#### Backend (.env)

Créez un fichier `.env` dans le dossier `Backend/` :

```env
FLASK_APP=app.py
FLASK_ENV=development
DATABASE_URL=mysql://reserv_user:votre_mot_de_passe@localhost/reservations_db
SECRET_KEY=votre_cle_secrete_aleatoire
```

#### Frontend (optionnel)

Si nécessaire, créez un fichier `.env` à la racine :

```env
VITE_API_URL=http://localhost:5000
```

## Lancement de l'application

### Développement

Plusieurs options selon votre environnement :

=== "Linux/Mac"
    ```bash
    npm run dev:lin
    ```

    Cette commande lance :
    - Le backend Flask sur le port 5000
    - Le frontend Vite sur le port 5173

=== "Windows"
    ```bash
    npm run dev:win
    ```

    Version Windows de la commande ci-dessus

=== "Avec Docker complet"
    ```bash
    npm run dev:all
    ```

    Lance Docker + Backend + Frontend

### Lancement séparé

Si vous préférez lancer les services séparément :

```bash
# Terminal 1 - Backend
cd Backend
source ../venv/bin/activate
python -m flask --app app run --no-debugger

# Terminal 2 - Frontend
npm run dev
```

## Vérification

### Frontend

Ouvrez http://localhost:5173 dans votre navigateur.

Vous devriez voir la page d'accueil de l'application.

### Backend

Testez l'API :

```bash
curl http://localhost:5000/api/health
```

Réponse attendue :

```json
{
  "status": "ok",
  "message": "API is running"
}
```

## Troubleshooting

### Port déjà utilisé

Si le port 5173 ou 5000 est déjà utilisé :

```bash
# Trouver le processus
lsof -i :5173  # Linux/Mac
netstat -ano | findstr :5173  # Windows

# Tuer le processus
kill -9 <PID>
```

Ou changez le port dans `vite.config.js` :

```javascript
export default {
  server: {
    port: 3000  // Nouveau port
  }
}
```

### Erreur de connexion à la base de données

1. Vérifiez que MySQL est démarré :
   ```bash
   sudo service mysql status  # Linux
   brew services list  # macOS
   ```

2. Vérifiez les identifiants dans `.env`

3. Testez la connexion :
   ```bash
   mysql -u reserv_user -p reservations_db
   ```

### Erreur "Module not found"

```bash
# Frontend
npm install

# Backend
pip install -r requirements.txt
```

### Docker ne démarre pas

```bash
# Vérifier que Docker est démarré
docker ps

# Relancer Docker Desktop (macOS/Windows)
# ou
sudo service docker start  # Linux

# Reconstruire les conteneurs
cd Backend/db
docker-compose down
docker-compose up -d --build
```

## Build de production

### Frontend

```bash
npm run build
```

Les fichiers optimisés seront dans le dossier `dist/`.

### Backend

Pour la production, utilisez un serveur WSGI comme Gunicorn :

```bash
pip install gunicorn
gunicorn -w 4 -b 0.0.0.0:5000 app:app
```

## Prochaines étapes

- Consultez l'[Architecture](architecture/overview.md) pour comprendre le système
- Lisez le [Guide de développement](development/getting-started.md)
- Explorez l'[API](api/endpoints.md)
