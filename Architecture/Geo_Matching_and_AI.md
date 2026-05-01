---
status: active
version: v5.0
last_updated: 2026-04-21
phase: 1
---

# [[Geo-Matching and AI Architecture]]

## 1. Matching Engine — Latency Budget: p95 < 500ms

The [[Geo_Matching_and_AI|Matching Engine]] lives inside the [[Go Modular Monolith]]. It is the most latency-critical component in the system.

| Operation | Target | Tool |
|---|---|---|
| Request validation | <10ms | In-memory check |
| PostgreSQL: create ride | <20ms | Indexed write |
| Redis GEOSEARCH: find drivers | <5ms | Redis GEO |
| OSRM: batch ETA (1→20 drivers) | <20ms | Parallel goroutines |
| Scoring + ranking | <5ms | In-memory |
| PostgreSQL Advisory Lock + update | <15ms | Advisory Lock |
| Redis Pub/Sub: notify driver | <5ms | Pub/Sub |
| **Total p95** | **<500ms** | |

### Candidate Retrieval
Redis `GEOSEARCH` finds all active drivers within a 3km radius in sub-millisecond time. Drivers update their location every 5 seconds — data is always fresh.

### Batch ETA Computation
Instead of slow, costly external mapping APIs, the backend uses concurrent async requests to a **self-hosted OSRM** (Open Source Routing Machine). Matrix ETA (one source → many destinations) completes in <20ms.

### Composite Scoring
Candidates are not ranked purely by proximity. The scoring algorithm weighs:
- ETA to passenger pickup
- Driver rating
- System load balancing (avoid clustering all rides on one driver)
- Corridor history match (prefer drivers familiar with the route)

### Atomic Assignment
Once the best candidate is selected, the assignment uses a PostgreSQL Advisory Lock (see [[Ride_State_Machine.md]]) to prevent two passengers booking the same driver simultaneously, then dispatches the offer via [[Redis Infrastructure]] Pub/Sub.

---

## 2. Geospatial Foundation — H3 Indexing

BIRGE uses Uber's open-source **H3** hierarchical hexagonal grid instead of raw GPS coordinates.

**Why hexagons over squares (S2):**
- Hexagons are equidistant to all 6 neighbours — no diagonal distortion
- S2 cells have diagonal neighbours that are farther than edge neighbours — introduces systematic bias in proximity calculations

| Resolution | Approx diameter | Usage |
|---|---|---|
| Resolution 8 | ~460m | Precise pickup node clustering, real-time operations |
| Resolution 7 | ~1.2km | Origin-Destination matrix construction, district-level commute flow analysis |

**Storage:** [[PostgreSQL Database]] uses the `h3-pg` extension alongside PostGIS to compute and query hexagonal indexes natively, reducing memory footprint during spatial lookups.

---

## 3. AI Route System — Python Pipeline

ML workload is strictly isolated from the transactional backend. CPU-bound training workloads must not compete with request-serving API workers.

### Corridor Detection — DBSCAN

**Why DBSCAN, not K-means:**
- K-means requires specifying cluster count in advance — we don't know how many natural corridors exist
- K-means cannot represent irregular shapes — real commute flows follow road topology
- DBSCAN automatically filters GPS noise (parking lots, outliers) without requiring a noise threshold

```python
from sklearn.cluster import DBSCAN

# eps=0.01 degrees ≈ 1km, min_samples=50 pickup events
db = DBSCAN(eps=0.01, min_samples=50, metric='haversine', algorithm='ball_tree')
labels = db.fit_predict(pickup_coords_rad)
# label == -1 → noise, filtered out
```

### Demand Forecasting — XGBoost

Predicts corridor demand 24 hours in advance per corridor.

**Why XGBoost, not deep learning:**
- Highly sample-efficient on CPU infrastructure
- Trains quickly on tabular time-series data
- No GPU required for inference (<50ms on CPU)
- Interpretable feature importance

**Features:** hour of day, day of week, holiday flag, weather (temperature, precipitation, wind), H3 zone density, previous 7-day corridor occupancy.

**Critical: TimeSeriesSplit cross-validation** — standard k-fold shuffling would let future data leak into past training folds, producing fraudulently good validation scores.

### MLOps Pipeline

```
Nightly Airflow DAG:
    │
    ├── Extract daily ride aggregates from ClickHouse (Phase 2)
    ├── Enrich with weather data (external API)
    ├── Write engineered features to MinIO as Parquet (compressed)
    │
    ├── Train XGBoost with TimeSeriesSplit
    ├── Log params + MAPE to MLflow
    └── Deploy if MAPE < 15%
```

### Data Quality & Drift Monitoring

| Check | Mechanism | Threshold | Action |
|---|---|---|---|
| Input validation | FastAPI Pydantic models | Feature outside expected range | Fall back to 7-day rolling average |
| Distribution drift | Population Stability Index (PSI) | PSI > 0.2 over 7-day window | Alert triggered |
| Accuracy degradation | MAPE on live predictions | MAPE > 25% for 3 consecutive days on any corridor | Airflow auto-triggers retraining |

---

## Related Files
- [[Backend_Architecture.md]] — Vapor monolith that calls the Python FastAPI service
- [[Ride_State_Machine.md]] — REQUESTED → MATCHED transition driven by this engine
- [[Data_Pipeline_and_Analytics.md]] — ClickHouse feed for ML features (Phase 2)
- [[Database_Schema_and_Migrations.md]] — PostGIS + H3-PG extensions
