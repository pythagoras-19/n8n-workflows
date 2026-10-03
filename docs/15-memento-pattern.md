# Memento Pattern (Snapshot/Diff)

**Workflow:** Custom JSON Diff & Patch Engine
**RTC Relevance:** None

## Why This Pattern

Sending a whole object downstream every time it changes wastes bandwidth and hides what actually changed. Computing a minimal diff — and being able to apply it as a patch — is how most real-time collaboration and sync systems keep clients up to date cheaply. "Before" acts as the saved memento you can diff against or roll back to.

## Workflow Structure

1. Two inputs: previous snapshot, current snapshot (from a file, or two API calls)
2. Code node — deep diff → patch

## Code Example

```javascript
function diff(before, after, path = '') {
  const patches = [];
  const keys = new Set([...Object.keys(before ?? {}), ...Object.keys(after ?? {})]);

  for (const key of keys) {
    const fullPath = path ? `${path}.${key}` : key;
    const beforeVal = before?.[key];
    const afterVal = after?.[key];

    const bothObjects =
      typeof beforeVal === 'object' && beforeVal !== null &&
      typeof afterVal === 'object' && afterVal !== null &&
      !Array.isArray(beforeVal) && !Array.isArray(afterVal);

    if (bothObjects) {
      patches.push(...diff(beforeVal, afterVal, fullPath));
    } else if (JSON.stringify(beforeVal) !== JSON.stringify(afterVal)) {
      patches.push({ path: fullPath, from: beforeVal, to: afterVal });
    }
  }

  return patches;
}

const before = $json.before;
const after = $json.after;

return [{ json: { patches: diff(before, after) } }];
```

## Notes

The object being diffed doesn't need to know anything about how it's being tracked — the memento (the "before" snapshot) lives entirely outside it, which is the core idea of the pattern.
