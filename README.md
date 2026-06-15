# 🚀 Maîtriser le CI/CD avec GitHub Actions et GitHub Pages

Ce document est un guide complet pas-à-pas pour comprendre, recréer et maîtriser un pipeline CI/CD (Continuous Integration / Continuous Deployment) de zéro en utilisant **GitHub Actions**.

---

## 🤔 Qu'est-ce que le CI/CD ?

*   **CI (Intégration Continue)** : À chaque fois que vous modifiez du code, un robot exécute automatiquement vos tests pour s'assurer que vous n'avez rien cassé.
*   **CD (Déploiement Continu)** : Si les tests passent (et uniquement s'ils passent !), le robot prend votre code et le met en ligne automatiquement.

L'objectif : **Automatiser et sécuriser la mise en production**.

---

## 🛠️ Les étapes pour le refaire soi-même

### 1. Initialiser le projet localement
Ouvrez votre terminal dans un dossier vide et lancez ces commandes :
```bash
# 1. Initialiser git
git init

# 2. Créer un fichier package.json par défaut
npm init -y

# 3. Installer Jest, notre outil pour créer et lancer des tests
npm install --save-dev jest
```

### 2. Écrire le code de l'application (`app.js`)
Créez un fichier `app.js` avec une simple fonction mathématique. C'est le code que l'on veut tester.
```javascript
// app.js
function addition(a, b) {
  return a + b;
}
// On exporte la fonction pour pouvoir l'utiliser dans nos tests
module.exports = addition;
```

### 3. Écrire le test (`app.test.js`)
Créez un fichier `app.test.js`. Le CI exécutera ce fichier pour vérifier que notre fonction fait bien son travail.
```javascript
// app.test.js
const addition = require('./app');

// On décrit ce qu'on attend de la fonction
test('1 + 2 doit donner 3', () => {
  expect(addition(1, 2)).toBe(3);
});
```

*N'oubliez pas d'indiquer à `npm` comment lancer le test. Dans le fichier `package.json`, modifiez la section scripts :*
```json
  "scripts": {
    "test": "jest"
  }
```

### 4. Créer la page web à déployer (`public/index.html`)
Pour l'aspect **CD**, nous allons mettre en ligne une page web. Créez un dossier `public` et un fichier `index.html` à l'intérieur.
```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>Démo CI/CD</title>
</head>
<body>
  <h1>🚀 Déploiement Automatique Réussi !</h1>
</body>
</html>
```

### 5. Le cœur de la magie : le fichier Workflow 🧙‍♂️
Pour que GitHub comprenne qu'il doit exécuter des actions, il faut créer un fichier spécifique : `.github/workflows/ci.yml`.

**Explication détaillée du code :**
```yaml
name: CI/CD Pipeline

# QUAND déclencher le pipeline ?
on:
  push:
    branches: [ main ] # À chaque push sur la branche 'main'

# DROITS nécessaires pour que le robot puisse publier la page web
permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  # ---------------------------------------------
  # JOB 1 : LE CI (Test)
  # ---------------------------------------------
  test:
    runs-on: ubuntu-latest # Le serveur virtuel utilisé
    steps:
      - uses: actions/checkout@v4 # 1. Le robot récupère ton code
      
      - uses: actions/setup-node@v4 # 2. Le robot installe Node.js
        with:
          node-version: '24'
          
      - run: npm ci # 3. Le robot installe les dépendances (Jest)
      - run: npm test # 4. Le robot LANCE LES TESTS. Si ça échoue, le pipeline s'arrête ici !

  # ---------------------------------------------
  # JOB 2 : LE CD (Déploiement)
  # ---------------------------------------------
  deploy:
    needs: test # CRUCIAL : Le déploiement NE SE FAIT QUE SI le job 'test' a réussi !
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/checkout@v4 # Récupère le code
      - uses: actions/configure-pages@v5 # Configure GitHub pages
      
      - uses: actions/upload-pages-artifact@v3 # Prépare le dossier 'public' pour l'envoi
        with:
          path: ./public
          
      - id: deployment
        uses: actions/deploy-pages@v4 # Met en ligne sur internet !
```

### 6. Tout envoyer sur GitHub
Il ne reste plus qu'à créer un dépôt public sur GitHub et à envoyer votre code :
```bash
git add .
git commit -m "Mon premier CI/CD"
git branch -M main
git remote add origin https://github.com/VOTRE_PSEUDO/VOTRE_DEPOT.git
git push -u origin main
```

### 7. Activer GitHub Pages
Pour que le site soit visible :
1. Allez sur la page de votre dépôt sur GitHub.
2. Allez dans **Settings** > **Pages**.
3. Dans la section **Build and deployment**, sous **Source**, choisissez **GitHub Actions**.

### 🎉 Conclusion
À partir de maintenant, dès que vous faites une modification et que vous lancez un `git push`, GitHub va automatiquement lancer les tests. Si les tests sont bons, votre site internet sera mis à jour quelques secondes plus tard sans aucune intervention de votre part. C'est la puissance du CI/CD !
