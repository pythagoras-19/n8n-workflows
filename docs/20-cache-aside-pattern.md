# Cache-Aside Pattern

**Workflow:** File-Backed TTL Cache
**RTC Relevance:** None

## Why This Pattern

Calling a rate-limited free API every single time wastes your quota when the answer hasn't changed. Check the cache first, only call through on a miss, and remember the result for next time with an expiry so it doesn't go stale forever. "Aside" means the workflow manages the cache explicitly, rather than sitting behind a transparent caching proxy.

## Workflow Structure

1. Code node — check cache (read-through)
2. IF node — cache hit skips the call; cache miss falls through to an HTTP Request
3. Code node — write the result into the cache with a TTL

## Code Example

Check:

```javascript
const fs = require('fs');
const CACHE_FILE = '/home/node/workflows/.state/api-cache.json';
const TTL_MS = 10 * 60 * 1000; // 10 minutes

function loadCache() {
  try { return JSON.parse(fs.readFileSync(CACHE_FILE, 'utf8')); }
  catch { return {}; }
}

const key = $json.cacheKey;
const cache = loadCache();
const entry = cache[key];
const isFresh = entry && (Date.now() - entry.cachedAt) < TTL_MS;

return [{ json: { cacheKey: key, hit: !!isFresh, value: isFresh ? entry.value : null } }];
```

Write back (after a cache miss + HTTP Request):

```javascript
const fs = require('fs');
const CACHE_FILE = '/home/node/workflows/.state/api-cache.json';

let cache = {};
try { cache = JSON.parse(fs.readFileSync(CACHE_FILE, 'utf8')); } catch {}

cache[$json.cacheKey] = { value: $json.value, cachedAt: Date.now() };
fs.mkdirSync(require('path').dirname(CACHE_FILE), { recursive: true });
fs.writeFileSync(CACHE_FILE, JSON.stringify(cache));

return [{ json: { cached: true } }];
```

## Notes

This is the distinction from a generic caching proxy: the application (your workflow) explicitly decides when to read and write the cache, rather than it happening transparently.
