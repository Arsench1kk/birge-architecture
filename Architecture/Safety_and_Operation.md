---
status: active
version: v5.0
last_updated: 2026-04-21
phase: 1
---

# [[Safety and Operations Architecture]]

## 1. Driver Risk Scoring

Beyond initial KYC, the backend continuously computes a composite **Driver Risk Score** (0–100, higher = safer).

| Metric | Description | Weight |
|---|---|---|
| Passenger Rating Score | Average rating from completed rides | High |
| Cancellation Pattern Score | Cancellation rate and timing patterns | High |
| Complaint Resolution Rate | Resolved vs unresolved passenger complaints | Medium |
| Vehicle Condition Score | Periodic vehicle inspection results | Medium |
| Speed Compliance Score | % of time below 80 km/h; acceleration events >0.4g from GPS telemetry | High |

### Automated Enforcement

| Score | Action |
|---|---|
| < 60 | Probationary monitoring — increased review frequency |
| < 20 | Automatic and permanent platform removal |

Risk score is recalculated after every completed ride. Speed Compliance uses the 5-second GPS stream from the [[WebSocket Hub Architecture]].

---

## 2. In-Ride Safety Mechanisms

### Route Deviation Monitor

The backend continuously compares vehicle GPS against the expected corridor route polygon.

| Deviation | Duration | Response |
|---|---|---|
| >500m from planned route | >90 seconds | Safety alert sent to passenger iOS app — one-tap emergency contact button appears |
| >500m from planned route | >3 minutes | Automatic escalation to on-call operations staff |

### Geofencing

Predefined exclusion zones (high-crime districts, restricted areas) are monitored in real-time. Any boundary breach immediately flags the ride for operations review.

### Multi-Passenger Boarding Verification

For shared corridors, a **4-digit verification code** is generated in the passenger's iOS app. The driver must confirm the code before departure — prevents unauthorised boarding.

### Live Location Sharing

Passengers can generate a **temporary tracking URL** (time-limited, read-only) to share with external contacts via mobile browser. Displays: real-time driver location, driver name, vehicle details, license plate.

---

## 3. Operations Dashboard

A React + TypeScript web application for internal platform management.

**Architecture notes:**
- Communicates via a **dedicated internal API** — separated from the passenger/driver API gateway to preserve rate-limiting budgets and enable SSO for staff
- Does not share the public Nginx gateway — internal network only

### Live Corridor Map

Polls [[Data_Pipeline_and_Analytics|ClickHouse]] every 3 seconds to display:
- Active vehicle positions
- Color-coded occupancy indicators per corridor
- Real-time incident flags

### Corridor Lifecycle Management

Operations staff manage corridors through strict lifecycle states:

```
proposed → validated → active → suspended → archived
```

A **nightly batch job** computes a composite performance score for each corridor:
- Demand utilisation rate
- On-time performance
- Driver retention on the corridor

Corridors falling below threshold automatically generate review tickets.

### Corridor Cold-Start Protocol

New corridors are only launched after:
1. 30+ days of confirmed demand via manual pilot
2. Operations validation of pickup node placement
3. Minimum 3 confirmed drivers on the corridor

If a scheduled corridor run has fewer than **2 confirmed passengers** 60 minutes before departure, the [[Ride State Machine]] automatically converts all bookings to standard on-demand dispatches. Passengers are notified; no service failure occurs.

---

## Related Files
- [[Security_and_Authentication.md]] — driver KYC pipeline
- [[Ride_State_Machine.md]] — corridor-to-on-demand fallback transition
- [[WebSocket_Hub_Architecture.md]] — GPS stream for speed compliance scoring
- [[Geo_Matching_and_AI.md]] — DBSCAN corridor detection
- [[Data_Pipeline_and_Analytics.md]] — ClickHouse feed for operations dashboard
