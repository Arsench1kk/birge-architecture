---
last_updated: 2026-05-09
---

# iOS Agent Context — BIRGE

> Читай этот файл перед любой iOS задачей. Здесь — всё что тебе нужно знать без поиска по архитектурным файлам.

## App Overview
BIRGE is now defined as **transportation-as-a-subscription** for recurring Almaty commuters, not taxi-first ride-hailing. Tier 3 subscription commute is the product wedge; on-demand is fallback/acquisition only.

Before major UI/iOS product work, read [[Business_Department/Product_Logic]] and [[Tasks/Business_Sprint_0_Foundation]]. Existing ride/corridor screens may remain for demo stability, but new implementation should not deepen taxi-first assumptions.

Technical pivot plan: [[Tasks/Technical_Subscription_Pivot]].

Frontend-only SwiftUI transfer plan: [[Tasks/Swift_Frontend_Mockup_Implementation_Plan]].

Current mockup source of truth:

```text
Passenger final: /Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/passenger-polished
Driver final:    /Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/driver
```

Implementation lane rule:

```text
Until backend resumes, build frontend-only native SwiftUI/TCA flows with mock dependencies and centralized fixtures.
Do not hardcode product data inside SwiftUI views.
Do not wire real payments, real document upload, or new backend endpoints in this lane.
```

Root routing rule:

```text
Do not route only by access-token existence.
Passenger: new phone -> registration; known complete phone -> main app; known incomplete phone -> resume setup.
Driver: new phone -> onboarding; pending verification -> status; approved active corridor -> home.
```

Passenger subscription tier IDs:

```text
solo_corridor
multi_corridor
flex_pack
```

Current app repo status from Claude Code audit:
- `TODO(subscription-pivot)` markers exist in the critical subscription/home/backend files.
- Driver tier pricing was changed to 19,000 / 28,000 / 38,000 KZT.
- Business direction after that adds Peak Starter at 9,900 KZT/month, so driver tier implementation is not final yet.
- Review/commit these changes intentionally before new implementation.

**Два отдельных таргета** в одном Xcode проекте:
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
  ws://localhost:8080/ws/ride/:rideId?token=<jwt>  (local dev)
  wss://api.birge.kz/ws/ride/:rideId?token=<jwt>   (production shape)
  Future hardening: Authorization: Bearer <token> header instead of query token
  Events: connected/subscribed control frames, driver_accepted, ride.status_changed,
          ride.location_update, ride.eta_updated, direct lifecycle aliases
```

## JWT Strategy
```swift
// Access token: 60 минут — в памяти (не персистентно)
// Refresh token: 30 дней — в Keychain
// При REST 401 — TokenRefreshClient refreshes once and retries
// Before ride WebSocket — SearchingFeature/RideFeature proactively call refreshAccessToken()
// При 401 — TokenRefreshClient пробует refresh, при ошибке → logout
```

Current iOS implementation note (2026-05-05):
- `APIClient.liveValue` is URLSession-backed for Phase 1 auth, ride, and location endpoints.
- `TokenRefreshClient` keeps the access token in memory and stores the refresh token in Keychain.
- 401 retry is implemented: refresh once, retry original request, clear tokens if still unauthorized.
- Token refresh now also keeps the legacy keychain access-token path in sync because WebSocket currently uses a query token.
- Timer-based refresh 60 seconds before JWT expiry is still pending because the backend response does not expose an expiry timestamp to iOS.
- WebSocket `Authorization: Bearer <token>` header is still pending; query token is the current implementation.

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
3. JWT refresh happens on REST 401 and before opening ride WebSockets; timer-based pre-expiry refresh is pending
4. Все платёжные операции идемпотентны (Kaspi callback)
5. Водитель не видит адрес пассажира до начала поездки (privacy)

## Files to read for deep dive
- [[Architecture/iOS_Architecture]] — TCA patterns, GRDB setup детально
- [[Architecture/Ride_State_Machine]] — полный FSM
- [[Architecture/WebSocket_Hub_Architecture]] — как backend организует WS
- [[Architecture/Security_and_Authentication]] — OTP/JWT flow детально
- [[Architecture/Open_Questions]] — проверь не заблокирована ли задача
