# FAQ & Dépannage

Questions fréquemment posées et solutions aux problèmes courants.

## Questions générales

### Qu'est-ce que Book By Click ?

Book By Click est une plateforme de réservation qui met en relation des étudiants avec des entreprises partenaires pour des entretiens, stages ou opportunités professionnelles.

### Comment fonctionne le système ?

1. Les **étudiants** créent un compte et recherchent des entreprises
2. Ils réservent des créneaux horaires disponibles
3. Les **entreprises** reçoivent les demandes et peuvent accepter/refuser
4. Les deux parties reçoivent des confirmations par email

### Est-ce gratuit ?

Oui, la plateforme est entièrement gratuite pour les étudiants et les établissements scolaires partenaires.

### Puis-je annuler une réservation ?

Oui, vous pouvez annuler une réservation en attente ou confirmée. Il est recommandé de le faire au moins 24h à l'avance.

---

## Inscription et connexion

### Je ne reçois pas l'email de confirmation

**Solutions:**

1. Vérifiez votre dossier **spam/courrier indésirable**
2. Vérifiez que l'adresse email est correcte dans votre profil
3. Attendez 5-10 minutes (délai de livraison)
4. Contactez le support si le problème persiste

### J'ai oublié mon mot de passe

1. Cliquez sur "Mot de passe oublié" sur la page de connexion
2. Entrez votre adresse email
3. Vous recevrez un lien de réinitialisation par email
4. Cliquez sur le lien et définissez un nouveau mot de passe

!!! tip "Conseil"
    Le lien de réinitialisation expire après **1 heure**.

### Mon mot de passe est refusé lors de l'inscription

Le mot de passe doit respecter ces critères:

- Minimum **8 caractères**
- Au moins une lettre majuscule
- Au moins une lettre minuscule
- Au moins un chiffre
- Au moins un caractère spécial recommandé

**Exemples valides:**
- `MonMotDePasse123!`
- `Secure@Pass2024`

### Je ne peux pas me connecter

**Vérifications:**

1. ✓ Email correct (pas d'espace avant/après)
2. ✓ Mot de passe correct (attention à la casse)
3. ✓ Compte activé (vérifiez votre email)
4. ✓ Compte non suspendu

Si tout est correct, réinitialisez votre mot de passe.

---

## Réservations (Étudiants)

### Aucune entreprise n'apparaît dans ma recherche

**Causes possibles:**

1. **Filtres trop restrictifs:** Essayez sans filtres
2. **Aucune entreprise dans votre région:** Élargissez la zone
3. **Problème de connexion:** Rafraîchissez la page

### Je ne vois pas de créneaux disponibles

**Raisons:**

1. L'entreprise n'a pas défini de disponibilités
2. Tous les créneaux sont déjà réservés
3. L'entreprise a bloqué certaines dates
4. Vous consultez une période trop éloignée

**Solution:** Contactez l'entreprise directement ou choisissez une autre période.

### Ma réservation est en attente depuis longtemps

Les entreprises ont **48-72 heures** pour répondre. Si le délai est dépassé:

1. Envoyez un message via la plateforme
2. Contactez l'entreprise directement
3. Annulez et réservez ailleurs si urgent

### Puis-je réserver plusieurs créneaux ?

Oui, mais:

- Maximum **3 réservations en attente** simultanément
- Pas de double réservation au même moment
- Annulez les réservations inutiles pour libérer des places

### L'entreprise a refusé ma demande

C'est normal, plusieurs raisons possibles:

- Créneau déjà pris entretemps
- Profil ne correspond pas au poste
- Entreprise débordée

**Action:** Consultez la raison donnée et postulez ailleurs.

---

## Gestion (Entreprises)

### Je ne reçois pas les notifications de nouvelles réservations

**Vérifications:**

1. Paramètres > Notifications > Activées ✓
2. Vérifiez les spams
3. Email correct dans le profil
4. Quota email non atteint

### Comment modifier mes disponibilités ?

1. Dashboard > **Paramètres**
2. Section **Créneaux horaires**
3. Modifiez les jours/heures
4. **Enregistrez**

!!! warning "Important"
    Les modifications n'affectent pas les réservations déjà confirmées.

### Je ne peux pas bloquer une date

Assurez-vous:

- Aucune réservation confirmée sur cette date
- Date dans le futur
- Connecté avec un compte entreprise

Pour annuler des réservations existantes, contactez les étudiants.

### Comment voir l'historique complet ?

Dashboard > **Statistiques** > **Exporter les données**

Vous pouvez exporter en CSV ou Excel toutes vos réservations avec filtres par période.

---

## Problèmes techniques

### La page ne charge pas / Erreur 404

**Solutions:**

```bash
# Vider le cache du navigateur
Ctrl + Shift + Del (Windows/Linux)
Cmd + Shift + Del (Mac)

# Ou mode navigation privée
Ctrl + Shift + N (Chrome)
Ctrl + Shift + P (Firefox)
```

Si le problème persiste:

1. Vérifiez votre connexion internet
2. Essayez un autre navigateur
3. Contactez le support

### Erreur "Impossible de se connecter au serveur"

**Causes:**

1. **Serveur en maintenance:** Attendez 5-10 minutes
2. **Problème réseau:** Vérifiez votre connexion
3. **Firewall/Antivirus:** Autorisez l'application

**Diagnostic:**

```bash
# Vérifier la connexion au serveur
ping api.book-by-click.com

# Vérifier l'API
curl https://api.book-by-click.com/api/health
```

### Les images ne s'affichent pas

1. Videz le cache du navigateur
2. Vérifiez que JavaScript est activé
3. Désactivez temporairement les extensions (AdBlock)

### Erreur "Token expired"

Votre session a expiré après 24h d'inactivité.

**Solution:** Reconnectez-vous.

Pour éviter cela: Cochez "Se souvenir de moi" à la connexion.

### L'application est lente

**Optimisations:**

1. Fermez les onglets inutiles
2. Videz le cache
3. Vérifiez votre connexion internet (minimum 1 Mbps)
4. Essayez en navigation privée

**Si problème serveur:** Contactez le support.

---

## Développement

### npm install échoue

**Erreur:** `ENOENT` ou `Permission denied`

**Solutions:**

```bash
# Nettoyer le cache npm
npm cache clean --force

# Supprimer node_modules et package-lock
rm -rf node_modules package-lock.json
npm install

# Permissions (Linux/Mac)
sudo chown -R $USER ~/.npm
```

### Python module not found

```bash
# Activer l'environnement virtuel
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows

# Réinstaller les dépendances
pip install -r requirements.txt

# Ou manuellement
pip install flask flask-cors mysql-connector-python
```

### Port déjà utilisé

**Erreur:** `EADDRINUSE` ou `Port 5000 already in use`

**Trouver le processus:**

```bash
# Linux/Mac
lsof -i :5000
kill -9 <PID>

# Windows
netstat -ano | findstr :5000
taskkill /PID <PID> /F
```

**Ou changez le port:**

```javascript
// vite.config.js
export default {
  server: { port: 3000 }
}
```

```python
# Backend Flask
app.run(port=5001)
```

### Erreur de connexion à MySQL

**Vérifier que MySQL est démarré:**

```bash
# Linux
sudo systemctl status mysql
sudo systemctl start mysql

# Mac
brew services list
brew services start mysql

# Windows
net start MySQL80
```

**Tester la connexion:**

```bash
mysql -u reserv_user -p reservations_db
```

Si échec:

1. Vérifiez le mot de passe dans `.env`
2. Recréez l'utilisateur:
   ```sql
   DROP USER 'reserv_user'@'localhost';
   CREATE USER 'reserv_user'@'localhost' IDENTIFIED BY 'password';
   GRANT ALL ON reservations_db.* TO 'reserv_user'@'localhost';
   ```

### CORS errors dans la console

**Erreur:** `Access to fetch blocked by CORS policy`

**Backend Flask - Vérifiez la configuration:**

```python
from flask_cors import CORS

CORS(app, resources={
    r"/api/*": {
        "origins": ["http://localhost:5173"],
        "methods": ["GET", "POST", "PUT", "DELETE"],
        "allow_headers": ["Content-Type", "Authorization"]
    }
})
```

### Hot reload ne fonctionne pas

**Vite (Frontend):**

```bash
# Relancez avec flag de debug
npm run dev -- --force

# Ou nettoyez le cache
rm -rf node_modules/.vite
```

**Flask (Backend):**

```bash
# Utilisez flask run au lieu de python app.py
flask --app app run --debug
```

---

## Base de données

### Réinitialiser la base de données

!!! danger "Attention"
    Cette opération supprime **toutes les données** !

```bash
mysql -u root -p
```

```sql
DROP DATABASE IF EXISTS reservations_db;
CREATE DATABASE reservations_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE reservations_db;
SOURCE /chemin/vers/schema.sql;
```

### Sauvegarder la base de données

```bash
# Export complet
mysqldump -u reserv_user -p reservations_db > backup.sql

# Compressé
mysqldump -u reserv_user -p reservations_db | gzip > backup.sql.gz

# Structure uniquement (sans données)
mysqldump -u reserv_user -p --no-data reservations_db > structure.sql
```

### Restaurer une sauvegarde

```bash
# Décompresser si nécessaire
gunzip backup.sql.gz

# Importer
mysql -u reserv_user -p reservations_db < backup.sql
```

---

## Performance

### L'API est lente

**Diagnostic:**

1. Vérifiez les logs: `/var/log/bookbyclick/`
2. Temps de réponse: `curl -w "@curl-format.txt" -o /dev/null -s URL`
3. Requêtes lentes MySQL: `SHOW PROCESSLIST;`

**Optimisations:**

- Ajouter des index sur les colonnes fréquemment recherchées
- Augmenter les workers Gunicorn: `-w 8`
- Activer le cache Redis

### Le frontend met du temps à charger

**Optimisations:**

1. **Build de production:**
   ```bash
   npm run build
   ```

2. **Lazy loading des images:**
   ```jsx
   <img loading="lazy" src="..." />
   ```

3. **Code splitting:**
   ```jsx
   const Dashboard = lazy(() => import('./Dashboard'));
   ```

4. **CDN pour les assets** en production

---

## Sécurité

### Mon compte a été piraté

**Actions immédiates:**

1. **Changez votre mot de passe** immédiatement
2. Contactez le support: support@book-by-click.com
3. Vérifiez les activités suspectes dans l'historique
4. Activez l'authentification à deux facteurs si disponible

### Je pense avoir trouvé une faille de sécurité

**Divulgation responsable:**

1. **NE PAS** exploiter la faille
2. **NE PAS** la rendre publique
3. Contactez: security@book-by-click.com
4. Fournissez les détails:
   - Description
   - Steps to reproduce
   - Impact potentiel
   - Votre nom (optionnel)

---

## Contact et support

### Comment contacter le support ?

**Email:** support@book-by-click.com
**Temps de réponse:** 24-48h ouvrées

**GitHub Issues:** Pour les bugs techniques
https://github.com/TISEPSE/Book-By-Click/issues

### Signaler un bug

1. Vérifiez qu'il n'est pas déjà rapporté
2. Créez une issue sur GitHub avec:
   - Titre descriptif
   - Steps to reproduce
   - Comportement attendu vs actuel
   - Screenshots si applicable
   - Navigateur/OS/Version

### Proposer une amélioration

Créez une "Feature Request" sur GitHub avec:

- Description détaillée
- Cas d'usage
- Mockups si possible
- Impact estimé

---

## Ressources additionnelles

- [Guide d'installation](installation.md)
- [Documentation API](api/endpoints.md)
- [Guide de déploiement](deployment.md)
- [Architecture](architecture/overview.md)
- [Contribuer au projet](development/contributing.md)
