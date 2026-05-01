---
status: active
version: v5.0
last_updated: 2026-05-02
phase: 1
---

# [[BIRGE System Overview]]

## Architectural Axioms

Three axioms govern every decision in this architecture.

**Axiom 1 — Boring Technology First.** Stability over innovation on the critical path. [[PostgreSQL Database]] beats TiDB. [[Redis Infrastructure]] beats Dragonfly. Predictable behavior, rich documentation, large community.

**Axiom 2 — Complexity Budget.** Every component costs engineering time — configuration, monitoring, debugging, onboarding. Kafka does not appear until PostgreSQL `LISTEN/NOTIFY` becomes a measured bottleneck. A microservice is not extracted until the monolith shows a concrete profiling signal.

**Axiom 3 — Observability First.** Every component and every critical path is covered by Prometheus metrics, OpenTelemetry tracing, and structured JSON logs *before* production deploy, not after an incident.

---

## The Problem BIRGE Solves

Almaty has a radial commuting structure — residential districts (Alatau, Bostandyk, Auyezov) on the periphery, business clusters (Esentai–Al-Farabi corridor, Almaly) in the center. This creates predictable, repeating OD (Origin-Destination) patterns ideal for demand pre-aggregation.

| Problem | Symptom | BIRGE Mechanism |
|---|---|---|
| Commission gap | Driver loses 19–22% of revenue | Fixed monthly subscription instead of per-ride commission |
| Price volatility | Surge pricing during peak hours | Corridor subscription — fixed monthly price |
| Low occupancy | 25–35% vehicle utilisation | Shared corridors — 2–4 passengers on matching routes |

---

## Three Service Tiers

```
┌─────────────────────────────────────────────────────────────────┐
│  Tier 3: Corridor Subscription (core product)                   │
│  Fixed OD, departure window, monthly pass                       │
├─────────────────────────────────────────────────────────────────┤
│  Tier 2: Shared Corridor Ride (pay-as-you-go)                   │
│  OD matches active corridor, per-ride payment                   │
├─────────────────────────────────────────────────────────────────┤
│  Tier 1: On-Demand Ride (acquisition + fallback)                │
│  Standard taxi dispatch, operates like Yandex Go                │
└─────────────────────────────────────────────────────────────────┘
```

A passenger completing 5+ on-demand rides on a similar route automatically receives a corridor subscription offer with a personalised savings calculation — the primary conversion funnel.

---

## High-Level System Map

```
┌──────────────────────────────────────────────────────────────────────────┐
│                            CLIENT LAYER                                  │
│         iOS Native (Swift + SwiftUI + TCA)    Driver App  Passenger App  │
└─────────────────────────────┬────────────────────────────────────────────┘
                              │  HTTPS REST / WebSocket (WSS)
┌─────────────────────────────▼────────────────────────────────────────────┐
│                    INFRASTRUCTURE LAYER                                  │
│           Nginx  (TLS 1.3 termination, JWT validation, rate limiting)    │
└───────────────────┬──────────────────────┬───────────────────────────────┘
                    │                      │
      ┌─────────────▼─────────────────────────────────────────────────────┐
      │         Swift Vapor Modular Monolith                              │
      │         (Fluent + SQLKit + async/await)                           │
      │                                                                   │
      │  auth, users, rides, payments, corridors, subscriptions,          │
      │  WebSocket routes, Redis Pub/Sub integration, background jobs     │
      └─────────┬─────────────────────────────────────────────────────────┘
                │
      ┌─────────▼─────────────────────────────────────────────────────────┐
      │                  DATA LAYER                                       │
      │   PostgreSQL 16 (primary) │ Redis 7 │ OSRM local                  │
      └─────────┬─────────────────────────────────────────────────────────┘
                │
      ┌─────────▼──────────────┐
      │   ML Service (Python)  │
      │   FastAPI + XGBoost    │
      │   DBSCAN + H3          │
      └────────────────────────┘
```

---

## Approved Technology Stack

| Component | Technology | Rationale |
|---|---|---|
| API Monolith | Swift Vapor 4 | Same-language backend/client stack, async/await ergonomics, modular monolith simplicity |
| Geo-Matching + WebSocket | Swift Vapor + Redis Pub/Sub | Real-time ride updates stay in-process with Redis fan-out across instances |
| iOS App | Swift + SwiftUI + [[The Composable Architecture]] | Native performance, Apple ecosystem |
| Local Offline Cache | [[GRDB]] | Thread-safe WAL mode — critical for tunnel/elevator drops in Almaty |
| AI/ML Service | Python + FastAPI | scikit-learn, XGBoost, ML engineer availability |
| Primary DB | [[PostgreSQL Database]] 16 + PostGIS + H3-PG | Spatial queries, LISTEN/NOTIFY, proven at scale |
| Cache & Real-Time | [[Redis Infrastructure]] 7 | GEOSEARCH, Pub/Sub for WebSocket scaling, rate limiting |
| Connection Pooling | PgBouncer (transaction mode) | Protects PostgreSQL as Vapor instances scale horizontally |
| Routing | OSRM (self-hosted) | <20ms batch ETA, no external API cost |
| Geo Index | H3 (Uber) | Equidistant hexagons, eliminates S2 diagonal distortion |
| Migrations | Fluent + SQL migrations | App-owned schema evolution in the Swift backend |
| Contracts | OpenAPI 3.1 | Language-agnostic source of truth for iOS ↔ Vapor |
| Observability | Prometheus + OpenTelemetry + structured logs | Full metrics, tracing, structured logs |
| Infra | K3s + ArgoCD + GitHub Actions | GitOps, Kazakhstan-hosted (data localisation law) |

---

## Late-Mover Advantage

Uber rewrote its system 3 times (Python → Node.js → Go). BIRGE keeps the same "boring tech first" principle while using a modern Swift-first stack for the transactional core.

| Layer | Uber (2017) | BIRGE Phase 1 |
|---|---|---|
| API | Node.js Ringpop | Swift Vapor modular monolith |
| Storage | MySQL → Docstore | PostgreSQL + PostGIS |
| Geo | S2 → H3 | H3 from day 1 |
| Events | Kafka | PostgreSQL LISTEN/NOTIFY (Kafka at 150k rides/day) |
| WebSocket scale | Proprietary | Redis Pub/Sub |
| Mobile | Native iOS + Android | iOS Native (Swift + TCA) |

---

## Related Files
- [[Backend_Architecture.md]] — Vapor modular monolith internals
- [[Ride_State_Machine.md]] — FSM lifecycle
- [[Geo_Matching_and_AI.md]] — matching engine and ML pipeline
- [[iOS_Architecture.md]] — TCA and GRDB details
- [[Scaling_Roadmap.md]] — phase bottlenecks by MAU
