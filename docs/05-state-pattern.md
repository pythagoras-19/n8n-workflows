# State Pattern (State Machine)

**Workflow:** Mini Order State Machine
**RTC Relevance:** Medium — driven by live webhook calls

## Why This Pattern

Order/ticket/job lifecycles have legal and illegal transitions — you can't go from "delivered" back to "new". Encoding the transition table explicitly stops bad or out-of-order events from corrupting downstream systems.

## Workflow Structure

1. Webhook (POST `{ orderId, event }`)
2. Code node — validate transition against the state machine, persist
3. Respond to Webhook — 200 on a valid transition, 409 on an invalid one

## Code Example

```javascript
const fs = require('fs');
const STATE_FILE = '/home/node/workflows/.state/orders.json';

const TRANSITIONS = {
  new: ['processing', 'cancelled'],
  processing: ['shipped', 'cancelled'],
  shipped: ['delivered'],
  delivered: [],
  cancelled: [],
};

function loadOrders() {
  try { return JSON.parse(fs.readFileSync(STATE_FILE, 'utf8')); }
  catch { return {}; }
}

function saveOrders(orders) {
  fs.mkdirSync(require('path').dirname(STATE_FILE), { recursive: true });
  fs.writeFileSync(STATE_FILE, JSON.stringify(orders, null, 2));
}

const { orderId, event } = $json.body ?? $json;
const orders = loadOrders();
const currentState = orders[orderId] ?? 'new';
const allowed = TRANSITIONS[currentState] ?? [];

if (!allowed.includes(event)) {
  return [{ json: { ok: false, orderId, from: currentState, attempted: event, allowed } }];
}

orders[orderId] = event;
saveOrders(orders);

return [{ json: { ok: true, orderId, from: currentState, to: event } }];
```

## Notes

Wire the `ok` field into an IF node to choose a 200 vs. 409 response in the "Respond to Webhook" node.
