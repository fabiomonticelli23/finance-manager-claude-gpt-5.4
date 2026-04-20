# Project Research Summary

**Project:** Personal Portfolio Manager
**Domain:** Single-user personal portfolio management web app
**Researched:** 2026-04-20
**Confidence:** MEDIUM-HIGH

## Executive Summary

This project is a local-first personal portfolio manager focused on one job: turning fragmented balances, manual assets, and market prices into a trustworthy base-currency net worth and allocation view. The research consistently points to a consolidation-first architecture: normalize CCXT exchange balances and manual holdings into one canonical portfolio model, run explicit valuation with timestamped prices and FX, then derive allocation, protected emergency-fund status, and suggestions-only rebalancing from that trusted layer.

The recommended implementation is a modular monolith using NestJS, React, and PostgreSQL, with domain seams around sources, pricing, portfolio reads, allocation, policy rules, rebalancing, and snapshots. Experts would build this as a correctness-first finance dashboard, not as a real-time trading tool: exact decimals, explicit freshness state, adapter boundaries for external providers, append-only snapshots, and backend-owned portfolio truth are mandatory.

The main risks are not visual polish or scale. They are wrong numbers that look convincing: stale or partial valuations, broken asset/symbol mapping, float-based arithmetic, hidden sync failures, and recommendations that ignore protected liquidity. Mitigation is clear from the research: make valuation runs traceable, treat sync trust as a product feature, encode emergency-fund and Trading Bot logic in domain policy, and delay history/rebalancing sophistication until valuation reliability is stable.

## Key Findings

### Recommended Stack

The stack recommendation is strong and coherent with the project constraints. Use NestJS 11 for a modular TypeScript backend, React 19 with Vite for a fast local dashboard frontend, and PostgreSQL 18 as the authoritative relational store. Prisma is the default ORM because it supports productive schema evolution, typed access, and Decimal-backed financial data handling, while exact arithmetic should still use PostgreSQL `numeric` plus `decimal.js` in application logic.

The supporting stack should stay deliberately boring: CCXT behind adapters for exchange sync, `@nestjs/schedule` for in-process scheduled refreshes, Zod across request and form boundaries, TanStack Query for server state, React Hook Form for configuration/manual-entry flows, Recharts for dashboard visuals, and Playwright plus Testcontainers for trust-critical testing. Redis, BullMQ, Next.js, and real-time infrastructure are explicitly unnecessary for v1.

**Core technologies:**
- **NestJS 11.1.19**: backend API and orchestration — strong module boundaries fit sync, valuation, policy, and rebalancing services.
- **React 19.2.5 + Vite 8.0.9**: dashboard UI — fast local SPA workflow with mature forms, tables, and chart ecosystem.
- **PostgreSQL 18**: primary database — relational integrity, exact numerics, and straightforward snapshot/history queries.
- **Prisma 7.7.0**: ORM and migrations — productive schema management with typed access and better defaults for money-sensitive code.
- **CCXT 4.5.50**: exchange integration — broad exchange support, but only safe behind normalization adapters.
- **decimal.js 10.6.0**: financial arithmetic — required to avoid float errors in valuations, percentages, and rebalancing math.

### Expected Features

The feature research is opinionated: the product succeeds if users trust the numbers and the guidance, not if it ships broad analytics. Table stakes are unified net worth, multi-currency valuation, reliable sync with manual refresh, manual asset coverage, allocation views, protected emergency-fund handling, Trading Bot separation, simple charts, basic snapshots, and suggestions-only rebalancing.

Differentiation comes from domain-specific decision support rather than breadth: protected emergency-fund-aware rebalancing, hierarchical allocation targets, explainable suggestions, strong sync/freshness UX, and hybrid manual-asset valuation. Features that drag the app toward bookkeeping or brokerage operations should be deferred.

**Must have (table stakes):**
- Unified portfolio model across CCXT accounts and manual assets
- Base-currency net worth with timestamped multi-currency valuation
- Reliable sync with scheduled refresh, manual refresh, and visible sync status
- Allocation views across macro category, liquidity bucket, and asset level
- Protected emergency fund tracking with target-based warnings
- Trading Bot capital grouped separately from long-term crypto
- Suggestions-only rebalancing based on target drift
- Basic snapshots and simple current-state/history charts

**Should have (competitive):**
- Rebalancing that explicitly respects protected emergency-fund rules
- Cross-layer allocation targets from macro down to asset level
- Explainable suggestion output with rationale and assumptions
- Source freshness, partial-failure visibility, and trust-oriented sync UX
- Lightweight drift tracking over time once snapshots are reliable

**Defer (v2+):**
- Trade execution or automation
- Full transaction-ledger accounting for manual assets
- Advanced performance analytics, attribution, or tax reporting
- Cloud auth, multi-user support, and collaboration
- Real-time/live dashboards and heavy market-data infrastructure

### Architecture Approach

The architecture research strongly supports a modular monolith. The backend should own canonical portfolio truth, with separate modules for source ingestion, pricing and FX, portfolio read models, allocation, policy rules, rebalancing, snapshots, and settings. The frontend should remain thin and presentation-focused, consuming coarse-grained REST endpoints for dashboard, sources, settings, rebalancing, and history. Core patterns are adapter boundaries for CCXT/manual/pricing providers, derived read models over authoritative core tables, append-only snapshots, and pure domain services for policy/rebalancing.

**Major components:**
1. **Source ingestion/orchestration** — sync CCXT and manual sources, normalize positions, persist run outcomes and freshness.
2. **Pricing, FX, and valuation** — resolve prices and exchange rates, compute exact base-currency values, and materialize valued positions.
3. **Portfolio and allocation read layer** — answer dashboard/net-worth/allocation queries from canonical valued data.
4. **Policy rules** — enforce emergency-fund protection and strategic bucket semantics before recommendations.
5. **Rebalancing suggestions** — generate explainable, suggestions-only target-gap actions with safeguards.
6. **Snapshot history** — persist append-only portfolio state after successful valuation runs for trends and drift views.

### Critical Pitfalls

The pitfall research is clear: most failure modes come from semantic and trust issues, not framework choice.

1. **Timestamp-free valuation** — avoid showing a single “current” number without balance, price, and FX freshness metadata; every displayed total should tie back to a valuation run.
2. **Float-based money math** — avoid JavaScript `number` and floating DB types for finance logic; use PostgreSQL `numeric`, Prisma Decimal, and `decimal.js`.
3. **Broken symbol/instrument mapping** — avoid using raw symbols as canonical identity; create canonical asset/instrument mapping with explicit source mappings and overrides.
4. **Hidden partial refreshes** — avoid implying sync success when sources or prices failed; expose partial success, stale-in-use, and per-source status in the product.
5. **Emergency fund or Trading Bot dilution** — avoid treating protected cash and active trading capital as generic balances; model both as first-class policy/taxonomy concepts before allocation and rebalancing.

## Implications for Roadmap

Based on research, suggested phase structure:

### Phase 1: Foundation, Canonical Domain, and Valuation
**Rationale:** Everything downstream depends on trusted portfolio truth. Research across features, architecture, and pitfalls all agree that normalized holdings, exact decimals, canonical asset mapping, and timestamped valuation must come first.
**Delivers:** Core schema, source/account/holding model, canonical asset/instrument mapping, valuation runs, FX and pricing abstractions, exact money math, base-currency net worth API, and initial settings.
**Addresses:** Unified portfolio model, multi-currency valuation, base-currency net worth.
**Avoids:** Timestamp-free valuations, float precision errors, flat “everything is an asset” schema, broken symbol mapping.

### Phase 2: Sync Reliability and Manual Asset Coverage
**Rationale:** Once the valuation layer exists, the next highest risk is bad inputs and hidden failures. The product must earn trust before it expands UX breadth.
**Delivers:** CCXT adapter layer, manual current-value assets, manual market-priced holdings, scheduled sync, manual refresh, sync-run records, per-source freshness/health, unresolved pricing/mapping handling, and secure server-side credential handling.
**Uses:** NestJS modules, `@nestjs/schedule`, CCXT adapters, Prisma/PostgreSQL lineage records.
**Implements:** Ingestion orchestrator, source adapters, pricing/FX provider boundaries, sync status surfaces.

### Phase 3: Dashboards, Allocation, and Portfolio Semantics
**Rationale:** Once values and sync reliability are credible, the app can explain the portfolio with confidence. This is the right point to encode domain-specific semantics that make the product useful.
**Delivers:** Dashboard DTOs, macro/liquidity/asset allocation trees, emergency-fund policy evaluation, Trading Bot grouping, holdings/allocation tables, and basic current-state charts.
**Addresses:** Allocation views, dashboard charts, protected emergency-fund tracking, Trading Bot macro bucket.
**Avoids:** Misleading charts, emergency-fund dilution, Trading Bot mixing, frontend-derived portfolio truth.

### Phase 4: Suggestions-Only Rebalancing
**Rationale:** Rebalancing should come only after allocation and policy outputs are stable, because it is allocation analysis plus protected-capital rules. Shipping it earlier would create low-trust recommendations.
**Delivers:** Target configuration, drift analysis, thresholds/deadbands, explainable suggestion output, stale-data blocking, and protected-capital-aware recommendation logic.
**Addresses:** Suggestions-only rebalancing, hierarchical allocation decision support.
**Avoids:** Unsafe or noisy recommendations, dust-sized actions, suggestions that cross protected boundaries.

### Phase 5: Snapshot History and Drift Tracking
**Rationale:** History is valuable, but only after valuation semantics are stable. Otherwise the app just preserves bad numbers and misleading trends.
**Delivers:** Append-only snapshots tied to valuation runs, net-worth history, allocation history, basic drift-over-time views, and history-oriented dashboard charts.
**Addresses:** Basic historical snapshots, trend context, lightweight drift tracking.
**Avoids:** Non-reproducible snapshots, retroactively shifting history, decorative but untrustworthy trend charts.

### Phase Ordering Rationale

- The research strongly supports the user’s preferred order: valuation before sync breadth, sync reliability before dashboards, dashboards/allocation before rebalancing, and history only after valuation stability.
- Feature dependencies explicitly show allocation depends on trusted valuation, rebalancing depends on allocation plus policy logic, and snapshots depend on a stable valuation pipeline.
- This grouping mirrors the recommended architecture seams, which reduces module bleed and keeps roadmap phases testable.
- It also prevents the top pitfalls in the order they are most dangerous: wrong numbers first, hidden sync issues second, semantic chart/allocation mistakes third, and misleading suggestions/history last.

### Research Flags

Phases likely needing deeper research during planning:
- **Phase 2:** CCXT exchange behavior, source credential handling, pricing fallback behavior, and symbol/market mapping edge cases need implementation-specific planning research.
- **Phase 4:** Rebalancing thresholds, explainability format, and safety constraints need careful domain tuning to avoid low-quality guidance.
- **Phase 5:** Snapshot lineage, restatement policy, and history semantics need explicit planning if historical comparability matters.

Phases with standard patterns (skip research-phase):
- **Phase 1:** NestJS modular monolith, PostgreSQL numeric modeling, Prisma migrations, and valuation-run architecture are already well supported by the research.
- **Phase 3:** Standard dashboard/read-model patterns are well documented once the domain model is settled.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Strong alignment with required stack, official docs, and current package ecosystem; only market-data provider choice remains somewhat provisional. |
| Features | MEDIUM-HIGH | Feature direction is coherent and well supported by project context plus competitor/product pattern research, but prioritization still includes judgment calls. |
| Architecture | HIGH | Strong consensus from official framework/database patterns and clear fit for a local single-user modular monolith. |
| Pitfalls | MEDIUM-HIGH | Highly credible and actionable, though some items are expert synthesis rather than directly verifiable official-source claims. |

**Overall confidence:** MEDIUM-HIGH

### Gaps to Address

- **Canonical instrument mapping policy:** Need explicit rules for ambiguous exchange symbols, wrapped assets, stablecoins, and manual security ticker normalization during planning.
- **Price-provider fallback strategy:** Need a defined fallback/reconciliation path for unsupported or zero-priced manual securities and edge-case assets.
- **Credential storage approach for local deployment:** Need a concrete implementation choice for secure local secret storage and rotation workflow.
- **Rebalancing thresholds and exclusions:** Need specific minimum trade sizes, deadbands, and stale-data gating criteria before roadmap tasks are finalized.
- **Snapshot restatement policy:** Need to decide whether historical points remain as originally computed or can be recalculated after mapping/pricing corrections.

## Sources

### Primary (HIGH confidence)
- `C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/PROJECT.md` — project scope, constraints, required stack, and domain rules
- NestJS official docs: https://docs.nestjs.com/modules , https://docs.nestjs.com/providers — modular backend patterns
- React official docs: https://react.dev/learn/thinking-in-react — frontend composition guidance
- PostgreSQL official docs: https://www.postgresql.org/docs/current/index.html , https://www.postgresql.org/docs/current/datatype-numeric.html — relational modeling and exact numeric guidance
- Docker Compose docs: https://docs.docker.com/compose/ , https://docs.docker.com/compose/how-tos/networking/ — local deployment/networking patterns
- OWASP cheat sheets: https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html , https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html — secret handling guidance

### Secondary (MEDIUM confidence)
- CCXT Manual: https://raw.githubusercontent.com/ccxt/ccxt/master/wiki/Manual.md and https://github.com/ccxt/ccxt/wiki/Manual — exchange integration constraints and adapter implications
- Prisma PostgreSQL and Decimal docs: https://www.prisma.io/docs/orm/overview/databases/postgresql , https://www.prisma.io/docs/orm/prisma-client/special-fields-and-types#working-with-decimal — ORM/database compatibility and decimal handling
- Kubera product page: https://kubera.com/ — competitor baseline for unified net worth/manual asset expectations
- getquin portfolio tracker: https://getquin.com/portfolio-tracker — competitor baseline for dashboard/manual import/history expectations
- npm registry/package docs cited in `STACK.md` — version verification for recommended packages

### Tertiary (LOW confidence)
- Domain synthesis from research documents for roadmap implications, recommendation ordering, and some pitfall severity judgments — needs validation during implementation planning where provider-specific behavior matters most

---
*Research completed: 2026-04-20*
*Ready for roadmap: yes*
