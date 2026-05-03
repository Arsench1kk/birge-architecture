---
last_updated: 2026-05-03
sprint: 1
---

# Current Focus — BIRGE

## ▶ Цель сейчас: Passenger Liquid Glass UI по итоговым mockups

### Active branch
- App repo: `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp`
- Branch: `feature/passenger-liquid-glass-ui`
- Remote: pushed to GitHub

### Session summary (2026-05-03, Passenger Liquid Glass UI)
- Read final mockups/context from `docs/` and codex prompts.
- Created grouped commits for existing local UI work and pushed them.
- Added native iOS 26 Liquid Glass support in BIRGECore with material fallback for older iOS.
- Rebuilt/polished passenger Splash, Onboarding, Home, BoardingCode, RideComplete, RideMap, and RideRequest surfaces.
- Replaced visible passenger ride emoji/stickers with SF Symbols where applicable.
- Added real TCA navigation for corridor list and corridor detail screens from Home.
- Added `CorridorListFeature/View` and `CorridorDetailFeature/View`; they now load live corridor data through `APIClient`.
- Added P-08 OfferFound confirmation flow: `Searching → OfferFound → RideMap` on confirm, decline/expiry returns to searching.
- Implemented Vapor `/api/v1/corridors` with default seeded corridors and passenger booking endpoint.
- Connected Passenger Home, corridor list, and corridor detail screens to live `APIClient.fetchCorridors` / `bookCorridor` calls with loading/error states.
- Pushed app commit `2fd2a124 feat(corridors): connect passenger corridors to API`.
- Improved RideMap WebSocket recovery UI and production lifecycle event aliases.
- Pushed app commit `51a890d7 feat(ride): improve websocket recovery events`.
- Unblocked the `SwiftNavigation` / `CasePathsCore` linker failure by pinning a compatible package graph and adding explicit runtime package products.
- Pushed app commit `cf9265e3 fix(tests): unblock passenger package linking`.
- Fixed `OTPFlowE2ETests` for the current splash-first app startup and restored full passenger XCTest suite.
- Pushed app commit `e8a38820 test(passenger): align otp flow tests with splash startup`.
- Added P-23 `AIExplanationFeature/View` with Liquid Glass cards, SF Symbols, Home AI pill navigation, CTA into corridor list, and reducer coverage.
- Pushed app commit `da9001cf feat(passenger): add ai explanation screen`.
- Rebuilt onboarding into an 8-page flow: 3 intro slides plus P-03a–P-03e commute setup for origin, destination, times, weekdays, and AI route summary.
- Pushed app commit `fe208327 feat(passenger): add commute setup onboarding`.
- Added P-17/P-18 subscriptions flow with TCA state for plans, detail, activation, and Home subscription navigation.
- Pushed app commit `9f06a2a4 feat(passenger): add subscriptions flow`.
- Implemented Vapor `/api/v1/subscriptions` overview/activation API with persisted passenger subscription state.
- Connected `SubscriptionsFeature/View` to live subscription plans and activation through `APIClient`, with loading/error/activating states.
- Pushed app commit `50c3915f feat(subscriptions): connect passenger plans to api`.
- Added Vapor Payments module for Kaspi checkout deep links and append-only/idempotent webhook events.
- Connected subscription activation UI to Kaspi handoff before demo confirmation.
- Pushed app commit `01955808 feat(payments): add kaspi subscription handoff`.
- Added persisted corridor booking records with passenger/corridor uniqueness and idempotent repeat booking behavior.
- Pushed app commit `ab5237fe feat(corridors): persist passenger bookings`.

### Verification
- ✅ `git diff --check` passed in app repo.
- ✅ `xcodebuild build -skipMacroValidation -project BIRGEPassenger.xcodeproj -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 17 Pro'` succeeds.
- ✅ Focused `RideFeatureTests`, `OTPFeatureTests`, and `OTPFlowE2ETests` pass with `-skipMacroValidation`.
- ✅ Full `BIRGEPassengerTests` pass with `-skipMacroValidation` on installed `iPhone 17 Pro` simulator; live OTP E2E remains skipped unless `RUN_LIVE_OTP_E2E=1`.
- ✅ `PassengerAppFeatureTests` cover `Home → AI Explanation → Corridor List`.
- ✅ `OnboardingFeatureTests` cover commute setup paging, inputs, day selection, add-another-route, and finish delegate.
- ✅ `SubscriptionsFeatureTests` cover API loading, plan selection, activation, and detail dismissal.
- ✅ Vapor `swift build` succeeds after subscriptions API integration.
- ✅ Vapor `swift build` succeeds after Kaspi payments module integration.
- ✅ Vapor `swift build` succeeds after persisted corridor bookings integration.

### Next best steps
1. Harden remaining passenger backend integrations: real Kaspi API signature/webhook validation and richer corridor booking UX/history.
2. Continue driver-side Sprint 1 gaps once passenger blocker/UI pass is stable.

### Agent reminder
Before continuing iOS UI work, always read [[docs/CLAUDE_for_mockups]] and the relevant HTML mockup. The mockup gives the product idea; SwiftUI implementation should improve it with native Liquid Glass and SF Symbols.


## ▶ Цель: Sprint 1 stabilization for demo + auth loop

### Стратегия
UI пассажира и водителя уже собран.
По умолчанию тесты опираются на mock/test зависимости, а live OTP E2E включается только явно.

### Порядок экранов
1. OTP Auth экран (телефон → код → войти)
2. Home экран пассажира (карта + кнопка "Вызвать")
3. Экран поиска водителя (анимация поиска)
4. Экран активной поездки (водитель едет, ETA)
5. Экран завершения (рейтинг)
6. Driver Dashboard (`BIRGEDrive` target)

### Last session (2026-04-27)
- Task: Full MVP complete — Passenger + Driver apps
- Screens done: OTP, Home, RideRequest, Searching, ActiveRide, 
  RideComplete, Profile, DriverDashboard, Earnings
- Build: ✅ Clean (both targets)
- Notes: All 5+ MVP features complete. Ready for demo.

### Session summary (2026-05-02, backend)
- Task: Backend Vapor build recovery for Swift 6.3.1 / Vapor 4.x
- Result: `birge-vapor` now builds cleanly with `swift build`
- Verified: WebSocket route handler, Redis async/Future compatibility, JWT middleware path, and async entrypoint lifecycle
- Runtime: `swift run` reaches startup, then stops on local PostgreSQL authentication for user `birge`

### Session summary (2026-05-02, iOS tests)
- Task: restore `BIRGEPassengerTests` linking and verify OTP test flow
- Result: `xcodebuild test -scheme BIRGEPassenger` now succeeds
- Verified:
  - `BIRGEPassengerTests` links Point-Free products correctly (`ComposableArchitecture`, `Dependencies`, `CasePathsCore`, `ConcurrencyExtras`)
  - `BIRGEPassengerTests/OTPFeatureTests` pass
  - `OTPFlowE2ETests.testOTPFlowSuccess` is now opt-in via `RUN_LIVE_OTP_E2E=1`
- Live E2E prerequisites:
  - local Vapor on `http://localhost:8080`
  - PostgreSQL + Redis
  - Vapor writes `/tmp/birge-otp.log`

### Session summary (2026-05-02, iOS WebSocket)
- Task: IOS-014 — WebSocketClient TCA Dependency
- Result: production-grade `WebSocketClient` implemented in BIRGECore
- Delivered:
  - `WebSocketClient.swift` — interface struct, `WebSocketEvent`/`WebSocketMessage`/`WebSocketError` enums, `DependencyKey` with `liveValue`/`testValue`
  - `LiveWebSocketClient.swift` — actor-isolated `URLSessionWebSocketTask` with 5s ping, exponential backoff (1→2→4→8→16→30s), cancellation safety
  - `WebSocketClient+DependencyValues.swift` — TCA dependency registration
  - `WebSocketClientTests.swift` — 5 unit tests using controllable mock
- BIRGECore linked to all targets (BIRGEPassenger, BIRGEDrive, BIRGEPassengerTests)

### Session summary (2026-05-02, iOS Location)
- Task: IOS-015 — LocationClient TCA Dependency
- Result: production-grade `LocationClient` implemented in BIRGECore
- Delivered:
  - `LocationClient.swift` — interface struct, `LocationUpdate`, `LocationTrackingID`, `DependencyKey` with `liveValue`/`testValue`
  - `LiveLocationClient.swift` — actor-isolated `CLLocationManager` wrapper with `LocationDelegate` bridge, offline-first GRDB writes
  - `LocationSyncService.swift` — stateless batch sync utility (200 records/batch), injectable upload closure
  - `LocationClient+DependencyValues.swift` — TCA dependency registration
  - `LocationClientTests.swift` — 5 unit tests (stream, stop, sync marks synced, batch groups of 200, idempotent)
- Note: `POST /locations/bulk` endpoint not yet defined — upload closure is a no-op placeholder

### Session summary (2026-05-02, iOS RideFeature)
- Task: IOS-016 — RideFeature State Machine
- Result: `RideFeature` implemented for passenger production ride flow
- Delivered:
  - `Coordinate` wrapper in BIRGECore for `CLLocationCoordinate2D` interoperability
  - `RideEvent` WebSocket payload models for `ride.status_changed`, `ride.location_update`, `ride.eta_updated`
  - Stub `APIClient` dependency with `fetchRide` and `cancelRide`
  - `RideFeature` 7-state TCA reducer: `requested → matched → driverAccepted → driverArriving → passengerWait → inProgress → completed`
  - `RideMapView` with live MapKit driver annotation, pickup pin, status pill, and bottom sheet
  - `PassengerAppFeature` navigation updated: `Searching → Ride → RideComplete`
  - `RideFeatureTests` added with 7 reducer tests, including WebSocket JSON decoding, cancel, reconnect, and disconnect recovery
- Verification:
  - Fixed Xcode Cmd+B errors in `RideFeature`:
    - removed `ReducerOf<Self>` circular reference by using `some Reducer<State, Action>`
    - replaced custom cancellation marker types with namespaced `String` IDs to satisfy `Hashable & Sendable`
  - Verified: `xcodebuild build -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 17 Pro'` succeeds
  - Verified: `xcodebuild build -scheme BIRGEDrive -destination 'platform=iOS Simulator,name=iPhone 17 Pro'` succeeds
  - Attempted: `xcodebuild test -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 17 Pro' -only-testing:BIRGEPassengerTests/RideFeatureTests`
  - Focused tests are still blocked before `RideFeatureTests` execute by third-party package linker failure in `swift-navigation`
  - Error: undefined `CasePathsCore.CasePathable` / `CasePathsCore.AnyCasePath` symbols while linking `SwiftNavigation.framework`
  - Note: `iPhone 16 Pro` simulator is not installed locally; used installed `iPhone 17 Pro`

### Next session
- Task: IOS-017 — API Client + Token Refresh
- Done: replaced stub `APIClient.liveValue` with authenticated URLSession implementation
- Done: added `TokenRefreshClient` dependency with in-memory access token, Keychain refresh token, 401 refresh-and-retry, and token clearing on auth expiry
- Done: wired `GET /rides/:id`, `PATCH /rides/:id/cancel`, `POST /locations/bulk`, OTP auth endpoints, `POST /auth/refresh`, `GET /auth/me`, and `POST /rides`
- Done: `xcodebuild build -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 17 Pro' -quiet` succeeds after API client work
- Remaining: proactive refresh-before-expiry behavior needs a JWT expiry source/decoder
- Remaining: WebSocket auth header
- First test blocker to resolve: `SwiftNavigation` / `CasePathsCore` linker failure before focused RideFeature tests can run
- Model: GPT 5.4/5.5 High for Xcode package/linker triage, then Claude Opus 4.6 for API architecture review


- Vault настроен (v5.1)
- BIRGE.code-workspace создан
- RULES.md в корне проекта
