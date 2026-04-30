---
status: active
version: v5.0
last_updated: 2026-04-21
---

# [[Scaling Roadmap]]

## Bottleneck Timeline by MAU

| MAU | Bottleneck | Solution |
|---|---|---|
| 1k | None — server idle 95% | Develop features |
| 10k | OSRM ETA latency | Cache frequent ETA pairs in Redis |
| 50k | PostgreSQL write throughput | Read replicas, PgBouncer tuning |
| 100k | Redis single-instance limits | Redis Cluster (3 shards) |
| 200k | Monolith → matching hotspot | Extract Matching Service |
| 500k | PostgreSQL table scans | Range partitioning (already implemented on `rides`) |
| 1M | Geo data distribution | Geographic DB sharding |

---

## Phase Roadmap

### Phase 1 — Months 1–6 (Build)
- ✅ Go Modular Monolith
- ✅ PostgreSQL 16 + PostGIS + H3-PG
- ✅ Redis GEO + Pub/Sub
- ✅ iOS Native — Swift + SwiftUI + TCA + GRDB
- ✅ OTP + JWT authentication
- ✅ Manual corridor operations
- ✅ On-demand rides (Tier 1)
- ✅ OSRM self-hosted routing
- ✅ Prometheus + OpenTelemetry + Jaeger
- ✅ K3s deployment + ArgoCD GitOps
- ✅ WebSocket Redis Pub/Sub horizontal scaling ← **v5.0**
- ✅ Network disconnect/reconnect with GRDB ← **v5.0**

### Phase 2 — Months 7–18 (Scale)
- Extract Matching Service (when profiling shows monolith bottleneck)
- Kafka cluster (when PostgreSQL LISTEN/NOTIFY hits throughput limit at ~150k rides/day)
- ClickHouse analytical warehouse
- XGBoost demand forecasting (24-hour ahead per corridor)
- DBSCAN corridor auto-detection
- Dynamic pricing engine
- Full analytics stack (Airflow, MLflow, Parquet/MinIO)
- Kaspi deep integration — recurring billing, QR
- Redis Cluster (3 shards)
- HPA auto-scaling tuned on `birge_matching_queue_depth`

### Phase 3 — Month 18+ (Expand)
- Event Sourcing
- Multi-city geographic sharding
- Autonomous corridor lifecycle (no manual validation)
- RL pricing agent
- Graph Neural Networks for routing optimisation
- Corporate partnership API

---

## Industry Comparison

| Layer | Uber (2017) | Grab (2019) | BIRGE Phase 1 |
|---|---|---|---|
| API | Node.js Ringpop | Go microservices | Go monolith (Gin) |
| Matching | ROMAD global optimisation | Greedy + local search | Greedy scoring |
| Storage | MySQL → Docstore | MySQL → PostgreSQL | PostgreSQL + PostGIS |
| Geo | S2 → H3 | S2 | H3 from day 1 |
| Events | Kafka | Kafka | PostgreSQL LISTEN/NOTIFY |
| Location | Internal geo service | Gamma service | Redis GEO |
| Mobile | Native iOS + Android | Native | iOS Native (Swift + TCA) |
| WebSocket scale | Proprietary | Proprietary | Redis Pub/Sub |

**Key insight:** Uber rewrote its system 3 times. BIRGE starts with Uber's final architecture.

---

## Five v5.0 Decisions That Prevent Early Rewrites

1. **Redis Pub/Sub for WebSocket** — solves multi-instance scaling before it becomes a crisis. Driver on Instance A always reaches Passenger on Instance B.
2. **GRDB instead of SwiftData** — thread-safe local cache with WAL mode. Critical for GPS accumulation in Almaty's tunnels and elevators.
3. **WebSocketClient in TCA DependencyValues** — fully mockable in unit tests and SwiftUI Previews. No production code changes for testing.
4. **Transactional Outbox** — reliable event delivery without Kafka. Defer Kafka to a measured threshold, not a guess.
5. **Effect.run for background GPS** — correct TCA lifecycle management. No UI thread blocking, explicit cancellation when ride ends.

---

## Related Files
- [[System_Overview.md]] — current stack overview
- [[Backend_Architecture.md]] — monolith internals
- [[Infrastructure_and_Deployment.md]] — K3s, HPA configuration
- [[Data_Pipeline_and_Analytics.md]] — Phase 2 analytics stack
- [[Open_Questions.md]] — Redis Cluster timing decision
