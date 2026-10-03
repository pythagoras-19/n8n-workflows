# Retry Pattern (Backoff + Jitter)

**Workflow:** Retry-with-Backoff HTTP Client
**RTC Relevance:** None

## Why This Pattern

Transient failures (network blips, momentary rate limits) shouldn't fail an entire workflow immediately. Retrying with exponential backoff and jitter gives a flaky dependency time to recover, and the jitter prevents every failed caller from retrying at the exact same instant (a "thundering herd").

## Workflow Structure

1. Manual or Schedule Trigger
2. Code node — retry loop around an HTTP call
3. Error Trigger workflow — captures and logs calls that exhaust all retries

## Code Example

```javascript
const MAX_ATTEMPTS = 5;
const BASE_DELAY_MS = 200;

function sleep(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

function backoffWithJitter(attempt) {
  const exp = BASE_DELAY_MS * 2 ** attempt;
  const jitter = Math.random() * exp * 0.5;
  return Math.min(exp + jitter, 10_000);
}

async function callWithRetry(url) {
  let lastError;
  for (let attempt = 0; attempt < MAX_ATTEMPTS; attempt++) {
    try {
      return await this.helpers.httpRequest({ method: 'GET', url, json: true });
    } catch (err) {
      lastError = err;
      if (attempt === MAX_ATTEMPTS - 1) break;
      await sleep(backoffWithJitter(attempt));
    }
  }
  throw new Error(`Failed after ${MAX_ATTEMPTS} attempts: ${lastError.message}`);
}

const url = $json.url || 'https://jsonplaceholder.typicode.com/todos/1';
const result = await callWithRetry.call(this, url);
return [{ json: result }];
```

## Notes

`this.helpers.httpRequest` is available inside an n8n Code node. An uncaught throw propagates to the workflow's Error Trigger (if one is attached) rather than silently failing.
