---
last_updated: 2026-05-02
status: draft
---

# OpenAPI Contracts — BIRGE

> Source of truth для iOS ↔ backend интеграции.
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

### GET /api/v1/rides/{ride_id}
```yaml
headers:
  Authorization: "Bearer <access_token>"
responses:
  200:
    ride_id: "uuid"  # backend may currently return `id`
    status: "requested" | "matched" | "driver_accepted" | "driver_arriving" | "passenger_wait" | "in_progress" | "completed" | "cancelled"
    driver_name: "Азамат К."        # optional
    driver_rating: 4.92             # optional
    driver_vehicle: "Toyota Camry"  # optional
    driver_plate: "777 AAA 02"      # optional
    eta_seconds: 180                # optional
    verification_code: "1234"       # optional
    pickup_latitude: 43.238         # optional; backend may currently return `originLat`
    pickup_longitude: 76.945        # optional; backend may currently return `originLng`
```

### PATCH /api/v1/rides/{ride_id}/cancel
```yaml
headers:
  Authorization: "Bearer <access_token>"
requestBody:
  reason: "Пассажир отменил"
responses:
  200:
    ride_id: "uuid"
    status: "cancelled"
```

### POST /api/v1/locations/bulk
```yaml
headers:
  Authorization: "Bearer <access_token>"
requestBody:
  ride_id: "uuid"
  records:
    - latitude: 43.238
      longitude: 76.945
      timestamp: 1715620000.0
      accuracy: 5.0
responses:
  200: { message: "Locations synced", count: 200 }
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

## iOS Client Implementation Status
Updated: 2026-05-02

- `BIRGECore/Sources/BIRGECore/Network/APIClient.swift` implements Phase 1 HTTP calls with `URLSession`.
- `BIRGECore/Sources/BIRGECore/Network/TokenRefreshClient.swift` stores access token in memory and refresh token in Keychain.
- Authenticated calls add `Authorization: Bearer <access_token>`.
- On `401`, the client refreshes once through `POST /auth/refresh`, retries the original request, and clears tokens if the retry is still unauthorized.
- `BIRGEAPIError` decodes `{ error_code, message, request_id }`.
- `POST /locations/bulk` is wired from `LocationSyncService`.
- `xcodebuild build -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 17 Pro' -quiet` passes.
- Full `xcodebuild test` is blocked by the known `SwiftNavigation` / `CasePathsCore` linker failure before tests run.

## iOS Generated Types
> Типы генерируются из OpenAPI spec через `swift-openapi-generator`
> Файл: `BIRGECore/Sources/APITypes/Generated.swift`
