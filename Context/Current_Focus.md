---
last_updated: 2026-05-02
sprint: 1
---

# Current Focus — BIRGE

## ▶ Цель: UI-демо для препода (ЭТАП 1)

### Стратегия
Строим полный UI на mock-данных. Бэкенд не нужен.
TCA Reducers используют .previewValue зависимости.

### Порядок экранов
1. OTP Auth экран (телефон → код → войти)
2. Home экран пассажира (карта + кнопка "Вызвать")
3. Экран поиска водителя (анимация поиска)
4. Экран активной поездки (водитель едет, ETA)
5. Экран завершения (рейтинг)
6. Driver Dashboard (отдельный target)

### Last session (2026-04-27)
- Task: Full MVP complete — Passenger + Driver apps
- Screens done: OTP, Home, RideRequest, Searching, ActiveRide, 
  RideComplete, Profile, DriverDashboard, Earnings
- Build: ✅ Clean (both targets)
- Notes: All 5+ MVP features complete. Ready for demo.

### Session summary (2026-05-02)
- Task: Backend Vapor build recovery for Swift 6.3.1 / Vapor 4.x
- Result: `birge-vapor` now builds cleanly with `swift build`
- Verified: WebSocket route handler, Redis async/Future compatibility, JWT middleware path, and async entrypoint lifecycle
- Runtime: `swift run` reaches startup, then stops on local PostgreSQL authentication for user `birge`

### Next session
- Task: Verify local PostgreSQL credentials/env, then continue to BE-V002 Auth hardening
- Blocked by: local DB auth for Vapor startup
- Model: Claude Sonnet 4.6 (Thinking)


- Vault настроен (v5.1)
- BIRGE.code-workspace создан
- RULES.md в корне проекта
