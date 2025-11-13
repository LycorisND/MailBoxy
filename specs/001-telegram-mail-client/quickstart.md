# Quickstart: Telegram Mail Client (local)

This quickstart shows how to run the MailBoxy feature locally for development.

Prerequisites

- Python 3.11
- Node 18+ (for frontend)
- PostgreSQL running locally
- Redis (for task queue) - optional for dev but recommended

Environment variables (example):

- `DATABASE_URL` - Postgres connection string
- `TELEGRAM_BOT_TOKEN` - Telegram bot token
- `OAUTH_CLIENT_ID_<PROVIDER>` and `OAUTH_CLIENT_SECRET_<PROVIDER>` for each provider
- `ENCRYPTION_KEY` - symmetric key for token encryption (use KMS in production)
- `FRONTEND_URL` - e.g., `https://localhost:5173` (for redirect URIs during dev)

Run backend (development)

```bash
# From repo root
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
# Set env vars, then:
uvicorn backend.main:app --reload --port 8000
```

Run frontend

```bash
cd frontend
npm install
npm run dev
```

Notes

- For OAuth testing you may need to use a tunnel (ngrok) to expose `mailboxy.app` or set up provider redirect URIs for localhost as allowed by provider.
- Do not store real production secrets in local env during development.

