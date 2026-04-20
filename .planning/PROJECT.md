# Personal Portfolio Manager

## What This Is

A single-user web application for consolidating all personal assets and investments into one place, including crypto exchange balances via CCXT and manual assets such as bank accounts, deposit accounts, emergency funds, stocks, ETFs, ETPs, and unsupported exchanges. It gives a complete net worth view in a base currency, shows allocation across macro and intra-category buckets, and turns fragmented data into clear rebalancing guidance. The first version is local-first, containerized with Docker, and optimized for reliable data consolidation over finance-platform complexity.

## Core Value

I can see my full net worth and allocation across all assets in one place, with reliable consolidated data that leads to clear rebalancing decisions.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Aggregate balances from CCXT-connected exchanges and manual asset sources into one portfolio view
- [ ] Convert all asset values into a configurable base currency for total net worth reporting
- [ ] Show portfolio allocation across macro categories, liquidity subcategories, and intra-category assets
- [ ] Visualize portfolio state with clear dashboards, pie charts, and bar charts
- [ ] Generate rebalancing suggestions at macro, liquidity, and intra-category levels
- [ ] Treat the emergency fund as a protected allocation and flag underfunded or overfunded states
- [ ] Group active-trading CCXT accounts under a Trading Bot macro category in analytics and rebalancing
- [ ] Keep basic historical snapshots for net worth and allocation over time

### Out of Scope

- Multi-user or SaaS account model — v1 is for one personal user only
- Direct trade or transfer execution — v1 provides suggestions only
- Full transaction-ledger accounting for manual assets — v1 tracks current values rather than full manual transaction history
- Advanced performance analytics such as drawdowns, attribution, or tax reporting — prioritize consolidation and rebalancing first
- Mandatory authentication for local usage — v1 assumes trusted local/private deployment

## Context

The product replaces a fragmented workflow spread across exchanges, banking apps, broker views, and manual tracking. The highest-value problem to solve first is reliable consolidation of data from many sources, not collaboration or automation. Manual assets need to coexist with exchange-connected accounts, and market-traded manual holdings should support automatic pricing rather than fully manual valuation.

The allocation model has multiple layers. At the top level, the portfolio is split across macro categories such as Crypto, Stocks, Liquidity, and Trading Bots. Within liquidity, balances are further structured across current accounts, deposit accounts, and emergency funds. Within categories such as crypto, the system should also evaluate target allocations for individual assets such as BTC and ETH.

The emergency fund is a special protected allocation. It can be defined by target value or target percentage, must remain visible in allocation logic, and should trigger warnings when it is below or above target. CCXT-connected accounts used for active trading should not be mixed into ordinary long-term crypto holdings and must instead roll up into a separate Trading Bot macro category.

The first deployment target is local Docker, using NestJS for the backend, React for the frontend, and PostgreSQL for persistence. The system should be modular and scalable so it can later support broader deployment or richer finance features without reworking the core domain model.

## Constraints

- **Tech stack**: NestJS backend, React frontend, PostgreSQL database — explicitly required for implementation
- **Containerization**: One Dockerfile for backend, one for frontend, one for database service, plus docker-compose.yml with env vars, shared network, and persistent volumes — required deployment model
- **User model**: Single-user local-first app with no required auth in v1 — minimize friction and keep scope focused
- **Data sync**: Support both scheduled sync and manual refresh — needs reliable updates plus user control
- **Execution boundary**: Suggestions only, no automated transfers or trades — avoid operational and security complexity in v1
- **Manual asset model**: Manual assets are maintained as current values, while market-traded manual holdings need fetched prices — balances speed of entry with useful valuation
- **History depth**: Basic snapshot history only — enough for net worth and allocation trends without expanding into full analytics

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Build v1 as a single-user personal application | The product is for one trusted user and should avoid premature SaaS complexity | — Pending |
| Optimize first for data consolidation | The main pain today is fragmented visibility across many sources | — Pending |
| Support both manual refresh and scheduled sync | The app needs automation without losing user-triggered control | — Pending |
| Limit v1 to analysis and suggestions | Rebalancing guidance is valuable without taking on execution risk | — Pending |
| Treat Trading Bot accounts as their own macro category | Active trading capital should be visible separately from long-term holdings | — Pending |
| Treat the emergency fund as a protected allocation | Liquidity safety should be preserved, not diluted inside generic cash balances | — Pending |
| Use current-value tracking for manual assets in v1 | This keeps manual asset entry manageable while still supporting broad coverage | — Pending |
| Keep only basic history in v1 | Historical snapshots are useful, but advanced performance analytics are not core yet | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-04-20 after initialization*