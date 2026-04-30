---
last_updated: 2026-04-22
status: draft
---

# OpenAPI Contracts — BIRGE

> Source of truth для iOS ↔ Go интеграции.
> Полный spec в `birge/api/openapi.yaml`

## Auth Endpoints

### POST /api/v1/auth/otp/request
```yaml
requestBody:
  phone: "+77771234567"  # E.164 format
responses:
  200: { message: "OTP sent" }
  429: { error_code: "RATE_LIMITED", retry_after: 60 }
```

### POST /api/v1/auth/otp/verify
```yaml
requestBody:
  phone: "+77771234567"
  code: "123456"
responses:
  200:
    access_token: "eyJ..."    # 60 min
    refresh_token: "eyJ..."   # 30 days
    user_id: "uuid"
    role: "passenger" | "driver"
  400: { error_code: "INVALID_OTP" }
  410: { error_code: "OTP_EXPIRED" }
```

### POST /api/v1/auth/refresh
```yaml
requestBody:
  refresh_token: "eyJ..."
responses:
  200:
    access_token: "eyJ..."
  401: { error_code: "INVALID_REFRESH_TOKEN" }
```

## Ride Endpoints

### POST /api/v1/rides
```yaml
headers:
  Authorization: "Bearer <access_token>"
requestBody:
  origin: { latitude: 43.238, longitude: 76.945 }
  destination: { latitude: 43.262, longitude: 76.912 }
  tier: 1  # 1=on_demand, 2=shared_corridor, 3=subscription
responses:
  201:
    ride_id: "uuid"
    status: "requested"
    estimated_wait_seconds: 180
```

### WebSocket: wss://api.birge.kz/ws

```
Connection headers:
  Authorization: Bearer <access_token>

Client → Server messages:
  { type: "subscribe", ride_id: "uuid" }
  { type: "ping" }
  { type: "location_update", latitude: 43.238, longitude: 76.945, accuracy: 5.0 }

Server → Client messages:
  { type: "pong" }
  { type: "ride_matched", ride_id: "uuid", driver: { name, rating, vehicle } }
  { type: "driver_location", latitude, longitude, eta_seconds }
  { type: "ride_status_changed", status: "driver_arriving" }
  { type: "eta_updated", eta_seconds: 240 }
```

## Error Format (все endpoints)
```json
{
  "error_code": "INVALID_OTP",
  "message": "The OTP code is invalid or expired",
  "request_id": "trace-id-for-support"
}
```

## iOS Generated Types
> Типы генерируются из OpenAPI spec через `swift-openapi-generator`
> Файл: `BIRGECore/Sources/APITypes/Generated.swift`
