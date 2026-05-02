---
sprint: 1
status: active
last_updated: 2026-05-02
---

# iOS Sprint 1 — Foundation

> Goal: работающее приложение с Auth + базовым Ride flow + офлайн GPS + XCTest coverage for OTP flow.
> Target: пассажирское приложение (BIRGEPassenger target).

---

## 🔴 In Progress

### [IOS-016] RideFeature State Machine
- `RideFeature` Reducer с 7 состояниями из [[Architecture/Ride_State_Machine]]
- WebSocket события → Actions → State transitions
- `RideMapView` — отображение карты (MapKit)
- ETA обновление каждые 10 секунд

**Architecture ref:** [[Architecture/Ride_State_Machine]], [[Architecture/iOS_Architecture]] Section 2

---

## 🟡 Ready to Start

### [IOS-017] API Client + Token Refresh
- `APIClient` TCA dependency: authenticated URLSession wrapper
- `TokenRefreshClient` — автоматический refresh за 60 секунд до expiry
- Wire `POST /locations/bulk` to `LocationSyncService`
- Wire WebSocket `Authorization: Bearer` header

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

---

## Notes
- Начинать с IOS-001 → IOS-002 → IOS-003 (строго по порядку, зависимости)
- IOS-004 зависит от IOS-003
- IOS-005 и IOS-006 можно параллельно после IOS-002
- IOS-007 зависит от IOS-005
- Live OTP E2E ожидает local Vapor + PostgreSQL + Redis + `/tmp/birge-otp.log`
- Обычный `xcodebuild test -scheme BIRGEPassenger` должен проходить без live backend; live success test теперь skipped by default
