---
status: active
version: v5.0
last_updated: 2026-04-21
phase: 1
---

# [[Infrastructure and Deployment]]

## 1. Containerisation — Distroless Docker

The [[Go Modular Monolith]] is compiled into a **distroless static image** via multi-stage Docker build.

```dockerfile
# Stage 1: Build
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
    go build -ldflags="-w -s" -o birge-api ./cmd/server

# Stage 2: Runtime (~12MB final image)
FROM gcr.io/distroless/static-debian12
WORKDIR /app
COPY --from=builder /app/birge-api .
COPY --from=builder /app/migrations ./migrations
USER nonroot:nonroot
EXPOSE 8080
ENTRYPOINT ["./birge-api"]
```

**Why distroless:**
- ~12MB image vs ~200MB Alpine — faster cold starts
- No shell, no package manager → minimal attack surface
- Only the Go binary and migrations — nothing else

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

All external traffic routes through Nginx. The Go backend never wastes compute cycles on invalid requests.

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

**PgBouncer protection:** As Go pods scale out, goroutines could exhaust PostgreSQL's connection limit. PgBouncer in transaction mode allows 5,000 goroutines to share 50 DB connections safely.

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
| Tracing | OpenTelemetry → Jaeger | Full trace: Nginx → Go handler → PostgreSQL → Redis → OSRM |
| Logging | zerolog (structured JSON) | `ride_id` + `user_id` in every log entry — aggregation without parsing |

A complete ride request generates a trace with spans across: Nginx gateway → Go monolith → PostgreSQL queries → Redis GEO search → OSRM ETA calls.

---

## Related Files
- [[Backend_Architecture.md]] — Go monolith, PgBouncer, Pub/Sub
- [[Database_Schema_and_Migrations.md]] — golang-migrate Kubernetes Job
- [[Security_and_Authentication.md]] — Nginx JWT validation, rate limiting
- [[Scaling_Roadmap.md]] — HPA thresholds by MAU phase
