# Watcher Pattern (Change Detection)

**Workflow:** Self-Healing Link Checker
**RTC Relevance:** None

## Why This Pattern

Nobody wants a notification every time a scheduled check runs — only when something actually changed. A watcher keeps the last-known state and diffs against it, so alerts only fire on real transitions.

## Workflow Structure

1. Schedule Trigger
2. HTTP Request over a list of URLs (looped via SplitInBatches)
3. Code node — evaluate status, diff against stored state
4. IF node — only continue to notification if something changed

## Code Example

```javascript
const fs = require('fs');
const STATE_FILE = '/home/node/workflows/.state/link-status.json';

function loadState() {
  try { return JSON.parse(fs.readFileSync(STATE_FILE, 'utf8')); }
  catch { return {}; }
}

function saveState(state) {
  fs.mkdirSync(require('path').dirname(STATE_FILE), { recursive: true });
  fs.writeFileSync(STATE_FILE, JSON.stringify(state, null, 2));
}

const previous = loadState();
const current = {};
const changes = [];

for (const item of items) {
  const { url, statusCode } = item.json;
  const isBroken = !statusCode || statusCode >= 400;
  current[url] = isBroken;

  if (previous[url] !== isBroken) {
    changes.push({ url, wasBroken: !!previous[url], isBroken });
  }
}

saveState(current);

return [{ json: { changed: changes.length > 0, changes } }];
```

## Notes

Pair this with a downstream IF node checking `{{$json.changed}}` so notifications only fire on real state transitions, not on every poll — even when nothing is broken.
