---
id: ADR-003
status: Closed
date: 2026-04-22
deciders: Architecture team
---

# ADR-003 — SwiftUI как основной UI фреймворк

## Context
Проект BIRGE начинается с нуля в 2026 году. Нет legacy UIKit кода. Нужен выбор: SwiftUI или UIKit?

## Decision
**SwiftUI** как основной фреймворк. UIKit только там, где SwiftUI не покрывает (Custom MapKit overlays, Camera для KYC).

## Analysis

### Почему SwiftUI подходит для BIRGE в 2026

**TCA интеграция.** TCA изначально построена на SwiftUI (`@ObservableState`, `WithViewStore`, `NavigationStack`). Использование UIKit с TCA требует `UIHostingController` обёрток — дополнительный слой сложности без выгоды.

**Deployment target iOS 17.** iOS 17 SwiftUI зрелый — нет серьёзных багов, все нужные компоненты есть: `NavigationStack`, `Map`, `Sheet`, `async image loading`.

**Скорость разработки.** Greenfield проект с AI-ассистентом (Antigravity) — SwiftUI генерируется быстрее и читается лучше. Меньше boilerplate = больше бизнес-логики.

**Нет UIKit-heavy требований.** BIRGE не нужны: сложные custom transitions, UICollectionView с diffable data source, видео редактирование. Всё что нужно — Map, List, Form, custom overlays.

### Где UIKit всё же нужен
- `MKMapView` с custom driver overlays (SwiftUI `Map` пока ограничен в кастомизации)
- `AVCaptureSession` для KYC (фото документов)
- Оба случая: `UIViewRepresentable` обёртка

### Почему НЕ переходить на UIKit
- **Нет legacy кода** — нет причин
- **UIKit + TCA** — возможно, но awkward: `UIViewController` не интегрируется с `@ObservableState`
- **2026 год** — Apple активно инвестирует в SwiftUI, UIKit получает минимальные обновления
- **Hiring** — junior iOS разработчики 2025-2026 знают SwiftUI лучше UIKit

## Consequences
- Основной фреймворк: SwiftUI
- TCA + `@ObservableState` — полная совместимость
- MapKit: `UIViewRepresentable(MKMapView)` для кастомных overlays
- KYC Camera: `UIViewRepresentable(AVCaptureSession)`
- Минимальная зависимость от UIKit — только точечно

## Related
[[Architecture/iOS_Architecture]]
[[Decisions/ADR-002_TCA_vs_MVVM]]
