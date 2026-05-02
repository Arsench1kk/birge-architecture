---
status: active
version: v5.0
last_updated: 2026-05-02
phase: 1
---

# [[BIRGE iOS Native Architecture]]

## 1. Why Native — No Cross-Platform

The mobile client is strictly native iOS (Swift + SwiftUI). All legacy cross-platform references are deprecated. **Flutter is forbidden.**

Reasons specific to BIRGE:
- Background location tracking reliability — `CLLocationManager` + `BGProcessingTask` require native APIs
- GRDB WAL-mode background bulk inserts — critical for GPS accumulation during tunnel/elevator connectivity drops in Almaty
- WebSocket persistent connection lifecycle — managed cleanly with `URLSessionWebSocketTask`
- TCA's `@MainActor`-safe state management — no bridge overhead

---

## 2. State Management — [[The Composable Architecture]] (TCA)

### Why Not MVVM

BIRGE's iOS app manages complex, simultaneous async state:
- Driver navigating in real-time (GPS stream)
- Passenger receiving live ETA updates via WebSocket
- Background location tracking during active rides
- Network drops and automated reconnect sequences
- Shared state across multiple concurrent screens

Standard MVVM was rejected because managing this across dozens of ViewModels leads to "ViewModel Hell" — untrackable state, race conditions between async updates.

### TCA Architecture

```
User action / WebSocket event / GPS update
        │
        ▼
Action dispatched to Store
        │
        ▼
Reducer: pure function (State, Action) → (State, Effect)
        │
        ├── State updated (single source of truth)
        └── Effect returned (async work, side effects)
                │
                ▼
        Effect executes (network, GPS, WebSocket)
                │
                ▼
        Action dispatched back to Store
```

**Key TCA patterns in BIRGE:**

| Pattern | Usage |
|---|---|
| `Effect.run` | Continuous background GPS tracking — wraps `CLLocationManager` stream, never blocks main thread |
| `DependencyValues` | `WebSocketClient` and `LocationService` registered as dependencies — safely mocked in unit tests and SwiftUI Previews without changing production code |
| `.cancellable(id:)` | Explicit lifecycle management for long-lived effects — GPS tracking cancelled when ride ends |
| Single state tree | All app state in one predictable tree — ride status, driver location, passenger ETA all consistent |

---

## 3. Offline Cache — [[GRDB]] (not SwiftData)

### Why GRDB Replaced SwiftData (v5.0 decision)

Almaty's urban topology causes frequent connectivity drops — tunnels on major routes, underground parking, elevator shafts in residential high-rises. A driver losing cellular must continue accumulating GPS coordinates.

| Factor | SwiftData | GRDB |
|---|---|---|
| Background bulk inserts | `@MainActor`-bound — blocks UI thread | Thread-safe with explicit WAL mode control |
| Write-Ahead Logging | Implicit, not configurable | Explicit WAL mode — safe concurrent reads/writes |
| Background sync | Slow, UI-blocking | Background `DatabaseQueue` — 100-200 records bulk-inserted without main thread contact |
| Testing | Difficult to mock | Full in-memory database support |

### Network Drop → Reconnect Flow

```
1. Driver enters tunnel — cellular lost
        │
        ▼
2. CLLocationManager continues firing (foreground or BGProcessingTask wakeup)
        │
        ▼
3. GPS coordinates written to GRDB local cache
   (thread-safe WAL mode, no UI blocking)
        │
        ▼
4. Network restores
        │
        ▼
5. App detects reachability change
        │
        ▼
6. Background GRDB DatabaseQueue executes bulk INSERT
   (100–200 cached records → Vapor backend in one batch)
        │
        ▼
7. WebSocket connection re-established
8. Ride state re-synced from backend FSM
```

---

## 4. Real-Time Connectivity — WebSocket

The iOS client maintains a persistent connection via `URLSessionWebSocketTask`.

```swift
// Registered as TCA Dependency — injectable and mockable
struct WebSocketClient: Sendable {
    var connect: @Sendable (URL) async -> AsyncStream<WebSocketEvent>
    var send: @Sendable (WebSocketMessage) async throws -> Void
    var disconnect: @Sendable () async -> Void
}
```

**Why wrapped in `DependencyValues`:**
- Unit tests run with a mock client that emits predefined events
- SwiftUI Previews render correct states without a running server
- No `#if DEBUG` branches in production code

### Ping Frequencies

| Mode | Frequency | Mechanism |
|---|---|---|
| Active foreground | Every 5 seconds | `URLSessionWebSocketTask` send loop |
| Background (significant movement) | On wakeup | `startMonitoringSignificantLocationChanges()` + `BGProcessingTask` |

**Background mode flow:**
1. App moves to background
2. iOS system monitors significant location changes
3. On movement: system wakes app, `BGProcessingTask` fires
4. App reconnects WebSocket, flushes GRDB cache to backend
5. App suspends again

---

## 5. Background Location Tracking

```swift
// Effect.run — correct long-lived background effect in TCA
case .startLocationTracking:
    return .run { send in
        for await location in LocationService.shared.locationStream() {
            await send(.locationUpdated(location))
        }
    }
    .cancellable(id: LocationTrackingID.self)

case .stopLocationTracking:
    return .cancel(id: LocationTrackingID.self)
```

`Effect.run` wraps the async stream safely — the main thread is never blocked. `.cancellable(id:)` provides explicit lifecycle control when the ride ends.

---

## Related Files
- [[WebSocket_Hub_Architecture.md]] — backend WebSocket hub and Redis Pub/Sub
- [[Ride_State_Machine.md]] — FSM states mirrored on iOS via TCA reducers
- [[Backend_Architecture.md]] — Vapor monolith that iOS communicates with
