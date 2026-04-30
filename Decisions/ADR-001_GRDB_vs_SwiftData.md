---
id: ADR-001
status: Closed
date: 2026-04-21
deciders: Architecture team
---

# ADR-001 — GRDB вместо SwiftData для iOS локального кэша

## Context
Водители в Алматы часто теряют связь — тоннели, подземные парковки, лифтовые шахты высоток. Водитель должен продолжать накапливать GPS-координаты при потере сети и синхронизировать их после восстановления. Это требует thread-safe фоновых записей без блокировки main thread.

## Decision
**GRDB с WAL mode**, не SwiftData, не CoreData.

## Comparison

| Factor | SwiftData | GRDB |
|---|---|---|
| Фоновые bulk inserts | `@MainActor`-bound — блокирует UI | Thread-safe, explicit WAL mode |
| WAL mode | Неявный, не конфигурируется | Явный, полный контроль |
| Background sync | Медленный, UI-блокирующий | Background `DatabaseQueue` — 100-200 записей без main thread |
| Тестируемость | Сложно мокировать | Полная поддержка in-memory DB |
| Зрелость | iOS 17, сырой API | 10+ лет, production-proven |

## Consequences
- Явный WAL mode: `var config = Configuration(); config.journalMode = .wal`
- Все DB операции на background queue — никогда на `@MainActor`
- In-memory база в тестах: `DatabaseQueue()`
- Код немного более verbose чем SwiftData, но предсказуем

## Related
[[Architecture/iOS_Architecture]] Section 3
