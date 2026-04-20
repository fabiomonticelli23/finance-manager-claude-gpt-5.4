# Stack Research

**Domain:** Self-hosted personal portfolio management web app with crypto exchange aggregation, manual assets, multi-currency analytics, and rebalancing guidance
**Researched:** 2026-04-20
**Confidence:** HIGH

## Recommended Stack

This project already has three fixed constraints from `PROJECT.md`: NestJS backend, React frontend, and PostgreSQL database. For a 2025 standard stack in this problem space, that is still the right foundation.

The opinionated recommendation is:
- **NestJS 11 + TypeScript 6** for a structured backend with scheduled sync jobs, provider-based integrations, and clean domain boundaries
- **React 19 + React Router 7 + TanStack Query 5** for a SPA dashboard that is local-first, fast to iterate on, and good at API-heavy data screens
- **PostgreSQL 18 + Prisma 7** for relational portfolio/account/price/snapshot data with strong consistency and migrations
- **CCXT 4** for exchange integrations, but only as an ingestion adapter layer, never as your core domain model
- **Decimal math, schema validation, and explicit FX/price snapshot tables** to avoid the finance-specific errors that destroy trust

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| NestJS | 11.1.19 | Backend API, sync orchestration, domain services | Best fit for this app because portfolio management has many distinct concerns: exchange sync, asset valuation, FX conversion, target allocation logic, and snapshot generation. NestJS gives modules, DI, schedulers, and strong TypeScript ergonomics without inventing your own structure. Confidence: HIGH |
| React | 19.2.5 | Frontend dashboard and data-entry UI | Standard choice for a personal analytics dashboard. React is mature, has the strongest ecosystem for forms/charts/data-heavy UIs, and works well with a separate NestJS API. Confidence: HIGH |
| TypeScript | 6.0.3 | End-to-end type safety | Mandatory for this domain. Financial apps suffer from subtle shape mismatches around currencies, asset types, and sync payloads. TypeScript reduces those failures across backend, frontend, and shared DTO/schema code. Confidence: HIGH |
| PostgreSQL | 18.3 | System-of-record database | Best default relational database for portfolios because you need transactions, constraints, joins, snapshots, auditability, and queryable historical data. This is not a document-store-first problem. Confidence: HIGH |
| Prisma ORM | 7.7.0 | Database schema, migrations, typed data access | Recommended because this app has a conventional relational model and benefits from fast iteration, readable migrations, and generated type-safe queries. It speeds up greenfield delivery more than lower-level SQL tooling. Confidence: HIGH |
| Docker Compose | Current Compose plugin/docs | Local multi-container orchestration | Matches the explicit deployment constraint. Compose is the standard way to run backend, frontend, DB, and optional worker/pgAdmin containers together in a self-hosted local-first setup. Confidence: HIGH |

### Supporting Libraries

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| CCXT | 4.5.49 | Exchange API abstraction for balances, markets, and ticker/price ingestion | Use for exchange connectors only. Wrap each exchange behind your own adapter interface so rate limits, symbol normalization, and account mapping stay under your control. Confidence: MEDIUM-HIGH |
| @tanstack/react-query | 5.99.2 | API caching, refetching, loading/error states | Use for all server-state in the React app: portfolio overview, account lists, sync status, prices, and historical snapshots. It is the standard solution for dashboard-style apps with frequent reads and occasional mutations. Confidence: HIGH |
| React Router | 7.14.1 | Client-side routing | Use for a classic SPA structure: dashboard, accounts, assets, targets, rebalancing, settings, sync logs. React Router 7 is the straightforward routing choice when Next.js-style full-stack features are not required. Confidence: HIGH |
| Recharts | 3.8.1 | Pie charts, bar charts, time-series visualization | Use for v1 dashboards because it is easy to ship with and sufficient for allocation charts, net worth trend charts, and drift visuals. If later you need very large datasets or highly customized charting, revisit. Confidence: MEDIUM-HIGH |
| Zod | 4.3.6 | Runtime schema validation | Use at all trust boundaries: environment variables, API payloads, manual asset forms, and normalized exchange payloads before domain processing. Finance apps fail silently when bad payloads sneak through. Confidence: HIGH |
| react-hook-form | 7.72.1 | Form state for manual assets, targets, settings | Use for manual asset entry, rebalancing target configuration, and protected emergency-fund rules. It pairs well with Zod and avoids overengineering local form state. Confidence: HIGH |
| @hookform/resolvers | 5.2.2 | Zod integration for forms | Use whenever React Hook Form validates with Zod schemas. Confidence: HIGH |
| decimal.js | 10.6.0 | Precise financial arithmetic | Use for all money, quantity, FX, and percentage calculations. JavaScript `number` is not acceptable for portfolio math that users must trust. Confidence: HIGH |
| date-fns | 4.1.0 | Date handling for snapshots and analytics windows | Use for snapshot intervals, rebalance timing, and report ranges. It is lightweight and sufficient unless you later need timezone-heavy scheduling across many regions. Confidence: MEDIUM-HIGH |
| @nestjs/schedule | 6.1.3 | Scheduled exchange sync jobs | Use for cron-like sync scheduling in a single-node self-hosted deployment. It is enough for v1 and simpler than introducing a full queue system too early. Confidence: HIGH |
| @nestjs/config | 4.0.4 | Environment configuration | Use for typed config loading for DB, exchange credentials, base currency, FX providers, and sync intervals. Confidence: HIGH |
| pino | 10.3.1 | Structured backend logging | Use for sync logs, exchange failures, pricing issues, and rebalancing calculation diagnostics. Structured logs matter when financial numbers look wrong and you must trace why. Confidence: HIGH |
| pino-http | 11.0.0 | HTTP request logging | Use to capture API request metadata cleanly in NestJS. Confidence: HIGH |
| class-validator | 0.15.1 | Nest DTO validation for controller inputs | Use if you keep Nest’s class-based DTO flow. If you standardize entirely on Zod at all boundaries, this becomes optional. Confidence: MEDIUM |
| class-transformer | 0.5.1 | DTO transformation in NestJS | Use only alongside class-validator/Nest DTO patterns. Avoid mixing too many validation styles. Confidence: MEDIUM |
| Zustand | 5.0.12 | Small client-only UI state | Use only for ephemeral UI state such as selected display currency, chart preferences, or panel visibility. Do not use it for server state. Confidence: HIGH |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| Vitest 4.1.4 | Unit and integration tests | Good default for frontend and shared TypeScript packages. Fast feedback matters when validating allocation math and FX conversion logic. |
| Playwright 1.59.1 | End-to-end testing | Use for critical flows: onboarding, manual asset entry, exchange sync visibility, and rebalance guidance review. |
| ESLint 10.2.1 | Code quality and consistency | Enforce no implicit any, no unsafe number formatting in money logic, and import hygiene across shared packages. |
| Prettier 3.8.3 | Formatting | Keep config boring and automatic. Portfolio logic is hard enough without style churn. |

## Installation

```bash
# Core frontend + backend runtime
npm install react@19.2.5 react-dom@19.2.5 react-router@7.14.1 @tanstack/react-query@5.99.2 recharts@3.8.1 zod@4.3.6 react-hook-form@7.72.1 @hookform/resolvers@5.2.2 zustand@5.0.12
npm install @nestjs/core@11.1.19 @nestjs/config@4.0.4 @nestjs/schedule@6.1.3 prisma@7.7.0 @prisma/client@7.7.0 ccxt@4.5.49 decimal.js@10.6.0 date-fns@4.1.0 pino@10.3.1 pino-http@11.0.0 class-validator@0.15.1 class-transformer@0.5.1

# Dev dependencies
npm install -D typescript@6.0.3 vitest@4.1.4 @playwright/test@1.59.1 eslint@10.2.1 prettier@3.8.3
```

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Prisma 7 | Drizzle ORM | Use Drizzle if you want tighter SQL control, lighter abstractions, and are comfortable owning more schema/query detail. For this greenfield app, Prisma is faster to stand up and easier for conventional CRUD + analytics reads. |
| React Router 7 SPA | Next.js | Use Next.js only if SEO, public content, SSR, or an integrated full-stack app become important. For a private self-hosted dashboard, Next adds framework surface area you do not need. |
| Recharts 3 | Apache ECharts / Nivo | Use a heavier charting library only if you later need much denser time-series analysis, advanced interactivity, or very customized visualizations. Recharts is simpler for v1. |
| @nestjs/schedule | BullMQ / Temporal | Use a queue/workflow system if you later run many accounts, long-running syncs, retries across multiple workers, or distributed execution. For a single-user self-hosted v1, scheduled jobs are enough. |
| PostgreSQL 18 | TimescaleDB extension | Use Timescale only if historical time-series analysis becomes a major product differentiator with large retained history and heavy rollups. Plain Postgres is the right starting point. |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| MongoDB as primary database | This domain is relational: accounts, holdings, snapshots, targets, FX rates, sync runs, and valuation records have integrity rules and query relationships. Document-first storage makes core reporting and consistency harder, not easier. | PostgreSQL 18 |
| JavaScript `number` for money math | Binary floating point introduces rounding drift in balances, percentages, and rebalance calculations. In a finance app, tiny math errors destroy trust. | decimal.js |
| Redux for main app state | Most state here is server state, not complex client workflow state. Redux adds ceremony and duplicated caching that TanStack Query already solves better. | TanStack Query + local component state/Zustand |
| Next.js by default | This is a private, self-hosted dashboard, not a content site. Next’s SSR/server actions/app router complexity is unnecessary when NestJS is already the backend. | React SPA + React Router |
| Direct CCXT payloads throughout the app | Exchange payloads differ, symbols vary, and exchange metadata changes. If CCXT leaks into your domain model, every integration detail infects the codebase. | Internal adapter layer and normalized portfolio domain models |
| Overbuilding with Kafka, Temporal, or microservices in v1 | Single-user self-hosted portfolio software does not need distributed systems complexity on day one. That complexity slows delivery more than it helps. | Modular monolith with NestJS modules and scheduled jobs |

## Stack Patterns by Variant

**If you stay single-user and local-first, which matches the current project:**
- Use a **modular monolith**: one NestJS backend, one React frontend, one PostgreSQL database
- Keep exchange sync in-process with `@nestjs/schedule`
- Because this minimizes operational complexity and is the standard sensible architecture for self-hosted personal finance software

**If sync volume grows or you later support many exchange accounts and long-running jobs:**
- Add a **separate worker service** that shares the same database and processes sync/repricing jobs
- Introduce a queue only when retries, concurrency limits, and isolation become painful in the web process
- Because exchange IO and rate-limit handling are the first parts likely to need operational separation

**If historical analytics becomes a bigger product pillar:**
- Keep PostgreSQL as source of truth, but add precomputed daily snapshots/materialized views
- Because rebalancing and net-worth trends benefit more from durable derived tables than from premature event-sourcing

## Version Compatibility

| Package A | Compatible With | Notes |
|-----------|-----------------|-------|
| NestJS 11.1.19 | TypeScript 6.0.3 | Strong current TypeScript fit; verify tsconfig defaults from Nest 11 starter before locking lint/test config. |
| React 19.2.5 | React Router 7.14.1 | React Router 7 explicitly targets modern React and is the right router line for current React apps. |
| React 19.2.5 | TanStack Query 5.99.2 | Standard pairing for API-heavy React dashboards. |
| Prisma 7.7.0 | PostgreSQL 18.3 | Good fit for relational schema/migrations; validate generated client and migration workflow in your exact Docker image early. |
| react-hook-form 7.72.1 | Zod 4.3.6 via @hookform/resolvers 5.2.2 | Standard validation stack for complex forms. |
| CCXT 4.5.49 | NestJS 11.1.19 | Works best when wrapped in provider/adapters with retry, throttling, and normalization policies owned by your app. |

## Opinionated Recommendation

If I were starting this app in 2025/2026 under your stated constraints, I would build it as:

- **Backend:** NestJS 11 + Prisma 7 + PostgreSQL 18 + CCXT 4 + decimal.js + @nestjs/schedule + pino
- **Frontend:** React 19 + React Router 7 + TanStack Query 5 + React Hook Form + Zod + Recharts
- **Infra:** Docker Compose for local orchestration, separate containers for frontend, backend, and database

That is the standard stack because it optimizes for the real problems in this domain:
1. trustworthy ingestion from heterogeneous data sources,
2. precise financial calculations,
3. relational analytics over current and historical positions,
4. fast iteration for a single-user self-hosted product.

Do not optimize for internet-scale architecture. Optimize for correctness, inspectability, and the ability to add one more asset type, exchange, or rebalance rule without rewriting the system.

## Sources

- `C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/PROJECT.md` — project constraints and domain goals
- Context7 `/nestjs/nest` — verified NestJS current major/version line availability and framework positioning. Confidence: HIGH
- https://docs.nestjs.com/ — verified NestJS is the official server-side framework source. Confidence: HIGH
- Context7 `/facebook/react` — verified current React version line availability. Confidence: HIGH
- https://react.dev/learn — verified React purpose and official positioning. Confidence: HIGH
- https://www.typescriptlang.org/ — verified TypeScript 6.0 availability and purpose. Confidence: HIGH
- https://www.postgresql.org/docs/ — verified PostgreSQL 18 current docs line and release banner showing 18.3. Confidence: HIGH
- Context7 `/prisma/prisma` and https://www.prisma.io/docs — verified Prisma v7 line and purpose. Confidence: HIGH
- Context7 `/tanstack/query` and https://tanstack.com/query/latest — verified TanStack Query v5 line and purpose. Confidence: HIGH
- https://reactrouter.com/ — verified React Router v7 line and purpose. Confidence: HIGH
- https://recharts.github.io/ — verified Recharts v3.8.1 display and purpose. Confidence: HIGH
- https://zod.dev/ — verified Zod 4 as current stable line. Confidence: HIGH
- https://docs.docker.com/compose/ — verified Docker Compose suitability for multi-container local orchestration. Confidence: HIGH
- https://github.com/ccxt/ccxt — verified CCXT purpose and broad exchange support; rate-limit guidance visibility was limited in fetched excerpt, so operational recommendations are partially inference-backed. Confidence: MEDIUM-HIGH
- npm registry package metadata via `npm view` on 2026-04-20 — verified exact package versions for install recommendations. Confidence: HIGH

---
*Stack research for: self-hosted personal portfolio management app*
*Researched: 2026-04-20*