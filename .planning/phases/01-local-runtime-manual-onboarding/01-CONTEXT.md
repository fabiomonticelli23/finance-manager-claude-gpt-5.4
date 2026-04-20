# Phase 1: local-runtime-manual-onboarding - Context

**Gathered:** 2026-04-20
**Status:** Ready for planning

<domain>
## Phase Boundary

Establish the first runnable local version of the app and the first-run onboarding path for manual portfolio entry. This phase should let a single user start the stack locally, complete guided onboarding, create at least one real manual source, classify protected cash during setup, and land on a live dashboard with persisted data. Automated exchange sync and later portfolio automation remain outside this phase.

</domain>

<decisions>
## Implementation Decisions

### Local startup
- **D-01:** The primary local runtime is a one-command Docker Compose flow that starts frontend, backend, and database together.
- **D-02:** The app should only present itself as ready once the required services are healthy.
- **D-03:** First launch starts empty — no demo or sample portfolio data is preloaded.
- **D-04:** Portfolio data persists across restarts by default through persistent local storage.

### First-run onboarding
- **D-05:** First run begins with a guided onboarding wizard rather than an empty dashboard.
- **D-06:** Onboarding uses a short step-based flow with visible progress.
- **D-07:** Onboarding is considered complete once the user saves the first real manual source.
- **D-08:** After onboarding, the user lands directly on the live dashboard with their real data.

### Manual source setup
- **D-09:** Manual onboarding is source-first: create the manual source or account first, then attach balances or holdings to it.
- **D-10:** Manual source entry uses type-specific setup flows rather than one generic form.
- **D-11:** Each source requires only the minimum information needed to identify it and value it correctly during onboarding.
- **D-12:** After the first source is saved, onboarding can finish immediately; additional sources are added later from the main app.

### Protected cash setup
- **D-13:** Protected cash and liquidity buckets are classified during source setup, not deferred until later.
- **D-14:** Emergency-fund target information is captured during onboarding.
- **D-15:** If protected cash is saved before its target is fully defined, the source is saved but clearly marked incomplete.
- **D-16:** Protected-cash status must be shown prominently on the first post-onboarding dashboard.

### Claude's Discretion
- Exact wizard copy, step phrasing, and field ordering.
- Exact health-check implementation for local readiness.
- Exact dashboard layout, as long as protected cash status stays prominent and onboarding lands on real data.

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Product scope and constraints
- `.planning/PROJECT.md` — product scope, single-user local-first constraints, protected emergency-fund semantics, and fixed tech stack
- `.planning/REQUIREMENTS.md` — v1 requirements for manual sources, dashboards, rebalancing, and local Docker Compose runtime

### Phase and architecture baseline
- `.planning/research/SUMMARY.md` — recommended phase ordering and manual-first foundation for the product
- `.planning/research/ARCHITECTURE.md` — proposed module boundaries, local runtime shape, and manual asset workflow architecture
- `.planning/research/STACK.md` — stack baseline and containerized local runtime recommendations

### Trust and modeling constraints
- `.planning/research/PITFALLS.md` — provenance, exact-decimal math, canonical asset identity, and protected-cash modeling constraints
- `.planning/research/FEATURES.md` — onboarding expectations, manual asset support, trust UX, and deferred feature boundaries

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- No application source code exists yet; current reusable inputs are the planning and research artifacts under `.planning/`.
- `.planning/research/ARCHITECTURE.md` provides the recommended module split to scaffold against.
- `.planning/research/STACK.md` provides the agreed React + NestJS + PostgreSQL + Docker Compose baseline.

### Established Patterns
- The project is expected to start as a local-first modular monolith with separate frontend, backend, and database services.
- Manual-source-first delivery is the established build order before exchange sync automation.
- Trust-first modeling is already locked by research: exact decimals, provenance, and protected-cash semantics must be first-class from the start.

### Integration Points
- Local runtime should create coordinated frontend, backend, and database services through Docker Compose.
- Onboarding should create manual source records that later valuation, dashboard, and rebalancing phases can build on.
- Protected-cash classification and emergency-fund targets must exist in the initial data model so later analytics behave correctly.

</code_context>

<specifics>
## Specific Ideas

No external visual references were given. Use standard UI patterns as long as onboarding is short, lands on the real dashboard after the first saved source, and makes protected cash status clearly visible.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 01-local-runtime-manual-onboarding*
*Context gathered: 2026-04-20*
