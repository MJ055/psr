# 🚀 Deploying this PasarGuard build on Railway

This repository is a **customized PasarGuard panel** with:

- 🗄️ a **Backup manager** in the sidebar (create / download / restore / delete + optional scheduled backups)
- 🧹 the **donation** section, donation popup and top ad banner removed
- 🚫 **no admin limits** — user-count and traffic caps no longer block creating accounts/configs, and periodic traffic resets stay idle
- ✨ a **modern dotted login screen** + a **"What's new" dialog** right after login
- 🐳 a self-contained **Dockerfile.railway** that builds the dashboard inside the image

Everything below assumes a fresh Railway account.

---

## 1. Put the code on GitHub

```bash
cd pg-panel
git init
git add .
git commit -m "feat: PasarGuard panel with backups + Railway deploy"
git branch -M main
git remote add origin https://github.com/<YOUR-USER>/<YOUR-REPO>.git
git push -u origin main
```

> The prebuilt dashboard (`dashboard/build/`) is **not** required — the Railway
> image builds it from source. If you want the smallest possible repo, add
> `dashboard/build` to `.gitignore`.

---

## 2. Create the Railway project

**Option A — one click**

1. Railway → *New Project* → *Deploy from GitHub repo* → pick your repo.
2. Railway auto-detects `railway.json` and builds `Dockerfile.railway`.
3. Open the service → *Settings → Networking → Generate Domain*.
4. Panel URL: `https://<generated-domain>/dashboard/`

**Option B — Railway CLI**

```bash
npm i -g @railway/cli
railway login
railway init
railway up
railway domain
```

---

## 3. Set the variables

Railway → service → **Variables** (or *Raw Editor* and paste the block below):

```env
UVICORN_HOST=0.0.0.0
UVICORN_PROXY_HEADERS=True
UVICORN_FORWARDED_ALLOW_IPS=*
ALLOWED_ORIGINS=*
SELF_REGISTRATION=True
ENFORCE_ADMIN_LIMITS=False
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=1440

# optional: create the owner automatically on first boot (12+ chars, 2 upper, 2 lower, 2 digits, 1 symbol)
SUDO_USERNAME=admin
SUDO_PASSWORD=ChangeMe@Panel2026

# optional: managed PostgreSQL instead of SQLite
# SQLALCHEMY_DATABASE_URL=postgresql+asyncpg://user:pass@host:5432/railway
```

`railway.env.example` documents every variable.

> **Database.** Without `SQLALCHEMY_DATABASE_URL` the panel runs on SQLite stored at
> `/var/lib/pasarguard/db.sqlite3`. On Railway attach a **Volume** mounted at
> `/var/lib/pasarguard` so data survives redeploys. For heavier use, add the
> **PostgreSQL** plugin and set `SQLALCHEMY_DATABASE_URL` to its connection string with
> `postgresql+asyncpg://`.

---

## 4. Volume (recommended)

Service → *Settings → Volumes* → **Add Volume** → mount path `/var/lib/pasarguard`.

This keeps the SQLite database **and** every backup (`/var/lib/pasarguard/backups`)
across deploys. The `pg_dump` / `mysqldump` / `pg_restore` clients are already installed
in the image, so backups also work against PostgreSQL or MySQL.

---

## 5. First login

1. Open `https://<your-domain>/dashboard/`.
2. If you set `SUDO_USERNAME` / `SUDO_PASSWORD`, log in with those.
3. Otherwise click **Create owner account** on the login page (no temp key needed) —
   or run `railway run pasarguard-cli generate-temp-key` and use *Owner access*.
4. The **"What's new"** window lists everything added in this build.
5. Sidebar → **Maintenance → Backups** to take your first snapshot.

---

## 6. Health, updates, troubleshooting

| Item | Value |
| --- | --- |
| Health endpoint | `GET /health` → `200` |
| Panel path | `/dashboard/` |
| Data dir | `/var/lib/pasarguard` |
| Backups | `/var/lib/pasarguard/backups` |

- **Build fails on `bun install`** — the lockfile drifted; the Dockerfile falls back to
  a plain `bun install` automatically, just rebuild.
- **`/health` returns 502** — first boot runs migrations; give it ~60 s
  (`restartPolicyType: ON_FAILURE` retries automatically).
- **Forgot the password** — `railway run pasarguard-cli generate-temp-key`, then use
  *Owner access → Reset* on the login page.
- **Restore a backup** — Sidebar → Backups → *Restore*. A safety copy of the current
  database is created first, so a bad restore is always reversible.

---

## 7. Local / VPS alternative

```bash
cp railway.env.example .env      # then edit
docker compose -f docker-compose.railway.yml up -d --build
# → http://localhost:8000/dashboard/
```

or without Docker:

```bash
uv sync                                     # python >= 3.14
./build_dashboard.sh                        # needs bun
python -m alembic upgrade head
python main.py
```
