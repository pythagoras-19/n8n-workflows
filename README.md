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

## Practice Agenda — 25 Workflows

A docket of workflows to build, roughly ordered by difficulty, aimed at building both n8n workflow-design skills and JavaScript skills (via n8n's Code node). Everything here uses either keyless free APIs or free tiers of accounts (GitHub, Google, Telegram, Discord, Gmail) — nothing requires a paid plan or paid API key.

### Beginner — core mechanics + basic JS

1. **Hello Cron Logger** — Schedule Trigger fires every minute; a Code node formats a timestamp with JS `Date` methods; write the line to a local file. *Skills: triggers, Code node basics, date formatting.*
2. **Webhook Echo** — Webhook receives a POST body; a Code node validates and reshapes it; Respond to Webhook returns the transformed JSON. *Skills: webhooks, request/response shape, object manipulation.*
3. **Random Joke Fetcher** — HTTP Request to a free, keyless jokes API (e.g. `icanhazdadjoke.com`); Code node picks/formats the result. *Skills: HTTP node, JSON parsing.*
4. **Quote of the Day to Email** — Schedule Trigger calls a free quotes API (e.g. ZenQuotes, Quotable); Code node formats a message; send via Gmail/SMTP (free account). *Skills: scheduling, external APIs, Email node.*
5. **CSV to JSON Converter** — Read a local CSV; write a Code node that parses it to JSON by hand (no library) to practice string/array parsing; write the JSON output. *Skills: manual parsing logic, file I/O.*
6. **Array Deduper** — Webhook receives an array of objects; a Code node dedups by key using JS `Set`/`Map`; return the cleaned array. *Skills: dedup algorithms in JS.*
7. **Temperature Converter API** — Webhook accepts Celsius; Code node converts to Fahrenheit/Kelvin; Respond to Webhook. *Skills: building a tiny API with n8n.*
8. **Weather Fetcher** — Schedule Trigger calls Open-Meteo (free, no key) for your coordinates; Code node translates weather codes into human text; notify via a Telegram bot or Discord webhook (both free). *Skills: keyless APIs, conditional logic, notifications.*

### Intermediate — multi-step logic, loops, conditionals

9. **RSS Digest Bot** — RSS Feed Read node pulls a few blogs; Code node merges, sorts by date, and dedups against a "seen IDs" file you read/write yourself; send the digest via Telegram/Discord. *Skills: RSS node, state persistence via files, JS sorting/filtering.*
10. **GitHub New Issue Notifier** — Schedule Trigger polls a public repo's issues via the GitHub API (no token needed for low-volume public reads); Code node diffs against the last-seen list; post new issues to Discord. *Skills: polling pattern, diffing, webhook notifications.*
11. **Pagination Practice** — HTTP Request against a paginated public API (e.g. Rick and Morty API); use a Code node plus the SplitInBatches node to walk every page and aggregate the full result set. *Skills: pagination logic, looping nodes.*
12. **Currency Converter Webhook** — Webhook accepts an amount + currency pair; call a free, keyless FX API (e.g. Frankfurter); Code node computes and rounds the conversion; respond. *Skills: webhook APIs, external data, numeric formatting.*
13. **Markdown Table Generator** — Given a JSON array, write a Code node that builds a Markdown table string from scratch (no library); write it to a file. *Skills: string-building algorithms.*
14. **Word Frequency Counter** — Ingest a block of text; Code node tokenizes, normalizes, and counts word frequency in plain JS; return the top N. *Skills: text-processing algorithms.*
15. **Repo Stats Logger** — Schedule Trigger queries the GitHub API for a few repos' stars/forks; Code node formats a row; append it to a Google Sheet (free tier). *Skills: Google Sheets node, recurring data logging.*
16. **Mini URL Shortener** — Webhook (POST) accepts a long URL; Code node generates a short id (simple hash function you write); store the mapping in a local JSON file acting as a DB; a second Webhook (GET) looks up the code and redirects. *Skills: stateful mini-service, hashing logic, redirect responses.*
17. **Form Validator API** — Webhook accepts form fields; Code node validates them by hand (regex email check, required-field checks); return pass/fail plus an error list. *Skills: regex, validation logic.*
18. **Weekday Standup Reminder** — Schedule Trigger with a cron expression limited to weekdays; IF node double-checks the day; send a Slack/Discord message with a randomly chosen prompt from a JS array. *Skills: cron expressions, conditional nodes, randomization.*

### Advanced — orchestration, error handling, harder JS

19. **Retry-with-Backoff Caller** — Code node implements a manual retry loop with exponential backoff against a public endpoint; an Error Trigger workflow captures and logs failures. *Skills: error-handling nodes, backoff algorithms.*
20. **Rate-Limited Batch Processor** — Process a list (e.g. rows from a Google Sheet) in batches via SplitInBatches, with a Wait node between batches to respect a self-imposed rate limit; track progress in a counter file. *Skills: batching, Wait node, progress tracking.*
21. **Scrape + Digest** — HTTP Request fetches a scrape-friendly public page; the built-in HTML Extract node pulls structured data; Code node cleans/dedups it; email the digest. *Skills: HTML Extract node, data cleaning in JS.*
22. **Multi-Source Aggregator** — Pull from 2–3 free APIs (weather, quote, joke) in parallel branches; a Merge node combines them; Code node assembles one "daily digest" object; send as a single message. *Skills: parallel branches, Merge node modes, object composition.*
23. **Self-Healing Link Checker** — Read a list of URLs from a file/sheet; loop an HTTP Request over them; Code node flags broken links (4xx/5xx/timeout); only notify if the broken-link set changed since the last run (diff against stored state). *Skills: looping, status handling, diffing, conditional notification.*
24. **Mini State Machine** — Model an order-status progression (new → processing → shipped → delivered) via repeated webhook calls; a Code node validates legal state transitions and rejects illegal jumps, persisting current state to a file. *Skills: state-machine logic, validation.*
25. **Capstone: Personal Automation Hub** — On a schedule: fetch weather, a top Hacker News story, and a random quote; build an HTML email from scratch with JS template strings; send it; log the run to a local JSON history file; raise an alert via an Error Trigger sub-workflow on failure. *Skills: templating, multi-API orchestration, persistence, error handling — everything above, combined.*
