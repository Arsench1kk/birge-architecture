# BIRGE Driver Product Mockup Roadmap

Last updated: 2026-05-09

This is the working roadmap for rebuilding BIRGE driver Open Design mockups under the current subscription-first product direction.

## Product Positioning

BIRGE Driver is not a taxi marketplace dashboard. It is the driver-side operating system for subscription commute corridors in Almaty.

Driver product foundation:
- drivers pay a fixed monthly BIRGE subscription / platform fee model, not per-ride commission
- zero per-ride commission ever
- driver value is predictable corridor work, recurring passengers, route density, lower idle time, and clear monthly economics
- MVP corridors are hub-to-hub: one pickup node, one dropoff node, no intermediate stops
- driver UI should avoid surge, bidding, offer-shopping, gamified urgency, and casino-like earnings language
- driver screens should make corridor commitments, schedule reliability, safety, support, payout, and compliance clear

Design language:
- iOS native, calm, operational, professional
- driver brand navy `#021638` with passenger blue `#037AFB` only as an accent when needed
- no pictographic emoji
- no external images
- no meaningless badges
- yellow/amber only for true warnings

## Passenger Mockup Lesson

The passenger mockups are not being rewritten now.

Current decision:

```text
Keep passenger mockups as-is.
Use them as a visual-quality benchmark, not as an information-density benchmark.
```

Why:
- passenger mockups took significant effort and already cover the product deeply
- they are visually polished and useful as a broad product prototype
- but UX feedback showed they are too dense: too much text, too many blocks, unclear next tap on some screens

Driver screens must therefore be:

```text
as polished as passenger screens
much less dense than passenger screens
```

Driver mockup principle:
- preserve premium device frame, spacing, native surfaces, refined typography
- reduce content density by roughly 50%
- one screen = one task
- one obvious primary CTA
- max two secondary actions
- max 3-4 major content blocks
- no long educational/product-pitch copy
- no full explanation of the BIRGE model inside onboarding screens

## Open Design Locations

Passenger improved mockups are checkpointed here:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/passenger
```

Driver legacy references live here and must not be overwritten:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/legacy-birge-mockups/driver
```

Create improved driver files in a separate working copy, preferably:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/driver
```


## Driver Mockup Checkpoint — 2026-05-09

Current Open Design state:
- improved driver mockups are complete as a broad product prototype: `D-00` through `D-46`
- index: `/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/driver/index.html`
- folder: `/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/driver`

Covered phases:
- Phase 1: entry, phone/OTP, profile, vehicle, documents, service zones, plan selection, verification review
- Phase 2: today/home, schedule, corridor detail, monthly commitment, availability, route preferences
- Phase 3: predeparture, pickup navigation, arrival, boarding, in-progress corridor, dropoff, next corridor
- Phase 4: late driver, passenger no-show, pickup change, replacement handoff, vehicle issue, traffic delay, emergency stop, corridor cancelled
- Phase 5: earnings, corridor income detail, driver subscription status, payout method/history/detail, documents/tax note, monthly summary
- Phase 6: support inbox, ticket detail, live support, safety center, incident report, passenger preferences, profile, settings, privacy/data

Final polish decisions:
- bottom tabs: `Сегодня / График / Доход / Профиль`
- profile tab destination: `D-44-profile.html`
- no `Подписка` tab and no `Support` tab
- driver avatar system uses circular initials `СА`
- row actions prefer iOS-style SVG icons instead of trailing action words
- `D-06` includes `Рядом с домом`, `точка выезда`, and `Не важно`
- driver plans include Peak Starter `9,900 KZT/month`, Starter `19,000`, Professional `28,000`, Premium `38,000`

Current decision:

```text
Treat improved/driver as checkpointed.
Do targeted fixes only.
Do not run broad driver rewrites unless a specific issue is found.
```

## Phase 0 — Driver Product Logic Review

Goal: make sure driver screens are not built from old taxi-first economics.

Confirmed Session 7 decisions:
- first driver persona: Серик, 32-42, full-time Almaty taxi driver with own CNG car, values predictability and stability
- minimum vehicle year: 2000 or newer
- no driver exclusivity at launch; Yandex/InDrive parallel work is allowed
- driver subscription activates only when the first corridor becomes active
- failed verification should lead to correction/re-upload, not permanent rejection by default
- passenger late cancellation/no-show should consume a ride/slot and protect the driver
- driver utilization positioning: BIRGE is peak-hour income layer at MVP, not full-day Yandex replacement
- Peak Starter plan: 9,900 KZT/month

Still clarify before later UI batches:
- exact driver compensation accounting after passenger no-show
- exact KYC rejection reasons/retry limits
- legal status of paid shared rides in Kazakhstan
- whether 2000+ vehicle rule tightens after MVP
- final differentiation between Peak Starter and Starter

## Phase 1 — Driver Entry And Onboarding

Create or redesign:
- launch / driver entry
- phone login and OTP
- personal profile
- vehicle profile
- documents and verification
- service area / preferred corridors
- driver subscription plan selection
- verification review / pending approval

Rule: do not frame onboarding as “start earning instantly”. Frame it as joining planned corridor work after verification.

Important copy constraints:
- show `0% комиссии с поездок`
- show `Оплата начнётся, когда ваш первый коридор будет активирован`
- show `Утро и вечер — по плану. Остальное время — твоё.` where useful
- do not show guaranteed income
- do not ask the driver to quit Yandex/InDrive
- do not imply all corridors are already full
- do not imply full-day BIRGE demand at launch

## Phase 2 — Driver Home And Corridor Schedule

Create or redesign:
- driver home / today overview
- upcoming corridor schedule
- corridor detail
- monthly commitment summary
- availability calendar
- pause availability
- route preference setup

This should be the driver core, equivalent to passenger route management.

## Phase 3 — Corridor Assignment And Ride-Day Operations

Create or redesign:
- assigned corridor overview
- pre-departure checklist
- navigation to pickup node
- arrived at pickup node
- passenger boarding / code verification
- in-progress corridor
- dropoff / completion
- next corridor preview

Rule: no offer-shopping energy. This is planned work, not random gig dispatch.

## Phase 4 — Operational Edge Cases

Create or redesign:
- driver running late
- passenger missing / no-show
- pickup point changed
- replacement driver handoff
- vehicle issue
- traffic delay
- emergency stop / safety issue
- corridor cancelled by BIRGE

Tone: calm, procedural, and support-backed.

## Phase 5 — Earnings, Subscription, And Payouts

Create or redesign:
- monthly earnings overview
- corridor income detail
- driver subscription status
- payout method
- payout history
- payout detail
- tax/document note if needed
- monthly performance summary

Rule: explain economics through predictable monthly corridor work. Do not show BIRGE taking a per-ride commission.

## Phase 6 — Support, Safety, And Account

Create or redesign:
- support inbox
- ticket detail
- live support during corridor
- safety center
- incident report
- passenger preference/block management
- profile
- settings
- privacy / data controls

## Current Next Step

Driver mockups are checkpointed as a broad product prototype.

Immediate mockup priority:
- do not rewrite `improved/driver/` broadly
- preserve `improved/passenger/` as-is
- create passenger cleanup only in a separate working copy, e.g. `improved/passenger-polished/`
- apply driver lessons to passenger carefully: fewer cards, fewer meaningless badges, clearer row icons, compact headers, one obvious primary CTA

Before SwiftUI transfer, choose an approved screen set and map HTML screens to native SwiftUI destinations.
