# Fan-Out/Fan-In Pattern

**Workflow:** Concurrent Paginated Crawler
**RTC Relevance:** None

## Why This Pattern

Fetching many independent pages or items serially is slow. Fanning requests out (in controlled batches) and fanning the results back in finishes the work faster while still respecting a concurrency ceiling the upstream API can handle.

## Workflow Structure

1. Code node — compute the list of page numbers to fetch
2. SplitInBatches — batch size is your concurrency limit
3. HTTP Request — runs once per item in the current batch
4. Code node — merge/dedup accumulated results across all batches

## Code Example

Compute pages:

```javascript
const PAGE_COUNT = 10;
return Array.from({ length: PAGE_COUNT }, (_, i) => ({ json: { page: i + 1 } }));
```

Merge/dedup after the loop:

```javascript
const seen = new Map();

for (const item of items) {
  for (const character of item.json.results ?? []) {
    seen.set(character.id, character);
  }
}

return [...seen.values()].map((character) => ({ json: character }));
```

## Notes

SplitInBatches' batch size *is* your concurrency limit — set it deliberately rather than leaving the default, to respect the upstream API's own rate limit.
