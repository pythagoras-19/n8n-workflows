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

## Running RabbitMQ (optional, for the EIP workflows)

A handful of workflows below exercise RabbitMQ and classic Enterprise Integration Patterns. Run it locally via the official free Docker image — no cloud account needed:

```bash
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```

The management UI is at http://localhost:15672 (default login `guest`/`guest`), and n8n connects to it over AMQP on `localhost:5672` using n8n's built-in RabbitMQ Trigger and RabbitMQ nodes.

## Practice Agenda — 30 Design-Pattern Workflows

A docket of advanced workflows to build, each centered on one named design pattern — resiliency patterns, rate-limiting algorithms, classic GoF patterns, distributed-systems patterns, real-time communication patterns, and classic Enterprise Integration Patterns (Hohpe & Woolf) — implemented concretely in n8n with real JavaScript (via the Code node), not just described. Real-time here means genuine event-driven push: n8n has no native WebSocket node, but it does have native MQTT and RabbitMQ nodes (true pub/sub and message queueing) and Telegram/Discord bot triggers (live two-way messaging), all free. Everything here uses either keyless free APIs, a free public MQTT broker, a self-hosted RabbitMQ via Docker, or free tiers of accounts (GitHub, Google, Telegram, Discord, Gmail) — nothing requires a paid plan or paid API key.

Each title links to a dedicated doc in [`docs/`](docs/) with the node layout and a working Code-node example.

| # | Title | Pattern | RTC Relevance | Description |
|---|-------|---------|----------------|--------------|
| 1 | [Retry-with-Backoff HTTP Client](docs/01-retry-pattern.md) | Retry Pattern (Backoff + Jitter) | None | Code node implements a manual retry loop with exponential backoff and jitter against a public endpoint; an Error Trigger workflow captures and logs failures that exhaust retries. |
| 2 | [Token-Bucket Rate Limiter](docs/02-token-bucket-pattern.md) | Token Bucket Pattern | Low — throttles outbound calls, not a live channel itself | Code node implements a token-bucket algorithm (persisted to a local file) to throttle outbound calls to a free API; items over the limit are queued and drained on later runs. |
| 3 | [Idempotent Webhook Receiver](docs/03-idempotency-key-pattern.md) | Idempotency Key Pattern | Medium — consumes live inbound webhook events | Webhook computes a content hash per request, checks it against a "processed" file, and skips duplicate deliveries while still responding 200. |
| 4 | [Self-Healing Link Checker](docs/04-watcher-pattern.md) | Watcher Pattern (Change Detection) | None | Loop an HTTP Request over a list of URLs; Code node flags broken links (4xx/5xx/timeout) and only notifies when the broken-link set changed since the last run. |
| 5 | [Mini Order State Machine](docs/05-state-pattern.md) | State Pattern (State Machine) | Medium — driven by live webhook calls | Model an order-status progression via repeated webhook calls; a Code node validates legal transitions, rejects illegal jumps, and persists current state to a file. |
| 6 | [Event-Sourced Todo List](docs/06-event-sourcing-pattern.md) | Event Sourcing Pattern | Medium — webhook appends events live | Webhook appends create/update/complete events to an append-only log file; a separate Code node replays the full event log to reconstruct current state on demand. |
| 7 | [Stale-Lock-Aware Scheduler Guard](docs/07-distributed-lock-pattern.md) | Distributed Lock (Mutex) Pattern | None | Schedule Trigger acquires a file-based lock before running and releases it after, with logic to detect and clear a stale lock left by a crashed previous run. |
| 8 | [Concurrent Paginated Crawler](docs/08-fan-out-fan-in-pattern.md) | Fan-Out/Fan-In Pattern | None | Crawl a paginated public API using SplitInBatches to fan out several page requests per batch, then fan back in via Merge into one deduped result set. |
| 9 | [RabbitMQ Work Queue Dispatcher](docs/09-competing-consumers-pattern.md) | Competing Consumers Pattern (EIP) | High — multiple live workers race to consume each message | Several worker workflows each run a RabbitMQ Trigger consuming from the same queue; RabbitMQ round-robins deliveries across whichever workers are online, so throughput scales by adding workers, not code. |
| 10 | [MQTT State-Change Notifier](docs/10-observer-pattern.md) | Observer Pattern (MQTT Pub/Sub) | High — true real-time pub/sub push via MQTT | MQTT Trigger node subscribes to a topic on a free public broker (e.g. `test.mosquitto.org`); a separate workflow publishes state-change messages via the MQTT node. The subscriber reacts to each message as it arrives. |
| 11 | [Multi-Subscriber Event Broker](docs/11-publish-subscribe-pattern.md) | Publish-Subscribe / Broker Pattern | High — one broadcast reaches many live subscribers at once | Multiple independent subscriber workflows (an MQTT Trigger each, or bound to a RabbitMQ fanout exchange) listen to the same topic and react differently — one notifies Discord, one archives to a file, one logs metrics — fully decoupled through the broker. |
| 12 | [Telegram Command Bot](docs/12-command-pattern.md) | Command Pattern (Real-Time Chat Bot) | High — live two-way chat | A Telegram Trigger node receives live messages; a Code node maps each `/command` string to a handler function via a command registry and replies in real time. |
| 13 | [RabbitMQ RPC Client/Server](docs/13-rpc-pattern.md) | RPC Pattern (Request-Reply, EIP) | High — synchronous-style live request/response over a broker | A "client" workflow publishes a request to a RabbitMQ queue with a `replyTo` queue and a Code-node-generated `correlationId`; a "server" workflow consumes it, processes it, and publishes the response to `replyTo` with the same `correlationId`, which the client's RabbitMQ Trigger uses to match the reply to the right in-flight request. |
| 14 | [Content-Based Router](docs/14-content-based-router-pattern.md) | Content-Based Router Pattern (EIP) | Medium — live routing of inbound messages by content | Publish messages to a RabbitMQ topic exchange; a Code node computes a routing key from message content (type, priority, region); differently-bound consumer workflows only ever receive the messages relevant to them. |
| 15 | [Custom JSON Diff & Patch Engine](docs/15-memento-pattern.md) | Memento Pattern (Snapshot/Diff) | None | Code node implements its own deep-diff algorithm (no library) comparing two JSON snapshots, and emits a minimal patch describing only the changed fields. |
| 16 | [Multi-Source Schema Normalizer](docs/16-adapter-pattern.md) | Adapter Pattern | None | Pull from 2–3 free APIs with different shapes (weather, quotes, repo stats) and write an adapter registry in a Code node that maps each source into one canonical schema before merging. |
| 17 | [Circuit Breaker for a Flaky API](docs/17-circuit-breaker-pattern.md) | Circuit Breaker Pattern | Low — guards a call path, not a persistent channel | Code node tracks consecutive failures against a public endpoint, "trips" the breaker after a threshold, short-circuits calls while open, and probes with a half-open retry after a cooldown. |
| 18 | [Recursive Tree Crawler](docs/18-composite-pattern.md) | Composite Pattern (Recursive Tree) | None | An Execute Workflow node calls itself to walk a nested/tree-shaped structure (e.g. a GitHub repo's directory tree) uniformly until it hits leaf nodes, aggregating results back up. |
| 19 | [HMAC Webhook Verifier & Router](docs/19-decorator-pattern.md) | Decorator Pattern | Medium — wraps a live inbound webhook | Wrap a webhook handler with an HMAC signature-verification step (computed manually via Node's `crypto` module) that runs before the real handler, then route verified payloads by type. |
| 20 | [File-Backed TTL Cache](docs/20-cache-aside-pattern.md) | Cache-Aside Pattern | None | Code node implements a cache layer with TTL in a local JSON file to avoid redundant calls to a rate-limited free API, including cache invalidation logic. |
| 21 | [RabbitMQ Priority Queue Processor](docs/21-priority-queue-pattern.md) | Priority Queue Pattern | Medium — live, broker-ordered delivery | Publish tasks to a RabbitMQ queue declared with `x-max-priority`; a Code node assigns each message's priority from its metadata, and the broker itself delivers higher-priority messages first — the queueing discipline lives in RabbitMQ, not a hand-rolled heap. |
| 22 | [Flat-File ETL Warehouse](docs/22-pipes-and-filters-pattern.md) | Pipes and Filters Pattern | None | Pull from multiple free APIs on a schedule, pass results through a chain of independent transform steps, and append normalized records to a growing NDJSON "warehouse" file. |
| 23 | [Sliding-Window Webhook Throttle](docs/23-sliding-window-pattern.md) | Sliding Window Pattern | Medium — throttles a live inbound webhook channel | Webhook implements sliding-window rate limiting in a Code node (backed by a file-based timestamp log) and rejects requests over the limit with a 429-style response. |
| 24 | [RabbitMQ Dead-Letter Exchange Handler](docs/24-dead-letter-queue-pattern.md) | Dead Letter Queue Pattern (EIP) | High — broker-native dead-lettering of live messages | A RabbitMQ queue is declared with a dead-letter-exchange; messages that are rejected or expire (TTL) are automatically routed by RabbitMQ itself to a DLQ, with no polling involved; a separate workflow consumes the DLQ, retries with backoff, and gives up after a max-attempts header count. |
| 25 | [Simulated Build Pipeline](docs/25-chain-of-responsibility-pattern.md) | Chain of Responsibility Pattern | Medium — triggered by a live webhook event | A webhook receives a GitHub-style push payload and runs it through a sequence of stage handlers (lint/test/deploy stand-ins), where any stage can halt the chain on failure. |
| 26 | [Custom Template Engine](docs/26-template-method-pattern.md) | Template Method Pattern | None | Code node implements a small mustache-style template renderer from scratch (variable substitution, loops, conditionals) used to generate HTML emails/reports from data. |
| 27 | [Config-Driven Multi-Tenant Workflow](docs/27-strategy-pattern.md) | Strategy Pattern | None | A single workflow reads a non-secret "tenant config" file and selects different formatting/destination/schedule behavior per tenant at runtime via JS-computed routing. |
| 28 | [Graph Traversal Workflow](docs/28-visitor-pattern.md) | Visitor Pattern (Graph Traversal) | None | Model a small graph (e.g. a dependency list or org chart) as JSON and implement BFS/DFS in a Code node to compute shortest path or detect cycles. |
| 29 | [Self-Monitoring Health Check](docs/29-watchdog-pattern.md) | Watchdog Pattern | Low — monitors execution health, can push live alerts | A secondary scheduled workflow queries n8n's own local REST API for recent execution results and alerts only on meaningful failure patterns (e.g. 3 failures in a row), not every blip. |
| 30 | [Automation Hub Orchestrator — Capstone](docs/30-saga-pattern.md) | Saga Pattern (Orchestration) | High — combines live MQTT/RabbitMQ/webhook events with scheduled orchestration | A central orchestrator workflow coordinates several sub-workflows (via Execute Workflow) for ingestion, transformation, and notification, tracked through a shared JSON event log with idempotency keys and backoff retry, finishing with a nightly health-report email. |
