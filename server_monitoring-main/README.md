# Server Monitoring API

A FastAPI-based monitoring backend that ingests metrics & file-hash batches from agents, stores raw JSON snapshots to disk, writes lightweight rows to your own `logs` table, maintains an in‑memory metrics cache for the UI, and raises alerts (with optional email via Gmail SMTP).

## Features
- `POST /collect-metrics` — save incoming metrics JSON to disk, insert a row into your `logs` table, update in‑memory cache, raise **RESOURCE** alerts on thresholds (CPU/RAM).
- `POST /collect-hashes` — bulk insert file hashes, compare against an IOC table, raise **HASH** alerts on matches.
- `GET /api/metrics` — returns current servers + history arrays for the UI.
- `GET /api/alerts` — latest alerts.
- `GET /api/logs` — latest rows from your `logs` table (id, device_id, device_name, log_path, severity, created_at).
- `POST /collect-data` — compatibility alias for `/collect-metrics`.
- `GET /health` — simple health check.
- Serves `/static` (e.g., `index.html`) if present.

---

## Requirements (pip)
Install the following Python packages (Python 3.10+ recommended):

```bash
pip install \
  fastapi \
  "uvicorn[standard]" \
  python-dotenv \
  "SQLAlchemy>=2.0" \
  psycopg2-binary
```

**Notes**
- `python-dotenv` is used to load the `.env` file when you run locally.
- `psycopg2-binary` is **only needed if you use PostgreSQL** via `DATABASE_URL=postgresql+psycopg2://...`.  
  If you stick with the default SQLite URL (`sqlite:///logs.db`), you can skip it.

Standard‑library modules used (no install needed): `os`, `json`, `datetime`, `pathlib`, `collections`, `smtplib`, `typing`.

---

## Quickstart

1) **Clone / copy** the project files.

2) **Create your `.env`** in the project root:
```env
# --- SMTP for email alerts (Gmail example) ---
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=you@gmail.com
SMTP_PASS=your_16_char_app_password
ALERT_EMAILS=you@gmail.com, teammate@example.com

# --- DB config ---
# Default (SQLite file in project dir):
DATABASE_URL=sqlite:///logs.db
# Or PostgreSQL (requires psycopg2-binary):
# DATABASE_URL=postgresql+psycopg2://user:pass@host:5432/dbname

# --- Optional tuning ---
CPU_HIGH=90
RAM_RATIO_HIGH=0.9
METRICS_STALE_SECS=30

# --- Optional IOC integration (external table) ---
# IOC_SCHEMA=public
# IOC_TABLE_NAME=suspicious_hashes
# IOC_COL_SHA=sha256
# LOGS_TABLE_NAME=logs  # defaults to "logs" if unset
```

3) **Run the API** (choose one):

- Load `.env` automatically via Uvicorn:
```bash
uvicorn app:app --reload --env-file .env
```
- Or load in code (already supported) and run:
```bash
uvicorn app:app --reload
```

4) **Ping it**:
```bash
curl http://127.0.0.1:8000/health
```

---

## Verify your environment variables
(Dev only) Add this debug route to confirm variables are visible without exposing secrets:
```python
@app.get("/env-debug")
def env_debug():
    import os
    def present(k): return bool(os.getenv(k) and os.getenv(k).strip())
    keys = ["SMTP_HOST","SMTP_PORT","SMTP_USER","SMTP_PASS","ALERT_EMAILS",
            "DATABASE_URL","LOGS_TABLE_NAME","IOC_SCHEMA","IOC_TABLE_NAME","IOC_COL_SHA"]
    return {k: present(k) for k in keys}
```
Open `GET /env-debug` — each expected key should be `true`.

---

## Gmail: Create an App Password (required)
Gmail blocks basic username/password SMTP with 2‑Step Verification enabled; you must use an **App Password**. If you don’t have 2‑Step Verification, enable it first.

1. Go to **Google Account → Security**: <https://myaccount.google.com/security>
2. Under **“How you sign in to Google”**, enable **2‑Step Verification** (follow the setup).
3. After 2‑Step Verification is on, open **App passwords** (it appears under the same section).
4. In **Select app**, choose **Mail** (or **Other** and type "Server Monitoring").
5. In **Select device**, choose your device (or **Other**).
6. Click **Generate**. Google shows a **16‑character password** (spaces are fine). Copy it.
7. Put that value into your `.env` as `SMTP_PASS` (without spaces is also OK).
8. Keep `SMTP_USER` as your full Gmail address and use:
   - `SMTP_HOST=smtp.gmail.com`
   - `SMTP_PORT=587` (TLS)

**Example**:
```env
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=you@gmail.com
SMTP_PASS=abcd efgh ijkl mnop
ALERT_EMAILS=you@gmail.com
```

---

## Email sending checklist
- Make sure `ALERT_EMAILS` is a valid comma/semicolon‑separated list of emails.
- Ensure your code calls `sendmail(user, recipients, message)` — **recipients must be a list of addresses** (not the password).
- For Gmail, the `MAIL FROM` should normally match the authenticated account (`SMTP_USER`).

---

## Endpoints (recap)
- `POST /collect-metrics`  
- `POST /collect-hashes`  
- `POST /collect-data` (alias)  
- `GET /api/metrics`  
- `GET /api/alerts`  
- `GET /api/logs`  
- `GET /health`

---

## Running behind PostgreSQL
If you switch to PostgreSQL, install `psycopg2-binary` and update `DATABASE_URL`. Example:
```env
DATABASE_URL=postgresql+psycopg2://monitor_user:monitor_pass@localhost:5432/monitor_db
```
The app will create its internal tables (`alerts`, `file_hashes`) if missing. Your `logs` table name defaults to `logs` (override with `LOGS_TABLE_NAME`).

---

## Troubleshooting
- **Env not loaded**: ensure `--env-file .env` or that you call `load_dotenv(...)` before reading variables.
- **553 invalid recipient**: you probably passed the password as the recipient by mistake — pass a list of emails to `sendmail`.
- **Auth error**: confirm `SMTP_USER` (full address) and `SMTP_PASS` (app password). App passwords are per-account; regenerate if revoked.
- **DB driver error**: install `psycopg2-binary` for PostgreSQL URLs; for SQLite no extra driver is needed.
- **CORS/static**: FastAPI mounts `/static`; ensure the folder exists if you serve a UI.

---

## License
MIT (or your choice)
