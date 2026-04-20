# Phase 1: Portfolio Domain & Valuation Foundation - Context

**Gathered:** 2026-04-20
**Status:** Ready for planning

<domain>
## Phase Boundary

Establish the canonical portfolio and valuation foundation so users can rely on one base-currency portfolio view, see total net worth in the selected base currency, and inspect original-currency plus converted values for accounts and holdings. Source connectivity, sync freshness, dashboards, allocation targets, and rebalancing are separate later phases.

</domain>

<decisions>
## Implementation Decisions

### Portfolio structure
- **D-01:** The canonical portfolio should be inspectable through separate top-level accounts and holdings sections, each with line items and subtotals.
- **D-02:** Summary views may roll up from those sections, but the canonical model should not flatten everything into one undifferentiated asset list.

### Base currency behavior
- **D-03:** A single app-wide base currency drives all totals and conversions in Phase 1; screens and widgets should not override it independently.

### Value presentation
- **D-04:** Relevant portfolio rows should show original-currency values and converted base-currency values together by default instead of hiding one behind expansion.

### Valuation transparency
- **D-05:** Users should be able to open a lightweight valuation breakdown on demand showing the original amount, price used, FX rate used, and converted result.
- **D-06:** Phase 1 should stop short of a full audit-trail interface; the transparency feature is for trust and explanation, not exhaustive lineage.

### Claude's Discretion
- Exact layout for accounts versus holdings sections
- Where the base currency control lives in the UI
- Formatting and density of side-by-side native and converted values
- Exact interaction pattern for opening the valuation breakdown

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Phase scope and requirements
- `.planning/ROADMAP.md` — Phase 1 goal, dependency boundary, and success criteria
- `.planning/REQUIREMENTS.md` — PORT-01, PORT-02, and PORT-03 definitions plus v1 scope boundaries
- `.planning/PROJECT.md` — product context, portfolio semantics, and v1 constraints

### Architecture guardrails
- `CLAUDE.md` — modular-monolith architecture, backend-as-canonical-truth rule, and finance math stack constraints

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- No application source files are present yet; Phase 1 will establish the first concrete backend and frontend assets for this project.

### Established Patterns
- Portfolio truth must live in the backend; the frontend should present canonical values rather than recalculate them.
- Financial values should use the project's exact-number stack: PostgreSQL numeric, Prisma Decimal, and decimal.js.
- The product is being built as a modular monolith with dedicated portfolio and pricing modules.

### Integration Points
- The Phase 1 model becomes the base for later source ingestion, pricing/FX valuation, allocation semantics, rebalancing, snapshots, and dashboard reads.
- Base-currency selection established here must feed later dashboard, allocation, and history views consistently.

</code_context>

<specifics>
## Specific Ideas

- Trust in totals is the primary product value for this phase, so transparency wins over minimal UI.
- The user delegated detailed implementation choices to Claude; standard trustworthy defaults are preferred over bespoke UX.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 01-portfolio-domain-valuation-foundation*
*Context gathered: 2026-04-20*
