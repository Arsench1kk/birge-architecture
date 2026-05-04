---
last_updated: 2026-05-04
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
- Updated CorridorDetail UX to show persisted booking status, booking id, refreshed seats/participants, and disabled re-join CTA after success.
- Pushed app commit `17d44560 feat(corridors): show passenger booking status`.
- Added `/api/v1/corridors/bookings` and Passenger `MyCorridorsFeature/View` reachable from the Home `Поездки` tab.
- Pushed app commit `061d6aaf feat(corridors): add passenger bookings list`.
- Hardened Kaspi webhook handling with `KASPI_WEBHOOK_SECRET`, HMAC-SHA256 signature validation, header fallback support, and backend unit tests.
- Pushed app commit `98948bf4 feat(payments): validate kaspi webhook signatures`.
- Started driver-side Sprint 1 gaps with D-03 registration onboarding: personal data, vehicle, documents, and subscription tier steps before dashboard.
- Pushed app commit `d83fec67 feat(driver): add registration onboarding flow`.
- Polished driver D-05 online waiting and `driver/offer` surfaces in `DriverAppView` with a map-style background, Liquid Glass top/bottom overlays, SF Symbols, offer alert, match pill, route card, metrics grid, and passenger confirmation row.
- Pushed app commit `136f91a0 feat(driver): polish online offer surfaces`.
- Polished driver D-07/D-08/D-09/D-10 lifecycle in `DriverAppView` and `DriverAppFeature`: active ride route sheet, pickup/boarding/in-progress state panels, boarding codes, passenger manifest, progress bar, and completed ride summary with next-ride action.
- Pushed app commit `079ac13a feat(driver): add active ride lifecycle polish`.
- Added driver background GPS sync path: BIRGEDrive starts `LocationClient` tracking on accepted rides, stops/syncs on completion/offline, enables driver background location Info.plist keys, and Vapor now exposes authenticated `POST /api/v1/locations/bulk` backed by `driver_location_records`.
- Pushed app commit `e778858b feat(driver): sync background locations`.
- Added live driver location broadcast: `/locations/bulk` now emits the latest batch point as canonical `ride.location_update` on `ride/<ride_id>` WebSocket channels, matching passenger `RideFeature` parsing.
- Pushed app commit `85a66398 feat(locations): broadcast driver updates`.
- Added backend Driver module: authenticated `/api/v1/drivers/me` GET/PUT/POST for registration/profile persistence and `/api/v1/drivers/corridors/today` for driver dashboard corridor candidates.
- Pushed app commit `c5012c77 feat(driver): add driver profile api`.
- Connected BIRGEDrive registration/dashboard to Driver API: registration saves through `APIClient.updateDriverProfile`, app startup loads `/drivers/me`, dashboard loads `/drivers/corridors/today`, and no-token demo fallback keeps local preview usable until driver auth exists.
- Pushed app commit `be190fa7 feat(driver): connect dashboard to profile api`.
- Added driver ride assignment/command path: backend exposes driver offers plus accept/arrived/start/complete endpoints, broadcasts canonical `ride.status_changed`, and BIRGEDrive now polls/accepts/sends lifecycle commands through `APIClient`.
- Pushed app commit `a12d75b6 feat(driver): add ride assignment commands`.
- Added BIRGEDrive driver auth/session flow: email login/register screen, APIClient `auth/login`/driver `auth/register`, token storage through shared refresh client, and removed no-token demo fallback from registration/offers.
- Pushed app commit `29903977 feat(driver): add auth session flow`.
- Persisted ride address labels: passenger ride requests now send origin/destination names, Vapor stores them on rides, and driver offers prefer these labels over coordinate fallback.
- Pushed app commit `5bf3fcc1 feat(rides): persist address labels`.
- Added BIRGEDrive accepted-ride navigation polish: active maneuver cue, route guidance panel, phase labels, ETA/speed/safety chips, and SF Symbol direction markers over the map.
- Pushed app commit `5fa70d6a feat(driver): add navigation guidance polish`.
- Restored full live backend Docker stack: Vapor image now builds on Swift 6.0, Kaspi HMAC uses Linux-compatible `swift-crypto`, Postgres config is compiler-friendly in release, `.dockerignore` keeps container context small, and Compose no longer exposes Postgres/Redis host ports.
- Pushed app commit `ef003825 fix(backend): restore vapor docker live stack`.
- Fixed live auth manual testing blockers: iOS decodes backend `userId`, unauthenticated 401/409 responses preserve Vapor reason messages, and driver registration now returns `409 Phone already registered` instead of a Postgres duplicate-key 500.
- Pushed app commit `f9485d50 fix(auth): align live login responses`.
- Stabilized BIRGEDrive manual QA: location tracking no longer enables background updates unless `UIBackgroundModes` actually contains `location`, preventing the simulator abort; driver registration completion now requires real profile fields so new driver accounts see personal/vehicle/documents/tier onboarding.
- Stabilized live Passenger search / BIRGEDrive offers path: WebSocket handler registration now stays on socket event loop, broadcasts send through each socket loop, driver offer decisions persist accepted/declined state, offers are limited to fresh unassigned requests, decline hides offers per driver, and Passenger search accepts canonical `driver_accepted` events.
- Verification passed: Vapor `swift build`, Vapor `swift test`, `BIRGEPassenger` build/test, `BIRGEDrive` build, Docker `compose build/up`, live OTP/ride/offers/decline/accept smoke, and raw WebSocket `101 Switching Protocols` smoke.
- Pushed app commit `eddea55d fix(driver): stabilize manual onboarding checks`.
- Fixed WebSocket connection bug: `LiveWebSocketActor` was yielding `.connected` immediately after `URLSessionWebSocketTask.resume()`, before the HTTP upgrade completed. If the handshake failed (bad JWT, 403, ride not found), the receive loop would error instantly, yielding `.disconnected` and showing the red "Нет соединения" banner to the passenger before the connection was ever truly established. Fixes: (1) `.connected` is now deferred until the first successful message is received; (2) `.disconnected` events are only yielded to UI if the connection was previously confirmed; (3) max retries increased 3→5, ping interval 5s→10s, backoff capped at 30s; (4) `SearchingFeature` now explicitly handles backend `{"type":"connected"}` and `{"type":"subscribed"}` control frames instead of trying to parse them as ride events; (5) comprehensive `os.log` debug logging added to both `LiveWebSocketActor` and `SearchingFeature` for live debugging.
- Fixed passenger WebSocket expired-token loop found in Docker logs (`JWTKit error: exp claim verification failed`). REST refresh was updating the in-memory `AccessTokenStore`, but WebSocket still read the old `birge_access_token` keychain value. `TokenRefreshClient`/`TokenRefreshTransport` now keep the legacy keychain access token in sync, and `SearchingFeature`/`RideFeature` proactively call `APIClient.refreshAccessToken()` before opening `/ws/ride/:rideId?token=...`, falling back to keychain only if refresh is unavailable.
- Verification: BIRGEPassenger build passes, BIRGEDrive build passes, all `BIRGEPassengerTests` pass (SearchingFeatureTests, RideFeatureTests, OTPFeatureTests, WebSocketClientTests, etc.).

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
- ✅ `CorridorDetailFeatureTests` cover booking status and refreshed corridor state after join.
- ✅ `MyCorridorsFeatureTests` cover booking list loading and selection navigation.
- ✅ Vapor `swift test` covers Kaspi webhook signature accept/reject behavior.
- ✅ `BIRGEDrive` builds after driver registration onboarding integration.
- ✅ `BIRGEDrive` builds after online waiting / driver offer Liquid Glass polish.
- ✅ `BIRGEDrive` builds after active ride lifecycle polish.
- ✅ `BIRGEDrive` and `BIRGEPassenger` build after driver background GPS sync work.
- ✅ Vapor `swift build` and `swift test` pass after `/locations/bulk` integration.
- ✅ Vapor `swift build`, `swift test`, and `BIRGEPassenger` build pass after location WebSocket broadcast.
- ✅ Vapor `swift build` and `swift test` pass after driver profile API integration.
- ✅ `BIRGEDrive` and `BIRGEPassenger` builds pass after Driver API iOS hookup.
- ✅ Vapor `swift build`, Vapor `swift test`, `BIRGEDrive` build, and `BIRGEPassenger` build pass after driver ride assignment/commands.
- ✅ `BIRGEDrive` and `BIRGEPassenger` builds pass after driver auth/session flow.
- ✅ Vapor `swift build`, Vapor `swift test`, `BIRGEPassenger` build, and `BIRGEDrive` build pass after ride address labels.
- ✅ `git diff --check` and `BIRGEDrive` build pass after driver navigation guidance polish.
- ✅ Vapor `swift build`, `swift build -c release`, `docker compose build vapor`, `docker compose up -d postgres redis vapor`, and local `POST /api/v1/auth/otp/request` return `200 OK` for the full live backend stack.
- ✅ Passenger OTP request/verify returns `200 OK` with `userId`; duplicate driver phone returns `409 Phone already registered`; `BIRGEPassenger` and `BIRGEDrive` builds pass after live auth response fixes.
- ✅ `BIRGEDrive` and `BIRGEPassenger` builds pass after driver location crash guard and onboarding completion gate fix.
- ✅ Vapor `swift build`/`swift test`, Passenger build/test, Drive build, Docker build/up, live ride offer decline/accept smoke, and raw WebSocket `101 Switching Protocols` pass after live ride search stability fix.
- ✅ `BIRGEPassenger` build passes, `BIRGEDrive` build passes, and all `BIRGEPassengerTests` pass after WebSocket connection bug fix (deferred `.connected`, control message handling, debug logging).
- ✅ `BIRGEPassenger` build, `BIRGEDrive` build, and full `BIRGEPassengerTests` pass after WebSocket access-token refresh synchronization.

### Next best steps
1. Manually run Passenger and Drive against the live backend and verify the full UI scenario: passenger creates ride, driver declines, another driver accepts, passenger moves to offer/ride screen.
2. Replace current driver guidance placeholders with persisted route geometry / live MapKit directions.
3. Add persisted named pickup/destination inputs beyond current fixed coordinates.
4. Later: replace demo Kaspi deep link with real merchant API contract when credentials/spec are available.

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
