# BIRGE Passenger — Codex Prompt Series
## Обзор и навигация

Это серия промптов для Codex, которые нужно выполнять **последовательно**, по одному за раз.
Каждый промпт охватывает чётко ограниченный объём работы чтобы не перегружать контекст.

---

## Состояние проекта (на 03.05.2026)

### ✅ Реализовано (BIRGEPassenger iOS app)
| Экран | Файл | Качество |
|-------|------|----------|
| OTP Auth (Phone + Code) | `Features/Auth/OTPView.swift` | Хорошее, есть `.ultraThinMaterial`, нет Liquid Glass |
| Home | `Features/Home/HomeView.swift` | Базовое — нет Liquid Glass, нет AI pill, нет scroll-коридоров |
| RideRequest | `Features/RideRequest/RideRequestView.swift` | Базовое — нет glassmorphism на карточках tier |
| Searching | `Features/Searching/SearchingView.swift` | Хорошее — radar animation есть |
| ActiveRide | `Features/ActiveRide/ActiveRideView.swift` | Базовое — нет Liquid Glass sheet |
| RideComplete | `Features/RideComplete/RideCompleteView.swift` | Хорошее — нет Liquid Glass |
| Profile | `Features/Profile/ProfileView.swift` | Базовое — старый List стиль |

### ❌ Не реализовано (нужно создавать с нуля)
| Экран | Мокап | Приоритет |
|-------|-------|-----------|
| Splash Screen | `P-00-splash.html` | P0 |
| Onboarding (3 шага) | `P-03-onboarding.html` | P0 |
| Corridor List | `P-06a-corridor-list.html` | P1 |
| Corridor Detail | `P-06b-corridor-detail.html` | P1 |
| Offer/Match Screen | `P-08-offer.html` | P1 |
| Boarding Code | `P-13-boarding-code.html` | P1 |
| Rides History | `P-14-rides-history.html` | P2 |
| Ride Detail | `P-15-ride-detail.html` | P2 |
| Subscriptions | `P-17-subscriptions.html` | P2 |
| Tab Bar Navigation | — | P0 |

### 🔧 Нужен рефакторинг (под Liquid Glass)
| Файл | Что менять |
|------|-----------|
| `BIRGECore/DesignSystem/BIRGEModifiers.swift` | Добавить `.liquidGlass()` modifier |
| `Features/Home/HomeView.swift` | Bottom sheet → Liquid Glass, AI pill, горизонтальный скролл коридоров |
| `Features/ActiveRide/ActiveRideView.swift` | Bottom sheet → Liquid Glass |
| `Features/RideComplete/RideCompleteView.swift` | Cards → Liquid Glass |
| `Features/Profile/ProfileView.swift` | Полный редизайн под мокап P-19 |

---

## Ключевые правила для всех промптов

### Liquid Glass в SwiftUI (iOS 26+)
```swift
// Способ 1 — .glassEffect() (iOS 26 нативный API)
.glassEffect(.regular.tinted(BIRGEColors.brandPrimary.opacity(0.08)))

// Способ 2 — .ultraThinMaterial (iOS 15+, fallback)
.background(.ultraThinMaterial)
.overlay(RoundedRectangle(cornerRadius: 20).stroke(.white.opacity(0.2), lineWidth: 1))

// Способ 3 — Custom LiquidGlass modifier в BIRGEModifiers
struct BIRGEGlassCard: ViewModifier { ... }
```

### SF Symbols вместо эмодзи
| Было (эмодзи) | Стало (SF Symbol) |
|---------------|-------------------|
| 🔍 | `magnifyingglass` |
| 🚗 / 🚙 | `car.fill` |
| ✨ | `sparkles` |
| 👤 | `person.fill` |
| 🗺 | `map.fill` |
| 📍 | `mappin.and.ellipse` |
| ⭐ | `star.fill` |
| 📋 | `list.bullet.clipboard` |
| 💳 | `creditcard.fill` |
| 🔔 | `bell.fill` |
| 💬 | `message.fill` |
| ☎️ | `phone.fill` |

### Архитектурные правила
- Всегда использовать `BIRGEColors`, `BIRGEFonts`, `BIRGELayout` из `BIRGECore`
- TCA: `@ViewAction(for: Feature.self)` + `@Bindable var store`
- `@Reducer` + `@ObservableState` + `@CasePathable` на enum Action
- Делегат-паттерн для навигации между экранами

---

## Порядок выполнения промптов

1. `01-design-system-liquid-glass.md` — Добавить Liquid Glass в BIRGECore
2. `02-splash-onboarding.md` — Splash + Onboarding экраны
3. `03-home-screen-refactor.md` — Рефакторинг Home под мокап
4. `04-ride-request-refactor.md` — Рефакторинг RideRequest
5. `05-searching-offer-screens.md` — Searching (улучшение) + Offer экран
6. `06-active-ride-refactor.md` — ActiveRide + BoardingCode
7. `07-ride-complete-refactor.md` — RideComplete улучшение
8. `08-corridor-screens.md` — CorridorList + CorridorDetail
9. `09-rides-history-detail.md` — RidesHistory + RideDetail
10. `10-profile-settings-refactor.md` — Profile полный редизайн
11. `11-subscriptions-screen.md` — Subscriptions экран
12. `12-tab-bar-navigation.md` — Tab Bar в Home
