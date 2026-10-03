# Distributed Lock (Mutex) Pattern

**Workflow:** Stale-Lock-Aware Scheduler Guard
**RTC Relevance:** None

## Why This Pattern

A Schedule Trigger can overlap if a run takes longer than the trigger interval. A file-based lock prevents two instances of the same workflow from processing the same data concurrently — and detecting a *stale* lock means a crashed run doesn't block every future run forever.

## Workflow Structure

1. Schedule Trigger
2. Code node — acquire lock, or exit if locked and not stale
3. ... the actual work ...
4. Code node — release lock (ideally on an always-run / error-handled path)

## Code Example

Acquire:

```javascript
const fs = require('fs');
const LOCK_FILE = '/home/node/workflows/.state/job.lock';
const STALE_AFTER_MS = 5 * 60 * 1000; // 5 minutes

function acquireLock() {
  if (fs.existsSync(LOCK_FILE)) {
    const heldSince = Number(fs.readFileSync(LOCK_FILE, 'utf8'));
    const age = Date.now() - heldSince;
    if (age < STALE_AFTER_MS) {
      return { acquired: false, reason: `locked for ${Math.round(age / 1000)}s` };
    }
    // stale lock from a crashed run — reclaim it
  }
  fs.mkdirSync(require('path').dirname(LOCK_FILE), { recursive: true });
  fs.writeFileSync(LOCK_FILE, String(Date.now()));
  return { acquired: true };
}

return [{ json: acquireLock() }];
```

Release:

```javascript
const fs = require('fs');
const LOCK_FILE = '/home/node/workflows/.state/job.lock';
if (fs.existsSync(LOCK_FILE)) fs.unlinkSync(LOCK_FILE);
return [{ json: { released: true } }];
```

## Notes

Put the release node on an "always output data" / error-workflow path so a crash mid-run doesn't leave the lock held past `STALE_AFTER_MS`.
