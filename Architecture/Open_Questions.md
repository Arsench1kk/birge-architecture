---
status: living
last_updated: 2026-05-07
---

# Open Questions & Unresolved Decisions

> This file tracks decisions that have not been finalised. Update when resolved — move closed items to the relevant domain file.

---

## 🔴 High Priority (blocks Phase 1)

### BQ-001 — Subscription Commute Validation
- **Question:** Do real Almaty commuters accept the core behavior change: fixed commute windows, hub pickup/dropoff, and monthly subscription instead of daily taxi ordering?
- **Context:** The 2026-05-07 foundation makes subscription commute the wedge. UI/tech should not scale before this is validated.
- **Impact:** [[Business_Department/Product_Logic]], passenger onboarding, subscription screens
- **Owner:** Arsen + Business Partner AI chat
- **Status:** Open

### BQ-002 — Driver Subscription Willingness
- **Question:** Will real Almaty drivers accept 19,000 / 28,000 / 38,000 KZT monthly subscription with zero per-ride commission?
- **Context:** This is the core driver monetization model and must be validated before driver launch.
- **Impact:** [[Business_Department/Driver_Economics]], DriverRegistration UI, pricing copy
- **Owner:** Arsen
- **Status:** Open

### BQ-003 — Shared Ride Legal Status In Kazakhstan
- **Question:** Does paid hub-to-hub shared commuting require a marshrutka/taxi/transport license category beyond normal ride-hailing?
- **Context:** This may block real launch or change the operating model.
- **Impact:** Product model, contracts, safety/legal, Kaspi merchant setup
- **Owner:** Arsen
- **Status:** Open — transport lawyer consultation required

### BQ-004 — UvU Shuttle Competitive Check
- **Question:** Does UvU Shuttle already serve the first corridor or similar Tier 3 commute use cases?
- **Context:** UvU may be the closest direct competitor to subscription commute.
- **Impact:** Positioning, pricing, first corridor choice
- **Owner:** Arsen
- **Status:** Open — download/test app

---

### OQ-001 — Push Notifications for Matched Driver Offer
- **Question:** APNs directly from the backend, or via a managed service (Firebase FCM as delivery layer)?
- **Context:** APNs requires certificate/key management; FCM adds a dependency but simplifies iOS delivery and supports Android if needed later.
- **Impact:** future `Sources/App/Modules/Notifications/` module and iOS notification handling
- **Owner:** —
- **Status:** Open

### OQ-002 — Kaspi Webhook Idempotency Key Format
- **Question:** Using `kaspi_transaction_id` as the idempotency key in `payment_records`. Is this guaranteed unique across Kaspi's system, or do we need a composite key with merchant ID?
- **Context:** Duplicate payment events must never double-credit a driver's balance.
- **Impact:** [[Payment_and_Financial_Architecture.md]] — `ON CONFLICT` constraint
- **Owner:** —
- **Status:** Open

### OQ-003 — Corridor Failure Fallback Logic Owner
- **Question:** Who owns the 60-minute pre-departure check that converts a low-occupancy corridor run to on-demand? The `corridors` service or the `rides` service?
- **Context:** This is a [[Ride State Machine]] concern but requires corridor subscription data.
- **Impact:** Service boundary in [[Backend_Architecture.md]]
- **Owner:** —
- **Status:** Open

---

## 🟡 Medium Priority (Phase 1 but not launch-blocking)

### OQ-004 — OSRM Kazakhstan Map Update Cadence
- **Question:** How often do we refresh the Kazakhstan OSM extract for OSRM? Manual trigger or automated monthly?
- **Context:** Road network changes affect ETA accuracy. Stale OSRM data = wrong matching scores.
- **Impact:** [[Infrastructure_and_Deployment.md]] — `osrm` Docker service
- **Status:** Open

### OQ-005 — Driver KYC Score Threshold Governance
- **Question:** The automated facial similarity model flags cases below 85% confidence for manual review. Who sets and adjusts this threshold? Operations team or hardcoded?
- **Context:** Too strict = high manual review burden at launch. Too loose = safety risk.
- **Impact:** [[Security_and_Authentication.md]] — KYC pipeline
- **Status:** Open

### OQ-006 — Redis Cluster vs Single Instance Timing
- **Question:** The [[Scaling_Roadmap.md]] suggests Redis Cluster at 100k MAU. Should we pre-configure Redis Sentinel at launch to make the cluster migration easier?
- **Context:** Sentinel gives HA without sharding complexity. Migrating from standalone → Cluster later requires key redistribution.
- **Status:** Open

---

## 🟢 Low Priority (Phase 2 decisions)

### OQ-007 — ClickHouse Hosting
- **Question:** Self-hosted on K3s or managed ClickHouse Cloud?
- **Context:** Phase 2 analytics. Kazakhstan data localisation law may require self-hosted.
- **Impact:** [[Data_Pipeline_and_Analytics.md]]
- **Status:** Open — decision needed before Phase 2 start

### OQ-008 — MLflow Registry Location
- **Question:** MLflow server on K3s or a managed experiment tracking service?
- **Context:** Nightly retraining jobs need a stable artifact registry.
- **Impact:** [[Geo_Matching_and_AI.md]] — MLOps pipeline
- **Status:** Open — decision needed before Phase 2 start

---

## ✅ Resolved

| ID | Question | Decision | Date |
|---|---|---|---|
| OQ-R01 | Original Go backend proposal | Superseded by OQ-R05; historical only | v5.0 |
| OQ-R02 | SwiftData vs GRDB for iOS local cache | GRDB — thread-safe WAL mode, background bulk inserts without `@MainActor` blocking | v5.0 |
| OQ-R03 | WebSocket horizontal scaling mechanism | Redis Pub/Sub global broadcast bus | v5.0 |
| OQ-R04 | Message broker at launch | PostgreSQL LISTEN/NOTIFY + Transactional Outbox — Kafka deferred to 150k rides/day | v5.0 |
| OQ-R05 | Go vs Vapor | Swift Vapor chosen — single language stack, shared Swift mental model with iOS | 2026-04-27 |
