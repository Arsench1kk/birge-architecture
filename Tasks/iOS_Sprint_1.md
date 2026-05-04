---
sprint: 1
status: active
last_updated: 2026-05-04
---

# iOS Sprint 1 — Foundation

> Goal: работающее приложение с Auth + базовым Ride flow + офлайн GPS + XCTest coverage for OTP flow.
> Target: пассажирское приложение (BIRGEPassenger target).

---

## 🔴 In Progress

### [UI-001] Passenger Liquid Glass Mockup Implementation
- [x] Add shared Liquid Glass design components in BIRGECore
- [x] Use native `.glassEffect` on iOS 26+ with `.ultraThinMaterial` fallback
- [x] Rebuild passenger Splash + Onboarding from final mockups
- [x] Rebuild Home with AI/corridor previews and glass surfaces
- [x] Polish BoardingCode and RideComplete screens
- [x] Replace visible ride-flow emoji/stickers with SF Symbols
- [x] Build `CorridorListFeature/View` using mock corridor data
- [x] Build `CorridorDetailFeature/View` using mock corridor data
- [x] Wire Home → Corridor list/detail navigation through `PassengerAppFeature`
- [x] Build P-08 `OfferFoundView`
- [x] Implement Vapor `/corridors` API and connect corridor screens to real data
- [x] Add RideMap disconnection banner and remaining production ride events
- [x] Build P-23 `AIExplanationFeature/View` and connect Home AI pill navigation
- [x] Build P-03a–P-03e commute setup inside passenger onboarding
- [x] Build P-17/P-18 subscriptions flow and connect Home subscription navigation
- [x] Implement Vapor `/subscriptions` API and connect subscription screen to live plans/activation
- [x] Add Kaspi subscription checkout handoff and payment event ledger skeleton
- [x] Persist passenger corridor bookings and make repeat booking idempotent
- [x] Show passenger corridor booking status/id in CorridorDetail after join
- [x] Add passenger My Corridors list backed by `/corridors/bookings`
- [x] Harden Kaspi webhook signature validation with HMAC-SHA256
- [x] Polish D-05 online waiting and driver offer surfaces in `DriverAppView`
- [x] Polish D-07/D-08/D-09/D-10 driver active ride lifecycle and completion surfaces
- [x] Wire BIRGEDrive active rides to `LocationClient` tracking/sync
- [x] Implement Vapor `POST /api/v1/locations/bulk` and `driver_location_records` persistence
- [x] Broadcast latest driver GPS batch point as `ride.location_update` over WebSocket
- [x] Add Vapor Driver profile API for registration persistence and today corridors
- [x] Connect BIRGEDrive registration/dashboard to Driver profile and today corridors API
- [x] Add backend driver ride offers plus accept/arrived/start/complete commands and wire BIRGEDrive to them
- [x] Add BIRGEDrive email login/register session flow and remove no-token demo fallback
- [x] Persist passenger ride address labels and show them in driver offers
- [x] Add driver-side navigation guidance polish for accepted rides
- [x] Restore Docker Compose live backend stack for Simulator testing
- [x] Fix live Passenger OTP decode and Driver duplicate-phone registration errors
- [x] Fix BIRGEDrive simulator location crash and new-driver onboarding skip
- [x] Stabilize live ride search, WebSocket connection, stale driver offers, and persisted driver decline behavior
- [x] Fix Passenger WebSocket expired-token loop during live ride search

Implementation note (2026-05-04):
- Passenger WebSocket auth stability — Docker logs showed `/ws/ride/:rideId` failing with expired JWT even after REST calls refreshed successfully. The refresh path now syncs the legacy keychain access token used by WebSocket, and `SearchingFeature`/`RideFeature` proactively refresh before opening ride sockets, with keychain fallback. Verified with Passenger build/test and Drive build.
- Live ride search stability — backend now persists driver ride decisions, filters offers to fresh unassigned requests with a 15-minute TTL, adds `POST /rides/:rideID/driver/decline`, keeps WebSocket registration/broadcast on socket event loops, BIRGEDrive decline calls the backend, and Passenger search accepts canonical `driver_accepted` events. Verified with Vapor build/test, Passenger build/test, Drive build, Docker build/up, live offer decline/accept smoke, and raw WebSocket 101 smoke.
- `eddea55d` — BIRGEDrive manual QA is stabilized: location manager only enables background updates when runtime Info.plist has `UIBackgroundModes=location`, and registration completion now depends on filled core profile fields instead of the initial `pending` KYC status.
- `f9485d50` — Live auth manual testing is unblocked: `APIAuthResponse` accepts backend `userId`, Vapor `reason` errors surface in iOS, unauthenticated 401/409 responses keep backend messages, and driver registration checks phone uniqueness before insert.
- `ef003825` — Docker live backend stack is restored: Dockerfile uses Swift 6.0, Kaspi HMAC imports `swift-crypto` instead of macOS-only `CryptoKit`, Postgres configuration compiles in Linux release, `.dockerignore` prevents copying `.build`, and Compose keeps Postgres/Redis internal to avoid local port conflicts.

Implementation note (2026-05-03):
- `5fa70d6a` — BIRGEDrive active rides now show a Liquid Glass maneuver cue, navigation guidance card, ETA/speed/safety chips, route phase labels, and SF Symbol direction markers; real MapKit route geometry remains the next routing step.
- `5bf3fcc1` — Ride requests now carry origin/destination names, Vapor persists `origin_name`/`destination_name`, and driver offers display stored labels instead of coordinate-only fallback; backend test covers the offer DTO.
- `29903977` — BIRGEDrive now has email login/register for driver accounts, uses shared API token storage for live Driver API calls, gates dashboard/registration behind auth, and no longer completes registration/offers through no-token fallback.
- `a12d75b6` — Added driver-facing ride assignment endpoints (`GET /rides/driver/offers`, accept, arrived, start, complete), canonical `ride.status_changed` broadcasts, APIClient methods, and BIRGEDrive polling/accept/command wiring with no-token demo fallback.
- `be190fa7` — BIRGEDrive now loads driver profile/today corridors on startup, saves registration via `APIClient.updateDriverProfile`, shows live profile/vehicle/corridor state on the dashboard, and keeps a no-token demo fallback until driver auth is added.
- `c5012c77` — Added authenticated Driver API: `/api/v1/drivers/me` GET/PUT/POST persists registration/profile fields, auto-creates draft profiles for driver users, and `/api/v1/drivers/corridors/today` returns active corridor candidates plus estimated earnings.
- `85a66398` — `/locations/bulk` now broadcasts the newest GPS point in each batch as canonical `ride.location_update` to `ride/<ride_id>` WebSocket subscribers; backend tests cover JSON contract.
- `e778858b` — Driver active rides now start offline-first GPS tracking, stop/sync on completion/offline, BIRGEDrive declares background location usage, and Vapor accepts authenticated location batches into `driver_location_records`.
- `079ac13a` — Driver active ride lifecycle now has pickup, arrived/boarding, in-progress, and completion surfaces with Liquid Glass route cards, passenger manifest/codes, progress state, and next-ride summary actions.
- `136f91a0` — Driver online waiting and offer surfaces now use a map-style background, Liquid Glass top/bottom overlays, SF Symbols, offer alert, match pill, route card, metrics grid, and passenger confirmation row.
- `d83fec67` — BIRGEDrive now launches through a 4-step driver registration onboarding flow before dashboard: personal data, vehicle, documents, and tier selection, using Liquid Glass surfaces and SF Symbols.
- `98948bf4` — Kaspi webhook handling now validates HMAC-SHA256 signatures when `KASPI_WEBHOOK_SECRET` is configured, supports signature headers, and has backend unit coverage.
- `061d6aaf` — Added authenticated `/api/v1/corridors/bookings`, `APIClient.fetchCorridorBookings`, `MyCorridorsFeature/View`, Home `Поездки` navigation, and reducer/navigation tests.
- `17d44560` — `CorridorDetailFeature/View` now stores booking id/status, refreshes corridor seats/participants from `CorridorBookingResponse`, disables repeat join, and has reducer coverage.
- `ab5237fe` — Vapor corridor bookings now persist passenger/corridor records with uniqueness; repeat booking returns the existing booking instead of decrementing seats again.
- `01955808` — Vapor Payments module added with Kaspi checkout deep link, append-only `payment_events`, idempotent webhook event handling, iOS `createKaspiCheckout`, and subscription UI handoff before demo activation.
- `50c3915f` — Vapor `/api/v1/subscriptions` added with persisted passenger plan state; iOS `APIClient` and `SubscriptionsFeature/View` now load plans and activate subscriptions through the API.
- `9f06a2a4` — P-17/P-18 subscriptions flow added with Liquid Glass tier cards, plan detail, local activation state, Home navigation, and `SubscriptionsFeatureTests`.
- `fe208327` — Onboarding now continues into commute setup for origin, destination, morning/evening times, weekdays, and AI summary; `OnboardingFeatureTests` added.
- `da9001cf` — P-23 AI explanation screen added with Liquid Glass cards, SF Symbols, Home AI pill navigation, corridor-list CTA, and `PassengerAppFeatureTests` coverage.
- `e8a38820` — `OTPFlowE2ETests` aligned with splash-first app startup; full `BIRGEPassengerTests` now pass with `-skipMacroValidation`.
- `cf9265e3` — Package graph adjusted to unblock `SwiftNavigation` / `CasePathsCore` linker failure; `RideFeatureTests` and `OTPFeatureTests` pass with `-skipMacroValidation`.
- `51a890d7` — RideMap recovery banner polished; `RideFeature` now handles direct production lifecycle WebSocket aliases like driver arrived / ride started / ride completed.
- `2fd2a124` — Vapor `/api/v1/corridors` added, Passenger corridor screens now load/book through `APIClient`; iOS and Vapor builds pass.
- `882230a1` — P-08 OfferFound confirmation flow added and pushed; build passes, focused tests blocked by known SwiftNavigation/CasePathsCore linker issue.
- Active branch: `feature/passenger-liquid-glass-ui`
- Pushed commits: `eddea55d`, `f9485d50`, `ef003825`, `5fa70d6a`, `5bf3fcc1`, `29903977`, `a12d75b6`, `be190fa7`, `c5012c77`, `85a66398`, `e778858b`, `079ac13a`, `136f91a0`, `d83fec67`, `98948bf4`, `061d6aaf`, `17d44560`, `ab5237fe`, `01955808`, `50c3915f`, `9f06a2a4`, `fe208327`, `da9001cf`, `e8a38820`, `cf9265e3`, `51a890d7`, `2fd2a124`, `882230a1`, `9a58800a`, `dcbdf02c`, `aa5e1da3`, `642f0127`, `6700e06c`, `6f074e02`, `55732eb7`, `f1150b11`, `a9d12867`
- Build verification passes for `BIRGEPassenger` on installed `iPhone 17 Pro` simulator using `-skipMacroValidation` for CLI macro approval.
- Focused `RideFeatureTests`, `OTPFeatureTests`, and `OTPFlowE2ETests` pass with `-skipMacroValidation`.
- Full `BIRGEPassengerTests` pass with `-skipMacroValidation`; live OTP E2E stays opt-in via `RUN_LIVE_OTP_E2E=1`.
- `PassengerAppFeatureTests` cover the new AI explanation navigation path.
- `OnboardingFeatureTests` cover the new commute setup reducer path.
- `SubscriptionsFeatureTests` cover API loading, plan selection, activation, and detail dismissal.
- Vapor `swift build` passes after subscriptions API integration.
- Vapor `swift build` passes after Payments/Kaspi handoff integration.
- Vapor `swift build` passes after corridor booking persistence.
- Full live backend verification passes: `swift build`, `swift build -c release`, `docker compose build vapor`, `docker compose up -d postgres redis vapor`, and local OTP request returns `200 OK` on `localhost:8080`.
- Live auth verification passes: OTP request/verify returns `200 OK` with `userId`, duplicate driver phone returns `409 Phone already registered`, and both Passenger/Drive builds pass.
- Driver manual QA verification passes for build: location background guard compiles and new-driver onboarding gate is corrected.
- Live ride search verification passes: Docker live stack runs, fresh offers appear, persisted decline hides the offer for the same driver, another driver can accept, accepted rides leave the offer list, and WebSocket upgrade no longer crashes Vapor.
- Passenger WebSocket token verification passes: `BIRGEPassenger` build/test and `BIRGEDrive` build pass after syncing refreshed access tokens to the WebSocket keychain path.

### [IOS-017] API Client + Token Refresh
- [x] `APIClient` TCA dependency: authenticated URLSession wrapper
- [x] `TokenRefreshClient` TCA dependency with in-memory access token + Keychain refresh token storage
- [x] 401 handling: refresh access token once, retry original request, clear tokens if retry remains unauthorized
- [x] Replace stub `APIClient.liveValue` with real URLSession implementation
- [x] Wire `POST /locations/bulk` to `LocationSyncService`
- [x] `BIRGEAPIError` decodes backend `{ error_code, message, request_id }`
- [x] Build verification: `xcodebuild build -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 17 Pro' -quiet` passes (2026-05-02)
- [ ] Timer-based proactive refresh 60 seconds before JWT expiry
- Wire WebSocket `Authorization: Bearer` header
- [x] First test unblocker: fixed current `swift-navigation` / `CasePathsCore` linker failure so focused iOS tests compile and run

Implementation note (2026-05-02):
- Added `BIRGECore/Sources/BIRGECore/Network/APIClient.swift` live methods for:
  - `POST /auth/otp/request`
  - `POST /auth/otp/verify`
  - `POST /auth/refresh`
  - `GET /auth/me`
  - `POST /rides`
  - `GET /rides/:id`
  - `PATCH /rides/:id/cancel`
  - `POST /locations/bulk`
- Added `BIRGECore/Sources/BIRGECore/Network/TokenRefreshClient.swift`.
- `AuthClient.liveValue` now delegates OTP/current-user calls to `APIClient.liveValue` so OTP login seeds the in-memory access token.
- `xcodebuild test -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 17 Pro'` still fails before tests run because `SwiftNavigation.framework` cannot link missing `CasePathsCore.CasePathable` / `CasePathsCore.AnyCasePath` symbols.

**Architecture ref:** [[Architecture/iOS_Architecture]] Section 4, [[Context/iOS_Agent_Context]] JWT Strategy

---

## 🟡 Ready to Start


### [IOS-FUTURE-001] Per-user onboarding routing after OTP
- Status: PLANNED
- Ref: [[Future_Demo_Stability_and_Onboarding]]
- Goal: after OTP, route fresh passenger/driver accounts into the correct onboarding based on backend user state; completed users skip onboarding.
- Demo priority: show passenger commute answers and driver profile setup as real data, not shared local-only flags.

### [IOS-018] Driver App RideFeature
- [x] Driver registration onboarding D-03 personal/vehicle/documents/tier flow
- [x] Driver D-05 online waiting and offer UI polish in `DriverAppView`
- [x] Driver D-07/D-08/D-09/D-10 lifecycle UI polish and completed ride summary
- [x] Driver background GPS tracking via `LocationClient` + `/locations/bulk`
- [x] Live driver GPS WebSocket broadcast compatible with passenger `RideFeature`
- [x] Backend Driver profile/today corridors API
- [x] BIRGEDrive Driver API hookup for registration/profile/today corridors
- [x] BIRGEDrive driver auth/session gate
- `DriverRideFeature` Reducer for BIRGEDrive target
- [x] Driver-side FSM backend commands: accept, pickup arrival, ride start, completion
- [x] Driver-side navigation guidance polish for accepted rides
- Live MapKit directions/route geometry for driver navigation
- [x] Background GPS tracking via `LocationClient` + `/locations/bulk` sync

---

## ✅ Done

- [x] In-App Project Demo screen for defense: `ProjectDemoFeature/View`, Home debug entry point, live Demo API state/seed/reset integration, and reducer tests (2026-05-04)
- [x] Demo stability fixes: configurable API/WebSocket base URL, RideFeature ignores backend control frames, OfferFound decline/expiry calls backend cancel, BIRGEDrive uploads live GPS every 5 seconds while tracking (2026-05-04)

- [x] IOS-001: Xcode Project Setup — `BIRGEPassenger` + `BIRGEDrive` targets, shared workspace, SPM wiring (2026-04-22)
- [x] IOS-002: GRDB Setup in `BIRGECore` — `DatabaseManager`, `LocationRecord`, `LocationRepository`, in-memory tests (2026-05-02)
- [x] IOS-003: OTP Auth screen — TCA Reducer + View + Mock flow (2026-04-23, updated 2026-04-27)
- [x] IOS-004: AppFeature Root Navigator — Root view switcher (2026-04-27)
- [x] IOS-005: HomeView — MapKit + bottom sheet + corridor cards (2026-04-23)
- [x] IOS-006: RideRequestView — tier selector + fare + address input (2026-04-23)
- [x] IOS-007: SearchingDriverView — radar animation + auto-transition (2026-04-23)
- [x] IOS-008: ActiveRideView — live simulation + status states (2026-04-23)
- [x] IOS-009: RideCompleteView — rating + summary + tags (2026-04-23)
- [x] IOS-010: Driver target — `BIRGEDrive`, `DriverAppFeature`, `DriverAppView`, `EarningsFeature` (2026-04-27)
- [x] IOS-011: OTP Auth + AppFeature root navigator (2026-04-27)
- [x] IOS-012: User Profile screen + logout (2026-04-27)
- [x] IOS-013: OTP XCTest coverage — `BIRGEPassengerTests`, `BIRGEPassenger.xctestplan`, mocked reducer tests, live OTP E2E gated by `RUN_LIVE_OTP_E2E` (2026-05-02)
- [x] IOS-014: WebSocketClient TCA Dependency — `WebSocketClient` struct, `LiveWebSocketActor`, `DependencyValues` extension, `WebSocketClientTests` (2026-05-02)
- [x] IOS-015: LocationClient TCA Dependency — `LocationClient` struct, `LiveLocationActor`, `LocationSyncService`, `LocationClientTests` (2026-05-02)
- [x] IOS-016: RideFeature State Machine — `RideFeature` 7-state Reducer, `RideMapView`, `Coordinate` wrapper, `RideEvent` models, stub `APIClient`, `RideFeatureTests` with 7 reducer scenarios (2026-05-02)

---

## Notes
- Начинать с IOS-001 → IOS-002 → IOS-003 (строго по порядку, зависимости)
- IOS-004 зависит от IOS-003
- IOS-005 и IOS-006 можно параллельно после IOS-002
- IOS-007 зависит от IOS-005
- Live OTP E2E ожидает local Vapor + PostgreSQL + Redis + `/tmp/birge-otp.log`
- Обычный `xcodebuild test -scheme BIRGEPassenger` должен проходить без live backend; live success test теперь skipped by default
- IOS-016 verification attempted on installed `iPhone 17 Pro` simulator because `iPhone 16 Pro` is not available locally
- Build verification now passes for both `BIRGEPassenger` and `BIRGEDrive` on `iPhone 17 Pro`
- Current test status: package linker blocker and splash-start OTP E2E mismatch are fixed; full `BIRGEPassengerTests` pass with `-skipMacroValidation` on `iPhone 17 Pro` simulator (2026-05-03), including the subscription Kaspi checkout handoff reducer path; corridor booking persistence is backend-verified with `swift build`; `CorridorDetailFeatureTests` cover visible booking status after join; `MyCorridorsFeatureTests` cover passenger booking history loading and selection; Vapor `swift test` covers Kaspi signature validation, ride status broadcast JSON, and driver offer address labels; `BIRGEDrive` build passes after driver registration onboarding, online/offer surface polish, active ride lifecycle polish, background GPS sync wiring, Driver API hookup, driver ride command wiring, driver auth/session gate, and ride address labels; `BIRGEPassenger` build and Vapor `swift test` also pass after `/locations/bulk`, location WebSocket broadcast, driver profile API integration, driver assignment commands, and address labels.
