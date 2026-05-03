---
sprint: 1
status: active
last_updated: 2026-05-03
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

Implementation note (2026-05-03):
- `e8a38820` — `OTPFlowE2ETests` aligned with splash-first app startup; full `BIRGEPassengerTests` now pass with `-skipMacroValidation`.
- `cf9265e3` — Package graph adjusted to unblock `SwiftNavigation` / `CasePathsCore` linker failure; `RideFeatureTests` and `OTPFeatureTests` pass with `-skipMacroValidation`.
- `51a890d7` — RideMap recovery banner polished; `RideFeature` now handles direct production lifecycle WebSocket aliases like driver arrived / ride started / ride completed.
- `2fd2a124` — Vapor `/api/v1/corridors` added, Passenger corridor screens now load/book through `APIClient`; iOS and Vapor builds pass.
- `882230a1` — P-08 OfferFound confirmation flow added and pushed; build passes, focused tests blocked by known SwiftNavigation/CasePathsCore linker issue.
- Active branch: `feature/passenger-liquid-glass-ui`
- Pushed commits: `e8a38820`, `cf9265e3`, `51a890d7`, `2fd2a124`, `882230a1`, `9a58800a`, `dcbdf02c`, `aa5e1da3`, `642f0127`, `6700e06c`, `6f074e02`, `55732eb7`, `f1150b11`, `a9d12867`
- Build verification passes for `BIRGEPassenger` on installed `iPhone 17 Pro` simulator using `-skipMacroValidation` for CLI macro approval.
- Focused `RideFeatureTests`, `OTPFeatureTests`, and `OTPFlowE2ETests` pass with `-skipMacroValidation`.
- Full `BIRGEPassengerTests` pass with `-skipMacroValidation`; live OTP E2E stays opt-in via `RUN_LIVE_OTP_E2E=1`.

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

### [IOS-018] Driver App RideFeature
- `DriverRideFeature` Reducer for BIRGEDrive target
- Driver-side FSM: accept/decline, navigation, pickup confirmation
- Background GPS tracking via `LocationClient` + `BGProcessingTask`

---

## ✅ Done

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
- Current test status: package linker blocker and splash-start OTP E2E mismatch are fixed; full `BIRGEPassengerTests` pass with `-skipMacroValidation` on `iPhone 17 Pro` simulator (2026-05-03).
