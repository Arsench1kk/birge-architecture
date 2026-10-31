---
version: v5.5
last_updated: 2026-05-12
---

# BIRGE Vault — Index

> Architecture version: v5.0 | Vault structure: v5.3 | Business foundation: v1.0

## Source of Truth

This vault is the planning and architecture source of truth for BIRGE.

Canonical paths:
- App repo: `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp`
- Dev Brain vault: `/Users/arsenabduhalyk/Documents/Dev_ArsensBrain/Projects/Birge Taxi`

Current state is resolved in this order:
1. [[Context/Current_Focus]]
2. [[Business_Department/]] for business/product logic
3. [[Tasks/]]
4. [[Architecture/]] and [[Decisions/]]
5. App repo implementation
6. [[Sessions/]] and [[docs/codex-prompts/]] as historical context only

Canonical current names: `BIRGEPassenger`, `BIRGEDrive`, `BIRGECore`, Swift Vapor backend `birge-vapor`.

Current product identity: BIRGE is a **transportation-as-a-subscription** platform. Tier 3 subscription commute is the wedge. On-demand is fallback/acquisition only.

## Quick Navigation

| Folder | Purpose |
|---|---|
| `Architecture/` | System design, stack decisions, domain specs |
| `Business_Department/` | Business model, product logic, pricing, economics, validation |
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

## Business Department

| File | Use when |
|---|---|
| [[Business_Department/00_Business_Index]] | Business department overview and current rules |
| [[Business_Department/Product_Logic]] | Product model, tiers, passenger/driver flow implications |
| [[Business_Department/Subscription_Strategy]] | Passenger subscription packages: MVP launches with all three tiers |
| [[Business_Department/Driver_Economics]] | Driver subscription tiers, breakeven, cold-start risk |
| [[Business_Department/Business_Open_Questions]] | Business/legal/market unknowns and validation needs |
| [[Business_Department/Project_Change_Plan]] | How the current app/docs should pivot toward subscription commute |

## Context Files (feed to Antigravity)

| File | Use when |
|---|---|
| [[Context/Current_Focus]] | Every session — что делаем прямо сейчас |
| [[Context/iOS_Agent_Context]] | Любая iOS задача |
| [[Context/Backend_Agent_Context]] | Любая Backend / Vapor задача |
| [[Context/Universal_AI_Handoff_BIRGE]] | Current v3.0 handoff: business foundation + tech context |
| [[Context/Universal_AI_Handoff_BIRGE.2]] | Same updated handoff snapshot from the 2026-05-07 business foundation pass |
| [[Context/implementation_plan_]] | Claude Code technical audit and transition plan artifact |
| [[Context/Business_Partner_Prompt_BIRGE]] | Новый бизнес-чат: стратегия, продуктовая логика, рынок, экономика, фичи |
| [[Context/Open_Design_Handoff_BIRGE]] | Новый чат для Open Design, mockups, дизайн-системы и улучшения экранов |

---

## Technical Department — iOS Frontend

| File | Purpose |
|---|---|
| [[Technical_Department/iOS/Swift_Frontend_Master_Plan]] | Canonical execution guide for Swift frontend work |
| [[Technical_Department/iOS/Swift_Frontend_Next_Tasks]] | Current executable task batch for coding agents |
| [[Technical_Department/iOS/Swift_Frontend_Agent_Task_Template]] | Reusable prompt template for weaker coding models |

---

## Active Tasks

| File | Domain | Sprint |
|---|---|---|
| [[Technical_Department/iOS/Swift_Frontend_Next_Tasks]] | iOS Frontend | Active — Batch 1 |
| [[Tasks/Swift_Frontend_Mockup_Implementation_Plan]] | iOS Frontend | Foundation plan |
| [[Tasks/Technical_Subscription_Pivot]] | iOS/Backend | Technical pivot plan |
| [[Tasks/Business_Sprint_0_Foundation]] | Business/Product | Foundation Sprint 0 (completed) |
| [[Tasks/iOS_Sprint_1]] | iOS | Sprint 1 (historical) |
| [[Tasks/UI_Navigation_Rescue_Audit]] | iOS/UI | Paused — rescue audit |
| [[Tasks/Backend_Sprint_1]] | Backend | Paused |

---

## Architecture Decisions

| ID | Decision | Status |
|---|---|---|
| [[Decisions/ADR-001_GRDB_vs_SwiftData]] | GRDB вместо SwiftData | ✅ Closed |
| [[Decisions/ADR-002_TCA_vs_MVVM]] | TCA вместо MVVM | ✅ Closed |
| [[Decisions/ADR-003_SwiftUI_vs_UIKit]] | SwiftUI как основной UI фреймворк | ✅ Closed |
| [[Decisions/ADR-005_Subscription_Pivot]] | Pivot from taxi-first MVP to subscription commute platform | ✅ Accepted |

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

- [[Thesis/00_Thesis_Index]] | Дипломный проект: scope, matching, эксперименты | 📝 Черновик
