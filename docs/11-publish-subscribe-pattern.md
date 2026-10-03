# Publish-Subscribe / Broker Pattern

**Workflow:** Multi-Subscriber Event Broker
**RTC Relevance:** High — one broadcast reaches many live subscribers at once

## Why This Pattern

The broker fully decouples producers from consumers — a publisher doesn't know or care how many subscribers exist, and you can add a new subscriber (another workflow) without touching the publisher at all.

## Workflow Structure

1. One publisher workflow → MQTT node (or a RabbitMQ node publishing to a `fanout` exchange)
2. Subscriber A: MQTT/RabbitMQ Trigger → Discord notification
3. Subscriber B: MQTT/RabbitMQ Trigger → append to an archive file
4. Subscriber C: MQTT/RabbitMQ Trigger → increment a metrics counter

## Code Example

Subscriber B — archive:

```javascript
const fs = require('fs');
const ARCHIVE_FILE = '/home/node/workflows/.state/event-archive.ndjson';

fs.mkdirSync(require('path').dirname(ARCHIVE_FILE), { recursive: true });
fs.appendFileSync(ARCHIVE_FILE, JSON.stringify($json) + '\n');

return [{ json: { archived: true } }];
```

Subscriber C — metrics counter:

```javascript
const fs = require('fs');
const COUNTER_FILE = '/home/node/workflows/.state/event-count.json';

let count = 0;
try { count = JSON.parse(fs.readFileSync(COUNTER_FILE, 'utf8')).count; } catch {}

count += 1;
fs.mkdirSync(require('path').dirname(COUNTER_FILE), { recursive: true });
fs.writeFileSync(COUNTER_FILE, JSON.stringify({ count }));

return [{ json: { count } }];
```

## Notes

With a RabbitMQ fanout exchange, each subscriber workflow declares its own queue bound to the exchange — every bound queue gets a copy of every message. That's the key difference from [Competing Consumers](09-competing-consumers-pattern.md), where workers share one queue and split the work instead of each getting a copy.
