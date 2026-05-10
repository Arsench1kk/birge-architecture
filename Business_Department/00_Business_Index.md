---
status: active
version: v1.3
last_updated: 2026-05-09
source: Context/Universal_AI_Handoff_BIRGE.md v5.0
---

# BIRGE Business Department

This folder is the business/product source of truth for BIRGE.

The current foundation:

> BIRGE is not a taxi-first app. BIRGE is a transportation-as-a-subscription platform for recurring Almaty commuters.

Core wedge:

```text
Tier 3: subscription commute
```

Secondary products:

```text
Tier 2: pay-as-you-go shared corridor
Tier 1: on-demand ride fallback / acquisition channel
```

## Canonical Business Rules

1. Subscription commute is the primary product.
2. On-demand taxi must not dominate the product narrative.
3. Driver monetization is fixed monthly subscription only.
4. Zero per-ride commission is a permanent strategic promise.
5. Corridor MVP is hub-to-hub: one pickup node, one dropoff node, no intermediate stops.
6. First corridor: Alatau City / Алатауский район -> Esentai / Al-Farabi, morning 07:15-08:30.
7. Passenger can declare multiple independent monthly routes.
8. Each declared route is matched independently.
9. Business logic must be clear before UI mockups and before new implementation work.
10. AI is future optimization, not the launch product promise unless it has a concrete function.
11. Driver subscription billing starts only after the driver's first active corridor.
12. Driver minimum vehicle year for MVP mockups/business logic is 2000 or newer.
13. Drivers may keep working on Yandex/InDrive during launch.
14. Passenger late cancellation/no-show should consume a ride/slot and protect the driver.
15. Driver MVP positioning is peak-hour income layer, not full-day Yandex replacement.
16. Peak Starter is the fourth driver plan at 9,900 KZT/month.
17. Real operations are legally blocked until legal entity, transport classification, insurance, public offer/refund policy, and privacy/data requirements are handled.

## Department Files

| File | Purpose |
|---|---|
| [[Business_Department/Product_Logic]] | Canonical product model, tiers, user flows, UI/tech implications |
| [[Business_Department/Subscription_Strategy]] | Passenger subscription packages: Solo Corridor, Multi Corridor, Flex Pack |
| [[Business_Department/Driver_Economics]] | Driver subscription economics, pricing, breakeven, cold-start risk |
| [[Business_Department/Driver_Utilization_Risk]] | Peak-hour driver utilization risk and Peak Starter positioning |
| [[Business_Department/Legal_Compliance_Risk_Assessment]] | AI-generated legal/compliance risk map and launch blockers |
| [[Business_Department/Business_Open_Questions]] | Critical unknowns, legal blockers, validation questions |
| [[Business_Department/Project_Change_Plan]] | How the current taxi-ish MVP should pivot toward subscription commute |
| [[Tasks/Business_Sprint_0_Foundation]] | Immediate business/product work plan before UI/tech expansion |
| [[Context/Legal_Advisor_Prompt_BIRGE]] | Prompt for legal/compliance AI review before real launch |

## Current Priority

Current priority is not “build more screens”.

Current priority:

```text
Define passenger and driver business flows screen-by-screen, then redesign mockups from that logic.
```

Order:

1. Passenger subscription commute flow.
2. Driver subscription/onboarding/economics flow.
3. Open Design driver onboarding mockups from confirmed Session 7 rules.
4. Passenger subscription pricing for all three MVP tiers: Solo Corridor, Multi Corridor, Flex Pack.
5. Legal/compliance review before real launch; see [[Business_Department/Legal_Compliance_Risk_Assessment]].
6. iOS/backend implementation changes.
