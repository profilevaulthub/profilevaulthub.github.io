# ProfileVault — Cloud Profile + Login Vault

A GoLogin-style profile manager as a **web app** (not a browser). Create an account,
create profiles (Personal, Work, Family...), and save your site logins in each profile.
Everything is encrypted in your browser with your own vault passphrase before it is
stored in the cloud, and it syncs across devices — change device, log in once, and
your saved logins are there.

## What it is

- Single-page app: pure HTML/CSS/vanilla JS (`index.html`), Firebase Auth + Firestore
  via CDN, CryptoJS (AES + PBKDF2) via CDN.
- Sign up / log in with email + password, or Continue with Google.
- Vault passphrase (min 6 chars) set on first login; PBKDF2 (10,000 iterations) derives
  the encryption key from the passphrase + a random salt stored in Firestore.
  A stored encrypted check string (`vault-check`) detects a wrong passphrase on unlock.
- Profiles: create, rename, delete (with confirm). Default profile "Personal" is created
  on first unlock.
- Saved logins per profile: site, URL, username, password, notes — **all AES-encrypted
  before storage**. Firestore never sees a plaintext password or username.
- Entry actions: Show/Hide password, Copy username, Copy password, Open site (new tab),
  Edit, Delete (confirm), search by site/username, password generator.
- "Lock vault" clears the key from memory without signing you out.

## Setup steps

1. **Create a Firebase project** at https://console.firebase.google.com
   (free Spark plan is enough to start).
2. **Authentication > Sign-in method**: enable **Email/Password** and **Google**.
   For Google sign-in, add your deployed domain to the authorized domains list.
3. **Firestore Database**: create a database (production mode), pick a region.
4. **Deploy the security rules**: in the Firebase console go to Firestore > Rules,
   paste the contents of `firestore.rules`, and Publish. Or with the Firebase CLI:
   `firebase deploy --only firestore:rules`
   (Rules allow each signed-in user to read/write only their own `/users/{uid}` tree.)
5. **Get the web config**: Project settings > Your apps > Web app > copy the config object.
6. **Paste the config** into `index.html` — replace the placeholder values inside the
   clearly marked `FIREBASE CONFIG` block near the top of the `<script>`.
   Until a real `apiKey` is pasted, the app shows a friendly "setup pending" notice
   instead of crashing.
7. **Deploy to GitHub Pages**: upload the folder contents to a repo, enable Pages
   from the main branch, and replace `REPLACE-WITH-YOUR-DOMAIN` in `sitemap.xml`
   and `robots.txt` with your real domain (e.g. `https://profilevault.github.io`).

## Data model (Firestore)

- `users/{uid}` — `vaultSalt` (hex), `vaultCheck` (AES-encrypted "vault-check"), `createdAt`
- `users/{uid}/profiles/{profileId}` — `name`, `color`, `createdAt`
- `users/{uid}/profiles/{profileId}/entries/{entryId}` — `site`, `url`, `username`,
  `password`, `notes` (all AES-encrypted strings), `createdAt`, `updatedAt`

## Honest limits — read this

- **This is a password vault, NOT a browser.** It cannot keep you logged in to
  third-party sites across devices the way GoLogin does. If you open a saved site
  on a new device, that site will still ask you to log in — but your saved username
  and password are one click away (Copy), so logging in again takes seconds.
- **If you forget your vault passphrase, your saved logins cannot be recovered.**
  There is no reset. The passphrase is never stored anywhere; only the salt and an
  encrypted check string are stored.
- Passwords are encrypted client-side with AES using a PBKDF2-derived key. The
  account password (Firebase Auth) and the vault passphrase are two different
  things — keep them different.
- Security uses `crypto.getRandomValues` for salts, IDs and generated passwords.
  No `Math.random` is used for anything security-related.
- No plaintext passwords or usernames are ever written to Firestore.
