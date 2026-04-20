# Domain Pitfalls

**Domain:** Single-user personal portfolio management dashboard with exchange ingestion, manual assets, market pricing, base-currency conversion, allocation analytics, emergency-fund protection, Trading Bot grouping, snapshots, and suggestions-only rebalancing
**Researched:** 2026-04-20
**Confidence:** MEDIUM

## Critical Pitfalls

### Pitfall 1: Treating "portfolio value" as a single number instead of a timestamped valuation
**What goes wrong:**
The app shows one clean total net worth number, but it mixes balances fetched at one time, market prices fetched at another time, and FX rates fetched at a third time. Users read the total as "current" even when parts are stale, partially refreshed, or failed silently.

**Why it happens:**
Dashboard projects optimize for a satisfying summary card early. Teams store only the latest numeric value instead of storing valuation context: source timestamp, pricing timestamp, FX timestamp, freshness status, and refresh outcome per asset/source.

**Consequences:**
- False confidence in net worth and allocation percentages
- Rebalancing suggestions based on stale prices or stale balances
- Charts that appear precise but are materially wrong
- User distrust after noticing numbers change significantly after refresh

**Prevention:**
- Persist balance snapshots, price snapshots, and FX snapshots with timestamps and source identifiers
- Compute portfolio valuation from an explicit valuation run record, not from ad hoc latest rows
- Show freshness state in the UI at portfolio, account, and asset level
- Mark totals as degraded when any required input is stale, missing, or partial
- Define freshness SLAs up front, e.g. balances fresher than X, prices fresher than Y, FX fresher than Z

**Detection:**
- Total net worth changes materially without any underlying balance change
- Refresh logs show mixed timestamps across balances/prices/FX in one portfolio render
- User asks "why did the pie chart move after I pressed refresh again?"
- Support/debugging requires reconstructing which prices were used and you cannot

**Phase to address:**
Phase 1 - domain model and ingestion foundation

---

### Pitfall 2: Using floating point for money, quantities, FX, or percentages
**What goes wrong:**
Valuations drift by cents or more, percentages do not add to 100%, tiny holdings disappear, and rebalancing suggestions oscillate because of rounding noise.

**Why it happens:**
Developers use JavaScript `number` end-to-end or PostgreSQL floating-point types because they are convenient. Finance dashboards often look fine in demos until many conversions and aggregations stack up.

**Consequences:**
- Incorrect totals and allocations
- Misleading "under target by 0.01%" or "sell 0.00000001 BTC" suggestions
- Snapshot diffs polluted by arithmetic noise
- Hard-to-reproduce bugs between backend and frontend calculations

**Prevention:**
- Store money and asset quantities in PostgreSQL `numeric`, not floating point
- Use a decimal library in application code for valuation math, allocation math, and FX conversion
- Define precision rules per concept: fiat amount, crypto quantity, market price, percentage, and display rounding
- Keep calculation precision higher than display precision
- Snap rebalancing outputs to practical minimums and tolerances

**Detection:**
- Percentages sum to 99.99% or 100.01% unpredictably
- Re-running the same valuation yields tiny differences
- Holdings with small quantities round to zero in one layer but not another
- Users see impossible recommendations driven by dust amounts

**Phase to address:**
Phase 1 - persistence and calculation primitives

---

### Pitfall 3: Broken symbol, instrument, or currency mapping
**What goes wrong:**
BTC on one exchange, BTC on another exchange, wrapped assets, broker tickers, ETFs, ETPs, fiat cash, stablecoins, and manual holdings are mapped inconsistently. The app merges things that should stay separate or splits things that should unify.

**Why it happens:**
Projects assume symbols are globally stable identifiers. They are not. Exchanges, brokers, and pricing providers all have their own identifiers, quote conventions, market metadata, and currency naming quirks.

**Consequences:**
- Wrong prices attached to holdings
- Duplicate assets in allocation dashboards
- Incorrect base-currency conversion
- Rebalancing suggestions against the wrong instrument
- Historical charts that break when symbol mappings change later

**Prevention:**
- Introduce a canonical instrument model separate from source-specific symbols
- Store source symbol, source market id, source account, and canonical asset/instrument mapping explicitly
- Require review/override for ambiguous mappings
- Version mappings so historical valuations can be reproduced
- Separate asset identity from account/category identity; BTC in long-term crypto and BTC inside Trading Bot may share asset identity but differ in portfolio bucket treatment

**Detection:**
- Same asset appears twice with different symbols or prices
- Holdings are present but priced at zero despite known market support
- One source reports a quantity and another source maps to a different quote currency unexpectedly
- Historical values change after you "fix" a symbol mapping

**Phase to address:**
Phase 1 - canonical asset model before dashboards

---

### Pitfall 4: Mixing balances, positions, valuations, and allocations into one table/model
**What goes wrong:**
The first version stores everything as "assets with a value". This seems productive until manual assets, market-priced securities, CCXT balances, emergency fund buckets, and Trading Bot grouping need different behavior.

**Why it happens:**
Portfolio apps are deceptively simple in the UI. The backend domain is not. A flat model feels fast initially but collapses when trying to explain where a number came from.

**Consequences:**
- Manual assets cannot coexist cleanly with market-priced holdings
- Hard to differentiate source balance vs derived valuation
- Snapshot history becomes non-reproducible
- Rebalancing logic gets entangled with ingestion details

**Prevention:**
Model at least these separately:
- Source account or source holding
- Canonical asset/instrument
- Price point
- FX rate
- Valuation run/result
- Allocation bucket/target
- Snapshot series
- Recommendation output
This separation is what keeps stale-data detection, replayability, and debugging manageable.

**Detection:**
- One table keeps gaining nullable columns for every new source type
- Recommendation code queries raw ingestion tables directly
- You cannot answer "is this number entered, fetched, converted, or derived?"

**Phase to address:**
Phase 1 - architecture and schema design

---

### Pitfall 5: Hiding refresh state or pretending partial refresh succeeded
**What goes wrong:**
The user clicks refresh, sees a spinner, then sees updated charts. But one exchange failed, one price source timed out, and FX remained stale. The UI implies success while the data is only partially updated.

**Why it happens:**
Projects optimize for a clean dashboard experience and suppress operational detail. In finance, operational detail is part of trust.

**Consequences:**
- User trusts a portfolio that is missing positions or old prices
- "Why is exchange A lower than yesterday?" turns into a debugging mystery
- Partial failures become silent corruption at the UX layer

**Prevention:**
- Make refresh a first-class workflow with run ids, statuses, counts, and per-source outcomes
- Distinguish success, partial success, failed, and stale-in-use states
- Show what updated, what did not, and when the app last had a full good refresh
- Allow users to inspect source-level refresh details from the dashboard

**Detection:**
- Refresh button always shows success unless there is a total outage
- Charts update even when one or more source fetches failed
- Logs show repeated source failures but no visible UI warning

**Phase to address:**
Phase 2 - sync orchestration and trust-focused UX

---

### Pitfall 6: Building snapshots that are not reproducible
**What goes wrong:**
The app stores daily totals or chart points only. Later, users ask what changed, why it changed, or why old numbers shifted after mapping fixes. You cannot reconstruct the historical state.

**Why it happens:**
Teams treat history as "just save total net worth every day". That is enough for a demo chart, not enough for trustworthy finance history.

**Consequences:**
- Historical trends shift retroactively with no audit trail
- Impossible to debug unexplained jumps
- Snapshot charts become decorative rather than dependable

**Prevention:**
- Snapshot the output of a valuation run with references to balances, prices, FX, and mappings used
- Store both aggregate totals and enough breakdown metadata for reconciliation
- If corrections happen later, mark whether history was restated or preserved as originally computed
- Add data lineage fields so every chart point is traceable

**Detection:**
- Old chart points change after refreshing current data
- Users ask what caused a jump and there is no attributable breakdown
- A snapshot stores only one number with no source context

**Phase to address:**
Phase 2 - history/snapshot implementation

---

### Pitfall 7: Emergency fund logic diluted inside generic cash handling
**What goes wrong:**
Emergency fund balances are treated as ordinary liquidity. Allocation dashboards then suggest reallocating protected cash into risk assets or fail to highlight underfunding clearly.

**Why it happens:**
Developers model all cash-like balances the same way and defer special handling. But your project explicitly makes emergency funds a protected allocation with target logic.

**Consequences:**
- Misleading recommendations
- Safety reserves appear investable
- Portfolio health looks better than it really is

**Prevention:**
- Model emergency fund as a first-class protected allocation bucket, not just a tag
- Support target-by-value and target-by-percentage as separate rule types
- Exclude protected reserves from reallocatable capital calculations unless user explicitly asks otherwise
- Add specific underfunded/overfunded status and explanation text

**Detection:**
- Rebalancing suggests deploying emergency-fund cash
- Allocation totals do not distinguish protected vs deployable liquidity
- Emergency-fund warnings disappear when generic liquidity increases elsewhere

**Phase to address:**
Phase 2 - allocation and rebalancing rules

---

### Pitfall 8: Trading Bot capital mixed with long-term holdings
**What goes wrong:**
Exchange accounts used for active trading are rolled into the same crypto bucket as long-term holdings. The dashboard shows a clean total but hides that some capital is operational, high-turnover, or strategy-bound.

**Why it happens:**
Portfolio consolidation logic usually groups by asset first. Your domain needs account-intent grouping as well.

**Consequences:**
- Allocation looks healthier or more stable than reality
- Rebalancing suggests moving assets that are actually tied to bot strategies
- Performance interpretation becomes misleading

**Prevention:**
- Bucket holdings by both canonical asset and strategic account grouping
- Treat Trading Bot as a macro category derived from account classification rules
- Keep reporting views for both "by asset" and "by strategic bucket"
- Ensure recommendations respect non-reallocatable or strategy-bound accounts

**Detection:**
- Large exchange balances distort long-term crypto allocation
- Bot accounts show up as regular investment positions in recommendations
- Users mentally subtract bot capital from the dashboard to understand reality

**Phase to address:**
Phase 2 - portfolio taxonomy and analytics

---

### Pitfall 9: Suggestion-only rebalancing that still feels operationally unsafe
**What goes wrong:**
The app does not place trades, but its recommendations imply executable precision, ignore liquidity/fees/minimums, or suggest crossing protected boundaries. Users may act on low-quality guidance as if it were authoritative.

**Why it happens:**
"Suggestions only" sounds low risk, so teams under-design recommendation safety. But advisory output still shapes user decisions.

**Consequences:**
- Misleading action plans
- Frequent recommendation churn from tiny market moves
- User blame when following suggestions leads to poor outcomes

**Prevention:**
- Present suggestions as scenario guidance, not orders
- Add thresholds, deadbands, and minimum meaningful trade sizes
- Explain assumptions: excluded accounts, stale inputs, fees not included, prices as-of timestamp
- Separate target-gap analysis from execution advice
- Never generate suggestions when freshness or mapping confidence is below threshold

**Detection:**
- Recommendations change on every refresh by tiny amounts
- Suggested moves involve dust quantities or protected funds
- Users cannot tell whether advice reflects stale or partial data

**Phase to address:**
Phase 3 - rebalancing UX and recommendation engine

---

### Pitfall 10: Unsafe storage and handling of exchange API credentials
**What goes wrong:**
API keys end up in source control, plaintext env files, browser local storage, logs, screenshots, or unencrypted database rows. Read-only assumptions are not validated. Keys are over-privileged and long-lived.

**Why it happens:**
Single-user local apps often relax discipline because deployment feels private. In reality, local backups, shell history, Docker inspect output, crash logs, and repo history leak secrets easily.

**Consequences:**
- Exchange account compromise or data exposure
- Inability to rotate without breaking ingestion
- Silent credential leakage through support/debug artifacts

**Prevention:**
- Never expose exchange secrets to the browser
- Keep secret use server-side only
- Avoid committing secrets or baking them into images
- Prefer protected secret storage over plaintext files; if local file/config storage is unavoidable, encrypt at rest and lock file permissions down
- Use least-privilege exchange keys, ideally read-only and IP-restricted where supported
- Design rotation and revocation into the credential model from day one
- Redact secrets from logs, errors, health checks, and telemetry

**Detection:**
- Secrets appear in `.env`, compose files, screenshots, logs, or browser storage
- App requires trade/withdraw permissions for read-only ingestion
- Rotating a key requires manual DB surgery or code changes

**Phase to address:**
Phase 1 - credentials architecture; Phase 2 - operational hardening

---

### Pitfall 11: Misleading charts caused by bad aggregation semantics
**What goes wrong:**
Pie charts and bar charts look polished but combine incomparable concepts: market value with manual estimates, protected cash with deployable cash, stale sources with fresh sources, or categories that changed definition over time.

**Why it happens:**
Dashboard work starts from chart components instead of semantic definitions. Finance charts are persuasive even when wrong.

**Consequences:**
- Users make allocation decisions from distorted visuals
- Trend lines imply precision that the underlying data does not support
- Category changes break chart comparability across time

**Prevention:**
- Define chart semantics in the domain model first: what is included, excluded, protected, stale, and estimated
- Visually distinguish estimated/manual/stale values from fresh market-valued amounts
- Version category definitions and preserve historical meaning where possible
- Avoid pie charts when slices are tiny or semantically mixed; use bars/tables with labels instead

**Detection:**
- Chart totals do not reconcile to the details table
- Category percentages jump after taxonomy changes rather than market moves
- Users misread deployable capital because protected cash is visually merged

**Phase to address:**
Phase 3 - dashboard visualization and analytics semantics

---

### Pitfall 12: No reconciliation workflow for missing, zero, or absurd prices
**What goes wrong:**
A holding is imported successfully but valued at zero, NaN, or an obviously wrong amount because the price fetch failed or the wrong market was selected. The dashboard still renders without forcing review.

**Why it happens:**
Many apps assume price lookup is a solved problem once a symbol is present. Real portfolios always contain edge cases: illiquid securities, delisted instruments, unsupported venues, stablecoins, wrapped assets, cash-like instruments, and manual estimates.

**Consequences:**
- Total net worth is understated or overstated
- Allocation advice becomes garbage while still looking numeric
- Users lose trust after spotting one absurd valuation

**Prevention:**
- Introduce valuation status states: priced, stale, missing, manual-estimate, blocked, ambiguous
- Require explicit fallback behavior for unpriced assets
- Add outlier detection and threshold alerts for sudden valuation jumps
- Provide a reconciliation queue for unresolved pricing/mapping issues

**Detection:**
- Assets with quantity > 0 and valuation = 0
- One-day net-worth jumps far beyond plausible market movement
- Frequent manual corrections outside the app after export/checking

**Phase to address:**
Phase 2 - pricing pipeline and reconciliation UX

## Technical Debt Patterns

Shortcuts that seem helpful early but become expensive fast.

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Store only current portfolio totals, not valuation-run lineage | Fast charts | No trust, no debugging, no reproducible history | Never |
| Use a single generic `symbol` field everywhere | Simple ingestion | Broken mappings, duplicate assets, bad pricing | Never |
| Keep finance math in frontend JS numbers | Fast UI iteration | Inconsistent totals and rounding bugs | Only for display-only derived labels, never for source-of-truth calculations |
| Treat manual assets as "just another fetched account" | Fewer models | Cannot support market-priced manual holdings or clear provenance | Never |
| Hide partial refresh errors to keep UI clean | Nicer demo | User distrust and silent stale data | Never |
| Put API keys in `.env` and call it done | Easy local setup | Secret leakage via files, process env, Docker inspection, backups | Only as a temporary local developer setup with non-production throwaway keys |
| Hard-code category logic in charts | Fast dashboard | Taxonomy changes break history and consistency | Only for throwaway prototypes |

## Integration Gotchas

Common mistakes when connecting to external services.

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| CCXT exchange balances | Assuming unified methods remove exchange-specific quirks | Treat CCXT as a unification layer, not a guarantee of identical semantics; capture exchange id, source payload references, and per-exchange exception handling |
| CCXT market metadata | Using symbols as stable identifiers without persisting source market identity | Persist canonical asset mapping plus exchange/source market identifiers and mapping confidence |
| CCXT refresh cadence | Polling all accounts/prices on every page load | Separate scheduled sync from user-triggered refresh and rate-limit work by source and freshness policy |
| Market price provider for securities | Assuming one provider covers every stock/ETF/ETP/currency needed | Build a fallback/reconciliation path for unsupported or ambiguous instruments |
| FX conversion | Converting through inconsistent FX timestamps or implicit provider defaults | Treat FX as its own timestamped dataset with explicit source, pair, and freshness |
| Dockerized local deployment | Passing secrets through compose and assuming they stay private | Minimize secret exposure, avoid logging env values, and document safe local secret handling |
| Snapshot jobs | Running snapshots before all dependencies finish refreshing | Trigger snapshots from completed valuation runs, not wall-clock alone |

## Performance Traps

Patterns that work at small scale but fail even in a single-user finance app.

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Recomputing full valuation from live sources on every dashboard visit | Slow dashboard, source throttling, inconsistent totals | Persist valuation runs and serve UI from stored results with explicit freshness | Breaks immediately once multiple sources or expensive price fetches are added |
| N+1 pricing and FX lookups per holding | Refresh takes longer as asset count grows | Batch pricing and FX resolution per valuation run | Breaks at tens to low hundreds of holdings |
| Rendering every chart from raw transactional-like rows | UI jank and hard-to-debug mismatches | Pre-aggregate snapshot and allocation views server-side | Breaks once history spans months with multiple categories |
| Overwriting "latest" state instead of appending immutable snapshots | Easy writes, impossible debugging | Use append-only run/snapshot records plus current-materialized views | Breaks the first time user asks why a number changed |

## Security Mistakes

Domain-specific security issues beyond generic web security.

| Mistake | Risk | Prevention |
|---------|------|------------|
| Storing exchange API keys in source control, Docker images, or browser storage | Account exposure and irreversible secret leakage | Keep secrets server-side only, out of git/images/browser, with encryption and restricted file access where local storage is necessary |
| Using exchange keys with trade or withdrawal permissions for a read-only dashboard | Operational and financial loss if compromised | Create read-only least-privilege keys and document permission checks during setup |
| Logging raw refresh errors that include credentials, request headers, or account identifiers | Secret leakage via logs and screenshots | Centralize redaction and scrub all sensitive fields before logging |
| Assuming "single-user local" means no threat model is needed | Underestimating backups, malware, shared machines, and accidental disclosure | Document a local threat model and design for compromise containment |
| Sending secrets to the frontend to perform direct exchange calls | Full credential exposure to browser/runtime | Keep all exchange communication in NestJS backend only |
| No credential rotation or revoke path | Long-lived exposure after leak | Store credentials with lifecycle metadata and support replace/test/disable flows |

## UX Pitfalls

Common user experience mistakes in this domain.

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Showing a beautiful total without freshness metadata | User trusts stale or partial data | Show last successful full refresh, per-source freshness, and degraded-state warnings |
| Using red/green changes without explaining whether move is market, FX, or manual edit driven | Users misinterpret why portfolio changed | Provide change attribution where possible and label unknown causes honestly |
| Presenting protected emergency-fund cash as ordinary liquidity | User thinks more capital is safely deployable than actually is | Visually separate protected reserves from reallocatable funds |
| Rebalancing output reads like executable orders | User over-trusts advisory numbers | Label as suggestions with assumptions, thresholds, and excluded costs/constraints |
| Hiding zero-priced or unpriced assets deep in tables | Portfolio appears cleaner but wrong | Surface unresolved valuation issues prominently in dashboard health/status |
| Pie charts with many tiny slices or mixed semantics | Hard to read and easy to misinterpret | Use grouped bars/tables and clear category definitions |
| Manual assets and fetched assets look identical | User cannot tell confidence/provenance | Display source badges such as manual, fetched, estimated, stale |

## Looks Done But Isn't Checklist

Things that look complete in demos but are not production-trustworthy yet.

- [ ] **Net worth total:** Verify it is tied to a valuation run with timestamps for balances, prices, and FX, not just a latest number.
- [ ] **Refresh button:** Verify it can report partial success, per-source failures, and last known good full refresh.
- [ ] **CCXT ingestion:** Verify every connected account is classified correctly, including Trading Bot vs long-term holdings.
- [ ] **Symbol mapping:** Verify ambiguous or unsupported symbols go to a reconciliation queue instead of silently defaulting.
- [ ] **Valuation:** Verify assets with quantity but missing price cannot disappear quietly from totals.
- [ ] **Base-currency conversion:** Verify FX timestamps, source, and fallback behavior are visible and testable.
- [ ] **Emergency fund:** Verify protected balances are excluded from reallocatable capital and have under/over-target logic.
- [ ] **Allocation charts:** Verify chart totals reconcile to detailed holdings and category definitions are versioned.
- [ ] **Snapshots:** Verify historical points can be explained from stored source inputs or run lineage.
- [ ] **Rebalancing suggestions:** Verify thresholds, dust suppression, and stale-data blocking exist.
- [ ] **Precision:** Verify backend/database/frontend use exact-decimal handling for finance math.
- [ ] **Credentials:** Verify exchange secrets never reach browser storage, logs, git history, or container image layers.
- [ ] **Manual assets:** Verify provenance is visible and market-priced manual holdings can use fetched prices without losing manual override capability.
- [ ] **Health status:** Verify dashboard can show degraded trust state, not just render whatever data is available.

## Recovery Strategies

When pitfalls happen anyway, how to recover without guessing.

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Stale/partial valuation shown as current | MEDIUM | Introduce valuation-run records, backfill timestamps where possible, mark prior snapshots as low-confidence, and update UI to show degraded states |
| Precision bugs from float usage | HIGH | Migrate storage/calculation paths to exact decimals, rebuild affected aggregates/snapshots, add golden test vectors, and explicitly communicate any restated history |
| Broken symbol/instrument mapping | HIGH | Freeze current mappings, create canonical mapping table with versioning, reprice affected holdings, and annotate any restated historical data |
| Unsafe API credential handling | HIGH | Revoke and rotate keys immediately, scrub logs/history/backups where feasible, move secrets server-side/protected storage, and add permission validation on reconnect |
| Snapshot history not reproducible | MEDIUM | Start storing valuation lineage now, preserve old snapshots as legacy/non-reproducible, and avoid pretending historical detail exists when it does not |
| Misclassified emergency fund or Trading Bot balances | MEDIUM | Reclassify buckets with migration scripts, restate affected allocations/suggestions, and provide a visible explanation for changed historical analytics |
| Misleading charts from semantic changes | MEDIUM | Version category definitions, add chart notes for restated periods, and prefer tables/bars until semantics are stable |

## Pitfall-to-Phase Mapping

How roadmap phases should prevent these pitfalls.

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Timestamp-free valuations | Phase 1 - domain model and valuation foundations | Every displayed number links to a valuation run with balance/price/FX timestamps |
| Precision/rounding errors | Phase 1 - persistence and math primitives | Golden tests for money/quantity/FX math pass end-to-end using exact decimals |
| Symbol/instrument mapping errors | Phase 1 - canonical asset model | Ambiguous mappings are blocked or queued; no silent default mapping |
| Flat "everything is an asset" schema | Phase 1 - architecture/schema | Separate entities exist for holdings, prices, FX, valuation runs, snapshots, and recommendations |
| Hidden partial refresh | Phase 2 - sync orchestration and UX | UI can show success, partial success, failure, stale-in-use, and per-source details |
| Non-reproducible snapshots | Phase 2 - history implementation | Historical points can be traced back to run metadata and component inputs |
| Emergency fund dilution | Phase 2 - allocation rules | Protected cash is visible separately and excluded from deployable capital calculations |
| Trading Bot mixing | Phase 2 - taxonomy/account classification | Bot accounts appear as their own macro category in analytics and suggestions |
| Missing/absurd price reconciliation | Phase 2 - pricing pipeline | Holdings with unresolved pricing are surfaced prominently and excluded/flagged per policy |
| Unsafe or misleading rebalancing suggestions | Phase 3 - recommendation engine and UX | Suggestions respect thresholds, protected buckets, stale-data blocking, and explanatory assumptions |
| Misleading charts | Phase 3 - dashboard analytics | Chart totals reconcile to detail views and disclose stale/estimated/protected semantics |
| Credential leakage and poor rotation | Phase 1 plus Phase 2 hardening | Secrets stay server-side, logs are redacted, and rotate/revoke flows are testable |

## Sources

- Project context: `C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/PROJECT.md` - HIGH confidence for project-specific scope and constraints
- PostgreSQL official docs, Numeric Types: https://www.postgresql.org/docs/current/datatype-numeric.html - HIGH confidence for exact numeric vs floating-point guidance
- OWASP Cryptographic Storage Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html - HIGH confidence for secret storage and key-handling guidance
- OWASP Secrets Management Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html - HIGH confidence for lifecycle, auditability, and env-var caveats
- CCXT Manual: https://github.com/ccxt/ccxt/wiki/Manual - MEDIUM confidence here because the accessible excerpt confirmed the unified-vs-exchange-specific model, but not every operational nuance requested
- Domain synthesis from portfolio/dashboard implementation patterns - MEDIUM confidence; practical and opinionated, but some items are based on engineering judgment rather than directly quoted official documentation
