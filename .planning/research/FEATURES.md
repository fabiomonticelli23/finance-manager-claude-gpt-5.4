# Feature Research

**Domain:** Personal portfolio management web application (self-hosted, single-user, aggregated net worth + allocation analytics + rebalancing)
**Researched:** 2026-04-20
**Confidence:** MEDIUM

## Feature Landscape

### Table Stakes (Users Expect These)

Features users assume exist. Missing these = product feels incomplete.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Unified holdings dashboard across all accounts | Every serious portfolio tracker promises a single view across brokers, exchanges, cash, and manual assets | MEDIUM | This is the product’s core promise. Must support both synced sources and manual accounts in one portfolio view. |
| Net worth in a base currency | Users expect one total number they can trust, even with multi-currency holdings | MEDIUM | Requires FX conversion, source timestamps, and clear handling for stale/manual prices. |
| Asset/account-level drilldown | Users need to answer “what do I own, where, and how much is it worth?” | LOW | Table, filters, per-account breakdown, and asset details are standard. |
| Allocation analytics by category and asset | Portfolio tools commonly show diversification, allocation, and concentration views | MEDIUM | Must support your domain-specific layers: macro categories, liquidity buckets, and intra-category assets. |
| Historical snapshots and trend charts | Users expect to see whether net worth and allocation are improving over time, not just current state | MEDIUM | Daily snapshots are enough for MVP; do not start with full performance accounting. |
| Manual asset and cash account support | Aggregation products still require manual entry for unsupported banks, real estate, private assets, and cash balances | LOW | Essential for a self-hosted app because connector coverage will always be incomplete. |
| Price and FX updates for market-priced assets | Without current valuations, totals and allocations become untrustworthy | MEDIUM | Separate market-price automation from manually maintained nominal cash balances. |
| Data freshness/status visibility | Users need to know what is current, stale, synced, or manually maintained | LOW | A trust feature, not a nice-to-have. Show last sync / last update clearly. |
| Rebalancing gap analysis against targets | Allocation-focused products are expected to show drift versus desired allocation | HIGH | For this project, this is table stakes because actionability is part of the core value, not an optional advanced feature. |
| Import/edit workflows for transactions or balances | Users expect a practical way to bootstrap and maintain data, not only API syncs | MEDIUM | CSV import plus manual balance editing is enough initially. Transaction-level imports can be selective. |

### Differentiators (Competitive Advantage)

Features that set the product apart. Not required, but valuable.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Multi-level target allocation engine | Most tools stop at one allocation layer; supporting macro, liquidity, and intra-category targets makes the app materially more useful for real decision-making | HIGH | This is the clearest differentiator. It turns a dashboard into an allocation operating system. |
| Emergency fund as protected allocation logic | Treating emergency cash as a protected target avoids misleading rebalance suggestions and matches real household portfolio behavior | MEDIUM | Distinguishes between “available to rebalance” and “must remain reserved.” |
| Rebalancing recommendations expressed as concrete actions | Users want “move €X from A to B” rather than only “you are 4% overweight BTC” | HIGH | Keep advisory-only. Recommendations should respect protected buckets and configurable tolerances. |
| Mixed-source portfolio model | Seamlessly combining CCXT exchange sync, manual accounts, and priced/unpriced assets is more realistic than assuming all wealth lives in one broker feed | MEDIUM | Strong fit for self-hosted users with fragmented finances. |
| Review-oriented workflow | A repeatable weekly/monthly review flow can make the app feel habit-forming instead of passive | MEDIUM | Example: refresh data → inspect net worth change → inspect drift → review suggested moves. |
| Liquidity-aware analytics | Showing liquid cash, emergency reserves, and deployable capital separately from long-term holdings is more actionable than generic allocation charts | MEDIUM | Especially useful for users balancing savings, investing, and active trading. |
| Category-aware treatment of trading-bot / active trading accounts | Grouping active exchange accounts separately prevents speculative capital from distorting long-term allocation decisions | MEDIUM | Good differentiation for users with both passive and active portfolios. |
| Opinionated trust UX | Explicit stale-data labels, source attribution, and confidence cues increase trust more than adding more charts | LOW | Underinvested area in many hobby/self-hosted finance apps. |

### Anti-Features (Commonly Requested, Often Problematic)

Features that seem good but create problems.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Direct trade execution from the app | Sounds convenient: detect drift and place trades immediately | Expands scope into brokerage/exchange permissions, order safety, compliance concerns, error handling, and irreversible money movement | Keep v1 advisory-only with action checklists and optional exportable rebalance instructions |
| Full tax reporting / tax-lot engine | Common ask from portfolio tools users | Tax logic is jurisdiction-specific, audit-sensitive, and much larger than the core allocation problem | Export clean transaction/balance data and defer tax workflows to dedicated tools |
| Retirement planning / Monte Carlo suite | Feels adjacent to net worth and portfolio tracking | Becomes a planning product instead of a portfolio operations product; requires assumptions, liability modeling, and UX expansion | Start with current-state analytics and simple trend history only |
| “Track every possible asset type in depth” | Users want one place for everything | Broad asset support at equal depth creates messy data models and weak UX | Support broad asset inclusion, but only deep analytics for assets relevant to allocation/rebalancing |
| Real-time streaming updates everywhere | Feels modern and impressive | Little value for long-horizon portfolio review, but adds infra cost, sync complexity, and noisy UX | Scheduled sync plus manual refresh and clear freshness indicators |
| Social / benchmark / community features | Many consumer finance apps add comparisons or social proof | Misaligned with a private self-hosted product; creates privacy and UX distractions | Focus on private, personal decision support |
| Full performance analytics before data trust is solved | Time-weighted returns and attribution look sophisticated | Performance metrics are misleading when imports, missing transactions, and stale valuations are unresolved | First ship trustworthy holdings, valuations, snapshots, and allocation drift |

## Feature Dependencies

```
Data source connections + manual account entry
    └──requires──> Asset/account model
                         └──requires──> Valuation model (prices + FX + manual values)
                                              └──requires──> Unified net worth view
                                                                   └──requires──> Allocation analytics
                                                                                        └──requires──> Target allocation configuration
                                                                                                             └──requires──> Rebalancing recommendations

Historical snapshots ──enhances──> Net worth view
Historical snapshots ──enhances──> Allocation analytics

Emergency fund rules ──constrains──> Rebalancing recommendations
Trading-bot category model ──constrains──> Allocation analytics
Data freshness visibility ──enhances──> Trust in all analytics
Direct trade execution ──conflicts──> Advisory-only MVP
Full tax engine ──conflicts──> Allocation-focused scope
```

### Dependency Notes

- **Data source connections + manual account entry require an asset/account model:** before aggregation works, the app needs a consistent way to represent accounts, holdings, balances, categories, and ownership.
- **Unified net worth requires valuation model:** net worth is only credible once prices, FX rates, and manual valuations resolve into one base currency.
- **Allocation analytics require unified net worth:** category percentages and drift are derived from normalized valuations.
- **Target allocation configuration requires allocation analytics:** there is little value in targets if the system cannot first compute actual allocation correctly.
- **Rebalancing recommendations require target allocation configuration:** suggested actions need target percentages, tolerances, and minimum unit rules.
- **Historical snapshots enhance both net worth and allocation analytics:** trend views become useful once the system stores normalized daily state.
- **Emergency fund rules constrain rebalancing recommendations:** protected cash must not be treated as deployable capital.
- **Trading-bot category model constrains allocation analytics:** active trading capital should be separated to avoid distorting long-term target comparisons.
- **Data freshness visibility enhances trust in all analytics:** users need to know whether a recommendation is based on up-to-date values.
- **Direct trade execution conflicts with advisory-only MVP:** combining them early increases product and operational complexity dramatically.
- **Full tax engine conflicts with allocation-focused scope:** it would dominate roadmap capacity without improving the core review-and-rebalance loop.

## MVP Definition

### Launch With (v1)

Minimum viable product — what’s needed to validate the concept.

- [ ] Unified portfolio dashboard across synced and manual accounts — essential because aggregation is the core job-to-be-done
- [ ] Base-currency net worth and holdings valuation — essential because every other analytic depends on normalized values
- [ ] Allocation analytics across macro categories, liquidity buckets, and key assets — essential because the product is allocation-focused
- [ ] Target allocation configuration with emergency-fund protection — essential because it makes drift analysis meaningful for this use case
- [ ] Advisory rebalancing recommendations — essential because the app must be actionable, not only descriptive
- [ ] Basic historical snapshots and charts — essential because users need trend context during recurring reviews
- [ ] Data freshness/source-status indicators — essential because trust is a core product requirement

### Add After Validation (v1.x)

Features to add once core is working.

- [ ] CSV onboarding/import flows — add when manual setup friction slows adoption or migration from spreadsheets
- [ ] Scenario comparison for proposed rebalances — add when users want to test “what if I move cash here?” before acting
- [ ] Tolerance bands, minimum trade thresholds, and rebalance modes — add when recommendations need to feel less noisy and more practical
- [ ] More asset-specific analytics (yield, income, concentration alerts) — add when the core allocation loop is stable
- [ ] Saved review workflows / monthly checklist UX — add when users are using the product repeatedly rather than exploring once

### Future Consideration (v2+)

Features to defer until product-market fit is established.

- [ ] Transaction-level performance accounting (IRR/TWR, attribution) — defer because it requires much cleaner historical data and more domain complexity
- [ ] Advanced connector ecosystem beyond initial CCXT + manual model — defer until source coverage is the main bottleneck
- [ ] Mobile-first companion experience — defer until the desktop/local review workflow is proven
- [ ] Household / multi-user support — defer because permissions, ownership splits, and collaboration complicate the model substantially
- [ ] Tax workflows or jurisdiction-specific exports — defer because this is effectively a separate product area

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Unified holdings dashboard | HIGH | MEDIUM | P1 |
| Base-currency net worth | HIGH | MEDIUM | P1 |
| Allocation analytics | HIGH | MEDIUM | P1 |
| Target allocation configuration | HIGH | MEDIUM | P1 |
| Advisory rebalancing recommendations | HIGH | HIGH | P1 |
| Historical snapshots/charts | HIGH | MEDIUM | P1 |
| Manual account support | HIGH | LOW | P1 |
| Data freshness/status visibility | HIGH | LOW | P1 |
| CSV import/onboarding | MEDIUM | MEDIUM | P2 |
| Tolerance bands / rebalance modes | MEDIUM | MEDIUM | P2 |
| Scenario comparison | MEDIUM | MEDIUM | P2 |
| Income/yield analytics | MEDIUM | MEDIUM | P2 |
| Performance attribution metrics | MEDIUM | HIGH | P3 |
| Trade execution | LOW | HIGH | P3 |
| Tax reporting | LOW | HIGH | P3 |

**Priority key:**
- P1: Must have for launch
- P2: Should have, add when possible
- P3: Nice to have, future consideration

## Competitor Feature Analysis

| Feature | Ghostfolio | Kubera | Portfolio Performance / Sharesight | Our Approach |
|---------|------------|--------|------------------------------------|--------------|
| Aggregated holdings view | Multi-account transaction/holding tracking; self-hosted positioning | Strong multi-institution and multi-asset aggregation | Strong portfolio/account visibility | Match this as table stakes, but optimize for mixed synced + manual sources |
| Net worth visibility | Present as wealth management / portfolio tracking | Major product focus | Present through holdings/performance visibility | Make unified net worth the primary home screen |
| Allocation/diversification analytics | Charts and portfolio insights | Broad wealth views, less allocation-opinionated | Diversity/allocation reporting exists | Go deeper with explicit allocation hierarchy |
| Historical tracking | Multi-window performance/history | Historical views and reporting | Strong history and performance reporting | Ship basic snapshots first, not full performance accounting |
| Rebalancing | Not clearly promoted | Not a core promoted feature | Portfolio Performance explicitly supports allocation rebalancing | Make rebalancing a first-class workflow, not a buried report |
| Privacy / self-hosting | Strong open-source/self-hosted angle | Hosted SaaS | Portfolio Performance is local-first desktop | Lean into local-first trust and control |
| Planning / adjacent features | API/imports and portfolio checks | Estate planning, forecasting, broad asset coverage | Sharesight emphasizes tax/reporting; PP emphasizes performance depth | Deliberately avoid broad adjacent categories until core allocation workflow is strong |

## Sources

- Project context: `C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/PROJECT.md`
- Ghostfolio homepage summary via WebFetch: https://ghostfol.io/en
- Ghostfolio README summary via WebFetch: https://github.com/ghostfolio/ghostfolio
- Kubera homepage via WebFetch: https://www.kubera.com/
- Kubera help/article attempt for supported assets: https://support.kubera.com/en/articles/2006450-what-can-i-track-with-kubera
- Portfolio Performance official site via WebFetch: https://www.portfolio-performance.info/en/
- Sharesight features page via WebFetch: https://www.sharesight.com/features/
- Sharesight investor-needs article via WebFetch: https://www.sharesight.com/au/blog/7-portfolio-tracking-features-that-all-investors-need/

---
*Feature research for: personal portfolio management*
*Researched: 2026-04-20*
