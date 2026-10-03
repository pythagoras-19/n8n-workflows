# n8n Workflows

Self-hosted n8n, run manually via `npx`, with workflow definitions version-controlled as JSON.

## Running n8n

```bash
npx n8n start
```

n8n will be available at http://localhost:5678. Its data (credentials, execution history, settings) lives in `~/.n8n` by default and is not stored in this repo.

## Exporting / importing workflows

Workflow JSON lives in `workflows/`, so it can be committed to git.

Export all workflows:

```bash
npx n8n export:workflow --all --output=workflows/ --separate
```

Import all workflows:

```bash
npx n8n import:workflow --input=workflows/ --separate
```

Note: credentials are not included in workflow exports and should not be committed. Re-create credentials manually in the n8n UI on a new instance.

## Practice Agenda — 25 Advanced Workflows

A docket of advanced workflows to build, aimed at exercising both n8n orchestration skills and harder JavaScript (via n8n's Code node) — state machines, custom algorithms, error recovery, and multi-step pipelines rather than single-node demos. Everything here uses either keyless free APIs or free tiers of accounts (GitHub, Google, Telegram, Discord, Gmail) — nothing requires a paid plan or paid API key.

1. **Retry-with-Backoff HTTP Client** — Code node implements a manual retry loop with exponential backoff and jitter against a public endpoint; an Error Trigger workflow captures and logs failures that exhaust retries. *Skills: error-handling nodes, backoff algorithms.*
2. **Token-Bucket Rate Limiter** — Code node implements a token-bucket algorithm (persisted to a local file) to throttle outbound calls to a free API; items over the limit are queued and drained on later runs. *Skills: rate-limiting algorithms, file-backed state.*
3. **Idempotent Webhook Receiver** — Webhook computes a content hash per request, checks it against a "processed" file, and skips duplicate deliveries while still responding 200. *Skills: idempotency keys, hashing, webhook semantics.*
4. **Self-Healing Link Checker** — Loop an HTTP Request over a list of URLs; Code node flags broken links (4xx/5xx/timeout) and only notifies when the broken-link set changed since the last run (diff against stored state). *Skills: looping, status handling, diffing, conditional notification.*
5. **Mini Order State Machine** — Model an order-status progression (new → processing → shipped → delivered) via repeated webhook calls; a Code node validates legal transitions, rejects illegal jumps, and persists current state to a file. *Skills: state-machine logic, validation.*
6. **Event-Sourced Todo List** — Webhook appends create/update/complete events to an append-only log file; a separate Code node replays the full event log to reconstruct current state on demand. *Skills: event sourcing, log replay.*
7. **Stale-Lock-Aware Scheduler Guard** — Schedule Trigger acquires a file-based lock before running and releases it after, with logic to detect and clear a stale lock left by a crashed previous run. *Skills: distributed-lock patterns, crash recovery.*
8. **Concurrent Paginated Crawler** — Crawl a paginated public API (e.g. Rick and Morty API) using SplitInBatches to fetch several pages per batch, respecting a self-imposed concurrency limit, then merge into one deduped result set. *Skills: pagination, controlled concurrency, Merge node.*
9. **Polling-to-Webhook Bridge** — Turn a polling-only public API into near-real-time notifications by tracking a cursor/ETag/last-seen-id in a file and only emitting events for genuinely new items. *Skills: cursor tracking, change detection.*
10. **Custom JSON Diff & Patch Engine** — Code node implements its own deep-diff algorithm (no library) comparing two JSON snapshots, and emits a minimal patch describing only the changed fields. *Skills: recursive algorithms, deep object comparison.*
11. **Multi-Source Schema Normalizer** — Pull from 2–3 free APIs with different shapes (weather, quotes, repo stats) and write a small "adapter registry" in a Code node that maps each source into one canonical schema before merging. *Skills: adapter pattern, schema normalization.*
12. **Circuit Breaker for a Flaky API** — Code node tracks consecutive failures against a public endpoint, "trips" the breaker after a threshold (persisted to file), short-circuits calls while open, and probes with a half-open retry after a cooldown. *Skills: circuit-breaker pattern, timed state transitions.*
13. **Recursive Tree Crawler** — An Execute Workflow node calls itself to walk a nested/tree-shaped structure (e.g. a GitHub repo's directory tree via the contents API) until it hits leaf nodes, aggregating results back up. *Skills: recursion via sub-workflows, base-case handling.*
14. **HMAC Webhook Verifier & Router** — Webhook receives a signed payload; a Code node manually computes an HMAC signature (Node's `crypto` module) to verify authenticity, then routes by payload type with a Switch node. *Skills: cryptographic verification, routing.*
15. **File-Backed TTL Cache** — Code node implements a cache layer with expiration (TTL) in a local JSON file to avoid redundant calls to a rate-limited free API, including cache invalidation logic. *Skills: caching strategy, expiration logic.*
16. **Priority Queue Task Runner** — Maintain a JSON-backed priority queue (implement a binary heap or sorted insert yourself in JS); each scheduled run pops and processes the highest-priority item. *Skills: heap/priority-queue implementation.*
17. **Flat-File ETL Warehouse** — Pull from multiple free APIs on a schedule, normalize and append records to a growing local NDJSON "warehouse" file, handling schema versioning in JS as fields are added over time. *Skills: ETL pipeline design, schema evolution.*
18. **Sliding-Window Webhook Throttle** — Webhook implements sliding-window rate limiting in a Code node (backed by a file-based timestamp log) and rejects requests over the limit with a 429-style response. *Skills: sliding-window algorithms, custom responses.*
19. **Dead-Letter Queue with Backoff** — Failed items from a processing workflow get routed to a DLQ file; a separate scheduled workflow retries them with increasing backoff and gives up after a max-attempts threshold, logging final failures to a report. *Skills: DLQ pattern, bounded retry.*
20. **Simulated Build Pipeline** — A webhook receives a GitHub-style push payload, parses commit/branch info, and runs a sequence of "stage" steps (lint/test/deploy stand-ins) with pass/fail aggregated into a single status report and notification. *Skills: multi-stage orchestration, status aggregation.*
21. **Custom Template Engine** — Code node implements a small mustache-style template renderer from scratch (variable substitution, loops, conditionals) used to generate HTML emails/reports from data. *Skills: parser/renderer design.*
22. **Config-Driven Multi-Tenant Workflow** — A single workflow reads a non-secret "tenant config" JSON file and branches its behavior per tenant (different formatting, destinations, schedules) using a Switch node driven by JS-computed routing. *Skills: config-driven design, dynamic branching.*
23. **Graph Traversal Workflow** — Model a small graph (e.g. a dependency list or org chart) as JSON and implement BFS/DFS in a Code node to compute shortest path or detect cycles. *Skills: graph algorithms.*
24. **Self-Monitoring Health Check** — A secondary scheduled workflow queries n8n's own local REST API for recent execution results, applies custom failure-pattern detection logic in JS, and alerts only on meaningful patterns (e.g. 3 failures in a row), not every blip. *Skills: self-monitoring, pattern detection, alert noise reduction.*
25. **Capstone: Automation Hub Orchestrator** — A central workflow calls several sub-workflows (via Execute Workflow) for ingestion, transformation, and notification, coordinated through a shared JSON event log with idempotency keys and backoff retry, finishing with a nightly health-report email summarizing every run. *Skills: orchestration, idempotency, retry, reporting — everything above, combined.*
