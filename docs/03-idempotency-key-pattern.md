# Idempotency Key Pattern

**Workflow:** Idempotent Webhook Receiver
**RTC Relevance:** Medium — consumes live inbound webhook events

## Why This Pattern

Webhook senders retry on timeouts and network errors, which can deliver the same event more than once. Processing it twice (double-charging, double-sending an email) is usually worse than occasionally doing nothing — so duplicate deliveries need to be detected and skipped.

## Workflow Structure

1. Webhook (POST)
2. Code node — compute/read the idempotency key, check and record it
3. IF node — branch on duplicate vs. new
4. Respond to Webhook — 200 either way (the sender shouldn't retry a duplicate either)

## Code Example

```javascript
const crypto = require('crypto');
const fs = require('fs');
const PROCESSED_FILE = '/home/node/workflows/.state/processed-ids.json';

function loadProcessed() {
  try {
    return new Set(JSON.parse(fs.readFileSync(PROCESSED_FILE, 'utf8')));
  } catch {
    return new Set();
  }
}

function saveProcessed(set) {
  fs.mkdirSync(require('path').dirname(PROCESSED_FILE), { recursive: true });
  fs.writeFileSync(PROCESSED_FILE, JSON.stringify([...set]));
}

const body = $json.body ?? $json;
const idempotencyKey = $json.headers?.['idempotency-key']
  ?? crypto.createHash('sha256').update(JSON.stringify(body)).digest('hex');

const processed = loadProcessed();
const isDuplicate = processed.has(idempotencyKey);

if (!isDuplicate) {
  processed.add(idempotencyKey);
  saveProcessed(processed);
}

return [{ json: { idempotencyKey, isDuplicate, body } }];
```

## Notes

Prefer a real `Idempotency-Key` header from the sender when one is available — falling back to a content hash only catches byte-identical duplicate payloads, not logically-duplicate ones with a different timestamp field.
