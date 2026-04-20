# Architecture Research

**Domain:** Single-user personal portfolio management web app
**Researched:** 2026-04-20
**Confidence:** HIGH

## Standard Architecture

### System Overview

```text
┌──────────────────────────────────────────────────────────────────────────────┐
│                           Presentation Layer                                │
├──────────────────────────────────────────────────────────────────────────────┤
│  ┌────────────────┐  ┌────────────────┐  ┌──────────────────────────────┐   │
│  │ Dashboard UI   │  │ Config / Admin │  │ Manual Refresh + Snapshot UI │   │
│  │ charts/tables  │  │ sources/targets│  │ sync status/history          │   │
│  └───────┬────────┘  └───────┬────────┘  └──────────────┬───────────────┘   │
│          │                   │                          │                   │
├──────────┴───────────────────┴──────────────────────────┴───────────────────┤
│                           API / Application Layer                            │
├──────────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌────────────────┐  ┌────────────────┐  ┌──────────────┐ │
│  │ Portfolio    │  │ Ingestion      │  │ Pricing & FX   │  │ Rebalancing  │ │
│  │ Query Module │  │ Orchestrator   │  │ Valuation      │  │ Suggestions   │ │
│  └──────┬───────┘  └──────┬─────────┘  └──────┬─────────┘  └──────┬───────┘ │
│         │                 │                   │                   │         │
│  ┌──────▼───────┐  ┌──────▼─────────┐  ┌──────▼─────────┐  ┌──────▼───────┐ │
│  │ Allocation   │  │ Source Adapters│  │ Snapshot       │  │ Policy Rules  │ │
│  │ Engine       │  │ CCXT/manual    │  │ Writer/Reader  │  │ emergency fund│ │
│  └──────┬───────┘  └──────┬─────────┘  └──────┬─────────┘  └──────────────┘ │
│         │                 │                   │                               │
├─────────┴─────────────────┴───────────────────┴──────────────────────────────┤
│                              Persistence Layer                                │
├──────────────────────────────────────────────────────────────────────────────┤
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐  ┌────────────┐ │
│  │ Source Tables  │  │ Holdings /     │  │ Derived Views  │  │ Snapshots   │ │
│  │ accounts/assets│  │ Prices / FX    │  │ portfolio read │  │ history     │ │
│  └────────────────┘  └────────────────┘  └────────────────┘  └────────────┘ │
└──────────────────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| React app | Render dashboard, forms, charts, manual refresh, and source/config screens | React feature folders with server-state fetching and mostly derived UI state |
| Portfolio query module | Serve current net worth, allocation trees, drilldowns, and dashboard DTOs | NestJS read-oriented services backed by SQL queries/views |
| Ingestion orchestrator | Coordinate sync runs across CCXT sources, manual sources, pricing, FX, and snapshot capture | NestJS application service invoked by scheduler and manual endpoint |
| Source adapters | Normalize external and manual inputs into canonical balance/holding records | One adapter per source type behind a shared interface |
| Pricing & FX valuation module | Fetch manual asset prices and currency conversion inputs, then compute base-currency values | NestJS services plus provider adapters and valuation functions |
| Allocation engine | Build macro, liquidity, and intra-category allocation trees including Trading Bot rollups | Pure domain service operating on normalized valued positions |
| Policy rules module | Apply emergency fund protection and other portfolio constraints before suggestions | Pure rule engine with explicit inputs/outputs |
| Rebalancing suggestion module | Produce suggestions-only deltas without execution side effects | Read-only planning service returning recommended actions |
| Snapshot module | Persist periodic portfolio state for trends and simple historical comparisons | Append-only snapshot writer plus read endpoints |
| PostgreSQL | Store authoritative config, holdings state, derived valuation inputs, and snapshots | Normalized tables, constraints, indexes, and a few read-optimized views |

## Recommended Project Structure

```text
backend/
├── src/
│   ├── app/                         # App bootstrap, config, scheduler wiring
│   ├── common/                      # Shared utilities, DTO primitives, errors, money types
│   ├── modules/
│   │   ├── sources/                 # Source registry and source management APIs
│   │   │   ├── ccxt/                # Exchange source configs and CCXT adapters
│   │   │   ├── manual-accounts/     # Bank/cash/manual current-value sources
│   │   │   ├── manual-holdings/     # Manual market-traded holdings needing prices
│   │   │   └── source-sync/         # Sync run orchestration and status tracking
│   │   ├── pricing/                 # Price providers, FX providers, valuation inputs
│   │   ├── portfolio/               # Holdings projection, net worth queries, dashboard reads
│   │   ├── allocation/              # Category tree construction and target comparison
│   │   ├── policies/                # Emergency fund and protected-allocation rules
│   │   ├── rebalancing/             # Suggestions engine and action explanations
│   │   ├── snapshots/               # Historical snapshot writes and reads
│   │   └── settings/                # Base currency, targets, scheduling, app preferences
│   ├── database/
│   │   ├── schema/                  # ORM schema/migrations
│   │   ├── repositories/            # Persistence adapters per aggregate or read model
│   │   └── views/                   # SQL views/materialized-read candidates if needed
│   └── main.ts                      # Nest bootstrap
├── test/                            # Integration tests by module and end-to-end flows
└── Dockerfile                       # Backend container

frontend/
├── src/
│   ├── app/                         # Router, layout, providers
│   ├── features/
│   │   ├── dashboard/               # Summary cards, charts, allocation widgets
│   │   ├── sources/                 # Source list, forms, sync controls
│   │   ├── holdings/                # Holdings tables and manual asset editing
│   │   ├── allocation/              # Macro/liquidity/intra-category screens
│   │   ├── rebalancing/             # Suggestion views and what-if display
│   │   ├── snapshots/               # Historical trend views
│   │   └── settings/                # Base currency, targets, policies
│   ├── components/                  # Reusable presentational UI
│   ├── lib/                         # API client, formatting, chart helpers
│   └── main.tsx                     # React bootstrap
└── Dockerfile                       # Frontend container

docker/
├── postgres/                        # Optional init scripts
└── compose/                         # Env-specific compose files later if needed

docker-compose.yml                   # Local stack: frontend, backend, db
```

### Structure Rationale

- **modules/ on the backend:** Organize by business capability, not by Nest artifact type. This keeps pricing, allocation, snapshots, and rebalancing from bleeding into each other.
- **Separate sources/, pricing/, portfolio/, allocation/, policies/, rebalancing/:** These are the real domain seams of this product. They will change at different rates and should remain independently testable.
- **portfolio as a read-focused module:** The UI mostly needs answers, not raw tables. Put read model composition here instead of letting controllers stitch together data.
- **policies as its own module:** Emergency fund protection is business policy, not UI logic and not generic allocation math.
- **Feature folders on the frontend:** Dashboard, allocation, and rebalancing screens can share primitives while keeping page-level logic local.
- **Single backend application:** For a local single-user tool, one modular monolith is the right complexity level. Separate modules, shared process.

## Architectural Patterns

### Pattern 1: Modular Monolith with Domain-Centric Modules

**What:** One NestJS application split into feature modules aligned to domain capabilities, each exposing clear services and DTOs.
**When to use:** Best fit for this product in 2025 because deployment is local-first, single-user, and operational simplicity matters more than horizontal scale.
**Trade-offs:** Easier debugging, transactions, and refactoring than microservices; requires discipline so modules do not bypass boundaries through shared repositories.

**Example:**
```typescript
// allocation/allocation.service.ts
@Injectable()
export class AllocationService {
  constructor(
    private readonly portfolioReadService: PortfolioReadService,
    private readonly policyService: PolicyService,
  ) {}

  async getAllocationTree(): Promise<AllocationTreeDto> {
    const positions = await this.portfolioReadService.getValuedPositions();
    return this.policyService.applyProtectedRules(buildAllocationTree(positions));
  }
}
```

### Pattern 2: Adapter Boundary for External and Manual Sources

**What:** Treat CCXT exchanges, manual cash accounts, manual broker positions, and price providers as adapters behind stable application interfaces.
**When to use:** Always. External APIs and manual-entry workflows have different failure modes, but they should feed the same canonical portfolio model.
**Trade-offs:** Slightly more code up front, but it prevents CCXT-specific assumptions from contaminating the rest of the app.

**Example:**
```typescript
export interface BalanceSourceAdapter {
  supports(source: SourceConfig): boolean;
  sync(source: SourceConfig): Promise<NormalizedPositionInput[]>;
}

export interface MarketPriceProvider {
  getQuote(symbol: string, quoteCurrency: string): Promise<MoneyQuote>;
}
```

### Pattern 3: Derived Read Models over Authoritative Core Tables

**What:** Keep source data, normalized positions, and configuration in authoritative tables, then build dashboard-friendly projections with queries or views.
**When to use:** Use from day one because the UI needs aggregate answers such as net worth, allocation trees, Trading Bot rollups, and emergency-fund status.
**Trade-offs:** More deliberate schema design, but avoids duplicating business logic in the frontend and reduces expensive recomputation spread across routes.

**Example:**
```typescript
// portfolio/portfolio-read.service.ts
async getDashboard(): Promise<DashboardDto> {
  const positions = await this.repo.getCurrentValuedPositions();
  const allocation = buildAllocationTree(positions);
  const emergencyFund = evaluateEmergencyFund(allocation, this.settings.getPolicy());

  return {
    netWorthBase: sumMoney(positions.map(p => p.baseValue)),
    allocation,
    emergencyFund,
  };
}
```

### Pattern 4: Append-Only Snapshot History

**What:** Persist periodic snapshots of already-computed portfolio state instead of reconstructing all history from raw events.
**When to use:** Ideal for v1 because the product needs basic trends, not full accounting-grade reconstruction.
**Trade-offs:** Cheap to implement and good enough for trend charts; not suitable for detailed performance attribution or tax lots later.

## Data Flow

### Request Flow

```text
[User opens dashboard]
    ↓
[React Dashboard Page]
    ↓
[GET /portfolio/dashboard]
    ↓
[Portfolio Controller]
    ↓
[Portfolio Read Service]
    ↓
[Read repositories / SQL views]
    ↓
[PostgreSQL]
    ↓
[Dashboard DTO with net worth, allocations, policy status, suggestions summary]
    ↓
[React renders cards, charts, and tables]
```

### State Management

```text
[Server state: dashboard/sources/history]
    ↓ (fetched/cached)
[Feature pages and widgets]
    ↔
[UI state: filters, selected category, date range, expanded tree rows]
    ↓
[Derived selectors for charts/tables]
```

Use server state for portfolio data and keep frontend state minimal: filters, selections, dialogs, draft forms. Do not duplicate computed net worth, allocation percentages, or emergency-fund status in client state when the backend can return canonical values.

### Key Data Flows

1. **Ingestion to normalized holdings**
   ```text
   Manual refresh or scheduler
       ↓
   Source Sync Module
       ↓
   Source registry selects adapter per source
       ↓
   CCXT fetchBalance / manual source read
       ↓
   Normalize into canonical positions/accounts
       ↓
   Transactional upsert into source-state tables
       ↓
   Sync result persisted with timestamps/errors
   ```

2. **Valuation to base currency**
   ```text
   Current normalized positions
       ↓
   Determine which positions already have direct values
       ↓
   For manual market-traded holdings, request market prices
       ↓
   For non-base currencies, request FX rates
       ↓
   Compute base-currency value per position
       ↓
   Persist/update valued-position read model
   ```

3. **Allocation tree construction**
   ```text
   Valued positions
       ↓
   Categorization rules assign macro category
       ↓
   Active trading CCXT accounts grouped into Trading Bot
       ↓
   Liquidity split into current/deposit/emergency fund
       ↓
   Intra-category asset weights computed
       ↓
   Allocation engine returns hierarchical tree + deviations vs targets
   ```

4. **Protected emergency fund evaluation**
   ```text
   Allocation tree + emergency fund policy
       ↓
   Policy module evaluates target value or target percentage
       ↓
   Mark underfunded / on-target / overfunded state
       ↓
   Expose warnings and reserve treatment to rebalancing module
   ```

5. **Suggestion generation**
   ```text
   Current valued positions + targets + policy outputs
       ↓
   Exclude protected emergency-fund amount from movable capital
       ↓
   Compare actual vs target at macro/liquidity/intra-category levels
       ↓
   Produce ordered suggestion list with rationale
       ↓
   Return suggestions only; no execution side effects
   ```

6. **Snapshot capture and dashboard history**
   ```text
   Successful valuation/allocation run
       ↓
   Snapshot module writes timestamped summary rows
       ↓
   Dashboard history endpoints query snapshots
       ↓
   React renders trend charts for net worth and allocation drift
   ```

## Scaling Considerations

| Scale | Architecture Adjustments |
|-------|--------------------------|
| 0-1 user, local machine | Single Nest app, single React app, one PostgreSQL instance, synchronous API requests plus simple scheduled jobs. This is the target design. |
| Power-user dataset growth | Add indexes, cache loaded markets/provider metadata in-process, move expensive dashboard queries into SQL views, and run sync/snapshot jobs in background within the same app. |
| Multi-user or SaaS later | Add auth, tenant boundaries, per-user source isolation, job queue, stronger secrets handling, and possibly split ingestion from read APIs. Do not build this now. |

### Scaling Priorities

1. **First bottleneck: external sync latency and provider limits** — CCXT and price providers will be slower and less reliable than local SQL. Fix with adapter reuse, per-provider pacing, cached metadata, and background sync status, not microservices.
2. **Second bottleneck: aggregate dashboard queries** — if allocations and suggestions become slow, add read-optimized views/indexes or precomputed current-state tables before introducing new infrastructure.

For a single-user local app, the main scaling problem is correctness and responsiveness, not concurrency. Optimize for recoverable sync, explicit status, and fast reads.

## Anti-Patterns

### Anti-Pattern 1: One giant portfolio service

**What people do:** Put sync, valuation, allocation, policy, and dashboard assembly into one service because the app is “small.”
**Why it's wrong:** Domain rules become tangled, tests become brittle, and a small rules change risks breaking unrelated flows.
**Do this instead:** Keep a modular monolith. Separate ingestion, pricing, allocation, policy, snapshots, and read APIs.

### Anti-Pattern 2: Let external schemas leak inward

**What people do:** Store CCXT response shapes or exchange-specific semantics as the core model.
**Why it's wrong:** The entire app becomes coupled to one library’s quirks, and manual sources stop fitting cleanly.
**Do this instead:** Normalize all inputs into canonical source, account, holding, and valuation concepts at the adapter boundary.

### Anti-Pattern 3: Compute portfolio truth in the frontend

**What people do:** Fetch raw balances and let React derive net worth, policy status, and allocation logic independently.
**Why it's wrong:** You get inconsistent numbers across screens, duplicated math, and hard-to-audit financial behavior.
**Do this instead:** Compute canonical valuations and allocation results in backend domain services; let the frontend focus on presentation and local interaction state.

### Anti-Pattern 4: Treat emergency fund as ordinary cash

**What people do:** Model emergency cash as just another liquidity bucket and only highlight it in charts.
**Why it's wrong:** Rebalancing suggestions can accidentally recommend consuming protected reserves.
**Do this instead:** Make emergency fund treatment an explicit policy boundary enforced before suggestions are generated.

### Anti-Pattern 5: Store only final aggregates

**What people do:** Save just total net worth and pie-chart percentages after each sync.
**Why it's wrong:** You lose traceability, cannot recalculate with improved rules, and cannot explain why a suggestion changed.
**Do this instead:** Store canonical current positions plus snapshots of computed summaries.

### Anti-Pattern 6: Premature event-driven/distributed architecture

**What people do:** Add queues, event buses, separate workers, and multiple deployables because portfolio apps sound data-heavy.
**Why it's wrong:** It adds operational complexity disproportionate to a single-user local app.
**Do this instead:** Start with transactional module calls inside one backend. Introduce background jobs inside the same app only when sync latency becomes annoying.

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| CCXT-supported exchanges | Adapter per exchange source using reused CCXT instance and normalized outputs | Reuse client instances, load markets once, respect rate limits, and isolate exchange-specific quirks inside adapter layer |
| Market price provider for manual traded assets | Provider adapter behind `MarketPriceProvider` interface | Keep swappable because symbol coverage and rate limits vary |
| FX rate provider | Provider adapter behind `FxRateProvider` interface | Needed whenever source or quote currency differs from base currency |
| Docker Compose services | Internal service-name networking between frontend, backend, and db | Use service names for app-to-db communication and persistent volumes for PostgreSQL data |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| sources ↔ pricing | API/service call | Source sync should produce canonical positions; pricing should not know CCXT details |
| pricing ↔ portfolio | API/service call + repository | Portfolio reads consume valued positions, not provider-specific responses |
| portfolio ↔ allocation | Pure DTO/domain calls | Allocation should depend on normalized valued positions only |
| allocation ↔ policies | Pure DTO/domain calls | Emergency fund protection modifies interpretation of allocations, not raw storage |
| policies ↔ rebalancing | Pure DTO/domain calls | Suggestions must consume protected-capital rules before generating moves |
| snapshots ↔ portfolio/allocation | API/service call | Snapshot write occurs after a successful current-state computation |
| frontend ↔ backend | REST/JSON | Keep API coarse-grained for dashboard/read screens; avoid chatty per-widget endpoints |

## Sources

- Project requirements and constraints: `C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/PROJECT.md`
- NestJS official docs and README: https://docs.nestjs.com/modules , https://docs.nestjs.com/providers , https://github.com/nestjs/nest
- React official docs, Thinking in React: https://react.dev/learn/thinking-in-react
- PostgreSQL official documentation index: https://www.postgresql.org/docs/current/index.html
- Docker Compose documentation: https://docs.docker.com/compose/ , https://docs.docker.com/compose/how-tos/networking/
- CCXT manual: https://raw.githubusercontent.com/ccxt/ccxt/master/wiki/Manual.md

---
*Architecture research for: single-user personal portfolio management web app*
*Researched: 2026-04-20*
