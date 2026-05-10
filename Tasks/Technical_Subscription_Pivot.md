---
status: active
version: v1.1
last_updated: 2026-05-09
source: Context/implementation_plan_.md
owner: Arsen + Claude Code / Codex implementation chats
---

# Technical Subscription Pivot

This is the technical transition task list for moving BIRGE from taxi-first MVP shape to transportation-as-a-subscription.

Source audit:

```text
Context/implementation_plan_.md
```

## Current Status

Claude Code audit produced:

- 28 code review findings.
- reusable infrastructure list.
- 6-phase technical transition plan.
- Phase 0 partially started in app repo with `TODO(subscription-pivot)` markers.
- Driver subscription prices were fixed in code to 19,000 / 28,000 / 38,000 KZT, but business now adds Peak Starter at 9,900 KZT/month.

Important repo state:

```text
App repo currently has uncommitted Claude Code changes.
Review and commit intentionally before continuing implementation.
```

## Critical Findings

| ID | Area | Problem | Needed Direction |
|---|---|---|---|
| CR-1 | iOS Subscriptions | old `free/lite/standard/pro` plans | `solo_corridor/multi_corridor/flex_pack` |
| CR-2 | Backend Subscriptions | backend hardcodes same wrong plans | rewrite plans to business tiers |
| CR-3 | Driver pricing | old 4,900/8,900/14,900 demo prices; then 19k/28k/38k Phase 0 fix | business now requires Peak Starter 9,900 + 19k/28k/38k |
| CR-4 | Passenger Home | “Вызвать такси” is primary CTA | commute dashboard/setup primary, taxi secondary |
| CR-5 | PassengerSubscription model | only stores `plan_id` | needs plan type, coverage, billing, flex balance |
| CR-6 | Corridor model | no schedule/recurrence fields | add weekdays/time window/recurrence |

## Reusable Infrastructure To Keep

Do not rewrite:

- OTP/JWT auth
- Keychain token refresh
- WebSocket client / actor
- LocationClient and driver GPS sync
- GRDB local persistence
- TCA architecture
- BIRGECore design system
- ride FSM for ride-day execution
- Kaspi payment skeleton
- Docker/Compose/CI
- driver profile/KYC base model

## Phase 0 — Documentation And Code Alignment

Goal:

```text
Make docs, ADRs, and code markers point to the same subscription pivot.
```

Status:

| Task | Status |
|---|---|
| Add `TODO(subscription-pivot)` markers to CR-1..CR-6 files | Done by Claude Code |
| Fix driver tier prices | Needs update again: add Peak Starter 9,900 before final implementation |
| Create ADR-005 | Done in Dev Brain |
| Update iOS agent context with tier IDs and current code status | Done in Dev Brain |
| Update backend agent context with planned models | Done in Dev Brain |
| Review app repo diff and remove accidental Xcode user-state churn before commit if needed | Todo |

Next implementation action:

```text
Review app repo diff, keep intentional source changes, decide whether to commit Phase 0 separately.
```

## Phase 1 — Data Model And API Design

Goal:

```text
Add backend data structures for recurring routes and monthly commute plans with additive migrations.
```

Planned models:

- `RecurringRoute`
- `MonthlyCommutePlan`
- `CommutePlanRoute`

Planned model extensions:

- `Corridor`: weekdays, departure window start/end, `is_recurring`
- `Ride`: optional `corridor_id`, `recurring_route_id`, `commute_plan_id`
- `DriverProfile`: subscription price/status, trial end date, available corridor ids

Planned modules:

- `CommuteRoutes`
- `CommutePlans`

Planned iOS/APIClient DTOs:

- `RecurringRouteDTO`
- `MonthlyCommutePlanDTO`
- `CommutePlanRouteDTO`
- `createRecurringRoute`
- `fetchMyRoutes`
- `deleteRoute`
- `createCommutePlan`
- `fetchMyPlan`
- `cancelPlan`

Blocked details:

- exact KZT prices
- Flex Pack expiry/refund/unused ride rules
- underfilled corridor fallback policy

## Phase 2 — Passenger Subscription Commute Flow

Goal:

```text
Replace taxi-first home/subscription UX with commute-first experience.
```

Planned features:

- `CommuteSetupFeature`
- optional `CommuteDashboardFeature`
- updated `HomeFeature`
- updated `SubscriptionsFeature`
- updated `OnboardingFeature`
- new passenger navigation paths

Rules:

- show all three MVP tiers
- `multi_corridor` is recommended/default
- on-demand ride remains accessible but secondary
- fallback ride flow remains intact

Business blockers:

- exact copy for “not taxi” positioning
- exact passenger prices
- final driver tier implementation after Peak Starter 9,900 addition
- upgrade/downgrade paths

## Phase 3 — Driver Subscription And Corridor Availability

Goal:

```text
Align BIRGEDrive with subscription economics and corridor work.
```

Immediate pricing note:

- Claude Code corrected driver prices to 19k/28k/38k in `DriverRegistrationFeature`.
- Business now adds Peak Starter at 9,900 KZT/month.
- Do not treat the existing app repo driver tier implementation as final until Peak Starter is reflected or intentionally deferred.

Planned:

- occupancy-based earnings table
- subscription tier + billing date on dashboard
- corridor availability selection
- backend driver subscription fields
- driver corridor availability endpoint

Business blockers:

- driver guarantee policy
- free trial exact terms
- real driver willingness validation

## Phase 4 — Pricing, Payments, Kaspi

Goal:

```text
Connect passenger and driver subscriptions to Kaspi/payment lifecycle.
```

Planned:

- Kaspi checkout accepts plan type, price, billing period
- payment event links to commute plan
- webhook activates plan
- cancellation/renewal lifecycle
- Flex Pack balance tracking
- driver subscription payment flow

Blocked:

- legal entity / Kaspi merchant contract
- exact prices
- Flex Pack rules

## Phase 5 — Fallback / On-Demand Integration

Goal:

```text
Keep on-demand ride as fallback without letting it dominate the product.
```

Planned:

- demote ride request navigation
- create ride links to corridor/route/plan
- underfilled corridor conversion into on-demand fallback

Blocked:

- underfilled corridor policy
- service owner for fallback: corridors vs rides

## Phase 6 — Tests And Cleanup

Needed tests:

- `CommuteSetupFeatureTests`
- `CommuteDashboardFeatureTests`
- rewritten `SubscriptionsFeatureTests`
- `HomeFeatureTests`
- `DriverRegistrationFeatureTests`
- backend recurring route CRUD tests
- backend commute plan lifecycle tests
- Flex Pack balance tests
- corridor fallback tests

Cleanup:

- remove stale `free/lite/standard/pro`
- remove stale static subscription lists after API-driven plans
- remove `TODO(subscription-pivot)` markers only after each item is actually solved

## Immediate Next Order

1. Review and commit/clean Claude Code Phase 0 app repo changes.
2. Finish Phase 0 docs: iOS/Backend context updates.
3. Implement ADR-backed backend model design as additive migrations.
4. Only then start passenger `CommuteSetupFeature`.
