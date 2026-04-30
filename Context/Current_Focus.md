---
last_updated: 2026-04-27
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

### Next session
- Task: BE-vapor-001 — verify docker compose up locally,
  then connect iOS AuthClient to real Vapor endpoints
- Starting prompt: "Read RULES.md and Context/Backend_Agent_Context.md.
  Task: Connect iOS app to Vapor backend..."
- Blocked by: swift build needs GitHub access (run locally on Mac)


- Vault настроен (v5.1)
- BIRGE.code-workspace создан
- RULES.md в корне проекта