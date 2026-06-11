# BigRock cPanel Deployment Guide
## Logix Finance & Investment — Static React Site

This is a **static React Single Page Application**. No Node.js or Python is
required on the server. Everything runs on plain Apache + PHP (for the contact
form), which BigRock shared hosting provides by default.

---

## 1. What you upload

Upload **ONLY the contents of the `build/` folder** to `public_html`.

After upload, your `public_html` should look like this:

```
public_html/
├── index.html              ← MUST be directly inside public_html
├── .htaccess               ← React Router fallback + caching + security
├── contact.php             ← Mail handler for the Contact form
├── asset-manifest.json
└── static/
    ├── css/
    │   └── main.<hash>.css
    └── js/
        └── main.<hash>.js
```

**Do NOT upload** any of: `src/`, `node_modules/`, `tests/`, `backend/`,
`memory/`, `.git/`, `package.json`, `craco.config.js`, etc. These are
build-time files only.

---

## 2. Step-by-step deployment

### Option A — Build locally, then upload (recommended)

On your local machine (Node.js 18+ required):

```bash
# 1. Install dependencies (one-time)
cd frontend
yarn install      # or: npm install

# 2. Build the production bundle
yarn build        # or: npm run build
```

This creates a `frontend/build/` folder. Now upload its contents:

1. Log in to **BigRock cPanel** → **File Manager** → open `public_html`.
2. **Delete anything that's currently in `public_html`** that was wrongly
   uploaded earlier (source files, `node_modules`, `package.json`, etc.).
3. Open `frontend/build/` on your computer.
4. Select **everything inside** `build/` (Ctrl/Cmd + A) — including the
   hidden `.htaccess` file.
5. Drag-and-drop / upload into `public_html`.
6. Make sure `index.html` is **directly inside** `public_html`, not inside
   any subfolder like `public_html/build/`.

> **Tip:** if `.htaccess` is not visible in File Manager, click
> **Settings → Show Hidden Files (dotfiles) → Save**.

### Option B — Upload the pre-built ZIP

If you don't want to install Node locally:

1. Take the file `logix-finance-cpanel.zip` (already generated for you).
2. In cPanel **File Manager**, go to `public_html`.
3. Delete the wrong files that are there from your earlier upload.
4. Click **Upload** and upload `logix-finance-cpanel.zip`.
5. Right-click the uploaded zip → **Extract** → extract into `public_html`.
6. Verify `index.html` is at `public_html/index.html`.
7. Delete the now-empty `logix-finance-cpanel.zip`.

---

## 3. Configure the Contact form email address

The contact form posts to `contact.php`. **You must edit one line:**

1. In cPanel File Manager, open `public_html/contact.php`.
2. Find this line near the top:

   ```php
   $TO_EMAIL = "info@logixfinanceandinvestment.com";
   ```

3. Change it to the actual inbox where you want enquiries delivered, for
   example:

   ```php
   $TO_EMAIL = "support@yourdomain.in";
   ```

4. Save.

That's it — the form is now wired. PHP's `mail()` function is enabled on
BigRock shared hosting by default, so no SMTP setup is required.

> **Recommended:** for the most reliable delivery (so emails don't go to
> spam), the receiving address should be on the **same domain** as your
> website (e.g., `info@yourdomain.com`). Create the mailbox via
> cPanel → **Email Accounts**.

---

## 4. After upload — verify these URLs

Open each in a browser:

| URL                                      | Should show                              |
|------------------------------------------|------------------------------------------|
| `https://yourdomain.com/`                | Home page                                |
| `https://yourdomain.com/about`           | About Us (refresh works — no 404)        |
| `https://yourdomain.com/loan-products`   | Loan Products                            |
| `https://yourdomain.com/how-it-works`    | How It Works                             |
| `https://yourdomain.com/contact`         | Contact page                             |
| `https://yourdomain.com/policies/interest-rate-policy` | Interest Rate Policy       |
| `https://yourdomain.com/policies/grievance-redressal`  | Grievance Redressal        |

Then submit the contact form once — you should receive the test email at
`$TO_EMAIL`.

---

## 5. Common issues and fixes

| Symptom | Cause | Fix |
|---|---|---|
| Refreshing `/about` shows **404 Not Found** | `.htaccess` missing or hidden files not uploaded | Re-upload `.htaccess` from `build/`. Enable "Show Hidden Files" in cPanel. |
| Home page loads but **CSS/JS broken** | Files uploaded into a sub-folder | Make sure `index.html` is directly in `public_html`, not in `public_html/build/`. |
| Contact form shows "Could not send" | PHP mail not configured for this domain | Edit `contact.php` and set `$TO_EMAIL` to a mailbox on your own domain (created in cPanel → Email Accounts). |
| Mixed content / HTTPS warnings | SSL not active yet | cPanel → **SSL/TLS Status** → **Run AutoSSL**. The `.htaccess` already redirects HTTP → HTTPS. |
| Old version still showing | Browser cache | `index.html` already has `no-cache` headers; do a hard refresh (Ctrl+F5) once. |

---

## 6. Re-deploying after updates

Whenever you make a code change:

```bash
cd frontend
yarn build
```

Then upload the **new** contents of `build/` into `public_html`, overwriting
existing files. You do **not** need to re-do the `contact.php` edit unless
you delete the file.

---

## 7. What's running where

- **HTML / CSS / JS** — served as static files by Apache. No build tools needed on the server.
- **React Router** — runs entirely in the browser. `.htaccess` rewrites all unknown URLs back to `index.html` so the React app can handle them.
- **Contact form** — posts JSON to `/contact.php`, which uses PHP `mail()` to deliver the enquiry to your inbox.

Nothing else is required on the server side.

---

If you run into anything, share a screenshot of `public_html` (with hidden
files visible) and the error from the browser console.
