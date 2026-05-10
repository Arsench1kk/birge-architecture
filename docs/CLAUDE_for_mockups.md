# BIRGE — Codex Context File
> **Read this file completely before writing any code.**
> This file is the UI/mockup source of truth for Codex sessions on BIRGE. For overall project truth, resolve state through `CLAUDE.md`, `Context/Current_Focus.md`, and the active `Tasks/` file first.

---

## What Is BIRGE

BIRGE is a native iOS **transportation-as-a-subscription** platform for recurring Almaty commuters.

The current business foundation says BIRGE is **not taxi-first**. The primary product is monthly subscription commute: passengers declare recurring routes once, BIRGE pre-plans drivers and co-passengers, and the passenger stops ordering transport every day.

UI mockup work should pause when a screen still assumes “call taxi” as the main product. First define the business flow in [[Business_Department/Product_Logic]], then design.

Core rules:
- Tier 3 subscription commute is the wedge.
- On-demand is fallback/acquisition only.
- Driver monetization is fixed monthly subscription.
- Zero per-ride commission ever.
- Old “drivers earn ~42% more” claims require occupancy assumptions and should not be shown as unconditional proof.

There are two iOS app targets in one Xcode project:
- **BIRGEPassenger** — passenger app (find corridors, book rides, pay)
- **BIRGEDrive** — driver app (manage corridors, track earnings, GPS)

Backend: **Swift Vapor 4** (not Go, not Node — pure Swift server).

---

## Project Location

```
App repo: /Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp
Dev Brain: /Users/arsenabduhalyk/Documents/Dev_ArsensBrain/Projects/Birge Taxi

/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/
├── BIRGEPassenger/              ← Xcode project root
│   ├── BIRGEPassenger/          ← Passenger app target
│   ├── BIRGEDrive/              ← Driver app target
│   ├── BIRGECore/               ← Shared Swift Package (models, network, websocket)
│   ├── BIRGEPassengerTests/     ← Passenger XCTest target
│   ├── BIRGEPassenger.xcodeproj
│   └── birge-vapor/             ← Swift Vapor backend
│   └── Sources/App/
│       ├── Modules/Auth/        ← OTP + JWT (DONE)
│       ├── Modules/Rides/       ← Create/fetch/cancel (DONE)
│       ├── Modules/WebSocket/   ← WSHub ride-scoped (DONE)
│       ├── Modules/Corridors/   ← DONE (list, booking, persisted booking records)
│       ├── Modules/Subscriptions/← DONE (API-backed, demo activation)
│       ├── Modules/Payments/    ← PARTIAL (Kaspi deep link + signed webhook event skeleton)
│       ├── Modules/Locations/   ← DONE (bulk GPS persistence + ride WebSocket broadcast)
│       └── Modules/Drivers/     ← PARTIAL (profile registration, today corridors, offers)

/Users/arsenabduhalyk/Documents/Dev_ArsensBrain/Projects/Birge Taxi/
└── docs/
    ├── mockups/                 ← HTML mockups (READ THESE before building any View)
    │   ├── tokens.css           ← Shared mockup tokens
    │   ├── shared/birge-v2.css  ← Current blue visual direction
    │   ├── passenger/           ← P-00 through P-23
    │   └── driver/              ← D-00 through D-17
    └── codex-prompts/           ← Step-by-step build prompts
```

---

## Current Mockup Plan — 2026-05-09

Current UI/mockup work is in a subscription-first redesign lane.

Order of work:

```text
Business/product flow -> Open Design HTML mockups -> approved SwiftUI implementation
```

Do not start new SwiftUI product implementation from the old taxi-first mockups until the relevant passenger/driver flow is clear in the business docs.

### Open Design Working Copy

Open Design improved mockups live here:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/passenger
```

Legacy Open Design mockups live here and must not be overwritten:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/legacy-birge-mockups
```

The original Dev Brain mockups under `docs/mockups/` are historical references, not the place for Open Design edits.

### Current Improved Passenger Screens

The current Open Design passenger set contains 78 improved screens and is checkpointed as the current full passenger product mockup set.

It covers:
- registration and trust onboarding
- first-time monthly route setup
- subscription selection, payment, billing, receipts, plan changes, pause/cancel flows
- profile, settings, privacy
- today plan and route management
- contextual BIRGE route insights and proactive assistant states
- daily corridor lifecycle and edge cases
- history, issue reporting, support, safety, sharing, emergency contacts, driver preferences, and credit/review request
- fallback taxi as a secondary flow outside subscription
- corridor activation and corridor ready states

`improved/passenger/index.html` is the active gallery for all improved passenger screens.

### Next Mockup Batch

Passenger mockup expansion is complete enough to pause new passenger screen creation.

Current Open Design priority:

```text
Driver app mockups
```

Use this driver roadmap as the source of truth for the next lane:

```text
/Users/arsenabduhalyk/Documents/Dev_ArsensBrain/Projects/Birge Taxi/docs/Driver_Product_Mockup_Roadmap.md
```

Do not copy the old driver taxi/ride-hailing assumptions forward. The driver app must be redesigned around subscription-first corridor work, fixed monthly driver monetization, predictable routes, reliable operations, and payout clarity.

### Demoted / Historical Concepts

Keep these only as fallback, support, or ride-day execution concepts:

- on-demand ride
- live driver search
- offer found as a decision moment
- per-ride pricing for main subscription plans
- AI as a marketing explanation

For Solo Corridor and Multi Corridor, never show per-ride pricing. They are monthly subscription plans with zero per-ride friction after activation.


## Passenger Full Product Mockup Roadmap — 2026-05-09

The passenger Open Design work has completed the full-product expansion pass. The current set contains 78 improved screens for the subscription commute product.

A dedicated roadmap is now tracked here:

```text
/Users/arsenabduhalyk/Documents/Dev_ArsensBrain/Projects/Birge Taxi/docs/Passenger_Product_Mockup_Roadmap.md
```

Current next Open Design batch:

```text
Driver app mockups
```

High-level passenger expansion order:

```text
Registration and trust onboarding
-> full route management
-> subscription lifecycle
-> daily commute edge cases
-> support and safety
-> proactive assistant states
-> fallback taxi as secondary flow
```

Use the roadmap file as the source of truth before generating new Open Design prompts for passenger screens.

## The Mockups — What They Are and How to Use Them

The `docs/mockups/` folder contains the original historical HTML mockups.
They are now **reference material**, not final visual specification.

For new product work, prefer the Open Design improved mockups after they have been reviewed and approved.

### Rules for using mockups:
1. **Read the mockup HTML before writing any View** — understand the layout, components, and data displayed.
2. **Do NOT copy the mockup 1:1** — the mockups are direction, not pixel-perfect spec. Make the SwiftUI implementation **better** than the mockup where possible.
3. **Replace emoji/stickers with SF Symbols** — mockups use emoji for icons. In SwiftUI, always use `Image(systemName:)` with the appropriate SF Symbol.
4. **Apply Liquid Glass to all floating surfaces** — any bottom sheet, card, pill, or overlay that appears on top of a map or blurred background must use `.glassEffect()` (iOS 26 API) or `.ultraThinMaterial` as fallback.
5. **Current brand colors** — passenger blue is `#037AFB`; driver / wordmark navy is `#021638`. Do not revert to old teal or older sky-blue-only direction.

### Mockup → SwiftUI mapping:
| Mockup file | SwiftUI file | Status |
|---|---|---|
| passenger/P-00-splash.html | SplashView.swift | ✅ Done |
| passenger/P-01-phone.html | OTPView.swift | ✅ Done |
| passenger/P-02-otp.html | OTPView.swift | ✅ Done |
| passenger/P-03-onboarding.html | OnboardingView.swift | ✅ Built with commute setup |
| passenger/P-03a-commute-step1.html | OnboardingView.swift | ✅ Built into onboarding |
| passenger/P-03b-commute-step2.html | OnboardingView.swift | ✅ Built into onboarding |
| passenger/P-03c-commute-step3.html | OnboardingView.swift | ✅ Built into onboarding |
| passenger/P-03d-commute-step4.html | OnboardingView.swift | ✅ Built into onboarding |
| passenger/P-03e-commute-step5.html | OnboardingView.swift | ✅ Built into onboarding |
| passenger/P-04-home.html | HomeView.swift | ✅ Rebuilt with Liquid Glass + corridor previews |
| passenger/P-05-search.html | SearchingView.swift | ✅ Built; ride-flow rescue remains |
| passenger/P-06a-corridor-list.html | CorridorListView.swift | ✅ API-backed corridor list |
| passenger/P-06b-corridor-detail.html | CorridorDetailView.swift | ✅ Built with API booking + visible persisted booking status |
| passenger/P-08-offer.html | OfferFoundView.swift | ✅ Built with countdown + confirm flow |
| passenger/P-09-driver-en-route.html | RideMapView.swift | ✅ Production events/recovery banner wired; view split still pending |
| passenger/P-12-ride-complete.html | RideCompleteView.swift | ✅ UI polished |
| passenger/P-13-boarding-code.html | BoardingCodeView.swift | ✅ Done |
| passenger/P-14-rides-history.html | MyCorridorsView.swift | ✅ Corridor bookings list |
| passenger/P-17-subscriptions.html | SubscriptionsView.swift | ✅ Built with live API plans |
| passenger/P-18-subscription-detail.html | SubscriptionsView.swift | ✅ Built with live API activation + Kaspi handoff |
| passenger/P-19-profile.html | ProfileView.swift | ✅ Real API |
| passenger/P-23-ai-explanation.html | AIExplanationView.swift | ✅ Built with Liquid Glass + SF Symbols |
| driver/dashboard.html | DriverAppView.swift + extracted driver surfaces | ✅ Live auth/profile/today corridors/ride commands/address labels hooked; live MapKit directions next |
| driver/D-05-online-waiting.html | DriverDashboardSurfaces.swift + DriverOnlineControlSheet.swift | ✅ Built into dashboard online state |
| driver/offer.html | DriverOfferViews.swift | ✅ Built into dashboard offer state |
| driver/D-07-en-route.html | DriverActiveRideViews.swift + DriverActiveRideDetails.swift | ✅ Built into active ride pickup state with guidance cue |
| driver/D-08-arrived.html | DriverActiveRideViews.swift + DriverActiveRideDetails.swift | ✅ Built into active ride boarding state |
| driver/D-09-ride-in-progress.html | DriverActiveRideViews.swift + DriverActiveRideDetails.swift | ✅ Built into active ride in-progress state with guidance panel |
| driver/D-10-ride-complete.html | DriverCompletedRideSheet.swift | ✅ Built into completed ride summary |
| driver/D-03a-vehicle.html | DriverRegistrationView.swift | ✅ Built into registration flow |
| driver/D-03b-documents.html | DriverRegistrationView.swift | ✅ Built into registration flow |
| driver/D-03c-tier-selection.html | DriverRegistrationView.swift | ✅ Built into registration flow |
| driver/D-12-earnings.html | EarningsView.swift | ⚠️ Mocked |
| driver/D-14-payout.html | Not implemented as native screen yet | ⚠️ Mockup only |

---

## Architecture Rules — Always Follow These

### iOS Stack
- **Swift 6** — strict concurrency enabled
- **TCA (The Composable Architecture)** — ALL state management goes through `@Reducer`. No `@State` in Views for business logic. No `ObservableObject`.
- **SwiftUI** — all UI. Use UIKit/AppKit bridges only for an explicit native API gap.
- **GRDB** — local persistence. Not CoreData, not SwiftData.
- **URLSession** — networking. No Alamofire, no third-party HTTP libraries.
- **Keychain** — token storage. Never UserDefaults for auth tokens.

### TCA Pattern (follow exactly):
```swift
@Reducer
struct FeatureNameFeature {
    @ObservableState
    struct State: Equatable { ... }
    
    enum Action { ... }
    
    @Dependency(\.apiClient) var apiClient
    @Dependency(\.continuousClock) var clock
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action { ... }
        }
    }
}

struct FeatureNameView: View {
    @Bindable var store: StoreOf<FeatureNameFeature>
    var body: some View { ... }
}
```

### Design System
**Brand color:** `#0EA5E9` (sky blue) — use `BIRGEColors.brandPrimary`
**Background:** white `#FFFFFF` / dark `#0C1A2E`
**Text primary:** `#0C1A2E`
**Text secondary:** `#64748B`
**Success:** `#10B981`
**Danger:** `#EF4444`

**BIRGEColors.swift** is in `BIRGECore/Sources/BIRGECore/DesignSystem/`.

### Liquid Glass — iOS 26
Apply `.glassEffect()` to:
- Bottom sheets (any sheet that slides over map)
- Floating cards (corridor cards, info cards on top of map)
- Pills and badges (AI match pill, status pills)
- Tab bar (custom tab bar overlay)

Fallback for iOS 18: `.ultraThinMaterial` + custom stroke.

```swift
// Correct Liquid Glass usage:
.glassEffect(.regular.interactive())      // interactive sheets
.glassEffect(.regular)                    // static cards
.glassEffect(.thin)                       // subtle overlays
```

**Never** use plain `Color.white` for surfaces that appear over a map or blurred background.

### SF Symbols — Always Prefer Over Emoji
| What | SF Symbol |
|---|---|
| Location / GPS | `location.fill`, `location.circle.fill` |
| Car / Vehicle | `car.fill`, `car.2.fill` |
| Person | `person.fill`, `person.circle.fill` |
| Time | `clock.fill`, `timer` |
| Money | `tengesign.circle.fill`, `creditcard.fill` |
| Star / Rating | `star.fill`, `star.leadinghalf.filled` |
| Route / Direction | `arrow.triangle.turn.up.right.circle.fill` |
| Checkmark / Done | `checkmark.circle.fill`, `checkmark.seal.fill` |
| Warning | `exclamationmark.triangle.fill` |
| AI / Sparkle | `sparkles`, `brain.head.profile` |
| Seat | `person.crop.rectangle.fill` |
| Corridor | `point.topleft.filled.down.to.point.bottomright.curvepath` |
| Wallet | `wallet.pass.fill` |
| Settings | `gearshape.fill` |
| Phone | `phone.fill` |
| Documents | `doc.fill`, `doc.badge.arrow.up.fill` |

---

## What Is Already Working (Do Not Break)

### Auth flow — 100% complete
- `POST /api/v1/auth/otp/request` — send OTP via SMS
- `POST /api/v1/auth/otp/verify` — verify OTP, get access + refresh tokens
- `GET /api/v1/auth/me` — get current user profile
- JWT access token (1h expiry) → Keychain
- JWT refresh token (30d expiry) → Keychain
- Automatic token refresh interceptor (one-flight actor, no duplicate refresh)
- AppFeature reads Keychain on launch → routes to OTP or Home

**Do NOT modify without a specific auth task:** `OTPFeature.swift`, `OTPView.swift`, `APIClient.swift` auth methods, `TokenRefreshClient`, `KeychainClient`.

### WebSocket — production ride flow working, still needs future polish
- `WebSocketClient.liveValue` connects to `/ws/ride/:rideId?token=<jwt>`
- Reconnect uses deferred connected state, retry/backoff, and control-frame filtering
- `SearchingFeature` listens for canonical `driver_accepted` / `ride_matched` flow
- `driver_accepted` → `PassengerAppFeature` → `OfferFoundFeature` → `RideFeature`
- Vapor WSHub: ride-scoped channels

**Still pending:** WebSocket Bearer header instead of query token, deeper driver WebSocket command flow, and cleaner passenger ride-flow coordination.

### Ride creation — done
- `POST /api/v1/rides` creates a ride in PostgreSQL
- Returns `rideId` used for WebSocket subscription
- `RideFeature` can fetch and cancel rides

---

## Current Sprint Priority (do tasks in this order)

Current active lane: **Business Foundation + Open Design Mockup Redesign**.

Priority order:
1. Define passenger subscription commute flow screen-by-screen.
2. Redesign first-time passenger onboarding around monthly route declaration.
3. Place subscription selection after route setup, reliability, and monthly savings.
4. Refine the current six improved passenger screens so they read as subscription commute, not taxi-first.
5. Define driver onboarding/economics flow before redesigning driver mockups.
6. Only after approved mockups, resume SwiftUI/iOS implementation.

Allowed technical work during this lane:
- build/test stabilization
- bug fixes
- documentation cleanup
- small UI safety fixes that do not add taxi-first product assumptions

Avoid:
- new taxi-first screens
- backend product expansion for one-off ride requests
- implementing new subscription data models before pricing/flow logic is clear

---

## Vapor Backend Rules

The backend is **Swift Vapor 4**, not Go, not Python.

```
birge-vapor/Sources/App/
├── Modules/Auth/        ← DONE — OTP, JWT, /auth/me
├── Modules/Rides/       ← DONE — create, fetch, cancel, WebSocket
├── Modules/Corridors/   ← DONE — GET /corridors, booking, booking history
├── Modules/Locations/   ← DONE — POST /locations/bulk stores GPS batches and broadcasts ride.location_update
├── Modules/Payments/    ← PARTIAL — Kaspi checkout deep link + signed webhook event skeleton
├── Modules/Drivers/     ← PARTIAL — profile, registration, today corridors, ride offers/commands
├── Modules/Subscriptions/← DONE — plans, activation, passenger subscription state
└── Modules/Demo/        ← DEV ONLY — defense/demo state, seed, reset
```

**API prefix:** all endpoints are at `/api/v1/...`
**Auth:** JWT via `Bearer` header, or `?token=` query param for WebSocket
**Database:** PostgreSQL via Fluent ORM
**Migrations:** in `birge-vapor/Sources/App/Migrations/`

When adding a new Vapor endpoint:
1. Add/extend Fluent model under `Sources/App/Models/` when persistence is needed.
2. Add migration under `Sources/App/Migrations/`.
3. Add DTO/Controller/Service under `Sources/App/Modules/<Name>/`.
4. Register route in `routes.swift`.
5. Add Vapor tests and update the iOS `APIClient` DTO/method if the endpoint is app-facing.

---

## How to Write a Codex Prompt for This Project

When you need to build something, structure your Codex prompt like this:

```
1. Read the mockup: docs/mockups/passenger/P-06a-corridor-list.html
2. Read the current state: [relevant existing Swift files]
3. Build: [specific feature/view/endpoint]
4. Rules:
   - Use .glassEffect() for all floating surfaces
   - Use SF Symbols, no emoji
   - TCA pattern: @Reducer struct + @ObservableState
   - Do not modify auth or websocket files
5. When done: list all files changed
```

**One prompt = one feature.** Do not combine corridor list + corridor detail + vapor endpoint in one prompt. Split into three.

---

## Common Pitfalls — Avoid These

| Pitfall | Correct approach |
|---|---|
| `@StateObject` / `ObservableObject` | Use `@Reducer` + `StoreOf<>` |
| `UserDefaults` for tokens | `KeychainClient` |
| `CoreData` / `SwiftData` | `GRDB` |
| Hard-coded coordinates (43.22, 76.85) | `LocationClient.currentLocation` |
| CSS gradient as map | Native MapKit surface or a clearly intentional dashboard background |
| Emoji icons | `Image(systemName:)` |
| Plain `Color.white` overlay on map | `.glassEffect()` or `.ultraThinMaterial` |
| Teal `#0D9E92` as brand color | Blue `#0EA5E9` |
| Per-ride commission language | Subscription model language |
| `fatalError` in client live values | Real implementation or non-fatal error |

---

## Key Business Concepts (for UI copy and data models)

- **Corridor** — a fixed route (origin → destination) with a scheduled departure time window (e.g. 07:30–08:00) that runs Monday–Friday. Not a one-off ride.
- **Corridor booking** — a passenger reserves a seat on a specific corridor. Different from requesting a ride.
- **AI Match Score** — percentage showing how well a corridor matches the passenger's commute profile. Shown as "98% совпадение" on corridor cards.
- **Commute profile** — the passenger's daily routine (origin, destination, time, days of week) collected during onboarding wizard (P-03a through P-03e).
- **Driver subscription** — drivers pay a fixed monthly fee (Starter 19,000₸ / Professional 28,000₸ / Premium 38,000₸) instead of per-ride commission. They keep 100% of ride revenue.
- **Passenger corridor pass** — passengers subscribe to a monthly pass for their specific corridor. Fixed price, guaranteed seat.
- **Boarding code** — 3-digit code shown to passenger (P-13) that they tell the driver to verify identity. Format: "4·2·7".
- **On-demand ride** — regular taxi request (Tier 1, acquisition/fallback product). Not the main product.

---

*Last updated: 2026-05-09*
*Next review: after driver app mockup Phase 0/1 is reviewed*
---

## Latest Codex UI Session (2026-05-03)

- Active app branch: `feature/passenger-liquid-glass-ui`
- Status: pushed to GitHub after grouped commits.
- Built: shared Liquid Glass modifiers, Splash, Onboarding, Home, BoardingCode polish, RideComplete polish, Ride/RideRequest SF Symbol cleanup, CorridorList, CorridorDetail, and Home navigation into corridors.
- Verified: `BIRGEPassenger` build succeeds on `iPhone 17 Pro` simulator.
- Blocked: `xcodebuild test` still hits `SwiftNavigation` / `CasePathsCore` linker errors before tests execute.
- Rule for next work: read the relevant mockup, improve the SwiftUI result rather than copying it exactly, keep Liquid Glass on floating surfaces, and use SF Symbols instead of emoji.
