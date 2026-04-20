# Project Research Summary

**Project:** Finance Manager
**Domain:** Self-hosted personal portfolio management and rebalancing web application
**Researched:** 2026-04-20
**Confidence:** HIGH

## Executive Summary

Finance Manager is best approached as a trust-first portfolio operations tool, not a generic wealth dashboard. The core job is to consolidate CCXT-synced exchange balances and manual assets into one base-currency portfolio model, then turn that normalized view into allocation analytics and advisory-only rebalancing guidance. Research consistently points to a modular monolith: React frontend, NestJS backend, PostgreSQL system of record, exact decimal math, and background sync/pricing pipelines that feed persisted read models rather than live-calculating from provider APIs.

The recommended v1 is opinionated: build the data spine first, ship manual assets and valuation before exchange automation, and treat allocation plus rebalancing as first-class workflows rather than post-processing reports. Table stakes are unified holdings, trustworthy net worth, allocation drilldowns, historical snapshots, manual asset support, data freshness visibility, and target-based drift analysis. The strongest product differentiation is multi-level allocation logic with protected emergency-fund handling and concrete rebalance actions.

The main early risks are correctness and trust failures, not scale. The highest-risk mistakes are weak canonical asset/account modeling, float-based money math, symbol-based pricing identity, hidden mixed-freshness data, and naive rebalance logic that treats protected cash or illiquid capital as investable. Mitigation is consistent across research: exact-decimal storage, canonical IDs, immutable snapshots with provenance, freshness-aware analytics, and deterministic server-side recommendation rules.

## Key Findings

### Recommended Stack

Use a local-first modular monolith built around NestJS 11, React 19, and PostgreSQL 18. This stack fits the problem because the app needs structured domain modules, background scheduling, strong relational modeling, and dashboard-heavy UI flows rather than SEO or distributed systems. Prisma 7 is the fastest path to a reliable relational schema; CCXT 4 is useful only behind an adapter layer; decimal.js and explicit pricing/FX tables are non-negotiable for financial correctness.

**Core technologies:**
- **NestJS 11 + TypeScript 6**: backend modules, validation, scheduling, and domain orchestration — best fit for sync, valuation, allocation, and recommendation boundaries.
- **React 19 + React Router 7 + TanStack Query 5**: SPA dashboard and forms — ideal for data-heavy review workflows and server-state driven screens.
- **PostgreSQL 18 + Prisma 7**: source of truth for accounts, holdings, targets, prices, and snapshots — strong consistency and migrations suit this relational domain.
- **CCXT 4**: exchange ingestion only — use via per-exchange adapters, never as the domain model.
- **decimal.js + Zod**: exact money math and boundary validation — required to avoid silent trust failures.
- **Docker Compose**: local multi-container deployment — matches the self-hosted constraint.

**Critical version requirements:**
- NestJS 11 / TypeScript 6
- React 19 / React Router 7 / TanStack Query 5
- PostgreSQL 18 / Prisma 7
- CCXT 4

### Expected Features

V1 should focus on the recurring review loop: refresh data, trust the totals, inspect drift, and decide next actions. Research is clear that users will consider the product incomplete without both mixed-source aggregation and actionable allocation analysis.

**Must have (table stakes):**
- Unified holdings dashboard across synced and manual accounts
- Base-currency net worth and valuation pipeline
- Asset/account drilldown and allocation analytics
- Manual asset and cash account support
- Historical snapshots and trend charts
- Data freshness and source-status visibility
- Target allocation configuration with emergency-fund protection
- Advisory rebalancing gap analysis and recommended actions

**Should have (competitive):**
- Multi-level target allocation engine across macro, liquidity, and asset layers
- Emergency-fund-aware rebalance logic
- Concrete rebalance actions instead of percentage-only drift output
- Mixed-source portfolio model that cleanly combines manual and synced assets
- Liquidity-aware analytics and a review-oriented workflow

**Defer (v2+):**
- Trade execution
- Tax reporting / tax-lot accounting
- Full performance attribution and returns engine
- Broad connector ecosystem beyond initial CCXT + manual model
- Multi-user / household support

### Architecture Approach

The architecture research strongly recommends a build order centered on a canonical portfolio domain model, persisted valuations, and snapshot-first analytics. Major components should be kept separate by responsibility: source management and ingestion, holdings, pricing/FX, allocation/targets, snapshots, rebalancing, and dashboard read models. Reads should be synchronous against persisted state; sync and pricing should happen asynchronously in scheduled jobs or workers. Rebalancing must stay server-side and deterministic.

**Major components:**
1. **Sources/Ingestion** — manage exchange/manual sources, run sync jobs, normalize raw data into canonical holdings.
2. **Holdings + Pricing/FX** — own asset/account records, valuation state, exact base-currency conversion, and freshness metadata.
3. **Allocations + Rebalancing** — classify holdings, compare against targets, and generate explainable advisory actions.
4. **Snapshots + Dashboard** — freeze historical portfolio state and serve read-optimized overview, drift, and history views.

### Critical Pitfalls

The biggest implementation risk is shipping a dashboard that looks polished but cannot explain its own numbers. Trust breaks faster in finance products than in typical CRUD apps.

1. **Treating exchange balances as uniform holdings** — model source account, account type, and availability state explicitly; support per-exchange capability metadata.
2. **No provenance or audit trail** — persist sync run, price source, FX source, timestamps, and manual-vs-synced origin for every valued position and snapshot.
3. **Using floats or weak money types** — use exact decimals end-to-end and centralize rounding/calculation rules.
4. **Matching assets by symbol only** — introduce canonical asset IDs plus provider-specific mappings for pricing.
5. **Hiding stale or mixed-freshness data behind one total** — expose freshness for sync, prices, FX, and manual balances; warn when rebalancing uses stale inputs.
6. **Naive rebalance logic** — sequence recommendations around protected cash, liquidity constraints, thresholds, and insufficient actionable capital states.

## Implications for Roadmap

Based on research, suggested phase structure:

### Phase 1: Canonical Portfolio Core
**Rationale:** Everything else depends on stable financial primitives and identity rules. If the data model is wrong, pricing, history, and rebalancing will all be untrustworthy.
**Delivers:** Canonical asset/account/holding model, exact-decimal schema, manual sources, provenance fields, target-allocation primitives, protected cash semantics.
**Addresses:** Manual asset support, base-currency foundation, target configuration prerequisites.
**Avoids:** Float math, symbol collisions, missing audit trail, protected-cash mis-modeling.

### Phase 2: Valuation and Current-State Dashboard
**Rationale:** The first useful product milestone is a trustworthy current portfolio view before automation complexity is introduced.
**Delivers:** Pricing/FX pipeline, valuation states, net worth dashboard, holdings drilldown, allocation analytics, freshness indicators.
**Uses:** PostgreSQL + Prisma, decimal.js, Zod, React Query.
**Implements:** Holdings, pricing, allocation, and dashboard read models.
**Avoids:** Mixed-freshness confusion, silent missing pricing, UI-side portfolio math.

### Phase 3: Snapshot History and Rebalancing Engine
**Rationale:** Once current-state calculations are trusted, history and actionability can build on top of them without rewriting the core.
**Delivers:** Immutable daily snapshots, trend charts, drift analysis, ordered advisory recommendations, emergency-fund-aware and liquidity-aware rebalance logic.
**Addresses:** Historical snapshots, rebalancing recommendations, review workflow.
**Avoids:** Recomputed history drift, noisy percentage-only advice, contradictory multi-level recommendations.

### Phase 4: Exchange Sync Automation and Connector Hardening
**Rationale:** Automated sync is operationally harder than manual modeling, so it should attach to a proven domain model rather than shape it.
**Delivers:** CCXT adapters, scheduled sync, job lifecycle, per-source locks, retries/backoff, sync-status UI, coverage indicators.
**Addresses:** Mixed-source aggregation and recurring refresh workflow.
**Avoids:** Overlapping sync jobs, unsupported account-type blind spots, rate-limit and exchange-quirk failures.

### Phase 5: UX Hardening and v1.x Enhancements
**Rationale:** Once the full loop works, improve onboarding and reduce friction rather than expanding scope sideways.
**Delivers:** CSV import, tolerance bands, scenario comparison, review checklists, reconciliation/audit views.
**Addresses:** Maintainability and repeat-use adoption.
**Avoids:** Premature expansion into tax, execution, or performance-accounting complexity.

### Phase Ordering Rationale

- Build normalized data and money semantics before any dashboard work; all analytics depend on trustworthy valuations.
- Deliver manual-source support before exchange sync so the portfolio model can be tested without flaky external APIs.
- Add snapshots only after current-state valuation and categorization are correct; otherwise history will preserve bad logic.
- Add rebalancing after targets, pricing, and allocation views are stable; recommendation quality depends on those modules being trusted.
- Treat exchange sync as an integration layer attached late to the canonical model, not as the backbone of the product.

### Research Flags

Phases likely needing deeper research during planning:
- **Phase 4:** Exchange sync automation — CCXT coverage, per-exchange capability gaps, account-type mapping, retries, and rate-limit handling need connector-specific planning.
- **Phase 3:** Rebalancing engine details — sequencing, threshold policy, and handling insufficient liquid capital need explicit domain examples before implementation.
- **Phase 5:** Pricing coverage for unsupported/manual assets — valuation-state UX and provider mapping rules may need additional source-specific decisions.

Phases with standard patterns (skip research-phase):
- **Phase 1:** Canonical schema, decimal math, and provenance-first persistence are well-supported by existing research.
- **Phase 2:** React dashboard + Nest read-model patterns are standard and well-documented.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Based largely on official docs, verified versions, and strong fit with project constraints. |
| Features | MEDIUM | Feature set is well-supported by competitor patterns and project context, but prioritization is partly product-opinion driven. |
| Architecture | HIGH | Strong synthesis from established NestJS/queue/portfolio-system patterns with clear dependency order. |
| Pitfalls | MEDIUM | Pitfalls are credible and actionable, with several backed by official docs, but some are domain synthesis rather than direct vendor guidance. |

**Overall confidence:** HIGH

### Gaps to Address

- **Initial exchange scope:** Which exchanges and which account types must v1 support first. This affects sync sequencing and adapter complexity.
- **Pricing provider strategy:** Exact provider choices for crypto, equities/ETFs, and FX were not fully locked. This affects asset registry design and valuation-state UX.
- **Snapshot policy:** Daily close only vs on-sync snapshots vs hybrid. This affects history semantics and storage/query design.
- **Rebalance policy details:** Minimum action thresholds, tolerance bands, and how to handle trading-bot capital in recommendation order need explicit product rules.
- **Manual asset depth:** Whether v1 manual support is balance-only or includes optional transaction import materially affects scope.

## Sources

### Primary (HIGH confidence)
- `C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/PROJECT.md` — project goals, constraints, and scope
- NestJS official docs and Context7 `/nestjs/nest` — backend framework and scheduling patterns
- PostgreSQL official docs — exact numeric storage and relational database guidance
- Prisma official docs and Context7 `/prisma/prisma` — schema/migrations for relational data
- React official docs and Context7 `/facebook/react` — frontend foundation
- TanStack Query official docs and Context7 `/tanstack/query` — server-state dashboard patterns
- React Router official docs — SPA routing patterns
- Docker Compose official docs — local multi-container orchestration
- CoinGecko docs — pricing ID/freshness behavior
- CCXT docs/wiki and Context7 `/ccxt/ccxt` — exchange adapter realities and capability caveats

### Secondary (MEDIUM confidence)
- Ghostfolio — self-hosted portfolio tracker positioning and table-stakes feature comparison
- Kubera — aggregation and wealth dashboard expectations
- Portfolio Performance — allocation/rebalancing and history comparison point
- Sharesight — investor feature expectations and reporting norms

### Tertiary (LOW confidence)
- Domain synthesis from cross-source portfolio-tool patterns where direct official guidance is sparse, especially for protected-cash allocation logic and multi-level advisory rebalancing.

---
*Research completed: 2026-04-20*
*Ready for roadmap: yes*
