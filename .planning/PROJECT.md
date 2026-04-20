# Finance Manager

## What This Is

A personal web application that gives a complete view of all assets and investments in one place. It combines automated exchange data, manually managed asset balances, multi-currency portfolio analytics, and multi-level rebalancing guidance so a single user can understand their real financial position and decide what to adjust.

## Core Value

Provide a trustworthy, unified view of total net worth and allocation across all assets so portfolio decisions and rebalancing are clear and actionable.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Aggregate asset data from CCXT-connected exchanges and manually managed accounts into one portfolio view
- [ ] Convert all holdings into a configurable base currency for net worth and allocation analysis
- [ ] Show portfolio allocation across macro categories, liquidity subcategories, and intra-category assets
- [ ] Generate actionable rebalancing guidance against target allocations, including protected emergency-fund handling
- [ ] Provide clear dashboards with charts and basic historical snapshots

### Out of Scope

- Trade execution from the app — v1 should recommend actions, not place trades or move funds automatically
- Tax reporting — not part of the initial value and would add major domain complexity
- Advanced authentication and multi-user roles — v1 is for a single personal user running the app locally

## Context

The product is a modular personal portfolio management system for a single user who currently has data scattered across crypto exchanges, banks, brokers, and manual tracking tools. The app must combine CCXT exchange data with manually entered assets such as bank accounts, current accounts, deposit accounts, emergency funds, stocks, ETFs, ETPs, and unsupported exchanges.

Portfolio analysis needs to work across multiple levels. At the macro level, the user wants target allocation across categories such as Crypto, Stocks, Liquidity, and Trading Bots. At the liquidity level, the user wants target allocation across current accounts, deposit accounts, and emergency funds. Within categories, the user also wants target allocation across specific assets such as BTC, ETH, and other holdings.

Emergency funds are a special protected allocation. The system must track whether the emergency fund is underfunded or overfunded relative to its target and reflect that in analytics and rebalancing guidance.

CCXT-connected accounts used for active trading should be grouped under a dedicated Trading Bot macro category and still participate in allocation and rebalancing logic.

The app should use mixed data freshness. Connected exchange accounts should support scheduled sync, while manual assets remain user-managed. For non-CCXT assets in v1, automatic pricing should focus on market prices and FX conversion, while cash-like balances remain manually maintained. The application should also capture basic history over time so the user can review net worth and allocation trends, not only the current snapshot.

The user wants the app to feel genuinely useful in three ways at once: trustworthy consolidated data, useful rebalancing advice, and a smooth repeatable review routine.

## Constraints

- **Tech stack**: NestJS backend, React frontend, and PostgreSQL database — explicitly required by the user
- **Deployment**: Fully containerized with separate Dockerfiles for backend, frontend, and database plus docker-compose orchestration — required for local-first operation and reproducible setup
- **User model**: Single-user personal app running primarily local-first — avoids multi-user complexity in v1
- **Automation boundary**: Automatic sync for CCXT sources and automatic pricing/FX for priced assets, but manual maintenance for bank and cash-like balances — matches realistic v1 integration scope
- **Visualization**: Must include clear dashboards with pie charts and bar charts — core to product usefulness
- **Rebalancing**: Must support macro, liquidity, and intra-category target comparisons — central product behavior, not an optional extra

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Build v1 as a single-user portfolio manager | The immediate need is personal asset visibility, not shared portfolio management | — Pending |
| Prioritize both visibility and actionability | The app must be equally strong at portfolio visibility, analytics, and rebalancing advice | — Pending |
| Keep v1 advisory-only for rebalancing | Suggested actions are valuable without introducing trade execution or money-movement risk | — Pending |
| Support basic history instead of full performance analytics | Trend visibility is useful in v1 without expanding into full returns and performance accounting | — Pending |
| Treat emergency fund as protected allocation logic | Emergency cash should be highlighted separately from normal allocation drift | — Pending |

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
