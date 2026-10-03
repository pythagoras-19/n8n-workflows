# Content-Based Router Pattern (EIP)

**Workflow:** Content-Based Router
**RTC Relevance:** Medium — live routing of inbound messages by content

## Why This Pattern

A single incoming stream of messages often needs to go to different destinations depending on what's inside them (type, priority, region). Doing that routing at the broker — via an exchange and routing key — keeps consumers simple: each only subscribes to what it actually cares about.

## Workflow Structure

1. Producer — Code node computes a routing key from message content, then a RabbitMQ node publishes to a `topic` exchange with that key
2. Consumer A — RabbitMQ Trigger bound with pattern `orders.eu.*`
3. Consumer B — RabbitMQ Trigger bound with pattern `orders.*.high`

## Code Example

```javascript
const order = $json;

const region = order.country === 'US' ? 'us' : 'eu';
const priority = order.total > 500 ? 'high' : 'normal';

return [{
  json: order,
  routingKey: `orders.${region}.${priority}`,
}];
```

## Notes

The routing decision is pure JS and fully testable on its own — the broker just mechanically matches routing keys against binding patterns (`orders.eu.*` matches any EU order regardless of priority).

Requires RabbitMQ running locally — see the "Running RabbitMQ" section in the [repo README](../README.md).
