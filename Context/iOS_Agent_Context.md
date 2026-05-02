---
last_updated: 2026-05-02
---

# iOS Agent Context — BIRGE

> Читай этот файл перед любой iOS задачей. Здесь — всё что тебе нужно знать без поиска по архитектурным файлам.

## App Overview
Ride-hailing для Алматы. **Два отдельных таргета** в одном Xcode проекте:
- `BIRGEPassenger` — пассажирское приложение
- `BIRGEDrive` — водительское приложение

Общий код вынесен в `BIRGECore` локальный Swift Package (внутри проекта).

## Stack
| Component | Technology | Why |
|---|---|---|
| UI | SwiftUI | Declarative, поддерживается Apple, лучшая интеграция с TCA |
| State | TCA (The Composable Architecture) | Единственный вариант для async state + WebSocket + GPS |
| Local cache | GRDB (WAL mode) | Thread-safe фоновые вставки без @MainActor блокировок |
| Network | URLSession async/await | Нет лишних зависимостей |
| WebSocket | URLSessionWebSocketTask | Native, не требует библиотек |
| Auth | Keychain (Security framework) | JWT в hardware-protected enclave |
| Location | CLLocationManager + BGProcessingTask | Background режим для водителей |

## iOS Target
- Architectural floor: **iOS 17+**
- Current `project.pbxproj` setting: `IPHONEOS_DEPLOYMENT_TARGET = 26.4` on the active Xcode 26.4 toolchain
- Language: **Swift 6** (strict concurrency)
- Minimum device: iPhone 12 (A14 Bionic)

## TCA Rules
```
// Каждая фича — это:
struct FeatureName: Reducer {
    @ObservableState struct State: Equatable { ... }
    enum Action: Sendable { ... }
    @Dependency(\.someClient) var someClient
    var body: some ReducerOf<Self> { ... }
}
```
- Все зависимости через `@Dependency` — никогда напрямую
- Все side effects через `Effect.run`
- Lifecycle через `.cancellable(id:)` — явная отмена
- Actions = past tense глаголы: `.locationUpdated`, `.rideStarted`

## Ride State Machine (iOS mirror)
```
requested → matched → driver_accepted → driver_arriving
         → passenger_wait → in_progress → completed
Any state → cancelled(reason:)
```
Полные определения: [[Architecture/Ride_State_Machine]]

## API Endpoints (Phase 1)
```
Base URL: https://api.birge.kz/api/v1  (production)
          http://localhost:8080/api/v1  (local dev)

Auth:
  POST /auth/otp/request   { phone: String }
  POST /auth/otp/verify    { phone: String, code: String } → { access_token, refresh_token }
  GET  /auth/me            → current user
  POST /auth/refresh       { refresh_token: String } → { access_token }

Rides:
  POST /rides              { origin: LatLng, destination: LatLng, tier: Int }
  GET  /rides/:id
  PATCH /rides/:id/cancel  { reason: String }

WebSocket:
  wss://api.birge.kz/ws   (Authorization: Bearer <token> в хедере)
  Ping: каждые 5 секунд
  Events: ride_matched, driver_location, ride_status_changed, eta_updated
```

## JWT Strategy
```swift
// Access token: 60 минут — в памяти (не персистентно)
// Refresh token: 30 дней — в Keychain
// Refresh за 60 секунд до expiry — TokenRefreshClient делает это автоматически
// При 401 — TokenRefreshClient пробует refresh, при ошибке → logout
```

Current iOS implementation note (2026-05-02):
- `APIClient.liveValue` is URLSession-backed for Phase 1 auth, ride, and location endpoints.
- `TokenRefreshClient` keeps the access token in memory and stores the refresh token in Keychain.
- 401 retry is implemented: refresh once, retry original request, clear tokens if still unauthorized.
- Proactive timer refresh 60 seconds before JWT expiry is not implemented yet because the current backend response does not expose an expiry timestamp to iOS.
- WebSocket `Authorization: Bearer <token>` header is still pending.

## Testing
- XCTest target: `BIRGEPassengerTests`
- Test plan: `BIRGEPassenger.xctestplan`
- Default command:
```
xcodebuild test -scheme BIRGEPassenger \
  -destination 'platform=iOS Simulator,name=iPhone 17 Pro'
```
- `OTPFeatureTests` use mocked/test dependencies and are expected to pass in regular CI/local runs
- `OTPFlowE2ETests.testOTPFlowSuccess` is **opt-in only**:
```
RUN_LIVE_OTP_E2E=1 xcodebuild test -scheme BIRGEPassenger \
  -destination 'platform=iOS Simulator,name=iPhone 17 Pro' \
  -only-testing:BIRGEPassengerTests/OTPFlowE2ETests/testOTPFlowSuccess
```
- Live E2E prerequisites:
  - local Vapor backend on `http://localhost:8080/api/v1`
  - PostgreSQL + Redis available to Vapor
  - OTP log writer active at `/tmp/birge-otp.log`

## GRDB Schema (GPS Cache)
```sql
CREATE TABLE location_records (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    ride_id TEXT NOT NULL,
    latitude REAL NOT NULL,
    longitude REAL NOT NULL,
    timestamp REAL NOT NULL,
    accuracy REAL,
    synced INTEGER NOT NULL DEFAULT 0  -- Bool
);
```
Bulk insert: 100–200 записей за раз. Background queue всегда.

## WebSocket Reconnect
```
Попытка 1: 1 сек
Попытка 2: 2 сек
Попытка 3: 4 сек
...max 30 сек
После 5 неудачных попыток: показать баннер "Нет соединения"
```

## Critical Business Rules
1. GPS координаты ВСЕГДА пишутся в GRDB даже без сети
2. WebSocket переподключается автоматически после восстановления сети
3. JWT refresh происходит за 60 секунд до истечения
4. Все платёжные операции идемпотентны (Kaspi callback)
5. Водитель не видит адрес пассажира до начала поездки (privacy)

## Files to read for deep dive
- [[Architecture/iOS_Architecture]] — TCA patterns, GRDB setup детально
- [[Architecture/Ride_State_Machine]] — полный FSM
- [[Architecture/WebSocket_Hub_Architecture]] — как backend организует WS
- [[Architecture/Security_and_Authentication]] — OTP/JWT flow детально
- [[Architecture/Open_Questions]] — проверь не заблокирована ли задача
