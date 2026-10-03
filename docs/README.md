# Pattern Docs

One page per workflow in the [practice agenda](../README.md#practice-agenda--30-design-pattern-workflows), each with the n8n node layout and a working Code-node JS example for that pattern.

| # | Doc | Pattern |
|---|-----|---------|
| 1 | [Retry Pattern](01-retry-pattern.md) | Retry Pattern (Backoff + Jitter) |
| 2 | [Token Bucket Pattern](02-token-bucket-pattern.md) | Token Bucket Pattern |
| 3 | [Idempotency Key Pattern](03-idempotency-key-pattern.md) | Idempotency Key Pattern |
| 4 | [Watcher Pattern](04-watcher-pattern.md) | Watcher Pattern (Change Detection) |
| 5 | [State Pattern](05-state-pattern.md) | State Pattern (State Machine) |
| 6 | [Event Sourcing Pattern](06-event-sourcing-pattern.md) | Event Sourcing Pattern |
| 7 | [Distributed Lock Pattern](07-distributed-lock-pattern.md) | Distributed Lock (Mutex) Pattern |
| 8 | [Fan-Out/Fan-In Pattern](08-fan-out-fan-in-pattern.md) | Fan-Out/Fan-In Pattern |
| 9 | [Competing Consumers Pattern](09-competing-consumers-pattern.md) | Competing Consumers Pattern (EIP) |
| 10 | [Observer Pattern](10-observer-pattern.md) | Observer Pattern (MQTT Pub/Sub) |
| 11 | [Publish-Subscribe Pattern](11-publish-subscribe-pattern.md) | Publish-Subscribe / Broker Pattern |
| 12 | [Command Pattern](12-command-pattern.md) | Command Pattern (Real-Time Chat Bot) |
| 13 | [RPC Pattern](13-rpc-pattern.md) | RPC Pattern (Request-Reply, EIP) |
| 14 | [Content-Based Router Pattern](14-content-based-router-pattern.md) | Content-Based Router Pattern (EIP) |
| 15 | [Memento Pattern](15-memento-pattern.md) | Memento Pattern (Snapshot/Diff) |
| 16 | [Adapter Pattern](16-adapter-pattern.md) | Adapter Pattern |
| 17 | [Circuit Breaker Pattern](17-circuit-breaker-pattern.md) | Circuit Breaker Pattern |
| 18 | [Composite Pattern](18-composite-pattern.md) | Composite Pattern (Recursive Tree) |
| 19 | [Decorator Pattern](19-decorator-pattern.md) | Decorator Pattern |
| 20 | [Cache-Aside Pattern](20-cache-aside-pattern.md) | Cache-Aside Pattern |
| 21 | [Priority Queue Pattern](21-priority-queue-pattern.md) | Priority Queue Pattern |
| 22 | [Pipes and Filters Pattern](22-pipes-and-filters-pattern.md) | Pipes and Filters Pattern |
| 23 | [Sliding Window Pattern](23-sliding-window-pattern.md) | Sliding Window Pattern |
| 24 | [Dead Letter Queue Pattern](24-dead-letter-queue-pattern.md) | Dead Letter Queue Pattern (EIP) |
| 25 | [Chain of Responsibility Pattern](25-chain-of-responsibility-pattern.md) | Chain of Responsibility Pattern |
| 26 | [Template Method Pattern](26-template-method-pattern.md) | Template Method Pattern |
| 27 | [Strategy Pattern](27-strategy-pattern.md) | Strategy Pattern |
| 28 | [Visitor Pattern](28-visitor-pattern.md) | Visitor Pattern (Graph Traversal) |
| 29 | [Watchdog Pattern](29-watchdog-pattern.md) | Watchdog Pattern |
| 30 | [Saga Pattern](30-saga-pattern.md) | Saga Pattern (Orchestration) — Capstone |

All file-backed state in these examples is assumed to live under `.state/` relative to the mounted `workflows/` directory inside the n8n container — create that folder (gitignored) as needed, or point `*_FILE` constants wherever you prefer.
