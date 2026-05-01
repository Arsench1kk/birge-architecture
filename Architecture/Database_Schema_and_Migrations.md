---
status: active
version: v5.0
last_updated: 2026-04-21
phase: 1
---

# [[Database Schema and Migrations]]

## 1. PostgreSQL 16 — Extensions

Following the "Boring Technology First" axiom, [[PostgreSQL Database]] v16 is the primary persistent store.

| Extension | Purpose |
|---|---|
| `uuid-ossp` | UUID primary keys |
| `PostGIS` | Native spatial queries — geofence checks, distance calculations |
| `h3-pg` | Hierarchical hexagonal indexing — corridor clustering, OD matrix |
| `pg_stat_statements` | Query performance monitoring |

**Connection Pooling:** [[Infrastructure_and_Deployment|PgBouncer]] runs in **transaction mode** — 5,000 application-side goroutines safely share a pool of 50 server-side connections. Without this, horizontal scaling would exhaust PostgreSQL's connection limit instantly.

---

## 2. Core Schema

### Users & Profiles

```sql
CREATE TABLE users (
    id          UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    phone       VARCHAR(20) UNIQUE NOT NULL,
    role        VARCHAR(20) NOT NULL CHECK (role IN ('passenger', 'driver')),
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Driver-specific data normalised into secondary table
CREATE TABLE driver_profiles (
    user_id         UUID PRIMARY KEY REFERENCES users(id),
    vehicle_model   VARCHAR(100),
    license_plate   VARCHAR(20),
    kyc_status      VARCHAR(20) DEFAULT 'pending',
    risk_score      NUMERIC(5,2) DEFAULT 100.0,
    subscription_tier VARCHAR(20),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### Rides Table — Partitioned

The `rides` table handles the highest transactional volume. **Range partition by month** on `requested_at`:

```sql
CREATE TABLE rides (
    id              UUID NOT NULL DEFAULT uuid_generate_v4(),
    passenger_id    UUID NOT NULL REFERENCES users(id),
    driver_id       UUID REFERENCES users(id),
    corridor_id     UUID REFERENCES corridors(id),
    status          VARCHAR(30) NOT NULL DEFAULT 'requested',
    origin_lat      NUMERIC(10,7),
    origin_lng      NUMERIC(10,7),
    dest_lat        NUMERIC(10,7),
    dest_lng        NUMERIC(10,7),
    origin_h3_r8    VARCHAR(20),  -- H3 index at resolution 8
    metadata        JSONB,        -- surge multiplier, seat count, etc.
    requested_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    completed_at    TIMESTAMPTZ
) PARTITION BY RANGE (requested_at);

CREATE TABLE rides_2026_04 PARTITION OF rides
    FOR VALUES FROM ('2026-04-01') TO ('2026-05-01');
```

**Why JSONB metadata:** Variable-structure data (surge multipliers for on-demand rides, passenger counts for shared corridors) avoids complex migrations for new ride types. GIN index maintains query speed.

**Partition benefit:** Query planner automatically excludes irrelevant partitions. Reduces query time from O(total_rides) to O(monthly_rides).

### Outbox Events

```sql
CREATE TABLE outbox_events (
    id              UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    aggregate_id    UUID NOT NULL,
    event_type      VARCHAR(100) NOT NULL,
    payload         JSONB NOT NULL,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    processed_at    TIMESTAMPTZ  -- NULL = unprocessed
);

CREATE INDEX idx_outbox_unprocessed ON outbox_events(created_at)
    WHERE processed_at IS NULL;
```

### Payment Records

```sql
CREATE TABLE payment_records (
    id                      UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    ride_id                 UUID REFERENCES rides(id),
    driver_id               UUID REFERENCES users(id),
    kaspi_transaction_id    VARCHAR(100) UNIQUE,  -- idempotency key
    amount_tenge            NUMERIC(12,2),
    status                  VARCHAR(20),
    created_at              TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Network retry safety: duplicate kaspi_transaction_id silently ignored
-- ON CONFLICT (kaspi_transaction_id) DO NOTHING
```

---

## 3. Advisory Locks — Race Condition Prevention

The [[Ride State Machine]] uses `pg_try_advisory_xact_lock` for atomic state transitions.

**Pattern:**
```sql
-- Non-blocking: returns FALSE immediately if another process holds the lock
SELECT pg_try_advisory_xact_lock(hashtext(ride_id::text));
```

Compared to `SELECT ... FOR UPDATE`:
- Advisory lock: returns immediately with `false` if contested
- `FOR UPDATE`: blocks the calling goroutine until the lock is released
- Under high concurrency (shared corridor seat assignment), Advisory Locks are significantly faster

---

## 4. Migration Strategy

Schema evolution is managed as code via versioned SQL/Swift backend migrations:

```
migrations/
├── 001_create_users.up.sql
├── 001_create_users.down.sql
├── 002_create_rides.up.sql
├── 002_create_rides.down.sql
├── 003_create_corridors.up.sql
├── 003_create_corridors.down.sql
└── ...
```

**CI/CD execution:**
1. GitHub Actions builds image
2. **Kubernetes Job** runs the production migration step against the DB
3. If migration fails → deployment halts, DB left in pre-migration state
4. Main API pods deploy only after successful migration

This guarantees structural consistency across all environments.

---

## Key Indexes

```sql
-- Driver location lookup (Redis GEO handles real-time; this for historical queries)
CREATE INDEX idx_rides_driver_status ON rides(driver_id, status)
    WHERE status NOT IN ('completed', 'cancelled');

-- Outbox poller performance
CREATE INDEX idx_outbox_unprocessed ON outbox_events(created_at)
    WHERE processed_at IS NULL;

-- Corridor occupancy queries
CREATE INDEX idx_rides_corridor_date ON rides(corridor_id, requested_at);

-- Spatial index for geo queries
CREATE INDEX idx_rides_origin_geo ON rides
    USING GIST(ST_MakePoint(origin_lng, origin_lat));
```

---

## Related Files
- [[Backend_Architecture.md]] — PgBouncer, Transactional Outbox
- [[Ride_State_Machine.md]] — Advisory Locks in state transitions
- [[Infrastructure_and_Deployment.md]] — migration flow in CI/CD
- [[Payment_and_Financial_Architecture.md]] — payment_records idempotency
