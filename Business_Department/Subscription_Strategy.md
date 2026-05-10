---
status: active
version: v1.0
last_updated: 2026-05-08
source: Session 6 subscription update, extracted 2026-05-07
---

# BIRGE Subscription Strategy

This file is the canonical business source for BIRGE passenger subscription tiers.

Important taxonomy:

```text
Service tiers != passenger subscription tiers.
```

Service tiers describe the whole product model:

```text
Tier 1: on-demand fallback
Tier 2: shared corridor pay-as-you-go
Tier 3: subscription commute wedge
```

Passenger subscription tiers describe packages inside the subscription commute business.

## Confirmed Passenger Subscription Tiers

Kaspi is the primary payment method for Kazakhstan.

| Tier | Name | What's Covered | For Whom |
|---|---|---|---|
| Тир 1 | Solo Corridor | One route, one time slot per day | Single predictable commute, e.g. only Home -> Work |
| Тир 2 | Multi Corridor | All pre-declared corridors covered | Main target user: Данияр with 4 routes |
| Тир 3 | Flex Pack | Fixed pack of N rides/month, fixed per-ride price, pay per use | Unpredictable schedule, remote workers, frequent travelers |

## Confirmed Decisions

- Тир 1 and Тир 2 have zero per-ride friction: user pays subscription, gets in the car, done.
- Тир 3 deducts a fixed per-ride price from pack balance.
- Тир 3 exists for users who cannot commit to a full monthly schedule.
- On-demand taxi remains fallback outside subscription; it is not a subscription tier.
- MVP launches with **all three passenger subscription tiers**: Solo Corridor, Multi Corridor, and Flex Pack.
- Multi Corridor is the recommended/default plan for the first target persona Данияр, but Solo Corridor and Flex Pack are available from MVP.
- Specific KZT prices are not finalized yet and belong to Session 8: Pricing & Unit Economics.

## MVP Subscription Scope

MVP:

```text
Solo Corridor + Multi Corridor + Flex Pack.
```

Why:

- Solo Corridor gives a simple entry point for one predictable commute.
- Multi Corridor proves the strongest transportation-as-a-subscription promise for Данияр-type users.
- Flex Pack captures users with less predictable schedules without pushing them into on-demand.
- Launching all three lets pricing/user research compare real subscription preferences from day one.

MVP plan emphasis:

```text
Multi Corridor is recommended/default.
Solo Corridor and Flex Pack are real available alternatives.
On-demand taxi is not a subscription package.
```

## UI Implications

For the first redesigned mockups:

- Show all three subscription tiers as available in MVP.
- Lead with Multi Corridor as the recommended/default plan for Данияр-type users.
- Make Solo Corridor feel simpler/cheaper and Flex Pack feel flexible/pay-per-use.
- Show what routes are covered by the monthly subscription.
- Show that the user has no per-ride payment friction inside the subscription.
- Keep Kaspi as the payment method, but do not invent exact prices yet.

## Backend/Data Implications

Future data model likely needs:

- subscription plan type: `solo_corridor` / `multi_corridor` / `flex_pack`
- covered recurring route ids
- monthly billing period
- route coverage status
- `flex_pack` ride balance and fixed per-ride deduction
- Kaspi payment state

Do not finalize Flex Pack balance amounts until the pricing/unit economics session defines pack sizes, expiry, refunds, and unused-ride behavior.
