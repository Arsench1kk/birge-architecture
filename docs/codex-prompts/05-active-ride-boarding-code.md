# Codex Prompt 05 — ActiveRide Refactor + BoardingCode

## Контекст
Проект: `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger`
Файлы:
- `BIRGEPassenger/Features/ActiveRide/ActiveRideView.swift`
- `BIRGEPassenger/Features/Ride/RideMapView.swift` (production ride)

Мокапы для вдохновения:
- `P-09-driver-en-route.html` — водитель едет к пассажиру
- `P-10-driver-arrived.html` — водитель ждёт у подъезда
- `P-11-ride-in-progress.html` — едем
- `P-13-boarding-code.html` — QR/код посадки

**Требование: выполни промпты 01–04 сначала.**

---

## Задача A — ActiveRideView полный редизайн

Текущий `ActiveRideView.swift` имеет базовый bottom sheet. Нужно:
1. Liquid Glass на весь bottom sheet
2. Улучшенная Status Pill — без эмодзи, SF Symbols
3. Кнопка "Посмотреть код посадки" в статусе `passengerWait`

### Структура остаётся:
```swift
ZStack(alignment: .top) {
    mapLayer              // без изменений
    statusPillLayer       // обновить
    rightButtons          // новое: floating кнопки
}
.safeAreaInset(edge: .bottom) {
    BIRGEGlassSheet { bottomSheetContent }
}
```

### Обнови statusPill — SF Symbols

```swift
@ViewBuilder
private var statusPill: some View {
    let config = pillConfig(for: store.status)
    HStack(spacing: BIRGELayout.xxs) {
        Image(systemName: config.symbol)
            .font(BIRGEFonts.captionBold)
        Text(config.text)
            .font(BIRGEFonts.captionBold)
    }
    .foregroundStyle(BIRGEColors.textOnBrand)
    .padding(.horizontal, BIRGELayout.s)
    .padding(.vertical, BIRGELayout.xxs)
    .background(config.color)
    .clipShape(Capsule())
    .shadow(color: config.color.opacity(0.4), radius: 10, y: 4)
    .padding(.top, 60)
    .animation(.spring(response: 0.5), value: store.status)
}

private func pillConfig(for status: RideStatus) -> (symbol: String, text: String, color: Color) {
    switch status {
    case .driverArriving:
        return ("car.fill", "Водитель едет · \(store.etaMinutes) мин", BIRGEColors.brandPrimary)
    case .passengerWait:
        return ("mappin.circle.fill", "Водитель ждёт вас", BIRGEColors.warning)
    case .inProgress:
        return ("arrow.triangle.turn.up.right.circle.fill", "В пути · \(store.etaMinutes) мин", BIRGEColors.success)
    default:
        return ("clock", "", BIRGEColors.textTertiary)
    }
}
```

### Обнови driverArrivingSheet — Liquid Glass стиль

```swift
@ViewBuilder
private var driverArrivingSheet: some View {
    VStack(spacing: 0) {
        // Driver info row
        HStack(alignment: .center, spacing: BIRGELayout.s) {
            // Avatar circle
            ZStack {
                Circle()
                    .fill(BIRGEColors.brandPrimary.opacity(0.15))
                    .frame(width: 56, height: 56)
                Image(systemName: "person.fill")
                    .font(.system(size: 24))
                    .foregroundStyle(BIRGEColors.brandPrimary)
            }

            VStack(alignment: .leading, spacing: BIRGELayout.xxxs) {
                Text(store.driver.name)
                    .font(BIRGEFonts.sectionTitle)

                HStack(spacing: BIRGELayout.xxxs) {
                    Image(systemName: "star.fill")
                        .font(.system(size: 11))
                        .foregroundStyle(.yellow)
                    Text(String(format: "%.2f", store.driver.rating))
                        .font(BIRGEFonts.caption)
                    Text("·")
                        .foregroundStyle(BIRGEColors.textTertiary)
                    Text(store.driver.car)
                        .font(BIRGEFonts.caption)
                        .foregroundStyle(BIRGEColors.textSecondary)
                }

                // License plate badge
                Text(store.driver.plate)
                    .font(BIRGEFonts.captionBold)
                    .padding(.horizontal, BIRGELayout.xxs)
                    .padding(.vertical, BIRGELayout.xxxs)
                    .background(BIRGEColors.surfacePrimary)
                    .clipShape(RoundedRectangle(cornerRadius: BIRGELayout.radiusXS))
            }

            Spacer()

            // Action buttons
            HStack(spacing: BIRGELayout.xs) {
                // Chat button
                Button { } label: {
                    Image(systemName: "message.fill")
                        .font(BIRGEFonts.body)
                        .foregroundStyle(BIRGEColors.brandPrimary)
                        .frame(width: 44, height: 44)
                        .liquidGlass(.card, tint: BIRGEColors.brandPrimary.opacity(0.08))
                }
                .accessibilityLabel("Написать водителю")

                // Call button
                Button { send(.callDriverTapped) } label: {
                    Image(systemName: "phone.fill")
                        .font(BIRGEFonts.body)
                        .foregroundStyle(.white)
                        .frame(width: 44, height: 44)
                        .background(BIRGEColors.brandPrimary)
                        .clipShape(Circle())
                        .shadow(color: BIRGEColors.brandPrimary.opacity(0.4), radius: 8, y: 4)
                }
                .accessibilityLabel("Позвонить водителю")
            }
        }
        .padding(.horizontal, BIRGELayout.m)
        .padding(.vertical, BIRGELayout.s)

        Divider()
            .padding(.horizontal, BIRGELayout.m)

        // ETA row
        HStack(spacing: BIRGELayout.xs) {
            Image(systemName: store.status == .passengerWait
                  ? "mappin.circle.fill"
                  : "clock.fill")
                .foregroundStyle(BIRGEColors.brandPrimary)

            if store.status == .passengerWait {
                Text("Водитель ждёт вас")
                    .font(BIRGEFonts.body)
            } else {
                Text("Вас заберут через ~\(store.etaMinutes) мин")
                    .font(BIRGEFonts.body)
            }
            Spacer()
        }
        .padding(.horizontal, BIRGELayout.m)
        .padding(.vertical, BIRGELayout.xs)

        // Boarding code button (только при passengerWait)
        if store.status == .passengerWait {
            Button { send(.showBoardingCodeTapped) } label: {
                Label("Посмотреть код посадки", systemImage: "qrcode")
                    .font(BIRGEFonts.bodyMedium)
                    .foregroundStyle(BIRGEColors.brandPrimary)
                    .frame(maxWidth: .infinity)
                    .padding(.vertical, BIRGELayout.xs)
                    .liquidGlass(.card, tint: BIRGEColors.brandPrimary.opacity(0.06))
                    .padding(.horizontal, BIRGELayout.m)
            }
            .buttonStyle(.plain)
        }

        // Cancel button
        Button { send(.cancelTapped) } label: {
            Text("Отменить поездку")
                .font(BIRGEFonts.subtext)
                .foregroundStyle(BIRGEColors.textSecondary)
        }
        .birgeTapTarget()
        .padding(.bottom, BIRGELayout.s)
    }
}
```

Добавь в `ActiveRideFeature.Action`:
```swift
case showBoardingCodeTapped
```

Добавь `@State private var isBoardingCodePresented = false` в `ActiveRideView`.
При `.showBoardingCodeTapped` — показывать sheet с `BoardingCodeView`.

---

## Задача B — BoardingCodeView (новый экран)

Создай файл `BIRGEPassenger/Features/ActiveRide/BoardingCodeView.swift`.

Мокап: `P-13-boarding-code.html` — большой QR-код или цифровой код посадки.

```swift
// BIRGEPassenger/Features/ActiveRide/BoardingCodeView.swift
struct BoardingCodeView: View {
    let code: String  // 6-значный код, например "847 291"
    let dismiss: () -> Void

    var body: some View {
        ZStack {
            // Blur background
            BIRGEColors.brandPrimary
                .opacity(0.08)
                .ignoresSafeArea()

            VStack(spacing: BIRGELayout.xl) {
                // Заголовок
                VStack(spacing: BIRGELayout.xxs) {
                    Image(systemName: "qrcode.viewfinder")
                        .font(.system(size: 40))
                        .foregroundStyle(BIRGEColors.brandPrimary)
                    Text("Код посадки")
                        .font(BIRGEFonts.title)
                    Text("Покажите водителю для подтверждения")
                        .font(BIRGEFonts.subtext)
                        .foregroundStyle(BIRGEColors.textSecondary)
                        .multilineTextAlignment(.center)
                }

                // Большой код
                Text(code)
                    .font(.system(size: 52, weight: .bold, design: .monospaced))
                    .foregroundStyle(BIRGEColors.textPrimary)
                    .tracking(8)
                    .padding(BIRGELayout.xl)
                    .birgeGlassCard()

                // Инструкция
                HStack(spacing: BIRGELayout.xs) {
                    Image(systemName: "info.circle.fill")
                        .foregroundStyle(BIRGEColors.brandPrimary)
                    Text("Код обновляется каждые 60 секунд")
                        .font(BIRGEFonts.caption)
                        .foregroundStyle(BIRGEColors.textSecondary)
                }

                // Закрыть
                BIRGESecondaryButton(title: "Закрыть") {
                    dismiss()
                }
                .padding(.horizontal, BIRGELayout.m)
            }
            .padding(BIRGELayout.m)
        }
        .presentationDetents([.medium])
        .presentationDragIndicator(.visible)
    }
}
```

В `ActiveRideView` добавь sheet:
```swift
.sheet(isPresented: $isBoardingCodePresented) {
    BoardingCodeView(code: "847 291") {
        isBoardingCodePresented = false
    }
}
```

---

## Проверка

```
xcodebuild -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 16 Pro' build 2>&1 | tail -30
```
