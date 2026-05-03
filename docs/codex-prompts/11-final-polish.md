# Codex Prompt 11 — Final Polish: Animations, Transitions & OTP Upgrade

## Контекст
Проект: `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger`

Финальный промпт — полировка всего приложения.
Выполнять ПОСЛЕДНИМ, после всех предыдущих промптов.

---

## Задача A — OTPView улучшение (Liquid Glass)

Текущий OTPView уже использует `.ultraThinMaterial`, нужно улучшить.

В `OTPView.swift` замени background карточки:
```swift
// Было:
.background(
    RoundedRectangle(cornerRadius: BIRGELayout.radiusL)
        .fill(.ultraThinMaterial)
        .overlay(...)
)

// Стало:
.liquidGlass(.card)
```

Обнови `brandHeader` — убери `car.fill`, добавь `car.2.fill` (или `bolt.car.fill`):
```swift
Image(systemName: "bolt.car.fill")
    .font(BIRGEFonts.verifyCode)
    .foregroundStyle(BIRGEColors.textOnBrand)
```

Добавь анимацию появления карточки:
```swift
.scaleEffect(cardAppeared ? 1.0 : 0.92)
.opacity(cardAppeared ? 1.0 : 0)
.animation(.spring(response: 0.5, dampingFraction: 0.75), value: cardAppeared)
.onAppear { cardAppeared = true }
```
Добавь `@State private var cardAppeared = false` в OTPView.

---

## Задача B — Глобальные анимации переходов

В `AppView.swift` (или `PassengerAppView.swift`) добавь плавные transitions:

```swift
// При смене State в AppFeature (splash → onboarding → otp → app)
.transition(.asymmetric(
    insertion: .opacity.combined(with: .scale(scale: 1.05)),
    removal: .opacity.combined(with: .scale(scale: 0.95))
))
.animation(.easeInOut(duration: 0.35), value: store.state)
```

---

## Задача C — Улучшить BIRGEPrimaryButton (Liquid Glass стиль)

В `BIRGECore/Sources/BIRGECore/DesignSystem/BIRGEModifiers.swift`:

Обнови `BIRGEPrimaryButton.body`:
```swift
public var body: some View {
    Button(action: action) {
        Group {
            if isLoading {
                HStack(spacing: 8) {
                    ProgressView()
                        .tint(BIRGEColors.textOnBrand)
                    Text("Загрузка...")
                        .font(BIRGEFonts.bodyMedium)
                }
            } else {
                Text(title)
                    .font(BIRGEFonts.bodyMedium)
            }
        }
        .foregroundStyle(BIRGEColors.textOnBrand)
        .frame(maxWidth: .infinity)
        .frame(height: 54)
        .background(
            Group {
                if #available(iOS 26, *) {
                    BIRGEColors.brandPrimary
                        .glassEffect(.regular.tinted(BIRGEColors.brandPrimary))
                } else {
                    BIRGEColors.brandPrimary
                }
            }
        )
        .clipShape(RoundedRectangle(cornerRadius: BIRGELayout.radiusM))
        .shadow(color: BIRGEColors.brandPrimary.opacity(0.35), radius: 12, y: 6)
    }
    .buttonStyle(PressableButtonStyle())
    .disabled(isLoading)
    .opacity(isLoading ? 0.72 : 1)
}
```

Добавь `PressableButtonStyle` (пружинный press эффект):
```swift
struct PressableButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .scaleEffect(configuration.isPressed ? 0.97 : 1.0)
            .animation(.spring(response: 0.2, dampingFraction: 0.6), value: configuration.isPressed)
    }
}
```

---

## Задача D — Searching View улучшение

В `SearchingView.swift` добавь пульсирующие анимации более агрессивно и добавь статус-текст:

Замени центральный контент:
```swift
VStack(spacing: BIRGELayout.l) {
    // RADAR
    ZStack {
        radarRings
        centralIcon
    }
    .frame(width: 200, height: 200)

    // STATUS
    VStack(spacing: BIRGELayout.xxs) {
        Text("Ищем водителя")
            .font(BIRGEFonts.title)
            .foregroundStyle(BIRGEColors.textOnBrand)

        Text(store.statusText)
            .font(BIRGEFonts.body)
            .foregroundStyle(BIRGEColors.textOnBrand.opacity(0.72))
            .multilineTextAlignment(.center)
            .animation(.easeInOut, value: store.statusText)

        // Live counter (время поиска)
        Text("Поиск: \(store.searchDuration ?? 0) сек")
            .font(BIRGEFonts.caption)
            .foregroundStyle(BIRGEColors.textOnBrand.opacity(0.5))
    }
    .padding(.horizontal, BIRGELayout.l)
}
```

Убедись что `store.searchDuration` есть в `SearchingFeature.State`, если нет — добавь:
```swift
// SearchingFeature.State:
var searchDuration: Int? = nil  // секунды поиска
```

И обновляй каждую секунду в Effect (если не реализовано):
```swift
// В SearchingFeature body, при .onAppear:
case .onAppear:
    return .merge(
        // WebSocket subscription (existing)
        subscribeToWebSocket(rideId: state.rideId),
        // Timer
        .run { send in
            var seconds = 0
            while true {
                try await Task.sleep(for: .seconds(1))
                seconds += 1
                await send(.timerTick(seconds))
            }
        }
    )
case .timerTick(let seconds):
    state.searchDuration = seconds
    return .none
```

---

## Задача E — Улучшить BIRGESheetHandle (Liquid Glass)

В `BIRGECore/Sources/BIRGECore/DesignSystem/BIRGEModifiers.swift`:
```swift
public struct BIRGESheetHandle: View {
    public init() {}

    public var body: some View {
        Capsule()
            .fill(BIRGEColors.textTertiary.opacity(0.5))
            .frame(
                width: BIRGELayout.sheetHandleWidth,
                height: BIRGELayout.sheetHandleHeight
            )
    }
}
```

---

## Задача F — CLAUDE.md обновление

Обнови `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger/CLAUDE.md`:

```markdown
# BIRGE — Agent Instructions

## Vault location
All architecture decisions, context and tasks are in:
../../../../Documents/Dev_ArsensBrain/Projects/Birge Taxi/

## Before every task read:
- ../../../../Documents/Dev_ArsensBrain/Projects/Birge Taxi/CLAUDE.md
- ../../../../Documents/Dev_ArsensBrain/Projects/Birge Taxi/Context/Current_Focus.md
- ../../../../Documents/Dev_ArsensBrain/Projects/Birge Taxi/Context/iOS_Agent_Context.md

## Key files:
- Tasks: ../../../../Documents/Dev_ArsensBrain/Projects/Birge Taxi/Tasks/iOS_Sprint_1.md
- Architecture: ../../../../Documents/Dev_ArsensBrain/Projects/Birge Taxi/Architecture/
- Decisions: ../../../../Documents/Dev_ArsensBrain/Projects/Birge Taxi/Decisions/
- Mockups: ../../../../Documents/Dev_ArsensBrain/Projects/Birge Taxi/docs/mockups/

## Design rules:
1. ALWAYS use `BIRGEColors`, `BIRGEFonts`, `BIRGELayout` from BIRGECore
2. Use `.liquidGlass()` modifier for all cards and sheets
3. Use `.birgeGlassCard()` for content cards
4. Use `BIRGEGlassSheet {}` for all bottom sheets
5. Replace ALL emoji with SF Symbols
6. TCA pattern: @ViewAction(for:) + @Bindable var store
7. Build check: xcodebuild -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 16 Pro' build

## Screen mapping (mockup → Swift file):
- P-00-splash → Features/Auth/SplashView.swift
- P-01-phone + P-02-otp → Features/Auth/OTPView.swift
- P-03-onboarding → Features/Auth/OnboardingView.swift
- P-04-home → Features/Home/HomeView.swift
- P-05-search + ride-request → Features/RideRequest/RideRequestView.swift
- P-06a-corridor-list → Features/Corridor/CorridorListView.swift
- P-06b-corridor-detail → Features/Corridor/CorridorDetailView.swift
- P-08-offer → Features/Searching/OfferView.swift
- P-09 + P-10 + P-11 → Features/ActiveRide/ActiveRideView.swift
- P-12-ride-complete → Features/RideComplete/RideCompleteView.swift
- P-13-boarding-code → Features/ActiveRide/BoardingCodeView.swift
- P-14-rides-history → Features/RidesHistory/RidesHistoryView.swift
- P-15-ride-detail → Features/RidesHistory/RideDetailView.swift
- P-17-subscriptions → Features/Subscriptions/SubscriptionsView.swift
- P-19-profile → Features/Profile/ProfileView.swift
- P-23-ai-explanation → Features/Profile/AIExplanationView.swift
```

---

## Финальная проверка

```bash
# Сборка
xcodebuild -scheme BIRGEPassenger \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
  build 2>&1 | tail -40

# Тесты
xcodebuild -scheme BIRGEPassenger \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
  test 2>&1 | tail -20
```

Ожидаемый результат: `BUILD SUCCEEDED` + `TEST SUCCEEDED`

---

## Итоговый чеклист

После выполнения всех 11 промптов приложение должно иметь:

- ✅ Splash с пульсирующими кольцами и Liquid Glass карточкой
- ✅ Onboarding — 3 слайда, SF Symbols, TabView анимация
- ✅ OTP Auth — Liquid Glass card, плавные transitions
- ✅ Home — Map + Glass search bar + AI pill + горизонтальный скролл коридоров + Quick Taxi
- ✅ Tab Bar — 4 вкладки, SF Symbols, встроен в Home sheet
- ✅ RideRequest — Map background + Glass sheet + Glass tier cards
- ✅ CorridorList — Filter pills + Glass corridor cards
- ✅ CorridorDetail — Map + Glass sheet + Join CTA
- ✅ Searching — Radar анимация + live timer
- ✅ Offer Screen — принятие/отклонение водителя
- ✅ ActiveRide — Liquid Glass sheet + SF Symbol status pill + Boarding code
- ✅ BoardingCode — QR/code sheet
- ✅ RideComplete — Glass summary card + улучшенная анимация
- ✅ RidesHistory — Filter + Glass ride cards
- ✅ RideDetail — Map stub + Glass info card
- ✅ Profile — Glass header + stat cards + iOS-style settings rows
- ✅ AIExplanation — Step cards
- ✅ Subscriptions — Plan cards (glassmorphism) + billing toggle
