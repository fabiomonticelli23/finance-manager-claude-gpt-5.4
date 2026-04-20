# Phase 1: Portfolio Domain & Valuation Foundation - Pattern Map

**Mapped:** 2026-04-20
**Files analyzed:** 23
**Analogs found:** 0 / 23

## File Classification

| New/Modified File | Role | Data Flow | Closest Analog | Match Quality |
|-------------------|------|-----------|----------------|---------------|
| `backend/prisma/schema.prisma` | model | CRUD | none in codebase | none |
| `backend/src/app.module.ts` | config | request-response | none in codebase | none |
| `backend/src/main.ts` | config | request-response | none in codebase | none |
| `backend/src/prisma/prisma.module.ts` | provider | CRUD | none in codebase | none |
| `backend/src/prisma/prisma.service.ts` | service | CRUD | none in codebase | none |
| `backend/src/common/decimal/decimal.serializer.ts` | utility | transform | none in codebase | none |
| `backend/src/common/validation/base-currency.dto.ts` | utility | request-response | none in codebase | none |
| `backend/src/settings/settings.module.ts` | config | request-response | none in codebase | none |
| `backend/src/settings/settings.controller.ts` | controller | request-response | none in codebase | none |
| `backend/src/settings/settings.service.ts` | service | CRUD | none in codebase | none |
| `backend/src/settings/dto/update-base-currency.dto.ts` | model | request-response | none in codebase | none |
| `backend/src/pricing/pricing.module.ts` | config | request-response | none in codebase | none |
| `backend/src/pricing/fx-rate.service.ts` | service | transform | none in codebase | none |
| `backend/src/pricing/valuation.service.ts` | service | transform | none in codebase | none |
| `backend/src/portfolio/portfolio.module.ts` | config | request-response | none in codebase | none |
| `backend/src/portfolio/portfolio.controller.ts` | controller | request-response | none in codebase | none |
| `backend/src/portfolio/portfolio.service.ts` | service | CRUD | none in codebase | none |
| `backend/src/portfolio/portfolio.mapper.ts` | utility | transform | none in codebase | none |
| `backend/src/portfolio/dto/portfolio-response.dto.ts` | model | request-response | none in codebase | none |
| `frontend/src/app/providers/QueryProvider.tsx` | provider | request-response | none in codebase | none |
| `frontend/src/api/portfolio.ts` | service | request-response | none in codebase | none |
| `frontend/src/features/settings/components/BaseCurrencyForm.tsx` | component | request-response | none in codebase | none |
| `frontend/src/pages/PortfolioPage.tsx` | component | request-response | none in codebase | none |

## Pattern Assignments

No application source scaffold exists yet in this repository. There are no backend or frontend implementation analogs to copy directly from. For Phase 1 planning, the closest usable references are the project constraints and research-backed patterns already documented in planning artifacts.

### `backend/prisma/schema.prisma` (model, CRUD)

**Analog:** none in codebase

**Use instead:** Research-backed schema-first pattern from `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 177-206 and 507-516.

**Pattern to copy conceptually:**
- Create durable tables before service code.
- Use explicit Decimal fields for money, price, quantity, and FX values.
- Preserve separate settings, accounts, and holdings structures so the API does not flatten the domain.

**Planner guidance:**
- Treat precision/scale choice as an explicit Wave 0 design decision.
- Keep accounts and holdings distinct in the schema because Phase 1 canonical output requires separate sections and subtotals.

---

### `backend/src/app.module.ts` (config, request-response)

**Analog:** none in codebase

**Fallback pattern source:** `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 429-441.

**Module wiring pattern** (research example, lines 429-441):
```typescript
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

**Planner guidance:**
- Root module should register shared validation once.
- Import `PrismaModule`, `SettingsModule`, `PricingModule`, and `PortfolioModule` from day one.

---

### `backend/src/main.ts` (config, request-response)

**Analog:** none in codebase

**Fallback pattern source:** `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 418-427.

**Bootstrap validation pattern** (research example, lines 418-427):
```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

**Planner guidance:**
- Use this as the initial bootstrap shape unless validation is centralized through `APP_PIPE` in `AppModule`.
- Keep startup thin; do not place domain logic here.

---

### `backend/src/settings/settings.controller.ts` (controller, request-response)

**Analog:** none in codebase

**Fallback pattern source:** `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 295-312 and 138-142.

**Thin controller pattern** (research example, lines 301-310):
```typescript
@Controller('portfolio')
export class PortfolioController {
  constructor(private readonly portfolioService: PortfolioService) {}

  @Get()
  getPortfolio() {
    return this.portfolioService.getCanonicalPortfolio();
  }
}
```

**Planner guidance:**
- Mirror this shape for settings endpoints: thin controller, delegate persistence and base-currency logic to a service.
- Base-currency update belongs behind a settings endpoint, not inside portfolio handlers.

---

### `backend/src/settings/settings.service.ts` (service, CRUD)

**Analog:** none in codebase

**Use instead:** Architecture flow from `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 131-175.

**Service responsibility pattern to copy:**
- Persist the single app-wide base currency.
- Expose the active base currency to `PortfolioService` and valuation logic.
- Keep write responsibility here rather than distributing settings reads and writes across modules.

---

### `backend/src/settings/dto/update-base-currency.dto.ts` (model, request-response)

**Analog:** none in codebase

**Fallback pattern source:** `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 580-590 and 378-379.

**Validation pattern to apply:**
- Validate the base currency at the request boundary.
- Use Nest validation pipes / DTO validation rather than ad hoc controller checks.
- Constrain to supported currency codes via enum or allowlist.

---

### `backend/src/pricing/fx-rate.service.ts` (service, transform)

**Analog:** none in codebase

**Use instead:** Architecture flow from `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 159-165 and 390-394.

**Pattern to copy conceptually:**
- Separate FX lookup responsibility from valuation assembly.
- Preserve `fxRateToBase` as a first-class input so breakdowns can explain conversions.
- Do not collapse FX rate, unit price, and native value into one field.

---

### `backend/src/pricing/valuation.service.ts` (service, transform)

**Analog:** none in codebase

**Fallback pattern source:** `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 280-293.

**Exact-decimal valuation pattern** (research example, lines 280-293):
```typescript
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

**Planner guidance:**
- Keep all canonical finance math in Decimal form until DTO serialization.
- Convert through strings, not JS `number`.
- Expose breakdown fields needed by the UI: native amount, unit price, price currency, FX rate, base currency, converted result.

---

### `backend/src/portfolio/portfolio.controller.ts` (controller, request-response)

**Analog:** none in codebase

**Fallback pattern source:** `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 301-310.

**Controller pattern** (research example, lines 301-310):
```typescript
@Controller('portfolio')
export class PortfolioController {
  constructor(private readonly portfolioService: PortfolioService) {}

  @Get()
  getPortfolio() {
    return this.portfolioService.getCanonicalPortfolio();
  }
}
```

**Planner guidance:**
- Keep controller read-only in this phase.
- The service should return canonical totals and subtotals already computed.

---

### `backend/src/portfolio/portfolio.service.ts` (service, CRUD)

**Analog:** none in codebase

**Fallback pattern source:** `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 227-272 and 150-168.

**Canonical portfolio DTO pattern** (research example, lines 233-272):
```typescript
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

**Planner guidance:**
- Service assembles one canonical response, not multiple ad hoc lists.
- Preserve top-level `accounts` and `holdings` sections with subtotals.
- Compute totals in backend using active settings base currency.

---

### `backend/src/portfolio/portfolio.mapper.ts` (utility, transform)

**Analog:** none in codebase

**Use instead:** Separation rule from `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 295-299 and 588-590.

**Pattern to copy conceptually:**
- Keep DTO mapping explicit.
- Do not return raw Prisma models or Decimal objects directly.
- Mapper is the boundary where internal records become stable API DTOs with stringified decimal values.

---

### `backend/src/portfolio/dto/portfolio-response.dto.ts` (model, request-response)

**Analog:** none in codebase

**Fallback pattern source:** `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 233-272.

**DTO shape pattern** (research example, lines 233-272):
```typescript
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

**Planner guidance:**
- Keep finance amounts as decimal strings.
- Include both native and converted values on each row.
- Make breakdown fields available for on-demand UI disclosure.

---

### `frontend/src/app/providers/QueryProvider.tsx` (provider, request-response)

**Analog:** none in codebase

**Fallback pattern source:** `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 314-327 and 498-503.

**Query pattern** (research example, lines 320-327):
```tsx
const portfolioQuery = useQuery({
  queryKey: ['portfolio'],
  queryFn: fetchPortfolio,
  staleTime: 60_000,
});
```

**Planner guidance:**
- Establish a shared QueryClient provider early.
- Use stable query keys and TanStack Query v5 naming.
- Frontend should refetch canonical backend values after settings changes rather than recomputing locally.

---

### `frontend/src/api/portfolio.ts` (service, request-response)

**Analog:** none in codebase

**Use instead:** Request boundary guidance from `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 314-327 and 397-400.

**Pattern to copy conceptually:**
- API function is a thin fetch wrapper around the canonical portfolio endpoint.
- Return backend DTOs as-is; do not derive totals client-side.
- Treat server response as the source of truth for totals and subtotals.

---

### `frontend/src/features/settings/components/BaseCurrencyForm.tsx` (component, request-response)

**Analog:** none in codebase

**Fallback pattern source:** `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 334-340 and `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-UI-SPEC.md` lines 115-118 and 160-163.

**Form state pattern** (research example, lines 335-340):
```tsx
const form = useForm<{ baseCurrency: string }>({
  defaultValues: { baseCurrency: 'EUR' },
});
```

**UI contract to copy:**
- Dedicated settings card near top of page.
- Single primary action text: `Save base currency`.
- Submit should invalidate/refetch portfolio data, not recalculate it client-side.

---

### `frontend/src/pages/PortfolioPage.tsx` (component, request-response)

**Analog:** none in codebase

**Fallback pattern sources:**
- `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-UI-SPEC.md` lines 104-149
- `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 227-272 and 314-327

**Layout pattern to copy from UI spec:**
- Header
- Base-currency settings card
- Net worth summary card
- Accounts section with subtotal and rows
- Holdings section with subtotal and rows

**Data pattern to copy from research:**
- Render both native and base-currency values on every relevant row.
- Show valuation breakdown on demand without leaving the page.
- Only one breakdown open at a time.

---

## Shared Patterns

### Validation
**Source:** `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 418-441, 580-590
**Apply to:** `backend/src/main.ts`, `backend/src/app.module.ts`, `backend/src/settings/dto/update-base-currency.dto.ts`

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

```typescript
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

Use Nest validation at the request boundary. Validate currency input before it reaches services.

### Decimal math and serialization
**Source:** `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 280-293, 275-277, 408-411
**Apply to:** all backend pricing, settings, portfolio, and DTO mapper files

```typescript
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

Keep exact math in Decimal form and serialize finance amounts as strings at the API edge.

### Canonical portfolio response shape
**Source:** `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 233-272
**Apply to:** `backend/src/portfolio/portfolio.service.ts`, `backend/src/portfolio/portfolio.mapper.ts`, `backend/src/portfolio/dto/portfolio-response.dto.ts`, `frontend/src/api/portfolio.ts`, `frontend/src/pages/PortfolioPage.tsx`

```typescript
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

Preserve separate `accounts` and `holdings` sections. Do not flatten them into one asset list.

### Thin controller -> service delegation
**Source:** `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 301-310
**Apply to:** `backend/src/settings/settings.controller.ts`, `backend/src/portfolio/portfolio.controller.ts`

```typescript
@Controller('portfolio')
export class PortfolioController {
  constructor(private readonly portfolioService: PortfolioService) {}

  @Get()
  getPortfolio() {
    return this.portfolioService.getCanonicalPortfolio();
  }
}
```

Controllers should stay thin and return service-owned canonical DTOs.

### Query-driven frontend reads
**Source:** `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-RESEARCH.md` lines 320-327, 335-340
**Apply to:** `frontend/src/app/providers/QueryProvider.tsx`, `frontend/src/api/portfolio.ts`, `frontend/src/features/settings/components/BaseCurrencyForm.tsx`, `frontend/src/pages/PortfolioPage.tsx`

```tsx
const portfolioQuery = useQuery({
  queryKey: ['portfolio'],
  queryFn: fetchPortfolio,
  staleTime: 60_000,
});
```

```tsx
const form = useForm<{ baseCurrency: string }>({
  defaultValues: { baseCurrency: 'EUR' },
});
```

Use TanStack Query for canonical server-state reads and React Hook Form for the narrow base-currency settings command surface.

### UI layout and transparency contract
**Source:** `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning/phases/01-portfolio-domain-valuation-foundation/01-UI-SPEC.md` lines 104-149
**Apply to:** `frontend/src/pages/PortfolioPage.tsx`, `frontend/src/features/settings/components/BaseCurrencyForm.tsx`, future portfolio row components

Key rules to copy:
- Dedicated base-currency settings card near the top.
- Net worth summary card.
- Always separate Accounts and Holdings sections.
- Show native and converted values together by default.
- Open valuation breakdown inline or in a lightweight popover/sheet; do not navigate away.

## No Analog Found

Application implementation analogs are absent because the repository currently has no backend or frontend source scaffold.

| File | Role | Data Flow | Reason |
|------|------|-----------|--------|
| `backend/prisma/schema.prisma` | model | CRUD | No Prisma schema exists yet |
| `backend/src/app.module.ts` | config | request-response | No Nest application files exist yet |
| `backend/src/main.ts` | config | request-response | No Nest bootstrap file exists yet |
| `backend/src/prisma/prisma.module.ts` | provider | CRUD | No backend provider pattern exists yet |
| `backend/src/prisma/prisma.service.ts` | service | CRUD | No Prisma integration exists yet |
| `backend/src/common/decimal/decimal.serializer.ts` | utility | transform | No shared serialization utilities exist yet |
| `backend/src/common/validation/base-currency.dto.ts` | utility | request-response | No validation helpers exist yet |
| `backend/src/settings/settings.module.ts` | config | request-response | No settings module exists yet |
| `backend/src/settings/settings.controller.ts` | controller | request-response | No controller files exist yet |
| `backend/src/settings/settings.service.ts` | service | CRUD | No service files exist yet |
| `backend/src/settings/dto/update-base-currency.dto.ts` | model | request-response | No DTO files exist yet |
| `backend/src/pricing/pricing.module.ts` | config | request-response | No pricing module exists yet |
| `backend/src/pricing/fx-rate.service.ts` | service | transform | No pricing service exists yet |
| `backend/src/pricing/valuation.service.ts` | service | transform | No valuation logic exists yet |
| `backend/src/portfolio/portfolio.module.ts` | config | request-response | No portfolio module exists yet |
| `backend/src/portfolio/portfolio.controller.ts` | controller | request-response | No portfolio controller exists yet |
| `backend/src/portfolio/portfolio.service.ts` | service | CRUD | No portfolio service exists yet |
| `backend/src/portfolio/portfolio.mapper.ts` | utility | transform | No mapper pattern exists yet |
| `backend/src/portfolio/dto/portfolio-response.dto.ts` | model | request-response | No portfolio DTOs exist yet |
| `frontend/src/app/providers/QueryProvider.tsx` | provider | request-response | No frontend provider files exist yet |
| `frontend/src/api/portfolio.ts` | service | request-response | No API client files exist yet |
| `frontend/src/features/settings/components/BaseCurrencyForm.tsx` | component | request-response | No React component analogs exist yet |
| `frontend/src/pages/PortfolioPage.tsx` | component | request-response | No page-level frontend files exist yet |

## Metadata

**Analog search scope:** `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/backend`, `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/frontend`, `/C:/Users/super/Documents/finance-manager-claude-gpt-5.4/.planning`
**Files scanned:** 15 planning/config files returned by repository scan, 0 application source files
**Pattern extraction date:** 2026-04-20
**Notes:** Because no implementation analogs exist, this map uses project docs and research-backed examples as planner fallbacks rather than codebase copy targets.
