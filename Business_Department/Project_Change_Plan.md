---
status: active
version: v1.0
last_updated: 2026-05-07
source: Context/Universal_AI_Handoff_BIRGE.md v3.0
---

# BIRGE Project Change Plan

This document translates the new business foundation into concrete changes for the current project.

## Diagnosis

The current app/repo is technically useful, but product-wise it still leans toward:

```text
open app -> request ride -> search driver -> offer -> active ride
```

The new foundation says the core product should be:

```text
open app -> declare monthly routes -> get pre-planned commute -> subscribe -> follow daily commute plan
```

This is not a small UI rename. It changes the center of the product.

## What Must Change

| Area | Current Bias | Needed Direction |
|---|---|---|
| Product narrative | taxi / ride-sharing | transportation-as-a-subscription |
| Passenger home | call taxi / browse corridors | today’s commute plan + monthly setup |
| Onboarding | demo route setup | full monthly route declaration |
| Ride request | one-off request | recurring route/corridor planning |
| Subscription | generic plans | monthly commute pass covering declared routes |
| Passenger plan choice | unclear/generic subscription plans | all three MVP tiers, with Multi Corridor recommended |
| Driver onboarding | profile + demo tier | subscription business model + corridor availability |
| Driver pricing | 4,900/8,900/14,900 demo values | 19,000/28,000/38,000 business values |
| Backend | rides/corridors/bookings | recurring routes, monthly plans, corridor schedules |
| AI narrative | explanation screen / marketing | route clustering, pre-matching, forecasting as future systems |

## What Should Not Change Yet

Do not delete the current ride-hailing flow.

Keep it as:

```text
fallback/acquisition/demo infrastructure
```

Reason:
- existing code already works
- live ride/offers/WebSocket stack is valuable
- fallback/on-demand remains part of Tier 1
- throwing it away would waste useful infrastructure

## Stage Plan

### Stage 0 — Business Foundation

Status: active now.

Outputs:
- passenger flow spec
- driver flow spec
- pricing/subscription logic
- open questions and validation plan

Do not start major UI/backend changes before this stage is complete.

### Stage 1 — Mockup Redesign

Input:

```text
Business_Department/Product_Logic.md
Business_Department/Driver_Economics.md
Business_Department/Business_Open_Questions.md
```

Actions:
- redesign passenger onboarding around monthly commute setup
- redesign passenger home as commute dashboard
- redesign subscription screens around declared routes and savings
- show Solo Corridor, Multi Corridor, and Flex Pack as MVP choices, with Multi Corridor recommended
- redesign driver registration tier step with correct pricing and economics
- mark on-demand as fallback, not primary CTA

### Stage 2 — iOS Product Pivot

Likely files/features:

| Feature | Change |
|---|---|
| `HomeView` | replace “Вызвать такси” center with commute plan/setup |
| `OnboardingView` | multi-route monthly commute declaration |
| `SubscriptionsView` | route-based monthly pass and savings logic |
| `SubscriptionsFeature` | support MVP plan choice: `solo_corridor`, `multi_corridor`, `flex_pack` |
| `CorridorList/Detail` | corridor reliability and hub pickup/dropoff clarity |
| `RideRequest/Search/Offer` | demote to ride-day/fallback state |
| `DriverRegistrationView` | correct tier prices and economics copy |
| `BIRGECore` UI kit | add reusable commute-plan components |

### Stage 3 — Backend/Data Pivot

Likely backend additions:

- recurring route model
- monthly commute plan model
- passenger subscription plan types: `solo_corridor`, `multi_corridor`, `flex_pack`
- passenger route schedule
- corridor schedule / departure window
- underfilled corridor fallback policy
- subscription coverage by route
- driver corridor availability

Do not implement these until business flow and data shape are defined.

### Stage 3A — Technical Pivot Plan

Source:

```text
Context/implementation_plan_.md
Tasks/Technical_Subscription_Pivot.md
Decisions/ADR-005_Subscription_Pivot.md
```

Claude Code audit confirmed the technical strategy:

- keep existing auth/WebSocket/location/TCA/Kaspi infrastructure
- add recurring route and monthly commute plan models with additive migrations
- replace stale passenger plans `free/lite/standard/pro` with `solo_corridor/multi_corridor/flex_pack`
- demote taxi ride request to fallback/secondary navigation
- align driver pricing with Peak Starter 9,900 + Starter 19k + Professional 28k + Premium 38k

Do this incrementally. No rebuild.

### Stage 4 — Validation

Before real launch:

- 10 passenger interviews
- 5 driver interviews
- UvU Shuttle competitor test
- lawyer consultation on shared rides
- ИП/ТОО decision
- Kaspi merchant process

## Immediate Next Decisions

1. What exactly does Данияр see when he opens BIRGE for the first time?
2. How does he enter 4 independent monthly routes?
3. What monthly price does BIRGE show?
4. What confidence/reliability does BIRGE promise before payment?
5. What does the driver see before accepting corridor work?
6. What happens if a corridor has only 1 passenger?

## Rule For Future Agents

If a task adds UI or backend behavior, ask:

```text
Does this serve subscription commute, or does it deepen taxi-first behavior?
```

If it deepens taxi-first behavior, pause and ask Arsen before building.
