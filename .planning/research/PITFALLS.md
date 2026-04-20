# Pitfalls Research

**Domain:** Personal portfolio management and asset aggregation web application
**Researched:** 2026-04-20
**Confidence:** MEDIUM

## Critical Pitfalls

### Pitfall 1: Treating exchange balances as one uniform concept

**What goes wrong:**
The app assumes a single `fetchBalance` result means "the user's holdings". In practice, exchanges split balances across spot, funding, earn, margin, futures, subaccounts, and bot/trading wallets. The product then shows incomplete net worth, double-counts balances moved between account types, or gives rebalancing advice on assets that are not actually available in the intended account.

**Why it happens:**
CCXT exposes a unified API, which makes it tempting to believe balances are fully normalized across exchanges. They are not. Exchange capabilities differ, support can be incomplete, and market/account-type behavior varies by exchange.

**How to avoid:**
- Model holdings as `source account + account type + asset + availability state`, not just `exchange + asset`.
- Add a per-exchange adapter layer that explicitly maps supported account types for the app.
- Store raw sync payloads for audit/debug, plus normalized holdings for analytics.
- Require a visible sync coverage indicator: "spot only", "spot + funding", etc.
- Make "Trading Bot" a first-class source/category, not an inference from exchange name.
- Treat unsupported account types as known gaps, not silent omissions.

**Warning signs:**
- Net worth differs from exchange UI in ways users cannot explain.
- Users report "missing funds" that actually sit in funding/earn/futures wallets.
- Same exchange needs ad hoc exceptions everywhere in the codebase.
- Rebalancing suggestions assume funds are liquid when they are locked elsewhere.

**Phase to address:**
Phase 1 - Domain model and source normalization foundation; Phase 2 - Exchange sync adapters

---

### Pitfall 2: No provenance or audit trail for synced vs manual holdings

**What goes wrong:**
Once values are normalized, the system loses where a number came from, when it was captured, what quote currency was used, and whether it was user-entered or fetched. When totals look wrong, there is no way to explain or repair them. Trust collapses quickly in a finance product.

**Why it happens:**
Teams optimize for a clean dashboard first and skip ledger-like traceability because it feels "internal". Portfolio tools need explainability, not just pretty totals.

**How to avoid:**
- Persist source provenance for every position snapshot: connector/manual source, account, asset identifier, timestamp, raw amount, priced value, pricing source, FX rate source, sync run id.
- Keep immutable snapshot/history tables separate from current-state materializations.
- Show "last synced" and "last priced" at asset, source, and portfolio level.
- Add a reconciliation view from the start: raw source value -> normalized holding -> base-currency valuation.
- Record manual edit history for balances and protected cash buckets.

**Warning signs:**
- Bugs are diagnosed by re-running sync and guessing.
- Users cannot tell whether a number is stale, manual, or synced.
- Engineers say "the total is right but we can't explain how".

**Phase to address:**
Phase 1 - Persistence model and auditability; Phase 5 - Trust/reconciliation UX

---

### Pitfall 3: Using floats or locale-bound money types for balances and prices

**What goes wrong:**
Rounding drift appears in holdings, FX conversions, allocation percentages, and rebalance advice. Tiny errors then cascade into visible inconsistencies: category totals do not match asset totals, or rebalancing thresholds trigger incorrectly.

**Why it happens:**
Developers use JavaScript `number`, PostgreSQL `double precision`, or `money` because they are convenient. Portfolio systems need exact decimal storage and explicit currency metadata.

**How to avoid:**
- Store all quantities, prices, FX rates, and computed values as exact decimals end-to-end.
- In PostgreSQL, use `numeric`, not floating point and not `money`.
- Store currency code separately from amount.
- Define explicit precision/scale policies per field type: asset quantity, fiat amount, price, FX rate, percentage.
- Centralize arithmetic and rounding rules so UI/API/background jobs use the same logic.

**Warning signs:**
- 99.999999-style values in logs or APIs.
- Category totals differ by small amounts from the sum of positions.
- Base-currency net worth changes when recomputed without any source change.

**Phase to address:**
Phase 1 - Financial data model and calculation primitives

---

### Pitfall 4: Matching assets by symbol instead of canonical identity

**What goes wrong:**
The app prices the wrong asset because symbols collide across tokens, wrapped assets, brokers, ETFs, and manual entries. Historical pricing also breaks when an asset is renamed, delisted, or mapped to the wrong vendor ID.

**Why it happens:**
Symbol strings look universal, but they are not. Pricing APIs explicitly prioritize IDs over symbols because symbol-only matching is ambiguous.

**How to avoid:**
- Introduce a canonical asset registry inside the app.
- Each asset must have: internal asset id, display name, type, optional pricing-provider ids, optional exchange symbol mappings, and optional ISIN/ticker metadata for traditional assets.
- Price by provider-specific ID whenever possible.
- Force user confirmation when creating a manual asset that could map to multiple instruments.
- Treat unsupported/unpriced assets as intentional manual valuations or excluded-from-market-value entries.

**Warning signs:**
- A manually entered asset suddenly inherits a wildly wrong market price.
- Two unrelated assets share the same symbol in app records.
- Historical charts jump after changing a symbol mapping.

**Phase to address:**
Phase 1 - Asset identity model; Phase 3 - Pricing and market-data integration

---

### Pitfall 5: Hiding stale or mixed-freshness data behind one "current" total

**What goes wrong:**
Exchange balances may be minutes old, manual cash may be weeks old, and prices may be cached on a different cadence. The dashboard still presents one authoritative-looking net worth figure. Users make decisions from mixed timestamps without realizing it.

**Why it happens:**
Teams think freshness metadata is noise. In portfolio software, freshness is part of correctness.

**How to avoid:**
- Track freshness separately for holdings sync, market prices, FX rates, and manual balances.
- Compute portfolio status from freshness-aware components and expose a portfolio confidence banner.
- Mark each source as fresh/stale/manual/failed.
- Let users exclude stale sources from rebalance advice, or at least warn when advice uses stale inputs.
- Snapshot valuation timestamps explicitly so history is reproducible.

**Warning signs:**
- Users ask why values differ from exchange UI even though sync "succeeded" earlier.
- Rebalancing advice changes materially after a refresh with no manual edits.
- Support/debugging needs to ask "when was this number from?" too often.

**Phase to address:**
Phase 2 - Sync orchestration; Phase 3 - Pricing/FX freshness model; Phase 4 - Analytics presentation

---

### Pitfall 6: Treating cash-like balances and emergency funds as ordinary allocation buckets

**What goes wrong:**
Emergency funds, deposit accounts, and operational cash are treated exactly like investable capital. The system recommends reallocating protected cash into risk assets, or marks the portfolio as underweight risk because protected cash is intentionally reserved.

**Why it happens:**
Generic portfolio tools optimize for investable assets only. Your project explicitly includes protected cash buckets, which changes both analytics and rebalance logic.

**How to avoid:**
- Model protected cash with explicit semantics: protected, target amount/target range, currency, liquidity classification, and whether excess is investable.
- Separate three concepts: total net worth, investable net worth, and protected reserves.
- Rebalancing should operate on investable capital by default, with emergency-fund shortfall treated as a top-priority funding action before investment reallocations.
- Handle underfunded and overfunded emergency funds differently.
- In charts, visually separate protected cash from drift that is actually actionable.

**Warning signs:**
- Advice says to buy risk assets while emergency fund is below target.
- Liquidity allocation appears "wrong" because reserve buckets distort normal category weights.
- Users cannot tell which cash is protected versus deployable.

**Phase to address:**
Phase 1 - Domain rules for cash buckets; Phase 4 - Rebalancing engine

---

### Pitfall 7: Calculating allocation from gross value without liquidity or actionability rules

**What goes wrong:**
The app compares target allocations against gross holdings even when some assets are illiquid, locked, fee-constrained, or operationally separate. Advice becomes theoretically correct but practically useless.

**Why it happens:**
Allocation math is easier if every holding is just market value. Real portfolios have different actionability constraints.

**How to avoid:**
- Attach actionability metadata to holdings: liquid, locked, protected, trading-bot capital, manually priced, unpriced.
- Compute multiple allocation views: gross, investable, liquid-only, and category-specific.
- Make the rebalance engine produce "reason codes" for excluded assets.
- Distinguish analysis mode from action mode: "portfolio composition" can include everything; "what should I do next" should use only actionable capital.

**Warning signs:**
- Recommended trades require moving protected or illiquid funds.
- Users need to mentally ignore parts of the dashboard to trust the recommendation.
- Same portfolio looks balanced in one screen and impossible to rebalance in another.

**Phase to address:**
Phase 4 - Allocation analytics and actionability-aware rebalancing

---

### Pitfall 8: Rebalancing advice that ignores thresholds, costs, and sequencing

**What goes wrong:**
The app emits noisy micro-adjustments, contradictory instructions, or advice that double-counts the same money across macro, liquidity, and intra-category targets. Users stop following recommendations because they are too chatty or impossible to execute.

**Why it happens:**
Rebalancing is often implemented as a simple diff-to-target percentage. Your product needs multi-level targets with protected buckets, which requires sequencing rules.

**How to avoid:**
- Define a strict rebalance hierarchy: fund protected cash shortfalls -> satisfy liquidity targets -> satisfy macro allocation -> satisfy intra-category targets.
- Add minimum drift thresholds and minimum action amounts.
- Prefer "next best actions" over a mathematically complete but operationally unrealistic plan.
- Include advisory text explaining why an action was suggested and what assumptions were used.
- Support "insufficient liquid capital" outcomes instead of forcing fake precision.

**Warning signs:**
- Recommendation list contains many tiny actions.
- Advice at one level conflicts with advice at another.
- A user cannot execute the plan in a plausible order.

**Phase to address:**
Phase 4 - Rebalancing rules engine and recommendation UX

---

### Pitfall 9: Snapshot history built from recomputation instead of stored valuations

**What goes wrong:**
Historical net worth and allocation charts change retroactively when market mappings, FX rates, or category rules change. Users lose trust because yesterday's numbers are rewritten by today's logic.

**Why it happens:**
Teams store only current holdings and recompute history on demand. That is fine for some analytics, but not for portfolio review and auditability.

**How to avoid:**
- Persist point-in-time snapshots of holdings, valuations, FX rates used, category assignments used, and protected-cash state.
- Version classification rules if portfolio categories may change over time.
- Be explicit about historical granularity: e.g. daily close snapshots versus ad hoc sync snapshots.
- Allow late data repair, but log when history was corrected and why.

**Warning signs:**
- Yesterday's chart moves after updating asset mappings.
- Historical allocation percentages differ depending on when the report is generated.
- Debugging history requires hitting live pricing APIs.

**Phase to address:**
Phase 3 - Snapshot model; Phase 5 - Historical charts and audit UX

---

### Pitfall 10: Sync scheduling without idempotency, locking, or failure isolation

**What goes wrong:**
Cron jobs overlap, duplicate syncs race each other, partial failures leave mixed snapshots, and a slow connector blocks all others. Portfolio totals become nondeterministic.

**Why it happens:**
Background sync looks simple in a single-user app, so teams skip job state management. Even one local app can have restarts, retries, duplicate triggers, or future multi-instance deployment.

**How to avoid:**
- Make each sync run idempotent and explicitly stateful: queued, running, succeeded, partial, failed.
- Use per-source locks so one connector cannot overlap with itself.
- Isolate connector failures; one failed exchange must not invalidate manual assets or other sources.
- Write snapshots atomically per run, then promote them to "current state" only when normalization and pricing complete.
- Add retry policy with backoff, but never retry blindly on auth/rate-limit/config errors.

**Warning signs:**
- Same source has multiple overlapping job records.
- Current portfolio totals flicker during sync windows.
- A partial sync updates some assets but not the source status.

**Phase to address:**
Phase 2 - Sync orchestration and job lifecycle

---

### Pitfall 11: Ignoring rate limits, server time drift, and exchange-specific options

**What goes wrong:**
Connectors work in development, then fail intermittently in real use because of rate limits, timestamp mismatch, symbol metadata not loaded, or exchange-specific required params. Sync reliability becomes fragile and expensive to debug.

**Why it happens:**
CCXT smooths over many API differences, but not all of them. Teams ship with a "one generic exchange service" design and discover every exchange still needs some custom handling.

**How to avoid:**
- Load and cache market metadata before relying on symbol interpretation.
- Honor exchange rate limits and configure client pacing deliberately.
- Support per-exchange options and time-drift adjustments.
- Maintain connector capability metadata in code and UI.
- Start with a narrow, well-tested set of exchanges rather than claiming broad support early.

**Warning signs:**
- Sync succeeds for one exchange and fails for another with the same code path.
- Timestamp/auth errors disappear after retries.
- Connector code accumulates opaque conditional branches without a declared capability model.

**Phase to address:**
Phase 2 - Exchange connector hardening

---

### Pitfall 12: No explicit strategy for unpriced, unsupported, or manually valued assets

**What goes wrong:**
The app either drops assets from totals, silently carries old prices forever, or fabricates valuations using loose symbol matches. The portfolio looks complete but is materially wrong.

**Why it happens:**
Asset aggregation projects focus on the happy path of exchange-traded assets. Real personal portfolios contain unsupported brokers, private holdings, cash-like products, and instruments with no clean API pricing.

**How to avoid:**
- Give every asset one of four valuation states: automatically priced, manually priced, quantity-only/unvalued, excluded from market value.
- Surface these states in totals and charts.
- Require explicit user choice for unsupported assets instead of silent fallback.
- Separate missing price from zero value.
- For manual valuations, store effective date and user-entered rationale.

**Warning signs:**
- Assets vanish from charts after sync.
- Portfolio totals look "complete" but some sources have no pricing coverage.
- Users cannot tell whether a manual asset is unpriced or just worth zero.

**Phase to address:**
Phase 3 - Pricing coverage and valuation-state UX

---

## Technical Debt Patterns

Shortcuts that seem reasonable but create long-term problems.

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Keep only one `current_balances` table with no immutable snapshots | Fast MVP implementation | No audit trail, unstable history, hard-to-debug sync issues | Never |
| Match assets by ticker/symbol only | Faster onboarding and pricing integration | Mispriced assets, broken history, ambiguous mappings | Only for disposable prototypes |
| Recompute all analytics directly from live APIs | Less storage/modeling work | Non-reproducible numbers, rate-limit pain, stale/live mixing | Never |
| Store values as JS number / DB float | Less plumbing | Rounding drift in money math | Never |
| Treat manual assets as a generic note field instead of modeled sources | Quick UI | Manual data becomes second-class, no provenance, impossible trust story | Only in first throwaway spike |
| One generic rebalance percentage formula for all levels | Easy initial demo | Contradictory advice, protected-cash errors, user distrust | Never |

## Integration Gotchas

Common mistakes when connecting to external services.

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| CCXT exchanges | Assuming all exchanges expose the same balance/account-type semantics | Build per-exchange capability metadata and adapter logic |
| CCXT exchanges | Syncing without loading market metadata or handling exchange options | Load markets up front and support per-exchange config/options |
| CCXT exchanges | Retrying every failure the same way | Distinguish rate-limit/network failures from auth/config/capability failures |
| CoinGecko / pricing APIs | Matching by symbol only | Store provider-specific IDs and use symbols only as a user-facing hint |
| CoinGecko / pricing APIs | Treating prices as real-time and timeless | Store quote currency, freshness timestamp, and provider snapshot time |
| Manual accounts | Assuming manual balances behave like synced balances | Mark as manual, editable, and potentially stale with separate review workflow |
| FX conversion | Using a single "latest" FX rate everywhere | Store rate source and timestamp used for each valuation snapshot |

## Performance Traps

Patterns that work at small scale but fail as usage grows.

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Repricing every asset on every page load | Slow dashboards, pricing API quota pressure | Precompute valuation snapshots and cache current portfolio materializations | Breaks as soon as history and multiple sources are present |
| Joining raw sync payloads directly into analytics queries | Slow analytics and brittle SQL | Normalize into reporting tables/materialized views | Breaks when daily history reaches months/years |
| Running all exchange syncs serially in one job | Long refresh windows, one slow source blocks all | Run per-source jobs with locks and isolated failure states | Breaks once 3-5 sources or slow APIs are involved |
| Regenerating full history after each manual edit | Charts lag after small changes | Incremental snapshoting and targeted recalculation | Breaks with multi-month daily history |
| Computing multi-level allocation recursively in the UI | Inconsistent numbers across screens | Centralize portfolio calculation engine in backend/domain layer | Breaks once there are multiple views and exports |

## Security Mistakes

Domain-specific security issues beyond general web security.

| Mistake | Risk | Prevention |
|---------|------|------------|
| Storing exchange API keys without a clear read-only contract | Credential misuse or accidental trade permissions | Require read-only keys, document exchange-specific permission setup, and reject risky scopes where possible |
| Logging raw connector errors with headers/secrets | Credential leakage in logs and screenshots | Redact credentials, signatures, account identifiers, and sensitive payload fragments |
| Exposing full holdings details in generic health/debug endpoints | Sensitive financial data exposure | Separate operational health from financial diagnostics; require authenticated local-only access |
| Using shared environment variables for all connectors with no per-source isolation | Cross-connector confusion and accidental data mixing | Store connector configs per source with explicit labels and validation |
| Treating local-first as "security not needed" | Sensitive personal financial data left unprotected at rest | Encrypt secrets, minimize retention of raw payloads, and document local backup/restore handling |

## UX Pitfalls

Common user experience mistakes in this domain.

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Showing one polished total without freshness/exclusion context | User trusts a number that is partly stale or incomplete | Add freshness badges, excluded-asset summaries, and valuation-state indicators |
| Hiding manual assets behind a secondary workflow | Manual holdings feel bolted on and are forgotten | Make manual assets first-class sources with review prompts |
| Rebalancing output as pure percentages | Users cannot turn advice into action | Provide reasoned, ordered recommendations with approximate amounts and constraints |
| No explanation for why a bucket is protected | Emergency-fund logic feels arbitrary | Show target, current level, shortfall/excess, and how it affects investable capital |
| Mixing net worth, investable capital, and liquid capital in one chart | Users misread what is actionable | Separate views explicitly and label them in plain language |
| Silent fallback when pricing is missing | Dashboard appears correct but is incomplete | Use visible missing-price states and force user decisions for unsupported assets |

## "Looks Done But Isn't" Checklist

Things that appear complete but are missing critical pieces.

- [ ] **Exchange sync:** Verify balances are scoped by account type and source coverage is visible per connector.
- [ ] **Manual assets:** Verify edits are versioned, dated, and included in reconciliation views.
- [ ] **Pricing:** Verify assets are mapped by canonical/provider IDs, not symbols only.
- [ ] **Base-currency net worth:** Verify the FX rate timestamp used for each valuation snapshot is stored.
- [ ] **Allocation analytics:** Verify protected cash is separated from investable allocation logic.
- [ ] **Rebalancing:** Verify recommendations are thresholded, sequenced, and can return "insufficient actionable capital".
- [ ] **History:** Verify charts read from stored snapshots, not live recomputation against current mappings.
- [ ] **Freshness:** Verify every dashboard total can be explained by source sync time and price time.
- [ ] **Unsupported assets:** Verify unpriced assets are surfaced explicitly instead of dropped or zeroed.

## Recovery Strategies

When pitfalls occur despite prevention, how to recover.

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Wrong asset mapping by symbol | HIGH | Freeze pricing for affected asset, correct canonical/provider mapping, backfill impacted snapshots with an audit note, and notify user that history changed |
| Missing exchange subaccount balances | MEDIUM | Add connector capability metadata, resync source with separate account types, reconcile before/after totals, and mark prior periods as incomplete where needed |
| Float-based financial math in persisted data | HIGH | Introduce exact-decimal columns, migrate and recompute all derived values, verify against fixture portfolios, and lock old fields |
| Non-idempotent sync job corruption | HIGH | Disable scheduler, isolate bad runs, restore last known-good snapshots, patch job locking/idempotency, then re-enable source by source |
| Protected cash handled as ordinary liquidity | MEDIUM | Reclassify bucket rules, recompute investable vs protected views, regenerate rebalance advice, and document behavioral change in release notes |
| History built from current logic only | HIGH | Introduce immutable snapshots, choose a backfill policy, label reconstructed history as estimated, and avoid pretending it is original truth |

## Pitfall-to-Phase Mapping

How roadmap phases should address these pitfalls.

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Exchange balances treated as uniform | Phase 1-2 | For each connector, app can show supported account types and reconcile source totals visibly |
| No provenance/audit trail | Phase 1 | Every portfolio number can be traced to source, timestamp, and valuation inputs |
| Float/locale money math | Phase 1 | Schema uses exact decimals and golden tests prove stable totals after repeated recalculation |
| Asset matching by symbol | Phase 1-3 | Manual and synced assets resolve to canonical IDs with explicit mapping review |
| Stale/mixed freshness hidden | Phase 2-4 | Dashboard exposes sync time, price time, and stale-source warnings |
| Protected cash treated as ordinary capital | Phase 1 and 4 | Rebalance engine blocks invest-risk advice while protected cash is under target |
| Gross-value-only allocation math | Phase 4 | Analytics can switch between gross, investable, and liquid views |
| Naive rebalance diff engine | Phase 4 | Recommendations are ordered, thresholded, and conflict-free in test portfolios |
| Recomputation-based history | Phase 3 and 5 | Historical reports remain stable after changing mappings/rules going forward |
| Scheduler without idempotency/locking | Phase 2 | Overlapping job attempts do not create duplicate or mixed snapshots |
| Ignoring rate limits/time drift | Phase 2 | Connector test matrix covers per-exchange pacing/options/time adjustments |
| No strategy for unpriced assets | Phase 3 | Portfolio totals disclose unpriced/excluded assets and do not silently zero them |

## Sources

- Project context: `C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/PROJECT.md`
- CCXT manual via official GitHub wiki mirror — unified API coverage can be incomplete; capability and exchange-specific options matter. Confidence: HIGH. URL: https://raw.githubusercontent.com/wiki/ccxt/ccxt/Manual.md
- CoinGecko docs — symbol lookup priority is lowest; IDs should be preferred; price cache/update cadence and `last_updated_at` matter. Confidence: HIGH. URLs: https://docs.coingecko.com/reference/coins-markets and https://docs.coingecko.com/reference/simple-price
- CoinGecko historical pricing docs — daily history is anchored to 00:00:00 UTC and appears with delay after day close. Confidence: HIGH. URL: https://docs.coingecko.com/reference/coins-id-history
- PostgreSQL docs — `numeric` is recommended for exact monetary quantities; `money` is locale-sensitive and less suitable for multi-currency systems. Confidence: HIGH. URLs: https://www.postgresql.org/docs/current/datatype-numeric.html and https://www.postgresql.org/docs/current/datatype-money.html
- Domain synthesis from portfolio-management product patterns and failure modes specific to asset aggregation, protected cash modeling, and advisory-only rebalancing. Confidence: MEDIUM.

---
*Pitfalls research for: personal portfolio management and asset aggregation*
*Researched: 2026-04-20*
