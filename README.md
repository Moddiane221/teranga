# TerangaImmo — Plateforme d'annonces immobilières

Scaffold Next.js 14 (App Router) + Prisma/PostgreSQL + Stripe, prêt à déployer sur **Netlify**.

## Ce qui est fonctionnel dans ce scaffold

- Inscription / connexion avec sessions JWT (cookie httpOnly) et 5 rôles (Admin, Agence, Agent, Propriétaire, Acheteur)
- Modèle de données complet (Prisma) : annonces, catégories, villes/pays, favoris, messages, packs, abonnements, paiements, codes promo, réglages du site
- Publication d'annonces avec modération (statut EN_ATTENTE → PUBLIEE/REFUSEE)
- Recherche multicritères (ville, type, mots-clés, prix)
- Page annonce avec balisage SEO schema.org (RealEstateListing)
- Paiement **Stripe** réel (Checkout + webhook) pour les abonnements
- Tableau de bord utilisateur et back-office admin (statistiques, modération)
- Middleware de protection des routes par rôle

## Ce qu'il reste à compléter avant une mise en production réelle

Le cahier des charges d'origine est très large ; certains éléments demandent des comptes/contrats tiers que je ne peux pas configurer à votre place :

| Élément | Statut | À faire |
|---|---|---|
| Wave, Orange Money, Free Money, Unitech Pay | Squelette (`src/lib/payments/*.ts`) | Créer un compte marchand chez chaque opérateur, implémenter `initiate()`/`verify()` selon leur doc API |
| PayPal | Squelette | Créer une app PayPal Developer, implémenter l'Orders API |
| Upload photos/vidéos (jusqu'à 30 photos) | Non inclus | Brancher un service de stockage (Cloudinary, AWS S3, Uploadthing) |
| Carte interactive | Non inclus | Ajouter Mapbox GL ou Google Maps avec les champs `latitude`/`longitude` déjà présents en base |
| Notifications Email/SMS/WhatsApp | Non inclus | Ex. Resend/SendGrid (email), Twilio (SMS/WhatsApp) |
| Facture PDF automatique | Non inclus | Générer avec `@react-pdf/renderer` à la confirmation d'un paiement |
| PWA | Non inclus | Ajouter un `manifest.json` + service worker (ex. `next-pwa`) |
| Unity Ads / Google AdSense | Non inclus | Scripts tiers à insérer selon leurs consoles respectives |
| Messagerie interne temps réel | Modèle de données prêt, UI non incluse | Ajouter Pusher/Ably ou polling simple |

## Déploiement sur Netlify

### 1. Base de données PostgreSQL externe

Netlify n'héberge pas de base de données : créez-en une gratuitement chez **[Neon](https://neon.tech)** ou **[Supabase](https://supabase.com)**, puis copiez la chaîne de connexion (`DATABASE_URL`).

```bash
# En local, une fois DATABASE_URL renseignée dans .env
npm install
npm run db:push      # crée les tables à partir de prisma/schema.prisma
npm run db:seed       # (optionnel) insère des données de démonstration
```

### 2. Déposer le code sur GitHub

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin <votre-repo-github>
git push -u origin main
```

### 3. Connecter le dépôt à Netlify

1. Sur [app.netlify.com](https://app.netlify.com) → **Add new site → Import an existing project**.
2. Sélectionnez votre dépôt GitHub.
3. Netlify détecte automatiquement Next.js grâce au fichier `netlify.toml` (plugin `@netlify/plugin-nextjs` déjà configuré).
4. Dans **Site settings → Environment variables**, ajoutez toutes les variables de `.env.example` (au minimum `DATABASE_URL` et `JWT_SECRET` pour démarrer ; ajoutez les clés Stripe pour activer les paiements).
5. Cliquez sur **Deploy site**.

### 4. Configurer le webhook Stripe

Une fois le site en ligne, dans le dashboard Stripe → Developers → Webhooks, ajoutez un endpoint :
`https://votre-site.netlify.app/api/payments/stripe/webhook`
et sélectionnez l'événement `checkout.session.completed`. Copiez le secret de signature généré dans `STRIPE_WEBHOOK_SECRET`.

## Développement local

```bash
cp .env.example .env   # puis renseignez DATABASE_URL et JWT_SECRET au minimum
npm install
npm run db:push
npm run dev
```

Compte admin de démo après `npm run db:seed` : `admin@terangaimmo.com` / `motdepasse123`
