# Decorator Pattern

**Workflow:** HMAC Webhook Verifier & Router
**RTC Relevance:** Medium — wraps a live inbound webhook

## Why This Pattern

Signature verification is a cross-cutting concern that has nothing to do with what a webhook handler actually does. Wrapping the handler with a verification step — rather than inlining checks into every handler — keeps the two responsibilities separate and reusable.

## Workflow Structure

1. Webhook (POST)
2. Code node — HMAC verification "decorator", rejects before the real handler ever runs
3. Switch node — route verified payloads by `type`
4. Real handler nodes per type

## Code Example

```javascript
const crypto = require('crypto');

const SECRET = 'replace-with-a-real-shared-secret'; // store in n8n credentials in practice, not inline

const rawBody = JSON.stringify($json.body);
const signatureHeader = $json.headers['x-signature'];

const expected = crypto.createHmac('sha256', SECRET).update(rawBody).digest('hex');
const verified = signatureHeader === expected;

if (!verified) {
  return [{ json: { verified: false, error: 'invalid signature' } }];
}

return [{ json: { verified: true, ...$json.body } }];
```

## Notes

Keep the decorator generic — it only knows about signatures, not payload shape. That's what makes it reusable across every future webhook you add, not just this one.
