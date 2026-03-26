# CLAUDE.md — MediaNinja Portal

AI assistant guide for the MediaNinja Portal codebase.

---

## Project Overview

**MediaNinja Portal** is a client management web application for a digital marketing agency. It provides:
- A **client-facing portal** where clients view their marketing plan, goals, content calendar, brand guidelines, and reports.
- An **admin panel** where staff manage client accounts, edit data, and view statistics.

The entire application lives in a **single HTML file** (`index.html`, ~1,900 lines) with embedded CSS and JavaScript. There is no build step, no bundler, and no package manager.

---

## Repository Structure

```
medianinja/
├── index.html      # Entire application (HTML + CSS + JS, ~1,900 lines)
├── images/
│   └── logo.png    # MediaNinja logo (2000×2000 PNG)
└── CNAME           # GitHub Pages custom domain: portal.medianinjagroup.com
```

---

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Markup | HTML5 |
| Styling | CSS3 (custom properties, flexbox, grid, animations) |
| Logic | Vanilla JavaScript (ES6+) |
| Database | Firebase Realtime Database (v9.23.0 compat) |
| Charts | Chart.js v4.4.1 |
| Fonts | Google Fonts (Playfair Display, Plus Jakarta Sans) |
| Hosting | GitHub Pages |
| Domain | portal.medianinjagroup.com (CNAME) |

All external dependencies are loaded from CDN — there is no `package.json` or install step.

---

## index.html Internal Layout

The file is divided into three logical sections:

| Lines | Section | Description |
|-------|---------|-------------|
| 1–11 | `<head>` | Meta, CDN scripts, font imports |
| 12–874 | `<style>` | ~860 lines of inline CSS |
| 875–1396 | `<body>` | All HTML screens and modals |
| 1397–1906 | `<script>` | All JavaScript application logic |

### JavaScript Sections (within `<script>`)

Sections are marked with banner comments (`// ═══ SECTION ═══`):

1. **Firebase configuration & DB helpers** — `initFirebase()`, `dbReadClients()`, `dbWriteClient()`, `dbDeleteClient()`, `dbListen()`
2. **Auth credentials** — `ADMIN_CREDS` (hard-coded staff logins), `clientDefaults()`, default client seed data
3. **Screen & tab management** — `show()`, `clTab()`, `admTab()`, `SCREENS`, `CL_TABS`, `ADM_TABS`
4. **Authentication** — `clientLogin()`, `adminLogin()`
5. **Client dashboard rendering** — `loadClient()` and related helpers
6. **Admin panel** — `renderTable()`, `openEdit()`, `saveClientData()`, `addNewClient()`, `initAdminCharts()`
7. **Utility functions** — `x()`, `s()`, `h()`, `v()`, `g()`
8. **Email helpers** — `openEmail(type)`
9. **Boot sequence** — `DOMContentLoaded` handler

---

## Firebase Database Schema

```
/clients
  /[username]/
    name:          string        # Display name
    password:      string        # Plain-text password (see security note)
    plan:          'starter' | 'growth' | 'pro' | 'basic' | 'advance' | 'custom'
    planName:      string
    platforms:     string        # Social platforms served
    posts:         string        # Posts per month
    ads:           string        # Ad management description
    manager:       string        # Assigned account manager
    renewal:       string        # Renewal date
    report:        string        # Reporting schedule
    includes:      string        # Newline-separated list of inclusions
    goals:         string        # Newline-separated goals
    strategy:      string        # Marketing strategy text
    focus:         string        # Focus areas
    contentMonth:  string        # Month label for content calendar
    monthStart:    number        # Start day of month
    monthDays:     number        # Total days in month
    contentPosts:  [{ day, platform, topic }]
    looker:        string        # Looker Studio embed URL
    brandColors:   [{ hex, label }]
    typography:    string
    brandVoice:    string
    logoNotes:     string
    brandNotes:    string
```

---

## Authentication Model

### Client Login
- `clientLogin()` reads all clients from Firebase, then compares username and password in-browser.
- No server-side session; `currentClientUser` variable holds the logged-in client key.

### Admin Login
- `adminLogin()` checks against the hard-coded `ADMIN_CREDS` object.
- Current credentials: `admin / ninja2025`, `sarah / staff123`.
- After login, starts a Firebase real-time listener (`dbListen()`) so the admin table auto-updates.

---

## Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| CSS classes | kebab-case | `.login-scene`, `.plan-card` |
| CSS variables | kebab-case | `--navy`, `--gold`, `--text` |
| HTML IDs | kebab-case | `screen-client`, `cl-user` |
| JS variables | camelCase | `currentClientUser`, `firebaseReady` |
| JS functions | camelCase | `clientLogin()`, `renderTable()` |
| JS constants | UPPER_CASE | `FIREBASE_CONFIG`, `ADMIN_CREDS` |
| Short DOM utilities | single letter | `x()`, `s()`, `h()`, `v()`, `g()` |

### Utility Function Reference

```javascript
x(str)         // HTML-escape a string (XSS protection)
s(id, val)     // Set element textContent by ID
h(id, html)    // Set element innerHTML by ID
v(id, val)     // Set input value by ID
g(id)          // Get trimmed input value by ID
```

---

## Design System

**Color palette (CSS variables):**

| Variable | Hex | Usage |
|----------|-----|-------|
| `--navy` | `#0d1829` | Primary background |
| `--gold` | `#f0a500` | Accent, Starter plan |
| `--green` | `#10b981` | Pro plan, success states |
| `--red` | `#ef4444` | Alerts, danger |
| `--blue` | `#3b82f6` | Secondary accent |

**Typography:**
- Headings / taglines: **Playfair Display** (700, 800)
- Body / UI: **Plus Jakarta Sans** (300–700)

**Visual style:** Glass-morphism with mesh gradients; modern enterprise SaaS aesthetic.

---

## Pricing Plans

| Key | Name | Price | Posts |
|-----|------|-------|-------|
| `starter` | Starter | ฿18,000/mo | 8 posts |
| `growth` | Growth | ฿27,000/mo | 12 posts |
| `pro` | Pro | ฿42,000/mo | 16 posts |
| `custom` | Custom | Negotiated | Variable |

Plan details are rendered in the client dashboard's **My Plan** tab.

---

## Development Workflow

### Making changes

Since there is no build step, edits to `index.html` are immediately reflected when the file is opened in a browser or pushed to GitHub Pages.

1. Edit `index.html` directly.
2. Test locally by opening the file in a browser (note: Firebase calls require an internet connection).
3. Commit and push to `master` to deploy.

### No package installation required

There is no `npm install`, `pip install`, or any other dependency installation step. All libraries are loaded from CDN at runtime.

### Testing

There is no automated test suite. Verify changes manually in a browser.

### Deployment

Pushing to `master` automatically deploys to GitHub Pages at `portal.medianinjagroup.com`.

---

## Common Tasks

### Add a new client
In the **admin panel**, click "Add New Client". This calls `addNewClient()`, which merges `clientDefaults()` with the entered username and persists via `dbWriteClient()`.

### Edit client data
`openEdit(key)` populates the edit modal from Firebase. `saveClientData()` collects all form fields and writes back. Content posts and brand colors have their own add/remove helpers.

### Change plan features
Plan metadata (names, prices, feature lists) is embedded in the `loadClient()` function and the HTML template inside `openEdit()`. Update both if adding or changing a plan tier.

### Add a new admin credential
Add an entry to the `ADMIN_CREDS` object near the top of the `<script>` block (line ~1486).

### Embed a new Looker Studio report
Set the `looker` field for a client to the full `https://lookerstudio.google.com/embed/...` URL. It is rendered as an `<iframe>` in the client's **Live Dashboard** tab.

---

## Security Notes

The following are known security limitations of the current architecture. Be aware of them when making changes — do not inadvertently make them worse, and flag any opportunity to improve them.

1. **Hard-coded credentials:** Admin usernames/passwords and Firebase config are embedded in the source file and visible to anyone who views page source. Do not add new secrets to the HTML.
2. **Plain-text passwords:** Client passwords are stored unencrypted in Firebase. When modifying auth logic, do not add more sensitive data to the same storage pattern.
3. **Client-side authentication:** Login logic runs entirely in the browser. Avoid building features that rely on client-side auth as a security gate for truly sensitive operations.
4. **No input sanitization beyond `x()`:** The `x()` utility escapes HTML output. Always use `x()` when rendering user-supplied data into the DOM via `innerHTML`.
5. **Firebase rules:** Database access is controlled by Firebase Security Rules (configured in the Firebase console, not in this repo). Do not assume the database is read/write restricted without checking those rules.

---

## AI Assistant Guidelines

- **Do not add a build system** unless explicitly requested. The single-file approach is intentional.
- **Do not introduce npm/node dependencies.** If a library is needed, add it as a CDN `<script>` tag in `<head>`.
- **Preserve the section comment banners** (`// ═══ SECTION ═══`) — they are the primary navigation aid in the 500-line script block.
- **Use the existing utility functions** (`x`, `s`, `h`, `v`, `g`) rather than writing verbose `document.getElementById` chains.
- **Follow existing naming conventions** (see table above).
- **Always escape user data with `x()`** before inserting into innerHTML.
- **Do not refactor for its own sake.** The monolithic structure is a deliberate trade-off for simplicity of deployment. Propose structural changes only when they solve a concrete problem.
- **Do not commit `.env` files or new hard-coded secrets.** Treat the existing Firebase config as legacy; do not replicate the pattern.
- **Test in a browser** after any CSS or JS change — there is no lint/type-check step to catch errors.
