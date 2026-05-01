---
status: active
version: v5.0
last_updated: 2026-04-21
phase: 1-2
---

# [[Data Pipeline and Analytics]]

## 1. Event Streaming — Phase Evolution

Following the "Complexity Budget" axiom, the data pipeline evolves in measured steps driven by metrics, not assumptions.

### Phase 1 — Launch (current)

No external message broker. Domain events flow via **Transactional Outbox + PostgreSQL LISTEN/NOTIFY**:

```
Ride FSM transition
        │
        ▼
outbox_events INSERT (same transaction as ride update)
        │
        ▼
Go poller: SELECT unprocessed outbox_events
        │
        ▼
PostgreSQL NOTIFY → broadcast to monolith consumers
        │
        ├──► Driver earnings calculation
        ├──► Passenger notification
        └──► Analytics write (PostgreSQL → ClickHouse via batch export)
```

**Threshold to upgrade:** When daily rides exceed **150,000**, PostgreSQL LISTEN/NOTIFY becomes a measured bottleneck — only then does Kafka appear.

### Phase 2 — Scale (Month 7–18)

**Apache Kafka (KRaft mode)** replaces LISTEN/NOTIFY as the central nervous system:
- Decouples [[Go Modular Monolith]] from downstream analytics consumers
- No ZooKeeper dependency (KRaft = Kafka's built-in consensus)
- Enables event replay for debugging and ML feature backfill

---

## 2. Analytical Warehouse — ClickHouse

**Why not run analytics on PostgreSQL:**
Complex aggregate queries (driver earnings summaries, historical corridor occupancy, demand heatmaps) compete with transactional workload. Under load, they degrade ride booking latency.

**ClickHouse** is strictly isolated from the production API:

| Factor | PostgreSQL | ClickHouse |
|---|---|---|
| Query type | Transactional (row-level) | Analytical (columnar aggregates) |
| Performance on 10M rows | Baseline | 10–100× faster |
| Ingestion | Synchronous writes | Async from event pipeline |
| Isolation | None | Complete — no impact on [[Ride State Machine]] |

**Operations Dashboard** queries ClickHouse every 3 seconds — zero impact on production [[PostgreSQL Database]].

---

## 3. MLOps Pipeline

The AI workload is isolated in a Python ecosystem to protect the Vapor monolith from CPU-bound training.

### Nightly Airflow DAG

```
00:00 — DAG triggers
        │
        ├── Extract daily ride aggregates from ClickHouse
        ├── Enrich with external weather data (temperature, precipitation, wind)
        ├── Join with H3 zone demand history
        │
        ▼
Feature Engineering
        ├── TimeSeriesSplit preparation (no future data leakage)
        ├── Write Parquet files to MinIO (compressed, partitioned by date)
        │
        ▼
Model Training (per active corridor)
        ├── XGBoost with TimeSeriesSplit cross-validation
        ├── Log params + MAPE to MLflow experiment tracker
        ├── If MAPE < 15%: promote model to production registry
        └── If MAPE ≥ 15%: alert operations, keep previous model
        │
        ▼
Deployment
        └── FastAPI inference service reloaded with new model artifact
```

**Critical: TimeSeriesSplit, not k-fold.** Standard k-fold shuffles the time series — future data bleeds into past training folds, producing artificially good validation scores that fail in production. TimeSeriesSplit strictly uses only past data to predict the future.

### MLflow Tracking

Every training run logs:
- Hyperparameter configuration
- Per-fold MAPE scores
- Feature importance
- Model artifact (serialised XGBoost)
- Dataset hash (reproducibility)

---

## 4. Data Quality & Drift Monitoring

A trained model's performance degrades silently as city topology, seasons, and user behaviour shift.

| Check | Mechanism | Threshold | Action |
|---|---|---|---|
| Input validation | Pydantic models in FastAPI | Feature outside expected range | Fall back to 7-day rolling average — guarantees system stability |
| Distribution drift | Population Stability Index (PSI) | PSI > 0.2 over 7-day rolling window | Alert fired — indicates shift in data distribution |
| Accuracy degradation | Live prediction MAPE | MAPE > 25% for 3 consecutive days on any corridor | Airflow auto-triggers retraining job |

---

## 5. Storage Architecture Summary

```
Real-time ops (writes):     PostgreSQL 16 (transactional)
Real-time ops (geo/cache):  Redis 7
Analytical queries:         ClickHouse (async ingestion)
ML features:                MinIO + Parquet (S3-compatible)
ML experiments:             MLflow
Event streaming (Phase 2):  Apache Kafka (KRaft)
```

---

## Related Files
- [[Backend_Architecture.md]] — Transactional Outbox, LISTEN/NOTIFY
- [[Geo_Matching_and_AI.md]] — XGBoost demand forecasting, DBSCAN
- [[Safety_and_Operation.md]] — Operations dashboard queries ClickHouse
- [[Scaling_Roadmap.md]] — Kafka introduction threshold
- [[Open_Questions.md]] — OQ-007 ClickHouse hosting, OQ-008 MLflow hosting
