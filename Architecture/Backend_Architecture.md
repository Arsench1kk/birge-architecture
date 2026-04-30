---
status: active
version: v5.0
last_updated: 2026-04-21
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

## 2. [[Go Modular Monolith]] Structure

```
birge/
├── cmd/server/main.go              # Entry point, DI container
│
├── internal/                       # Never imported externally
│   ├── auth/
│   │   ├── handler.go              # HTTP handlers (Gin)
│   │   ├── service.go              # Business logic
│   │   ├── repository.go           # DB layer (pgx)
│   │   └── middleware.go           # JWT validation middleware
│   │
│   ├── rides/
│   │   ├── handler.go
│   │   ├── service.go
│   │   ├── repository.go
│   │   ├── model.go
│   │   └── statemachine.go         # Ride lifecycle FSM → [[Ride_State_Machine]]
│   │
│   ├── matching/
│   │   ├── engine.go               # Matching algorithm → [[Geo_Matching_and_AI]]
│   │   ├── geo.go                  # Redis GEO operations
│   │   └── scoring.go              # Candidate scoring
│   │
│   └── websocket/
│       ├── hub.go                  # Connection manager
│       ├── room.go                 # Room abstraction
│       └── pubsub.go               # Redis Pub/Sub bridge ← KEY v5.0 component
│
├── pkg/
│   ├── database/postgres.go        # pgxpool connection
│   ├── cache/redis.go
│   ├── geo/
│   │   ├── h3.go                   # H3 geospatial indexing
│   │   └── osrm.go                 # OSRM client
│   └── telemetry/
│       ├── metrics.go              # Prometheus
│       ├── tracing.go              # OpenTelemetry
│       └── logging.go              # zerolog structured JSON
│
└── migrations/                     # golang-migrate SQL files
    ├── 001_users.up.sql
    ├── 001_users.down.sql
    └── ...
```

Go's compiler strictly prevents external imports of `internal/` packages — this creates hard architectural boundaries without microservice operational overhead.

---

## 3. Real-Time Infrastructure — [[Redis Pub/Sub]]

### The Multi-Instance Problem (v5.0 solution)

When the [[Go Modular Monolith]] scales horizontally, in-memory WebSocket hubs create isolated silos. A driver on Instance A and a passenger on Instance B cannot communicate directly.

**Solution:** [[Redis Infrastructure]] Pub/Sub as a global broadcast bus.

```
Driver (Instance A) sends GPS update
        │
        ▼
Go Server A publishes to Redis channel "ride:{ride_id}"
        │
        ▼
Redis broadcasts to ALL subscribed Go instances
        │
        ▼
Go Server B checks local memory → finds passenger WebSocket → forwards payload
```

Every Go instance:
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
3. Go poller reads outbox_events
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
| Metrics | Prometheus (`prometheus/client_golang`) | `birge_matching_duration_seconds` (p95 < 500ms budget), `birge_active_drivers`, `birge_corridor_occupancy` |
| Tracing | OpenTelemetry → Jaeger | Full request trace: Nginx → Go monolith → PostgreSQL → Redis |
| Logging | zerolog (structured JSON) | `ride_id` and `user_id` attached to every log entry |

---

## 6. Dependency Injection — `main.go` Pattern

```go
func main() {
    cfg, _ := config.Load()

    db, _  := database.NewPool(cfg)
    defer db.Close()

    rdb := cache.NewClient(cfg)
    defer rdb.Close()

    osrm := geo.NewOSRMClient(cfg.OSRMBaseURL)

    // Repositories
    userRepo     := users.NewRepository(db)
    rideRepo     := rides.NewRepository(db)
    corridorRepo := corridors.NewRepository(db)

    // Services — clean DI, no global variables
    authSvc     := auth.NewService(userRepo, rdb, cfg)
    pricingSvc  := pricing.NewService(corridorRepo)
    matchingEng := matching.NewEngine(rdb, osrm, pricingSvc)
    rideSvc     := rides.NewService(rideRepo, matchingEng, pricingSvc)

    // WebSocket Hub with Redis Pub/Sub bridge
    wsHub := websocket.NewHub(rdb)
    go wsHub.Run()

    router := setupRouter(cfg, authSvc, rideSvc, wsHub)

    // Graceful shutdown on SIGINT/SIGTERM
    srv := &http.Server{
        Addr: fmt.Sprintf(":%d", cfg.Port),
        Handler: router,
        ReadTimeout: 10 * time.Second,
        WriteTimeout: 30 * time.Second,
    }
    // ...
}
```

---

## Related Files
- [[Ride_State_Machine.md]] — FSM, Advisory Locks, Outbox detail
- [[WebSocket_Hub_Architecture.md]] — Pub/Sub, room management
- [[Geo_Matching_and_AI.md]] — matching engine internals
- [[Database_Schema_and_Migrations.md]] — PostgreSQL schema
- [[Infrastructure_and_Deployment.md]] — Docker, K3s, HPA
