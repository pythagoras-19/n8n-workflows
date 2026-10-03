# n8n Workflows

Self-hosted n8n via Docker, with workflow definitions version-controlled as JSON.

## Setup

1. Copy `.env.example` to `.env` and fill in values (generate `N8N_ENCRYPTION_KEY` with `openssl rand -hex 32`).
2. Start n8n:

   ```bash
   docker compose up -d
   ```

3. Open http://localhost:5678 and log in with the basic auth credentials from `.env`.

## Stopping

```bash
docker compose down
```

n8n's own data (credentials, execution history, settings) lives in the `n8n_data` Docker volume and persists across restarts. It is not stored in this repo.

## Exporting / importing workflows

Workflow JSON lives in `workflows/`, mounted into the container at `/home/node/workflows`, so it can be committed to git.

Export all workflows:

```bash
docker compose exec n8n n8n export:workflow --all --output=/home/node/workflows/ --separate
```

Import all workflows:

```bash
docker compose exec n8n n8n import:workflow --input=/home/node/workflows/ --separate
```

Note: credentials are not included in workflow exports and should not be committed. Re-create credentials manually in the n8n UI on a new instance.
