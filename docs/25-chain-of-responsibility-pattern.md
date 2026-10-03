# Chain of Responsibility Pattern

**Workflow:** Simulated Build Pipeline
**RTC Relevance:** Medium — triggered by a live webhook event

## Why This Pattern

A CI pipeline is naturally a sequence of handlers — lint, then test, then deploy — where any stage failing should stop the chain. Each stage only needs to know "do my job, then pass along, or stop."

## Workflow Structure

1. Webhook — receives the push payload
2. Code node chain — lint → test → deploy, each able to halt the chain
3. Code node — aggregate stage results into one status report
4. Notification node

## Code Example

```javascript
async function lintStage(ctx) {
  const hasObviousIssue = ctx.payload.commits?.some((c) => c.message.includes('WIP'));
  return { stage: 'lint', passed: !hasObviousIssue, detail: hasObviousIssue ? 'WIP commit found' : 'ok' };
}

async function testStage(ctx) {
  // Stand in for a real test run.
  const passed = Math.random() > 0.1;
  return { stage: 'test', passed, detail: passed ? 'all tests passed' : 'simulated test failure' };
}

async function deployStage(ctx) {
  return { stage: 'deploy', passed: true, detail: `deployed ${ctx.payload.ref}` };
}

const chain = [lintStage, testStage, deployStage];
const ctx = { payload: $json.body ?? $json };
const results = [];

for (const stage of chain) {
  const result = await stage(ctx);
  results.push(result);
  if (!result.passed) break; // halt the chain
}

const allPassed = results.length === chain.length && results.every((r) => r.passed);

return [{ json: { allPassed, results } }];
```

## Notes

Each stage function only takes `ctx` and returns a pass/fail — adding a new stage (e.g. a security scan) means adding one function to the `chain` array, not restructuring the others.
