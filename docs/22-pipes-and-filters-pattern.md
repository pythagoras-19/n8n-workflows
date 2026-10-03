# Pipes and Filters Pattern

**Workflow:** Flat-File ETL Warehouse
**RTC Relevance:** None

## Why This Pattern

A long transformation is easier to reason about — and reuse — as a sequence of small, independent steps, each taking input and producing output, than as one giant function doing everything at once.

## Workflow Structure

1. HTTP Request × N — one per source
2. Code node "filter" — normalize shape (the [Adapter pattern](16-adapter-pattern.md) can be reused here)
3. Code node "filter" — deduplicate by id
4. Code node "filter" — append to the NDJSON warehouse file

## Code Example

```javascript
function normalize(record) {
  return { id: record.id ?? record.uuid, fetchedAt: new Date().toISOString(), ...record };
}

function dedupe(records) {
  const byId = new Map(records.map((r) => [r.id, r]));
  return [...byId.values()];
}

function appendToWarehouse(records) {
  const fs = require('fs');
  const WAREHOUSE_FILE = '/home/node/workflows/.state/warehouse.ndjson';
  fs.mkdirSync(require('path').dirname(WAREHOUSE_FILE), { recursive: true });
  const lines = records.map((r) => JSON.stringify(r)).join('\n') + '\n';
  fs.appendFileSync(WAREHOUSE_FILE, lines);
  return records;
}

const pipeline = (input) => appendToWarehouse(dedupe(input.map(normalize)));

const records = items.map((item) => item.json);
const result = pipeline(records);

return result.map((r) => ({ json: r }));
```

## Notes

Each function (`normalize`, `dedupe`, `appendToWarehouse`) could just as easily be its own Code node wired in sequence — splitting them out as named functions here is the same idea in miniature, and keeps each stage independently testable.
