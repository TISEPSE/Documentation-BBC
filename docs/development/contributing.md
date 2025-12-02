# Guide de contribution

## Workflow

### 1. Fork et clone

```bash
git clone https://github.com/votre-username/Projet-BTS-Alan-Maxime-Tisba.git
cd Projet-BTS-Alan-Maxime-Tisba
```

### 2. Créer une branche

```bash
git checkout -b feature/ma-fonctionnalite
# ou
git checkout -b fix/mon-bug
```

### 3. Développer

Faites vos modifications en suivant les conventions.

### 4. Commit

```bash
git add .
git commit -m "feat: ajouter fonctionnalité X"
```

#### Convention de commit

```text
<type>(<scope>): <subject>

<body>
```

**Types:**
- `feat`: Nouvelle fonctionnalité
- `fix`: Correction de bug
- `docs`: Documentation
- `style`: Formatage
- `refactor`: Refactorisation
- `test`: Tests
- `chore`: Maintenance

**Exemples:**
```text
feat(auth): ajouter connexion Google
fix(reservation): corriger validation date
docs(api): documenter endpoint /users
```

### 5. Push

```bash
git push origin feature/ma-fonctionnalite
```

### 6. Pull Request

1. Créez une PR sur GitHub
2. Décrivez vos changements
3. Attendez la review

## Standards de code

### Frontend (React)

```jsx
// ✅ Bon
export default function MyComponent({ title, onClick }) {
  const [isOpen, setIsOpen] = useState(false)

  return (
    <div className="p-4">
      <h1>{title}</h1>
      <button onClick={onClick}>Click</button>
    </div>
  )
}

// ❌ Mauvais
function myComponent(props) {
  var open = false
  return <div><h1>{props.title}</h1></div>
}
```

### Backend (Python)

```python
# ✅ Bon
def create_user(email: str, password: str) -> dict:
    """Créer un nouvel utilisateur.

    Args:
        email: Email de l'utilisateur
        password: Mot de passe (sera hashé)

    Returns:
        dict: Utilisateur créé

    Raises:
        ValueError: Si l'email existe déjà
    """
    if User.find_by_email(email):
        raise ValueError("Email already exists")

    hashed_password = generate_password_hash(password)
    user = User.create(email, hashed_password)
    return user

# ❌ Mauvais
def CreateUser(e,p):
    u=User.create(e,p)
    return u
```

## Tests

### Frontend

```jsx
import { render, screen } from '@testing-library/react'
import MyComponent from './MyComponent'

describe('MyComponent', () => {
  it('renders title', () => {
    render(<MyComponent title="Test" />)
    expect(screen.getByText('Test')).toBeInTheDocument()
  })
})
```

### Backend

```python
def test_create_user():
    user = UserService.create_user({
        'email': 'test@example.com',
        'password': 'password123'
    })
    assert user['email'] == 'test@example.com'
```

## Checklist PR

Avant de soumettre :

- [ ] Code testé localement
- [ ] Tests ajoutés/mis à jour
- [ ] Pas d'erreurs de linting
- [ ] Documentation mise à jour
- [ ] Commits clairs et atomiques
- [ ] Branch à jour avec main

## Style Guide

### JavaScript/React

- Utiliser `const` par défaut
- Hooks en haut du composant
- PropTypes ou TypeScript
- Noms de composants en PascalCase
- Fonctions en camelCase

### Python

- PEP 8
- Type hints
- Docstrings
- snake_case pour fonctions/variables
- PascalCase pour classes

## Documentation

Mettez à jour la documentation si vous :

- Ajoutez une nouvelle page
- Modifiez une route API
- Changez la structure du projet
- Ajoutez une dépendance

## Questions

Si vous avez des questions :

- Ouvrez une issue
- Demandez dans les discussions
- Contactez les mainteneurs
