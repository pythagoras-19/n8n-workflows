# Watchdog Pattern

**Workflow:** Self-Monitoring Health Check
**RTC Relevance:** Low — monitors execution health, can push live alerts

## Why This Pattern

A workflow that silently starts failing every run is worse than one that fails loudly once. A watchdog checks for a *pattern* of failures — not just one blip, which might be a transient fluke — and only alerts when that pattern is real, keeping alert noise down.

## Workflow Structure

1. Schedule Trigger (e.g. every 15 minutes)
2. HTTP Request — n8n's own REST API (`/api/v1/executions`) for recent runs
3. Code node — failure-pattern detection
4. IF node — only notify on a genuine pattern

## Code Example

```javascript
const executions = $json.data ?? []; // from GET {n8n_url}/api/v1/executions?status=error&limit=10

const recentFailures = executions
  .filter((e) => e.status === 'error')
  .sort((a, b) => new Date(b.startedAt) - new Date(a.startedAt));

const CONSECUTIVE_THRESHOLD = 3;
const WINDOW_MINUTES = 30;

const now = Date.now();
const withinWindow = recentFailures.filter(
  (e) => now - new Date(e.startedAt).getTime() < WINDOW_MINUTES * 60_000
);

const isPattern = withinWindow.length >= CONSECUTIVE_THRESHOLD;

return [{
  json: {
    shouldAlert: isPattern,
    failureCount: withinWindow.length,
    workflowIds: [...new Set(withinWindow.map((e) => e.workflowId))],
  },
}];
```

## Notes

Requires an n8n API key (Settings → API) for the HTTP Request to authenticate against n8n's own REST API — still free, just a local credential, not a paid add-on.
