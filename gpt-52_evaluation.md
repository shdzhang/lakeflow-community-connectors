# Community Connectors Evaluation (Jan 2026)

Scoring uses a **1–3** scale per criterion:

| Criterion | Weight | 1 (Lowest) | 2 | 3 (Highest) |
|---|---:|---|---|---|
| Completeness & Functionality | 50% | Missing functionality / not fully working / small % API coverage | Working, but partial functionality or partial API coverage | Working, full functionality, complete or near-complete API coverage |
| Methodology & Reusability | 30% | Methodology docs missing; little reusable template value | Methodology documented, but no notable novel approaches; some reusable elements | Clear methodology + novel approaches; multiple reusable elements |
| Code Quality & Efficiency | 20% | Hard to maintain; inefficient ingestion | Reasonable quality; not optimized for ingestion | Well-written and efficient ingestion |

## Summary Scores

Weighted score formula: \(0.5 \cdot C + 0.3 \cdot M + 0.2 \cdot Q\).

| Connector | Completeness (50%) | Methodology (30%) | Code Quality (20%) | **Weighted (1–3)** |
|---|---:|---:|---:|---:|
| Qualtrics | 2 | 3 | 3 | **2.5** |
| GitHub | 2 | 2 | 2 | **2.0** |
| HubSpot | 2 | 2 | 2 | **2.0** |
| Zendesk | 2 | 2 | 2 | **2.0** |
| Mixpanel | 2 | 2 | 1 | **1.8** |
| Stripe | 1 | 2 | 2 | **1.5** |

---

## GitHub

**API cross-check (official docs)**: GitHub REST API covers hundreds of resource groups; this connector focuses on a repo-centric subset (issues/PRs/commits/etc.). See [GitHub REST API docs](https://docs.github.com/en/rest).

### Ratings

| Completeness | Methodology | Code Quality |
|---:|---:|---:|
| 2 | 2 | 2 |

### Completeness & Functionality (50%)

- **Strengths**
  - **Incremental support** for high-value entities (`issues`, `pull_requests`, `comments`) via `since` + `updated_at` cursor.
  - **Pagination** handled via `Link` header and max page safety limit (predictable runtime).
  - **Clear table contracts** (primary keys + ingestion types) in README; static table list reduces surprises.
- **Opportunities for improvement / new ideas**
  - **API surface coverage**: add high-demand domains (releases, workflows/actions, deployments, checks, security alerts, tags, labels, milestones, projects) to better match the REST surface.
  - **Rate limiting**: add backoff using `X-RateLimit-Remaining`, `X-RateLimit-Reset`, and secondary rate limit handling.
  - **Efficiency**: stream/yield records per page (avoid `records` accumulation for large repos); add conditional requests (ETag / `If-None-Match`) where applicable.

### Methodology & Reusability (30%)

- **Strengths**
  - **Consistent Lakeflow interface** and shared test harness integration (`LakeflowConnectTester`).
  - **Option hygiene** (`externalOptionsAllowList`) and table-specific options documented.
- **Opportunities for improvement / new ideas**
  - **Methodology artifact**: add a short “how we built this” doc (design choices: cursoring, Link pagination, schema strategy, limits).
  - **Reusable HTTP layer**: factor a shared `requests.Session` wrapper for retries/backoff across all connectors.
  - **Connector template extraction**: promote common patterns (cdc cursor + lookback, Link pagination) into `libs/` utilities.

### Code Quality & Efficiency (20%)

- **Strengths**
  - **Readable per-table implementation** and explicit cursor/lookback behavior.
  - **Schema-first approach** reduces downstream inference issues.
- **Opportunities for improvement / new ideas**
  - **Hardening**: add retry/backoff on transient 5xx/connection errors; improve error messages with request context.
  - **Incremental correctness**: verify all CDC tables use appropriate “updated” cursor semantics (some endpoints don’t support `since`).
  - **Performance**: consider repo-level concurrency controls with global rate-limit coordination.

### Top 3 recommended next actions

- **Add robust rate-limit + retry strategy** (primary + secondary rate limits) and shared HTTP utility across connectors.
- **Expand table coverage** to a “core analytics set” (releases, workflows/actions, tags, milestones, labels).
- **Stream results** (page iterator) to avoid storing whole datasets in memory.

---

## HubSpot

**API cross-check (official docs)**: HubSpot’s platform includes CRM + marketing + CMS + automation; this connector targets CRM objects (plus custom objects) via CRM v3 endpoints. See [HubSpot API docs](https://developers.hubspot.com/docs/api/overview).

### Ratings

| Completeness | Methodology | Code Quality |
|---:|---:|---:|
| 2 | 2 | 2 |

### Completeness & Functionality (50%)

- **Strengths**
  - **Dynamic schema discovery** via the Properties API enables custom fields without code changes.
  - **Incremental sync** uses Search API with `GTE` filter on last-modified property (good pattern for large accounts).
  - **Associations support** included via CRM object query parameters.
- **Opportunities for improvement / new ideas**
  - **API surface coverage**: add non-CRM domains commonly needed (owners, pipelines, properties metadata snapshots, engagements timeline APIs, marketing email events) where feasible.
  - **Associations depth**: add an explicit association “link table” strategy (object_id ↔ associated_id) to avoid embedding arrays in every row.
  - **Reliability**: add first-class handling for `429` + `Retry-After` and HubSpot-specific burst limits.

### Methodology & Reusability (30%)

- **Strengths**
  - **Central object config** (`_object_config`) is a reusable pattern for future CRM connectors.
  - **Caching** for schema/metadata reduces repeated discovery calls.
- **Opportunities for improvement / new ideas**
  - **Methodology doc**: add a short note on why Search API is used for incremental and how cursor fields map per object.
  - **Extract reusable discovery module** (properties → schema mapping) for other “schema-by-metadata” APIs.
  - **Connector generation**: document how `_generated_*` files relate to hand-maintained `hubspot.py`.

### Code Quality & Efficiency (20%)

- **Strengths**
  - **Separation** between full refresh and incremental paths.
  - **Type mapping** is centralized and easy to adjust.
- **Opportunities for improvement / new ideas**
  - **Memory**: avoid building `all_records` (yield per page).
  - **Error handling**: standardize exceptions; fix non-f-string error messages; add structured logs.
  - **Schema robustness**: handle Properties API failures deterministically (don’t return dict-as-error where list expected).

### Top 3 recommended next actions

- **Implement robust retry/backoff** for 429/5xx + request timeouts (shared utility).
- **Refactor to streaming reads** (iterator per page) to improve scalability.
- **Add “association link tables”** (normalized M:N) for analytics-friendly modeling.

---

## Qualtrics

**API cross-check (official docs)**: Qualtrics has multiple domains; this connector covers the core survey analytics workflow (surveys, definitions, response exports, distributions, directories/contacts). See [Qualtrics API docs](https://api.qualtrics.com/).

### Ratings

| Completeness | Methodology | Code Quality |
|---:|---:|---:|
| 2 | 3 | 3 |

### Completeness & Functionality (50%)

- **Strengths**
  - **Correct export workflow** implemented for responses (create → poll → download/parse).
  - **Auto-consolidation** across surveys reduces downstream unions and operational toil.
  - **Pragmatic schema strategy** for dynamic question structures (`MapType` / JSON strings).
- **Opportunities for improvement / new ideas**
  - **API surface coverage**: add commonly requested admin domains (users, groups, brands, libraries) where APIs allow.
  - **Operational controls**: add configurable concurrency (bounded by Qualtrics limits) for multi-survey consolidation.
  - **Incremental tuning**: add lookback window for `recordedDate` to handle eventual consistency / late availability.

### Methodology & Reusability (30%)

- **Strengths**
  - **Clear implementation methodology embedded in README** (workflow, rate limits, performance expectations, versioning).
  - **Reusable multi-entity pattern**: `_iterate_all_surveys(...)` is a strong template for “fan-out then consolidate” connectors.
- **Opportunities for improvement / new ideas**
  - **Short design doc**: document tradeoffs (JSON strings vs nested structs, per-survey offsets) and recommended downstream parsing.
  - **Template extraction**: lift export-job polling + retry module to `libs/` for other async-export APIs.
  - **Test strategy**: add deterministic unit tests for polling/retry logic using mocked HTTP responses.

### Code Quality & Efficiency (20%)

- **Strengths**
  - **Central config constants** for retries/polling, structured logging, and explicit timeouts.
  - **Rate-limit aware behavior** with controlled delays and retries.
- **Opportunities for improvement / new ideas**
  - **Streaming downloads** for large exports (avoid loading full ZIP/JSON into memory when possible).
  - **Observability**: add progress metrics (surveys processed, export durations, retry counts).
  - **Schema evolution**: add explicit schema versioning fields to records for long-term stability.

### Top 3 recommended next actions

- **Add configurable lookback + concurrency controls** for multi-survey consolidation (correctness + performance).
- **Extract reusable “async export job” helper** into `libs/` and reuse across future connectors.
- **Add mocked unit tests** for retries/polling/download parsing (no live API needed).

---

## Mixpanel

**API cross-check (official docs)**: Mixpanel’s commonly used ingestion surfaces include Raw Event Export, Engage (profiles), and Cohorts. This connector covers those core areas but with ingestion correctness/efficiency gaps. See [Mixpanel developer docs](https://developer.mixpanel.com/).

### Ratings

| Completeness | Methodology | Code Quality |
|---:|---:|---:|
| 2 | 2 | 1 |

### Completeness & Functionality (50%)

- **Strengths**
  - **Covers core analytics objects**: events export, cohorts, cohort membership, and profiles.
  - **Basic rate limiting** for export API (3 req/sec) via fixed sleep.
- **Opportunities for improvement / new ideas**
  - **Incremental correctness**: use a cursor based on `properties.time` (and add lookback); current “date chunking” can miss late-arriving events.
  - **API surface coverage**: consider adding annotations, schema/events metadata, funnels/retention exports (where available) as optional tables.
  - **Large-scale export**: support partitioned exports and resumable checkpoints beyond “start_date”.

### Methodology & Reusability (30%)

- **Strengths**
  - **Consistent Lakeflow entrypoints** and shared test harness integration.
  - **Reusable idea**: split exports into fixed-size time windows.
- **Opportunities for improvement / new ideas**
  - **Methodology doc**: document cursor strategy and why “time windows” were chosen.
  - **Refactor into common export framework**: reuse the same batching/cursor/retry module for other event-stream APIs.
  - **Generated vs hand-written**: document why `_generated_*` exists and what should be edited manually.

### Code Quality & Efficiency (20%)

- **Strengths**
  - **Simple control flow**; easy to follow.
- **Opportunities for improvement / new ideas**
  - **Replace `print()` with structured logging** and remove verbose per-line debug in hot paths.
  - **Stream JSONL**: iterate response lines as a generator (avoid storing all events in `all_records`).
  - **Retries/backoff**: add robust handling for 429/5xx/network failures; current “return partial on 429” is fragile.

### Top 3 recommended next actions

- **Fix incremental model**: cursor = max `properties.time` with lookback; ensure late events are captured.
- **Refactor to streaming + logging** (no full in-memory accumulation; replace `print()`).
- **Add retries/backoff + resumable checkpoints** per batch window.

---

## Stripe

**API cross-check (official docs)**: Stripe has a large object surface; list endpoints are easy, but **true CDC** generally needs Events or object-specific “updated” semantics (often not filterable). See [Stripe API docs](https://docs.stripe.com/api).

### Ratings

| Completeness | Methodology | Code Quality |
|---:|---:|---:|
| 1 | 2 | 2 |

### Completeness & Functionality (50%)

- **Strengths**
  - **Broad starter table set** (customers, charges, payment intents, subscriptions, invoices, products/prices, payouts, events).
  - **Correct pagination** for list endpoints via `starting_after` and `has_more`.
- **Opportunities for improvement / new ideas**
  - **CDC correctness gap**: using `created[gte]` as “incremental” will miss **updates** to existing objects; use the **Events API** (`/v1/events`) to drive CDC properly.
  - **API surface coverage**: add core finance/connect domains (transfers, balance, connected accounts, disputes evidence details, refunds relationships) based on common analytics needs.
  - **Deletions**: standardize deletion handling beyond customers/products/plans; consider event-driven tombstones.

### Methodology & Reusability (30%)

- **Strengths**
  - **Uniform list-endpoint pattern** is reusable for many Stripe resources.
  - **Central object config** encourages extension by adding new table entries.
- **Opportunities for improvement / new ideas**
  - **Document CDC strategy** (created-based vs event-driven) and tradeoffs; align with Stripe’s recommended patterns.
  - **Extract a generic “Stripe list reader”** helper into `libs/` for quick table additions.
  - **Schema strategy**: decide and document JSON-vs-struct approach per nested field family.

### Code Quality & Efficiency (20%)

- **Strengths**
  - **Simple, consistent pagination loops** and predictable offsets.
- **Opportunities for improvement / new ideas**
  - **Retries/backoff** for 429/5xx and network errors.
  - **Memory**: stream records instead of returning full `List[Dict]`.
  - **Observability**: record request counts and latency per endpoint for operational tuning.

### Top 3 recommended next actions

- **Rebuild incremental sync using Events API** as the CDC backbone (updates + deletes).
- **Add robust retry/backoff** (429 handling) and stream records to reduce memory.
- **Expand tables strategically** (connect payouts/transfers/balance; pick “top 10” business-critical objects).

---

## Zendesk

**API cross-check (official docs)**: Zendesk Support has strong incremental export endpoints (tickets/users/orgs/events) plus many additional domains (audit logs, ticket fields/forms, satisfaction, SLAs, etc.). See [Zendesk API docs](https://developer.zendesk.com/api-reference/).

### Ratings

| Completeness | Methodology | Code Quality |
|---:|---:|---:|
| 2 | 2 | 2 |

### Completeness & Functionality (50%)

- **Strengths**
  - **Uses incremental export endpoints** for high-volume objects (`tickets`, `users`, `organizations`, `ticket_events`).
  - **Extracts comment events** into a dedicated `ticket_comments` table (useful normalization).
  - **Covers core support analytics primitives** (tickets, users, orgs, groups, brands, articles, topics).
- **Opportunities for improvement / new ideas**
  - **Offset correctness**: use `end_time` (from incremental export responses) as the checkpoint instead of recomputing from record timestamps.
  - **API surface coverage**: add ticket fields/forms, satisfaction ratings, ticket audits, metrics, SLAs, and group/user memberships.
  - **Help Center pagination**: support cursor-based pagination where available and avoid page caps like 1000.

### Methodology & Reusability (30%)

- **Strengths**
  - **Clear endpoint mapping** per table (easy to extend).
  - **Shared test harness** integration is consistent with other connectors.
- **Opportunities for improvement / new ideas**
  - **Methodology doc**: explicitly document why incremental export endpoints were chosen and how start_time/end_time works.
  - **Extract “incremental export reader”** helper into `libs/` (Zendesk-style cursoring can apply to similar APIs).
  - **Add connector-level option knobs** (page size, max pages, safety limits) consistently.

### Code Quality & Efficiency (20%)

- **Strengths**
  - **Straightforward implementation** and readable transformations for comments.
- **Opportunities for improvement / new ideas**
  - **Retries/backoff** for rate limits and transient errors; use `Retry-After` when present.
  - **Streaming**: yield records as pages arrive (avoid `all_records`).
  - **Schema hygiene**: ensure timestamps are consistently typed (string vs timestamp) across tables.

### Top 3 recommended next actions

- **Fix incremental checkpointing** to use `end_time` and add a configurable lookback window.
- **Add robust retry/backoff** (429 + transient errors) and stream records.
- **Expand support tables** (ticket audits/metrics, satisfaction, ticket fields/forms, memberships).


