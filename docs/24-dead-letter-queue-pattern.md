# Dead Letter Queue Pattern (EIP)

**Workflow:** RabbitMQ Dead-Letter Exchange Handler
**RTC Relevance:** High — broker-native dead-lettering of live messages

## Why This Pattern

If a consumer can't process a message (bad data, downstream outage) and just requeues it forever, it can block the whole queue. Routing permanently-failed messages to a separate dead-letter queue lets the main queue keep flowing while failures get handled, or inspected, separately.

## Workflow Structure

1. Main queue declared with `x-dead-letter-exchange` pointing at a DLX, plus a message TTL or max-retry count
2. Consumer workflow — RabbitMQ Trigger on the main queue; on failure, nack/reject without requeue (RabbitMQ then auto-routes it to the DLX)
3. DLQ workflow — RabbitMQ Trigger on the dead-letter queue; Code node retries with backoff, tracking attempt count from message headers

## Code Example

Consumer — decide ack vs. reject:

```javascript
try {
  // ... real processing here ...
  if (!$json.payload || !$json.payload.id) {
    throw new Error('missing required field: id');
  }
  return [{ json: { ...$json, processed: true } }];
} catch (err) {
  // This shape lets a downstream IF node route to a "reject without
  // requeue" RabbitMQ node, which RabbitMQ then dead-letters automatically
  // because of the queue's DLX config.
  return [{ json: { ...$json, processed: false, error: err.message } }];
}
```

DLQ workflow — bounded retry with backoff:

```javascript
const MAX_ATTEMPTS = 5;

const attempts = ($json.headers?.['x-death']?.[0]?.count ?? 0) + 1;

if (attempts > MAX_ATTEMPTS) {
  return [{ json: { ...$json, giveUp: true, attempts } }];
}

const delayMs = Math.min(1000 * 2 ** attempts, 60_000);

return [{ json: { ...$json, giveUp: false, attempts, retryAfterMs: delayMs } }];
```

## Notes

RabbitMQ automatically stamps an `x-death` header with the rejection count each time a message is dead-lettered, so the DLQ handler doesn't need its own file-based counter — the broker already tracks it.

Requires RabbitMQ running locally — see the "Running RabbitMQ" section in the [repo README](../README.md).
