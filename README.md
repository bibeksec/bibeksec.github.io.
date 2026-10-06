# Bibek — Offensive Security Portfolio (Full Stack)

A working website for an ethical hacker / penetration tester with a real backend:
public site + a contact form that saves engagement requests to a database, and
a password-protected admin dashboard to review and manage them.

## Stack

- **Backend:** Node.js + Express
- **Database:** SQLite (via `better-sqlite3`) — a single file, no external DB server needed
- **Security:** Helmet (HTTP headers), rate limiting on the public form, token-based admin auth, input validation, HTML-escaping in the admin UI
- **Frontend:** Plain HTML/CSS/JS (no build step) in `/public`

## Project structure

```
bibek-site/
├── server.js          # Express app, all API routes
├── db.js              # SQLite connection + schema
├── public/
│   ├── index.html      # Public site (with working contact form)
│   └── admin.html       # Admin dashboard (token-protected)
├── data/
│   └── bibek.db        # SQLite database (created automatically)
├── .env.example        # Copy to .env and fill in
└── package.json
```

## 1. Install

```bash
cd bibek-site
npm install
```

## 2. Configure

```bash
cp .env.example .env
```

Generate a real admin token and put it in `.env`:

```bash
node -e "console.log(require('crypto').randomBytes(24).toString('hex'))"
```

Paste the output as `ADMIN_TOKEN` in `.env`. **Never share this token or commit `.env`.**

## 3. Run

```bash
npm start
```

Visit:
- **Public site:** http://localhost:3000
- **Admin dashboard:** http://localhost:3000/admin.html (enter your `ADMIN_TOKEN` to log in)

For auto-restart on file changes during development:
```bash
npm run dev
```

## How the contact form works

1. A visitor fills out the form on the public site and submits.
2. The browser sends a `POST /api/contact` request with their details.
3. The server validates the input (organization, email format, engagement type, scope length), rate-limits by IP (5 submissions per 15 minutes), and saves it to the SQLite database.
4. You review new requests in `/admin.html`, update their status (new → reviewing → scoping → closed/declined), or delete spam.

## API reference

| Method | Route | Auth | Description |
|---|---|---|---|
| POST | `/api/contact` | none (rate-limited) | Submit an engagement request |
| GET | `/api/admin/submissions` | Bearer token | List submissions, optional `?status=` filter |
| PATCH | `/api/admin/submissions/:id` | Bearer token | Update a submission's status |
| DELETE | `/api/admin/submissions/:id` | Bearer token | Delete a submission |
| GET | `/api/health` | none | Health check |

Admin routes require an `Authorization: Bearer <ADMIN_TOKEN>` header — the admin dashboard handles this for you once you log in.

## Deploying it for real

This app has no external dependencies beyond Node, so it deploys easily to:

- **Render / Railway / Fly.io:** connect your repo, set the `ADMIN_TOKEN` env var in their dashboard, done. (On platforms with ephemeral filesystems, attach a persistent disk/volume for the `data/` folder so the database survives restarts.)
- **A VPS (DigitalOcean, Linode, etc.):** `git clone`, `npm install`, run with `pm2 start server.js` or as a `systemd` service, put Nginx in front for TLS.
- **Docker:** straightforward to containerize — ask if you'd like a `Dockerfile` added.

Put the site behind HTTPS in production (Render/Railway do this automatically; on your own VPS use Let's Encrypt via Certbot).

## Security notes

- The `ADMIN_TOKEN` is your only line of defense for the admin dashboard — treat it like a password. Rotate it if you think it's leaked.
- The contact form only ever writes to the database; it can't execute anything, so it isn't a vector for someone to run commands on your server.
- Submitter IP addresses are stored as a one-way hash (not the raw IP), enough to spot abuse patterns without keeping raw personal data.
- Consider adding real email notifications (e.g. via Resend, Postmark, or SMTP) when a new submission comes in — happy to wire that up if you want it.
