# Déploiement — CityLoop Quest Copenhague

Guide généré par **CLQ App Factory** pour **Copenhague** (`copenhague`).  
Dossier de sortie : `CLQ-COPENHAGUE/clq-copenhague` — URL Netlify cible : **https://clq-copenhague.netlify.app**

---

## Vue d’ensemble

| Élément | Valeur pour cette ville |
|--------|-------------------------|
| Slug backend / DB | `copenhague` |
| Package npm | `clq-copenhague` |
| Site Netlify | `clq-copenhague` |
| Site ID Netlify | `779a34a4-01bd-4ad6-ac4b-3820c7440639` |
| Dépôt CI (GitHub → Netlify) | `TogaThrust/cityloopquest-Copenhague` |
| URL publique | https://clq-copenhague.netlify.app |
| API (Render) | https://cityloopquest-api.onrender.com |
| Région | Région capitale, Danemark |

**Ordre recommandé :** Factory (contenu) → Supabase + Stripe + Render → Google Maps → Netlify → tests → `npm run deploy`.

---

## 1. Génération depuis la Factory

1. Ouvrir **CLQ App Factory** (`npm start` → http://localhost:3847).
2. Charger ou compléter le JSON ville (**Copenhague**).
3. Lancer **Générer l’app** → sortie dans `CLQ-COPENHAGUE/clq-copenhague`.
4. Vérifier dans la Factory : audios, images POI, quiz, traductions sans erreur bloquante.
5. Tester en local :
   ```bash
   cd "CLQ-COPENHAGUE/clq-copenhague"
   npm install
   npm run dev
   ```
   Ouvrir http://localhost:5173 — en dev, le Service Worker est désactivé automatiquement.

> **API en local :** par défaut l’app pointe vers `https://cityloopquest-api.onrender.com`. Pour forcer une API locale :  
> `?api_base=http://localhost:8081` ou `setApiBase('http://localhost:8081')` en console.

---

## 2. GitHub

Deux dépôts possibles (recommandé pour les villes déjà en prod) :

| Rôle | Exemple | Usage |
|------|---------|--------|
| **Dev / Factory** | `clq-copenhague` (privé) | Travail quotidien, génération Factory |
| **CI Netlify** | `TogaThrust/cityloopquest-Copenhague` (public) | Push déclenche le build Netlify |

1. Pousser le code vers le dépôt dev (`origin`).
2. Ajouter le remote CI si besoin :
   ```bash
   git remote add cityloopquest https://github.com/TogaThrust/cityloopquest-Copenhague.git
   ```
3. Après chaque release : `npm run deploy:ci` (push `main` vers `cityloopquest`).
4. Vérifier `.gitignore` : `.env`, `dist/`, `node_modules/`, secrets — **pas** `.netlify/state.json` ni `netlify.site.json`.
5. Secrets GitHub (dépôt CI) pour le workflow `.github/workflows/netlify-deploy.yml` :
   - `NETLIFY_AUTH_TOKEN`, `NETLIFY_SITE_ID` (= ``)
   - `GOOGLE_MAPS_API_KEY` (build)

> **Lille / villes déjà liées dans Netlify :** le push sur `TogaThrust/cityloopquest-Copenhague` suffit souvent ; le workflow GitHub Actions sert de secours si l’app GitHub Netlify n’a pas accès au dépôt.

---

## 3. Netlify

### 3.1 Création du site

- **Build command :** `npm run build`
- **Publish directory :** `dist`
- **Functions :** `netlify/functions` (voir `netlify.toml`)
- **Node :** 18 (déjà dans `netlify.toml`)

### 3.2 Variables d’environnement (build)

À définir dans **Site settings → Environment variables** (pas la liste complète d’une autre ville — uniquement ce dont **Copenhague** a besoin) :

| Variable | Usage |
|----------|--------|
| `GOOGLE_MAPS_API_KEY` | Injectée dans `api-key.js` au build (`build.mjs`) |
| `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS` | Formulaire contact / POI (`mail.html`, fonctions Netlify) |
| `POI_STAFF_EMAIL` | Destinataire des propositions POI |
| `POI_APPROVE_SECRET` | Secret d’approbation POI communautaires |
| `SITE_URL` | https://clq-copenhague.netlify.app |
| `NETLIFY_SITE_ID` + `NETLIFY_AUTH_TOKEN` | Blobs POI (ou `NETLIFY_BLOB_READ_WRITE_TOKEN`) |

Ne **pas** y mettre `STRIPE_SECRET_KEY` (reste sur Render).

### 3.3 Déploiement

**CI (recommandé après changements Git) :**
```bash
npm run deploy:ci
```

**Manuel (CLI Netlify, sans prompt) :**
```bash
cd "CLQ-COPENHAGUE/clq-copenhague"
npm install
npm run deploy
```

(`npm run deploy` = build + deploy prod ; le Site ID est lu depuis `netlify.site.json` / `.netlify/state.json`.)

**Site ID :** `779a34a4-01bd-4ad6-ac4b-3820c7440639` — admin : https://app.netlify.com/projects/clq-copenhague

---

## 4. Google Cloud — clé Maps

Dans [Google Cloud Console](https://console.cloud.google.com/) → APIs & Services → Credentials → votre clé **Maps JavaScript API** :

**Référents HTTP autorisés :**

- `https://clq-copenhague.netlify.app/*`
- `http://localhost:5173/*` (dev Vite)
- `https://localhost:5173/*` (dev Vite HTTPS)

Activer : **Maps JavaScript API**, **Places API**, **Directions API** (ou Routes API selon migration future).

---

## 5. Render — API `cityloopquest-api`

Dashboard Render → service API → **Environment** :

> **Noms de variables :** pour un slug multi-mots (`copenhague`), utiliser des **underscores** dans le préfixe env (`COPENHAGUE` = tirets du slug remplacés par `_`, ex. `caravaca-de-la-cruz` → `CARAVACA_DE_LA_CRUZ`). Ne pas utiliser de tirets ni un slug raccourci.

| Variable | Exemple / valeur |
|----------|------------------|
| `CLIENT_BASE_URL_COPENHAGUE` | `https://clq-copenhague.netlify.app` |
| `STRIPE_COPENHAGUE_FULL_7D_PRICE_ID` | `price_xxx` (depuis Stripe) |
| `STRIPE_COPENHAGUE_LITE_7D_PRICE_ID` | si applicable |
| `STRIPE_COPENHAGUE_UPGRADE_7D_PRICE_ID` | si applicable |

Mettre à jour **CORS / ALLOWED_ORIGINS** pour inclure `https://clq-copenhague.netlify.app`.

Dans le code API (si nouvelle ville) :

- Ajouter `copenhague` aux villes supportées (checkout, claim-session, activate-code).
- Mapper les Price IDs et `CLIENT_BASE_URL_COPENHAGUE` dans la config (ex. `STRIPE_PRICE_IDS`, `FRONTEND_URLS`).

Redéployer l’API après modification.

---

## 6. Stripe

1. Créer les **produits / prix** 7 jours pour **Copenhague** (FULL, LITE, UPGRADE si utilisés).
2. Copier les **Price ID** (`price_…`) dans les variables Render ci-dessus.
3. **Webhook Stripe** → pointer vers **Render** (`https://cityloopquest-api.onrender.com/…`), pas vers Netlify.
4. Ne pas configurer d’URL success/cancel figées dans Stripe : elles sont construites côté API avec `CLIENT_BASE_URL_COPENHAGUE`.

Flux utilisateur après paiement : `post-checkout.html` → `claim-session` sur Render → redirection `index.html?from=checkout`.

---

## 7. Supabase

Projet partagé (recommandé) :

1. Table **`cities`** : entrée `slug = 'copenhague'`, nom `Copenhague`, métadonnées région.
2. Table **`stripe_products_map`** (ou équivalent) : lier les Price ID Stripe à `copenhague`.
3. Table **`licenses`** : codes d’activation `xxx-xxx-xxx` rattachés à `city = 'copenhague'`.

Tester un code manuel sur `activation-manual.html` et via le lien email post-achat.

---

## 8. Garde licence par ville

Le fichier `js/license-city-guard.js` associe le hostname au slug `copenhague`.  
Après génération Factory, vérifier que la ligne contient bien :

```js
if (host.includes('copenhague')) return 'copenhague';
```

**Piège :** `localStorage` est partagé entre tous les sites `*.netlify.app` sur un même navigateur. Une licence **Murcia** ne doit pas débloquer **Copenhague** — le guard compare `clq_city` / JWT au slug du site.

---

## 9. Checklist avant mise en prod

- [ ] `npm run build` sans erreur ; `dist/` contient audio, images, traductions JSON **valides**.
- [ ] Parcours petit / moyen / grand : GPS, audio, quiz.
- [ ] Paiement test Stripe → email → activation lien **et** saisie manuelle du code.
- [ ] Bouton 🏠 (reset) → `parcours.html` sans redemander le code si accès valide.
- [ ] `mail.html` mentionne **Copenhague**.
- [ ] PWA : `manifest.json`, icônes `cityLoopQuest_Copenhague_*.png`.
- [ ] Popup fin de parcours + invitation selfie.
- [ ] CORS : appels API depuis https://clq-copenhague.netlify.app OK.

---

## 10. Dépannage fréquent

| Symptôme | Piste |
|----------|--------|
| `Failed to fetch` / API indisponible en local | Vider `localStorage.api_base` ou `useProdApi()` ; ne pas laisser `localhost:8081` si l’API locale n’est pas lancée. |
| Activation manuelle échoue, lien email OK | Même origine API ; vérifier `api-base.js` et CORS Render. |
| Service Worker / SSL en local | Sous Vite (`npm run dev`), le SW est désactivé. Sous `npm start` / preview HTTP, le SW s’enregistre pour tester l’install PWA. |
| Install PWA absente (barre d’adresse) | `index.html` doit charger `js/clq-sw-register.js` ; icônes `cityLoopQuest_Copenhague_{192,512}.png` présentes ; HTTPS ou localhost. |
| Licence d’une autre ville acceptée | `license-city-guard.js`, vider `localStorage` ou tester en navigation privée. |
| Images absentes / hotlink Wikimedia | La génération doit télécharger `directImageUrl`/`sourceUrl` vers `images/…`. Relancer la génération Factory ; vérifier les logs « Photos par lien » / « Relance téléchargement ». |
| Webhook 400 | Secret webhook Stripe = celui configuré sur Render. |

---

## 11. Mise à jour de contenu

1. Modifier le JSON dans la **Factory**.
2. Régénérer l’app (les images/audio existants sont préservés si déjà sur disque).
3. `npm run build && npm run deploy` dans `CLQ-COPENHAGUE/clq-copenhague`.

---

*Document généré pour CityLoop Quest Copenhague — slug `copenhague`.*
