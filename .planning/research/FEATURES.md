# Feature Research

**Domain:** Single-user personal portfolio management dashboard
**Researched:** 2026-04-20
**Confidence:** MEDIUM

## Feature Landscape

### Table Stakes (Users Expect These)

Features users assume exist. Missing these = product feels incomplete.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Unified net worth across all sources in a base currency | The core job is "show me what I own in one number." Without this, the app is just a collection of disconnected balances. | MEDIUM | Must normalize CCXT balances and manual assets into one valuation model with per-source timestamps and FX conversion. |
| Reliable source sync plus manual refresh controls | For a portfolio dashboard, stale or failed imports destroy trust faster than missing analytics. Users expect both automation and explicit refresh. | HIGH | Treat sync status, last successful sync, partial failure reporting, and retry behavior as product features, not backend details. |
| Manual asset support alongside connected accounts | Most personal portfolios are hybrid. Users expect to track bank cash, emergency funds, unsupported exchanges, and broker positions even when no API exists. | MEDIUM | Manual entries should support current-value assets and market-priced holdings separately; do not force full transaction accounting in v1. |
| Multi-currency valuation into a configurable base currency | Cross-currency holdings are normal for cash, exchanges, and securities. A personal wealth app that cannot convert cleanly feels broken. | HIGH | Needs asset currency, account currency, FX rates, valuation timestamps, and explicit handling for missing FX data. |
| Allocation views by macro category and asset | Once net worth exists, users expect to see where money is concentrated. This is a baseline portfolio-dashboard expectation. | MEDIUM | Support macro categories such as Crypto, Stocks, Liquidity, Trading Bot, then drill into asset-level allocation. |
| Protected emergency fund tracking | In this exact app, liquidity is not generic cash. Users need emergency reserves visible and excluded from casual rebalancing pressure. | MEDIUM | Emergency fund should have explicit target logic, protected status, and under/over target warnings. |
| Dashboard charts for current state | Pie/bar visuals are expected in portfolio tools because allocation is easier to understand visually than in tables alone. | LOW | Keep charts focused on current net worth, category allocation, liquidity split, and top positions. Avoid advanced charting in MVP. |
| Basic historical snapshots for net worth and allocation | Users expect at least trend visibility and proof that the portfolio is moving over time, even in simple tools. | MEDIUM | Snapshot model should be periodic portfolio states, not transaction reconstruction. Daily or sync-triggered snapshots are enough for v1. |
| Suggestions-only rebalancing guidance | A portfolio manager without any "what should I do next" output feels passive. For this app, rebalancing is core value, not an optional report. | HIGH | Start with target-vs-actual drift and suggested moves by category/asset. No execution, optimization engine, or tax-aware logic in v1. |
| Clear handling of Trading Bot capital as its own macro bucket | Active trading funds distort long-term allocation if mixed with core crypto holdings. For this project, this separation is foundational. | MEDIUM | Model Trading Bot as a first-class category with dedicated inclusion rules in dashboards and rebalancing. |

### Differentiators (Competitive Advantage)

Features that set the product apart. Not required, but valuable.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Rebalancing engine that respects protected emergency fund logic | Most trackers show allocation; fewer encode "this cash is intentionally untouchable." This makes suggestions feel usable rather than naive. | HIGH | Suggestions should treat emergency-fund deficits as priority funding needs before proposing risk-asset purchases. |
| Cross-layer allocation model: macro, liquidity sub-allocation, and asset-level targets | Many tools stop at asset class views. This app becomes more useful by aligning high-level allocation with liquidity purpose and per-asset drift. | HIGH | Strong differentiator if the hierarchy stays simple and explainable. Requires category model before suggestion logic. |
| Hybrid valuation model for manual holdings | Supporting current-value manual assets plus market-priced manual securities covers real-world portfolios better than "manual means fully manual." | MEDIUM | Important for stocks/ETFs/ETPs held outside supported integrations. Use fetched prices when ticker mapping is available. |
| Explainable rebalancing output | Users trust suggestions more when the app says why: target, current, drift, protected balances, and proposed action. | MEDIUM | Prefer plain-language suggestions over optimizer-style black boxes. |
| Sync trust surface: source health, freshness, and partial-data warnings | Consumer tools often hide import problems. Making reliability visible is a product advantage for a consolidation-first app. | MEDIUM | Show per-source freshness, last success, stale flags, and whether values are estimated or confirmed. |
| Lightweight historical allocation drift tracking | Not full performance analytics, but seeing how allocation moved versus targets over time is genuinely useful for rebalancing discipline. | MEDIUM | More valuable here than advanced return attribution. Build on snapshot data already needed for history. |

### Anti-Features (Commonly Requested, Often Problematic)

Features that seem good but create problems.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Trade execution or automatic transfers | Sounds convenient: "just rebalance for me." | Explodes scope into broker/exchange actions, security concerns, permissions, error handling, and regulatory complexity. It also conflicts with the local-first suggestions-only v1. | Generate actionable suggestions with source-aware instructions and manual checklists. |
| Full transaction-ledger accounting for every manual asset | Feels more "complete" and finance-serious. | It turns a dashboard into bookkeeping software, massively increases data-entry burden, and delays the core value of consolidated visibility. | Track current value for non-market assets and quantity-plus-price for market-traded manual holdings. |
| Real-time tick-by-tick syncing and live dashboards | Feels premium and exciting. | High complexity, fragile integrations, noisy UX, and little incremental value for personal allocation decisions. | Scheduled sync plus manual refresh, with explicit freshness timestamps. |
| Advanced performance analytics in v1 (TWR/IRR attribution, factor analysis, drawdowns) | Power users ask for it and competitors market it. | Requires transaction fidelity, benchmark models, and richer history than this product needs to validate its core value. | Keep basic net-worth and allocation history; defer deeper analytics until the data model proves stable. |
| Tax optimization and jurisdiction-specific reporting | Users care about taxes. | Highly country-specific, high-maintenance, and dependent on transaction-level data the v1 model explicitly avoids. | Exportable summary views and later tax-specific modules if demand appears. |
| Auth, teams, and cloud collaboration | Common SaaS default assumption. | Solves the wrong problem for a trusted single-user local deployment and adds account lifecycle overhead immediately. | Stay single-user local-first; add auth only when remote deployment becomes a real requirement. |

## Feature Dependencies

```text
[Source connectors + manual asset entry]
    └──requires──> [Unified asset/account domain model]
                           └──requires──> [Currency + valuation engine]
                                                  └──requires──> [Base-currency net worth]
                                                                     └──requires──> [Allocation views]
                                                                                         └──requires──> [Rebalancing suggestions]
                                                                                         └──requires──> [Allocation charts]

[Emergency fund protected logic]
    └──requires──> [Liquidity category model]
                           └──enhances──> [Rebalancing suggestions]

[Trading Bot category]
    └──requires──> [Account/source classification rules]
                           └──enhances──> [Macro allocation accuracy]
                           └──enhances──> [Rebalancing suggestions]

[Historical snapshots]
    └──requires──> [Stable valuation pipeline]
                           └──enhances──> [Trend charts]
                           └──enhances──> [Allocation drift tracking]

[Advanced performance analytics]
    ──conflicts──> [Ruthless MVP scope]

[Trade execution]
    ──conflicts──> [Suggestions-only local-first v1]
```

### Dependency Notes

- **Source connectors + manual asset entry require a unified asset/account domain model:** CCXT balances, bank cash, unsupported exchanges, and manually tracked securities must land in one normalized schema before any meaningful analytics are possible.
- **Unified asset/account domain model requires a currency + valuation engine:** Every holding needs a valuation path in base currency, or net worth and allocation become inconsistent.
- **Base-currency net worth requires allocation views:** Allocation percentages are downstream of trusted total valuation.
- **Allocation views require rebalancing suggestions:** Rebalancing is just target-aware allocation analysis. Build the allocation engine first, then attach suggestion logic.
- **Emergency fund protected logic requires a liquidity category model:** You cannot protect what the model does not distinguish. Emergency funds must not be generic cash tags bolted on later.
- **Trading Bot category requires account/source classification rules:** The app needs deterministic rules for which balances roll into Trading Bot versus long-term Crypto.
- **Historical snapshots require a stable valuation pipeline:** Snapshotting bad or inconsistent values creates misleading history. Reliability beats frequency.
- **Advanced performance analytics conflict with ruthless MVP scope:** They pull the product toward transaction accounting and away from consolidation plus rebalancing.
- **Trade execution conflicts with suggestions-only local-first v1:** It introduces operational responsibility the product explicitly does not need yet.

## MVP Definition

### Launch With (v1)

Minimum viable product — what's needed to validate the concept.

- [ ] Unified portfolio model across CCXT sources and manual assets — essential because consolidation is the product's primary job.
- [ ] Base-currency net worth with explicit multi-currency conversion — essential because all downstream analytics depend on one trusted valuation layer.
- [ ] Source sync system with manual refresh, scheduled sync, and visible sync status — essential because reliability is a first-class concern and trust driver.
- [ ] Manual asset entry for current-value assets and market-priced manual holdings — essential because unsupported assets are guaranteed in real personal portfolios.
- [ ] Allocation views across macro categories, liquidity buckets, and asset level — essential because users need more than a total number.
- [ ] Protected emergency fund modeling with target and warnings — essential because the app's liquidity logic is part of its core value, not a minor setting.
- [ ] Trading Bot category support — essential because this project explicitly separates active-trading capital from long-term holdings.
- [ ] Suggestions-only rebalancing based on target drift — essential because the app promises actionable allocation decisions, not just reporting.
- [ ] Basic historical snapshots for net worth and allocation — essential because users need trend context and snapshot history is specifically in scope.
- [ ] Simple dashboard charts for current state and history — essential because visual comprehension is part of the dashboard job, but keep them intentionally basic.

### Add After Validation (v1.x)

Features to add once core is working.

- [ ] Explainable suggestion breakdowns by funding source and target rule — add when basic rebalancing logic is trusted and users want more guidance detail.
- [ ] Historical drift views versus targets — add when snapshot data quality is proven over time.
- [ ] CSV import/export for manual positions and snapshots — add when repeated manual entry becomes a clear pain point.
- [ ] Smarter stale-data handling and reconciliation workflows — add when real-world sync edge cases are observed across sources.
- [ ] Price-source fallback rules for manual traded assets — add when ticker mapping and missing-price issues show up in practice.

### Future Consideration (v2+)

Features to defer until product-market fit is established.

- [ ] Performance analytics beyond simple history — defer because they need deeper data fidelity than the MVP should carry.
- [ ] Tax reports and jurisdiction-specific logic — defer because the maintenance burden is high and unrelated to validating consolidation/rebalancing value.
- [ ] Mobile-first apps or push notifications — defer until desktop/local usage patterns prove demand.
- [ ] Cloud sync, auth, and multi-device collaboration — defer until local-first single-user constraints stop fitting the product.
- [ ] Trade execution or automation — defer indefinitely unless the product strategy changes materially.

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Unified asset/account domain model | HIGH | HIGH | P1 |
| Base-currency valuation and FX conversion | HIGH | HIGH | P1 |
| Source sync reliability surface | HIGH | HIGH | P1 |
| Manual asset entry | HIGH | MEDIUM | P1 |
| Macro/liquidity/asset allocation views | HIGH | MEDIUM | P1 |
| Protected emergency fund logic | HIGH | MEDIUM | P1 |
| Trading Bot category | HIGH | MEDIUM | P1 |
| Suggestions-only rebalancing | HIGH | HIGH | P1 |
| Basic historical snapshots | HIGH | MEDIUM | P1 |
| Dashboard charts | MEDIUM | LOW | P1 |
| Explainable suggestion details | MEDIUM | MEDIUM | P2 |
| Historical drift vs targets | MEDIUM | MEDIUM | P2 |
| CSV import/export | MEDIUM | LOW | P2 |
| Price-source fallback sophistication | MEDIUM | MEDIUM | P2 |
| Advanced performance analytics | LOW | HIGH | P3 |
| Trade execution | LOW | VERY HIGH | P3 |
| Tax reporting | LOW | HIGH | P3 |
| Auth and cloud collaboration | LOW | HIGH | P3 |

**Priority key:**
- P1: Must have for launch
- P2: Should have, add when possible
- P3: Nice to have, future consideration

## Competitor Feature Analysis

| Feature | Competitor A | Competitor B | Our Approach |
|---------|--------------|--------------|--------------|
| Unified net worth across many asset types | Kubera emphasizes one net-worth view across public, private, crypto, and manual assets | getquin emphasizes a live portfolio dashboard across linked and manual assets | Match this baseline, but keep the implementation local-first and single-user. |
| Manual asset coverage | Kubera strongly supports "track anything" style asset coverage | getquin supports manual and CSV-based imports across multiple asset classes | Support broad manual assets, but keep data entry minimal: current-value for non-market assets, quantity-plus-price for market-traded holdings. |
| Historical tracking | Kubera highlights portfolio growth over time | getquin highlights performance and historical asset data | Do the minimum useful version: snapshots for net worth and allocation, not full performance analytics. |
| Allocation breakdowns | Kubera highlights allocation and dynamic reporting | getquin highlights geography/industry/asset-class views and look-through analysis | Focus on the app's actual decision layers: macro category, liquidity, and asset allocation. |
| Rebalancing | I did not find rebalancing emphasized on Kubera's main page | I did not find rebalancing emphasized on getquin's portfolio tracker page | Make rebalancing a core workflow, especially with protected emergency fund and Trading Bot logic. |
| Sync/import trust | Kubera emphasizes broad connectors and AI/manual import for unsupported sources | getquin emphasizes automatic bank connection plus manual/CSV import | Treat sync freshness, source health, and fallback manual entry as first-class UX, not hidden plumbing. |

## Ruthless MVP Recommendation

If this product ships with weak sync reliability, fuzzy FX conversion, or naive rebalancing, it fails even if the UI looks polished. The MVP should therefore optimize for trusted data consolidation and decision support, not analytics breadth.

Build in this order:

1. **Portfolio domain model + valuation engine** — normalize sources, currencies, and categories first.
2. **Sync reliability + manual asset entry** — get trustworthy inputs before richer outputs.
3. **Current-state dashboards and allocation views** — prove the app can explain the portfolio now.
4. **Protected emergency fund + Trading Bot categorization** — encode the app's domain-specific logic early.
5. **Suggestions-only rebalancing** — turn visibility into action.
6. **Basic historical snapshots** — add trend context once valuations are stable.

Defer anything that depends on transaction-level fidelity, real-time infrastructure, or operational automation. For this exact app, the winning MVP is: "I can trust the numbers, see where my money is, protect my emergency fund, and know what to rebalance next." Anything beyond that is secondary.

## Sources

- Project context: `C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/PROJECT.md`
- Feature template: `C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.claude/get-shit-done/templates/research-project/FEATURES.md`
- Official product page, Kubera: https://kubera.com/ (WebFetch summary, 2026-04-20, MEDIUM confidence due page-level marketing source)
- Official product page, getquin portfolio tracker: https://getquin.com/portfolio-tracker (WebFetch summary, 2026-04-20, MEDIUM confidence)

---
*Feature research for: single-user personal portfolio management dashboard*
*Researched: 2026-04-20*