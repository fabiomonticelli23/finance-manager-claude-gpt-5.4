---
phase: 01
slug: portfolio-domain-valuation-foundation
status: approved
nyquist_compliant: true
wave_0_complete: true
created: 2026-04-20
updated: 2026-04-20
---

# Phase 01 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Planned in-phase: Nest + Supertest (01-01), Vitest (01-04), and Playwright (01-05) |
| **Config file** | `backend/vitest.config.ts`, `frontend/vitest.config.ts`, `e2e/playwright.config.ts` |
| **Quick run command** | `npm run test:ui -- portfolio --runInBand` |
| **Focused backend command** | `npm run test:backend -- portfolio --runInBand` |
| **Full suite command** | `npm run test:backend -- settings -- portfolio --runInBand && npm run test:ui -- portfolio --runInBand && npm run test:e2e -- portfolio` |
| **Estimated runtime** | ~60 seconds for quick smoke, ~120 seconds for full suite |

---

## Sampling Rate

- **After every task commit:** Run that task’s `<automated>` command.
- **During common UI iteration loops:** Run `npm run test:ui -- portfolio --runInBand` first.
- **During portfolio backend iteration loops:** Run `npm run test:backend -- portfolio --runInBand`.
- **After every plan wave:** Run the wave’s relevant backend/UI suite.
- **Before `/gsd-verify-work`:** Full suite must be green.
- **Max feedback latency:** 60 seconds for the preferred quick smoke path.

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Threat Ref | Secure Behavior | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|------------|-----------------|-----------|-------------------|-------------|--------|
| 01-01-01 | 01 | 1 | PORT-01 | T-01-01 | Backend request validation, workspace scripts, and runnable HTTP scaffold exist before settings/portfolio routes are added | integration | `npm run test:backend -- --runInBand` | ✅ planned | ⬜ pending |
| 01-02-01 | 02 | 2 | PORT-01 | T-01-04 | Unsupported base-currency inputs are rejected and valid values persist app-wide through the same task that creates schema, seed, and settings endpoints | integration | `npm run test:backend -- settings --runInBand` | ✅ planned | ⬜ pending |
| 01-03-01 | 03 | 3 | PORT-02, PORT-03 | T-01-07 | Canonical portfolio totals, breakdown data, and decimal-string serialization remain backend-owned and exact through one verifiable valuation task | integration | `npm run test:backend -- portfolio --runInBand` | ✅ planned | ⬜ pending |
| 01-04-01 | 04 | 4 | PORT-01, PORT-03 | T-01-12 | Frontend scaffold, query provider, and typed canonical API boundaries have fast non-watch feedback before detailed page rendering | component | `npm run test:ui -- --runInBand` | ✅ planned | ⬜ pending |
| 01-05-01 | 05 | 5 | PORT-01, PORT-02, PORT-03 | T-01-13 | Base-currency save flow, visible selected-base-currency net worth, side-by-side values, and row breakdown UI work without local canonical recomputation | component + e2e | `npm run test:ui -- portfolio --runInBand` and `npm run test:e2e -- portfolio` | ✅ planned | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [x] `backend/test/` — Nest + Supertest test bootstrap for settings and portfolio endpoints
- [x] `frontend/src/**/*.test.tsx` — Vitest component/hook coverage for portfolio rendering and base-currency interactions
- [x] `e2e/` — Playwright setup for base-currency workflow and valuation breakdown checks
- [x] `backend/prisma/seed.ts` — deterministic seed data with exact-valued settings, accounts, holdings, prices, and FX rates
- [x] Decimal assertion path that compares string or Decimal outputs without float conversion

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Valuation breakdown interaction feels lightweight and understandable | PORT-03 | Interaction design quality and readability still need human review | Open the portfolio page, inspect an account and a holding row, open the valuation breakdown, and confirm the UI shows original amount, price used, FX rate, and converted result without navigating away |

---

## Validation Sign-Off

- [x] All tasks have `<automated>` verify or valid prior-plan dependency coverage
- [x] Sampling continuity: no 3 consecutive tasks without automated verify
- [x] Wave 0 coverage is complete for backend, frontend, e2e, and deterministic seed data
- [x] No watch-mode flags
- [x] Feedback latency < 120s
- [x] `nyquist_compliant: true` set in frontmatter

**Approval:** revised and ready for execution
