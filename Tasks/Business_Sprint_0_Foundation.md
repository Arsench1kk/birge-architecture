---
status: active
version: v1.1
last_updated: 2026-05-09
owner: Arsen + Business Partner AI chat
---

# Business Sprint 0 — BIRGE Foundation

Goal:

```text
Define the business/product logic before continuing UI mockups or new technical implementation.
```

## Non-Negotiable Foundation

- BIRGE is transportation-as-a-subscription.
- Tier 3 subscription commute is the wedge.
- Tier 1 on-demand is fallback/acquisition, not the product center.
- Drivers pay subscription only.
- Zero per-ride commission ever.
- MVP corridor is hub-to-hub.
- First corridor is Alatau City -> Esentai / Al-Farabi.

## Sprint Outputs

| Output | Status |
|---|---|
| Passenger subscription commute flow, screen by screen | Todo |
| Passenger subscription tier structure | Done — MVP launches with Solo Corridor / Multi Corridor / Flex Pack |
| Driver onboarding/economics flow, screen by screen | Done — Session 7 foundation confirmed for onboarding mockups |
| Passenger KZT pricing/subscription logic | Todo — Session 8 |
| Driver subscription pricing reconciliation | Updated — Peak Starter 9,900 added; app repo change needs review/commit |
| First corridor pilot rules | Todo |
| Underfilled corridor fallback policy | Todo |
| Passenger no-show / late cancellation policy | Working decision — consumes ride/slot and protects driver |
| Interview plan: passengers | Todo |
| Interview plan: drivers | Todo |
| UvU Shuttle competitor notes | Todo |
| Legal/Kaspi readiness checklist | Prompt prepared; real legal review still needed |

## Work Plan

### 1. Passenger Flow Spec

Define:

- what user sees on first launch
- how user declares monthly routes
- how multiple independent routes work
- how pickup/dropoff hubs are chosen
- how monthly price is explained
- what the dashboard shows after subscription
- what happens on ride day
- what fallback looks like

### 2. Driver Flow Spec

Confirmed:

- driver value proposition: planned recurring corridor work, 0% commission, no guaranteed income claims
- first driver persona: Серик
- tier pricing: Peak Starter 9,900; Starter 19k; Professional 28k; Premium 38k KZT/month
- MVP positioning: peak-hour income layer, not full-day Yandex replacement
- minimum vehicle year: 2000 or newer
- driver subscription activates after first active corridor
- failed verification should lead to correction/re-upload
- Yandex/InDrive parallel work allowed during launch
- passenger late cancellation/no-show consumes ride/slot and should protect driver
- Professional should be offered only when enough active corridors justify upgrade; working idea is 2+ active corridors

Still define later:
- exact driver dashboard after approval
- active run lifecycle
- payout/subscription billing details
- exact compensation accounting after no-show
- final differentiation between Peak Starter and Starter
- legal constraints after lawyer review

### 3. Pricing Logic

Define:

- passenger monthly pass formula
- MVP Solo Corridor price and what one route/time slot covers
- MVP Multi Corridor price and what all pre-declared routes cover
- MVP Flex Pack pack sizes, per-ride deduction, expiry, refunds, and unused-ride rules
- pay-as-you-go corridor pricing
- on-demand fallback pricing
- driver subscription tiers
- launch discounts
- refund/cancel policy

### 4. Validation

Prepare:

- 10 passenger interview script
- 5 driver interview script
- real survey plan
- UvU Shuttle test checklist
- legal consultation questions

Legal AI prompt prepared:

```text
Context/Legal_Advisor_Prompt_BIRGE.md
```

### 5. UI Gate

Only after the business outputs above:

- update Open Design prompts
- redesign passenger mockups
- redesign driver mockups
- then update iOS implementation

## Technical Changes Later

Do not start these until product logic is ready:

- Replace home “Вызвать такси” center with monthly commute dashboard/setup.
- Replace demo driver prices with Peak Starter 9,900 + Starter 19k + Professional 28k + Premium 38k.
- Expand onboarding into multi-route monthly commute setup.
- Rework subscriptions around route coverage and savings.
- Add backend models for recurring routes/monthly commute plans if needed.
