---
title: Universal AI Handoff Context - BIRGE
version: v3.0
created: 2026-05-05
updated: 2026-05-06
timezone: Asia/Almaty
purpose: self-contained context prompt for Claude, Gemini, ChatGPT, NotebookLM, or any AI without file access
source_snapshot: Dev Brain + app repo + Business Sessions 1–5, inspected 2026-05-06
---

# Universal AI Handoff Context — BIRGE

Use this whole document as the starting context for any AI chat about BIRGE. The receiving AI may not have file access, so this file intentionally includes project facts, architecture, current status, business decisions, and next priorities. If a later message conflicts with this document, the later message wins.

---

## Instructions To The Receiving AI

You are helping Arsen plan, design, build, document, and deeply understand BIRGE — a native iOS ride-sharing / subscription commute platform for Almaty, Kazakhstan.

Your default language with Arsen should be **Russian**, but keep technical names in English when they are code/framework names. Be structured, practical, and honest. Arsen wants both product planning and technical mentorship, but he does not want every code task explained like a classroom lecture. Help create durable Markdown knowledge files and explain deeply when he asks.

Important behavior:
- Treat this document as the source context if you cannot access files.
- Do not invent that you inspected the repo if you cannot see it.
- If you need exact code, ask Arsen to paste the file.
- Prefer concrete next actions over abstract advice.
- Keep BIRGE's current architecture intact unless Arsen explicitly asks for a redesign.
- Never suggest Flutter, React Native, SwiftData, CoreData, MVVM, Combine, Node.js, or Go as replacements for the chosen core stack.
- Do not tell Arsen to rebuild from scratch. The project is already far along.
- When planning, separate immediate demo/product work from future research/scale work.
- When discussing iOS: SwiftUI + TCA + GRDB + URLSession + Keychain + WebSocket.
- When discussing backend: Swift Vapor 4 + Fluent + PostgreSQL + Redis.
- When discussing AI/ML: Python/FastAPI/XGBoost/DBSCAN is the future ML lane, separate from the transactional backend.

---

## Canonical Project Identity

**Project name:** BIRGE.

**Domain:** Subscription-based shared commute platform for Almaty, Kazakhstan.

**Core product idea (updated after Business Sessions 1–5):**

BIRGE is **not a taxi app**. It is a **transportation-as-a-subscription** service for daily commuters.

The core mechanic:
1. User opens the app once at the start of the month.
2. User declares all their recurring commute routes (work, gym, university, etc.).
3. Each route becomes an independent corridor — system matches separately.
4. System pre-matches drivers and co-passengers ahead of time for each corridor.
5. System builds a full commute plan for the month.
6. User pays a subscription and stops thinking about transport — rides just happen.

This is the primary product. It competes not with Yandex Go (on-demand), but with monthly bus passes, corporate shuttles, and UvU Shuttle-type services.

On-demand taxi (Tier 1) is a fallback/acquisition channel, not the core wedge.

**Strategic product decisions (confirmed Sessions 1–5):**
> Wedge = subscription commute, not on-demand. Do not compete with Yandex Go on its own turf.
> Driver monetization = subscription only, zero per-ride commission. Ever. This is the core narrative.
> Corridor model = hub-to-hub. One pickup point, one dropoff point. No intermediate stops on MVP.
> First corridor = Alatau City (Алатауский район) → Есентай / Аль-Фараби, 07:15–08:30.
> Current priority = business logic for UI Mockup design, not real-world launch.

**Maршрутная модель пассажира (confirmed Session 4):**
Пользователь заносит несколько независимых маршрутов. Каждый маршрут — отдельный коридор с независимым matching. Пример для типичного пользователя:
```
Коридор 1: Дом → Работа        (5 раз/неделю, утро)
Коридор 2: Работа → Дом        (2 раза/неделю, вечер)
Коридор 3: Работа → Зал        (3 раза/неделю, вечер)
Коридор 4: Зал → Дом           (3 раза/неделю, поздний вечер)
```

**User's personal goals:**
- Build BIRGE as a serious taxi/mobility startup.
- Use BIRGE as a deep iOS learning vehicle.
- Become a strong iOS developer / software engineer with AI.
- Build enough technical and research depth for future European university applications, potentially through Bolashak.
- Eventually write a scientific article on AI-assisted route optimization / matching / dispatch.

---

## Canonical Local Paths

App repo:
`/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp`

Dev Brain vault:
`/Users/arsenabduhalyk/Documents/Dev_ArsensBrain/Projects/Birge Taxi`

Xcode project root:
`/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger`

Backend:
`/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger/birge-vapor`

Canonical current names:
- Passenger app target: `BIRGEPassenger`
- Driver app target: `BIRGEDrive`
- Shared Swift package: `BIRGECore`
- Backend app: `birge-vapor`
- Main branch currently used: `feature/passenger-liquid-glass-ui`

Historical names to avoid:
- `BIRGEDriver` is stale → current is `BIRGEDrive`.
- Go/Gin backend references are stale → current backend is Swift Vapor 4.
- `BIRGEApp.xcworkspace` is stale → current is `BIRGEPassenger.xcodeproj`.

---

## Source Of Truth Order

When files are available, resolve current truth in this order:
1. `Context/Current_Focus.md` — active lane, branch, recent verification, next best task.
2. `Tasks/` — sprint scope, progress logs, acceptance criteria.
3. `Architecture/` and `Decisions/` — durable architecture decisions and trade-offs.
4. App repo code — implementation truth when docs and code disagree.
5. `Sessions/` and `docs/codex-prompts/` — historical context only, not current truth if they conflict.

Current active lane: **UI + Navigation Rescue** (tech) + **Business Logic Definition** (product/biz).

---

## Product And Business Model

### Three Service Tiers

```text
Tier 1: On-demand ride
  Standard taxi dispatch, acquisition and fallback. Similar to Yandex Go behavior.
  Status: secondary. Not the launch wedge.

Tier 2: Shared corridor ride (pay-as-you-go)
  Passenger takes a shared ride on a corridor on demand.
  Status: secondary to subscription tier.

Tier 3: Subscription commute (PRIMARY WEDGE)
  User pre-declares monthly commute routes.
  System pre-matches drivers + co-passengers for the full month.
  Passenger pays a monthly subscription, not per-ride.
  Rides are pre-planned and predictable.
  Status: core product. Build and validate this first.
```

### Confirmed Product Direction (Session 1)

BIRGE is **transportation-as-a-subscription**, not ride-hailing.

Arsen's own phrasing of the core idea:
> *"Пользователь обозначает все маршруты на месяц, система за него всё находит заранее — водителей и пассажиров, строит план поездок. Юзер платит подписку и больше не беспокоится о такси."*

Closest global analogues:
- **Via** (New York, Berlin) — pre-scheduled shared rides on corporate contracts.
- **Shotl** (Barcelona) — demand-responsive transit for corporations.
- **Bridj** (USA, closed) — predictive bus routes based on demand.
- Informal Kazakh "ауыл таксист-попутчики" — same mechanics, just offline.

### Driver Monetization (finalized Session 3)

Drivers pay fixed monthly subscription instead of per-ride commission. **Zero per-ride commission — ever.** This is the core narrative and must never be compromised.

**Onboarding model:** Free trial (30–60 days) → fixed monthly subscription. No percentage on rides at any tier.

Documented driver tier pricing:
| Tier | Monthly fee |
|---|---|
| Starter | 19,000 KZT |
| Professional | 28,000 KZT |
| Premium | 38,000 KZT |
| Launch promo | Professional at Starter price for first 6 months |

⚠️ **Pricing conflict:** current iOS driver registration UI shows 4,900 / 8,900 / 14,900 KZT. These are incorrect demo values. The documented tiers above are the intended business truth. Must be reconciled before UI Mockup work on driver registration.

**Driver economics breakeven (calculated Session 3, route: Alatau City → Есентай, ~25 km, CNG car):**

| Passengers in car | Driver revenue/run | Driver net profit/run |
|---|---|---|
| 1 | ~1,900 KZT | ~1,200 KZT |
| 2 | ~3,800 KZT | ~3,000 KZT ≈ Yandex Go level |
| 3 | ~5,700 KZT | ~4,900 KZT ✅ clear advantage |
| 4 | ~7,600 KZT | ~6,800 KZT ✅✅ |

Yandex Go comparison (same route, solo): ~2,270 KZT net after ~19% commission.

**Conclusion:** minimum viable corridor = **2 passengers**. 3+ = driver clearly wins vs Yandex Go. Platform must guarantee 3+ passengers per run to justify driver switch.

**Driver risk during cold start:** In early months platform cannot guarantee 3 passengers. Driver may earn less than on Yandex Go. Cold start strategy must solve this before asking drivers to switch.

### Passenger Monetization (Sessions 4 + 6)

Kaspi is the primary payment method in Kazakhstan.

Confirmed passenger subscription tiers:

| Tier | Name | What's covered | For whom |
|---|---|---|---|
| **Тир 1** | Solo Corridor | One route, one time slot per day | Single predictable commute, e.g. only Home→Work |
| **Тир 2** | Multi Corridor | All pre-declared corridors covered | Main target user — Данияр with 4 routes |
| **Тир 3** | Flex Pack | Fixed pack of N rides/month, fixed per-ride price, pay per use | Unpredictable schedule, remote workers, frequent travelers |

Important decisions:
- MVP launches with all three passenger subscription tiers.
- Multi Corridor is recommended/default for the first target persona, but Solo Corridor and Flex Pack are real MVP choices.
- Тир 1 and Тир 2: zero per-ride friction. User pays subscription, gets in car, done.
- Тир 3: fixed per-ride price deducted from pack balance.
- On-demand taxi remains as fallback outside subscription, not a subscription tier.
- Specific KZT prices for each tier: TBD in Session 8 (Pricing & Unit Economics).

**First passenger profile — "Данияр" (confirmed Session 4):**

| Parameter | Value |
|---|---|
| Age | 28–30 |
| Role | Middle-level office worker, Есентай / Аль-Фараби area |
| Income | 600,000–800,000 KZT/month |
| District | Алатауский район, Alatau City ЖК |
| Weekly routes | Home→Work ×5, Work→Home ×2, Work→Gym ×3, Gym→Home ×3 |
| Current transport | Taxi only (Yandex Go) |
| Main pain | Price (~129,000 KZT/month) + daily ordering friction + waiting |
| BIRGE savings | ~45,000–51,000 KZT/month (35–40% reduction) [ASSUMPTION] |

**Why Данияр is the ideal first user:** his life is already predictable enough to commit routes a month ahead. Fixed work schedule, fixed gym days, one home address. He is a commuter, not a chaotic user.

**Corridor pickup model (confirmed Session 4–5):**
- Hub-to-hub: one fixed pickup point, one fixed dropoff point per corridor.
- No intermediate stops on MVP.
- Passengers walk max ~100m to pickup node.
- First pickup node: main entrance / parking of Alatau City ЖК.
- First dropoff node: Есентай Park area.

### Core Economic Math (calculated in Session 1)

Route assumption: Alatau → Esentai (~22 km, ~45 min).

| Passengers in car | Driver revenue/run | Driver net profit (after ~700 KZT costs) |
|---|---|---|
| 1 | ~1,900 KZT | ~1,200 KZT |
| 2 | ~3,800 KZT | ~3,000 KZT |
| 3 | ~5,700 KZT | ~4,900 KZT |
| 4 | ~7,600 KZT | ~6,800 KZT |

Yandex Go comparison (same route, solo driver): ~2,800 KZT fare, minus ~19% commission = ~2,270 KZT net.

**Conclusion:** minimum viable corridor = **2 passengers**. 3+ passengers = clear driver advantage over Yandex Go.

This holds only when BIRGE takes **no per-ride commission**, only a monthly subscription fee. This is the structural advantage.

### Competitive Positioning

| Competitor | Model | Driver take | Passenger UX |
|---|---|---|---|
| Yandex Go | Per-ride commission ~15–25% | Variable, commission-heavy | On-demand, fast |
| InDrive | Negotiated fare, lower commission | Higher but unpredictable | On-demand |
| Maxim | Commission-based | Lower take | On-demand |
| UvU Shuttle | Fixed routes, schedule | Employed driver | Fixed shuttle schedule |
| **BIRGE** | Driver subscription, no per-ride cut | Fixed cost, full fare from passenger | Pre-planned monthly commute |

BIRGE does not compete with Yandex Go on on-demand. It competes on **predictability, savings, and commute convenience**.

---

## Business Uncertainties And Open Questions (from Session 1)

These are **unresolved as of 2026-05-06**. They must be answered before major UI/tech investments.

### Critical (🔴)

| # | Uncertainty | Type |
|---|---|---|
| U1 | How many Almaty drivers are willing to switch from Yandex Go for a subscription? | Unknown |
| U2 | Minimum passenger density for a viable corridor (working assumption: 2–3 pax/run) | Assumption |
| U3 | Real passenger willingness to accept behavioral constraints (fixed time, walk 200m) | Assumption |
| U4 | **Regulatory status of paid shared rides in Kazakhstan** — may require marshrutka-type license | Unknown — consult a transport lawyer |
| U5 | When can Arsen get a Kaspi merchant contract? Is there an ИП/ТОО yet? | Unknown — no legal entity exists yet |

### High Priority (🟡)

| # | Uncertainty |
|---|---|
| U6 | How quickly can Yandex Go replicate corridor feature? (Working assumption: 12–18 months) |
| U7 | Demand seasonality — winter (−20°C) significantly changes willingness to walk to pickup nodes |
| U8 | What % of Almaty drivers currently multi-home across platforms? |
| U9 | CAC for first 100 drivers |
| U10 | Does UvU Shuttle already cover key corridors? (Not yet checked — must download and test the app) |

### Strategic Contradictions Identified (Session 1)

**Contradiction 1 — Chicken-and-egg:**
Driver won't pay subscription without passengers. Passenger won't commit without reliable drivers. Corridor doesn't work below minimum density. This is harder than standard marketplace cold-start because corridor model requires simultaneous density on the same route at the same time.

**Contradiction 2 — Two fundamentally different products:**
On-demand taxi (Tier 1) and subscription commute (Tier 3) have completely different liquidity requirements, CAC, and operational logic. Building both at once dilutes focus. Confirmed decision: **Tier 3 is the wedge.**

**Contradiction 3 — AI is marketing, not product:**
BIRGE is described as "AI-powered" but current product has zero ML. Planned Phase 2–3 AI: DBSCAN corridor detection, XGBoost demand forecasting, H3 geo-indexing. Onboarding "AI" is a filter/recommendation, not ML. This is a known conscious decision, not a fact. Real differentiator at launch must be defined.

**Contradiction 4 — Driver value proposition depends on platform density:**
"Driver earns 42% more" is true only when corridor is 70%+ full. In early months, driver may earn less than on Yandex Go. Driver acquisition strategy must account for this.

**Contradiction 5 — Survey data integrity issue:**
The 52-respondent survey used in academic lab reports and nFactorial application was filled out by Arsen himself. The Pearson r = 0.769, chi-square, and F1 scores derived from it are not real market validation. This must not be presented as real data in investor materials. Options: (A) label it as synthetic methodology demo, or (B) run a real survey of 30–50 actual Almaty commuters.

---

## Legal And Operational Blockers (must resolve before launch)

1. **No legal entity (ИП or ТОО):** Without this, Kaspi merchant account is impossible, payments are illegal, contracts with drivers cannot be signed, and investment cannot be received officially. ИП opens in 1–3 days via eGov.kz, ~1,500 KZT state fee.

2. **Regulatory status of shared rides in Kazakhstan:** The Law on Road Transport of Kazakhstan may require a different license category for vehicles carrying multiple paying passengers on a fixed route (similar to marshrutka). One consultation with a transport law attorney in Almaty: ~20,000–50,000 KZT. This must happen before writing more code.

3. **Kaspi merchant contract:** Not yet started. Without this, there is no real payment flow in Kazakhstan.

---

## Validated Assumptions And Working Hypotheses

As of Session 1, the following are working hypotheses, not confirmed facts:

| Hypothesis | Status | Confidence |
|---|---|---|
| Almaty commuters have predictable recurring routes (80% of people) | Plausible | Medium |
| Passengers spend ~100,000 KZT/month on taxi, savings of 30–40% motivates switch | Plausible math | Medium |
| Minimum viable corridor occupancy = 2 passengers | Calculated | Medium |
| Target passenger: 25–40, office worker, 70,000+ KZT/month on transport | Reasonable profile | Medium |
| Target driver: full-time taxi driver, frustrated with Yandex commission | Reasonable profile | Medium |
| Yandex Go will not react until BIRGE reaches 500+ active drivers | Assumption | Low |
| Winter (−20°C) significantly reduces willingness to walk 200m to pickup node | High likelihood | High |
| UvU Shuttle is a direct Tier 3 competitor | Needs investigation | Unknown |

---

## Current Business Priority

**Arsen's current goal:** define complete business/product logic to use as foundation for UI Mockup redesign. The existing app UI does not match the product vision. Business logic must be defined first, then UI Mockups, then implementation.

This means the next outputs needed are:
1. Complete passenger user flow (screen by screen)
2. Complete driver user flow (screen by screen)
3. Pricing & subscription logic (what user sees on payment screens)

## Business Sessions Log

| Session | Topic | Status | Key Output |
|---|---|---|---|
| **1** | Diagnostic | ✅ Done | Product defined as transportation-as-a-subscription. 5 strategic contradictions identified. Survey data integrity issue flagged. |
| **2** | Wedge Decision | ✅ Done | Wedge = Tier 3 subscription commute. On-demand is fallback only. |
| **3** | Driver Economics | ✅ Done | Breakeven at 2 pax, advantage at 3+. Free trial → subscription model. Zero per-ride commission forever. |
| **4** | Passenger Value Proposition | ✅ Done | Данияр profile. 4 independent corridors. ~129k KZT/month pain. ~45–51k savings. Alatau City → Есентай first corridor. |
| **5** | Cold Start & Liquidity | ✅ Done | Manual-first approach documented. Current priority: UI Mockup logic, not real launch. |
| **6** | Passenger User Flow + Subscription Tiers | ✅ Done | Screen-by-screen flow defined. Three subscription tiers confirmed: Solo Corridor / Multi Corridor / Flex Pack. MVP launches with all three. |
| **7** | Driver User Flow | 🔜 Next | Full screen-by-screen flow for UI Mockup |
| **8** | Pricing & Unit Economics | 🔜 Next | Concrete numbers, subscription screen logic |

---

## Immediate Next Steps

### Business / Product (current priority):
1. Define complete **passenger user flow** — screen by screen, for UI Mockup.
2. Define complete **driver user flow** — screen by screen, for UI Mockup.
3. Define **pricing & subscription screen logic** — what user sees, what they choose.
4. Use outputs above as foundation for UI Mockup redesign.

### Legal / Operational (not current priority, but must happen before real launch):
1. Open ИП via eGov.kz (~1,500 KZT, 1–3 days).
2. Consult transport law attorney on shared rides legal status (~20,000–50,000 KZT).
3. Start Kaspi merchant contract process.
4. Run 10 real user interviews (passengers + drivers).
5. Download and test UvU Shuttle app — direct Tier 3 competitor.

### Tech (parallel lane):
1. Continue UI + Navigation Rescue.
2. Reconcile driver subscription pricing (4,900/8,900/14,900 KZT in UI vs 19,000/28,000/38,000 KZT in docs).

---

## Repository Structure Snapshot

App repo conceptual structure:

```text
BIRGEApp/
  README.md
  RULES.md
  .github/workflows/
    ios.yml
    vapor.yml
  BIRGEPassenger/
    BIRGEPassenger.xcodeproj
    BIRGEPassenger.xctestplan
    .swiftlint.yml
    README_E2E_TESTS.md
    CLAUDE.md
    scripts/
    BIRGEPassenger/
      BIRGEPassengerApp.swift
      App/
        AppFeature.swift
        AppView.swift
        PassengerAppFeature.swift
        PassengerAppView.swift
      Features/
        Auth/
        Home/
        RideRequest/
        Searching/
        OfferFound/
        Ride/
        ActiveRide/
        RideComplete/
        Corridor/
        Subscriptions/
        Profile/
        AIExplanation/
        ProjectDemo/
      DesignSystem/
      UI/
      Models/
      Assets.xcassets/
    BIRGEDrive/
      BIRGEDriveApp.swift
      App/
        DriverAppFeature.swift
        DriverAppView.swift
        DriverDashboardSurfaces.swift
        DriverOnlineControlSheet.swift
        DriverOfferViews.swift
        DriverRidePrimitives.swift
        DriverActiveRideViews.swift
        DriverActiveRideDetails.swift
        DriverCompletedRideSheet.swift
        DriverRideFormatting.swift
      Features/
        DriverRegistrationFeature.swift
        DriverRegistrationView.swift
        EarningsFeature.swift
        EarningsView.swift
      UI/
      Assets.xcassets/
    BIRGECore/
      Package.swift
      Sources/BIRGECore/
        Network/APIClient.swift
        WebSocket/
        Location/
        Database/
        Models/
        DesignSystem/
      Tests/BIRGECoreTests/
    BIRGEPassengerTests/
    birge-vapor/
      Package.swift
      Dockerfile
      docker-compose.yml
      Sources/App/
        entrypoint.swift
        configure.swift
        routes.swift
        Models/
        Migrations/
        Middleware/
        Modules/
          Auth/
          Rides/
          WebSocket/
          Corridors/
          Subscriptions/
          Payments/
          Locations/
          Drivers/
          Demo/
        Support/
      Tests/AppTests/
```

Dev Brain conceptual structure:

```text
Birge Taxi/
  00_Index.md
  CLAUDE.md
  Git_Commit_Push_Rules.md
  Context/
    Current_Focus.md
    iOS_Agent_Context.md
    Backend_Agent_Context.md
    Universal_AI_Handoff_BIRGE.md
  Architecture/
    System_Overview.md
    Backend_Architecture.md
    iOS_Architecture.md
    Ride_State_Machine.md
    WebSocket_Hub_Architecture.md
    Geo_Matching_and_AI.md
    Database_Schema_and_Migrations.md
    Payment_and_Financial_Architecture.md
    Infrastructure_and_Deployment.md
    Security_and_Authentication.md
    Safety_and_Operation.md
    Data_Pipeline_and_Analytics.md
    Network_Effects_and_Growth.md
    Scaling_Roadmap.md
    Open_Questions.md
  Decisions/
    ADR-001_GRDB_vs_SwiftData.md
    ADR-002_TCA_vs_MVVM.md
    ADR-003_SwiftUI_vs_UIKit.md
    ADR-004_JWT_Storage.md
  Tasks/
    iOS_Sprint_1.md
    Backend_Sprint_1.md
    UI_Navigation_Rescue_Audit.md
    Future_Demo_Stability_and_Onboarding.md
  Business_Department/
    00_Business_Index.md
    Product_Logic.md
    Business_Open_Questions.md
    Driver_Economics.md
    Pricing.md                  ← planned
    Unit_Economics.md           ← planned
    GTM_Strategy.md
    Validation_Experiments.md
  References/
  Sessions/
  docs/
    CLAUDE_for_mockups.md
    mockups/
    codex-prompts/
```

---

## Non-Negotiable Architecture Decisions

**Backend:**
- Swift Vapor 4, not Go, not Node, not Python for transactional API.
- Fluent ORM.
- PostgreSQL 16 primary database.
- Redis 7 for OTP, rate limiting, Pub/Sub, and future GEO/search flows.
- Docker Compose for local backend stack.
- Future production: K3s, ArgoCD, Nginx.

**iOS:**
- Native iOS only.
- Swift 6.
- SwiftUI.
- TCA (The Composable Architecture) for state management.
- GRDB for local persistence. SwiftData and CoreData are rejected.
- URLSession async/await for networking.
- URLSessionWebSocketTask for WebSocket.
- Keychain for auth tokens.
- No MVVM, no ObservableObject as app architecture pattern.
- No Combine for core async work. Use async/await, AsyncStream, TCA effects.

**AI/ML:**
- Future ML service: Python/FastAPI, separate from Vapor.
- XGBoost for tabular demand forecasting.
- DBSCAN for corridor detection.
- H3 for geospatial indexing.
- OSRM self-hosted for route/ETA matrix.
- ClickHouse/Airflow/MLflow/MinIO are Phase 2 analytics/MLOps items, not launch blockers.

**Infrastructure:**
- Boring technology first.
- No Kafka until PostgreSQL LISTEN/NOTIFY/outbox becomes a measured bottleneck.
- Observability: structured logs, Prometheus/OpenTelemetry/Jaeger planned before production.

---

## Current Active Work Lane

**Active lane as of 2026-05-07:** Business Foundation First. UI + Navigation Rescue remains useful for stability, but it is no longer the primary lane until passenger/driver product flows and pricing logic are defined.

**Tech priority order during business pause:**
1. Bug fixes and build/test stabilization only.
2. Documentation cleanup.
3. Reconcile driver subscription pricing before driver mockup/UI work.
4. Avoid new product screens until business flows are defined.

**Business priority order:**
1. Real user interviews (10 passengers, 5 drivers).
2. Regulatory legal check.
3. Open ИП.
4. Define MVP scope for first real corridor pilot.
5. Continue maintaining `Business_Department/Product_Logic.md`, `Driver_Economics.md`, and `Business_Open_Questions.md`.

---

## iOS Architecture

### Global iOS Stack

```text
Swift 6
SwiftUI
TCA (The Composable Architecture)
GRDB
URLSession async/await
URLSessionWebSocketTask
Keychain
MapKit / CoreLocation
```

Minimum deployment: iOS 17+. Current `IPHONEOS_DEPLOYMENT_TARGET` observed as 26.4 on active Xcode 26.4 toolchain.

### TCA Rules

Every feature:

```swift
@Reducer
struct FeatureName {
    @ObservableState
    struct State: Equatable { }

    enum Action: Sendable { }

    @Dependency(\.apiClient) var apiClient

    var body: some Reducer<State, Action> {
        Reduce { state, action in
            switch action { }
        }
    }
}
```

Rules:
- Dependencies through `@Dependency`, not direct construction.
- Side effects through `Effect.run`.
- Long-lived lifecycle effects through `.cancellable(id:)`.
- State is `Equatable`. Actions are `Sendable`.
- Navigation delegates are explicit.
- No app-wide view-only state for business logic.
- No MVVM ViewModels.

### Passenger Root Flow

Root reducer: `AppFeature`.

Root states:
```swift
case splash(SplashFeature.State)
case needsOnboarding(OnboardingFeature.State)
case unauthenticated(OTPFeature.State)
case authenticated(PassengerAppFeature.State)
```

Root behavior:
- Splash checks Keychain for `birge_access_token`.
- Token exists → authenticated.
- No token → onboarding → OTP → authenticated.
- Logout deletes `birge_access_token`, `birge_refresh_token`, `birge_user_id`, routes to OTP.

### Passenger Main App Navigation

Main reducer: `PassengerAppFeature`.

State:
- `home = HomeFeature.State()`
- `path = StackState<Path.State>()`

Path destinations: `rideRequest`, `searching`, `offerFound`, `corridorList`, `corridorDetail`, `myCorridors`, `aiExplanation`, `projectDemo`, `subscriptions`, `ride`, `rideComplete`, `profile`, debug-only `activeRide`.

Current rescue need:
- Ride-flow navigation still uses one global `NavigationStack`.
- Next cleanup: extract real ride-flow coordinator or equivalent local pattern.
- `OfferFound` may become a sheet/overlay rather than push destination.

### Driver App Architecture

Root reducer/view: `DriverAppFeature`, `DriverAppView`, `BIRGEDriveApp`.

Root enum:
```swift
enum Root {
    case auth
    case registration
    case dashboard
    case activeRide
}
```

Driver flow:
- Auth → registration → dashboard → online → receive offer → accept → activeRide → complete → back to dashboard.
- Logout: clears tokens, resets state, stops/syncs location.

Driver UI extraction completed:
- `DriverAppView.swift` ~160 lines (was 1155).
- Extracted: `DriverDashboardSurfaces.swift`, `DriverOnlineControlSheet.swift`, `DriverOfferViews.swift`, `DriverRidePrimitives.swift`, `DriverActiveRideViews.swift`, `DriverActiveRideDetails.swift`, `DriverCompletedRideSheet.swift`, `DriverRideFormatting.swift`.

Driver rescue remaining:
- Rebuild `DriverRegistrationView` into smaller native form steps.
- Clean mock/demo naming around earnings and offer polling.
- Real driver profile/settings screen decision.
- Replace fake map background with real MapKit or calmer operational dashboard.

### BIRGECore

Shared Swift package used by both targets.

Responsibilities: `APIClient`, `TokenRefreshClient`, `KeychainClient`, WebSocket client, Location client, GRDB local database, shared DTOs/models, design system components.

`APIClient.swift` is ~2000 lines — large but central. Do not refactor carelessly.

Key API methods:
`requestOTP`, `verifyOTP`, `login`, `registerDriver`, `refreshAccessToken`, `fetchMe`, `currentUser`, `createRide`, `fetchRide`, `cancelRide`, `uploadLocationsBulk`, `fetchCorridors`, `bookCorridor`, `fetchCorridorBookings`, `fetchSubscriptions`, `activateSubscription`, `createKaspiCheckout`, `fetchDriverProfile`, `updateDriverProfile`, `fetchDriverTodayCorridors`, `fetchDriverRideOffers`, `acceptDriverRide`, `declineDriverRide`, `markDriverArrived`, `startDriverRide`, `completeDriverRide`, `fetchDemoState`, `seedDemoData`, `resetDemoData`.

### iOS Auth And Token Strategy

Base URLs:
- Debug: `http://localhost:8080/api/v1`
- Production shape: `https://api.birge.kz/api/v1`
- Override key: `BIRGE_API_BASE_URL`

Token strategy:
- Access token: in memory + legacy Keychain path for WebSocket.
- Refresh token: Keychain.
- 401 → refresh once → retry → logout if fails.
- Timer-based refresh pending (backend does not expose expiry timestamp).
- Proactive refresh before ride WebSocket open.

WebSocket auth: query token currently (`?token=<jwt>`). Future: Authorization Bearer header.

### iOS Design System

Brand:
- Primary blue: `#0EA5E9`.
- Avoid old teal `#0D9E92`.
- SF Symbols, not emoji.
- Liquid Glass for floating surfaces.

Design direction:
- Mockups give product intent, not pixel-perfect implementation.
- SwiftUI implementation should be more native, calmer, adaptive.
- Avoid nested cards. Avoid fixed sizes. Use native forms/lists/sheets.

---

## Backend Architecture

### Stack

```text
Swift Vapor 4
Fluent
PostgreSQL 16
Redis 7
JWT HS256
Docker / docker-compose
```

Modular monolith. Supersedes earlier Go/Gin idea (stale).

### Modules

- **Auth:** OTP request/verify, driver email login/register, JWT access/refresh, `/auth/me`. Local E2E OTP log (`/tmp/birge-otp.log`) must be removed before production.
- **Rides:** Create, fetch, cancel. Driver offers, accept/decline, arrived, start, complete. Row lock for concurrency. Address labels persisted.
- **Corridors:** List, book, fetch bookings. Idempotent repeat bookings.
- **Subscriptions:** Plans, activate, persist state.
- **Payments:** Kaspi deep link, append-only `payment_events`, idempotent webhook skeleton, HMAC-SHA256 validation. Real merchant contract not yet started.
- **Locations:** `POST /api/v1/locations/bulk`, persists `driver_location_records`, broadcasts `ride.location_update`.
- **Drivers:** `/drivers/me` GET/PUT/POST, `/drivers/corridors/today`, offer/command path.
- **Demo:** Dev-only `GET /demo/state`, `POST /demo/seed`, `POST /demo/reset`.

### Backend Config

Env vars: `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASS`, `DB_NAME`, `REDIS_HOST`, `REDIS_PORT`, `JWT_SECRET` (default `dev_secret_birge_2026`).

Docker Compose: Vapor on `8080:8080`, Postgres 16 internal, Redis 7 Alpine internal.

---

## Architecture Decision Records (ADRs)

ADR-001: GRDB vs SwiftData → **GRDB**. Reason: production stability, no SwiftData migration risks.

ADR-002: TCA vs MVVM → **TCA**. Reason: unidirectional data flow, testability, explicit effects.

ADR-003: SwiftUI vs UIKit → **SwiftUI**. Reason: declarative, Apple direction, TCA fit. UIKit bridge only for explicit native API gaps.

ADR-004: JWT storage → **Keychain**. Tokens must not live in UserDefaults.

Backend: Swift Vapor chosen over Go on 2026-04-27. All Go notes are historical and superseded.

---

## Open Questions And Risks

### Technical

| # | Question | Priority |
|---|---|---|
| OQ-001 | APNs direct vs FCM for driver offer push notifications | 🔴 High |
| OQ-002 | Kaspi webhook idempotency key: `kaspi_transaction_id` globally unique or needs composite? | 🔴 High |
| OQ-003 | Corridor failure fallback owner: corridors service or rides service for 60-min low-occupancy? | 🔴 High |
| OQ-004 | OSRM Kazakhstan map update cadence | 🟡 Medium |
| OQ-005 | Driver KYC score threshold governance | 🟡 Medium |
| OQ-006 | Redis Cluster vs Sentinel timing | 🟡 Medium |
| OQ-007 | ClickHouse hosting | 🟢 Low / Phase 2 |
| OQ-008 | MLflow registry location | 🟢 Low / Phase 2 |

### Business (added from Session 1)

| # | Question | Priority |
|---|---|---|
| BQ-001 | Legal status of paid shared rides in Kazakhstan | 🔴 Critical |
| BQ-002 | ИП / ТОО registration | 🔴 Critical |
| BQ-003 | Kaspi merchant contract timeline | 🔴 Critical |
| BQ-004 | Real driver interview: does the subscription model actually solve their problem? | 🔴 Critical |
| BQ-005 | Real passenger interview: does subscription commute match real commute patterns? | 🔴 Critical |
| BQ-006 | UvU Shuttle route coverage in Almaty — direct competitor to Tier 3? | 🟡 High |
| BQ-007 | Winter behavior: does −20°C kill willingness to walk to pickup nodes? | 🟡 High |
| BQ-008 | Survey data integrity — must be corrected in academic materials | 🔴 Critical |

### Known Implementation Risks

- Passenger ride-flow navigation still too centralized in global path.
- `RideMapView` is large and should be split.
- `OnboardingView` is large, split later.
- `DriverRegistrationView` is large, rescue soon.
- `APIClient.swift` very large — any refactor must be careful.
- Auth/onboarding root logic not fully per-user backend-driven yet.
- Kaspi is skeleton/deep-link, not final merchant integration.
- Driver earnings/payout not production-complete.
- Real route geometry/directions not complete.
- Driver tier pricing conflict: documented (19k/28k/38k KZT) vs UI (4,900/8,900/14,900 KZT).

---

## Recent Major Completed Work

Important app commits:
- `20ed72a1 refactor(driver): add account logout routing`
- `9a0d8905 refactor(driver): extract active ride surfaces`
- `87a442bc refactor(driver): extract dashboard components`
- `5a4a24b5 feat(core): expand shared UI component kit`
- `02d04ffa refactor(passenger): name navigation transitions`
- `4fa81d8e feat(ios): add in-app project demo mode`
- `ce265714 feat(backend): add protected demo state API`
- Earlier: subscriptions, payments, corridors, booking persistence, driver auth, driver API hookup, ride commands, background GPS, WebSocket stability, Docker stack restore, live auth fixes, ride search stability.

Completed capabilities:
- OTP auth, Keychain token storage, token refresh and retry.
- Passenger home, ride request/search/offer/ride/complete flow.
- WebSocket recovery and token refresh sync.
- Corridor list/detail/booking API.
- Passenger bookings list.
- Subscriptions API and UI.
- Kaspi checkout deep link and signed webhook skeleton.
- Driver registration onboarding.
- Driver dashboard/today corridors.
- Driver auth/session, offers accept/decline.
- Driver ride lifecycle commands.
- Driver background GPS sync.
- Driver location broadcast to passenger WebSocket.
- Demo hub with DB/Redis snapshots.
- Docker live backend stack restored.

---

## Desired Knowledge System Structure

**Business Department** (to build):
- Market, Product Logic, Pricing, Finance, Subscriptions, Legal/Safety, Metrics.

**Tech Department:**
- iOS (deepest), Backend, Database, Docker/Infra, AI/ML, QA/Testing, Architecture Decisions.

**Research Department:**
- Route optimization, AI-assisted dispatch, Graph algorithms, Traffic prediction, Scientific paper, Literature review, Experiments, Datasets.

**Career / Bolashak Department:**
- Europe universities, SE / AI Engineering programs, Scholarship requirements, IELTS/TOEFL, Motivation letter, Research proposal.

**AI Tools / Sources Department:**
- OpenRouter, OpenClaw, Agents/subagents/MCP, AI engineer learning path, Harvard/MIT/Stanford lectures.

**Planned BIRGE_iOS Book:**
- Markdown book covering: project overview, Swift foundations, SwiftUI, TCA, navigation, state management, APIClient, auth, rides, maps/location, WebSocket, design system, testing/debugging, OOP/architecture patterns, glossary.

---

## How To Help Arsen Productively

**If Arsen asks for planning:**
- Structure the chaos. Separate now / next / later. Keep one active lane.

**If Arsen asks for code:**
- Respect existing TCA patterns. No unrelated refactors. Include tests for meaningful reducer/backend changes.

**If Arsen asks for business logic:**
- Build durable Markdown docs. Use frameworks: pricing, unit economics, funnel, metrics, market assumptions. Mark assumptions clearly.

**If Arsen asks for research:**
- Define research question → literature review → method → experiment → metrics → paper structure. Do not jump straight to writing a paper.

**If Arsen asks for learning:**
- Connect theory to BIRGE code. Prefer "book chapter" style Markdown output.

---

## Current Best Next Steps

### Immediate non-tech (this week):
1. Conduct 10 real interviews with daily taxi users spending 70k+ KZT/month.
2. Download and ride UvU Shuttle. Document observations.
3. Consult transport law attorney about shared rides legal status in Kazakhstan.
4. Open ИП via eGov.kz.

### Immediate tech:
1. Continue UI + Navigation Rescue.
2. Choose: `DriverRegistrationView` rescue OR passenger ride-flow rescue.
3. Resolve driver subscription pricing conflict (4,900/8,900/14,900 vs 19,000/28,000/38,000 KZT).

### Next business session:
- Build complete passenger subscription commute flow screen-by-screen.
- Then build complete driver onboarding/economics flow screen-by-screen.
- Then define pricing/subscription screen logic.

### Best documentation next step:
- Expand `Business_Department/Product_Logic.md` into full passenger and driver flow specs.
- Add pricing/unit economics docs after the next business session.

---

## Compact Glossary

**BIRGE:** The whole platform.

**Corridor:** Fixed commute route with origin, destination, time window, recurring seats. Each user can have multiple independent corridors.

**Hub-to-hub corridor:** MVP corridor model. One fixed pickup point, one fixed dropoff point. No intermediate stops. Passengers walk max ~100m to pickup node.

**Corridor booking:** Passenger reserves a seat in a corridor.

**Solo Corridor (Тир 1):** Passenger subscription covering one route and one time slot.

**Multi Corridor (Тир 2):** Passenger subscription covering all pre-declared corridors. Recommended/default plan for Данияр-type users.

**Flex Pack (Тир 3):** Pack of N rides per month. Fixed per-ride price deducted from balance. For users with unpredictable schedules.

**Passenger corridor pass:** Monthly subscription covering one or more pre-declared corridors depending on selected tier.

**Driver subscription:** Driver pays fixed monthly platform fee. Zero per-ride commission. Ever.

**Free trial:** 30–60 day trial period for new drivers before subscription kicks in.

**Transportation-as-a-subscription:** Arsen's core product concept. User pre-declares monthly routes, system pre-plans all rides, user pays subscription and doesn't think about transport day-to-day.

**Данияр:** Name for the first target passenger persona. 28–30 years old, middle-level office worker, lives in Alatau City ЖК, spends ~129,000 KZT/month on taxis, predictable commute schedule.

**First corridor:** Alatau City ЖК (Алатауский район) → Есентай / Аль-Фараби. Morning: 07:15–08:30.

**Ride FSM:** Backend + iOS state machine for ride lifecycle.

**TCA:** The Composable Architecture. Reducer, State, Action, Effect, Dependency.

**GRDB:** SQLite wrapper for local GPS cache/offline persistence.

**OSRM:** Open Source Routing Machine, planned self-hosted ETA/routing engine.

**H3:** Hexagonal geospatial grid for clustering/indexing.

**DBSCAN:** Clustering algorithm for detecting natural corridors from GPS data.

**XGBoost:** ML model for demand forecasting.

**Kaspi:** Kazakhstan payment ecosystem. Required for real payments/subscriptions. Merchant contract not yet obtained.

**UvU Shuttle:** Almaty shuttle service, potential direct competitor to BIRGE Tier 3. Not yet analyzed.

**Demo Hub:** In-app/dev API diagnostic mode for defense/demo.

**UI + Navigation Rescue:** Current active tech lane — stability, view quality, routing sanity, native SwiftUI composition.

**ИП / ТОО:** Kazakh legal entity types. Required for legal operation. Not yet registered.

---

## One-Line Summary For Another AI

BIRGE is a native iOS + Swift Vapor transportation-as-a-subscription platform for Almaty commuters — passengers pre-declare monthly routes as independent corridors (e.g. Home→Work, Work→Gym, Gym→Home), system pre-matches drivers and co-passengers per corridor using hub-to-hub pickup model, users pay subscription and stop thinking about transport — driver monetization is fixed monthly subscription with zero per-ride commission ever — first target corridor is Alatau City ЖК → Есентай/Аль-Фараби — currently in UI Mockup redesign phase (business logic is being defined to inform new UI, existing app UI does not match product vision) — tech stack: SwiftUI + TCA + GRDB + WebSocket on iOS, Vapor/PostgreSQL/Redis backend — current business priority: define passenger user flow and driver user flow screen-by-screen for UI Mockup work.
