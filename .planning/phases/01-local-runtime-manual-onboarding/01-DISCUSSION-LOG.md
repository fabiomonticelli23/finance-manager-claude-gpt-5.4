# Phase 1: local-runtime-manual-onboarding - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-04-20
**Phase:** 01-local-runtime-manual-onboarding
**Areas discussed:** Local startup flow, First-run onboarding, Manual source model, Protected cash setup

---

## Local startup flow

### How should local startup work for v1?
| Option | Description | Selected |
|--------|-------------|----------|
| One command | Use Docker Compose as the main path so backend, frontend, and database come up together with one command. | ✓ |
| Hybrid dev | Run database in Docker, but start app services manually for faster iteration after setup. | |
| Manual only | Start each service separately. Most flexible, but more setup friction. | |
| You decide | Leave the exact startup mode to Claude during planning. | |

**User's choice:** One command
**Notes:** Primary startup path should be a single Docker Compose command.

### How should the app signal that first launch is actually ready to use?
| Option | Description | Selected |
|--------|-------------|----------|
| Wait for ready | Only present the app once the required services are healthy, so the first impression feels stable. | ✓ |
| Open early | Show the app immediately and let screens resolve as services come online. | |
| Separate check | Require a separate readiness step or health page before the app is used. | |
| You decide | Leave the readiness behavior to Claude during planning. | |

**User's choice:** Wait for ready
**Notes:** Startup should feel stable rather than partially available.

### What should happen with sample data on first launch?
| Option | Description | Selected |
|--------|-------------|----------|
| Empty start | Start with no seeded portfolio so onboarding reflects the user’s real accounts from the beginning. | ✓ |
| Optional demo | Offer sample data the user can choose to load for orientation. | |
| Always demo | Preload a demo portfolio so the app never starts empty. | |
| You decide | Leave sample-data behavior to Claude during planning. | |

**User's choice:** Empty start
**Notes:** The product should begin with the user's real data, not demo data.

### How should local data behave across restarts?
| Option | Description | Selected |
|--------|-------------|----------|
| Persist by default | Keep portfolio data in a persistent database volume so setup is not lost between runs. | ✓ |
| Reset often | Treat local data as disposable unless the user explicitly exports or preserves it. | |
| Ask each time | Prompt about keeping or resetting data during startup workflows. | |
| You decide | Leave persistence behavior to Claude during planning. | |

**User's choice:** Persist by default
**Notes:** Local setup should survive restarts.

---

## First-run onboarding

### How should first-run onboarding begin?
| Option | Description | Selected |
|--------|-------------|----------|
| Guided wizard | Walk the user through setup step by step instead of dropping them into an empty app. | ✓ |
| Action hub | Open on a dashboard-style start screen with clear actions for what to add first. | |
| Settings first | Start in configuration screens before any portfolio entry. | |
| You decide | Leave the onboarding entry point to Claude during planning. | |

**User's choice:** Guided wizard
**Notes:** The first experience should actively guide setup.

### What pacing should the onboarding use?
| Option | Description | Selected |
|--------|-------------|----------|
| Short guided steps | A few focused steps with progress so setup feels lightweight but structured. | ✓ |
| Single long form | Capture everything in one screen for speed. | |
| Loose checklist | Show recommended tasks but let the user jump around freely. | |
| You decide | Leave the onboarding pacing to Claude during planning. | |

**User's choice:** Short guided steps
**Notes:** Onboarding should feel structured without becoming heavy.

### What should count as a successful onboarding finish?
| Option | Description | Selected |
|--------|-------------|----------|
| First real source added | Finish once at least one real manual source or holding is saved, so the app stops feeling empty. | ✓ |
| Dashboard visible | Finish once the user reaches the dashboard, even with no saved data. | |
| Core setup complete | Require base settings plus at least one source before finishing. | |
| You decide | Leave the completion rule to Claude during planning. | |

**User's choice:** First real source added
**Notes:** Onboarding should only end after the product has real portfolio data.

### Where should the user land after onboarding completes?
| Option | Description | Selected |
|--------|-------------|----------|
| Live dashboard | Take them straight to the real dashboard with their newly entered data. | ✓ |
| Review summary | Show a summary screen first, then continue to the dashboard. | |
| Add more prompt | Immediately offer another add-source flow before showing the dashboard. | |
| You decide | Leave the post-onboarding destination to Claude during planning. | |

**User's choice:** Live dashboard
**Notes:** The onboarding flow should hand off directly into the real product.

---

## Manual source model

### What should the primary onboarding unit be for manual data?
| Option | Description | Selected |
|--------|-------------|----------|
| Source first | Create a manual source/account first, then attach balances or holdings to it. This best fits provenance, drilldown, and later analytics. | ✓ |
| Holding first | Enter assets directly first, then organize them later. | |
| Mixed flow | Let the user add either an account or a holding from the start. | |
| You decide | Leave the data-entry model to Claude during planning. | |

**User's choice:** Source first
**Notes:** The model should start from manual sources rather than free-floating holdings.

### How should manual source setup be structured at the start?
| Option | Description | Selected |
|--------|-------------|----------|
| Type-specific forms | Use tailored setup for liquidity accounts, investment holdings, and protected cash so the data is cleaner from day one. | ✓ |
| One generic form | Use a single flexible form for every manual source type. | |
| Generic then refine | Start simple, then ask follow-up questions based on selections. | |
| You decide | Leave source-form structure to Claude during planning. | |

**User's choice:** Type-specific forms
**Notes:** Different manual source types should guide different setup details.

### What minimum information should be required before saving a manual source?
| Option | Description | Selected |
|--------|-------------|----------|
| Just enough to value | Require only the fields needed to identify the source and calculate its current value, keeping onboarding fast. | ✓ |
| Detailed upfront | Capture richer metadata immediately for future organization and reporting. | |
| Name only | Allow very minimal saves, even if value details are incomplete until later. | |
| You decide | Leave the required fields to Claude during planning. | |

**User's choice:** Just enough to value
**Notes:** Onboarding should favor speed while still capturing usable portfolio data.

### After saving one manual source, what should the flow do next?
| Option | Description | Selected |
|--------|-------------|----------|
| Offer add another | Prompt to add more sources while momentum is high, but still allow finishing. | |
| Finish immediately | Treat the first real source as enough to complete onboarding and let additional entry happen from the app. | ✓ |
| Require more | Make the user add multiple sources before onboarding can end. | |
| You decide | Leave the next-step behavior to Claude during planning. | |

**User's choice:** Finish immediately
**Notes:** The first saved real source is enough to transition into the app.

---

## Protected cash setup

### How should emergency fund and protected cash be handled during onboarding?
| Option | Description | Selected |
|--------|-------------|----------|
| Classify during setup | Ask during source creation whether the money is protected cash, current account, deposit account, or regular holding so analytics start correctly. | ✓ |
| Classify later | Let users add balances first and organize protected cash after onboarding. | |
| Auto-infer | Try to infer protected cash from source type or naming. | |
| You decide | Leave the protected-cash flow to Claude during planning. | |

**User's choice:** Classify during setup
**Notes:** Protected-cash meaning should be explicit from the beginning.

### What should onboarding do about emergency-fund targets?
| Option | Description | Selected |
|--------|-------------|----------|
| Capture target now | Ask for the protected target during setup so later dashboards and rebalancing can reflect it immediately. | ✓ |
| Optional now | Offer target setup, but allow skipping it during onboarding. | |
| Later only | Defer all target setup until after onboarding. | |
| You decide | Leave emergency-target timing to Claude during planning. | |

**User's choice:** Capture target now
**Notes:** Emergency-fund targets should be part of initial setup.

### If the user creates protected cash but does not fully define a target, what should the product do?
| Option | Description | Selected |
|--------|-------------|----------|
| Mark incomplete | Save the source but clearly show that protected-cash rules are incomplete until the target is set. | ✓ |
| Block save | Do not allow protected cash to be saved without a full target. | |
| Assume zero | Save it and treat the target as zero until edited later. | |
| You decide | Leave this behavior to Claude during planning. | |

**User's choice:** Mark incomplete
**Notes:** The source should still save, but the missing target must stay visible.

### How visible should protected cash status be right after onboarding?
| Option | Description | Selected |
|--------|-------------|----------|
| Prominent badge | Show protected-cash status clearly on the dashboard so the user immediately understands it is treated differently. | ✓ |
| Normal listing | Show it like any other source and reveal the distinction only in drilldowns. | |
| Separate panel | Give protected cash its own dedicated section on the initial dashboard. | |
| You decide | Leave the presentation choice to Claude during planning. | |

**User's choice:** Prominent badge
**Notes:** The dashboard should immediately communicate that protected cash is special.

## Claude's Discretion

- Exact onboarding copy and micro-interactions
- Exact service health-check mechanics
- Exact dashboard layout within the locked onboarding and visibility decisions

## Deferred Ideas

None
