# Roadmap: Personal Portfolio Manager

## Overview

This roadmap delivers the product in dependency order: first establish a trustworthy portfolio and valuation foundation, then connect and configure real sources with visible sync trust signals, then expose dashboard and allocation semantics with emergency-fund and Trading Bot behavior as first-class concepts, then layer in rebalancing guidance, and finally add snapshot history and Dockerized local deployment polish. Each phase delivers a user-verifiable capability and every v1 requirement maps to exactly one phase.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Portfolio Domain & Valuation Foundation** - Users get trustworthy base-currency portfolio totals from a canonical portfolio model.
- [ ] **Phase 2: Source Sync & Asset Configuration** - Users can connect sources, maintain manual assets, and trust sync freshness and valuation warnings.
- [ ] **Phase 3: Dashboards & Allocation Semantics** - Users can understand portfolio composition, targets, emergency-fund protection, and Trading Bot grouping.
- [ ] **Phase 4: Rebalancing Guidance** - Users can see target drift and receive protected-capital-aware rebalancing suggestions.
- [ ] **Phase 5: History & Local Deployment** - Users can review portfolio history and run the full app locally with Docker.

## Phase Details

### Phase 1: Portfolio Domain & Valuation Foundation
**Goal**: Users can rely on one canonical portfolio view with exact base-currency valuation across all assets.
**Depends on**: Nothing (first phase)
**Requirements**: PORT-01, PORT-02, PORT-03
**Success Criteria** (what must be TRUE):
  1. User can choose a base currency for reporting and see portfolio totals recalculated in that currency.
  2. User can view total net worth across the portfolio in the selected base currency.
  3. User can inspect both original-currency values and converted base-currency values for accounts and holdings.
**Plans**: TBD

### Phase 2: Source Sync & Asset Configuration
**Goal**: Users can configure portfolio sources, refresh data reliably, and understand whether portfolio inputs are fresh enough to trust.
**Depends on**: Phase 1
**Requirements**: SRC-01, SRC-02, SRC-03, SYNC-01, SYNC-02, SYNC-03, SYNC-04, CONF-01, CONF-02, CONF-03
**Success Criteria** (what must be TRUE):
  1. User can configure CCXT exchange sources, manual cash assets, and manual market-traded holdings from the application.
  2. User can trigger a manual refresh and the application can also run scheduled refreshes for configured sources.
  3. User can see per-source sync freshness, success state, and partial-failure warnings before relying on totals.
  4. User can see missing, stale, or unresolved valuations clearly flagged, including assets that need pricing rules, ticker mappings, or source classification updates.
**Plans**: TBD
**UI hint**: yes

### Phase 3: Dashboards & Allocation Semantics
**Goal**: Users can understand current portfolio composition through dashboards and allocation views that preserve emergency-fund and Trading Bot semantics.
**Depends on**: Phase 2
**Requirements**: PORT-04, ALLO-01, ALLO-02, ALLO-03, ALLO-04, ALLO-05, ALLO-06, ALLO-07
**Success Criteria** (what must be TRUE):
  1. User can view a dashboard summarizing net worth, allocation, and protected-liquidity status.
  2. User can view allocation across macro categories, liquidity buckets, and intra-category holdings through tables, pie charts, and bar charts.
  3. User can define target allocations for macro categories, liquidity buckets, and intra-category assets from the application.
  4. User can see emergency-fund status as underfunded or overfunded against a value or percentage target, and can see Trading Bot accounts grouped separately from long-term holdings in allocation views.
**Plans**: TBD
**UI hint**: yes

### Phase 4: Rebalancing Guidance
**Goal**: Users can turn portfolio drift into actionable suggestions that respect protected emergency-fund balances and Trading Bot classification.
**Depends on**: Phase 3
**Requirements**: REBL-01, REBL-02
**Success Criteria** (what must be TRUE):
  1. User can see current-versus-target allocation gaps in both percentage and value terms at macro, liquidity, and intra-category levels.
  2. User can receive concrete rebalancing suggestions as transfers or buy/sell amounts derived from the current portfolio state.
  3. Rebalancing suggestions do not consume protected emergency-fund balances and do not merge Trading Bot capital into ordinary long-term allocation guidance.
**Plans**: TBD
**UI hint**: yes

### Phase 5: History & Local Deployment
**Goal**: Users can review portfolio changes over time and run the full product locally with durable Dockerized infrastructure.
**Depends on**: Phase 4
**Requirements**: HIST-01, HIST-02, HIST-03, DEPL-01, DEPL-02
**Success Criteria** (what must be TRUE):
  1. User can start the frontend, backend, and database stack locally with Docker Compose.
  2. User can restart containers without losing database data and can configure services through environment variables.
  3. User can view net-worth and allocation trends over time from stored portfolio snapshots.
  4. User can review a timestamped history of recorded portfolio snapshots.
**Plans**: TBD
**UI hint**: yes

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Portfolio Domain & Valuation Foundation | 0/TBD | Not started | - |
| 2. Source Sync & Asset Configuration | 0/TBD | Not started | - |
| 3. Dashboards & Allocation Semantics | 0/TBD | Not started | - |
| 4. Rebalancing Guidance | 0/TBD | Not started | - |
| 5. History & Local Deployment | 0/TBD | Not started | - |
