# Competing Consumers Pattern (EIP)

**Workflow:** RabbitMQ Work Queue Dispatcher
**RTC Relevance:** High — multiple live workers race to consume each message

## Why This Pattern

A single consumer processing a queue serially caps throughput at one message at a time. Running several independent consumer workflows against the same queue lets RabbitMQ load-balance work across whichever workers happen to be online — throughput scales by adding workers, not by changing code.

## Workflow Structure

Producer workflow:
1. Code node — build task payloads
2. RabbitMQ node — publish to queue `tasks`

Worker workflow (run two or more copies):
1. RabbitMQ Trigger — consume from queue `tasks`
2. Code node — do the work, implicit ack on success

## Code Example

Producer (before the RabbitMQ node):

```javascript
return items.map((item, i) => ({
  json: {
    taskId: `task-${Date.now()}-${i}`,
    payload: item.json,
  },
}));
```

Worker (after the RabbitMQ Trigger):

```javascript
const { taskId, payload } = $json;

// Simulate work; replace with the real processing step.
const result = {
  taskId,
  processedBy: process.pid,
  processedAt: new Date().toISOString(),
  outputSummary: JSON.stringify(payload).slice(0, 80),
};

return [{ json: result }];
```

## Notes

Set the RabbitMQ Trigger's `prefetchCount` to 1 so each worker only pulls one unacked message at a time — otherwise one fast worker can hog the backlog instead of it being shared evenly across workers.

Requires RabbitMQ running locally — see the "Running RabbitMQ" section in the [repo README](../README.md).
