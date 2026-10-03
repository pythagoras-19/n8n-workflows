# n8n Workflows

Self-hosted n8n, run manually via `npx`, with workflow definitions version-controlled as JSON.

## Running n8n

```bash
npx n8n start
```

n8n will be available at http://localhost:5678. Its data (credentials, execution history, settings) lives in `~/.n8n` by default and is not stored in this repo.

## Exporting / importing workflows

Workflow JSON lives in `workflows/`, so it can be committed to git.

Export all workflows:

```bash
npx n8n export:workflow --all --output=workflows/ --separate
```

Import all workflows:

```bash
npx n8n import:workflow --input=workflows/ --separate
```

Note: credentials are not included in workflow exports and should not be committed. Re-create credentials manually in the n8n UI on a new instance.
