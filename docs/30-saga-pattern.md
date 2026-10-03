# Saga Pattern (Orchestration) — Capstone

**Workflow:** Automation Hub Orchestrator
**RTC Relevance:** High — combines live MQTT/RabbitMQ/webhook events with scheduled orchestration

## Why This Pattern

A multi-step process spanning several sub-workflows — ingest, transform, notify — needs one place that knows the overall sequence, can retry a failed step, and, in a full saga, can run compensating actions if a later step fails after an earlier one already succeeded.

## Workflow Structure

1. Schedule Trigger (nightly)
2. Code node — generate a run id, initialize the event log entry
3. Execute Workflow — "Ingest" sub-workflow
4. Execute Workflow — "Transform" sub-workflow
5. Execute Workflow — "Notify" sub-workflow
6. Code node — finalize the event log entry, compose a health-report email
7. Email node — send the report
8. Error Trigger workflow — on any step's failure, log to the same event log and optionally run a compensating sub-workflow

## Code Example

Orchestrator — drives the steps and records outcomes:

```javascript
const fs = require('fs');
const LOG_FILE = '/home/node/workflows/.state/saga-runs.ndjson';

function logStep(runId, step, status, detail) {
  const entry = { runId, step, status, detail, at: new Date().toISOString() };
  fs.mkdirSync(require('path').dirname(LOG_FILE), { recursive: true });
  fs.appendFileSync(LOG_FILE, JSON.stringify(entry) + '\n');
  return entry;
}

const runId = `run-${Date.now()}`;
const steps = ['ingest', 'transform', 'notify'];
const results = [];

for (const step of steps) {
  try {
    // In the real workflow, this calls an Execute Workflow node per step instead.
    const idempotencyKey = `${runId}-${step}`;
    results.push(logStep(runId, step, 'succeeded', { idempotencyKey }));
  } catch (err) {
    results.push(logStep(runId, step, 'failed', { error: err.message }));
    break; // stop the saga; a compensating sub-workflow could run here for completed steps
  }
}

const allSucceeded = results.every((r) => r.status === 'succeeded');

return [{ json: { runId, allSucceeded, results } }];
```

Nightly health report, reading the whole log:

```javascript
const fs = require('fs');
const LOG_FILE = '/home/node/workflows/.state/saga-runs.ndjson';

const lines = fs.existsSync(LOG_FILE)
  ? fs.readFileSync(LOG_FILE, 'utf8').trim().split('\n').filter(Boolean)
  : [];

const entries = lines.map((l) => JSON.parse(l));
const today = new Date().toDateString();
const todaysEntries = entries.filter((e) => new Date(e.at).toDateString() === today);

const byRun = {};
for (const e of todaysEntries) {
  byRun[e.runId] ??= [];
  byRun[e.runId].push(e);
}

const summary = Object.entries(byRun).map(([runId, steps]) => ({
  runId,
  succeeded: steps.every((s) => s.status === 'succeeded'),
  steps: steps.map((s) => `${s.step}:${s.status}`).join(', '),
}));

return [{ json: { html: `<pre>${JSON.stringify(summary, null, 2)}</pre>` } }];
```

## Notes

Each step's `idempotencyKey` means re-running a failed saga from the top is safe — a step that already succeeded can check the log and skip redoing side-effecting work, which is what keeps a saga from double-sending notifications on retry. See also the [Idempotency Key Pattern](03-idempotency-key-pattern.md).
