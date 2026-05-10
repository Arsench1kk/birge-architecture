---
id: ADR-005
status: Accepted
date: 2026-05-08
deciders: Arsen + Business/Tech planning
---

# ADR-005 — Subscription Commute Product Pivot

## Context

BIRGE originally had a strong taxi-first / ride-hailing MVP shape:

```text
open app -> request ride -> search driver -> offer -> active ride
```

After business sessions, the product foundation changed:

```text
BIRGE = transportation-as-a-subscription for recurring Almaty commuters.
```

The passenger should declare monthly routes, BIRGE should pre-plan drivers and co-passengers, and subscription commute should be the main product. On-demand ride remains useful, but only as fallback/acquisition.

## Decision

BIRGE will pivot technical product direction toward subscription commute.

Non-negotiable rules:

- Keep native iOS + SwiftUI + TCA.
- Keep Swift Vapor + PostgreSQL + Redis.
- Do not rebuild from scratch.
- Preserve existing ride/WebSocket/location/payment infrastructure as reusable fallback/demo infrastructure.
- Add new commute/subscription models and flows incrementally.
- MVP passenger subscription includes all three tiers: `solo_corridor`, `multi_corridor`, `flex_pack`.
- `multi_corridor` is recommended/default for the first persona, but not the only MVP tier.
- Driver monetization remains fixed monthly subscription with zero per-ride commission.

## Consequences

Current taxi-first surfaces become secondary:

- `RideRequest`
- `Searching`
- `OfferFound`
- active ride map

New primary surfaces are needed:

- recurring route declaration
- commute setup
- commute dashboard
- route-covered subscription plan selection
- driver corridor availability
- subscription-aware payments and billing

Backend must evolve with additive migrations:

- `RecurringRoute`
- `MonthlyCommutePlan`
- `CommutePlanRoute`
- corridor schedule fields
- ride links to corridor/route/plan
- driver subscription and corridor availability fields

## Implementation Approach

Use phased, small commits:

1. Documentation/code alignment.
2. Backend data model and API design.
3. Passenger subscription commute flow.
4. Driver subscription and corridor availability.
5. Pricing/Kaspi alignment.
6. Fallback/on-demand integration.
7. Tests and cleanup.

## Open Business Dependencies

- Exact KZT prices for Solo/Multi/Flex.
- Flex Pack pack size, expiry, refund, unused ride logic.
- Underfilled corridor fallback policy.
- Driver guarantee/trial terms.
- Kaspi merchant/legal entity readiness.

## Related

[[Business_Department/Product_Logic]]
[[Business_Department/Subscription_Strategy]]
[[Business_Department/Project_Change_Plan]]
[[Tasks/Technical_Subscription_Pivot]]
[[Context/implementation_plan_]]

