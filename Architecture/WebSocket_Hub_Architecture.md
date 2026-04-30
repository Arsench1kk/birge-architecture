---
status: active
version: v5.0
last_updated: 2026-04-21
phase: 1
---

# [[WebSocket Hub Architecture]]

## 1. The Multi-Instance Scaling Problem

Real-time communication is fundamental — constant bidirectional GPS and status updates between drivers and passengers. When the [[Go Modular Monolith]] scales horizontally, a standard in-memory WebSocket hub breaks:

```
Instance A: Driver connected
Instance B: Passenger connected

Driver sends GPS update → Instance A's in-memory hub has no Passenger
Result: Passenger never receives update ❌
```

This is the problem that kills most ride-hailing systems at their first autoscaling event.

---

## 2. Solution — [[Redis Infrastructure]] Pub/Sub Global Bus (v5.0)

```
Driver (→ Instance A)
        │ GPS update
        ▼
Instance A publishes to Redis channel "ride:{ride_id}"
        │
        ▼ Redis broadcasts to ALL subscribers
        │
        ├──► Instance A: checks local hub → Driver is here, skip
        ├──► Instance B: checks local hub → Passenger is here → forwards
        └──► Instance C: checks local hub → nobody → skip
```

Every Go server instance:
1. Maintains its own **local WebSocket hub** (in-memory map of connected clients)
2. **Subscribes to Redis Pub/Sub channels** for all ride events
3. On receiving a broadcast: checks local clients, forwards if found

No state shared between instances except through Redis — purely stateless horizontal scaling.

---

## 3. Room Management

The hub is organised around **Rooms**:

| Room Type | Channel | Subscribers |
|---|---|---|
| Ride room | `ride:{ride_id}` | Driver + all passengers on that ride |
| Driver room | `driver:{driver_id}` | Operations dashboard, matched passenger |
| Corridor room | `corridor:{corridor_id}` | All subscribers of that corridor |

Room lifecycle is managed by the [[Ride State Machine]]:
- `MATCHED` → Driver and Passenger subscribe to `ride:{ride_id}`
- `COMPLETED` or `CANCELLED` → Both unsubscribe, room cleaned up

**Driver location ping frequency:** every 5 seconds → 20 updates/minute per driver → at 50,000 active drivers = 1,000,000 Redis publishes/minute. Redis handles this comfortably with Pub/Sub throughput.

---

## 4. Payload Structure

```json
{
  "event": "ride.location_update",
  "ride_id": "uuid",
  "payload": {
    "lat": 43.2567,
    "lng": 76.9286,
    "heading": 182.4,
    "speed_kmh": 45.2,
    "eta_seconds": 180
  },
  "timestamp_ms": 1714000000000
}
```

Events flowing through the hub:
- `ride.location_update` — driver GPS, 5s interval
- `ride.status_changed` — FSM state transition (from [[Ride State Machine]])
- `ride.eta_updated` — OSRM recalculation
- `corridor.occupancy_changed` — seat count update

---

## 5. [[Native iOS Architecture]] Integration

```swift
// WebSocketClient registered as TCA Dependency
// Backed by URLSessionWebSocketTask
struct WebSocketClient: DependencyKey {
    var connect: (URL) -> AsyncStream<WebSocketEvent>
    var send: (WebSocketMessage) async throws -> Void
    var disconnect: () -> Void
}
```

**TCA reducer processes events:**
```swift
case .webSocketEvent(.received(let data)):
    let event = try JSONDecoder().decode(RideEvent.self, from: data)
    switch event.type {
    case "ride.status_changed":
        state.rideStatus = event.payload.status  // single source of truth
    case "ride.location_update":
        state.driverLocation = event.payload.coordinate
    }
    return .none
```

**Offline resilience via [[GRDB]]:**
If WebSocket drops (tunnel, elevator), [[iOS_Architecture|GRDB]] accumulates GPS tracks locally. On reconnect:
1. WebSocket re-established
2. GRDB bulk-syncs cached tracks to backend
3. TCA reducer re-fetches current ride state
4. UI updates — no stale state visible to user

---

## Related Files
- [[Backend_Architecture.md]] — Redis Pub/Sub setup in Go monolith
- [[Ride_State_Machine.md]] — events that flow through the hub
- [[iOS_Architecture.md]] — TCA WebSocketClient dependency
- [[Infrastructure_and_Deployment.md]] — Redis 7, horizontal pod scaling
