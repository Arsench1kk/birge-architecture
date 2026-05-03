# Codex Prompt 03 — Home Screen Refactor (Liquid Glass)

## Контекст
Проект: `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger`
Файлы для редактирования:
- `BIRGEPassenger/Features/Home/HomeView.swift`
- `BIRGEPassenger/Features/Home/HomeFeature.swift`

Мокап для вдохновения: `P-04-home.html` (читать идею, делать ЛУЧШЕ)

**Требования (выполни промпты 01 и 02 сначала).**

---

## Анализ текущего состояния

Текущий `HomeView.swift` имеет:
- ✅ Map layer с MKMapView
- ✅ Базовый bottom sheet
- ❌ Нет Liquid Glass на search bar и bottom sheet
- ❌ Нет горизонтального скролла корридоров с карточками
- ❌ Нет AI notification pill
- ❌ Нет QuickTaxi card внизу
- ❌ Bottom sheet не адаптивный по высоте

---

## Задача A — Обнови HomeFeature.State

В `HomeFeature.swift` убедись что State содержит:
```swift
@ObservableState
struct State: Equatable {
    var driverLat: Double = 43.2220
    var driverLng: Double = 76.8512
    var corridors: [CorridorOption] = CorridorOption.mockData  // stub пока
    var aiMatchCount: Int = 3  // для AI pill текста
}
```

Добавь `CorridorOption` mock-данные если нет:
```swift
// BIRGEPassenger/Models/CorridorOption.swift (или в HomeFeature)
struct CorridorOption: Identifiable, Equatable {
    let id: UUID
    let name: String        // "Алатау → Есентай"
    let departure: String   // "07:30 утром"
    let price: Int          // 890
    let seatsLeft: Int      // 1
    let seatsTotal: Int     // 4
    let matchPercent: Int   // 98
    let passengerInitials: [String]  // ["А", "М", "Д"]

    static let mockData: [CorridorOption] = [
        .init(id: UUID(), name: "Алатау → Есентай", departure: "07:30 утром",
              price: 890, seatsLeft: 1, seatsTotal: 4, matchPercent: 98,
              passengerInitials: ["А", "М", "Д"]),
        .init(id: UUID(), name: "Бостандык → Алмалы", departure: "08:00 утром",
              price: 750, seatsLeft: 2, seatsTotal: 4, matchPercent: 87,
              passengerInitials: ["К", "Н"])
    ]
}
```

---

## Задача B — Полный редизайн HomeView

Перепиши `HomeView.swift` полностью. Структура:

```swift
ZStack(alignment: .top) {
    // LAYER 0: Map (полноэкранный, ignoresSafeArea)
    mapLayer

    // LAYER 1: Top overlays (search bar + AI pill)
    topOverlays

    // LAYER 2: Right floating buttons (location, profile)
    rightFloatingButtons
}
.safeAreaInset(edge: .bottom) {
    bottomSheet
}
.navigationBarHidden(true)
```

### mapLayer
```swift
Map(position: $position) {
    // Driver annotations из store.corridors (mock — пока показывать UserAnnotation)
    UserAnnotation()
}
.mapStyle(.standard(elevation: .realistic))
.mapControls { }
.ignoresSafeArea()
```

### topOverlays (СЛОЙ 2 из мокапа)

**Search Bar** — Liquid Glass, полная ширина, отступ top 59pt:
```swift
HStack(spacing: BIRGELayout.xs) {
    Image(systemName: "magnifyingglass")
        .foregroundStyle(BIRGEColors.textSecondary)
        .font(BIRGEFonts.body)
    Text("Куда едем?")
        .font(BIRGEFonts.body)
        .foregroundStyle(BIRGEColors.textSecondary)
    Spacer()
}
.padding(.horizontal, BIRGELayout.s)
.padding(.vertical, BIRGELayout.xs)
.liquidGlass(.card)    // ← из промпта 01
.padding(.horizontal, BIRGELayout.s)
.padding(.top, BIRGELayout.mapSearchBarTop) // 59pt
.onTapGesture { send(.searchBarTapped) }
```

**AI Pill** — под search bar, центр:
```swift
BIRGEAIPill("AI нашёл \(store.aiMatchCount) коридора рядом")  // ← из промпта 01
    .padding(.top, BIRGELayout.xxs)
```

### rightFloatingButtons (правая сторона, центр экрана)

```swift
VStack(spacing: BIRGELayout.xs) {
    // Profile button
    Button { send(.profileButtonTapped) } label: {
        Image(systemName: "person.crop.circle.fill")
            .font(.title2)
            .foregroundStyle(BIRGEColors.textOnBrand)
            .frame(width: 44, height: 44)
            .liquidGlass(.card, tint: BIRGEColors.brandPrimary.opacity(0.15))
    }
    .accessibilityLabel("Профиль")

    // My location button
    Button { } label: {
        Image(systemName: "location.fill")
            .font(.title3)
            .foregroundStyle(BIRGEColors.brandPrimary)
            .frame(width: 44, height: 44)
            .liquidGlass(.card)
    }
    .accessibilityLabel("Моё местоположение")
}
.padding(.trailing, BIRGELayout.s)
// Позиционировать через .frame(maxHeight: .infinity, alignment: .center) в ZStack
```

### bottomSheet (СЛОЙ 3 из мокапа)

Используй `BIRGEGlassSheet` из промпта 01.

**Содержимое:**
```swift
VStack(spacing: 0) {
    // СЕКЦИЯ 1: AI Corridors
    HStack {
        Text("Подобрано для вас")
            .font(BIRGEFonts.sectionTitle)
        Spacer()
        Button("Все →") { send(.showAllCorridorsTapped) }
            .font(BIRGEFonts.captionBold)
            .foregroundStyle(BIRGEColors.brandPrimary)
    }
    .padding(.horizontal, BIRGELayout.m)
    .padding(.top, BIRGELayout.s)

    // Горизонтальный скролл карточек коридоров
    ScrollView(.horizontal, showsIndicators: false) {
        HStack(spacing: BIRGELayout.xs) {
            ForEach(store.corridors) { corridor in
                corridorCard(corridor)
            }
        }
        .padding(.horizontal, BIRGELayout.m)
        .padding(.vertical, BIRGELayout.xs)
    }

    Divider()
        .padding(.horizontal, BIRGELayout.m)

    // СЕКЦИЯ 2: Quick Taxi
    quickTaxiCard
        .padding(BIRGELayout.m)
}
```

**Карточка коридора** (горизонтальный скролл, ширина 220pt):
```swift
func corridorCard(_ corridor: CorridorOption) -> some View {
    Button { send(.corridorTapped(corridor)) } label: {
        VStack(alignment: .leading, spacing: BIRGELayout.xxs) {
            BIRGEMatchBadge(corridor.matchPercent)  // SF sparkles badge

            Text(corridor.name)
                .font(BIRGEFonts.sectionTitle)
                .foregroundStyle(BIRGEColors.textPrimary)
                .lineLimit(1)

            Text(corridor.departure)
                .font(BIRGEFonts.caption)
                .foregroundStyle(BIRGEColors.textSecondary)

            Divider()

            HStack(spacing: BIRGELayout.xxs) {
                // Seats
                Label("\(corridor.seatsLeft)/\(corridor.seatsTotal)", systemImage: "person.2.fill")
                    .font(BIRGEFonts.captionBold)
                    .foregroundStyle(BIRGEColors.brandPrimary)
                Text("·")
                // Price
                Text("\(corridor.price)₸")
                    .font(BIRGEFonts.captionBold)
                    .foregroundStyle(BIRGEColors.textPrimary)
            }

            // Passenger initials stack
            HStack(spacing: -BIRGELayout.xxs) {
                ForEach(corridor.passengerInitials.prefix(3), id: \.self) { initial in
                    Text(initial)
                        .font(.system(size: 11, weight: .bold))
                        .foregroundStyle(.white)
                        .frame(width: 26, height: 26)
                        .background(BIRGEColors.brandPrimary)
                        .clipShape(Circle())
                        .overlay(Circle().stroke(.white, lineWidth: 2))
                }
            }
        }
        .padding(BIRGELayout.s)
        .frame(width: 220, alignment: .leading)
        .birgeGlassCard()   // ← из промпта 01
    }
    .buttonStyle(.plain)
}
```

**Quick Taxi Card:**
```swift
var quickTaxiCard: some View {
    Button { send(.callTaxiTapped) } label: {
        HStack(spacing: BIRGELayout.s) {
            ZStack {
                Circle()
                    .fill(BIRGEColors.brandPrimary.opacity(0.1))
                    .frame(width: 44, height: 44)
                Image(systemName: "car.fill")
                    .font(BIRGEFonts.sectionTitle)
                    .foregroundStyle(BIRGEColors.brandPrimary)
            }
            VStack(alignment: .leading, spacing: BIRGELayout.xxxs) {
                Text("Вызвать такси")
                    .font(BIRGEFonts.bodyMedium)
                    .foregroundStyle(BIRGEColors.textPrimary)
                Text("Стандарт · ~4 мин · от 900₸")
                    .font(BIRGEFonts.caption)
                    .foregroundStyle(BIRGEColors.textSecondary)
            }
            Spacer()
            Image(systemName: "chevron.right")
                .font(BIRGEFonts.caption)
                .foregroundStyle(BIRGEColors.textTertiary)
        }
        .padding(BIRGELayout.s)
        .birgeGlassCard()
    }
    .buttonStyle(.plain)
}
```

---

## Действия в HomeFeature

Добавь в `HomeFeature.Action` (если нет):
```swift
case showAllCorridorsTapped
```

Добавь в `HomeFeature.Delegate`:
```swift
case openCorridorList
```

Обработай в body:
```swift
case .showAllCorridorsTapped:
    return .send(.delegate(.openCorridorList))
```

Обнови `PassengerAppFeature.swift` — добавь обработку `.home(.delegate(.openCorridorList))`:
```swift
case .home(.delegate(.openCorridorList)):
    // Пока просто corridor list, feature добавим в промпте 08
    return .none
```

---

## Проверка после изменений

```
xcodebuild -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 16 Pro' build 2>&1 | tail -30
```

Ожидаемый результат: `BUILD SUCCEEDED`
