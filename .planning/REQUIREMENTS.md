# Requirements: Finance Manager

**Defined:** 2026-04-20
**Core Value:** Provide a trustworthy, unified view of total net worth and allocation across all assets so portfolio decisions and rebalancing are clear and actionable.

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Sources & Onboarding

- [ ] **SRC-01**: User can create manual sources for liquidity accounts including current accounts, deposit accounts, and emergency funds
- [ ] **SRC-02**: User can create manual holdings for traditional assets and unsupported sources including stocks, ETFs, ETPs, and exchanges not supported by CCXT
- [ ] **SRC-03**: User can connect a supported crypto exchange via CCXT using read-only API credentials
- [ ] **SRC-04**: User can import balances or holdings from CSV to accelerate initial onboarding
- [ ] **SRC-05**: User can classify a synced exchange account as Trading Bot capital so it appears as a separate macro category in analytics and rebalancing

### Valuation & Currency

- [ ] **VAL-01**: User can choose a base currency and see portfolio totals recalculated in that currency
- [ ] **VAL-02**: User can see total net worth converted into the selected base currency across all manual and synced assets
- [ ] **VAL-03**: User can automatically fetch market prices for manually tracked stocks, ETFs, and ETPs
- [ ] **VAL-04**: User can automatically convert holdings across currencies using current FX rates
- [ ] **VAL-05**: User can see whether each source or holding is synced, manual, fresh, stale, auto-priced, manually valued, or unpriced

### Dashboards & History

- [ ] **DASH-01**: User can view a consolidated dashboard of all assets across manual and synced sources
- [ ] **DASH-02**: User can drill into portfolio data by source, account, and asset
- [ ] **DASH-03**: User can view allocation by macro category with pie charts and bar charts
- [ ] **DASH-04**: User can view allocation within liquidity buckets such as current accounts, deposit accounts, and emergency funds
- [ ] **DASH-05**: User can view allocation within a category down to individual assets such as BTC, ETH, and other holdings
- [ ] **DASH-06**: User can view historical net worth and allocation trends from daily and sync-driven snapshots

### Targets & Rebalancing

- [ ] **REB-01**: User can define target allocations for macro categories such as Crypto, Stocks, Liquidity, and Trading Bots
- [ ] **REB-02**: User can define target allocations for liquidity buckets such as current accounts, deposit accounts, and emergency funds
- [ ] **REB-03**: User can define target allocations within a category for specific assets such as BTC, ETH, and other holdings
- [ ] **REB-04**: User can set a protected emergency-fund target and see whether it is underfunded or overfunded
- [ ] **REB-05**: User can compare current allocation versus target allocation at every configured level
- [ ] **REB-06**: User can receive ordered rebalance suggestions with amounts, reasons, and constraints that respect protected cash rules

### Sync, Operations & Deployment

- [ ] **OPS-01**: User can trigger exchange sync manually on demand
- [ ] **OPS-02**: User can schedule recurring sync for connected exchange accounts
- [ ] **OPS-03**: User can see per-source sync status, last successful run, freshness, and failures in the application UI
- [ ] **OPS-04**: User can run the full application locally with Docker Compose using separate backend, frontend, and database services, environment variables, a shared network, and persistent database storage

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Rebalancing Enhancements

- **REBX-01**: User can test what-if rebalance scenarios before acting
- **REBX-02**: User can configure tolerance bands and minimum action thresholds for rebalance suggestions

### Data & Analytics

- **DATX-01**: User can import detailed transaction history instead of balance-level onboarding only
- **DATX-02**: User can view advanced portfolio performance metrics such as returns and attribution

### Platform Expansion

- **PLAT-01**: User can use dedicated bank/open-banking integrations for cash account balances instead of manual maintenance
- **PLAT-02**: User can manage multiple users or household portfolios with separate access controls

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Direct trade execution | V1 should recommend actions, not place trades or move funds automatically |
| Tax reporting | Adds major jurisdiction-specific complexity outside the core allocation workflow |
| Advanced authentication and roles | V1 is a single-user local-first product |
| Broad connector coverage beyond initial CCXT + manual model | Source coverage should expand only after the core portfolio model is trusted |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| SRC-01 | — | Pending |
| SRC-02 | — | Pending |
| SRC-03 | — | Pending |
| SRC-04 | — | Pending |
| SRC-05 | — | Pending |
| VAL-01 | — | Pending |
| VAL-02 | — | Pending |
| VAL-03 | — | Pending |
| VAL-04 | — | Pending |
| VAL-05 | — | Pending |
| DASH-01 | — | Pending |
| DASH-02 | — | Pending |
| DASH-03 | — | Pending |
| DASH-04 | — | Pending |
| DASH-05 | — | Pending |
| DASH-06 | — | Pending |
| REB-01 | — | Pending |
| REB-02 | — | Pending |
| REB-03 | — | Pending |
| REB-04 | — | Pending |
| REB-05 | — | Pending |
| REB-06 | — | Pending |
| OPS-01 | — | Pending |
| OPS-02 | — | Pending |
| OPS-03 | — | Pending |
| OPS-04 | — | Pending |

**Coverage:**
- v1 requirements: 26 total
- Mapped to phases: 0
- Unmapped: 26 ⚠️

---
*Requirements defined: 2026-04-20*
*Last updated: 2026-04-20 after initial definition*
