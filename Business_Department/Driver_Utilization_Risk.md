---
status: active
version: v1.0
last_updated: 2026-05-09
source: Business chat Session 7 follow-up — driver utilization risk
---

# BIRGE Driver Utilization Risk

## Core Problem

BIRGE starts from recurring commute corridors.
Commute demand is naturally peak-hour:

```text
06:00-10:00   high demand: home -> work
10:00-17:00   low BIRGE demand
17:00-21:00   high demand: work -> home
21:00-23:00   low demand
```

This creates a driver utilization risk.

If BIRGE gives drivers only morning/evening work, then BIRGE is not a full-day replacement for Yandex/InDrive at launch.
That is acceptable, but it must be positioned honestly.

## Strategic Conclusion

MVP driver positioning:

```text
BIRGE is predictable morning and evening corridor work with 0% commission.
It is not a full-day Yandex replacement at launch.
Drivers can keep using Yandex/InDrive during the day.
```

Short version:

```text
BIRGE = peak-hour income layer
```

This reduces false expectations and makes the subscription model more credible.

## Economics Snapshot

Assumptions:

```text
Route: Alatau City -> Esentai / Al-Farabi
Fare per passenger/run: ~1,900 KZT
Cost per run: ~700 KZT
Working days/month: 22
Yandex/InDrive monthly net baseline: ~390,000 KZT
```

| Runs/day | Passengers/run | Net/month before subscription | Net after Starter 19k | Net after Professional 28k | Interpretation |
|---:|---:|---:|---:|---:|---|
| 2 | 2 | 136,400 KZT | 117,400 KZT | 108,400 KZT | Too weak |
| 2 | 3 | 220,000 KZT | 201,000 KZT | 192,000 KZT | Supplemental only |
| 2 | 4 | 303,600 KZT | 284,600 KZT | 275,600 KZT | Still below full-time Yandex baseline |
| 4 | 2 | 272,800 KZT | 253,800 KZT | 244,800 KZT | Weak unless supplemental |
| 4 | 3 | 440,000 KZT | 421,000 KZT | 412,000 KZT | Roughly viable vs Yandex baseline |
| 4 | 4 | 607,200 KZT | 588,200 KZT | 579,200 KZT | Strong |
| 6 | 2 | 409,200 KZT | 390,200 KZT | 381,200 KZT | Barely comparable but operationally hard |
| 6 | 3 | 660,000 KZT | 641,000 KZT | 632,000 KZT | Strong but requires dense corridor |
| 6 | 4 | 910,800 KZT | 891,800 KZT | 882,800 KZT | Excellent but not MVP baseline |

Practical MVP baseline:

```text
4 runs/day = 2 morning + 2 evening.
At 3 passengers/run, driver can be around Yandex baseline while getting predictability and 0% commission.
```

## Confirmed Pricing Decision

Add a fourth driver plan:

| Plan | Price | Purpose |
|---|---:|---|
| Peak Starter | 9,900 KZT/month | Entry plan for peak-hour BIRGE work parallel to Yandex/InDrive |
| Starter | 19,000 KZT/month | Regular morning/evening corridor work; exact differentiation from Peak Starter needs final product definition |
| Professional | 28,000 KZT/month | Recommended after 2+ active corridors / stronger peak-hour coverage |
| Premium | 38,000 KZT/month | Maximum corridor load in peak windows; not an MVP default |

Important:

```text
Professional should not be pushed too early.
Offer upgrade when the driver has enough active corridors to justify it.
```

## Subscription Activation Rule

Still confirmed:

```text
Driver subscription starts only when the first active corridor is ready.
```

Do not charge drivers for unused platform access while corridor demand is still being built.

## Driver Promise

BIRGE may promise:

- predictable morning/evening work blocks
- recurring planned routes
- 0% commission in BIRGE corridors
- no exclusivity; driver can work elsewhere outside BIRGE corridor windows
- week-ahead schedule once corridors are active

BIRGE must not promise:

- full-day replacement for Yandex/InDrive
- guaranteed monthly income
- always-full corridors
- "quit Yandex"
- stable income from day one

## UI / Mockup Implications

Driver onboarding should say:

```text
Утро и вечер — по плану. Остальное время — твоё.
```

Service area screen should show:

```text
Утро: 06:00-10:00
Вечер: 17:00-21:00
День: скоро / по мере роста платформы
```

Subscription screen should show:

```text
Peak Starter — 9,900 ₸/мес
Для работы в пиковые часы параллельно с другими платформами.
```

Required disclaimer:

```text
BIRGE запускает плановые коридоры в утренние и вечерние часы.
Дневные коридоры появятся по мере роста платформы.
```

Avoid:

- guaranteed monthly income
- direct Yandex income comparison in UI
- large earnings calculator in onboarding
- "full-day" language

## Phase 2 Opportunities

Potential future ways to reduce midday underutilization:

| Opportunity | MVP? | Notes |
|---|---|---|
| Corporate lunch / office shuttle corridors | Later | Good B2B fit, but operationally separate |
| University corridors | Later | Good second segment after office workers |
| Airport / train station scheduled corridors | Later | Different demand pattern |
| B2B employee shuttle contracts | Later | Strong but sales-heavy |
| Parcel/courier midday utilization | No | Brand dilution and operational/legal complexity |
| School pickup corridors | No | High legal/safety complexity |
| Medical/clinic trips | No | High operational and liability risk |

## Open Questions

| ID | Question | Priority |
|---|---|---|
| OQ-D1 | Final differentiation between Peak Starter and legacy Starter | High |
| OQ-D2 | Exact rules for Professional upgrade after 2+ active corridors | High |
| OQ-D3 | How to prevent BIRGE corridor conflicts with Yandex/InDrive jobs | Medium |
| OQ-D4 | Retention dashboard: MVP or Phase 2? | Medium |
| OQ-D5 | Corporate lunch/university corridors: which is the first Phase 2 segment? | Low |

