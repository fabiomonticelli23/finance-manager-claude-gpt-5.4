# Architecture Research

**Domain:** Self-hosted personal portfolio management web application
**Researched:** 2026-04-20
**Confidence:** HIGH

## Standard Architecture

### System Overview

```text
┌──────────────────────────────────────────────────────────────────────────────┐
│                              Presentation Layer                             │
├──────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────────────────────┐ │
│  │ Portfolio       │  │ Holdings &      │  │ Rebalancing & Review        │ │
│  │ Dashboard UI    │  │ Account UI      │  │ Workflow UI                 │ │
│  └────────┬────────┘  └────────┬────────┘  └──────────────┬───────────────┘ │
│           │                    │                          │                 │
├───────────┴────────────────────┴──────────────────────────┴─────────────────┤
│                               Application API                               │
├──────────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌────────────────────┐ │
│  │ Portfolio    │ │ Ingestion    │ │ Pricing & FX │ │ Rebalancing        │ │
│  │ Query Module │ │ Command API  │ │ Engine       │ │ Engine             │ │
│  └──────┬───────┘ └──────┬───────┘ └──────┬───────┘ └─────────┬──────────┘ │
│         │                │                │                   │            │
│  ┌──────┴───────┐ ┌──────┴────────┐ ┌─────┴─────────┐ ┌───────┴──────────┐ │
│  │ Snapshot     │ │ Sync          │ │ Classification │ │ Recommendation   │ │
│  │ Service      │ │ Orchestrator  │ │ & Allocation   │ │ Formatter        │ │
│  └──────┬───────┘ └──────┬────────┘ └─────┬─────────┘ └───────┬──────────┘ │
├─────────┴────────────────┴────────────────┴───────────────────┴────────────┤
│                            Background Processing                             │
├──────────────────────────────────────────────────────────────────────────────┤
│  ┌────────────────┐  ┌────────────────┐  ┌───────────────────────────────┐ │
│  │ Scheduler      │  │ Queue Workers  │  │ Snapshot / Price Refresh Jobs │ │
│  │ (cron/timers)  │  │ (sync/pricing) │  │ and reconciliation            │ │
│  └────────┬───────┘  └────────┬───────┘  └──────────────┬────────────────┘ │
│           │                   │                         │                  │
├───────────┴───────────────────┴─────────────────────────┴──────────────────┤
│                              Persistence Layer                              │
├──────────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌────────────────────┐ │
│  │ PostgreSQL   │ │ Snapshot     │ │ Job / Sync   │ │ Config / Targets   │ │
│  │ source data  │ │ history      │ │ state        │ │ and rules          │ │
│  └──────────────┘ └──────────────┘ └──────────────┘ └────────────────────┘ │
└──────────────────────────────────────────────────────────────────────────────┘

External integrations:
User/manual input → Account & asset commands
CCXT exchanges → Sync workers → normalized holdings
FX / market price providers → Pricing engine → valuation tables
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| React frontend | Portfolio review workflow, editing manual assets, dashboards, sync visibility, rebalancing views | React app with route-level pages, charts, mutation/query hooks |
| NestJS API | Owns domain modules, validation, orchestration, read models, and business rules | Modular NestJS monolith |
| Sync orchestrator | Starts exchange sync jobs, manages retries, records job state, prevents overlapping runs | Nest scheduler plus BullMQ workers |
| Exchange adapters | Pull balances/positions from CCXT exchanges and map them to canonical internal records | CCXT-backed adapter layer per source type |
| Manual asset module | Creates and updates bank, broker, cash, emergency fund, and unsupported asset records | CRUD + validation + audit timestamps |
| Pricing and FX engine | Resolves market prices, FX conversion rates, and portfolio base-currency valuations | Service layer with cached quotes and valuation pipeline |
| Classification and allocation engine | Maps holdings into macro categories, liquidity buckets, and per-asset allocation trees | Rule-based domain service |
| Snapshot service | Captures point-in-time portfolio totals/allocation states for history and charts | Scheduled write model into append-only snapshot tables |
| Rebalancing engine | Compares actual allocation to targets and produces advisory actions | Deterministic calculation service |
| PostgreSQL | System of record for sources, assets, targets, valuations, snapshots, and job metadata | Relational schema with strong constraints |
| Redis | Queue backend and transient worker coordination | BullMQ dependency |

## Recommended Project Structure

```text
backend/
├── src/
│   ├── app/                       # Nest bootstrap and shared module wiring
│   ├── modules/
│   │   ├── sources/               # Exchange accounts, manual accounts, sync config
│   │   ├── ingestion/             # Sync commands, job dispatch, source normalization
│   │   ├── holdings/              # Canonical holdings and balances
│   │   ├── pricing/               # Market prices, FX rates, valuation logic
│   │   ├── allocations/           # Category mapping, targets, drift calculation
│   │   ├── snapshots/             # Historical snapshot creation and querying
│   │   ├── rebalancing/           # Recommendation engine and explanation layer
│   │   ├── dashboard/             # Optimized read models for charts and overview pages
│   │   └── system/                # Health, jobs, settings, observability
│   ├── infrastructure/
│   │   ├── ccxt/                  # Exchange adapter implementations
│   │   ├── queues/                # BullMQ queues and processors
│   │   ├── scheduler/             # Cron registration and dynamic schedules
│   │   ├── persistence/           # TypeORM/Prisma models, repositories, migrations
│   │   └── market-data/           # FX and price provider clients
│   └── shared/                    # Types, money utils, date/time helpers, errors
├── test/                          # Integration and domain tests
└── Dockerfile                     # Backend container

frontend/
├── src/
│   ├── app/                       # Routing, layout, app shell
│   ├── pages/                     # Dashboard, assets, accounts, targets, history
│   ├── features/
│   │   ├── portfolio/             # Overview widgets and allocation charts
│   │   ├── manual-assets/         # Forms and editing flows
│   │   ├── sync-status/           # Last sync, job history, source health
│   │   ├── snapshots/             # Trend charts and historical views
│   │   └── rebalancing/           # Drift views and recommended actions
│   ├── api/                       # Query/mutation clients and DTO mapping
│   └── components/                # Shared UI primitives and chart wrappers
└── Dockerfile                     # Frontend container

docker/
├── postgres/                      # DB config and initialization
├── redis/                         # Optional local redis config
└── compose/                       # docker-compose files
```

### Structure Rationale

- **modules/** separates domain ownership. Portfolio math, snapshots, and rebalancing change for different reasons and should not be mixed into one giant service.
- **infrastructure/** keeps CCXT, queue, and provider code outside core business rules so integrations can change without rewriting allocation logic.
- **dashboard/** should own read-optimized responses rather than forcing the frontend to assemble multiple low-level endpoints.
- **sources + ingestion + holdings** is the critical split: source configuration is not the same thing as fetched balances, and fetched balances are not the same thing as priced portfolio analytics.

## Architectural Patterns

### Pattern 1: Canonical portfolio ledger over raw source payloads

**What:** Persist raw sync payload metadata for troubleshooting, but convert every source into a canonical internal representation: account, holding, asset, quantity, cost basis if known, valuation currency, classification tags.

**When to use:** Always. Exchange APIs and manual records have incompatible schemas; analytics become unstable if downstream modules depend on source-specific payloads.

**Trade-offs:** Requires upfront mapping work, but prevents the rest of the system from becoming CCXT- or provider-shaped.

**Example:**
```typescript
interface CanonicalHolding {
  sourceId: string;
  accountId: string;
  assetCode: string;
  quantity: string;
  nativeCurrency?: string;
  nativeValue?: string;
  category: 'CRYPTO' | 'STOCKS' | 'LIQUIDITY' | 'TRADING_BOT';
  subcategory?: string;
  capturedAt: string;
}
```

### Pattern 2: Asynchronous ingestion, synchronous reads

**What:** Sync and pricing happen in background jobs; dashboard and rebalancing endpoints read only persisted data.

**When to use:** For any self-hosted finance app that talks to exchanges or price providers. External APIs are slow, flaky, and rate-limited.

**Trade-offs:** Users may see slightly stale data, but the UI becomes reliable and deterministic. This is the correct trade-off for personal portfolio review.

**Example:**
```typescript
// API command
await syncQueue.add('sync-source', { sourceId });

// API query
return dashboardReadModel.getPortfolioOverview({ asOf: latestCompletedSnapshot });
```

### Pattern 3: Snapshot-first analytics

**What:** Persist point-in-time computed portfolio states rather than recomputing all historical analytics from raw transactions or past balances on every request.

**When to use:** When the product needs trend charts, net worth history, and historical allocation views but does not need full tax/performance accounting in v1.

**Trade-offs:** Snapshot storage grows over time, but query logic stays simple. PostgreSQL range partitioning is useful only once snapshot tables become large; for v1, append-only timestamped rows are enough.

**Example:**
```typescript
interface PortfolioSnapshot {
  capturedAt: string;
  baseCurrency: string;
  totalNetWorth: string;
  emergencyFundValue: string;
  categoryBreakdown: Array<{ category: string; value: string; weight: string }>;
}
```

### Pattern 4: Deterministic recommendation engine

**What:** Rebalancing should be a pure calculation over latest holdings, valuations, targets, protected-allocation rules, and configurable tolerance bands.

**When to use:** Always, especially because v1 is advisory-only.

**Trade-offs:** Less flexible than AI-generated advice, but auditable and explainable. For finance recommendations, explainability beats novelty.

## Data Flow

### Primary Flow: Exchange sync to dashboard

```text
Scheduler / user-triggered sync
    ↓
Sync Orchestrator
    ↓
BullMQ job per source
    ↓
CCXT adapter
    ↓
Normalize balances/positions to canonical holdings
    ↓
Persist source refresh result + holdings snapshot
    ↓
Pricing & FX engine values holdings in base currency
    ↓
Allocation engine classifies holdings into target trees
    ↓
Snapshot service stores current portfolio snapshot
    ↓
Dashboard read model serves frontend queries
```

### Primary Flow: Manual asset update to analytics

```text
User edits manual account / asset
    ↓
NestJS command endpoint validates payload
    ↓
Manual asset module persists canonical record
    ↓
Pricing & FX engine revalues affected assets
    ↓
Allocation engine recomputes category totals
    ↓
Optional snapshot capture or next scheduled snapshot includes change
    ↓
Dashboard and rebalancing read models update
```

### Primary Flow: Rebalancing recommendation generation

```text
Latest valued holdings + target allocations + protected rules
    ↓
Rebalancing engine
    ↓
Drift calculation by macro category
    ↓
Drift calculation by liquidity bucket
    ↓
Drift calculation by in-category asset
    ↓
Emergency fund override / exclusion logic
    ↓
Action suggestions with reasons and amounts
    ↓
Frontend explanation UI
```

### State Management

```text
PostgreSQL = source of truth
    ↓
Backend read models expose stable query DTOs
    ↓
Frontend query cache subscribes to API responses
    ↓
Pages render overview, drift, and history
    ↓
Mutations invalidate affected queries
```

### Key Data Flows

1. **Source configuration flow:** User defines exchange/manual account, credentials/config are stored securely, sync schedule metadata is registered, and workers use only source IDs plus server-side secrets.
2. **Valuation flow:** Holdings in source currency move through market price lookup and FX conversion into base-currency valuations consumed by dashboards and rebalancing.
3. **Historical flow:** Current computed state is periodically frozen into snapshots so trend charts read snapshot tables, not ad hoc recomputation from live source data.
4. **Recommendation flow:** The engine reads computed current state and target definitions; it should never call external providers directly.

## Component Boundaries

| Component | Responsibility | Communicates With |
|-----------|---------------|-------------------|
| Frontend dashboard | Reads current overview, history, drift, sync status | Dashboard API, snapshots API, rebalancing API |
| Source management module | Creates exchange/manual sources and schedules | Ingestion module, system settings, persistence |
| Ingestion module | Dispatches sync jobs and records lifecycle state | Scheduler, BullMQ, CCXT adapters, holdings persistence |
| CCXT adapter layer | Talks to exchanges and returns normalized source data | Ingestion module only |
| Holdings module | Owns canonical asset/account/holding records | Ingestion, manual asset module, pricing engine |
| Pricing module | Resolves prices and FX, produces valuations | Holdings, market-data providers, snapshots, rebalancing |
| Allocation module | Assigns categories and computes actual allocation trees | Holdings, pricing, targets config, rebalancing |
| Snapshot module | Freezes current computed state into history tables | Pricing, allocations, dashboard queries |
| Rebalancing module | Produces advisory actions from current state and targets | Allocations, pricing, snapshots for context |
| Dashboard module | Serves aggregated read models for UI | Holdings, pricing, allocations, snapshots, sync state |

## Suggested Build Order

The right dependency order is not UI-first. Build the data spine first.

1. **Canonical domain model and persistence**
   - Accounts, sources, assets, holdings, categories, target allocations, snapshots, sync jobs.
   - Reason: every later component depends on stable IDs and money/date semantics.

2. **Manual asset workflow**
   - Create/update manual accounts and balances.
   - Reason: gives a fully testable portfolio path before dealing with exchange volatility.

3. **Pricing and FX valuation pipeline**
   - Base-currency conversion and current valuation.
   - Reason: dashboards and rebalancing are meaningless without normalized valuation.

4. **Allocation and target engine**
   - Macro categories, liquidity buckets, emergency fund handling, intra-category mapping.
   - Reason: this defines the business language used everywhere else.

5. **Dashboard read models**
   - Net worth, allocation charts, current holdings views.
   - Reason: lets the product become useful early, even before automated sync.

6. **Snapshot service and history queries**
   - Scheduled or event-based capture of current portfolio state.
   - Reason: trend charts should build on already-correct current-state calculations.

7. **Rebalancing engine**
   - Drift calculations and recommendation formatting.
   - Reason: recommendations depend on canonical holdings, valuation, and target logic already being trusted.

8. **Exchange sync orchestration via CCXT**
   - Add scheduler, queues, source adapters, retries, status tracking.
   - Reason: automated sync is operationally harder than manual entry; adding it after the core model avoids coupling business rules to transport details.

9. **Operational hardening**
   - Sync observability, idempotency controls, backoff, stale-data warnings, admin/system screens.
   - Reason: this matters once real sources are attached and failures become normal.

### Build Order Dependencies

```text
Canonical schema
  → Manual assets
  → Pricing/FX
  → Allocation targets
  → Dashboard read models
  → Snapshots
  → Rebalancing
  → Exchange sync automation
  → Operational hardening
```

## Scaling Considerations

| Scale | Architecture Adjustments |
|-------|--------------------------|
| 1 user / local-first | Single NestJS app, PostgreSQL, Redis, React frontend, Docker Compose. Keep a modular monolith. |
| 10-100 portfolios or heavier sync usage | Separate queue workers from API process, add provider-level caches, tighten job concurrency and retry policies. |
| Far beyond original scope | Split market-data and ingestion into dedicated services only if background work starts degrading request latency. |

### Scaling Priorities

1. **First bottleneck:** External integrations. Exchange and market-data calls will fail or rate-limit before PostgreSQL becomes the problem. Fix with queues, retries, caching, and stale-data indicators.
2. **Second bottleneck:** Historical snapshot growth and expensive aggregate queries. Fix with read models, pre-aggregated daily snapshots, and only then consider PostgreSQL time-based partitioning.

## Anti-Patterns

### Anti-Pattern 1: Live-calculate everything from external APIs on page load

**What people do:** Frontend requests trigger fresh exchange sync, price fetch, FX conversion, and analytics calculation in the same request.

**Why it's wrong:** The UI becomes slow, flaky, rate-limited, and impossible to reason about. Failures in any provider become dashboard failures.

**Do this instead:** Treat ingestion as background work and serve the UI from persisted current state plus freshness metadata.

### Anti-Pattern 2: Mixing raw source schema with business logic

**What people do:** Dashboards and rebalancing logic read exchange-specific asset names, balance shapes, and account semantics directly.

**Why it's wrong:** Every new source leaks special cases into every module.

**Do this instead:** Normalize all holdings into canonical models at the ingestion boundary.

### Anti-Pattern 3: Storing only current state and no historical snapshots

**What people do:** Overwrite holdings tables and expect to reconstruct historical charts later.

**Why it's wrong:** You lose trend visibility and eventually face a rewrite when history becomes a requirement.

**Do this instead:** Capture periodic append-only portfolio snapshots from the start, even if they are basic daily or on-sync snapshots.

### Anti-Pattern 4: Embedding recommendation rules in frontend code

**What people do:** The UI computes drift and advisory actions from raw API data.

**Why it's wrong:** Results become inconsistent across screens and hard to test.

**Do this instead:** Keep rebalancing rules server-side and return explainable recommendation DTOs.

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| CCXT exchanges | Adapter per source using server-side credentials and background workers | `loadMarkets` should happen before unified API use; exchange responses vary and need normalization |
| FX / market price providers | Pull latest rates into valuation service with caching | Separate cash FX from asset pricing logic |
| Redis | BullMQ queue backend | Required if using NestJS BullMQ for durable background jobs |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Source management ↔ ingestion | Direct service call + queue dispatch | Source config should not own sync execution logic |
| Ingestion ↔ CCXT adapters | Interface-based adapter API | Keeps exchange quirks isolated |
| Holdings ↔ pricing | Direct service/repository calls | Pricing reads holdings but should not mutate source config |
| Pricing/allocations ↔ snapshots | Direct write into snapshot tables | Snapshots should depend on computed state, not raw sources |
| Allocations ↔ rebalancing | Pure function/service calls | Rebalancing should remain deterministic and explainable |
| Dashboard ↔ all read-side modules | Query DTOs / aggregated services | Prefer dedicated read models over UI stitching |

## Sources

- NestJS task scheduling docs: https://docs.nestjs.com/techniques/task-scheduling
- NestJS Bull/BullMQ docs and Context7 excerpts: /nestjs/bull and https://github.com/nestjs/docs.nestjs.com/blob/master/content/techniques/queues.md
- CCXT docs and Context7 excerpts: /ccxt/ccxt and https://github.com/ccxt/ccxt/blob/master/doc/manual.rst
- CCXT spec for balance fetching: https://github.com/ccxt/ccxt/wiki/Spec
- PostgreSQL partitioning docs: https://www.postgresql.org/docs/current/ddl-partitioning.html
- PostgreSQL range types docs: https://www.postgresql.org/docs/current/rangetypes.html

---
*Architecture research for: self-hosted personal portfolio management systems*
*Researched: 2026-04-20*
