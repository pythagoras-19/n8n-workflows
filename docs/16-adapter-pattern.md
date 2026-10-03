# Adapter Pattern

**Workflow:** Multi-Source Schema Normalizer
**RTC Relevance:** None

## Why This Pattern

Three APIs rarely agree on field names or shapes. Writing one small adapter function per source — each conforming to the same output shape — means everything downstream only ever deals with one canonical schema.

## Workflow Structure

1. Parallel branches — HTTP Request to a weather API, a quotes API, and the GitHub API
2. Code node per branch (or one shared node after a Merge) — adapt each to the canonical shape

## Code Example

```javascript
const adapters = {
  weather(raw) {
    return {
      source: 'weather',
      summary: `${raw.current.temperature_2m}°C, wind ${raw.current.wind_speed_10m} km/h`,
      raw,
    };
  },
  quote(raw) {
    return {
      source: 'quote',
      summary: `"${raw.content}" — ${raw.author}`,
      raw,
    };
  },
  githubRepo(raw) {
    return {
      source: 'github',
      summary: `${raw.full_name}: ${raw.stargazers_count} stars`,
      raw,
    };
  },
};

return items.map((item) => {
  const adapt = adapters[item.json.sourceType];
  return { json: adapt ? adapt(item.json.data) : { source: 'unknown', raw: item.json } };
});
```

## Notes

Downstream nodes only ever read `.source` and `.summary` — swapping the weather provider later means rewriting one adapter function, not every node that consumes weather data.
