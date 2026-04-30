---
last_updated: 2026-04-22
---

# iOS Agent Context — BIRGE

> Читай этот файл перед любой iOS задачей. Здесь — всё что тебе нужно знать без поиска по архитектурным файлам.

## App Overview
Ride-hailing для Алматы. **Два отдельных таргета** в одном Xcode проекте:
- `BIRGEPassenger` — пассажирское приложение
- `BIRGEDriver` — водительское приложение

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
- Deployment target: **iOS 17+**
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
  POST /auth/refresh       { refresh_token: String } → { access_token }

Rides:
  POST /rides              { origin: LatLng, destination: LatLng, tier: Int }
  GET  /rides/:id
  POST /rides/:id/cancel   { reason: String }

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
