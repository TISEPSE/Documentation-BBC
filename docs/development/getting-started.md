# Guide de développement

**Projet GitHub :** [https://github.com/TISEPSE/Book-By-Click.git](https://github.com/TISEPSE/Book-By-Click.git)

## Installation rapide

Voir le [Guide d'installation](../installation.md) complet.

```bash
git clone https://github.com/TISEPSE/Book-By-Click.git
cd Book-By-Click
npm install
cd Backend && python -m venv ../venv && source ../venv/bin/activate
pip install flask flask-sqlalchemy flask-cors psycopg2-binary python-dotenv
cd .. && npm run dev:lin  # ou dev:win pour Windows
```

## Développer une nouvelle fonctionnalité

### 1. Structure du projet

**Frontend (`src/`):**
```text
src/
├── pages/          # Pages de l'application (11 routes)
├── components/     # Composants réutilisables
├── Hook/           # Custom hooks (useCalendar, etc.)
└── App.jsx         # Routes principales
```

**Backend (`Backend/src/`):**
```text
Backend/src/
├── app.py          # Application Flask (factory pattern)
├── pages.py        # Blueprint avec tous les endpoints
├── models.py       # Modèles SQLAlchemy (9 tables)
├── extension.py    # Extensions (db, cors)
└── data/           # Fichiers JSON (services, villes)
```

### 2. Ajouter une page frontend

**a) Créer le composant** dans `src/pages/` :

```jsx
// src/pages/MaNouvellePage.jsx
import React from 'react';

export default function MaNouvellePage() {
  return (
    <div className="p-6">
      <h1 className="text-2xl font-bold">Ma nouvelle page</h1>
    </div>
  );
}
```

**b) Ajouter la route** dans `src/App.jsx` :

```jsx
import MaNouvellePage from './pages/MaNouvellePage';

// Dans les routes :
<Route path="/ma-page" element={<MaNouvellePage />} />
```

### 3. Ajouter un endpoint API

**Éditer `Backend/src/pages.py`** :

```python
@pages_blueprint.route('/api/mon-endpoint', methods=['POST'])
def mon_endpoint():
    data = request.get_json()
    # Traitement...
    return jsonify({"success": True, "data": result})
```

**Appeler depuis le frontend** :

```javascript
const response = await fetch('http://localhost:5000/api/mon-endpoint', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ param: 'valeur' })
});
const data = await response.json();
```

### 4. Ajouter un modèle de base de données

**Éditer `Backend/src/models.py`** :

```python
class MonModele(db.Model):
    __tablename__ = 'mon_modele'

    id = db.Column(db.Integer, primary_key=True)
    nom = db.Column(db.String(100), nullable=False)

    def to_dict(self):
        return {
            'id': self.id,
            'nom': self.nom
        }
```

**Les tables sont créées automatiquement** au démarrage de Flask (`db.create_all()`).

### 5. Tester vos modifications

```bash
# Tester un endpoint
curl http://localhost:5000/api/mon-endpoint

# Vérifier la base de données
docker exec -it db psql -U appuser -d appdb
\dt  # Lister les tables
SELECT * FROM mon_modele;
```

## Services disponibles

### PostgreSQL (port 5432)

```bash
# Connexion
docker exec -it db psql -U appuser -d appdb

# Voir les logs
docker logs db
```

### SMTP (port 25)

Serveur SMTP configuré dans `Backend/src/docker-compose.yml` pour l'envoi d'emails.

Configuration :
- Relay: smtp.gmail.com:587
- Voir `Backend/src/mailer.py` pour l'utilisation

### Hot reload

- **Frontend** : Vite recharge automatiquement
- **Backend** : Flask recharge automatiquement en mode développement

## Conventions de code

**Frontend :**
- Composants : PascalCase (`MaPage.jsx`)
- Hooks : camelCase avec "use" (`useCalendar.jsx`)
- Tailwind CSS pour le style

**Backend :**
- Fichiers : snake_case (`my_file.py`)
- Classes : PascalCase (`MyModel`)
- Variables/fonctions : snake_case (`my_function`)

**Git :**
```bash
git checkout -b feature/ma-fonctionnalite
git commit -m "feat: ajouter ma fonctionnalité"
git commit -m "fix: corriger le bug"
```
