# Event Sourcing Pattern

**Workflow:** Event-Sourced Todo List
**RTC Relevance:** Medium — webhook appends events live

## Why This Pattern

Instead of storing only the current state, you store every event that ever happened. Current state becomes a derived view, which gives you a full audit trail and the ability to replay or rebuild state at any point in time.

## Workflow Structure

1. Webhook (POST `{ todoId, type, payload }`) — appends an event
2. Code node — append to an append-only event log file
3. A separate Manual Trigger + Code node replays the full log into current state on demand

## Code Example

Append a new event:

```javascript
const fs = require('fs');
const EVENT_LOG = '/home/node/workflows/.state/todo-events.ndjson';

const event = {
  ...($json.body ?? $json),
  timestamp: new Date().toISOString(),
};

fs.mkdirSync(require('path').dirname(EVENT_LOG), { recursive: true });
fs.appendFileSync(EVENT_LOG, JSON.stringify(event) + '\n');

return [{ json: event }];
```

Replay the log into current state (separate node/workflow):

```javascript
const fs = require('fs');
const EVENT_LOG = '/home/node/workflows/.state/todo-events.ndjson';

const lines = fs.existsSync(EVENT_LOG)
  ? fs.readFileSync(EVENT_LOG, 'utf8').trim().split('\n').filter(Boolean)
  : [];

const todos = {};

for (const line of lines) {
  const event = JSON.parse(line);
  switch (event.type) {
    case 'created':
      todos[event.todoId] = { ...event.payload, done: false };
      break;
    case 'updated':
      todos[event.todoId] = { ...todos[event.todoId], ...event.payload };
      break;
    case 'completed':
      if (todos[event.todoId]) todos[event.todoId].done = true;
      break;
    case 'deleted':
      delete todos[event.todoId];
      break;
  }
}

return Object.entries(todos).map(([todoId, todo]) => ({ json: { todoId, ...todo } }));
```

## Notes

The log is append-only and never mutated — "undo" means appending a compensating event, not editing history.
