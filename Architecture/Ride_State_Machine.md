---
status: active
version: v5.0
last_updated: 2026-05-05
phase: 1
---

# [[Ride State Machine]]

> Incorrect FSM implementation is the primary source of production bugs in ride-hailing systems: double billing, lost ride status, state race conditions.

## 1. State Diagram

```
                    ┌─────────────┐
                    │  REQUESTED  │ ◄── passenger submits request
                    └──────┬──────┘
                           │ matching engine assigns driver
                           ▼
                    ┌─────────────┐
                    │   MATCHED   │ driver offer sent via push notification
                    └──────┬──────┘
                ┌──────────┴──────────┐
          accept│                     │decline → cascade to next candidate
                ▼                     ▼
       ┌─────────────────┐    ┌─────────────┐
       │ DRIVER_ACCEPTED │    │  REQUESTED  │ (retry)
       └────────┬────────┘    └─────────────┘
                │ driver confirms departure
                ▼
       ┌─────────────────┐
       │ DRIVER_ARRIVING │ ETA updates every 10s via WebSocket
       └────────┬────────┘
                │ geofence trigger at pickup node
                ▼
       ┌─────────────────┐
       │  PASSENGER_WAIT │ 3-minute boarding window
       └────────┬────────┘
                │ passenger boards (4-digit verification code confirmed)
                ▼
       ┌─────────────────┐
       │   IN_PROGRESS   │ live tracking, route deviation monitoring
       └────────┬────────┘
                │ geofence trigger at destination
                ▼
       ┌─────────────────┐
       │   COMPLETED     │ payment processing, driver earnings event
       └─────────────────┘
  Any state → CANCELLED (with reason code)
```

---

## 2. State Definitions

| State | Description | Duration |
|---|---|---|
| `requested` | Passenger submitted the request | Until matching engine responds |
| `matched` | [[Geo_Matching_and_AI\|Matching Engine]] found candidate, offer dispatched | Until driver accepts/declines |
| `driver_accepted` | Driver confirmed the offer | Until departure confirmed |
| `driver_arriving` | Driver navigating to pickup node, GPS streaming | Until pickup geofence triggers |
| `passenger_wait` | Driver at node, waiting for passenger | 3-minute window |
| `in_progress` | Passenger boarded, vehicle moving | Until destination geofence |
| `completed` | Terminal state — payment, earnings calc triggered | — |
| `cancelled` | Terminal state — reason code logged | — |

---

## 3. Atomic Transitions — Advisory Locks

Shared corridor assignments are highly concurrent: multiple passengers may attempt to claim the final seat simultaneously.

**Why Advisory Locks, not `FOR UPDATE`:**
- `FOR UPDATE` holds a row-level lock for the entire transaction duration — slow under heavy load
- `pg_try_advisory_xact_lock` is non-blocking: returns `false` immediately if another process holds the lock
- Significantly faster, no table-row blocking

```swift
// Sources/App/Modules/Rides/RidesService.swift
func updateStatus(
    rideID: UUID,
    from expectedStatus: RideStatus,
    to nextStatus: RideStatus,
    metadata: [String: String]
) async throws {
    try await req.db.transaction { database in
        // Non-blocking advisory lock on ride ID.
        let lockAcquired = try await database.raw(
            "SELECT pg_try_advisory_xact_lock(\(bind: rideID.uuidString))"
        ).first(decoding: Bool.self)

        guard lockAcquired == true else {
            throw Abort(.conflict, reason: "Ride is already being updated.")
        }

        // CAS-style update: only transition if the current status is still expected.
        guard var ride = try await Ride.find(rideID, on: database),
              ride.status == expectedStatus else {
            throw Abort(.conflict, reason: "Ride status changed.")
        }

        ride.status = nextStatus
        ride.updatedAt = Date()
        try await ride.save(on: database)

        // Transactional Outbox event would be inserted in the same transaction.
    }
}
```

---

## 4. Transactional Outbox — At-Least-Once Delivery

Every successful state transition generates a domain event to trigger downstream actions (driver earnings calculation, passenger notification).

**Why Outbox, not direct publish:**
Direct publish after DB write creates a two-phase commit problem — the DB write can succeed while the publish fails, leaving the system inconsistent.

```
State transition SQL           ┐
outbox_events INSERT           ┘ single atomic transaction
        │
        ▼
Vapor background worker reads outbox_events
        │
        ▼
PostgreSQL LISTEN/NOTIFY → broadcast to monolith
        │
        ├──► Driver earnings calculation
        ├──► Passenger WebSocket notification (via [[WebSocket Hub Architecture]])
        └──► Analytics event (ClickHouse pipeline in Phase 2)
```

This pattern provides reliable pub/sub without Kafka until 150,000 daily rides.

---

## 5. iOS Sync — Perfect State Mirror

The backend FSM state is perfectly mirrored on the mobile client for a seamless UX.

```
Backend: FSM transitions to DRIVER_ARRIVING
        │
        ▼
Payload broadcast via [[WebSocket Hub Architecture]] (Redis Pub/Sub scaled)
        │
        ▼
[[Native iOS Architecture]] receives WebSocket event
        │
        ▼
TCA Reducer processes action → single source of truth updated
        │
        ▼
SwiftUI view re-renders automatically — no "ViewModel Hell" across screens
```

---

## Related Files
- [[Backend_Architecture.md]] — Vapor monolith, Outbox, Pub/Sub
- [[WebSocket_Hub_Architecture.md]] — real-time state broadcasting
- [[Geo_Matching_and_AI.md]] — matching engine that drives REQUESTED → MATCHED
- [[iOS_Architecture.md]] — TCA reducer handling FSM events
- [[Database_Schema_and_Migrations.md]] — `rides` table schema, Advisory Locks
