# BIRGE — Codex Context File
> **Read this file completely before writing any code.**
> This file is the single source of truth for every Codex session on the BIRGE project.

---

## What Is BIRGE

BIRGE is a native iOS ride-sharing platform for Almaty, Kazakhstan.
The core idea: passengers who commute the same route every day are grouped into **shared corridor rides** by an AI matching system. They pay 30–40% less. Drivers earn ~42% more because they use a monthly subscription instead of paying per-ride commission to the platform.

There are two iOS apps in one Xcode workspace:
- **BIRGEPassenger** — passenger app (find corridors, book rides, pay)
- **BIRGEDriver** — driver app (manage corridors, track earnings, GPS)

Backend: **Swift Vapor 4** (not Go, not Node — pure Swift server).

---

## Project Location

```
~/Documents/Dev_ArsensBrain/Projects/Birge Taxi/
├── BIRGEApp/                    ← Xcode workspace (iOS)
│   ├── BIRGEPassenger/          ← Passenger app target
│   ├── BIRGEDriver/             ← Driver app target
│   ├── BIRGECore/               ← Shared Swift Package (models, network, websocket)
│   └── BIRGEApp.xcworkspace
├── birge-vapor/                 ← Swift Vapor backend
│   └── Sources/App/
│       ├── Modules/Auth/        ← OTP + JWT (DONE)
│       ├── Modules/Rides/       ← Create/fetch/cancel (DONE)
│       ├── Modules/WebSocket/   ← WSHub ride-scoped (DONE)
│       ├── Modules/Corridors/   ← NOT YET IMPLEMENTED
│       ├── Modules/Locations/   ← NOT YET IMPLEMENTED
│       └── Modules/Payments/    ← NOT YET IMPLEMENTED
└── docs/
    ├── mockups/                 ← HTML mockups (READ THESE before building any View)
    │   ├── tokens-v3.css        ← Final unified design tokens
    │   ├── passenger/           ← P-00 through P-23
    │   └── driver/              ← D-00 through D-17
    └── codex-prompts/           ← Step-by-step build prompts
```

---

## The Mockups — What They Are and How to Use Them

The `docs/mockups/` folder contains **54 HTML files** — one per screen.
They are the **visual specification** for every SwiftUI view you write.

### Rules for using mockups:
1. **Read the mockup HTML before writing any View** — understand the layout, components, and data displayed.
2. **Do NOT copy the mockup 1:1** — the mockups are direction, not pixel-perfect spec. Make the SwiftUI implementation **better** than the mockup where possible.
3. **Replace emoji/stickers with SF Symbols** — mockups use emoji for icons. In SwiftUI, always use `Image(systemName:)` with the appropriate SF Symbol.
4. **Apply Liquid Glass to all floating surfaces** — any bottom sheet, card, pill, or overlay that appears on top of a map or blurred background must use `.glassEffect()` (iOS 26 API) or `.ultraThinMaterial` as fallback.
5. **Mockups use birge-v2 blue** — the final brand color is `#0EA5E9` (sky blue), not the old teal `#0D9E92`. All new screens use blue.

### Mockup → SwiftUI mapping:
| Mockup file | SwiftUI file | Status |
|---|---|---|
| passenger/P-00-splash.html | SplashView.swift | ✅ Done |
| passenger/P-01-phone.html | PhoneEntryView.swift | ✅ Done |
| passenger/P-02-otp.html | OTPView.swift | ✅ Done |
| passenger/P-03-onboarding.html | OnboardingView.swift | ✅ Basic done |
| passenger/P-03a-commute-step1.html | CommuteStep1View.swift | ❌ Not built |
| passenger/P-03b-commute-step2.html | CommuteStep2View.swift | ❌ Not built |
| passenger/P-03c-commute-step3.html | CommuteStep3View.swift | ❌ Not built |
| passenger/P-03d-commute-step4.html | CommuteStep4View.swift | ❌ Not built |
| passenger/P-03e-commute-step5.html | CommuteStep5View.swift | ❌ Not built |
| passenger/P-04-home.html | HomeView.swift | ✅ Rebuilt with Liquid Glass + corridor previews |
| passenger/P-05-search.html | SearchView.swift | ⚠️ Partial |
| passenger/P-06a-corridor-list.html | CorridorListView.swift | ✅ Built with mock data |
| passenger/P-06b-corridor-detail.html | CorridorDetailView.swift | ✅ Built with mock data |
| passenger/P-08-offer.html | OfferFoundView.swift | ❌ Not built |
| passenger/P-09-driver-en-route.html | RideMapView.swift | ⚠️ UI polished, production events/banner pending |
| passenger/P-12-ride-complete.html | RideCompleteView.swift | ✅ UI polished, mock data |
| passenger/P-13-boarding-code.html | BoardingCodeView.swift | ✅ Done |
| passenger/P-17-subscriptions.html | SubscriptionsView.swift | ⚠️ Wrong model |
| passenger/P-19-profile.html | ProfileView.swift | ✅ Real API |
| passenger/P-23-ai-explanation.html | AIExplanationView.swift | ❌ Not built |
| driver/dashboard.html | DriverAppView.swift | ⚠️ Mocked |
| driver/D-05-online-waiting.html | DriverOnlineView.swift | ❌ Not built |
| driver/offer.html | DriverOfferView.swift | ❌ Not built |
| driver/D-03a-vehicle.html | VehicleInfoView.swift | ❌ Not built |
| driver/D-03b-documents.html | DocumentsUploadView.swift | ❌ Not built |
| driver/D-03c-tier-selection.html | TierSelectionView.swift | ❌ Not built |
| driver/D-12-earnings.html | EarningsView.swift | ⚠️ Mocked |
| driver/D-14-payout.html | PayoutView.swift | ⚠️ Mocked |

---

## Architecture Rules — Always Follow These

### iOS Stack
- **Swift 6** — strict concurrency enabled
- **TCA (The Composable Architecture)** — ALL state management goes through `@Reducer`. No `@State` in Views for business logic. No `ObservableObject`.
- **SwiftUI** — all UI. No UIKit except `BIRGEMapView` (UIViewRepresentable wrapping MKMapView).
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

**BIRGEColors.swift** is in `BIRGECore/Sources/BIRGECore/`

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

**Do NOT modify:** `AuthFeature.swift`, `OTPFeature.swift`, `APIClient.swift` (auth parts), `AuthSession.swift`

### WebSocket — 75% complete
- `WebSocketClient.liveValue` connects to `/ws/ride/:rideId?token=<jwt>`
- Reconnect: 3 attempts, 2s/4s/8s backoff
- `SearchingFeature` listens for `ride_matched` event
- `ride_matched` → `PassengerAppFeature` → navigate to `RideFeature`
- Vapor WSHub: ride-scoped channels

**What's missing:** UI for disconnection banner, `ride_started` / `driver_arrived` / `ride_completed` events, Driver WebSocket.

### Ride creation — done
- `POST /api/v1/rides` creates a ride in PostgreSQL
- Returns `rideId` used for WebSocket subscription
- `RideFeature` can fetch and cancel rides

---

## Current Sprint Priority (do tasks in this order)

1. **Merge PRs** — 5 branches need to merge into main (see Solutions doc)
2. **Task 6** — "No connection" banner in RideMapView (30 min task, state exists, UI missing)
3. **Task 9** — CorridorListFeature + CorridorDetailFeature are built with mock data; Vapor `/corridors` API is still CRITICAL and next for real product data
4. **Task 10** — OfferFoundView (P-08) — the AI match found screen is now the next passenger UI screen
5. **Task 13** — Driver Registration multi-step (D-03a through D-03c)
6. **Task 16** — Background GPS for driver (IOS-018)
7. **Task 20** — Kaspi payment (deep link first, webhook later)

---

## Vapor Backend Rules

The backend is **Swift Vapor 4**, not Go, not Python.

```
birge-vapor/Sources/App/
├── Modules/Auth/        ← DONE — OTP, JWT, /auth/me
├── Modules/Rides/       ← DONE — create, fetch, cancel, WebSocket
├── Modules/Corridors/   ← TODO — GET /corridors, POST /corridors/:id/book
├── Modules/Locations/   ← TODO — POST /locations/bulk
├── Modules/Payments/    ← TODO — Kaspi webhook, payout
└── Modules/Driver/      ← TODO — registration, corridors/today
```

**API prefix:** all endpoints are at `/api/v1/...`
**Auth:** JWT via `Bearer` header, or `?token=` query param for WebSocket
**Database:** PostgreSQL via Fluent ORM
**Migrations:** in `birge-vapor/Sources/App/Migrations/`

When adding a new Vapor endpoint:
1. Create Model in `Modules/<Name>/Models/`
2. Create Migration in `Migrations/`
3. Create Controller in `Modules/<Name>/Controllers/`
4. Register route in `routes.swift`

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
| CSS gradient as map | `BIRGEMapView` (MKMapView UIViewRepresentable) |
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

*Last updated: 2026-05-03*
*Next review: after Corridor Flow (Task 9) is complete*
---

## Latest Codex UI Session (2026-05-03)

- Active app branch: `feature/passenger-liquid-glass-ui`
- Status: pushed to GitHub after grouped commits.
- Built: shared Liquid Glass modifiers, Splash, Onboarding, Home, BoardingCode polish, RideComplete polish, Ride/RideRequest SF Symbol cleanup, CorridorList, CorridorDetail, and Home navigation into corridors.
- Verified: `BIRGEPassenger` build succeeds on `iPhone 17 Pro` simulator.
- Blocked: `xcodebuild test` still hits `SwiftNavigation` / `CasePathsCore` linker errors before tests execute.
- Rule for next work: read the relevant mockup, improve the SwiftUI result rather than copying it exactly, keep Liquid Glass on floating surfaces, and use SF Symbols instead of emoji.
