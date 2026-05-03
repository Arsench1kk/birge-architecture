# Codex Prompt 04 — RideRequest Refactor + Tab Bar

## Контекст
Проект: `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger`
Файлы:
- `BIRGEPassenger/Features/RideRequest/RideRequestView.swift`
- `BIRGEPassenger/App/PassengerAppView.swift`

Мокапы для вдохновения: `P-05-search.html`, `ride-request.html`

**Требование: выполни промпт 03 сначала.**

---

## Задача A — RideRequest полный редизайн

Текущий `RideRequestView.swift` имеет базовую вёрстку. Нужно добавить:
- Liquid Glass на tier cards
- Полноэкранный map background
- Glass bottom sheet с контентом

### Новая структура:
```swift
ZStack {
    // LAYER 0: Map background
    Map(position: $position)
        .mapStyle(.standard(elevation: .realistic))
        .mapControls { }
        .ignoresSafeArea()

    // LAYER 1: Floating back button (top-left)
    floatingBackButton
}
.safeAreaInset(edge: .bottom) {
    rideRequestSheet
}
.navigationBarHidden(true)
```

### floatingBackButton
```swift
VStack {
    HStack {
        Button { send(.backTapped) } label: {
            Image(systemName: "chevron.left")
                .font(BIRGEFonts.bodyMedium)
                .foregroundStyle(BIRGEColors.textPrimary)
                .frame(width: 44, height: 44)
                .liquidGlass(.card)
        }
        .accessibilityLabel("Назад")
        .padding(.leading, BIRGELayout.s)
        .padding(.top, BIRGELayout.mapSearchBarTop)
        Spacer()
    }
    Spacer()
}
```

### rideRequestSheet (Glass bottom sheet)

Используй `BIRGEGlassSheet`:
```swift
var rideRequestSheet: some View {
    BIRGEGlassSheet {
        VStack(spacing: BIRGELayout.s) {
            // ЗАГОЛОВОК
            Text("Новая поездка")
                .font(BIRGEFonts.title)
                .frame(maxWidth: .infinity, alignment: .leading)
                .padding(.horizontal, BIRGELayout.m)
                .padding(.top, BIRGELayout.xs)

            // АДРЕСНАЯ СЕКЦИЯ
            addressSection
                .padding(.horizontal, BIRGELayout.m)

            // ВЫБОР ТАРИФА
            Text("Выберите тариф")
                .font(BIRGEFonts.captionBold)
                .foregroundStyle(BIRGEColors.textSecondary)
                .frame(maxWidth: .infinity, alignment: .leading)
                .padding(.horizontal, BIRGELayout.m)

            tierScroll

            // МАРШРУТ ИНФО
            routeSummaryPill

            // ОШИБКА
            if let error = store.errorMessage {
                BIRGEToast(message: error, style: .error)
                    .padding(.horizontal, BIRGELayout.m)
            }

            // CTA BUTTON
            BIRGEPrimaryButton(
                title: store.isLoading ? "Создаём поездку..." : "Найти водителя · \(store.fare)₸",
                isLoading: store.isLoading
            ) { send(.findDriverTapped) }
            .disabled(store.isLoading)
            .padding(.horizontal, BIRGELayout.m)
            .padding(.bottom, BIRGELayout.m)
        }
    }
}
```

### addressSection

```swift
var addressSection: some View {
    HStack(alignment: .center, spacing: BIRGELayout.xs) {
        // Иконки маршрута
        VStack(spacing: 0) {
            Circle()
                .fill(BIRGEColors.success)
                .frame(width: 10, height: 10)
            Rectangle()
                .fill(BIRGEColors.textTertiary.opacity(0.4))
                .frame(width: 1.5, height: 36)
                .padding(.vertical, 2)
            Image(systemName: "mappin.circle.fill")
                .foregroundStyle(BIRGEColors.brandPrimary)
                .font(.system(size: 14))
        }

        VStack(alignment: .leading, spacing: BIRGELayout.xs) {
            // Откуда
            Text(store.origin)
                .font(BIRGEFonts.caption)
                .foregroundStyle(BIRGEColors.textSecondary)

            Divider()

            // Куда
            TextField("Куда едем?", text: destinationBinding)
                .font(BIRGEFonts.bodyMedium)
                .foregroundStyle(BIRGEColors.textPrimary)
        }
    }
    .padding(BIRGELayout.s)
    .birgeGlassCard()
}
```

### tierScroll — горизонтальный скролл тарифов

```swift
ScrollView(.horizontal, showsIndicators: false) {
    HStack(spacing: BIRGELayout.xs) {
        ForEach(RideRequestFeature.RideTier.allCases, id: \.self) { tier in
            tierCard(for: tier)
        }
    }
    .padding(.horizontal, BIRGELayout.m)
}
```

**Обнови `tierCard`** — использовать `liquidGlass`:
```swift
func tierCard(for tier: RideRequestFeature.RideTier) -> some View {
    let isSelected = store.selectedTier == tier
    Button {
        withAnimation(.spring(response: 0.3)) {
            _ = send(.tierSelected(tier))
        }
    } label: {
        VStack(alignment: .leading, spacing: BIRGELayout.xxs) {
            HStack(alignment: .top) {
                Image(systemName: icon(for: tier))
                    .font(BIRGEFonts.title)
                    .foregroundStyle(isSelected ? BIRGEColors.brandPrimary : BIRGEColors.textSecondary)
                Spacer()
                if tier == .corridor {
                    Text("−52%")
                        .font(BIRGEFonts.captionBold)
                        .foregroundStyle(.white)
                        .padding(.horizontal, BIRGELayout.xxs)
                        .padding(.vertical, BIRGELayout.xxxs)
                        .background(BIRGEColors.success)
                        .clipShape(Capsule())
                }
            }
            Spacer()
            Text(tier.rawValue)
                .font(BIRGEFonts.bodyMedium)
                .lineLimit(2)
            Text(subtitle(for: tier))
                .font(BIRGEFonts.caption)
                .foregroundStyle(BIRGEColors.textSecondary)
            Text("\(store.fares[tier] ?? 0)₸")
                .font(BIRGEFonts.sectionTitle)
                .foregroundStyle(isSelected ? BIRGEColors.brandPrimary : BIRGEColors.textPrimary)
        }
        .padding(BIRGELayout.s)
        .frame(minWidth: 136, maxWidth: 180, minHeight: 136, alignment: .leading)
        .liquidGlass(
            .card,
            tint: isSelected ? BIRGEColors.brandPrimary.opacity(0.1) : .clear
        )
        .overlay(
            RoundedRectangle(cornerRadius: BIRGELayout.radiusL)
                .stroke(
                    isSelected ? BIRGEColors.brandPrimary : Color.clear,
                    lineWidth: 1.5
                )
        )
    }
    .buttonStyle(.plain)
}
```

### routeSummaryPill

```swift
var routeSummaryPill: some View {
    HStack(spacing: BIRGELayout.xxs) {
        Image(systemName: "mappin.and.ellipse")
            .font(BIRGEFonts.caption)
            .foregroundStyle(BIRGEColors.brandPrimary)
        Text("Алатау → Есентай  ·  18 км  ·  ~35 мин")
            .font(BIRGEFonts.caption)
            .foregroundStyle(BIRGEColors.textSecondary)
    }
    .padding(.horizontal, BIRGELayout.s)
    .padding(.vertical, BIRGELayout.xxs)
    .liquidGlass(.pill)
    .padding(.horizontal, BIRGELayout.m)
}
```

---

## Задача B — Tab Bar в PassengerAppView

Текущий `PassengerAppView.swift` скорее всего использует `NavigationStack`.
Нужно добавить `TabView` с 4 вкладками для главного экрана.

> ⚠️ Tab Bar показывается ТОЛЬКО на главном экране (Home).
> Во время поездки (ActiveRide, Searching) — таб бар скрыт.

Структура должна быть:

```swift
// PassengerAppView.swift
@ViewAction(for: PassengerAppFeature.self)
struct PassengerAppView: View {
    @Bindable var store: StoreOf<PassengerAppFeature>

    var body: some View {
        NavigationStack(path: $store.scope(state: \.path, action: \.path)) {
            HomeView(
                store: store.scope(state: \.home, action: \.home)
            )
            .navigationDestination(for: PassengerAppFeature.Path.State.self) { pathState in
                // ... existing destinations
            }
        }
    }
}
```

Добавь в `HomeView.swift` (в `bottomSheet`, внизу):
```swift
// Tab Bar — часть bottom sheet
tabBar
    .padding(.top, BIRGELayout.xxs)
```

**Tab Bar view:**
```swift
var tabBar: some View {
    HStack {
        tabItem(symbol: "map.fill", label: "Главная", isActive: true) { }
        tabItem(symbol: "car.fill", label: "Поездки", isActive: false) {
            send(.rideHistoryTapped)
        }
        tabItem(symbol: "creditcard.fill", label: "Подписка", isActive: false) {
            send(.subscriptionTapped)
        }
        tabItem(symbol: "person.fill", label: "Профиль", isActive: false) {
            send(.profileButtonTapped)
        }
    }
    .padding(.top, BIRGELayout.xxs)
    .padding(.bottom, BIRGELayout.s) // safe area bottom handled by safeAreaInset
}

func tabItem(symbol: String, label: String, isActive: Bool, action: @escaping () -> Void) -> some View {
    Button(action: action) {
        VStack(spacing: 3) {
            Image(systemName: symbol)
                .font(.system(size: 20))
                .foregroundStyle(isActive ? BIRGEColors.brandPrimary : BIRGEColors.textTertiary)
            Text(label)
                .font(.system(size: 10, weight: .medium))
                .foregroundStyle(isActive ? BIRGEColors.brandPrimary : BIRGEColors.textTertiary)
        }
        .frame(maxWidth: .infinity)
    }
    .buttonStyle(.plain)
}
```

Добавь в `HomeFeature.Action`:
```swift
case rideHistoryTapped
case subscriptionTapped
```

Добавь в `HomeFeature.Delegate`:
```swift
case openRideHistory
case openSubscription
```

---

## Проверка

```
xcodebuild -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 16 Pro' build 2>&1 | tail -30
```
