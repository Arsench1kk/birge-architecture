---
status: active
version: v1.2
last_updated: 2026-05-09
source: Context/Universal_AI_Handoff_BIRGE.md v5.0 + Business Session 7 + Driver Utilization Risk
---

# BIRGE Product Logic

## Product Identity

BIRGE is a **transportation-as-a-subscription** service for recurring commuters in Almaty.

It is not a taxi-first app.

The user promise:

```text
Declare your recurring monthly routes once.
BIRGE pre-plans matching drivers and co-passengers.
Pay a subscription.
Stop thinking about daily taxi ordering.
```

## Three Tiers

These are service tiers, not passenger subscription packages.

| Tier | Product | Strategic Role |
|---|---|---|
| Tier 3 | Subscription commute | Primary wedge |
| Tier 2 | Shared corridor ride, pay-as-you-go | Secondary/fallback for non-subscribers |
| Tier 1 | On-demand ride | Acquisition and service fallback only |

## Passenger Subscription Packages

Passenger subscription packages live inside the Tier 3 subscription commute product.

| Subscription Tier | Name | Coverage | Role |
|---|---|---|---|
| Тир 1 | Solo Corridor | One route, one time slot per day | MVP plan for users with one predictable commute |
| Тир 2 | Multi Corridor | All pre-declared corridors covered | MVP recommended/default plan for Данияр |
| Тир 3 | Flex Pack | N rides/month with fixed per-ride deduction | MVP plan for less predictable users |

Confirmed decisions:

- MVP launches with **all three passenger subscription tiers**.
- Тир 1 and Тир 2 have zero per-ride friction after subscription payment.
- Тир 3 is a pack balance product, not a full monthly route subscription.
- On-demand taxi is outside subscription and remains fallback only.
- Exact KZT prices are still TBD in Pricing & Unit Economics work.

## Core Passenger Model

The passenger does not create one generic “ride”.

The passenger creates multiple independent recurring routes:

```text
Route 1: Home -> Work      5x/week, morning
Route 2: Work -> Home      2x/week, evening
Route 3: Work -> Gym       3x/week, evening
Route 4: Gym -> Home       3x/week, late evening
```

Each route becomes its own corridor with independent matching, pricing, schedule, and reliability status.

## MVP Corridor Model

MVP is hub-to-hub:

```text
one fixed pickup point
one fixed dropoff point
no intermediate stops
max ~100m walk to pickup node
```

First corridor:

```text
Alatau City / Алатауский район -> Esentai / Al-Farabi
Morning window: 07:15-08:30
```

## First Target Persona

Persona name: Данияр.

| Field | Value |
|---|---|
| Age | 28-30 |
| Role | Middle-level office worker |
| Income | 600k-800k KZT/month |
| Home | Alatau City |
| Work | Esentai / Al-Farabi |
| Current transport | Yandex Go / taxi only |
| Pain | ~129k KZT/month transport cost, daily ordering friction, waiting |
| BIRGE promise | 35-40% monthly savings and predictable commute |

This persona is ideal because his life is predictable enough for monthly route commitment.

## What The UI Must Change Toward

Current app surfaces still feel too on-demand/taxi-first.

Future passenger UI should make these concepts primary:

1. Monthly commute setup.
2. Add multiple recurring routes.
3. See corridor reliability before paying.
4. Understand fixed pickup/dropoff hubs.
5. Understand monthly savings vs current taxi spend.
6. See a calendar/month plan, not only a single ride request.
7. Treat on-demand as fallback, not homepage hero.

## Passenger Flow To Define Next

Before more UI mockups:

1. Welcome / positioning: BIRGE as monthly commute plan.
2. Phone/OTP.
3. Monthly commute onboarding.
4. Add Home -> Work route.
5. Add Work -> Home / Work -> Gym / Gym -> Home optional routes.
6. Confirm pickup/dropoff hubs.
7. Pick time windows and weekdays.
8. See corridor match/reliability estimate.
9. See monthly price and savings.
10. Subscribe / activate via Kaspi.
11. Dashboard: today’s commute plan.
12. Ride day: driver/co-passenger/status flow.
13. Fallback: if corridor underfilled, convert to on-demand dispatch.

## Driver Flow To Define Next

Session 7 confirmed the first driver-side product foundation.

First driver persona:

```text
Серик, 32-42, full-time Almaty taxi driver, owns a CNG Hyundai Accent / Cobalt / Nexia,
works across Yandex Go / InDrive, values predictable planned income more than risky upside.
```

Core driver positioning:

```text
BIRGE Driver is not a random order feed.
It is an operating system for planned recurring commute corridors.
Drivers can keep working on Yandex/InDrive during launch; BIRGE must not demand exclusivity.
```

Confirmed driver onboarding/business rules:

| Rule | Decision |
|---|---|
| Minimum vehicle year | 2000 or newer |
| Driver subscription billing | Starts after first active corridor, not immediately after verification |
| Verification failure | Show what failed and allow repeat submission/correction |
| Passenger no-show / late cancellation | Passenger loses slot/ride; driver should be protected |
| Yandex/InDrive parallel work | Allowed |
| MVP driver positioning | Peak-hour income layer, not full-day Yandex replacement |
| Entry driver plan | Peak Starter — 9,900 KZT/month |

Driver onboarding screens:

1. Driver value proposition: keep 100% of fares, pay fixed subscription.
2. Phone login and OTP.
3. Personal profile.
4. Vehicle profile and seats.
5. Documents and compliance.
6. Preferred service area / recurring corridors.
7. Driver subscription plan.
8. Verification review.
9. Later dashboard: assigned corridor runs, occupancy, projected earnings.
10. Later active run lifecycle.
11. Later earnings and subscription billing.

Driver UI copy must avoid:

- guaranteed income claims
- "start earning instantly"
- "quit Yandex"
- surge/bidding/random dispatch language
- unsupported comparisons like "2x more"

Canonical driver subscription copy:

```text
Оплата начнётся, когда ваш первый коридор будет активирован.
0% комиссии с поездок.
```

Canonical driver utilization copy:

```text
Утро и вечер — по плану. Остальное время — твоё.
```

Driver onboarding must not imply full-day BIRGE demand at launch.
Morning/evening peak windows should be visible and honest.

## Implications For Existing Code

Do not rewrite the app immediately.

But future changes should pivot:

| Current Bias | Needed Direction |
|---|---|
| Home CTA: “Вызвать такси” | Monthly commute setup / today’s commute plan |
| Single ride request | Recurring route declaration |
| Corridor booking as one-off | Monthly corridor pass |
| Driver registration demo pricing | Business-approved Peak Starter 9.9k + 19k/28k/38k pricing |
| AI explanation screen | Concrete route/corridor planning value |
| Subscription screen generic tiers | Monthly commute pass with route coverage and savings |
| All passenger plans shown without guidance | MVP shows all three; Multi Corridor is recommended/default |

## Definition Of Ready For UI Mockups

UI mockup work is ready only when these are defined:

- passenger screen-by-screen flow
- driver screen-by-screen flow
- pricing/subscription logic
- first corridor service rules
- underfilled corridor fallback rules
- copy language for “not taxi, commute subscription”
