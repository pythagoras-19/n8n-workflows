# Token Bucket Pattern

**Workflow:** Token-Bucket Rate Limiter
**RTC Relevance:** Low — throttles outbound calls, not a live channel itself

## Why This Pattern

Smooths bursts of outbound calls to respect a rate limit (e.g. N calls/second) without dropping work outright — requests over the limit wait for tokens to refill on a later run instead of failing.

## Workflow Structure

1. Schedule or Webhook trigger produces work items
2. Code node — token bucket (reads/writes bucket state to a local JSON file)
3. HTTP Request — only runs for items that acquired a token

## Code Example

```javascript
const fs = require('fs');
const BUCKET_FILE = '/home/node/workflows/.state/token-bucket.json';
const CAPACITY = 5;
const REFILL_RATE = 1; // tokens per second

function loadBucket() {
  try {
    return JSON.parse(fs.readFileSync(BUCKET_FILE, 'utf8'));
  } catch {
    return { tokens: CAPACITY, lastRefill: Date.now() };
  }
}

function saveBucket(bucket) {
  fs.mkdirSync(require('path').dirname(BUCKET_FILE), { recursive: true });
  fs.writeFileSync(BUCKET_FILE, JSON.stringify(bucket));
}

function refill(bucket) {
  const now = Date.now();
  const elapsedSec = (now - bucket.lastRefill) / 1000;
  bucket.tokens = Math.min(CAPACITY, bucket.tokens + elapsedSec * REFILL_RATE);
  bucket.lastRefill = now;
  return bucket;
}

const results = [];
let bucket = refill(loadBucket());

for (const item of items) {
  if (bucket.tokens >= 1) {
    bucket.tokens -= 1;
    results.push({ json: { ...item.json, allowed: true } });
  } else {
    results.push({ json: { ...item.json, allowed: false, reason: 'rate-limited, requeue next run' } });
  }
}

saveBucket(bucket);
return results;
```

## Notes

`items` is the full input array in an n8n Code node running in "Run Once for All Items" mode. Requeue `allowed: false` items (e.g. write them to a pending file) so they get picked up on a later run instead of being dropped.
