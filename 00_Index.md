---
version: v5.1
last_updated: 2026-05-02
---

# BIRGE Vault — Index

> Architecture version: v5.0 | Vault structure: v5.1

## Quick Navigation

| Folder | Purpose |
|---|---|
| `Architecture/` | System design, stack decisions, domain specs |
| `Tasks/` | Active sprints, TODO lists, Done archive |
| `Decisions/` | ADR — Architecture Decision Records |
| `Context/` | Agent briefing files — feed to Antigravity before tasks |
| `References/` | Code patterns, recipes, API contracts |

---

## Architecture Files

| File | Domain | Status |
|---|---|---|
| [[CLAUDE]] | Agent instructions & rules | ✅ Active |
| [[Architecture/System_Overview]] | Bird's-eye view, service tiers, stack | ✅ Active |
| [[Architecture/Backend_Architecture]] | Swift Vapor modular monolith, Redis Pub/Sub, Outbox | ✅ Active |
| [[Architecture/Ride_State_Machine]] | FSM states, Advisory Locks, Outbox | ✅ Active |
| [[Architecture/WebSocket_Hub_Architecture]] | Multi-instance scaling, Redis Pub/Sub | ✅ Active |
| [[Architecture/Geo_Matching_and_AI]] | Matching engine, H3, OSRM, XGBoost | ✅ Active |
| [[Architecture/iOS_Architecture]] | TCA, GRDB, background modes | ✅ Active |
| [[Architecture/Database_Schema_and_Migrations]] | PostgreSQL, partitioning, Advisory Locks | ✅ Active |
| [[Architecture/Payment_and_Financial_Architecture]] | Kaspi, ledger, subscriptions | ✅ Active |
| [[Architecture/Infrastructure_and_Deployment]] | K3s, ArgoCD, HPA, Dockerfile | ✅ Active |
| [[Architecture/Security_and_Authentication]] | OTP, JWT, KYC, fraud detection | ✅ Active |
| [[Architecture/Safety_and_Operation]] | Risk scoring, in-ride safety, ops dashboard | ✅ Active |
| [[Architecture/Data_Pipeline_and_Analytics]] | ClickHouse, Kafka roadmap, MLOps | ✅ Active |
| [[Architecture/Network_Effects_and_Growth]] | Cold-start, density, data moat | ✅ Active |
| [[Architecture/Open_Questions]] | Unresolved decisions & open risks | 🔄 Living |
| [[Architecture/Scaling_Roadmap]] | Phase bottlenecks, MAU thresholds | ✅ Active |

---

## Context Files (feed to Antigravity)

| File | Use when |
|---|---|
| [[Context/Current_Focus]] | Every session — что делаем прямо сейчас |
| [[Context/iOS_Agent_Context]] | Любая iOS задача |
| [[Context/Backend_Agent_Context]] | Любая Go/Backend задача |

---

## Active Tasks

| File | Domain | Sprint |
|---|---|---|
| [[Tasks/iOS_Sprint_1]] | iOS | Sprint 1 |
| [[Tasks/Backend_Sprint_1]] | Backend | Sprint 1 |

---

## Architecture Decisions

| ID | Decision | Status |
|---|---|---|
| [[Decisions/ADR-001_GRDB_vs_SwiftData]] | GRDB вместо SwiftData | ✅ Closed |
| [[Decisions/ADR-002_TCA_vs_MVVM]] | TCA вместо MVVM | ✅ Closed |
| [[Decisions/ADR-003_SwiftUI_vs_UIKit]] | SwiftUI как основной UI фреймворк | ✅ Closed |

---

## Phase Status

| Phase | Scope | Status |
|---|---|---|
| Phase 1 (Months 1–6) | Swift Vapor monolith, PostgreSQL, Redis, iOS, OTP/JWT, manual corridors, OSRM, observability, K3s | 🏗️ Build |
| Phase 2 (Months 7–18) | Matching Service extraction, Kafka, ClickHouse, XGBoost forecasting, DBSCAN, dynamic pricing | 📋 Planned |
| Phase 3 (Month 18+) | Multi-city sharding, Event Sourcing, RL pricing, GNN routing, corporate partnerships | 🔮 Future |

## Key Metrics — Phase 1 Target (Month 18)

- Subscribed drivers: **1,200**
- Monthly active passengers: **18,000**
- Active corridors: **35–50**
- Monthly revenue: **30–55M tenge**
- Matching latency: **p95 < 500ms**
- WebSocket connections: **up to 50,000 simultaneous**
