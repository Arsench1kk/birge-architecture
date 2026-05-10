---
status: active
version: v1.2
last_updated: 2026-05-09
source: Context/Universal_AI_Handoff_BIRGE.md v5.0 + Business Session 7 + Driver Utilization Risk
---

# BIRGE Driver Economics

## Strategic Rule

Driver monetization is:

```text
fixed monthly subscription
zero per-ride commission
```

This is a permanent strategic promise and the core driver narrative.

## Driver Subscription Tiers

Canonical business pricing:

| Tier | Monthly Fee |
|---|---:|
| Peak Starter | 9,900 KZT |
| Starter | 19,000 KZT |
| Professional | 28,000 KZT |
| Premium | 38,000 KZT |

Important implementation conflict:

```text
Claude Code fixed the app repo driver prices to 19,000 / 28,000 / 38,000 KZT.
Business direction now adds Peak Starter at 9,900 KZT.
Implementation must be reviewed again before driver pricing is coded/committed.
```

### Tier Logic

| Tier | Business meaning |
|---|---|
| Peak Starter | Entry plan for morning/evening BIRGE corridor work while the driver keeps Yandex/InDrive during the day |
| Starter | Regular morning/evening corridor work; exact distinction from Peak Starter must be finalized before implementation |
| Professional | Recommended only when driver has enough active corridors, likely 2+ active corridors |
| Premium | Maximum peak-window corridor load; not MVP default |

Do not push Professional too early. Upgrade should be offered when active corridor coverage justifies the higher subscription.

## Session 7 — Driver Product Decisions

These decisions are now confirmed for driver onboarding and driver mockups.

| Decision | Current Truth |
|---|---|
| First driver persona | Серик: 32-42, full-time Almaty taxi driver, owns CNG Hyundai Accent / Cobalt / Nexia, values predictability over risky upside |
| Parallel work with Yandex/InDrive | Allowed. Do not require exclusivity at launch. |
| Minimum vehicle year | 2000 or newer for MVP mockups/business logic |
| Verification failure | Repeat submission / correction flow, not permanent rejection by default |
| Driver subscription activation | Subscription billing starts only when the driver's first corridor becomes active |
| Cancellation / no-show protection | Passenger late cancellation/no-show consumes a ride/slot and should compensate/protect the driver |
| Driver utilization positioning | BIRGE is peak-hour income layer at MVP, not full-day Yandex replacement |
| Peak Starter plan | Confirmed at 9,900 KZT/month |

Rationale for minimum vehicle year 2000:

```text
Arsen confirmed 2000+ as the MVP rule. This is intentionally broad for early driver supply.
Quality risk remains and should be managed with manual inspection, document checks, passenger ratings, and potential later tightening.
```

## Driver Value Proposition

Promise clearly:

- predictable planned corridor work
- recurring passengers and known commute windows
- fixed monthly platform subscription
- 0% commission from rides
- less idle time and fewer empty kilometers when corridor density exists
- BIRGE covers peak commute windows first; driver may keep other platforms during the day

Do not promise:

- guaranteed monthly income
- always-full corridors
- immediate replacement for Yandex/InDrive
- full-day utilization
- 2x earnings or other unsupported claims

Canonical explanation of 0% commission:

```text
Ты платишь фиксированную подписку раз в месяц.
Всё что платят пассажиры за поездки — твоё полностью.
BIRGE не берёт процент с каждого рейса.
```

## Driver Subscription Activation Rule

Billing must not start immediately after registration or document verification.

Confirmed rule:

```text
Driver subscription activates when the driver's first corridor becomes active.
Before that, the driver can pass onboarding, verification, and corridor matching without paying for unused access.
```

UI implication:

```text
Оплата начнётся, когда ваш первый коридор будет активирован.
```

This protects BIRGE from the "driver pays for air" cold-start problem.

## Driver Utilization Risk

BIRGE commute demand is naturally peak-hour:

```text
06:00-10:00  morning commute demand
10:00-17:00  low BIRGE demand at MVP
17:00-21:00  evening commute demand
```

This means BIRGE must not position itself as a full-day replacement for Yandex/InDrive at MVP.

Correct MVP driver positioning:

```text
BIRGE — это предсказуемые утро и вечер без комиссии.
Не замена Yandex на весь день, а дополнительный плановый слой дохода.
```

Mockup copy direction:

```text
Утро и вечер — по плану. Остальное время — твоё.
```

See also:

```text
Business_Department/Driver_Utilization_Risk.md
```

## Passenger Cancellation / No-Show Policy

Working policy for driver trust:

| Passenger action | Consequence |
|---|---|
| Cancels more than 2 hours before ride | Free cancellation, slot returns |
| Cancels 30 min to 2 hours before ride | First time warning/free; repeated cases may consume slot |
| Cancels less than 30 minutes before ride | One ride/slot is consumed |
| No-show | One ride/slot is consumed + warning |
| 3 no-shows in one month | Temporary suspension/review of corridor subscription |

Driver-facing principle:

```text
If a passenger causes a late cancellation/no-show, the driver should still be protected through the passenger subscription/slot rules.
```

Do not overload driver onboarding with the full cancellation table. Put the detailed rules in FAQ/help and operational screens.

## Route Economics Model

Route assumption:

```text
Alatau City -> Esentai / Al-Farabi
~22-25 km
~45 min
CNG car
```

Working table:

| Passengers | Driver revenue/run | Driver net profit/run |
|---:|---:|---:|
| 1 | ~1,900 KZT | ~1,200 KZT |
| 2 | ~3,800 KZT | ~3,000 KZT |
| 3 | ~5,700 KZT | ~4,900 KZT |
| 4 | ~7,600 KZT | ~6,800 KZT |

Yandex Go comparison:

```text
~2,800 KZT solo fare
minus ~19% commission
= ~2,270 KZT driver net
```

Interpretation:

| Occupancy | Meaning |
|---|---|
| 1 passenger | Driver may lose vs Yandex Go |
| 2 passengers | Roughly viable / near Yandex Go |
| 3 passengers | Clear driver advantage |
| 4 passengers | Strong driver advantage |

## Cold-Start Risk

The driver pitch is only true when corridor density exists.

Risk:

```text
In early months BIRGE cannot guarantee 3 passengers/run.
Driver may earn less than on Yandex Go.
```

Therefore driver acquisition cannot simply say “earn more”.

It must explain:

- free trial
- expected corridor occupancy
- what happens if the route is underfilled
- whether BIRGE offers a guarantee/subsidy
- how long the pilot phase lasts
- what driver commitment is expected

## Guarantee Policy Status

Older architecture docs mention a 90-day 1.5x Corridor Performance Guarantee.

Current status:

```text
Not finalized.
Treat as a candidate strategy, not approved business truth.
```

Before using it in UI, investor materials, or legal docs, decide:

1. Can BIRGE afford the subsidy?
2. What exact run qualifies?
3. How is fraud prevented?
4. Is guarantee paid in cash, credit, or subscription discount?
5. Does it create legal/employment obligations?

## Validation Needed

Before real driver launch:

- interview 5-10 real Almaty taxi drivers
- verify willingness to pay Peak Starter 9.9k and higher 19k/28k/38k subscriptions
- test reaction to zero commission narrative
- test reaction to corridor-only work
- test reaction to free trial
- calculate monthly breakeven at real fuel/maintenance assumptions
- validate how many drivers multi-home across Yandex/inDrive/Maxim

## Product Implications

Driver UI must show:

- subscription instead of commission
- clear fee and billing date
- earnings scenarios by occupancy
- assigned corridor runs
- passenger count per run
- underfilled-run policy
- projected weekly/monthly net
- trust: payouts, tax receipts, support, safety
- subscription activation timing: only after first active corridor
- no exclusivity requirement with Yandex/InDrive during launch
- verification correction flow if documents fail
