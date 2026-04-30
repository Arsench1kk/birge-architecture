---
sprint: 1
status: active
last_updated: 2026-04-27
---

# iOS Sprint 1 — Foundation

> Goal: работающее приложение с Auth + базовым Ride flow + офлайн GPS.
> Target: пассажирское приложение (BIRGEPassenger target).

---

## 🔴 In Progress

*(пусто — все текущие задачи завершены)*

---

## 🟡 Ready to Start

### [IOS-001] Xcode Project Setup
- Создать проект: `BIRGEPassenger` + `BIRGEDriver` targets
- Добавить SPM зависимости:
  - `https://github.com/pointfreeco/swift-composable-architecture` (1.x)
  - `https://github.com/groue/GRDB.swift` (6.x)
- Настроить Info.plist: `NSLocationWhenInUseUsageDescription`, `NSLocationAlwaysUsageDescription`
- Background modes: `location`, `fetch`, `processing`
- Создать `BIRGE.code-workspace` (multi-root: iOS + Obsidian vault)

**Antigravity prompt prefix:** `Read RULES.md and Context/iOS_Agent_Context.md first.`

---

### [IOS-002] GRDB Setup
- `DatabaseManager.swift` — singleton, WAL mode, background queue
- `LocationRecord` модель (id, ride_id, lat, lng, timestamp, accuracy, synced)
- Миграции через `DatabaseMigrator`
- `LocationRepository.swift` — `insert(_:)`, `fetchUnsynced() -> [LocationRecord]`, `markSynced(_:)`
- Unit тест: `LocationRepositoryTests` с in-memory DB

**Architecture ref:** [[Architecture/iOS_Architecture]] Section 3

---


---


---

### [IOS-005] WebSocketClient TCA Dependency
- `WebSocketClient` struct: `connect(URL) -> AsyncStream<WebSocketEvent>`, `send(_:)`, `disconnect()`
- Live реализация через `URLSessionWebSocketTask`
- Ping каждые 5 секунд
- Exponential backoff reconnect: 1→2→4→8→16→30s
- Mock реализация для тестов и Previews
- Unit тест: `WebSocketClientTests`

**Architecture ref:** [[Architecture/iOS_Architecture]] Section 4, [[Architecture/WebSocket_Hub_Architecture]]

---

### [IOS-006] LocationService TCA Dependency
- `LocationClient` dependency: `locationStream() -> AsyncStream<CLLocation>`
- `Effect.run` обёртка над `CLLocationManager`
- При потере сети → пишет в GRDB через `LocationRepository`
- При восстановлении → bulk sync (fetchUnsynced → POST /locations/bulk)
- `.cancellable(id: LocationTrackingID.self)`

**Architecture ref:** [[Architecture/iOS_Architecture]] Section 5

---

### [IOS-007] RideFeature State Machine
- `RideFeature` Reducer с 7 состояниями из [[Architecture/Ride_State_Machine]]
- WebSocket события → Actions → State transitions
- `RideMapView` — отображение карты (MapKit)
- ETA обновление каждые 10 секунд

---

## ✅ Done

- [x] IOS-003: OTP Auth screen — TCA Reducer + View + Mock flow (2026-04-23, updated 2026-04-27)
- [x] IOS-004: AppFeature Root Navigator — Root view switcher (2026-04-27)
- [x] IOS-005: HomeView — MapKit + bottom sheet + corridor cards (2026-04-23)
- [x] IOS-006: RideRequestView — tier selector + fare + address input (2026-04-23)
- [x] IOS-007: SearchingDriverView — radar animation + auto-transition (2026-04-23)
- [x] IOS-008: ActiveRideView — live simulation + status states (2026-04-23)
- [x] IOS-009: RideCompleteView — rating + summary + tags (2026-04-23)
- [x] IOS-010: Driver Dashboard target — DriverAppFeature + DriverAppView (2026-04-27)
- [x] IOS-011: OTP Auth + AppFeature root navigator (2026-04-27)
- [x] IOS-012: User Profile screen + logout (2026-04-27)

---

## Notes
- Начинать с IOS-001 → IOS-002 → IOS-003 (строго по порядку, зависимости)
- IOS-004 зависит от IOS-003
- IOS-005 и IOS-006 можно параллельно после IOS-002
- IOS-007 зависит от IOS-005
