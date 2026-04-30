---
id: ADR-002
status: Closed
date: 2026-04-21
deciders: Architecture team
---

# ADR-002 — TCA вместо MVVM для iOS

## Context
Приложение управляет сложным, одновременным async state: GPS-стрим, WebSocket события в реальном времени, фоновый трекинг локации, падения сети и переподключения, общий state на нескольких экранах. Стандартный MVVM с десятками ViewModels ведёт к race conditions и непредсказуемому state.

## Decision
**The Composable Architecture (TCA)** от Point-Free, не MVVM.

## Why TCA Wins for BIRGE

| Requirement | MVVM | TCA |
|---|---|---|
| GPS stream (background) | Нет стандартного решения | `Effect.run` + `AsyncStream` |
| WebSocket lifecycle | Ручное управление | `.cancellable(id:)` |
| Testability | Тяжело мокировать | `DependencyValues` — полная инъекция |
| State consistency | ViewModel Hell | Единое state дерево |
| SwiftUI Previews | `#if DEBUG` везде | Mock dependencies из коробки |

## Consequences
- Boilerplate выше чем MVVM на маленьких фичах
- Кривая обучения для новых iOS разработчиков
- Зато: нулевые race conditions в async state
- Зато: unit тесты без симулятора, без сети

## Related
[[Architecture/iOS_Architecture]] Section 2
