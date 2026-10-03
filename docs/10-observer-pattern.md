# Observer Pattern (MQTT Pub/Sub)

**Workflow:** MQTT State-Change Notifier
**RTC Relevance:** High — true real-time pub/sub push via MQTT

## Why This Pattern

Polling wastes requests and adds latency between a change happening and you finding out. A subscriber that's notified the instant a publisher emits a message is the textbook Observer pattern, implemented here over MQTT instead of in-process listeners.

## Workflow Structure

Publisher workflow:
1. Schedule/Webhook trigger
2. Code node — detect/format the state change
3. MQTT node — publish to a topic (e.g. `home/n8n/state`)

Subscriber workflow:
1. MQTT Trigger — subscribe to the same topic
2. Code node — react to the message

## Code Example

Publisher (before the MQTT node):

```javascript
return [{
  json: {
    topic: 'home/n8n/state',
    message: JSON.stringify({
      event: 'state-changed',
      value: $json.value,
      at: new Date().toISOString(),
    }),
  },
}];
```

Subscriber (after the MQTT Trigger):

```javascript
const payload = JSON.parse($json.message);

if (payload.event !== 'state-changed') {
  return []; // ignore messages this observer doesn't care about
}

return [{ json: { ...payload, receivedAt: new Date().toISOString() } }];
```

## Notes

`test.mosquitto.org` is a free public broker with no auth, good for quick experiments. Switch to a free-tier HiveMQ Cloud cluster (TLS + credentials) for anything beyond a toy.
