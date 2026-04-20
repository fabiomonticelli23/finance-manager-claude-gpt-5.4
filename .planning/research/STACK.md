# Stack Research

**Domain:** Single-user personal portfolio management web app
**Researched:** 2026-04-20
**Confidence:** MEDIUM-HIGH

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| NestJS | 11.1.19 | Backend API, scheduled sync orchestration, domain services | This is the required backend and still the standard opinionated Node backend for structured TypeScript services. It fits this product because portfolio aggregation, pricing, snapshots, and rebalancing rules benefit from strong module boundaries, DI, and predictable testing. |
| React | 19.2.5 | Frontend dashboard UI | This is the required frontend and the standard React baseline in 2025/2026. For a local-first finance dashboard, React’s ecosystem is the real advantage: forms, charts, tables, routing, and cache/state tooling are all mature. |
| PostgreSQL | 18 | Primary relational database | This is the required database and the right fit for finance data because you want relational integrity, transactions, precise numeric columns, and easy historical snapshot queries. Avoid document stores here. |
| Vite | 8.0.9 | React app build tool and dev server | For a greenfield React SPA, Vite is the standard fast default. It keeps the frontend simple, especially for a local Docker workflow where startup speed and low config matter. |
| Prisma ORM | 7.7.0 | Database ORM, schema, client generation, migrations | Given NestJS + PostgreSQL + finance rules, Prisma is the best default unless you explicitly want SQL-first control. It gives a stable schema workflow, typed client, and standard migration path. Most importantly, Prisma Decimal uses Decimal.js rather than raw JS numbers, which is a better default for money-sensitive code. |

### Supporting Libraries

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| @prisma/client | 7.7.0 | Typed database client | Always. Use for all persistence and querying from NestJS services. |
| pg | 8.20.0 | PostgreSQL driver | Required under Prisma and useful for low-level DB tooling if needed. |
| ccxt | 4.5.50 | Exchange integration for crypto balances and tickers | Use for exchange account sync, balances, symbols, and exchange metadata. Keep it behind an adapter layer because exchange quirks are real. |
| @nestjs/schedule | 6.1.3 | Cron/interval/timeout scheduling in NestJS | Use for local scheduled sync jobs, snapshot generation, and price refresh. This is enough for a single-instance local app. |
| zod | 4.3.6 | Runtime validation and schema definitions | Use for request/response boundaries, config validation, DTO parsing, and frontend form schemas. Prefer one validation language across frontend and backend where possible. |
| nestjs-zod | 5.3.0 | Zod integration for NestJS | Use if you want Zod-first DTOs in Nest rather than class-validator decorators. Good fit for keeping validation logic consistent across backend and React forms. |
| react-hook-form | 7.72.1 | Form state management | Use for manual asset entry, target allocation rules, settings, and exchange credential forms. It is the standard low-rerender choice for serious forms. |
| @hookform/resolvers | 5.2.2 | Connect RHF to Zod | Use whenever forms should validate against shared Zod schemas. |
| @tanstack/react-query | 5.99.2 | Server-state fetching, caching, retries, refetching | Use for all API-backed data: portfolio summary, snapshots, sync status, prices, and settings. This is standard React app infrastructure in 2025. |
| react-router-dom | 7.14.1 | SPA routing | Use for dashboard, accounts, holdings, settings, and history routes. Stick to simple SPA routing rather than introducing a heavier framework. |
| zustand | 5.0.12 | Lightweight client state | Use only for small client-only state such as selected base currency UI state, chart filters, panel preferences, and optimistic local draft state. Do not use it as a server-state cache. |
| @tanstack/react-table | 8.21.3 | Portfolio/account/holding tables | Use for sortable, filterable holdings tables and rebalancing views. Better fit than building custom table logic. |
| recharts | 3.3.0 | Dashboard charts | Recommended default for pie, bar, area, and stacked allocation charts in an admin/dashboard-style product. It is easier to work with than lower-level charting systems and is sufficient for this product’s chart complexity. |
| date-fns | 4.1.0 | Date manipulation | Use for snapshot labeling, scheduling displays, rolling windows, and timezone-safe formatting helpers. Keep date logic explicit and utility-based. |
| decimal.js | 10.6.0 | Exact decimal arithmetic | Use in domain logic whenever money, percentages, FX conversion, or allocation calculations leave the database layer. Never use native JS floating point for financial calculations. |
| yahoo-finance2 | 3.14.0 | Market price ingestion for manual securities | Recommended default source library for stocks, ETFs, and ETPs when you need automated pricing for manual holdings. Good enough for a personal local app, but treat it as best-effort market data rather than institutional-grade sourcing. |
| pino | 10.3.1 | Structured backend logging | Use for backend logs, sync diagnostics, exchange API traces, and job outcomes. Structured logs matter when finance syncs fail silently. |
| nestjs-pino | 4.6.1 | Pino integration for NestJS | Use if you want request-scoped logging and standard Nest integration without custom logger wiring. |
| vitest | 4.1.4 | Unit and component testing | Use as the main frontend test runner and also for shared TypeScript utility packages if you create them. Fast and aligned with Vite. |
| @playwright/test | 1.59.1 | End-to-end browser testing | Use for critical flows: create assets, run sync, inspect allocation, verify rebalancing suggestions, verify emergency fund warnings. |
| supertest | 7.2.2 | HTTP API integration tests | Use with NestJS testing module for backend endpoint coverage. |
| testcontainers | 11.14.0 | Real-container integration tests | Use for PostgreSQL-backed integration tests so migrations and numeric behavior are tested against a real DB, not mocks. |
| tsx | 4.21.0 | Run TypeScript scripts directly | Use for local scripts such as seeding, backfills, one-off imports, and maintenance jobs. |
| eslint | 9.38.0 | Linting | Use the flat config format. Keep consistency strict because finance dashboards accumulate edge-case conditionals fast. |
| prettier | 3.8.3 | Formatting | Use to keep backend/frontend/shared code friction low. |
| pino-pretty | 13.1.3 | Dev log readability | Use only in local development, not as your stored log format. |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| Docker Engine + Docker Compose | Local-first multi-container deployment | Use `docker compose`, one service each for backend, frontend, and postgres, plus named volumes. This matches the required deployment style and reduces “works on my machine” drift. |
| Prisma Migrate | Schema migrations | Standard Prisma migration path. Use checked-in migrations and never rely on `db push` for production-like workflows. |
| Playwright UI mode | Fast e2e debugging | Very useful for verifying dashboard behavior and chart-driven UI regressions locally. |
| Testcontainers | Database-realistic testing | Especially valuable for PostgreSQL decimal columns, timestamp handling, and migration verification. |
| Docker healthchecks | Startup coordination | Add healthchecks for backend/frontend/postgres so local startup is robust. This matters more than usual for scheduled-job apps. |

## Prescriptive Stack by Area

### Backend support libraries

Use:
- `@nestjs/config` for environment/config management
- `@nestjs/schedule` for cron-like refresh jobs
- `pino` + `nestjs-pino` for structured logs
- `zod` + `nestjs-zod` for validation at system boundaries
- `ccxt` behind exchange-specific adapters
- `decimal.js` in domain services for all financial math

Why this fits:
- This app is single-instance and local-first, so in-process scheduling is appropriate.
- It is finance-like enough that validation and decimal safety are not optional.
- Exchange sync will fail in messy ways; structured logs are much more valuable than pretty console output.

### React app choices

Use:
- React + Vite SPA
- `react-router-dom` for routing
- `@tanstack/react-query` for all API/server state
- `react-hook-form` + `zod` for forms
- `zustand` only for small client-only state
- `@tanstack/react-table` for holdings and allocation tables
- `recharts` for dashboard charts

Why this fits:
- The product is dashboard-centric, not marketing-site or content-site heavy.
- You do not need Next.js complexity for a local-first single-user app.
- React Query cleanly models refreshable backend data, stale views, retries, and manual refetch.

### ORM and migrations

Use:
- Prisma schema as the single source of truth
- PostgreSQL `numeric`/decimal columns for monetary amounts and percentages where precision matters
- Prisma Migrate for all schema evolution

Why this fits:
- The project has clear relational entities: accounts, assets, holdings, FX rates, snapshots, targets, and sync runs.
- Prisma is more productive than SQL-heavy alternatives for this scope, while still handling PostgreSQL well.
- Decimal support is a deciding factor here.

Recommended modeling notes:
- Store money values as decimal columns plus explicit currency codes.
- Store quantities separately from prices and valuations.
- Store snapshots as denormalized facts for historical queries, not as recomputed-on-read only.
- Store source provenance on imported prices and balances.

### Scheduling and jobs

Use:
- `@nestjs/schedule` for recurring syncs and snapshots
- Plain DB-backed sync-run records for idempotency, auditability, and retry state

Do not start with:
- BullMQ/Redis for v1

Why:
- BullMQ requires Redis. That adds another container and another failure mode for a local-first product that does not need distributed workers yet.
- For one user and one backend instance, scheduled Nest jobs plus careful DB state is the standard simpler choice.

Promote to Redis/BullMQ only if:
- You add long-running imports with concurrency control
- You need durable retries across many independent job types
- You move beyond single-instance execution

### Charting

Recommended default: `recharts@3.3.0`

Use it for:
- net worth over time
- macro allocation pie/donut charts
- liquidity breakdown bars
- target vs actual allocation bars
- emergency fund status visualization

Alternative:
- `echarts@6.0.0` + `echarts-for-react@3.0.6` when you need denser interactive analytics or much larger timeseries

Why Recharts first:
- Faster implementation for dashboard charts
- Simpler React mental model
- Enough for this product’s first two phases

### Data validation

Recommended default: Zod-first stack.

Use:
- `zod` for shared schemas
- `nestjs-zod` on backend DTO boundaries
- `react-hook-form` + `@hookform/resolvers/zod` on frontend forms

Avoid mixed validation stacks unless needed.

Why:
- Shared schemas reduce drift between frontend and backend.
- This app has many “small but important” validations: currency codes, percentages, macro category enums, emergency fund rule modes, account source types, manual security tickers.

### Market-price ingestion for manual securities

Recommended default: `yahoo-finance2@3.14.0`

Use it for:
- stocks
- ETFs
- many ETP-like exchange-traded products
- simple quote/history retrieval for manual holdings

Important caveats:
- It is not an official exchange or brokerage feed.
- Symbol coverage and market suffix behavior vary by exchange.
- Historical consistency and data licensing assumptions should not be treated as institutional-grade.
- Build a price-source abstraction now so you can swap to Twelve Data, Alpha Vantage, Polygon, or broker-specific feeds later.

For v1 this is still the right choice because:
- Local-first personal portfolio software values convenience and breadth over enterprise SLAs.
- You need broad retail securities coverage without adding paid infrastructure immediately.

### Date and money handling

Use:
- `date-fns` for explicit date utilities
- UTC storage in PostgreSQL for event timestamps
- user-display timezone handling in frontend formatting only
- `decimal.js` for arithmetic outside SQL/Prisma

Rules:
- Never use JS `number` for money math.
- Never mix “asset quantity” and “base-currency value” in one column.
- Never compute allocation percentages with floating point and then persist the rounded result as truth.

### Testing

Use:
- Vitest for frontend unit tests and utility/domain tests
- Nest testing module + Supertest for API integration tests
- Playwright for end-to-end flows
- Testcontainers for PostgreSQL integration tests and migration verification

What to prioritize testing in this product:
- FX conversion correctness
- emergency fund protection logic
- trading-bot account grouping behavior
- snapshot generation correctness
- idempotent sync behavior
- rebalancing suggestion math and threshold handling

### Docker and developer tooling

Use:
- separate backend/frontend/db services under `docker compose`
- named PostgreSQL volume
- backend and frontend multi-stage Dockerfiles
- healthchecks for DB and backend
- `.env` + `.env.example` discipline
- one-command local startup

Compatibility notes for local-first finance app:
- Scheduled jobs can start before the DB is ready unless you add healthchecks and startup guards.
- Hot reload inside Docker on Windows can be slower; mount only what is needed and prefer polling configs only if file watching fails.
- Persist PostgreSQL data in a named volume so snapshot history survives container recreation.

## Installation

```bash
# Backend core
npm install @nestjs/common@11.1.19 @nestjs/core@11.1.19 @nestjs/config @nestjs/schedule@6.1.3 prisma@7.7.0 @prisma/client@7.7.0 pg@8.20.0 ccxt@4.5.50 zod@4.3.6 nestjs-zod@5.3.0 decimal.js@10.6.0 pino@10.3.1 nestjs-pino@4.6.1 date-fns@4.1.0

# Frontend core
npm install react@19.2.5 react-dom@19.2.5 react-router-dom@7.14.1 @tanstack/react-query@5.99.2 react-hook-form@7.72.1 @hookform/resolvers@5.2.2 zustand@5.0.12 @tanstack/react-table@8.21.3 recharts@3.3.0 zod@4.3.6 date-fns@4.1.0

# Market data for manual securities
npm install yahoo-finance2@3.14.0

# Dev dependencies
npm install -D vite@8.0.9 typescript@5.9.3 vitest@4.1.4 @playwright/test@1.59.1 supertest@7.2.2 testcontainers@11.14.0 eslint@9.38.0 prettier@3.8.3 @typescript-eslint/parser@8.58.2 @typescript-eslint/eslint-plugin@8.58.2 pino-pretty@13.1.3 tsx@4.21.0
```

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Prisma | Drizzle ORM | Choose Drizzle if you explicitly want SQL-first control, lighter abstractions, and are comfortable owning more query/migration detail. For this app, Prisma is the better default because finance logic is already complex enough without adding ORM ceremony. |
| React + Vite SPA | Next.js App Router | Choose Next.js only if you later need SSR, server actions, public pages, or remote deployment patterns. For a private local dashboard, it adds more moving pieces than value. |
| @nestjs/schedule | BullMQ + Redis | Choose BullMQ only if job orchestration becomes a product area of its own or you need durable distributed queues. Not needed for a single-user local app v1. |
| Recharts | ECharts | Choose ECharts if you need highly interactive dense financial visualizations or large timeseries with richer tooling. Otherwise Recharts is faster to ship and easier to maintain. |
| yahoo-finance2 | Paid market data APIs | Choose a paid source if quote coverage, SLA, licensing clarity, or exchange-specific accuracy becomes critical. v1 can start with Yahoo-backed convenience behind an abstraction. |
| Zod-first validation | class-validator + class-transformer | Choose decorator-based validation only if the team strongly prefers Nest idioms and does not care about shared frontend/backend schemas. For this app, shared schemas are more valuable. |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| TypeORM | It is no longer the strongest default for greenfield TypeScript apps. For this product, Prisma or Drizzle are clearer choices with better current mindshare and less incidental complexity. | Prisma |
| class-validator + class-transformer as the primary validation stack | Works, but duplicates schema logic and weakens frontend/backend sharing. This app benefits from one schema language across the stack. | Zod + nestjs-zod + RHF resolvers |
| BullMQ for v1 job scheduling | Requires Redis and adds operational complexity that is hard to justify in a local-first single-instance app. | @nestjs/schedule + DB-backed sync records |
| Chart.js as the default dashboard chart layer | Good library, but less ergonomic than Recharts for React dashboard composition and custom finance UI layouts. | Recharts |
| Native JS number for money, FX, and allocation math | Floating-point errors will eventually surface in rebalancing suggestions and snapshot totals. | PostgreSQL numeric + Prisma Decimal + decimal.js |
| Global client state stores for server data | Using Zustand/Redux as the main API cache makes refresh, staleness, retries, and invalidation harder than necessary. | TanStack Query |
| Next.js for this v1 by default | It is not the standard best fit for a private local dashboard with a separate Nest backend already mandated. | React + Vite SPA |
| Redis as mandatory infrastructure from day one | It complicates Docker, startup ordering, and backup/recovery for limited product value in this scope. | PostgreSQL + in-process scheduling |

## Stack Patterns by Variant

**If you keep the app strictly local and single-machine:**
- Use Nest scheduled jobs only
- Use Prisma + PostgreSQL as the single persistence backbone
- Keep market data ingestion best-effort with source provenance recorded
- Because operational simplicity matters more than distributed scalability

**If you later expand into remote hosting or multiple users:**
- Introduce auth/session management
- Consider BullMQ + Redis for heavier job isolation
- Reassess charting if analytics depth grows
- Because concurrency, durability, and observability requirements will rise sharply

**If manual securities become a bigger part of the product than crypto:**
- Strengthen the market-data abstraction immediately
- Add symbol normalization and exchange/suffix mapping tables
- Consider premium quote providers
- Because retail ticker symbols are messy and Yahoo-style convenience data becomes a support burden

## Version Compatibility

| Package A | Compatible With | Notes |
|-----------|-----------------|-------|
| NestJS 11.1.19 | @nestjs/schedule 6.1.3 | Good current Nest ecosystem pairing. Verify peer ranges in package.json when installing. |
| React 19.2.5 | react-dom 19.2.5 | Keep versions matched exactly. |
| React 19.2.5 | TanStack Query 5.99.2 | Standard current pairing. |
| React 19.2.5 | React Router 7.14.1 | Standard current pairing for SPA routing. |
| React 19.2.5 | Recharts 3.3.0 | Common dashboard stack, but visually verify charts after major React upgrades. |
| Prisma 7.7.0 | PostgreSQL 18 | Prisma docs list PostgreSQL support through version 18. |
| Prisma 7.7.0 | decimal.js via Prisma Decimal | Prisma Decimal values are represented using Decimal.js concepts rather than native numbers. |
| Vite 8.0.9 | Vitest 4.1.4 | Natural pairing in modern React projects. |
| Testcontainers 11.14.0 | Docker Desktop / Engine | Integration tests depend on container runtime availability; local setup docs should call this out clearly. |
| BullMQ 5.75.2 | Redis | Not compatible without Redis; this is why it is not recommended for v1. |

## Product-Specific Trouble Spots

1. **CCXT symbol and balance normalization will be messier than expected.**
   - Exchanges disagree on asset symbols, account types, and sometimes valuation semantics.
   - Normalize assets into your own canonical asset table early.

2. **Manual securities pricing is not the same problem as crypto pricing.**
   - Crypto pairs often have direct exchange quotes.
   - Stocks/ETFs/ETPs often require symbol mapping, market suffix handling, and a different data-source abstraction.

3. **Emergency fund logic should be modeled as policy, not just a tag.**
   - You need target mode, protected status, under/over thresholds, and exclusion logic for some rebalancing outputs.

4. **Local-first does not mean “ignore reliability.”**
   - Scheduled jobs, stale prices, and partial sync failures need visible status in the UI.
   - Add sync-run records and “last successful refresh” indicators from the start.

5. **Docker on Windows can surface file-watch and networking quirks.**
   - Keep the dev setup boring.
   - Prefer standard ports, named volumes, and explicit healthchecks.

## Sources

- https://docs.nestjs.com — official NestJS docs consulted for framework baseline; version not clearly exposed in fetched content
- npm registry — verified current package versions for `@nestjs/core` 11.1.19, `@nestjs/schedule` 6.1.3, `ccxt` 4.5.50, `pino` 10.3.1, `nestjs-pino` 4.6.1, `zod` 4.3.6, `react-hook-form` 7.72.1, `@tanstack/react-query` 5.99.2, `react-router-dom` 7.14.1, `zustand` 5.0.12, `recharts` 3.3.0, `echarts` 6.0.0, `prisma` 7.7.0, `drizzle-orm` 0.45.2, `pg` 8.20.0, `date-fns` 4.1.0, `decimal.js` 10.6.0, `vitest` 4.1.4, `@playwright/test` 1.59.1, `supertest` 7.2.2, `testcontainers` 11.14.0, `eslint` 9.38.0, `prettier` 3.8.3, `tsx` 4.21.0, `yahoo-finance2` 3.14.0
- https://react.dev — official React docs; version verified against npm registry because fetched content did not expose it clearly
- https://reactrouter.com/home — verified React Router v7 line and positioning as a standard React router
- https://tanstack.com/query/latest — verified Query v5 line from docs
- https://www.postgresql.org/docs/ — official PostgreSQL docs showing current version 18
- https://www.prisma.io/docs/orm/overview/databases/postgresql — verified Prisma PostgreSQL support through version 18
- https://www.prisma.io/docs/orm/prisma-client/special-fields-and-types#working-with-decimal — verified Prisma Decimal handling via Decimal.js
- https://orm.drizzle.team/docs/overview — consulted Drizzle positioning; version verification relied on npm registry because docs page emphasized RC/beta materials
- https://www.chartjs.org/docs/latest/ — consulted for Chart.js positioning as alternative, not recommendation
- https://docs.bullmq.io/ — verified BullMQ purpose and Redis requirement
- https://testcontainers.com/modules/postgresql/ — verified Testcontainers PostgreSQL support exists, with community module note on fetched page
- https://playwright.dev/docs/intro — official positioning for Playwright e2e usage; version verified against npm registry
- https://vitest.dev/guide/ — verified Vitest v4 positioning
- https://www.npmjs.com/package/yahoo-finance2 — attempted for package details; fetched page was blocked, so package version was verified from npm registry and recommendation confidence is MEDIUM

---
*Stack research for: single-user personal portfolio management web app*
*Researched: 2026-04-20*