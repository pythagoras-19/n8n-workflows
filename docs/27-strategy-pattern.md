# Strategy Pattern

**Workflow:** Config-Driven Multi-Tenant Workflow
**RTC Relevance:** None

## Why This Pattern

Different tenants often need genuinely different behavior, not just different data — one wants Slack notifications, another wants email, a third wants a different schedule. Strategy picks the right behavior at runtime instead of a sprawling if/else per tenant.

## Workflow Structure

1. Code node — load tenant config, select the strategy
2. Switch node — route to the chosen strategy's branch

## Code Example

```javascript
const tenantConfigs = {
  acme: { notifyVia: 'discord', webhookUrl: 'https://discord.com/api/webhooks/acme/...' },
  globex: { notifyVia: 'email', to: 'ops@globex.example' },
  initech: { notifyVia: 'telegram', chatId: '123456789' },
};

const strategies = {
  discord: (cfg, message) => ({ channel: 'discord', url: cfg.webhookUrl, body: { content: message } }),
  email: (cfg, message) => ({ channel: 'email', to: cfg.to, subject: 'Notification', body: message }),
  telegram: (cfg, message) => ({ channel: 'telegram', chatId: cfg.chatId, text: message }),
};

const tenantId = $json.tenantId;
const config = tenantConfigs[tenantId];
const strategy = strategies[config.notifyVia];

const dispatch = strategy(config, $json.message);

return [{ json: dispatch }];
```

## Notes

The Switch node downstream reads `dispatch.channel` to route to the right delivery node — the Code node's only job is picking *which* strategy applies, not executing it.
