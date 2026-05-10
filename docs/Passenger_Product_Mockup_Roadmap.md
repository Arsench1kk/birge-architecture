# BIRGE Passenger Product Mockup Roadmap

Last updated: 2026-05-10

This is the checkpointed roadmap for the completed full-product BIRGE passenger Open Design mockup set.

## Product Positioning

BIRGE passenger is a transportation-as-a-subscription product for recurring Almaty commuters.

Primary product:
- monthly commuter routes/corridors
- passenger declares recurring home/work/after-work routes once
- BIRGE pre-plans drivers and co-passengers
- payment supports Apple Pay, saved bank card, Kaspi, and card option depending on payment context
- MVP route model is hub-to-hub: one pickup node and one dropoff node, no intermediate stops
- on-demand taxi is secondary fallback/acquisition, not the main product

Design rule: do not make new screens taxi-first. Daily ordering, surge, per-ride urgency, and offer-shopping patterns should stay out of the primary product.

## Current Approved Passenger Mockups

Final Open Design working copy:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/passenger-polished
```

Current passenger gallery:

```text
improved/passenger-polished/index.html
```

Status: full-product passenger polished mockup pass is complete and ready for final review / SwiftUI mapping.

Screen count preserved: 78 screens.

Original reference folders:
- `improved/passenger/` remains the earlier improved full-product pass.
- `legacy-birge-mockups/` remains legacy reference material.
- `improved/passenger-polished/` is the current approved visual/product prototype.

Current passenger set covers:
- launch, phone, OTP, profile basics, trust permissions, and product intro
- first-time route setup with address/search first, then suggested pickup/dropoff nodes
- subscription tier selection, payment confirmation, activation success, billing, receipts, pause/cancel/change plan
- profile, settings, privacy
- today plan and my routes
- full route management: add/edit route, pickup/dropoff nodes, schedule, paused/low-match/waitlist states
- contextual BIRGE route insights and proactive assistant states
- ride-day planned route lifecycle
- daily commute edge cases and recovery states
- history, completed ride detail, issue reporting
- support inbox, ticket detail, live support, safety center, share status, emergency contact, driver preferences, trip review request
- fallback taxi as a secondary flow outside subscription
- route activation and route ready states

## Final Polish Design Language

The successful polish direction was not “remove all cards”. The successful direction was replacing generic card stacks with explicit native/product compositions.

Use these patterns in future prompts and SwiftUI mapping:
- route canvas / mini map route preview
- ticket/pass surfaces for subscription, confirmation, boarding, and receipts
- account ledger layout with thin dividers for billing/settings/details
- timeline/status rails for activation, delay, pause, cancellation, and support progress
- iOS-style wheel time picker for time selection
- horizontal payment picker/carousel for Apple Pay, saved card, Kaspi, and card option
- icon action rails for call/chat/share/SOS/support actions
- map/node canvas plus native selector for pickup/dropoff node selection
- compact route strips and live activity-like surfaces for ride-day states
- inline insights instead of benefit/tradeoff tables when the meaning is simple

Avoid:
- card stacks as the default layout
- card inside card
- generic row groups for every idea
- meaningless badges
- long explanatory/product-pitch text
- AI/ИИ spectacle or visible reasoning sections
- `ETA`; use `Прибытие около ...` or `До прибытия ...`
- user-facing internal language like `плотность маршрута`, `fallback`, `hub-to-hub`, or business-model explanations

Hard Open Design prompting lesson:

```text
Do not ask Open Design to “make it beautiful”.
Specify the exact visual composition: route canvas, pass/ticket, ledger, timeline rail, wheel picker, payment carousel, live activity surface, or native form.
```

## Final Product/UX Decisions Captured

- Subscription commute is the primary product; on-demand taxi is secondary.
- `Подписка` should not be a bottom tab for passengers; subscription management belongs in profile/account surfaces.
- AI/proactive help should not be a tab or chatbot-first feature; it should appear as compact contextual suggestions.
- The user first enters/searches home/work addresses; BIRGE then suggests nearby pickup/dropoff nodes.
- Different nodes may later affect match chance, walking time, and monthly plan price, but do not show per-ride taxi-like pricing in the subscription flow until pricing logic is defined.
- Core subscription screens should support Apple Pay, saved bank card, Kaspi, and card option where relevant; Kaspi is not forced during registration.
- Solo Corridor and Multi Corridor must not show per-ride pricing. Flex Pack may use ride-pack language. Fallback taxi may show per-ride estimate because it is outside subscription.

## Completed Expansion Principle

Passenger mockups were expanded as a full product, not only an MVP slice.

Completed order:

```text
Registration and trust onboarding
-> first-route setup
-> subscription lifecycle
-> route management
-> daily planned ride lifecycle
-> daily commute edge cases
-> support and safety
-> proactive assistant states
-> fallback taxi as secondary flow
-> route activation/ready states
-> final composition and QA polish
```

## Phase 1 — Registration And Trust Onboarding

Status: complete and polished.

Screens:
- `P-00-splash.html` — launch screen, BIRGE brand, passenger logo/route motif when available, no taxi-first copy.
- `P-01-phone.html` — phone entry with Kazakhstan `+7` format.
- `P-02-otp.html` — SMS confirmation code, resend, change phone.
- `P-02a-profile-basics.html` — name, city Almaty, optional email; native form without outer wrapper card.
- `P-02c-trust-consent.html` — permissions/trust rail: notifications, location, route privacy.
- `P-02d-product-intro.html` — short transition into route setup, distinct from the actual route builder.

Note: `P-02b-kaspi-link.html` may still exist as an inactive file, but it is not part of the current gallery flow. Payment choice now happens after subscription selection, not as forced registration linking.

## Phase 2 — First Route Setup

Status: complete and polished.

Screens:
- `P-03-onboarding.html` — actual start of route builder, not another intro.
- `P-03a-route-origin.html` — enter/search home or pickup address first, then choose suggested pickup node.
- `P-03b-route-destination.html` — enter/search work/destination first, then choose suggested dropoff node.
- `P-03c-route-schedule.html` — iOS-style wheel time picker plus flexibility selector and computed planning window.
- `P-03d-route-review.html` — clean route review before monthly plan.
- `P-03e-monthly-plan.html` — short confirmation transition into subscription selection.

Business note: the setup flow must not imply users can only pick hardcoded BIRGE nodes. Address/search comes first; suggested nodes come second.

## Phase 3 — Subscription Lifecycle

Status: complete and polished.

Covers:
- `subscription-selection.html` — three MVP tiers with Multi Corridor recommended, plus route readiness/value context.
- `subscription-kaspi.html` — payment confirmation, despite legacy filename; payment UI supports Apple Pay, saved card, Kaspi, and card option.
- `subscription-success.html` — subscription active and planning starts.
- `P-17-subscriptions.html`, `P-18-subscription-detail.html` — account/subscription management.
- `P-32-change-plan.html`, `P-33-change-plan-confirm.html` — plan change flow.
- `P-34-payment-failed.html` — calm payment recovery with horizontal payment picker.
- `P-35-billing-history.html`, `P-36-receipt-detail.html` — ledger/receipt style billing.
- `P-37-pause-subscription.html`, `P-38-cancel-subscription.html` — calm pause/cancel flow without warning overload.

Rule: Solo Corridor and Multi Corridor must not show per-ride pricing. Flex Pack can show ride-pack language. Fallback taxi can show per-ride estimate only because it is outside subscription.

## Phase 4 — Route Management

Status: complete and polished.

Covers:
- `P-04-home.html` — today plan/home.
- `P-05-search.html` — My Routes, not generic search.
- `P-24-add-route.html`, `P-25-edit-route.html` — add/edit saved route.
- `P-26-edit-pickup-node.html`, `P-27-edit-dropoff-node.html` — map/node canvas and native selector.
- `P-28-edit-schedule.html` — schedule editing.
- `P-29-route-paused.html`, `P-30-route-low-density.html`, `P-31-route-waitlist.html` — route states and recovery actions.

Business note: route management is more important than generic search. It is the heart of BIRGE subscription value.

## Phase 5 — Daily Planned Ride Lifecycle

Status: complete and polished.

Covers:
- `ride-request.html` — today's planned route detail.
- `P-09-driver-en-route.html` — driver heading to pickup.
- `P-10-driver-arrived.html` — driver at pickup, boarding code next.
- `P-13-boarding-code.html` — prominent code pass.
- `P-11-ride-in-progress.html` — live ride state.
- `P-12-ride-complete.html` — completion/receipt/next route.

Use passenger language: `Прибытие`, `Водитель`, `Посадка`, `Код`, `Поездка завершена`. Do not use visible `ETA`.

## Phase 6 — Daily Commute Edge Cases

Status: complete and polished.

Covers:
- driver late
- passenger running late
- missed pickup
- replacement driver assigned
- pickup point changed
- route delayed due to traffic/weather
- route not confirmed
- no commute today

These screens preserve trust and clarity instead of creating ride-hailing urgency or panic.

## Phase 7 — Support And Safety

Status: complete and polished.

Covers:
- support inbox
- support ticket detail
- live support chat
- safety center
- share route status
- emergency contact setup
- driver preferences
- trip review request when a planned route fails

Support attaches route, driver, time, and subscription context automatically.

## Phase 8 — Proactive BIRGE Suggestions

Status: complete and polished.

Covers contextual states inside Home, Routes, Subscription, and Support:
- repeated route detected
- pickup node reliability suggestion
- better schedule window found
- subscription tier recommendation
- tomorrow commute risk
- renewal reminder
- route matching explanation

Language should stay calm and non-magical: “BIRGE заметил...”, “можно попробовать...”, “выше шанс...”, “ориентировочно...”.

Do not create AI as a bottom tab.

## Phase 9 — Fallback Taxi

Status: complete and polished.

Covers secondary-flow screens:
- fallback taxi entry point
- fallback taxi estimate
- fallback taxi confirmation
- fallback taxi active ride
- fallback taxi complete

This flow must remain visually subordinate to monthly routes and clearly outside subscription.

## Current Next Step

Passenger and driver Open Design mockups are both checkpointed.

Next work should not be broad Open Design rewriting. Recommended next lanes:

1. Final visual QA only if specific issues are found.
2. Define final passenger tier pricing and node/route price impact logic.
3. Prepare legal/compliance consultation packet.
4. Plan SwiftUI transfer from:

```text
improved/passenger-polished/
improved/driver/
```

SwiftUI mapping should preserve the successful composition language: route canvas, pass/ticket, account ledger, timeline rail, wheel picker, payment carousel, icon action rail, and native form controls.
