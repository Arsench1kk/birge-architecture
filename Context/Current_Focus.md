---
last_updated: 2026-05-02
sprint: 1
---

# Current Focus — BIRGE

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

### Next session
- Task: IOS-016 — RideFeature State Machine
- `RideFeature` Reducer с 7 состояниями
- WebSocket события → Actions → State transitions
- Wire `LocationClient` + `WebSocketClient` into ride lifecycle
- Blocked by: nothing (IOS-015 complete)
- Model: Claude Opus 4.6 (Thinking)


- Vault настроен (v5.1)
- BIRGE.code-workspace создан
- RULES.md в корне проекта
