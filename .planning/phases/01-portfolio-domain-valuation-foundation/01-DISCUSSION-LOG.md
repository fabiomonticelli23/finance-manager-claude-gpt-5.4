# Phase 1: Portfolio Domain & Valuation Foundation - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-04-20T19:31:45+02:00
**Phase:** 01-Portfolio Domain & Valuation Foundation
**Areas discussed:** Portfolio breakdown, Base currency control, Value visibility, Trust drilldown

---

## Portfolio breakdown

| Option | Description | Selected |
|--------|-------------|----------|
| Accounts + holdings | Separate top-level sections for accounts and holdings, each with line items and subtotals. | ✓ |
| Single asset list | Flatten everything into one list ordered by contribution to net worth. | |
| Summary first | Show category totals first and let the user drill into underlying lines later. | |

**User's choice:** You decide.
**Notes:** Claude applied the recommended default: separate accounts and holdings sections with line items and subtotals.

---

## Base currency control

| Option | Description | Selected |
|--------|-------------|----------|
| One global base | A single app-wide base currency controls all totals and line-item conversions. | ✓ |
| Per-screen override | Each screen can temporarily choose its own reporting currency. | |
| Per-widget override | Individual cards or tables can use different base currencies. | |

**User's choice:** You decide.
**Notes:** Claude applied the recommended default: one app-wide base currency that drives all totals and conversions.

---

## Value visibility

| Option | Description | Selected |
|--------|-------------|----------|
| Show both together | Every relevant row shows native value and converted base-currency value side by side. | ✓ |
| Converted first | Show base-currency value by default and reveal original value in expanded details. | |
| Original first | Show native amount by default and reveal converted value on demand. | |

**User's choice:** You decide.
**Notes:** Claude applied the recommended default: show original-currency and converted base-currency values together on relevant rows.

---

## Trust drilldown

| Option | Description | Selected |
|--------|-------------|----------|
| Simple breakdown | Show the original amount, price used, FX rate used, and converted result on demand. | ✓ |
| High-level only | Keep Phase 1 at totals and line items with no valuation inputs shown. | |
| Full audit trail | Expose every valuation input and derivation path in detail. | |

**User's choice:** You decide.
**Notes:** Claude applied the recommended default: a lightweight valuation breakdown for trust, not a full audit interface.

---

## Claude's Discretion

- Portfolio breakdown
- Base currency control
- Value visibility
- Trust drilldown

## Deferred Ideas

None.
