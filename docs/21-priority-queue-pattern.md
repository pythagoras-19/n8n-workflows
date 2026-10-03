# Priority Queue Pattern

**Workflow:** RabbitMQ Priority Queue Processor
**RTC Relevance:** Medium — live, broker-ordered delivery

## Why This Pattern

Not all work is equally urgent. Rather than hand-rolling a heap, RabbitMQ can be told to prioritize a queue natively (`x-max-priority`), so higher-priority messages are delivered to consumers first even if they arrived later.

## Workflow Structure

1. Code node — assign a priority (0–9) to each task from its metadata
2. RabbitMQ node — publish with the `priority` property set, to a queue declared with `x-max-priority: 9`
3. RabbitMQ Trigger — the consumer simply processes messages in the order RabbitMQ delivers them

## Code Example

```javascript
function priorityFor(task) {
  if (task.vip) return 9;
  if (task.dueInMinutes <= 5) return 7;
  if (task.dueInMinutes <= 60) return 4;
  return 1;
}

return items.map((item) => ({
  json: item.json,
  priority: priorityFor(item.json),
}));
```

## Notes

Queue priority only has an effect when messages are actually backed up waiting for a consumer — if consumers are keeping up with the queue, nothing waits long enough for priority to matter, so this pattern shows itself best under load.

Requires RabbitMQ running locally — see the "Running RabbitMQ" section in the [repo README](../README.md).
