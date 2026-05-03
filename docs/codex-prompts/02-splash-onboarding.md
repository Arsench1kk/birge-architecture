# Codex Prompt 02 — Splash Screen + Onboarding

## Контекст
Проект: `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger`
Мокапы (для справки, брать идею, не копировать 1в1):
- `P-00-splash.html` — синий градиент, пульсирующие кольца, стеклянная карточка с "BIRGE"
- `P-03-onboarding.html` — 3 шага, иллюстрация, dot-индикаторы, кнопка "Далее"

**Сначала выполни промпт 01 (Liquid Glass design system).**

---

## Задача A — SplashView

Создай файл `BIRGEPassenger/Features/Auth/SplashView.swift`.

**Дизайн (лучше мокапа):**
- Полноэкранный синий радиальный градиент: `BIRGEColors.brandPrimary` → `BIRGEColors.brandPrimary.opacity(0.7)`
- 3 пульсирующих кольца (белые, разные размеры), анимация `easeOut` повторяющаяся
- В центре — Liquid Glass карточка с:
  - SF Symbol `car.2.fill` (или `car.fill`) — крупный, белый, 56pt
  - Текст "BIRGE" — 52pt, bold, белый
  - Текст "Поехали вместе" — subheadline, белый opacity 0.72
- Внизу footer: "© 2026 BIRGE" — caption, белый opacity 0.4
- Через 2.0 сек авто-переход (через `@Environment(\\.dismiss)` или callback)

**TCA Feature:**
```swift
// BIRGEPassenger/Features/Auth/SplashFeature.swift
@Reducer
struct SplashFeature {
    @ObservableState
    struct State: Equatable {
        var hasFinished = false
    }

    enum Action: Sendable {
        case onAppear
        case timerFired
        case delegate(Delegate)

        enum Delegate: Sendable {
            case splashFinished
        }
    }

    var body: some Reducer<State, Action> {
        Reduce { state, action in
            switch action {
            case .onAppear:
                return .run { send in
                    try await Task.sleep(for: .seconds(2.0))
                    await send(.timerFired)
                }
            case .timerFired:
                state.hasFinished = true
                return .send(.delegate(.splashFinished))
            case .delegate:
                return .none
            }
        }
    }
}
```

**View паттерн:**
- Используй `@ViewAction(for: SplashFeature.self)`
- Пульс-кольца: `Circle().fill(.white.opacity(opacity)).scaleEffect(isAnimating ? 2.8 : 1.0).opacity(isAnimating ? 0 : 1).animation(.easeOut(duration: 2.0).repeatForever(autoreverses: false).delay(delay), value: isAnimating)`
- Центральная карточка: `.liquidGlass(.card)` из BIRGECore
- Уважай `@Environment(\.accessibilityReduceMotion)` — при `reduceMotion = true` убрать анимацию колец

---

## Задача B — OnboardingView (3 слайда)

Создай файл `BIRGEPassenger/Features/Auth/OnboardingView.swift`.

**Данные слайдов:**
```swift
struct OnboardingSlide: Identifiable {
    let id: Int
    let symbol: String          // SF Symbol name
    let symbolColor: Color
    let title: String
    let body: String
}

let slides: [OnboardingSlide] = [
    .init(id: 0,
          symbol: "arrow.down.forward.and.arrow.up.backward",
          symbolColor: BIRGEColors.brandPrimary,
          title: "Экономия до 40%",
          body: "Фиксированные маршруты — меньше пробок, меньше цена. Никаких сюрпризов."),
    .init(id: 1,
          symbol: "brain",
          symbolColor: Color.purple,
          title: "AI подбирает маршрут",
          body: "Умный алгоритм анализирует ваши поездки и предлагает лучший коридор."),
    .init(id: 2,
          symbol: "leaf.fill",
          symbolColor: Color.green,
          title: "Меньше машин — чище город",
          body: "Carpooling снижает выбросы CO₂ и разгружает алматинские пробки.")
]
```

**TCA Feature:**
```swift
// BIRGEPassenger/Features/Auth/OnboardingFeature.swift
@Reducer
struct OnboardingFeature {
    @ObservableState
    struct State: Equatable {
        var currentPage: Int = 0
        let totalPages: Int = 3
    }

    enum Action: Sendable {
        case nextTapped
        case skipTapped
        case delegate(Delegate)

        enum Delegate: Sendable {
            case onboardingFinished
        }
    }

    var body: some Reducer<State, Action> {
        Reduce { state, action in
            switch action {
            case .nextTapped:
                if state.currentPage < state.totalPages - 1 {
                    state.currentPage += 1
                    return .none
                }
                return .send(.delegate(.onboardingFinished))
            case .skipTapped:
                return .send(.delegate(.onboardingFinished))
            case .delegate:
                return .none
            }
        }
    }
}
```

**Дизайн View:**
- Фон: белый/`BIRGEColors.background`
- Верхняя часть (40% экрана): большой SF Symbol в центре, на фоне круга с `BIRGEColors.brandPrimary.opacity(0.1)`, размер символа ~80pt
- Нижняя часть:
  - Dot-индикаторы: активная — `Capsule()` 24×8pt синяя, неактивные — `Circle()` 8pt серые
  - Заголовок `title` — 32pt bold
  - Текст `body` — 17pt regular, secondary
- Кнопки внизу:
  - `BIRGEPrimaryButton` "Далее" / "Начать" (на последнем слайде)
  - Ghost-кнопка "Пропустить" (скрыть на последнем слайде)
- Анимация смены слайдов: `TabView(selection: ...)` с `.tabViewStyle(.page(indexDisplayMode: .never))`
  - НЕ показывать нативный page-indicator — делаем свой dot-индикатор
- Свайп → тоже работает (через `TabView`)

---

## Задача C — Встроить в AppFeature

Отредактируй `BIRGEPassenger/App/AppFeature.swift`:

**Добавь состояние `splash` и `onboarding`:**
```swift
@ObservableState
enum State: Equatable {
    case splash(SplashFeature.State)
    case onboarding(OnboardingFeature.State)
    case unauthenticated(OTPFeature.State)
    case authenticated(PassengerAppFeature.State)

    init() {
        self = .splash(SplashFeature.State())
    }
}

@CasePathable
enum Action: Sendable {
    case splash(SplashFeature.Action)
    case onboarding(OnboardingFeature.Action)
    case otp(OTPFeature.Action)
    case passengerApp(PassengerAppFeature.Action)
}
```

**Логика переходов:**
- `.splash(.delegate(.splashFinished))` → проверить Keychain: если токен есть → `.authenticated`, иначе → `.onboarding`
- `.onboarding(.delegate(.onboardingFinished))` → `.unauthenticated(OTPFeature.State())`

**Обнови AppView.swift** — добавь `case .splash`, `case .onboarding` в switch.

---

## Проверка после изменений

```
xcodebuild -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 16 Pro' build 2>&1 | tail -20
```

Ожидаемый результат: `BUILD SUCCEEDED`
