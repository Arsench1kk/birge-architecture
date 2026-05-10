# BIRGE Technical Transition Plan
## Taxi-First MVP → Transportation-as-a-Subscription Platform

> **Read-only analysis completed 2026-05-07.** No code changes made. This document covers code review findings and a phased transition plan.

---

## Part 1 — Code Review Findings

### 🔴 Critical (blocks new product direction)

| # | File | Issue | Conflict | Fix |
|---|---|---|---|---|
| CR-1 | `SubscriptionsFeature.swift` L223-268 | Plans are `free/lite/standard/pro` with per-ride pricing (`650₸/850₸/1200₸ per trip`) | Must be `solo_corridor / multi_corridor / flex_pack` with monthly pricing | Replace `SubscriptionPlan.all` with new 3-tier structure |
| CR-2 | `SubscriptionsService.swift` L58-116 | Backend hardcodes same wrong plans | Backend and iOS are synchronized on wrong tiers | Rewrite `SubscriptionsService.plans` to match business tiers |
| CR-3 | `DriverRegistrationFeature.swift` L231-236 | `DriverTier.price` shows `4,900₸/8,900₸/14,900₸` | Business truth is `19,000/28,000/38,000 KZT` | Update `DriverTier.price` values |
| CR-4 | `HomeView.swift` L242-269 | "Вызвать такси" card is prominent CTA on home | Home should center on monthly commute plan, not taxi call | Demote to secondary; add commute dashboard |
| CR-5 | `PassengerSubscription.swift` (backend) | Only stores `plan_id` string — no route coverage, billing period, flex balance | Cannot represent Solo/Multi/Flex Pack semantics | Extend model with plan_type, covered_route_ids, billing fields |
| CR-6 | `Corridor.swift` (backend) | No schedule fields (weekdays, time window, recurrence) — treats corridors as static listings | Subscription commute needs recurring corridor schedules | Add schedule/recurrence columns |

### 🟡 High (deepens taxi-first bias if left unchanged)

| # | File | Issue | Conflict | Fix |
|---|---|---|---|---|
| CR-7 | `RideRequestFeature.swift` | Entire feature is one-off taxi dispatch with `standard/corridor/comfort` tiers | Core flow should be recurring route declaration | Keep as fallback; add new `CommuteSetupFeature` |
| CR-8 | `Ride.swift` (backend) | Model is purely one-off: origin→dest coords, single passenger, no corridor/route link | No way to track recurring commute rides | Add optional `corridor_id`, `recurring_route_id` FKs |
| CR-9 | `HomeFeature.swift` L8-13 | State starts with mock corridors and `aiMatchCount`, no commute plan state | Home should show today's commute schedule | Add `commuteRoutes`, `todaySchedule`, `activeSubscription` to state |
| CR-10 | `OnboardingFeature.swift` | Collects single origin/dest + times but doesn't persist routes or support multiple routes properly | Must support N independent recurring routes with backend persistence | Extend to multi-route collection → POST to backend |
| CR-11 | `CorridorBooking.swift` (backend) | One-off booking, no monthly/subscription linkage | Bookings should be tied to subscription plan + billing period | Add `subscription_id` FK, `billing_period` |
| CR-12 | `HomeView.swift` L66 | `searchBarTapped` and `callTaxiTapped` both → `openRideRequest` | Both CTAs lead to taxi-first flow | `searchBarTapped` should open commute setup or today's plan |
| CR-13 | `DriverRegistrationFeature.swift` L247-256 | `DriverTier.features` mentions "До 30 коридоров", "Гарантия дохода 90 дней" — unconfirmed | 90-day guarantee is not finalized business truth | Mark as placeholder; guard behind feature flag |
| CR-14 | `SubscriptionsView.swift` L300-303 | Comparison table uses `free/standard/pro` columns with wrong prices | Must show Solo Corridor / Multi Corridor / Flex Pack | Rebuild comparison table |

### 🟢 Medium (cleanup needed but not blocking)

| # | File | Issue | Conflict | Fix |
|---|---|---|---|---|
| CR-15 | `RideRequestFeature.swift` L13-16 | Hardcoded fares `1850/890/2400` | Fares should come from backend or subscription context | Move to API-driven pricing |
| CR-16 | `RideRequestFeature.swift` L68-69 | Hardcoded coordinates for origin | Should use user's declared pickup hub | Use location client or route state |
| CR-17 | `CorridorsService.swift` L102-167 | Default seed corridors are static, no schedule | Should seed with schedule/recurrence for MVP corridor | Add schedule fields to seeds |
| CR-18 | `DriverAppFeature.swift` L107 | `earnings: DriverEarnings = .mock` with `12500/8/67000` | Should reflect subscription-based economics | Replace mocks with subscription-aware earnings |
| CR-19 | `Corridor.swift` L44 | `price_tenge` is per-ride pricing on corridor | In subscription model, corridor price is part of monthly plan | Rename to `estimated_per_ride_value` or derive from plan |
| CR-20 | `PassengerAppFeature.swift` L29 | `rideRequest` is first path destination | Should be `commuteSetup` or `routeDeclaration` as primary | Add new path cases |
| CR-21 | `DriverRegistrationView.swift` L278 | Tier step info card says "зарабатывают больше за счёт подписки вместо комиссии" | Correct direction, but needs earnings scenarios at 1-4 passengers | Add occupancy-based earnings table |
| CR-22 | `APIClient.swift` | No endpoints for recurring routes, monthly plans, or commute schedules | Backend API surface is taxi-first | Add new endpoint methods |

### ℹ️ Low (cosmetic / future)

| # | File | Issue | Fix |
|---|---|---|---|
| CR-23 | `HomeView.swift` L62 | "Куда едем?" search placeholder | Change to "Мой маршрут" or commute-oriented copy |
| CR-24 | `HomeView.swift` L78 | "AI нашёл N коридора рядом" | Change to "AI спланировал N маршрутов на месяц" |
| CR-25 | `SubscriptionsView.swift` L346-348 | "/ поездка" price suffix | Change to "/ месяц" for Solo/Multi tiers |
| CR-26 | `HomeView.swift` L277 | Tab bar "Поездки" label | Consider "Маршруты" or "Коммьют" |
| CR-27 | `DriverRegistrationView.swift` L301 | "Для демо можно продолжить без всех документов" | Remove before production |
| CR-28 | `EarningsView.swift` | Shows per-ride earnings model | Add subscription billing view alongside earnings |

---

## Part 2 — Reusable Infrastructure (KEEP)

These modules are stack-correct and product-agnostic — **do not rewrite**:

| Module | Why it's safe |
|---|---|
| **Auth (OTP + JWT + Keychain + TokenRefresh)** | Authentication is product-agnostic |
| **WebSocketClient + LiveWebSocketActor** | Reusable for ride-day real-time updates |
| **LocationClient + LocationSyncService** | Driver GPS tracking works for corridors too |
| **GRDB local cache** | Offline-first GPS still needed |
| **TCA architecture + navigation stack** | Add new reducers, don't replace pattern |
| **BIRGECore DesignSystem** | Liquid Glass, BIRGEColors, BIRGEFonts, buttons — all reusable |
| **Ride FSM (backend)** | States work for corridor rides too |
| **Kaspi payment skeleton** | Payment flow is plan-agnostic |
| **Docker/Compose/CI** | Infrastructure unchanged |
| **Driver profile + KYC model** | Registration flow just needs price update |

---

## Part 3 — Technical Transition Plan

### Phase 0 — Documentation & Code Alignment

**Goal:** Eliminate doc↔code contradictions so every developer/AI sees one truth.

**Affected files:**
- `Business_Department/Subscription_Strategy.md`
- `Business_Department/Driver_Economics.md`
- `Context/iOS_Agent_Context.md`
- `Context/Backend_Agent_Context.md`

**Steps:**
1. Add `[STALE]` markers to subscription plan IDs in iOS/backend code comments referencing `free/lite/standard/pro`
2. Add `// TODO(subscription-pivot)` markers to all CR-1 through CR-6 files
3. Update `iOS_Agent_Context.md` with new subscription tier IDs
4. Update `Backend_Agent_Context.md` with planned new models
5. Create `ADR-005_Subscription_Pivot.md` documenting the transition decision

**Tests:** None — documentation only.

**Risks:** Low. Pure docs.

**Business dependencies:** None.

**Commits:**
- `docs: mark stale subscription tier references`
- `docs: add ADR-005 subscription pivot decision`

---

### Phase 1 — Data Model & API Design

**Goal:** Design and migrate backend models for recurring routes, monthly plans, and subscription tiers without breaking existing flows.

**Affected files:**
- `birge-vapor/Sources/App/Models/` — new models
- `birge-vapor/Sources/App/Migrations/` — new migrations
- `birge-vapor/Sources/App/Modules/` — new module stubs
- `BIRGECore/Sources/BIRGECore/Network/APIClient.swift` — new DTOs

**Steps:**

1. **New model: `RecurringRoute`**
   ```
   recurring_routes:
     id UUID PK
     passenger_id UUID FK → users
     origin_name String
     origin_lat Double
     origin_lng Double
     destination_name String
     destination_lat Double
     destination_lng Double
     weekdays [String]  // ["mon","tue","wed","thu","fri"]
     departure_window String  // "07:15-08:30"
     corridor_id UUID? FK → corridors  // matched after route analysis
     is_active Bool DEFAULT true
     created_at, updated_at
   ```

2. **New model: `MonthlyCommutePlan`**
   ```
   monthly_commute_plans:
     id UUID PK
     passenger_id UUID FK → users
     plan_type String  // "solo_corridor" | "multi_corridor" | "flex_pack"
     billing_period_start Date
     billing_period_end Date
     status String  // "active" | "cancelled" | "expired" | "pending_payment"
     price_tenge Int
     flex_rides_total Int?  // only for flex_pack
     flex_rides_used Int?   // only for flex_pack
     kaspi_payment_id String?
     created_at, updated_at
   ```

3. **New join: `CommutePlanRoute`**
   ```
   commute_plan_routes:
     id UUID PK
     plan_id UUID FK → monthly_commute_plans
     recurring_route_id UUID FK → recurring_routes
   ```

4. **Extend `Corridor`** — add: `weekdays [String]`, `departure_window_start String`, `departure_window_end String`, `is_recurring Bool DEFAULT false`

5. **Extend `Ride`** — add: `corridor_id UUID? FK`, `recurring_route_id UUID? FK`, `commute_plan_id UUID? FK`

6. **Extend `DriverProfile`** — add: `subscription_price_tenge Int?`, `subscription_status String?`, `trial_end_date Date?`, `available_corridor_ids [UUID]?`

7. **New Vapor module: `CommuteRoutes/`** — controller + service + DTOs for CRUD on recurring routes

8. **New Vapor module: `CommutePlans/`** — controller + service + DTOs for plan creation, activation, status

9. **New iOS DTOs in APIClient.swift:**
   - `RecurringRouteDTO`, `MonthlyCommutePlanDTO`, `CommutePlanRouteDTO`
   - API methods: `createRecurringRoute`, `fetchMyRoutes`, `deleteRoute`, `createCommutePlan`, `fetchMyPlan`, `cancelPlan`

**Tests:**
- Migration up/down tests
- `RecurringRoute` CRUD unit tests
- `MonthlyCommutePlan` lifecycle tests
- API endpoint integration tests

**Risks:**
- Migrations must be additive (new columns/tables only) to avoid breaking existing flows
- `flex_rides_total/used` logic needs business rules for expiry and refunds

**Business dependencies:**
- ⚠️ **BQ-007:** Exact KZT prices for Solo/Multi/Flex — blocked
- ⚠️ **BQ-015:** Flex Pack balance/expiry/refund rules — blocked
- ⚠️ **BQ-008:** Underfilled corridor fallback — blocked

**Commits:**
- `feat(backend): add RecurringRoute model and migration`
- `feat(backend): add MonthlyCommutePlan model and migration`
- `feat(backend): extend Corridor with schedule fields`
- `feat(backend): extend Ride with corridor/route FKs`
- `feat(backend): add CommuteRoutes module`
- `feat(backend): add CommutePlans module`
- `feat(core): add recurring route and commute plan DTOs`

---

### Phase 2 — Passenger Subscription Commute Flow

**Goal:** Replace taxi-first home/subscription screens with commute-first experience.

**Affected files:**
- `Features/Home/HomeFeature.swift` + `HomeView.swift`
- `Features/Subscriptions/SubscriptionsFeature.swift` + `SubscriptionsView.swift`
- `Features/Auth/OnboardingFeature.swift` + `OnboardingView.swift`
- `App/PassengerAppFeature.swift`
- New: `Features/CommuteSetup/`
- New: `Features/CommuteDashboard/`

**Steps:**

1. **New `CommuteSetupFeature`** — multi-route declaration flow:
   - State: `routes: [RouteDeclaration]`, `currentRouteIndex`, `selectedPlanType`
   - Each route: origin hub, destination hub, weekdays, time window
   - "Add another route" action (resets form for next route)
   - Finish → POST routes to backend → show plan selection
   - Plan selection: Solo Corridor / Multi Corridor / Flex Pack cards
   - Multi Corridor = recommended badge

2. **Rework `OnboardingFeature`:**
   - Keep 3 intro slides (reword: "monthly commute plan" not "taxi")
   - Route setup pages → delegate to `CommuteSetupFeature` or inline
   - `addAnotherRouteTapped` saves current route to array, resets fields
   - Final page: AI summary of matched corridors + plan recommendation

3. **Rework `HomeFeature.State`:**
   - Add: `activeSubscription: MonthlyCommutePlanDTO?`
   - Add: `commuteRoutes: [RecurringRouteDTO]`
   - Add: `todaySchedule: [TodayCommuteRide]` (pre-planned rides for today)
   - Keep: `corridors` for discovery
   - `onAppear` loads subscription status + today schedule + corridors

4. **Rework `HomeView`:**
   - If subscribed: show today's commute card (next ride time, driver, corridor)
   - If not subscribed: show commute setup CTA ("Спланируй маршруты на месяц")
   - Demote "Вызвать такси" to small secondary button
   - Keep corridor discovery scroll below

5. **Rewrite `SubscriptionsFeature`:**
   - Replace plan IDs: `solo_corridor`, `multi_corridor`, `flex_pack`
   - Remove `free/lite/standard/pro`
   - Add route coverage display per plan
   - Add monthly savings estimate
   - Multi Corridor = `isPopular: true`, `badge: "Рекомендуем"`
   - Flex Pack shows pack size + per-ride deduction

6. **Update `PassengerAppFeature.Path`:**
   - Add `.commuteSetup(CommuteSetupFeature)`
   - Add `.commuteDashboard(CommuteDashboardFeature)` if needed
   - Home delegate: `.openCommuteSetup` (primary CTA)

**Tests:**
- `CommuteSetupFeatureTests`: add route, add second route, select plan, finish
- `SubscriptionsFeatureTests`: update for new tier IDs and monthly pricing
- `HomeFeatureTests`: subscribed vs unsubscribed home state
- `OnboardingFeatureTests`: update for multi-route flow
- `PassengerAppFeatureTests`: new navigation paths

**Risks:**
- Large UI change — split into sub-PRs (data layer first, then views)
- Onboarding rewrite may break existing test coverage
- Must keep fallback ride request accessible

**Business dependencies:**
- ⚠️ **BQ-007:** KZT prices for each tier
- ⚠️ **BQ-009:** Copy language for "not taxi" positioning
- ⚠️ **BQ-013:** Whether on-demand appears on home at all
- ⚠️ **BQ-014:** Upgrade/downgrade paths between tiers

**Commits:**
- `feat(passenger): add CommuteSetupFeature with multi-route declaration`
- `feat(passenger): rework SubscriptionsFeature for Solo/Multi/Flex tiers`
- `feat(passenger): rework HomeFeature for commute dashboard`
- `refactor(passenger): demote taxi CTA on HomeView`
- `feat(passenger): update onboarding for monthly route setup`
- `feat(passenger): add commute navigation paths`

---

### Phase 3 — Driver Subscription & Corridor Availability

**Goal:** Align driver registration pricing and add corridor availability management.

**Affected files:**
- `BIRGEDrive/Features/DriverRegistrationFeature.swift`
- `BIRGEDrive/Features/DriverRegistrationView.swift`
- `BIRGEDrive/App/DriverAppFeature.swift`
- `BIRGEDrive/App/DriverDashboardSurfaces.swift`
- `birge-vapor/Sources/App/Models/Driver.swift`
- `birge-vapor/Sources/App/Modules/Drivers/`

**Steps:**

1. **Fix driver tier pricing (CR-3):**
   ```swift
   enum DriverTier {
     case starter  // "19 000₸ / мес"
     case professional  // "28 000₸ / мес"
     case premium  // "38 000₸ / мес"
   }
   ```

2. **Add occupancy-based earnings display** to tier step:
   - Table: 1/2/3/4 passengers → revenue per run
   - Yandex Go comparison line
   - "При 3+ пассажирах вы зарабатываете больше, чем на Yandex Go"

3. **Add driver corridor availability** to DriverAppFeature:
   - New state: `availableCorridorIDs: Set<UUID>`
   - New UI: corridor selection checklist on dashboard
   - Backend: `PUT /api/v1/drivers/corridors` to save availability

4. **Update driver dashboard** to show:
   - Subscription tier + billing date
   - Assigned corridor runs with passenger counts
   - Projected daily/weekly/monthly earnings
   - "Zero commission" badge

5. **Backend: extend DriverProfile** for subscription billing:
   - `subscription_price_tenge`, `subscription_status`, `trial_end_date`
   - `PUT /drivers/me/corridors` endpoint

6. **Add launch promo logic:**
   - If `trial_end_date` is set and not passed → show "Бесплатный период"
   - Professional at Starter price for first 6 months

**Tests:**
- `DriverRegistrationFeatureTests`: verify new prices render correctly
- Driver corridor availability toggle tests
- Backend driver subscription status tests

**Risks:**
- Driver tier features ("Гарантия дохода 90 дней") are not confirmed — guard with placeholder

**Business dependencies:**
- ⚠️ **BQ-001:** Will drivers accept subscription?
- ⚠️ Driver guarantee policy — not finalized
- ⚠️ Free trial exact terms — not finalized

**Commits:**
- `fix(driver): update tier pricing to 19k/28k/38k`
- `feat(driver): add occupancy earnings table to registration`
- `feat(driver): add corridor availability management`
- `feat(driver): show subscription billing on dashboard`
- `feat(backend): extend driver profile for subscription billing`

---

### Phase 4 — Pricing, Payments & Kaspi Alignment

**Goal:** Connect subscription activation to Kaspi payment for monthly commute plans.

**Affected files:**
- `birge-vapor/Sources/App/Modules/Payments/`
- `birge-vapor/Sources/App/Modules/Subscriptions/` → merge into `CommutePlans/`
- `BIRGECore/Network/APIClient.swift` — payment DTOs
- `Features/Subscriptions/SubscriptionsFeature.swift`

**Steps:**

1. **Extend Kaspi checkout** for monthly plan types:
   - `createKaspiCheckout` accepts `plan_type` + `price_tenge` + `billing_period`
   - Payment event links to `monthly_commute_plan_id`

2. **Add subscription lifecycle:**
   - Kaspi webhook confirms payment → activate plan → set billing period
   - Auto-renewal: create next-month plan on billing date
   - Cancellation: mark plan `cancelled`, active until period end

3. **Flex Pack balance management:**
   - On ride completion → decrement `flex_rides_used`
   - Show remaining rides in UI
   - ⚠️ Expiry/refund rules — **blocked by BQ-015**

4. **Driver subscription payment:**
   - Same Kaspi flow for driver monthly subscription
   - Payment type: `driver_subscription`
   - Separate from passenger plan

5. **Update `PaymentEvent` model:**
   - Add `plan_type`, `plan_id`, `billing_period` fields

**Tests:**
- Kaspi webhook → plan activation integration test
- Flex Pack ride deduction test
- Driver subscription payment test
- Plan expiry/renewal tests

**Risks:**
- ⚠️ **BQ-005:** No legal entity (ИП/ТОО) → Kaspi merchant contract impossible
- Real Kaspi integration requires merchant credentials not yet available
- Subscription auto-renewal logic is complex

**Business dependencies:**
- ⚠️ **BQ-005:** Legal entity for Kaspi
- ⚠️ **BQ-007:** Exact prices
- ⚠️ **BQ-015:** Flex Pack rules

**Commits:**
- `feat(payments): extend Kaspi checkout for monthly plans`
- `feat(backend): add subscription lifecycle (activate/cancel/renew)`
- `feat(backend): add Flex Pack ride balance tracking`
- `feat(payments): add driver subscription payment`

---

### Phase 5 — Fallback / On-Demand Integration

**Goal:** Keep on-demand ride as Tier 1 fallback without dominating the UX.

**Affected files:**
- `Features/RideRequest/` — keep, demote in navigation
- `Features/Searching/` — keep as-is
- `Features/Ride/` — keep as-is
- `Features/OfferFound/` — keep as-is
- `HomeView.swift` — demote taxi CTA

**Steps:**

1. **Preserve existing ride flow** entirely — no deletions
2. **Add corridor fallback trigger:**
   - If corridor has <2 passengers 60 min before departure → offer on-demand dispatch
   - Backend logic: `CorridorFallbackService` checks occupancy, converts to individual rides
   - ⚠️ Fallback owner (corridors vs rides) — **blocked by OQ-003**

3. **Navigation change:**
   - Ride request reachable from: small "Такси" button on home, profile menu, underfilled corridor fallback
   - Not reachable from: primary home CTA (which becomes commute setup)

4. **Ride model linkage:**
   - When corridor ride fires, create `Ride` with `corridor_id` + `recurring_route_id` + `commute_plan_id`
   - When fallback fires, create `Ride` with `tier: "on_demand"`, no corridor link

**Tests:**
- Fallback trigger test (corridor < 2 pax → on-demand offer)
- Ride creation with corridor linkage test
- Navigation: taxi CTA still works from secondary position

**Risks:**
- Fallback policy not finalized — implement behind feature flag
- Must not break existing ride WebSocket/FSM

**Business dependencies:**
- ⚠️ **BQ-008:** What happens with 1 passenger?
- ⚠️ **OQ-003:** Fallback owner service

**Commits:**
- `refactor(passenger): demote ride request to secondary navigation`
- `feat(backend): add corridor fallback occupancy check`
- `feat(backend): link rides to corridors and commute plans`

---

### Phase 6 — Tests & Migration Cleanup

**Goal:** Full test coverage for new flows, clean migration path, remove stale code.

**Affected files:**
- `BIRGEPassengerTests/` — new + updated tests
- `birge-vapor/Tests/AppTests/` — new tests
- All migration files

**Steps:**

1. **New iOS tests:**
   - `CommuteSetupFeatureTests` — multi-route CRUD, plan selection
   - `CommuteDashboardFeatureTests` — today schedule loading
   - `SubscriptionsFeatureTests` — rewrite for new tiers
   - `HomeFeatureTests` — subscribed vs unsubscribed states
   - `DriverRegistrationFeatureTests` — new pricing verification

2. **New backend tests:**
   - `RecurringRoute` CRUD
   - `MonthlyCommutePlan` lifecycle (create → activate → use → renew → cancel)
   - Corridor schedule matching
   - Flex Pack balance deduction
   - Driver subscription status
   - Corridor fallback trigger

3. **Migration cleanup:**
   - Ensure all migrations are additive and reversible
   - Test migration up/down on clean database
   - Seed data updated for new corridor format

4. **Remove stale code:**
   - Remove `SubscriptionPlan.all` static list (replaced by API-driven)
   - Remove demo driver prices from `DriverTier.price` once real prices are in
   - Remove `// TODO(subscription-pivot)` markers as each is resolved

5. **Update CI:**
   - Add new test targets to `BIRGEPassenger.xctestplan`
   - Verify `swift test` covers new backend modules

**Tests:** This phase IS the test phase.

**Risks:**
- Test coverage for Flex Pack depends on finalized business rules
- CI may need simulator update for new test targets

**Business dependencies:** None (tests can use placeholder prices).

**Commits:**
- `test(passenger): add CommuteSetupFeature tests`
- `test(passenger): rewrite SubscriptionsFeature tests for new tiers`
- `test(backend): add recurring route and commute plan tests`
- `test(backend): add corridor fallback tests`
- `chore: remove stale subscription plan static data`
- `chore: update CI test plan`

---

## Summary — What Can Start Now vs What's Blocked

### ✅ Can start immediately (no business blockers):

| Task | Phase |
|---|---|
| Add `// TODO(subscription-pivot)` markers | Phase 0 |
| Write ADR-005 | Phase 0 |
| Fix driver prices to 19k/28k/38k | Phase 3 |
| Create `RecurringRoute` model + migration | Phase 1 |
| Create `MonthlyCommutePlan` model + migration | Phase 1 |
| Extend `Corridor` with schedule fields | Phase 1 |
| Extend `Ride` with corridor/route FKs | Phase 1 |
| Build `CommuteRoutes` backend module | Phase 1 |
| Add new DTOs to `APIClient.swift` | Phase 1 |
| Rename subscription tier IDs to `solo_corridor/multi_corridor/flex_pack` | Phase 2 |
| Demote "Вызвать такси" in HomeView | Phase 2 |

### ⚠️ Blocked by business decisions:

| Task | Blocker |
|---|---|
| Set KZT prices on subscription tiers | BQ-007 (Session 8) |
| Implement Flex Pack balance/expiry logic | BQ-015 |
| Implement corridor fallback policy | BQ-008, OQ-003 |
| Set driver guarantee features | Driver guarantee not finalized |
| Real Kaspi payments | BQ-005 (no legal entity) |
| Copy/UX text for "not taxi" positioning | BQ-009 |
| Subscription upgrade/downgrade paths | BQ-014 |

---

## Recommended Execution Order

```
Week 1:  Phase 0 (docs) + Phase 1 (models/migrations/DTOs)
Week 2:  Phase 3 partial (driver price fix) + Phase 2 partial (subscription tier rename)
Week 3:  Phase 2 (CommuteSetupFeature + Home rework)
Week 4:  Phase 3 (driver corridor availability)
Week 5:  Phase 4 (payments alignment) — needs BQ-007 resolved
Week 6:  Phase 5 (fallback integration) — needs BQ-008 resolved
Week 7:  Phase 6 (tests + cleanup)
```

Each week produces 2-4 safe, self-contained commits. No big-bang rewrite.

# BIRGE Subscription Pivot — Task Tracker

## Phase 0 — Documentation & Code Alignment
- [x] Add `// TODO(subscription-pivot)` markers to all CR-1–CR-6 files
- [ ] Create `ADR-005_Subscription_Pivot.md`
- [ ] Update `iOS_Agent_Context.md` with new tier IDs
- [ ] Update `Backend_Agent_Context.md` with planned new models

## Phase 3 — Immediate Safe Fix
- [x] Fix driver tier pricing: 4,900/8,900/14,900 → 19,000/28,000/38,000

## Build Verification
- [x] BIRGEPassenger — BUILD SUCCEEDED ✅
- [x] BIRGEDrive — BUILD SUCCEEDED ✅

## Phase 1 — Data Model & API Design (next)
- [ ] New model: `RecurringRoute` + migration
- [ ] New model: `MonthlyCommutePlan` + migration
- [ ] New join: `CommutePlanRoute` + migration
- [ ] Extend `Corridor` with schedule fields
- [ ] Extend `Ride` with corridor/route FKs
- [ ] Extend `DriverProfile` with subscription billing fields
- [ ] New Vapor module: `CommuteRoutes/`
- [ ] New Vapor module: `CommutePlans/`
- [ ] New iOS DTOs in APIClient.swift

## Phase 2 — Passenger Subscription Commute Flow (later)
- [ ] New `CommuteSetupFeature`
- [ ] Rework `SubscriptionsFeature` for Solo/Multi/Flex
- [ ] Rework `HomeFeature` for commute dashboard
- [ ] Rework `OnboardingFeature` for multi-route
- [ ] Update `PassengerAppFeature.Path`

## Phase 3 continued — Driver Corridor Availability (later)
- [ ] Add occupancy earnings table to driver registration
- [ ] Add driver corridor availability management
- [ ] Show subscription billing on driver dashboard

## Phase 4–6 — deferred until business decisions resolve

# BIRGE Subscription Pivot — Walkthrough

## Session: 2026-05-07

### What was done

#### 1. Full Code Review (read-only)
Analyzed all 12 Dev Brain documents + 20+ source files across BIRGEPassenger, BIRGEDrive, BIRGECore, and birge-vapor. Identified **28 code review findings** (6 critical, 8 high, 8 medium, 6 low).

#### 2. Phase 0 — TODO Markers (6 files)
Added `// TODO(subscription-pivot)` markers to every critical file that needs transition:

| File | Marker |
|---|---|
| [SubscriptionsFeature.swift](file:///Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger/BIRGEPassenger/Features/Subscriptions/SubscriptionsFeature.swift) | Replace plan IDs + per-ride pricing |
| [HomeFeature.swift](file:///Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger/BIRGEPassenger/Features/Home/HomeFeature.swift) | Add commute dashboard state |
| [HomeView.swift](file:///Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger/BIRGEPassenger/Features/Home/HomeView.swift) | Demote taxi CTA |
| [PassengerSubscription.swift](file:///Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger/birge-vapor/Sources/App/Models/PassengerSubscription.swift) | Extend model for plan_type, billing, flex balance |
| [Corridor.swift](file:///Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger/birge-vapor/Sources/App/Models/Corridor.swift) | Add schedule/recurrence fields |
| [SubscriptionsService.swift](file:///Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger/birge-vapor/Sources/App/Modules/Subscriptions/SubscriptionsService.swift) | Rewrite plans array |

#### 3. Phase 3 — Driver Pricing Fix (CR-3)

```diff:DriverRegistrationFeature.swift
import ComposableArchitecture
import BIRGECore
import Foundation
import SwiftUI

@Reducer
struct DriverRegistrationFeature {
    @ObservableState
    struct State: Equatable {
        var step: Step = .personal
        var firstName = ""
        var lastName = ""
        var birthDate = ""
        var iin = ""
        var carMake = ""
        var carModel = ""
        var carYear = ""
        var plateNumber = ""
        var selectedColor: CarColor = .white
        var seats = 4
        var uploadedDocuments: Set<DocumentKind> = [.driverLicenseFront]
        var selectedTier: DriverTier = .professional
        var isSaving = false
        var errorMessage: String?

        var progress: Double {
            Double(step.rawValue + 1) / Double(Step.allCases.count)
        }

        var canGoBack: Bool {
            step != .personal
        }

        enum Step: Int, CaseIterable, Equatable, Sendable {
            case personal
            case vehicle
            case documents
            case tier
        }
    }

    enum Action: Equatable, Sendable {
        case firstNameChanged(String)
        case lastNameChanged(String)
        case birthDateChanged(String)
        case iinChanged(String)
        case carMakeChanged(String)
        case carModelChanged(String)
        case carYearChanged(String)
        case plateNumberChanged(String)
        case colorSelected(CarColor)
        case seatsSelected(Int)
        case documentTapped(DocumentKind)
        case tierSelected(DriverTier)
        case backTapped
        case nextTapped
        case saveResponse(Result<DriverProfileDTO, DriverRegistrationError>)
        case delegate(Delegate)

        enum Delegate: Equatable, Sendable {
            case completed(DriverProfileDTO)
        }
    }

    @Dependency(\.apiClient) var apiClient

    var body: some Reducer<State, Action> {
        Reduce { state, action in
            switch action {
            case .firstNameChanged(let value):
                state.firstName = value
                return .none
            case .lastNameChanged(let value):
                state.lastName = value
                return .none
            case .birthDateChanged(let value):
                state.birthDate = value
                return .none
            case .iinChanged(let value):
                state.iin = String(value.filter(\.isNumber).prefix(12))
                return .none
            case .carMakeChanged(let value):
                state.carMake = value
                return .none
            case .carModelChanged(let value):
                state.carModel = value
                return .none
            case .carYearChanged(let value):
                state.carYear = String(value.filter(\.isNumber).prefix(4))
                return .none
            case .plateNumberChanged(let value):
                state.plateNumber = value.uppercased()
                return .none
            case .colorSelected(let color):
                state.selectedColor = color
                return .none
            case .seatsSelected(let seats):
                state.seats = seats
                return .none
            case .documentTapped(let document):
                if state.uploadedDocuments.contains(document) {
                    state.uploadedDocuments.remove(document)
                } else {
                    state.uploadedDocuments.insert(document)
                }
                return .none
            case .tierSelected(let tier):
                state.selectedTier = tier
                return .none
            case .backTapped:
                guard state.step.rawValue > 0,
                      let previous = State.Step(rawValue: state.step.rawValue - 1) else {
                    return .none
                }
                state.step = previous
                state.errorMessage = nil
                return .none
            case .nextTapped:
                state.errorMessage = nil
                if let next = State.Step(rawValue: state.step.rawValue + 1) {
                    state.step = next
                    return .none
                }
                state.isSaving = true
                let request = state.driverProfileRequest
                return .run { send in
                    do {
                        await send(.saveResponse(.success(try await apiClient.updateDriverProfile(request))))
                    } catch {
                        await send(.saveResponse(.failure(DriverRegistrationError(error))))
                    }
                }
            case .saveResponse(.success(let profile)):
                state.isSaving = false
                return .send(.delegate(.completed(profile)))
            case .saveResponse(.failure(let error)):
                state.isSaving = false
                state.errorMessage = error.message
                return .none
            case .delegate:
                return .none
            }
        }
    }
}

struct DriverRegistrationError: Error, Equatable, Sendable {
    let message: String

    init(_ error: any Error) {
        if let localizedError = error as? LocalizedError,
           let description = localizedError.errorDescription,
           !description.isEmpty {
            self.message = description
        } else {
            self.message = "Не удалось сохранить профиль водителя"
        }
    }
}

private extension DriverRegistrationFeature.State {
    var driverProfileRequest: UpdateDriverProfileRequest {
        UpdateDriverProfileRequest(
            firstName: firstName,
            lastName: lastName,
            birthDate: birthDate,
            iin: iin,
            vehicleMake: carMake,
            vehicleModel: carModel,
            vehicleYear: carYear,
            licensePlate: plateNumber,
            vehicleColor: selectedColor.rawValue,
            seats: seats,
            uploadedDocuments: uploadedDocuments
                .sorted { $0.rawValue < $1.rawValue }
                .map(\.apiValue),
            subscriptionTier: selectedTier.apiValue
        )
    }
}

enum CarColor: String, CaseIterable, Equatable, Sendable {
    case white = "Белый"
    case black = "Чёрный"
    case red = "Красный"
    case blue = "Синий"
    case green = "Зелёный"
    case gray = "Серый"

    var color: Color {
        switch self {
        case .white: return .white
        case .black: return .black
        case .red: return .red
        case .blue: return .blue
        case .green: return .green
        case .gray: return .gray
        }
    }
}

enum DocumentKind: String, CaseIterable, Equatable, Hashable, Sendable {
    case driverLicenseFront = "Права: лицевая"
    case driverLicenseBack = "Права: обратная"
    case vehicleRegistration = "Техпаспорт"
    case identityCard = "Удостоверение"

    var symbol: String {
        switch self {
        case .driverLicenseFront, .driverLicenseBack: return "doc.text.fill"
        case .vehicleRegistration: return "car.fill"
        case .identityCard: return "person.text.rectangle.fill"
        }
    }

    var apiValue: String {
        switch self {
        case .driverLicenseFront: return "driverLicenseFront"
        case .driverLicenseBack: return "driverLicenseBack"
        case .vehicleRegistration: return "vehicleRegistration"
        case .identityCard: return "identityCard"
        }
    }
}

enum DriverTier: String, CaseIterable, Equatable, Sendable {
    case starter = "Стартер"
    case professional = "Профессионал"
    case premium = "Премиум"

    var price: String {
        switch self {
        case .starter: return "4 900₸ / мес"
        case .professional: return "8 900₸ / мес"
        case .premium: return "14 900₸ / мес"
        }
    }

    var subtitle: String {
        switch self {
        case .starter: return "Для первых коридоров"
        case .professional: return "Лучший баланс для полной смены"
        case .premium: return "Максимальный приоритет"
        }
    }

    var features: [String] {
        switch self {
        case .starter:
            return ["До 30 коридоров", "Стандартная поддержка", "Гарантия дохода 90 дней"]
        case .professional:
            return ["Безлимит коридоров", "Высокий приоритет", "Поддержка 24/7", "Аналитика смен"]
        case .premium:
            return ["Всё из Профессионала", "Персональный менеджер", "Эксклюзивные маршруты"]
        }
    }

    var apiValue: String {
        switch self {
        case .starter: return "starter"
        case .professional: return "professional"
        case .premium: return "premium"
        }
    }
}
===
import ComposableArchitecture
import BIRGECore
import Foundation
import SwiftUI

@Reducer
struct DriverRegistrationFeature {
    @ObservableState
    struct State: Equatable {
        var step: Step = .personal
        var firstName = ""
        var lastName = ""
        var birthDate = ""
        var iin = ""
        var carMake = ""
        var carModel = ""
        var carYear = ""
        var plateNumber = ""
        var selectedColor: CarColor = .white
        var seats = 4
        var uploadedDocuments: Set<DocumentKind> = [.driverLicenseFront]
        var selectedTier: DriverTier = .professional
        var isSaving = false
        var errorMessage: String?

        var progress: Double {
            Double(step.rawValue + 1) / Double(Step.allCases.count)
        }

        var canGoBack: Bool {
            step != .personal
        }

        enum Step: Int, CaseIterable, Equatable, Sendable {
            case personal
            case vehicle
            case documents
            case tier
        }
    }

    enum Action: Equatable, Sendable {
        case firstNameChanged(String)
        case lastNameChanged(String)
        case birthDateChanged(String)
        case iinChanged(String)
        case carMakeChanged(String)
        case carModelChanged(String)
        case carYearChanged(String)
        case plateNumberChanged(String)
        case colorSelected(CarColor)
        case seatsSelected(Int)
        case documentTapped(DocumentKind)
        case tierSelected(DriverTier)
        case backTapped
        case nextTapped
        case saveResponse(Result<DriverProfileDTO, DriverRegistrationError>)
        case delegate(Delegate)

        enum Delegate: Equatable, Sendable {
            case completed(DriverProfileDTO)
        }
    }

    @Dependency(\.apiClient) var apiClient

    var body: some Reducer<State, Action> {
        Reduce { state, action in
            switch action {
            case .firstNameChanged(let value):
                state.firstName = value
                return .none
            case .lastNameChanged(let value):
                state.lastName = value
                return .none
            case .birthDateChanged(let value):
                state.birthDate = value
                return .none
            case .iinChanged(let value):
                state.iin = String(value.filter(\.isNumber).prefix(12))
                return .none
            case .carMakeChanged(let value):
                state.carMake = value
                return .none
            case .carModelChanged(let value):
                state.carModel = value
                return .none
            case .carYearChanged(let value):
                state.carYear = String(value.filter(\.isNumber).prefix(4))
                return .none
            case .plateNumberChanged(let value):
                state.plateNumber = value.uppercased()
                return .none
            case .colorSelected(let color):
                state.selectedColor = color
                return .none
            case .seatsSelected(let seats):
                state.seats = seats
                return .none
            case .documentTapped(let document):
                if state.uploadedDocuments.contains(document) {
                    state.uploadedDocuments.remove(document)
                } else {
                    state.uploadedDocuments.insert(document)
                }
                return .none
            case .tierSelected(let tier):
                state.selectedTier = tier
                return .none
            case .backTapped:
                guard state.step.rawValue > 0,
                      let previous = State.Step(rawValue: state.step.rawValue - 1) else {
                    return .none
                }
                state.step = previous
                state.errorMessage = nil
                return .none
            case .nextTapped:
                state.errorMessage = nil
                if let next = State.Step(rawValue: state.step.rawValue + 1) {
                    state.step = next
                    return .none
                }
                state.isSaving = true
                let request = state.driverProfileRequest
                return .run { send in
                    do {
                        await send(.saveResponse(.success(try await apiClient.updateDriverProfile(request))))
                    } catch {
                        await send(.saveResponse(.failure(DriverRegistrationError(error))))
                    }
                }
            case .saveResponse(.success(let profile)):
                state.isSaving = false
                return .send(.delegate(.completed(profile)))
            case .saveResponse(.failure(let error)):
                state.isSaving = false
                state.errorMessage = error.message
                return .none
            case .delegate:
                return .none
            }
        }
    }
}

struct DriverRegistrationError: Error, Equatable, Sendable {
    let message: String

    init(_ error: any Error) {
        if let localizedError = error as? LocalizedError,
           let description = localizedError.errorDescription,
           !description.isEmpty {
            self.message = description
        } else {
            self.message = "Не удалось сохранить профиль водителя"
        }
    }
}

private extension DriverRegistrationFeature.State {
    var driverProfileRequest: UpdateDriverProfileRequest {
        UpdateDriverProfileRequest(
            firstName: firstName,
            lastName: lastName,
            birthDate: birthDate,
            iin: iin,
            vehicleMake: carMake,
            vehicleModel: carModel,
            vehicleYear: carYear,
            licensePlate: plateNumber,
            vehicleColor: selectedColor.rawValue,
            seats: seats,
            uploadedDocuments: uploadedDocuments
                .sorted { $0.rawValue < $1.rawValue }
                .map(\.apiValue),
            subscriptionTier: selectedTier.apiValue
        )
    }
}

enum CarColor: String, CaseIterable, Equatable, Sendable {
    case white = "Белый"
    case black = "Чёрный"
    case red = "Красный"
    case blue = "Синий"
    case green = "Зелёный"
    case gray = "Серый"

    var color: Color {
        switch self {
        case .white: return .white
        case .black: return .black
        case .red: return .red
        case .blue: return .blue
        case .green: return .green
        case .gray: return .gray
        }
    }
}

enum DocumentKind: String, CaseIterable, Equatable, Hashable, Sendable {
    case driverLicenseFront = "Права: лицевая"
    case driverLicenseBack = "Права: обратная"
    case vehicleRegistration = "Техпаспорт"
    case identityCard = "Удостоверение"

    var symbol: String {
        switch self {
        case .driverLicenseFront, .driverLicenseBack: return "doc.text.fill"
        case .vehicleRegistration: return "car.fill"
        case .identityCard: return "person.text.rectangle.fill"
        }
    }

    var apiValue: String {
        switch self {
        case .driverLicenseFront: return "driverLicenseFront"
        case .driverLicenseBack: return "driverLicenseBack"
        case .vehicleRegistration: return "vehicleRegistration"
        case .identityCard: return "identityCard"
        }
    }
}

enum DriverTier: String, CaseIterable, Equatable, Sendable {
    case starter = "Стартер"
    case professional = "Профессионал"
    case premium = "Премиум"

    var price: String {
        switch self {
        case .starter: return "19 000₸ / мес"
        case .professional: return "28 000₸ / мес"
        case .premium: return "38 000₸ / мес"
        }
    }

    var subtitle: String {
        switch self {
        case .starter: return "Для первых коридоров"
        case .professional: return "Лучший баланс для полной смены"
        case .premium: return "Максимальный приоритет и поддержка"
        }
    }

    var features: [String] {
        switch self {
        case .starter:
            return ["До 30 коридоров", "Стандартная поддержка", "0% комиссии с поездок"]
        case .professional:
            return ["Безлимит коридоров", "Высокий приоритет", "Поддержка 24/7", "Аналитика смен"]
        case .premium:
            return ["Всё из Профессионала", "Персональный менеджер", "Эксклюзивные маршруты"]
        }
    }

    var apiValue: String {
        switch self {
        case .starter: return "starter"
        case .professional: return "professional"
        case .premium: return "premium"
        }
    }
}
```

Key changes in [DriverRegistrationFeature.swift](file:///Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger/BIRGEDrive/Features/DriverRegistrationFeature.swift#L226-L265):
- **Starter:** 4,900₸ → **19,000₸** / мес
- **Профессионал:** 8,900₸ → **28,000₸** / мес  
- **Премиум:** 14,900₸ → **38,000₸** / мес
- Removed unconfirmed "Гарантия дохода 90 дней" → replaced with "0% комиссии с поездок"

### Build Verification
- ✅ `BIRGEPassenger` — BUILD SUCCEEDED
- ✅ `BIRGEDrive` — BUILD SUCCEEDED

### What's next
- Phase 0 remaining: ADR-005, context doc updates
- Phase 1: Backend data models (RecurringRoute, MonthlyCommutePlan, migrations)
- Phase 2: Passenger subscription tier rename + CommuteSetupFeature

Итог
Выполнено:

Полный code review — 28 findings задокументированы в implementation_plan.md
Phase 0 (частично) — TODO(subscription-pivot) маркеры добавлены в 6 критических файлов (iOS + backend)
Phase 3 safe fix — цены водительской подписки исправлены: 4,900/8,900/14,900 → 19,000/28,000/38,000₸. Убрана неподтверждённая "Гарантия дохода 90 дней"
Верификация — оба приложения (BIRGEPassenger + BIRGEDrive) BUILD SUCCEEDED ✅
Следующие шаги (готовы к исполнению по команде):

Phase 0: ADR-005 + обновление context документов
Phase 1: Backend модели RecurringRoute, MonthlyCommutePlan, миграции
Phase 2: Переименование тиров подписки → solo_corridor/multi_corridor/flex_pack
Task
Walkthrough
SubscriptionsFeature.swift
+6
-0
HomeView.swift
+2
-1
HomeFeature.swift
+2
-0
PassengerSubscription.swift
+3
-0
Corridor.swift
+2
-0
SubscriptionsService.swift
+2
-0
DriverRegistrationFeature.swift
+5
-5
23:48
