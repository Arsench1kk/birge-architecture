# Codex Prompt 06 — RideComplete Refactor + Offer Screen

## Контекст
Проект: `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger`
Файлы:
- `BIRGEPassenger/Features/RideComplete/RideCompleteView.swift`

Мокапы: `P-12-ride-complete.html`, `P-08-offer.html`

**Требование: выполни промпты 01–05 сначала.**

---

## Задача A — RideCompleteView улучшение

Текущий `RideCompleteView.swift` достаточно хороший, но нужно:
1. Добавить Liquid Glass на summary card
2. Убрать эмодзи из `summaryRow`, заменить на SF Symbols
3. Улучшить checkmark анимацию

### Изменения в `summaryRow`:

Убери параметр `icon: String?` и замени на `symbol: String?`:
```swift
private func summaryRow(
    label: String,
    value: String,
    symbol: String? = nil,
    symbolColor: Color = BIRGEColors.textSecondary
) -> some View {
    HStack {
        Text(label)
            .font(label == "Стоимость" ? BIRGEFonts.sectionTitle : BIRGEFonts.subtext)
            .foregroundStyle(BIRGEColors.textSecondary)
        Spacer()
        HStack(spacing: BIRGELayout.xxxs) {
            Text(value)
                .font(label == "Стоимость" ? BIRGEFonts.heroNumber : BIRGEFonts.subtext)
                .foregroundStyle(label == "Стоимость" ? BIRGEColors.textPrimary : BIRGEColors.textSecondary)
            if let symbol = symbol {
                Image(systemName: symbol)
                    .font(BIRGEFonts.caption)
                    .foregroundStyle(symbolColor)
            }
        }
    }
    .padding(.bottom, BIRGELayout.xxs)
}
```

Обнови вызовы:
```swift
summaryRow(label: "Стоимость", value: "1 850₸", symbol: "circle.fill", symbolColor: BIRGEColors.brandPrimary)
summaryRow(label: "Время в пути", value: "34 мин", symbol: "clock.fill")
summaryRow(label: "Дистанция", value: "17.8 км", symbol: "arrow.right.circle")
summaryRow(label: "Водитель", value: "Азамат К.", symbol: "star.fill", symbolColor: .yellow)
```

Последняя строка "Оплачено":
```swift
HStack {
    Label("Оплачено через Kaspi Pay", systemImage: "creditcard.fill")
        .font(BIRGEFonts.captionBold)
        .foregroundStyle(BIRGEColors.brandPrimary)
    Spacer()
}
```

### Замени `.birgeCard()` на `.birgeGlassCard()`:
```swift
// Было:
.birgeCard()
.shadow(color: .black.opacity(0.08), radius: 8, y: 2)

// Стало:
.birgeGlassCard()
```

### Улучши checkmark анимацию:
```swift
ZStack {
    // Pulse ring
    Circle()
        .fill(BIRGEColors.success.opacity(0.15))
        .frame(width: 110, height: 110)
        .scaleEffect(store.isCheckmarkVisible ? 1.2 : 0.8)
        .opacity(store.isCheckmarkVisible ? 0 : 0.5)
        .animation(
            reduceMotion ? nil : .easeOut(duration: 1.0).repeatForever(autoreverses: false),
            value: store.isCheckmarkVisible
        )

    // Main circle
    Circle()
        .fill(BIRGEColors.success)
        .frame(width: 80, height: 80)
        .scaleEffect(store.isCheckmarkVisible ? 1.0 : 0.3)
        .opacity(store.isCheckmarkVisible ? 1.0 : 0)
        .animation(
            reduceMotion ? nil : .spring(response: 0.5, dampingFraction: 0.6),
            value: store.isCheckmarkVisible
        )

    // Checkmark
    Image(systemName: "checkmark")
        .font(.system(size: 36, weight: .bold))
        .foregroundStyle(.white)
        .scaleEffect(store.isCheckmarkVisible ? 1.0 : 0.3)
        .opacity(store.isCheckmarkVisible ? 1.0 : 0)
        .animation(
            reduceMotion ? nil : .spring(response: 0.5, dampingFraction: 0.6).delay(0.1),
            value: store.isCheckmarkVisible
        )
}
.padding(.top, BIRGELayout.xxxl)
```

---

## Задача B — OfferView (новый экран)

Мокап `P-08-offer.html` — экран когда водитель предлагает поездку (для corridor flow).

Создай файл `BIRGEPassenger/Features/Searching/OfferView.swift`.

**Это промежуточный экран между Searching и ActiveRide для corridor-поездок.**

```swift
// BIRGEPassenger/Features/Searching/OfferView.swift
struct OfferView: View {
    let driverName: String
    let driverRating: Double
    let vehicleName: String
    let plate: String
    let price: Int
    let corridorName: String
    let etaMinutes: Int
    let onAccept: () -> Void
    let onDecline: () -> Void

    var body: some View {
        ZStack {
            BIRGEColors.brandPrimary
                .ignoresSafeArea()

            VStack(spacing: BIRGELayout.xl) {
                Spacer()

                // ЗАГОЛОВОК
                VStack(spacing: BIRGELayout.xs) {
                    Image(systemName: "car.2.fill")
                        .font(.system(size: 48))
                        .foregroundStyle(.white)
                    Text("Найден водитель!")
                        .font(BIRGEFonts.title)
                        .foregroundStyle(.white)
                    Text("Примите предложение в течение 15 сек")
                        .font(BIRGEFonts.caption)
                        .foregroundStyle(.white.opacity(0.72))
                }

                // OFFER CARD
                VStack(spacing: BIRGELayout.s) {
                    // Driver info
                    HStack(alignment: .center, spacing: BIRGELayout.s) {
                        ZStack {
                            Circle()
                                .fill(BIRGEColors.brandPrimary.opacity(0.15))
                                .frame(width: 56, height: 56)
                            Image(systemName: "person.fill")
                                .font(.system(size: 28))
                                .foregroundStyle(BIRGEColors.brandPrimary)
                        }
                        VStack(alignment: .leading, spacing: BIRGELayout.xxxs) {
                            Text(driverName)
                                .font(BIRGEFonts.sectionTitle)
                            HStack(spacing: BIRGELayout.xxxs) {
                                Image(systemName: "star.fill")
                                    .font(.system(size: 11))
                                    .foregroundStyle(.yellow)
                                Text(String(format: "%.1f", driverRating))
                                    .font(BIRGEFonts.caption)
                                Text("·")
                                Text(vehicleName)
                                    .font(BIRGEFonts.caption)
                                    .foregroundStyle(BIRGEColors.textSecondary)
                            }
                        }
                        Spacer()
                        Text(plate)
                            .font(BIRGEFonts.captionBold)
                            .padding(.horizontal, BIRGELayout.xs)
                            .padding(.vertical, BIRGELayout.xxxs)
                            .background(BIRGEColors.surfacePrimary)
                            .clipShape(RoundedRectangle(cornerRadius: BIRGELayout.radiusXS))
                    }

                    Divider()

                    // Route and price info
                    HStack {
                        VStack(alignment: .leading, spacing: BIRGELayout.xxxs) {
                            Label(corridorName, systemImage: "arrow.forward.circle.fill")
                                .font(BIRGEFonts.bodyMedium)
                                .foregroundStyle(BIRGEColors.textPrimary)
                            Label("~\(etaMinutes) мин до вас", systemImage: "clock.fill")
                                .font(BIRGEFonts.caption)
                                .foregroundStyle(BIRGEColors.textSecondary)
                        }
                        Spacer()
                        VStack(alignment: .trailing, spacing: BIRGELayout.xxxs) {
                            Text("\(price)₸")
                                .font(BIRGEFonts.heroNumber)
                                .foregroundStyle(BIRGEColors.brandPrimary)
                            Text("Стоимость")
                                .font(BIRGEFonts.caption)
                                .foregroundStyle(BIRGEColors.textSecondary)
                        }
                    }
                }
                .padding(BIRGELayout.m)
                .birgeGlassCard()
                .padding(.horizontal, BIRGELayout.m)

                // КНОПКИ
                VStack(spacing: BIRGELayout.xs) {
                    BIRGEPrimaryButton(title: "Принять поездку") {
                        onAccept()
                    }
                    .padding(.horizontal, BIRGELayout.m)

                    Button("Отклонить") {
                        onDecline()
                    }
                    .font(BIRGEFonts.body)
                    .foregroundStyle(.white.opacity(0.72))
                    .birgeTapTarget()
                }

                Spacer()
            }
        }
        .navigationBarHidden(true)
    }
}
```

---

## Проверка

```
xcodebuild -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 16 Pro' build 2>&1 | tail -30
```
