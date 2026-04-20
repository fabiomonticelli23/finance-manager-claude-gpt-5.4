# Phase 1: Portfolio Domain & Valuation Foundation - Research

**Researched:** 2026-04-20
**Domain:** Canonical portfolio modeling, exact valuation, and first-phase API/UI foundation
**Confidence:** MEDIUM

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
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

### Deferred Ideas (OUT OF SCOPE)
None — discussion stayed within phase scope.
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| PORT-01 | User can choose a base currency for portfolio reporting | App-wide settings-backed base currency, backend valuation service, and settings endpoint pattern |
| PORT-02 | User can view total net worth across all assets in the selected base currency | Canonical portfolio aggregate response with backend-computed totals and subtotals |
| PORT-03 | User can view original currency values and converted base-currency values for accounts and holdings | Row DTO shape carrying native values, converted values, and on-demand valuation breakdown fields |
</phase_requirements>

## Summary

Phase 1 should establish a backend-owned canonical portfolio read model, not just a few totals on an endpoint. The project context already locks in that portfolio truth lives in the backend and that exact math must use PostgreSQL `numeric`, Prisma `Decimal`, and `decimal.js` [VERIFIED: CLAUDE.md] [CITED: https://www.postgresql.org/docs/current/datatype-numeric.html] [CITED: https://www.prisma.io/docs/orm/prisma-client/special-fields-and-types#working-with-decimal]. That means Phase 1 planning should start from durable domain shapes and valuation contracts rather than from UI screens.

The safest Phase 1 boundary is: one global base currency setting, a small settings surface to change it, a pricing/valuation pipeline that can convert native values into the selected base currency with exact decimal math, and one canonical portfolio API that returns accounts, holdings, subtotals, and total net worth already computed by the backend [VERIFIED: 01-CONTEXT.md] [VERIFIED: CLAUDE.md]. Frontend work should stay presentation-focused and use TanStack Query for server-state reads instead of duplicating valuation logic client-side [VERIFIED: CLAUDE.md] [CITED: https://tanstack.com/query/latest/docs/framework/react/overview].

The main planning risk is shallow modeling. If Phase 1 treats everything as a flat “asset list,” or if converted values are computed in React with `number`, later phases for allocation, emergency-fund semantics, Trading Bot grouping, snapshots, and rebalancing will need expensive rewrites [VERIFIED: 01-CONTEXT.md] [VERIFIED: PROJECT.md] [CITED: https://www.postgresql.org/docs/current/datatype-numeric.html].

**Primary recommendation:** Build Phase 1 around a backend-computed canonical portfolio snapshot DTO with separate `accounts` and `holdings` sections, exact decimal valuation services, and a single app-wide base-currency setting.

## Architectural Responsibility Map

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| Base currency setting persistence | API / Backend | Database / Storage | The base currency is app-wide and must drive all canonical totals consistently [VERIFIED: 01-CONTEXT.md]. |
| Base currency selection UI | Browser / Client | API / Backend | The client provides the control, but persisted state and recalculation authority belong to the backend [VERIFIED: CLAUDE.md]. |
| Native-to-base valuation math | API / Backend | Database / Storage | Canonical financial values must not be recalculated in the frontend; exact math stack is backend-oriented [VERIFIED: CLAUDE.md]. |
| Portfolio account and holding aggregation | API / Backend | Database / Storage | The canonical portfolio view is a composed read model over persisted entities [VERIFIED: ROADMAP.md] [VERIFIED: 01-CONTEXT.md]. |
| Exact decimal storage | Database / Storage | API / Backend | PostgreSQL `numeric` is the exact storage type; Prisma/decimal.js are the application bridge [CITED: https://www.postgresql.org/docs/current/datatype-numeric.html] [CITED: https://www.prisma.io/docs/orm/reference/prisma-schema-reference#decimal]. |
| Valuation breakdown display | Browser / Client | API / Backend | The backend should supply the explanation fields; the frontend only reveals them on demand [VERIFIED: 01-CONTEXT.md] [VERIFIED: CLAUDE.md]. |
| Portfolio totals and subtotals API contract | API / Backend | Browser / Client | The backend owns summary computation; the frontend renders those returned sections and totals [VERIFIED: CLAUDE.md]. |

## Project Constraints (from CLAUDE.md)

- Use the defined stack: NestJS 11 + Prisma 7 + PostgreSQL 18 for backend, React 19 + Vite 8 + TanStack Query + React Hook Form for frontend [VERIFIED: CLAUDE.md].
- Keep the application as a modular monolith with backend modules including `pricing`, `portfolio`, and `settings` [VERIFIED: CLAUDE.md].
- Keep portfolio truth in the backend; the frontend should not recalculate canonical financial values [VERIFIED: CLAUDE.md].
- Treat financial math as exact arithmetic using PostgreSQL `numeric`, Prisma `Decimal`, and `decimal.js` [VERIFIED: CLAUDE.md].
- Phase 1 establishes the first real app/backend/frontend assets because no application source files are present yet [VERIFIED: 01-CONTEXT.md] [VERIFIED: repo glob scan].
- Do not recommend approaches that assume multi-user auth or SaaS account semantics for v1 because the product is single-user and local-first [VERIFIED: PROJECT.md].

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| `@nestjs/common` | 11.1.19 | HTTP controllers, providers, modules, pipes | NestJS is the project’s required backend framework and its module/controller/provider model fits a modular monolith [VERIFIED: npm registry modified 2026-04-13] [VERIFIED: CLAUDE.md] [CITED: https://docs.nestjs.com/controllers] |
| `prisma` | 7.7.0 | Schema, migrations, generated ORM client | Prisma is the required ORM and supports Decimal fields plus relation loading for nested portfolio read models [VERIFIED: npm registry modified 2026-04-20] [VERIFIED: CLAUDE.md] [CITED: https://www.prisma.io/docs/orm/reference/prisma-schema-reference#decimal] [CITED: https://www.prisma.io/docs/orm/prisma-client/queries/relation-queries] |
| `@prisma/client` | 7.7.0 | Runtime database client | Required to query accounts, holdings, and related valuation data from Nest services [VERIFIED: npm registry modified 2026-04-20] [CITED: https://www.prisma.io/docs/orm/prisma-client/queries/relation-queries] |
| `decimal.js` | 10.6.0 | Exact arithmetic in application code | Prisma Decimal values are backed by Decimal.js and the project explicitly chose it for financial math [VERIFIED: npm registry modified 2025-07-06] [VERIFIED: CLAUDE.md] [CITED: https://www.prisma.io/docs/orm/prisma-client/special-fields-and-types#working-with-decimal] [CITED: https://mikemcl.github.io/decimal.js/] |
| `react` | 19.2.5 | Frontend rendering | Required frontend runtime for the first portfolio UI [VERIFIED: npm registry modified 2026-04-17] [VERIFIED: CLAUDE.md]. |
| `vite` | 8.0.9 | Frontend dev/build tool | Vite is the required frontend build tool and its fast dev server/HMR suit first-phase UI setup [VERIFIED: npm registry modified 2026-04-20] [VERIFIED: CLAUDE.md] [CITED: https://vite.dev/guide/] |
| `@tanstack/react-query` | 5.99.2 | Server-state fetching and caching | Officially designed for server state and lets React consume backend portfolio responses without custom caching [VERIFIED: npm registry modified 2026-04-19] [VERIFIED: CLAUDE.md] [CITED: https://tanstack.com/query/latest/docs/framework/react/overview] |
| `react-hook-form` | 7.72.1 | Base currency/settings form state | Standard lightweight form management for settings controls and later source forms [VERIFIED: npm registry modified 2026-04-18] [VERIFIED: CLAUDE.md] [CITED: https://react-hook-form.com/docs/useform] |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| `class-validator` | 0.15.1 | DTO validation decorators in Nest request layer | Use if the implementation follows the common Nest `ValidationPipe` + class-based DTO path [VERIFIED: npm registry modified 2026-02-26] [ASSUMED] |
| `class-transformer` | 0.5.1 | DTO transformation/serialization companion | Use alongside class-based DTO validation/serialization if chosen in Wave 0 [VERIFIED: npm registry modified 2022-12-09] [ASSUMED] |
| `vitest` | 4.1.4 | Frontend/unit tests | Use for fast component, formatting, and hook tests [VERIFIED: npm registry modified 2026-04-09] [VERIFIED: CLAUDE.md]. |
| `playwright` | 1.59.1 | Browser workflow validation | Use for end-to-end verification of base-currency switching and side-by-side value display [VERIFIED: npm registry modified 2026-04-20] [VERIFIED: CLAUDE.md]. |
| `supertest` | 7.2.2 | Backend endpoint tests | Use for Nest HTTP contract tests around portfolio and settings endpoints [VERIFIED: npm registry modified 2026-01-06] [VERIFIED: CLAUDE.md]. |
| `testcontainers` | 11.14.0 | Real Postgres-backed integration tests | Use when verifying Prisma + PostgreSQL numeric behavior end to end [VERIFIED: npm registry modified 2026-04-08] [VERIFIED: CLAUDE.md]. |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| TanStack Query | Plain `fetch` + React state | Simpler at first, but you would hand-roll cache, staleness, retries, and refetch behavior for server state [VERIFIED: stack constraint] [CITED: https://tanstack.com/query/latest/docs/framework/react/overview]. |
| Prisma relations with `include`/`select` | Manual SQL shaping in this phase | Raw SQL can be appropriate later for heavy reporting, but Prisma is sufficient and lower-friction for the first canonical read model [CITED: https://www.prisma.io/docs/orm/prisma-client/queries/relation-queries] [ASSUMED]. |
| React Hook Form | Hand-managed controlled inputs | RHF reduces re-renders and keeps settings forms light, which matches the project stack and later admin-style forms [VERIFIED: stack constraint] [CITED: https://react-hook-form.com/docs/useform]. |

**Installation:**
```bash
npm install @nestjs/common prisma @prisma/client decimal.js react vite @tanstack/react-query react-hook-form class-validator class-transformer
npm install -D vitest playwright supertest testcontainers
```

**Version verification:**
- `@nestjs/common` 11.1.19 — registry modified 2026-04-13 [VERIFIED: npm registry]
- `prisma` 7.7.0 — registry modified 2026-04-20 [VERIFIED: npm registry]
- `@prisma/client` 7.7.0 — registry modified 2026-04-20 [VERIFIED: npm registry]
- `decimal.js` 10.6.0 — registry modified 2025-07-06 [VERIFIED: npm registry]
- `react` 19.2.5 — registry modified 2026-04-17 [VERIFIED: npm registry]
- `vite` 8.0.9 — registry modified 2026-04-20 [VERIFIED: npm registry]
- `@tanstack/react-query` 5.99.2 — registry modified 2026-04-19 [VERIFIED: npm registry]
- `react-hook-form` 7.72.1 — registry modified 2026-04-18 [VERIFIED: npm registry]
- `vitest` 4.1.4 — registry modified 2026-04-09 [VERIFIED: npm registry]
- `playwright` 1.59.1 — registry modified 2026-04-20 [VERIFIED: npm registry]
- `supertest` 7.2.2 — registry modified 2026-01-06 [VERIFIED: npm registry]
- `testcontainers` 11.14.0 — registry modified 2026-04-08 [VERIFIED: npm registry]

## Architecture Patterns

### System Architecture Diagram

```text
[User selects base currency]
          |
          v
[React settings control]
          |
          v
[Settings API: PATCH /settings/base-currency]
          |
          v
[Settings module persists app-wide base currency]
          |
          v
[Portfolio page query]
          |
          v
[GET /portfolio]
          |
          v
[Portfolio service] ---> [Settings service: active base currency]
          |                         |
          |                         v
          |                [PostgreSQL settings table]
          |
          +--> [Accounts repository] ----> [PostgreSQL numeric values]
          |
          +--> [Holdings repository] ----> [PostgreSQL numeric values]
          |
          +--> [Pricing/valuation service]
                         |
                         +--> [native amount/value]
                         +--> [asset price used]
                         +--> [FX rate used]
                         v
              [canonical row valuations + subtotals + total net worth]
                         |
                         v
              [JSON DTO with accounts + holdings + breakdown fields]
                         |
                         v
         [React portfolio sections render native + converted values]
                         |
                         v
     [User opens lightweight valuation breakdown for a selected row]
```

### Recommended Project Structure
```text
backend/
├── src/
│   ├── app.module.ts                    # Root Nest module
│   ├── common/
│   │   ├── decimal/                     # Decimal helpers, serializers, formatting guards
│   │   └── validation/                  # Shared pipes / DTO helpers
│   ├── prisma/
│   │   ├── prisma.module.ts             # Prisma integration module
│   │   └── prisma.service.ts            # Prisma client wrapper
│   ├── settings/
│   │   ├── settings.module.ts
│   │   ├── settings.controller.ts       # Base currency endpoint
│   │   ├── settings.service.ts
│   │   └── dto/
│   ├── pricing/
│   │   ├── pricing.module.ts
│   │   ├── fx-rate.service.ts           # Base-currency conversion boundary
│   │   ├── valuation.service.ts         # Canonical Decimal valuation logic
│   │   └── dto/
│   └── portfolio/
│       ├── portfolio.module.ts
│       ├── portfolio.controller.ts      # Canonical portfolio read endpoint
│       ├── portfolio.service.ts
│       ├── portfolio.mapper.ts          # DB -> DTO mapping
│       └── dto/
├── prisma/
│   ├── schema.prisma                    # Accounts, holdings, settings, optional rate snapshot tables
│   └── migrations/
frontend/
├── src/
│   ├── app/
│   │   └── providers/                   # Query client provider
│   ├── api/
│   │   ├── client.ts                    # fetch wrapper
│   │   ├── portfolio.ts                 # portfolio query function
│   │   └── settings.ts                  # base currency mutation
│   ├── features/
│   │   ├── portfolio/
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   └── types.ts
│   │   └── settings/
│   │       ├── components/
│   │       └── hooks/
│   └── pages/
│       └── PortfolioPage.tsx            # First canonical portfolio screen
```

### Pattern 1: Canonical portfolio read model, not ad hoc joined responses
**What:** Return one stable portfolio DTO with `summary`, `accounts`, and `holdings` sections; each row carries both native and converted values plus optional breakdown fields [VERIFIED: 01-CONTEXT.md].

**When to use:** For the main portfolio endpoint and any later dashboard-consuming portfolio snapshot response.

**Example:**
```typescript
// Source: project-constrained pattern derived from 01-CONTEXT.md + Prisma nested relations docs
export type DecimalString = string;

export interface MoneyValueDto {
  amount: DecimalString;
  currency: string;
}

export interface ValuationBreakdownDto {
  nativeAmount: DecimalString;
  nativeCurrency: string;
  unitPrice: DecimalString | null;
  unitPriceCurrency: string | null;
  fxRateToBase: DecimalString;
  baseCurrency: string;
  convertedValue: DecimalString;
}

export interface PortfolioRowDto {
  id: string;
  label: string;
  nativeValue: MoneyValueDto;
  baseValue: MoneyValueDto;
  breakdown?: ValuationBreakdownDto;
}

export interface PortfolioResponseDto {
  baseCurrency: string;
  totalNetWorth: MoneyValueDto;
  accounts: {
    subtotal: MoneyValueDto;
    items: PortfolioRowDto[];
  };
  holdings: {
    subtotal: MoneyValueDto;
    items: PortfolioRowDto[];
  };
}
```

### Pattern 2: Keep Decimal values as Decimal/string until the display edge
**What:** Do math in PostgreSQL `numeric`, Prisma `Decimal`, and `decimal.js`; serialize to strings in API DTOs instead of JS `number` [CITED: https://www.postgresql.org/docs/current/datatype-numeric.html] [CITED: https://www.prisma.io/docs/orm/prisma-client/special-fields-and-types#working-with-decimal] [CITED: https://mikemcl.github.io/decimal.js/].

**When to use:** Everywhere in valuation and totals code, including DTO mapping.

**Example:**
```typescript
// Source: Prisma Decimal docs + decimal.js docs
import { Prisma } from '@prisma/client';
import Decimal from 'decimal.js';

const native = new Prisma.Decimal('1234.56');
const fxRate = new Decimal('0.92');
const baseValue = new Decimal(native.toString()).mul(fxRate);

const response = {
  nativeValue: native.toFixed(2),
  baseValue: baseValue.toFixed(2),
};
```

### Pattern 3: Feature modules with controller -> service -> mapper separation
**What:** Use Nest feature modules with thin controllers, business logic in services, and explicit mappers for API DTOs [CITED: https://docs.nestjs.com/controllers] [CITED: https://docs.nestjs.com/modules].

**When to use:** From the first backend assets onward, especially because this phase seeds later `pricing`, `portfolio`, and `settings` modules [VERIFIED: CLAUDE.md].

**Example:**
```typescript
// Source: NestJS controllers/modules/providers docs pattern
@Controller('portfolio')
export class PortfolioController {
  constructor(private readonly portfolioService: PortfolioService) {}

  @Get()
  getPortfolio() {
    return this.portfolioService.getCanonicalPortfolio();
  }
}
```

### Pattern 4: Query portfolio as server state with stable query keys
**What:** Fetch the canonical portfolio DTO through TanStack Query and let the backend drive recalculation after base-currency updates [CITED: https://tanstack.com/query/latest/docs/framework/react/overview].

**When to use:** Portfolio page loads and any mutation that invalidates portfolio totals.

**Example:**
```tsx
// Source: TanStack Query overview + useQuery examples
const portfolioQuery = useQuery({
  queryKey: ['portfolio'],
  queryFn: fetchPortfolio,
  staleTime: 60_000,
});
```

### Pattern 5: Settings form as a narrow command surface
**What:** Use React Hook Form only for the base-currency control, then invalidate/refetch canonical portfolio data after submit [CITED: https://react-hook-form.com/docs/useform] [CITED: https://tanstack.com/query/latest/docs/framework/react/overview].

**When to use:** Phase 1 settings UI.

**Example:**
```tsx
// Source: React Hook Form useForm docs + TanStack Query mutation/query invalidation pattern
const form = useForm<{ baseCurrency: string }>({
  defaultValues: { baseCurrency: 'EUR' },
});
```

### Anti-Patterns to Avoid
- **Flat unified asset list as the source of truth:** Conflicts with locked decisions D-01/D-02 and will complicate later allocation semantics [VERIFIED: 01-CONTEXT.md].
- **Frontend recalculates totals:** Violates project architecture and invites drift between screens [VERIFIED: CLAUDE.md].
- **Using JS `number` for finance math:** PostgreSQL explicitly distinguishes exact `numeric` from inexact floating-point types [CITED: https://www.postgresql.org/docs/current/datatype-numeric.html].
- **Formatting or rounding before all intermediate calculations finish:** Early rounding will make subtotals diverge from line-item sums [ASSUMED].
- **Returning raw Prisma Decimal objects directly to the client without an explicit serialization policy:** This creates unstable API contracts and display ambiguity [ASSUMED].

## Recommended File Targets

These are the highest-leverage initial file targets for planners because the repo currently has no application source files [VERIFIED: 01-CONTEXT.md] [VERIFIED: repo glob scan].

| Priority | File Target | Why It Likely Comes First |
|----------|-------------|---------------------------|
| 1 | `backend/prisma/schema.prisma` | Defines the durable domain and numeric precision choices before service code [CITED: https://www.prisma.io/docs/orm/reference/prisma-schema-reference#decimal]. |
| 2 | `backend/src/app.module.ts` | Establishes the modular monolith root [VERIFIED: CLAUDE.md]. |
| 3 | `backend/src/prisma/prisma.module.ts` | Needed before portfolio/settings services can query data [ASSUMED]. |
| 4 | `backend/src/settings/settings.module.ts` | PORT-01 needs app-wide base currency persistence. |
| 5 | `backend/src/settings/settings.controller.ts` | Minimal API to read/update base currency. |
| 6 | `backend/src/pricing/valuation.service.ts` | Central exact valuation logic for conversion and breakdowns. |
| 7 | `backend/src/portfolio/portfolio.service.ts` | Canonical aggregate assembly. |
| 8 | `backend/src/portfolio/portfolio.controller.ts` | Phase 1 portfolio read endpoint. |
| 9 | `backend/src/portfolio/dto/*.ts` | Stable API contract for accounts, holdings, totals, and breakdowns. |
| 10 | `frontend/src/app/providers/QueryProvider.tsx` | Required app wiring for TanStack Query. |
| 11 | `frontend/src/api/portfolio.ts` | Frontend portfolio read boundary. |
| 12 | `frontend/src/features/portfolio/components/*` | Render accounts/holdings sections with side-by-side values. |
| 13 | `frontend/src/features/settings/components/BaseCurrencyForm.tsx` | PORT-01 control surface. |
| 14 | `frontend/src/pages/PortfolioPage.tsx` | First end-user slice that proves the phase. |

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Decimal arithmetic | Custom money math helpers on JS `number` | PostgreSQL `numeric` + Prisma `Decimal` + `decimal.js` | Exactness is required for monetary amounts; float math is the wrong substrate [VERIFIED: CLAUDE.md] [CITED: https://www.postgresql.org/docs/current/datatype-numeric.html]. |
| Server-state cache | Homegrown fetch cache and invalidation layer | TanStack Query | Cache, staleness, and refetch semantics are standard server-state concerns [CITED: https://tanstack.com/query/latest/docs/framework/react/overview]. |
| Form state plumbing | Manual controlled-input boilerplate | React Hook Form | The chosen stack already includes a lightweight form library for this [VERIFIED: CLAUDE.md] [CITED: https://react-hook-form.com/docs/useform]. |
| Nested relation graph loading | Custom hand-assembled object stitching in every endpoint | Prisma `include` / `select` | Prisma already supports relation queries appropriate for this first read model [CITED: https://www.prisma.io/docs/orm/prisma-client/queries/relation-queries]. |
| Request validation pipeline | Ad hoc validation in controllers | Nest pipes / ValidationPipe pattern | Nest provides a standard request validation layer [CITED: https://docs.nestjs.com/techniques/validation]. |

**Key insight:** In this domain, the dangerous custom code is not the UI markup; it is any homegrown money math, serialization, or cache invalidation logic that quietly diverges over time.

## Common Pitfalls

### Pitfall 1: Mixing storage precision policy with display precision policy
**What goes wrong:** Values are stored or computed at display precision too early, so subtotals and totals drift.
**Why it happens:** Teams use two-decimal formatting rules as internal arithmetic rules.
**How to avoid:** Keep internal arithmetic at Decimal precision and round only when the API/display contract requires it [CITED: https://www.postgresql.org/docs/current/functions-math.html] [CITED: https://mikemcl.github.io/decimal.js/].
**Warning signs:** A row’s converted value multiplied or summed manually differs from a subtotal by a cent or more.

### Pitfall 2: Treating native currency value and unit price as the same concept
**What goes wrong:** Holdings valuation becomes ambiguous because quantity, price currency, and value currency are collapsed into one field.
**Why it happens:** The first phase models “value” without preserving enough breakdown inputs for D-05.
**How to avoid:** Separate quantity/native amount, unit price, price currency, FX rate, and converted result in the valuation pipeline [VERIFIED: 01-CONTEXT.md].
**Warning signs:** You cannot explain how a holding’s base value was derived without reading service code.

### Pitfall 3: Letting the frontend infer canonical totals
**What goes wrong:** Different screens show slightly different totals or rounding behavior.
**Why it happens:** React components recompute subtotals from row data or local formatting rules.
**How to avoid:** Return canonical totals and subtotals from the backend and treat them as display data, not recomputation inputs [VERIFIED: CLAUDE.md].
**Warning signs:** Utility functions like `sumPortfolioRows()` appear in UI code.

### Pitfall 4: Flattening accounts and holdings because Phase 1 seems simple
**What goes wrong:** Phase 3 allocation semantics and Phase 4 rebalancing later need major reshaping.
**Why it happens:** Greenfield teams optimize for the first table instead of the long-lived domain model.
**How to avoid:** Preserve section boundaries from day one, even if the first screen is compact [VERIFIED: 01-CONTEXT.md].
**Warning signs:** The API response has one array called `assets` and no section subtotals.

### Pitfall 5: Unclear Decimal serialization policy
**What goes wrong:** Clients receive inconsistent numeric shapes, or precision is lost when converting to JS `number`.
**Why it happens:** Prisma Decimal values are returned directly without a stable DTO convention.
**How to avoid:** Choose one API rule in Wave 0: serialize all finance amounts as decimal strings and format in the UI from strings/Decimal, not numbers [ASSUMED].
**Warning signs:** Mixed endpoint responses where some amounts are strings and others are numbers.

## Code Examples

Verified patterns from official sources:

### Global request validation in Nest
```typescript
// Source: https://github.com/nestjs/docs.nestjs.com/blob/master/content/pipes.md
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

### ValidationPipe via APP_PIPE provider
```typescript
// Source: https://github.com/nestjs/docs.nestjs.com/blob/master/content/pipes.md
@Module({
  providers: [
    {
      provide: APP_PIPE,
      useClass: ValidationPipe,
    },
  ],
})
export class AppModule {}
```

### Prisma Decimal creation
```typescript
// Source: https://www.prisma.io/docs/orm/prisma-client/special-fields-and-types#working-with-decimal
import { PrismaClient, Prisma } from '@prisma/client';

const newTypes = await prisma.sample.create({
  data: {
    cost: new Prisma.Decimal(24.454545),
  },
});
```

### Prisma nested relation loading
```typescript
// Source: https://www.prisma.io/docs/orm/prisma-client/queries/relation-queries
const result = await prisma.user.findMany({
  include: {
    posts: {
      include: {
        categories: true,
      },
    },
  },
});
```

### TanStack Query basic server-state read
```tsx
// Source: https://context7.com/tanstack/query/llms.txt
const { isPending, isError, data, error, isFetching } = useQuery({
  queryKey: ['todos'],
  queryFn: async () => {
    const response = await fetch('https://api.example.com/todos')
    if (!response.ok) throw new Error('Network response was not ok')
    return response.json()
  },
  staleTime: 5 * 60 * 1000,
  gcTime: 10 * 60 * 1000,
  refetchOnWindowFocus: true,
  retry: 3,
})
```

### decimal.js fixed-point output to avoid exponent notation
```javascript
// Source: https://github.com/mikemcl/decimal.js/blob/master/README.md
x = new Decimal('0.0000001')
x.toString()   // '1e-7'
x.toFixed()    // '0.0000001'
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| TanStack Query `cacheTime` option name | `gcTime` option name | v5 migration [CITED: https://github.com/tanstack/query/blob/main/docs/framework/react/guides/migrating-to-v5.md] | Use `gcTime` in new config and research examples. |
| Client-heavy recalculation for small React apps | Server-state libraries + backend-owned canonical values | Ongoing ecosystem norm [CITED: https://tanstack.com/query/latest/docs/framework/react/overview] [ASSUMED] | Fits this project’s “backend is truth” rule better than local recomputation. |

**Deprecated/outdated:**
- Using JavaScript floating-point numbers as the canonical financial representation is outdated for exact-money domains [CITED: https://www.postgresql.org/docs/current/datatype-numeric.html].
- Using `cacheTime` terminology in new TanStack Query v5 code is outdated; use `gcTime` [CITED: https://github.com/tanstack/query/blob/main/docs/framework/react/guides/migrating-to-v5.md].

## Resolved Planning Decisions

1. **Database precision/scale policy for Phase 1**
   - Decision: Use explicit PostgreSQL decimal mappings in Prisma rather than unconstrained defaults.
   - Money totals and row values: `@db.Decimal(24, 8)`
   - Market quantities: `@db.Decimal(24, 12)`
   - Unit prices: `@db.Decimal(24, 8)`
   - FX rates: `@db.Decimal(24, 12)`
   - Why: This preserves exact storage with enough headroom for portfolio-scale values, fractional asset quantities, and precise FX conversion while still forcing every finance field to declare its contract explicitly.
   - Planning impact: Plan 01-02 Task 1 must encode these exact `@db.Decimal(p, s)` choices in `backend/prisma/schema.prisma`.

2. **Finance serialization policy for Phase 1 APIs**
   - Decision: Serialize all canonical finance amounts as decimal strings everywhere in API DTOs, not selectively.
   - Scope: `nativeValue.amount`, `baseValue.amount`, totals, subtotals, unit prices, quantities returned in breakdowns, and FX rates.
   - Why: A single contract prevents precision loss and avoids mixed number/string payloads that would undermine trust.
   - Planning impact: Plans 01-02 and 01-03 must provide shared decimal-string serialization utilities and DTO mappers; Plan 01-04/01-05 must consume string finance values directly in the frontend.

3. **Seed-data strategy before Phase 2 ingestion exists**
   - Decision: Use deterministic Prisma seed data as the runnable Phase 1 portfolio source and assert against that seeded data in backend and UI tests. Do not build an admin/manual-entry path in this phase.
   - Minimum seed set:
     - one EUR-denominated account
     - one USD-denominated account
     - one holding with quantity, unit price, price currency, and FX-converted base value
     - one persisted app-wide settings row with default `baseCurrency: 'EUR'`
   - Why: This demonstrates same-currency valuation, FX conversion, section subtotals, and base-currency switching entirely within Phase 1 scope.
   - Planning impact: Plan 01-02 seeds the settings row, Plan 01-03 extends seed data for accounts/holdings, and downstream tests rely on these deterministic records.

**Status:** All previously open planning questions are resolved for execution.

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| Node.js | Nest/Vite/Prisma toolchain | ✓ | v24.14.1 | — |
| npm | Package installation and scripts | ✓ | 11.11.0 | — |
| Docker | Local Postgres and later compose workflow | ✓ | 29.3.1 | — |
| Docker Compose | Local multi-service orchestration | ✓ | v5.1.1 | `docker compose` also available |
| Git | Repo workflow | ✓ | 2.53.0.windows.2 | — |
| Python | Not required for this phase core path | ✓ | 3.14.3 | — |
| `psql` CLI | Optional DB inspection | ✗ | — | Use Prisma CLI or Dockerized Postgres tooling |
| Graph context (`graphify`) | Semantic code/project graph queries | ✗ | disabled | Continue without graph context |

**Missing dependencies with no fallback:**
- None for Phase 1 planning.

**Missing dependencies with fallback:**
- `psql` CLI missing; use Prisma migration tooling and Dockerized DB access instead.
- Graphify disabled; research relied on direct planning docs instead.

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | None detected yet in repo — Phase 1 plans create Nest/Vitest/Playwright test wiring |
| Config file | created by Plans 01-01 and 01-04 |
| Quick run command | `npm run test:backend -- settings --runInBand && npm run test:ui -- portfolio --runInBand` |
| Full suite command | `npm run test:backend -- settings -- portfolio --runInBand && npm run test:ui -- portfolio --runInBand && npm run test:e2e -- portfolio` |

### Phase Requirements → Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| PORT-01 | Update app-wide base currency and observe persisted/reporting currency change | integration + component + e2e | `npm run test:backend -- settings --runInBand` and `npm run test:ui -- portfolio --runInBand` | planned in 01-02 and 01-05 |
| PORT-02 | Return canonical total net worth in selected base currency | integration | `npm run test:backend -- portfolio --runInBand` | planned in 01-03 |
| PORT-03 | Show native and converted values for account/holding rows and breakdown on demand | integration + component + e2e | `npm run test:backend -- portfolio --runInBand`, `npm run test:ui -- portfolio --runInBand`, and `npm run test:e2e -- portfolio` | planned in 01-03, 01-05 |

### Sampling Rate
- **Per task commit:** Use the task’s `<automated>` command.
- **Per wave merge:** Run the wave’s relevant backend/UI suite.
- **Phase gate:** Full backend, UI, and e2e commands green before `/gsd-verify-work`.

### Wave 0 Coverage Status
- [x] Backend test framework bootstrap for Nest + Supertest — Plan 01-01
- [x] Frontend test framework bootstrap for Vite + Vitest — Plan 01-04
- [x] Playwright config for end-to-end base-currency workflow — Plan 01-05
- [x] Seed/fixture utilities for deterministic exact-valuation examples — Plans 01-02 and 01-03
- [x] Numeric/Decimal assertion helpers through string serialization policy — Plans 01-02 and 01-03

## Security Domain

### Applicable ASVS Categories

| ASVS Category | Applies | Standard Control |
|---------------|---------|-----------------|
| V2 Authentication | no | Single-user local-first v1; no mandatory auth in current project scope [VERIFIED: PROJECT.md] |
| V3 Session Management | no | Same reason as V2 for this phase scope [VERIFIED: PROJECT.md] |
| V4 Access Control | no | Single trusted local user in v1 scope [VERIFIED: PROJECT.md] |
| V5 Input Validation | yes | Nest validation pipes and DTO validation at API boundaries [CITED: https://docs.nestjs.com/techniques/validation] |
| V6 Cryptography | no | No crypto primitives required for Phase 1 valuation foundation |

### Known Threat Patterns for NestJS + Prisma + React finance data

| Pattern | STRIDE | Standard Mitigation |
|---------|--------|---------------------|
| Invalid currency code or unsupported base currency input | Tampering | Validate enum/allowed-code list at request boundary [ASSUMED] |
| Precision loss from float coercion | Tampering | Keep finance values in `numeric`/Decimal/string path; forbid JS `number` as canonical storage/transport [CITED: https://www.postgresql.org/docs/current/datatype-numeric.html] |
| Overexposed internal valuation fields in API | Information Disclosure | Use explicit DTO mapping rather than returning ORM objects directly [ASSUMED] |
| Inconsistent serialization causing client misread | Tampering | Standardize finance DTOs on stringified decimals and typed currency fields [ASSUMED] |

## Sources

### Primary (HIGH confidence)
- `CLAUDE.md` - stack, architecture boundaries, backend-as-canonical-truth, exact math constraints
- `.planning/phases/01-portfolio-domain-valuation-foundation/01-CONTEXT.md` - locked decisions D-01 through D-06 and phase-specific context
- `.planning/ROADMAP.md` - Phase 1 success criteria and dependency boundary
- `.planning/REQUIREMENTS.md` - PORT-01, PORT-02, PORT-03 definitions
- `https://www.postgresql.org/docs/current/datatype-numeric.html` - exactness, money suitability, precision/scale, performance tradeoffs for `numeric`
- `https://www.postgresql.org/docs/current/functions-math.html` - `round(numeric, s)`, truncation, numeric rounding behavior
- `https://www.prisma.io/docs/orm/prisma-client/special-fields-and-types#working-with-decimal` - Prisma Decimal usage
- `https://www.prisma.io/docs/orm/reference/prisma-schema-reference#decimal` - Decimal type mapping for PostgreSQL
- `https://www.prisma.io/docs/orm/prisma-client/queries/relation-queries` - nested `include` / `select` patterns
- `https://mikemcl.github.io/decimal.js/` - decimal.js arithmetic and formatting behavior
- `https://vite.dev/guide/` - Vite dev/build guidance
- `https://tanstack.com/query/latest/docs/framework/react/overview` - TanStack Query role for server state
- `https://react-hook-form.com/docs/useform` - `useForm` behavior and intended usage
- `https://github.com/nestjs/docs.nestjs.com/blob/master/content/pipes.md` - ValidationPipe registration patterns

### Secondary (MEDIUM confidence)
- Context7 `/tanstack/query` - v5 `gcTime` examples and `useQuery` reference snippets
- Context7 `/mikemcl/decimal.js` - fixed-point formatting examples
- Context7 `/prisma/prisma` - relation-query examples

### Tertiary (LOW confidence)
- `[ASSUMED]` judgments about recommended initial file split and which validation packages to standardize in Wave 0

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | `class-validator` + `class-transformer` should be the default Nest validation pairing for this phase | Standard Stack | Medium — planner may choose a different validation stack, altering Wave 0 setup |
| A2 | Prisma relation loading is sufficient for the first canonical portfolio read model without raw SQL | Alternatives Considered | Low — raw SQL could still be chosen later for performance |
| A3 | Early rounding is a likely source of cent drift if applied before all intermediate computations finish | Anti-Patterns / Pitfalls | Low — still directionally safe, but exact manifestation depends on implementation |
| A4 | `prisma.module.ts` is among the first backend files planners should create | Recommended File Targets | Low — file ordering could vary without changing architecture |
| A5 | Invalid currency codes should be constrained by an allowlist/enum at the request boundary | Security Domain | Low — exact implementation detail may vary |
| A6 | Explicit DTO mapping is the preferred control against overexposed internal valuation fields | Security Domain | Low — serialization interceptors could also solve it |

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - project stack is explicitly locked and package versions were verified against npm registry.
- Architecture: MEDIUM - domain and boundaries are strongly constrained by context, and the remaining execution choices are now resolved for precision, serialization, and seed strategy.
- Pitfalls: MEDIUM - core exact-arithmetic and backend-truth pitfalls are well supported, but some implementation-specific failure modes are experience-based.

**Research date:** 2026-04-20
**Valid until:** 2026-05-20
