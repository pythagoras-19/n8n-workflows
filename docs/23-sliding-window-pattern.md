# Sliding Window Pattern

**Workflow:** Sliding-Window Webhook Throttle
**RTC Relevance:** Medium — throttles a live inbound webhook channel

## Why This Pattern

A fixed-window rate limit (e.g. "100 requests per minute, reset on the clock") lets a client send 100 requests at 0:59 and another 100 at 1:01 — 200 in two seconds. A sliding window looks at the actual last N seconds on every request, closing that loophole.

## Workflow Structure

1. Webhook (POST)
2. Code node — sliding-window check (file-backed timestamp log)
3. IF node — within limit continues; over limit responds 429

## Code Example

```javascript
const fs = require('fs');
const LOG_FILE = '/home/node/workflows/.state/request-timestamps.json';
const WINDOW_MS = 60_000;
const LIMIT = 100;

function loadTimestamps() {
  try { return JSON.parse(fs.readFileSync(LOG_FILE, 'utf8')); }
  catch { return []; }
}

function saveTimestamps(timestamps) {
  fs.mkdirSync(require('path').dirname(LOG_FILE), { recursive: true });
  fs.writeFileSync(LOG_FILE, JSON.stringify(timestamps));
}

const now = Date.now();
const windowStart = now - WINDOW_MS;

const timestamps = loadTimestamps().filter((t) => t > windowStart);

const allowed = timestamps.length < LIMIT;
if (allowed) timestamps.push(now);

saveTimestamps(timestamps);

return [{ json: { allowed, count: timestamps.length, limit: LIMIT } }];
```

## Notes

This is O(n) per request on the stored timestamp array — fine for a practice workflow at `LIMIT=100`, but a production sliding-window limiter would use a more compact structure (a fixed-size ring buffer, or a Redis sorted set) to avoid that growing cost.
