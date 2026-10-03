# Circuit Breaker Pattern

**Workflow:** Circuit Breaker for a Flaky API
**RTC Relevance:** Low — guards a call path, not a persistent channel

## Why This Pattern

Hammering a dependency that's already down wastes time waiting for timeouts and can make its recovery harder. A circuit breaker stops calling it after enough consecutive failures, then periodically tests ("half-open") whether it's safe to resume.

## Workflow Structure

1. Code node — check breaker state before calling
2. HTTP Request — only runs if the breaker allows it
3. Code node — record success/failure, update breaker state

## Code Example

Check before calling:

```javascript
const fs = require('fs');
const STATE_FILE = '/home/node/workflows/.state/circuit-breaker.json';
const COOLDOWN_MS = 30_000;

function loadState() {
  try { return JSON.parse(fs.readFileSync(STATE_FILE, 'utf8')); }
  catch { return { status: 'closed', failures: 0, openedAt: null }; }
}

let state = loadState();

if (state.status === 'open') {
  const elapsed = Date.now() - state.openedAt;
  if (elapsed < COOLDOWN_MS) {
    return [{ json: { allowed: false, status: 'open', retryInMs: COOLDOWN_MS - elapsed } }];
  }
  state.status = 'half-open'; // allow exactly one probe through
}

return [{ json: { allowed: true, status: state.status } }];
```

Record the outcome (after the HTTP Request):

```javascript
const fs = require('fs');
const STATE_FILE = '/home/node/workflows/.state/circuit-breaker.json';
const FAILURE_THRESHOLD = 3;

let state;
try { state = JSON.parse(fs.readFileSync(STATE_FILE, 'utf8')); }
catch { state = { status: 'closed', failures: 0, openedAt: null }; }

const succeeded = $json.callSucceeded; // set upstream based on the HTTP Request's success/error branch

if (succeeded) {
  state = { status: 'closed', failures: 0, openedAt: null };
} else {
  state.failures += 1;
  if (state.status === 'half-open' || state.failures >= FAILURE_THRESHOLD) {
    state.status = 'open';
    state.openedAt = Date.now();
  }
}

fs.writeFileSync(STATE_FILE, JSON.stringify(state));
return [{ json: state }];
```

## Notes

Route the HTTP Request node's error output into the recording Code node too (via "Continue on Fail") — the breaker needs to see failures, not just successes, to ever trip open.
