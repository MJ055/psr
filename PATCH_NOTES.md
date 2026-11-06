# PasarGuard — customized build

Base: **PasarGuard/panel v5.6.0** (upstream `main`, 2026-11-03)

## Changes

### 1. Backup manager (sidebar → Maintenance → Backups)
- New API: `GET/POST /api/backups`, `GET /api/backups/{name}/download`,
  `POST /api/backups/{name}/restore`, `DELETE /api/backups/{name}` (owner only).
- Engine aware: SQLite snapshot (`sqlite3.backup`, WAL safe), PostgreSQL `pg_dump`/
  `pg_restore`, MySQL `mysqldump`, with a portable JSON logical dump as fallback so a
  backup taken on one engine can be restored on another.
- Every restore keeps an automatic **pre-restore safety copy**.
- Retention (`BACKUP_MAX_KEEP`, default 20) + optional scheduler
  (`BACKUP_AUTO_ENABLED`, `BACKUP_AUTO_INTERVAL_HOURS`).
- Files: `app/utils/backup.py`, `app/routers/backup.py`, `app/models/backup.py`,
  `app/jobs/auto_backup.py`, `dashboard/src/pages/_dashboard.backups.tsx`,
  `dashboard/src/service/backups.ts`.

### 2. Donation removed
- Sidebar "Support Us" entry, donation popup, goal progress widget and the top ad
  banner are gone (`donation-popup.tsx`, `goal-progress.tsx`, `use-goal.ts` deleted,
  `DONATION_URL` removed).

### 3. Admin limits disabled
- `ENFORCE_ADMIN_LIMITS=false` (default) short-circuits role/override quota checks in
  `app/operation/user.py`, the admin auto-limiting job (`app/jobs/review_admins.py`)
  and the periodic traffic-reset job (`app/jobs/reset_user_data_usage.py`) — so
  creating accounts/configs is never blocked by user-count or traffic caps.
- User dialog now sends `data_limit: null` (unlimited) when no positive limit is set
  and labels the field "Unlimited · not enforced".

### 4. Login screen
- Modern dotted + grid backdrop with soft primary glow, glass card, gradient accent,
  light/dark aware, reduced-motion friendly (`index.css`: `.login-*`).

### 5. "What's new" window after login
- `dashboard/src/components/layout/whats-new-dialog.tsx` — shown once per release
  (versioned localStorage key), lists every change with quick links.

### 6. Instant owner sign-up
- `GET /api/setup/signup` + `POST /api/setup/signup` create the owner directly from the
  login page (no temp key); can be disabled with `SELF_REGISTRATION=False`.
  Auto-owner creation from `SUDO_USERNAME`/`SUDO_PASSWORD` on first boot.

### 7. Railway deployment
- `Dockerfile.railway`, `railway/entrypoint.sh`, `railway.json`, `railway.env.example`,
  `docker-compose.railway.yml`, `RAILWAY.md`.
- i18n (`en`, `fa`) keys for every new string.
