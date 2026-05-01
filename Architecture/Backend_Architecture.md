---
status: active
version: v5.0
last_updated: 2026-05-02
phase: 1
---

# [[BIRGE Backend Architecture]]

## 1. Why Swift Vapor — Decision Update (2026-04-27)

Original architecture used Go. Decision revised: Swift Vapor chosen for:

| Factor | Reason |
|---|---|
| Single language | iOS + backend in Swift — shared mental model |
| Shared models | BIRGEShared Swift Package (Phase 2) — one source of truth |
| Type safety | Codable models compile-checked end-to-end |
| Developer velocity | One language, one toolchain |

Trade-offs accepted:
- Docker image larger (~250MB vs ~12MB Go)
- Redis/geo ecosystem less mature than Go
- Fewer production case studies

Mitigation: BIRGEShared package will share models between
Vapor and iOS — no OpenAPI codegen needed.

---

## 2. [[Swift Vapor Modular Monolith]] Structure

```
birge-vapor/
├── Package.swift
├── Sources/App/
│   ├── entrypoint.swift            # App bootstrap
│   ├── configure.swift             # Services, DB, Redis, middleware
│   ├── routes.swift                # Route registration
│   ├── Models/                     # Fluent models
│   │   ├── User.swift
│   │   ├── Driver.swift
│   │   └── Ride.swift
│   ├── Modules/
│   │   ├── Auth/
│   │   │   ├── AuthController.swift
│   │   │   ├── AuthService.swift
│   │   │   └── AuthDTO.swift
│   │   ├── Rides/
│   │   │   ├── RidesController.swift
│   │   │   ├── RidesService.swift
│   │   │   └── RidesDTO.swift
│   │   └── WebSocket/
│   │       ├── WSController.swift
│   │       └── WSHub.swift
│   ├── Middleware/
│   │   ├── JWTMiddleware.swift
│   │   └── RateLimitMiddleware.swift
│   ├── Migrations/
│   │   ├── CreateUsers.swift
│   │   ├── CreateDriverProfiles.swift
│   │   └── CreateRides.swift
│   └── Support/
│       └── AuthContext.swift
└── docker-compose.yml
```

The current implementation uses module folders under `Sources/App/Modules/` instead of Go `internal/` packages. Architectural boundaries are maintained through Vapor route/controller/service separation plus Fluent models and middleware composition.

---

## 3. Real-Time Infrastructure — [[Redis Pub/Sub]]

### The Multi-Instance Problem (v5.0 solution)

When the [[Swift Vapor Modular Monolith]] scales horizontally, in-memory WebSocket hubs create isolated silos. A driver on Instance A and a passenger on Instance B cannot communicate directly.

**Solution:** [[Redis Infrastructure]] Pub/Sub as a global broadcast bus.

```
Driver (Instance A) sends GPS update
        │
        ▼
Vapor Instance A publishes to Redis channel "ride:{ride_id}"
        │
        ▼
Redis broadcasts to ALL subscribed Vapor instances
        │
        ▼
Vapor Instance B checks local memory → finds passenger WebSocket → forwards payload
```

Every Vapor instance:
1. Maintains its own local WebSocket hub for connected clients
2. Subscribes to global Redis Pub/Sub channels
3. On receiving a broadcast, checks local clients and forwards

See [[WebSocket_Hub_Architecture.md]] for room management and iOS integration.

---

## 4. Persistent Storage & Event Streaming

Primary transactional store: [[PostgreSQL Database]] v16.

**Complexity Budget enforced:** No Kafka until 150,000 daily rides.

| Pattern | Mechanism | Purpose |
|---|---|---|
| Transactional Outbox | `outbox_events` table + `LISTEN/NOTIFY` | At-least-once event delivery without Kafka |
| Spatial indexing | PostGIS + H3-PG | Hexagonal corridor queries, pickup node clustering |
| Advisory Locks | `pg_try_advisory_xact_lock` | Race-condition-free state transitions in [[Ride State Machine]] |
| Partition pruning | Range partition on `rides.requested_at` by month | O(monthly_rides) instead of O(total_rides) |

### Transactional Outbox Flow

```
1. rides.status = COMPLETED           ┐
2. outbox_events INSERT (same tx)     ┘ single atomic transaction
        │
        ▼
3. Vapor worker/poller reads outbox_events
4. PostgreSQL LISTEN/NOTIFY broadcasts
        │
        ▼
5. Downstream: driver earnings calc, passenger notification
```

---

## 5. Observability First

All critical paths instrumented before production:

| Signal | Tool | Key Metrics |
|---|---|---|
| Metrics | Prometheus / OpenTelemetry metrics | `birge_matching_duration_seconds` (p95 < 500ms budget), `birge_active_drivers`, `birge_corridor_occupancy` |
| Tracing | OpenTelemetry → Jaeger | Full request trace: Nginx → Vapor → PostgreSQL → Redis |
| Logging | Vapor Logger (structured) | `ride_id` and `user_id` attached to every log entry |

---

## 6. Application Bootstrap — `entrypoint.swift` + `configure.swift`

```swift
@main
enum Entrypoint {
    static func main() async throws {
        var env = try Environment.detect()
        try LoggingSystem.bootstrap(from: &env)

        let app = try await Application.make(env)
        defer { app.shutdown() }

        try await configure(app)
        try await app.execute()
    }
}

public func configure(_ app: Application) async throws {
    app.databases.use(.postgres(...), as: .psql)
    app.redis.configuration = try .init(hostname: "127.0.0.1", port: 6379)

    app.migrations.add(CreateUsers())
    app.migrations.add(CreateDriverProfiles())
    app.migrations.add(CreateRides())

    try routes(app)
}
```

---

## Related Files
- [[Ride_State_Machine.md]] — FSM, Advisory Locks, Outbox detail
- [[WebSocket_Hub_Architecture.md]] — Pub/Sub, room management
- [[Geo_Matching_and_AI.md]] — matching engine internals
- [[Database_Schema_and_Migrations.md]] — PostgreSQL schema
- [[Infrastructure_and_Deployment.md]] — Docker, K3s, HPA
