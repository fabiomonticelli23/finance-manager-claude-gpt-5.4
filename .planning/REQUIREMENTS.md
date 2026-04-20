# Requirements: Personal Portfolio Manager

**Defined:** 2026-04-20
**Core Value:** I can see my full net worth and allocation across all assets in one place, with reliable consolidated data that leads to clear rebalancing decisions.

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Sources

- [ ] **SRC-01**: User can connect a CCXT-supported exchange account for portfolio ingestion
- [ ] **SRC-02**: User can record manual cash assets including current accounts, deposit accounts, and emergency-fund balances
- [ ] **SRC-03**: User can record manual market-traded holdings including stocks, ETFs, and ETPs

### Sync & Valuation

- [ ] **SYNC-01**: User can trigger a manual refresh of portfolio data
- [ ] **SYNC-02**: User can run scheduled portfolio refresh for configured sources
- [ ] **SYNC-03**: User can see per-source sync freshness, success state, and partial-failure warnings
- [ ] **SYNC-04**: User can see missing, stale, or unresolved valuations flagged before relying on portfolio totals

### Portfolio Overview

- [ ] **PORT-01**: User can choose a base currency for portfolio reporting
- [ ] **PORT-02**: User can view total net worth across all assets in the selected base currency
- [ ] **PORT-03**: User can view original currency values and converted base-currency values for accounts and holdings
- [ ] **PORT-04**: User can view a dashboard summary of net worth, allocation, and protected-liquidity status

### Allocation

- [ ] **ALLO-01**: User can view portfolio allocation by macro category including Crypto, Stocks, Liquidity, and Trading Bot
- [ ] **ALLO-02**: User can view liquidity allocation across current accounts, deposit accounts, and emergency funds
- [ ] **ALLO-03**: User can view intra-category allocation for holdings within a category such as BTC, ETH, and other assets
- [ ] **ALLO-04**: User can view portfolio composition through pie charts and bar charts
- [ ] **ALLO-05**: User can define target allocations for macro categories, liquidity buckets, and intra-category assets
- [ ] **ALLO-06**: User can see CCXT-connected active-trading accounts grouped under the Trading Bot macro category
- [ ] **ALLO-07**: User can set an emergency-fund target as a value or percentage and see underfunded or overfunded status

### Rebalancing

- [ ] **REBL-01**: User can see current-versus-target allocation gaps in percentage and value terms at macro, liquidity, and intra-category levels
- [ ] **REBL-02**: User can receive concrete rebalancing suggestions as transfers or buy/sell amounts that respect protected emergency-fund balances and Trading Bot classification

### History

- [ ] **HIST-01**: User can view net-worth trend over time from stored portfolio snapshots
- [ ] **HIST-02**: User can view allocation trend over time from stored portfolio snapshots
- [ ] **HIST-03**: User can review a timestamped log of historical portfolio snapshots

### Settings & Pricing

- [ ] **CONF-01**: User can configure exchange sources and manual assets from the application
- [ ] **CONF-02**: User can configure pricing rules or ticker mappings for manually tracked market assets
- [ ] **CONF-03**: User can classify CCXT-connected accounts as Trading Bot or long-term holdings for allocation logic

### Deployment

- [ ] **DEPL-01**: User can start the full frontend, backend, and database stack locally with Docker Compose
- [ ] **DEPL-02**: User can keep database data across container restarts and configure services through environment variables

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Guidance

- **REBL-03**: User can see explanation and assumptions behind each rebalancing suggestion
- **REBL-04**: User can see allocation drift against targets across historical periods

### Data Management

- **DATA-01**: User can import or export manual asset data through CSV workflows
- **DATA-02**: User can use fallback pricing and reconciliation workflows for unsupported or ambiguous market instruments

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Automated trade or transfer execution | V1 provides suggestions only and should avoid operational, security, and compliance complexity |
| Advanced tax reports or deep performance analytics | V1 is validating consolidation, allocation, and rebalancing rather than full portfolio analytics |
| Full transaction-ledger accounting for manual assets | V1 tracks current values for manual assets instead of becoming bookkeeping software |
| Multi-user auth or SaaS account model | V1 is a single-user local-first product |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|

**Coverage:**
- v1 requirements: 27 total
- Mapped to phases: 0
- Unmapped: 27 ⚠️

---
*Requirements defined: 2026-04-20*
*Last updated: 2026-04-20 after initial definition*