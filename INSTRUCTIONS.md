# Mise en place - Circuit CTS

## Architecture

```
GitHub Pages (public)
├── index.html   → étudiants  (tu distribues ce lien)
└── admin.html   → professeur (tu gardes ce lien pour toi)

Firebase
├── Realtime Database  → données des réservations
├── Authentication     → login sécurisé pour le prof
└── Security Rules     → étudiants peuvent lire + écrire leurs reservations
                         admin (toi) peut tout faire
```

---

## Étape 1 - Créer le projet Firebase

1. Va sur https://console.firebase.google.com
2. "Ajouter un projet" → donne un nom (ex. `circuit-cts`)
3. Désactive Google Analytics (inutile ici) → Créer

---

## Étape 2 - Activer Realtime Database

1. Menu gauche → "Realtime Database" → "Créer une base de données"
2. Choisir la région **europe-west1 (Belgique)**
3. Démarrer en **mode verrouillé** (on va mettre les vraies règles juste après)

### Règles de sécurité

Dans l'onglet "Règles" de la Realtime Database, colle ceci :

```json
{
  "rules": {
    "reservations": {
      ".read": true,
      "$slotId": {
        ".write": "auth != null || newData.exists()",
        "$entryId": {
          ".write": true,
          ".validate": "newData.hasChildren(['type', 'ts'])",
          "status": {
            ".write": "auth != null"
          }
        }
      }
    }
  }
}
```

**Ce que ça fait :**
- Tout le monde peut **lire** (voir le planning)
- Tout le monde peut **ajouter** une réservation
- Seul un prof **connecté** (Firebase Auth) peut changer le `status` (valider/refuser)
- Seul un prof **connecté** peut bloquer un créneau (type=blocked)

---

## Étape 3 - Activer Firebase Authentication

1. Menu gauche → "Authentication" → "Commencer"
2. Onglet "Sign-in method" → activer **Email/Mot de passe**
3. Onglet "Users" → "Ajouter un utilisateur"
   - Email : ton adresse (ex. `erwin@heh.be`)
   - Mot de passe : ce que tu veux (min 6 caractères)

---

## Étape 4 - Récupérer la config Firebase

1. Roue dentée → "Paramètres du projet"
2. Section "Vos applications" → "Ajouter une application" → icône Web `</>`
3. Donne un nom (ex. `cts-web`) → "Enregistrer"
4. Copie le bloc `firebaseConfig` qui apparaît

Il ressemble à ça :
```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "circuit-cts.firebaseapp.com",
  databaseURL: "https://circuit-cts-default-rtdb.europe-west1.firebasedatabase.io",
  projectId: "circuit-cts",
  storageBucket: "circuit-cts.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abcdef"
};
```

5. **Colle cette config** dans les deux fichiers `index.html` et `admin.html`
   (cherche `REMPLACE_API_KEY` dans les deux fichiers)

---

## Étape 5 - Mettre en ligne sur GitHub Pages

1. Crée un compte sur https://github.com si tu n'en as pas
2. "New repository" → nom : `circuit-cts` → **Public** → Créer
3. Upload les deux fichiers : `index.html` et `admin.html`
4. Dans le repo → "Settings" → "Pages"
5. Source : **Deploy from a branch** → branche `main` → dossier `/ (root)`
6. Après 1-2 min, ton site est disponible à :
   ```
   https://TON-USERNAME.github.io/circuit-cts/
   ```

### URLs finales
- **Étudiants** : `https://TON-USERNAME.github.io/circuit-cts/`
  (ou `/index.html`, même chose)
- **Toi** : `https://TON-USERNAME.github.io/circuit-cts/admin.html`

> L'URL admin n'est pas secrète par nature (GitHub repo public),
> mais elle est protégée par Firebase Authentication :
> sans ton email + mot de passe Firebase, la page ne fait rien.
> Si tu veux que l'URL soit aussi privée, passe le repo en **Private**
> (GitHub Pages reste dispo sur les repos privés avec GitHub Free).

---

## Résumé sécurité

| Qui | Peut faire |
|-----|-----------|
| Étudiant (index.html) | Voir le planning, ajouter/annuler SA réservation |
| Étudiant | Changer le statut d'une réservation | NON (bloqué par les règles Firebase) |
| Prof non connecté (admin.html) | Juste voir la page de login |
| Prof connecté (admin.html) | Tout : valider, refuser, bloquer, supprimer |

**Le mot de passe admin n'existe plus dans le code.**
La sécurité est garantie par Firebase Authentication côté serveur.
