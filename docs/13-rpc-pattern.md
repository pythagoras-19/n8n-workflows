# RPC Pattern (Request-Reply, EIP)

**Workflow:** RabbitMQ RPC Client/Server
**RTC Relevance:** High — synchronous-style live request/response over a broker

## Why This Pattern

Messaging is normally fire-and-forget, but sometimes you need a synchronous-feeling answer — call a remote function, wait for its result — without a direct HTTP connection between the two sides. RabbitMQ's reply-to + correlation-id convention is the standard way to layer RPC over a message queue.

## Workflow Structure

Client workflow:
1. Code node — generate a `correlationId`, build the request
2. RabbitMQ node — publish to `rpc_requests` with `replyTo: 'rpc_replies'` and the correlation id
3. RabbitMQ Trigger — consume `rpc_replies`, filter by matching `correlationId`

Server workflow:
1. RabbitMQ Trigger — consume `rpc_requests`
2. Code node — do the work
3. RabbitMQ node — publish the response to the queue named in `replyTo`, with the same `correlationId`

## Code Example

Client — build the request:

```javascript
const crypto = require('crypto');

const correlationId = crypto.randomUUID();
const request = {
  correlationId,
  replyTo: 'rpc_replies',
  method: 'square',
  params: { n: $json.n },
};

return [{ json: request }];
```

Server — handle the request:

```javascript
const { method, params, correlationId, replyTo } = $json;

const methods = {
  square: ({ n }) => n * n,
  reverse: ({ text }) => [...text].reverse().join(''),
};

const handler = methods[method];
const result = handler ? handler(params) : { error: `Unknown method: ${method}` };

return [{ json: { correlationId, replyTo, result } }];
```

Client — match the reply:

```javascript
// $json is one message off the rpc_replies queue; $('Build Request').item.json
// holds the correlationId this branch is actually waiting on.
const expected = $('Build Request').item.json.correlationId;

if ($json.correlationId !== expected) {
  return []; // not our reply — ignore it
}

return [{ json: { result: $json.result } }];
```

## Notes

This is "RPC" in the enterprise-integration sense (Request-Reply over messaging), not literal gRPC/JSON-RPC — the correlation id is what lets one shared reply queue correctly serve many concurrent in-flight requests.

Requires RabbitMQ running locally — see the "Running RabbitMQ" section in the [repo README](../README.md).
