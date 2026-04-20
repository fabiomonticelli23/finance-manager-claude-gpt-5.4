<!-- GSD:project-start source:PROJECT.md -->
## Project

Personal Portfolio Manager

Single-user local-first web app for consolidating CCXT exchange balances and manual assets into one portfolio view with base-currency net worth, allocation analysis, protected emergency fund handling, Trading Bot grouping, and suggestions-only rebalancing.
<!-- GSD:project-end -->

<!-- GSD:stack-start source:STACK.md -->
## Technology Stack

- Backend: NestJS 11 + Prisma 7 + PostgreSQL 18
- Frontend: React 19 + Vite 8 + TanStack Query + React Hook Form
- Integrations: CCXT 4.5.50 for exchange sync, yahoo-finance2 3.14.0 for manual security pricing
- Data handling: PostgreSQL numeric + Prisma Decimal + decimal.js for financial math
- Ops/testing: Docker Compose, Playwright, Vitest, Supertest, Testcontainers
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Use a modular monolith:
- Backend modules: sources, pricing, portfolio, allocation, policies, rebalancing, snapshots, settings
- Data flow: ingestion -> normalization -> pricing/FX valuation -> allocation/policy -> rebalancing -> snapshots -> dashboard reads
- Keep portfolio truth in the backend; frontend stays presentation-focused and should not recalculate canonical financial values.
- Treat emergency fund protection and Trading Bot grouping as first-class domain rules.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->
## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, or `.github/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->

<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` — do not edit manually.
<!-- GSD:profile-end -->