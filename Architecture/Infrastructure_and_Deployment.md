---
status: active
version: v5.0
last_updated: 2026-05-02
phase: 1
---

# [[Infrastructure and Deployment]]

## 1. Containerisation — Swift Vapor Runtime Image

The [[Swift Vapor Modular Monolith]] is built with Swift Package Manager and shipped as a lean Linux runtime image.

```dockerfile
# Stage 1: Build
FROM swift:6.3-jammy AS builder
WORKDIR /app
COPY . .
RUN swift build -c release

# Stage 2: Runtime
FROM ubuntu:22.04
WORKDIR /app
COPY --from=builder /app/.build/release/birge-vapor .
EXPOSE 8080
ENTRYPOINT ["./birge-vapor"]
```

**Why this shape:**
- Matches the current Swift backend toolchain and package layout
- Keeps build/runtime concerns separate for predictable CI images
- Leaves room for Swift runtime dependencies without inventing a stale distroless setup

---

## 2. Kubernetes — K3s on Kazakhstan Servers

**Kazakhstan hosting is mandatory** — data localisation law requires user data to remain on Kazakhstan-hosted infrastructure.

K3s (lightweight Kubernetes) provides the full Kubernetes API at reduced operational overhead — suitable for a Phase 1 team.

### GitOps — ArgoCD

```
Developer pushes code
        │
        ▼
GitHub Actions: tests pass, Docker image built and pushed to registry
        │
        ▼
Kubernetes manifests updated with new image tag
        │
        ▼
ArgoCD detects manifest change
        │
        ▼
ArgoCD reconciles cluster state → rolling deployment
```

No manual `kubectl apply` ever runs in production.

---

## 3. API Gateway — Nginx + Lua

All external traffic routes through Nginx. The Vapor backend should not waste compute cycles on invalid requests.

| Concern | Mechanism |
|---|---|
| TLS termination | TLS 1.3 — plaintext HTTP/2 forwarded internally |
| JWT validation | Lua script validates signature and expiry before passing downstream |
| Rate limiting | Sliding window via [[Redis Infrastructure]] — 60 req/min (passengers), 20 req/min (drivers) |
| WebSocket upgrade | Nginx `proxy_pass` with `Upgrade` header forwarding |

Driver location updates are capped at 20 req/min = 1 update per 3 seconds minimum. The 5-second ping interval provides headroom.

---

## 4. Horizontal Pod Autoscaler (HPA)

Urban mobility demand spikes are sudden — a rain event causes 3x demand in 10 minutes.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: birge-api-hpa
  namespace: birge-production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: birge-api
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
  - type: Pods
    pods:
      metric:
        name: birge_matching_queue_depth  # custom Prometheus metric
      target:
        type: AverageValue
        averageValue: "50"
```

**PgBouncer protection:** As Vapor pods scale out, async request concurrency can still exhaust PostgreSQL's connection limit. PgBouncer in transaction mode lets many concurrent requests share a small stable DB pool safely.

---

## 5. Local Development — Docker Compose

```yaml
services:
  postgres:
    image: postgis/postgis:16-3.4
    environment:
      POSTGRES_DB: birge_dev
      POSTGRES_USER: birge
      POSTGRES_PASSWORD: birge_dev_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U birge -d birge_dev"]
      interval: 5s

  redis:
    image: redis:7-alpine
    command: redis-server --maxmemory 512mb --maxmemory-policy allkeys-lru

  osrm:
    image: osrm/osrm-backend:latest
    command: osrm-routed --algorithm mld /data/kazakhstan.osrm
    volumes: ["./data/osrm:/data"]

  api:
    build: .
    environment:
      DATABASE_URL: postgres://birge:birge_dev_password@postgres:5432/birge_dev
      REDIS_URL: redis://redis:6379
      OSRM_URL: http://osrm:5000
      JWT_SECRET: dev_secret_CHANGE_IN_PRODUCTION
    depends_on:
      postgres: { condition: service_healthy }

  ml-service:
    build: ./ml_service
    environment:
      DATABASE_URL: postgres://birge:birge_dev_password@postgres:5432/birge_dev
```

---

## 6. Observability Stack

| Signal | Tool | Key Dashboards |
|---|---|---|
| Metrics | Prometheus + Grafana | `birge_matching_duration_seconds` p95 SLA, active drivers by H3 zone, corridor occupancy |
| Tracing | OpenTelemetry → Jaeger | Full trace: Nginx → Vapor route → PostgreSQL → Redis → OSRM |
| Logging | Structured backend logs | `ride_id` + `user_id` in every log entry — aggregation without parsing |

A complete ride request generates a trace with spans across: Nginx gateway → Vapor monolith → PostgreSQL queries → Redis GEO search → OSRM ETA calls.

---

## Related Files
- [[Backend_Architecture.md]] — Vapor monolith, PgBouncer, Pub/Sub
- [[Database_Schema_and_Migrations.md]] — schema lifecycle and deployment concerns
- [[Security_and_Authentication.md]] — Nginx JWT validation, rate limiting
- [[Scaling_Roadmap.md]] — HPA thresholds by MAU phase
