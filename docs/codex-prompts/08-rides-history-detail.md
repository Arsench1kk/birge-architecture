# Codex Prompt 08 — Rides History + Ride Detail

## Контекст
Проект: `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger`
Мокапы: `P-14-rides-history.html`, `P-15-ride-detail.html`, `P-16-report-issue.html`

**Требование: выполни промпты 01–07 сначала.**

---

## Модель данных — RideHistoryItem

Создай или обнови `BIRGEPassenger/Models/RideHistoryItem.swift`:

```swift
import Foundation

struct RideHistoryItem: Identifiable, Equatable {
    let id: UUID
    let date: Date
    let origin: String
    let destination: String
    let price: Int
    let durationMinutes: Int
    let distanceKm: Double
    let driverName: String
    let driverRating: Double
    let rideType: RideType
    let status: Status

    enum RideType: String, Equatable {
        case standard = "Такси"
        case corridor = "Коридор"
        case comfort = "Комфорт"
    }

    enum Status: Equatable {
        case completed
        case cancelled
    }

    var formattedDate: String {
        let formatter = DateFormatter()
        formatter.locale = Locale(identifier: "ru_RU")
        formatter.dateStyle = .medium
        formatter.timeStyle = .short
        return formatter.string(from: date)
    }

    static let mockData: [RideHistoryItem] = [
        .init(id: UUID(), date: Date().addingTimeInterval(-3600),
              origin: "Алатау", destination: "Есентай",
              price: 890, durationMinutes: 34, distanceKm: 17.8,
              driverName: "Азамат К.", driverRating: 4.95,
              rideType: .corridor, status: .completed),
        .init(id: UUID(), date: Date().addingTimeInterval(-86400),
              origin: "Бостандык", destination: "Алмалы",
              price: 750, durationMinutes: 28, distanceKm: 14.2,
              driverName: "Нурлан С.", driverRating: 4.87,
              rideType: .corridor, status: .completed),
        .init(id: UUID(), date: Date().addingTimeInterval(-172800),
              origin: "Медеу", destination: "Абай",
              price: 1200, durationMinutes: 22, distanceKm: 9.5,
              driverName: "Арман Б.", driverRating: 4.78,
              rideType: .standard, status: .completed),
        .init(id: UUID(), date: Date().addingTimeInterval(-259200),
              origin: "Орбита", destination: "Достык",
              price: 580, durationMinutes: 18, distanceKm: 7.3,
              driverName: "—", driverRating: 0,
              rideType: .corridor, status: .cancelled)
    ]
}
```

---

## Задача A — RidesHistoryFeature

Создай `BIRGEPassenger/Features/RidesHistory/RidesHistoryFeature.swift`:

```swift
@Reducer
struct RidesHistoryFeature {
    @ObservableState
    struct State: Equatable {
        var rides: [RideHistoryItem] = RideHistoryItem.mockData
        var isLoading: Bool = false
        var selectedFilter: Filter = .all

        enum Filter: String, CaseIterable, Equatable {
            case all = "Все"
            case completed = "Завершённые"
            case cancelled = "Отменённые"
        }

        var filteredRides: [RideHistoryItem] {
            switch selectedFilter {
            case .all: return rides
            case .completed: return rides.filter { $0.status == .completed }
            case .cancelled: return rides.filter { $0.status == .cancelled }
            }
        }
    }

    enum Action: Sendable {
        case onAppear
        case filterSelected(State.Filter)
        case rideTapped(RideHistoryItem)
        case delegate(Delegate)

        enum Delegate: Sendable {
            case rideSelected(RideHistoryItem)
        }
    }

    var body: some Reducer<State, Action> {
        Reduce { state, action in
            switch action {
            case .onAppear:
                // TODO: load from network
                return .none
            case .filterSelected(let filter):
                state.selectedFilter = filter
                return .none
            case .rideTapped(let ride):
                return .send(.delegate(.rideSelected(ride)))
            case .delegate:
                return .none
            }
        }
    }
}
```

---

## Задача B — RidesHistoryView

Создай `BIRGEPassenger/Features/RidesHistory/RidesHistoryView.swift`:

```swift
@ViewAction(for: RidesHistoryFeature.self)
struct RidesHistoryView: View {
    @Bindable var store: StoreOf<RidesHistoryFeature>

    var body: some View {
        ZStack {
            BIRGEColors.background.ignoresSafeArea()

            VStack(spacing: 0) {
                // Filter Pills
                filterBar
                    .padding(.horizontal, BIRGELayout.m)
                    .padding(.vertical, BIRGELayout.xs)
                    .background(BIRGEColors.background)

                Divider()

                // Ride List
                if store.filteredRides.isEmpty {
                    emptyState
                } else {
                    ScrollView(showsIndicators: false) {
                        LazyVStack(spacing: BIRGELayout.xs) {
                            ForEach(store.filteredRides) { ride in
                                rideCard(ride)
                            }
                        }
                        .padding(.horizontal, BIRGELayout.m)
                        .padding(.vertical, BIRGELayout.xs)
                    }
                }
            }
        }
        .navigationTitle("Мои поездки")
        .navigationBarTitleDisplayMode(.large)
        .onAppear { send(.onAppear) }
    }

    // MARK: - Filter Bar

    private var filterBar: some View {
        HStack(spacing: BIRGELayout.xxs) {
            ForEach(RidesHistoryFeature.State.Filter.allCases, id: \.self) { filter in
                let isActive = store.selectedFilter == filter
                Button {
                    withAnimation(.spring(response: 0.3)) {
                        send(.filterSelected(filter))
                    }
                } label: {
                    Text(filter.rawValue)
                        .font(BIRGEFonts.captionBold)
                        .foregroundStyle(isActive ? .white : BIRGEColors.brandPrimary)
                        .padding(.horizontal, BIRGELayout.s)
                        .padding(.vertical, BIRGELayout.xxs)
                        .background(isActive ? BIRGEColors.brandPrimary : BIRGEColors.brandPrimary.opacity(0.1))
                        .clipShape(Capsule())
                }
                .buttonStyle(.plain)
            }
            Spacer()
        }
    }

    // MARK: - Ride Card

    private func rideCard(_ ride: RideHistoryItem) -> some View {
        Button { send(.rideTapped(ride)) } label: {
            HStack(alignment: .top, spacing: BIRGELayout.s) {
                // Status icon
                ZStack {
                    Circle()
                        .fill(ride.status == .completed
                              ? BIRGEColors.success.opacity(0.12)
                              : BIRGEColors.danger.opacity(0.12))
                        .frame(width: 44, height: 44)
                    Image(systemName: ride.status == .completed
                          ? "checkmark.circle.fill"
                          : "xmark.circle.fill")
                        .font(BIRGEFonts.sectionTitle)
                        .foregroundStyle(ride.status == .completed
                                         ? BIRGEColors.success
                                         : BIRGEColors.danger)
                }

                VStack(alignment: .leading, spacing: BIRGELayout.xxxs) {
                    // Route
                    Text("\(ride.origin) → \(ride.destination)")
                        .font(BIRGEFonts.bodyMedium)
                        .foregroundStyle(BIRGEColors.textPrimary)
                        .lineLimit(1)

                    // Date and type
                    HStack(spacing: BIRGELayout.xxs) {
                        Text(ride.formattedDate)
                            .font(BIRGEFonts.caption)
                            .foregroundStyle(BIRGEColors.textSecondary)
                        Text("·")
                            .foregroundStyle(BIRGEColors.textTertiary)
                        rideTypeBadge(ride.rideType)
                    }
                }

                Spacer()

                // Price
                VStack(alignment: .trailing, spacing: BIRGELayout.xxxs) {
                    Text("\(ride.price)₸")
                        .font(BIRGEFonts.bodyMedium)
                        .foregroundStyle(ride.status == .completed
                                         ? BIRGEColors.textPrimary
                                         : BIRGEColors.textSecondary)
                    Image(systemName: "chevron.right")
                        .font(BIRGEFonts.caption)
                        .foregroundStyle(BIRGEColors.textTertiary)
                }
            }
            .padding(BIRGELayout.s)
            .birgeGlassCard()
        }
        .buttonStyle(.plain)
    }

    private func rideTypeBadge(_ type: RideHistoryItem.RideType) -> some View {
        let color: Color = type == .corridor ? BIRGEColors.brandPrimary : BIRGEColors.textSecondary
        return Text(type.rawValue)
            .font(.system(size: 10, weight: .semibold))
            .foregroundStyle(color)
            .padding(.horizontal, BIRGELayout.xxs)
            .padding(.vertical, 2)
            .background(color.opacity(0.12))
            .clipShape(Capsule())
    }

    // MARK: - Empty State

    private var emptyState: some View {
        VStack(spacing: BIRGELayout.s) {
            Image(systemName: "car.fill")
                .font(.system(size: 48))
                .foregroundStyle(BIRGEColors.textTertiary)
            Text("Нет поездок")
                .font(BIRGEFonts.title)
                .foregroundStyle(BIRGEColors.textPrimary)
            Text("Ваши поездки будут отображаться здесь")
                .font(BIRGEFonts.body)
                .foregroundStyle(BIRGEColors.textSecondary)
                .multilineTextAlignment(.center)
        }
        .padding(BIRGELayout.xxxl)
        .frame(maxWidth: .infinity, maxHeight: .infinity)
    }
}
```

---

## Задача C — RideDetailView (простой, без Feature)

Создай `BIRGEPassenger/Features/RidesHistory/RideDetailView.swift`:

```swift
struct RideDetailView: View {
    let ride: RideHistoryItem
    @State private var isReportSheetPresented = false

    var body: some View {
        ScrollView(showsIndicators: false) {
            VStack(spacing: BIRGELayout.s) {
                // MAP STUB
                RoundedRectangle(cornerRadius: BIRGELayout.radiusL)
                    .fill(BIRGEColors.surfacePrimary)
                    .frame(height: 180)
                    .overlay(
                        Image(systemName: "map.fill")
                            .font(.system(size: 40))
                            .foregroundStyle(BIRGEColors.textTertiary)
                    )
                    .padding(.horizontal, BIRGELayout.m)

                // SUMMARY CARD
                VStack(alignment: .leading, spacing: BIRGELayout.xs) {
                    HStack {
                        Text("\(ride.origin) → \(ride.destination)")
                            .font(BIRGEFonts.sectionTitle)
                        Spacer()
                    }

                    Divider()

                    infoRow(label: "Стоимость", value: "\(ride.price)₸",
                            symbol: "tengesign.circle.fill", symbolColor: BIRGEColors.brandPrimary)
                    infoRow(label: "Время в пути", value: "\(ride.durationMinutes) мин",
                            symbol: "clock.fill")
                    infoRow(label: "Дистанция", value: String(format: "%.1f км", ride.distanceKm),
                            symbol: "arrow.forward.circle.fill")
                    infoRow(label: "Водитель", value: ride.driverName,
                            symbol: "person.fill")

                    if ride.driverRating > 0 {
                        infoRow(label: "Рейтинг водителя",
                                value: String(format: "%.2f", ride.driverRating),
                                symbol: "star.fill", symbolColor: .yellow)
                    }

                    Divider()

                    Label("Оплачено через Kaspi Pay", systemImage: "creditcard.fill")
                        .font(BIRGEFonts.captionBold)
                        .foregroundStyle(BIRGEColors.brandPrimary)
                }
                .padding(BIRGELayout.m)
                .birgeGlassCard()
                .padding(.horizontal, BIRGELayout.m)

                // DATE
                Text(ride.formattedDate)
                    .font(BIRGEFonts.caption)
                    .foregroundStyle(BIRGEColors.textSecondary)

                // REPORT BUTTON
                Button {
                    isReportSheetPresented = true
                } label: {
                    Label("Сообщить о проблеме", systemImage: "exclamationmark.triangle.fill")
                        .font(BIRGEFonts.body)
                        .foregroundStyle(BIRGEColors.danger)
                }
                .birgeTapTarget()
                .padding(.bottom, BIRGELayout.xl)
            }
            .padding(.top, BIRGELayout.xs)
        }
        .background(BIRGEColors.background)
        .navigationTitle("Поездка")
        .navigationBarTitleDisplayMode(.inline)
        .sheet(isPresented: $isReportSheetPresented) {
            // Переиспользуем существующий ReportIssueView из RideCompleteView
            // или создай аналогичный лёгкий sheet
            Text("Сообщить о проблеме")
                .presentationDetents([.medium])
        }
    }

    private func infoRow(label: String, value: String, symbol: String, symbolColor: Color = .secondary) -> some View {
        HStack {
            Text(label)
                .font(BIRGEFonts.subtext)
                .foregroundStyle(BIRGEColors.textSecondary)
            Spacer()
            HStack(spacing: BIRGELayout.xxxs) {
                Text(value)
                    .font(BIRGEFonts.subtext)
                    .foregroundStyle(BIRGEColors.textPrimary)
                Image(systemName: symbol)
                    .font(.system(size: 12))
                    .foregroundStyle(symbolColor)
            }
        }
        .padding(.bottom, BIRGELayout.xxxs)
    }
}
```

---

## Задача D — Встроить в PassengerAppFeature

В `PassengerAppFeature.Path` добавь:
```swift
case ridesHistory(RidesHistoryFeature)
```

Обработай делегат из Home:
```swift
case .home(.delegate(.openRideHistory)):
    state.path.append(.ridesHistory(RidesHistoryFeature.State()))
    return .none
```

Обработай выбор поездки (в PassengerAppFeature, navigate to detail — через простой NavigationLink в View без отдельного Feature):
```swift
case .path(.element(_, action: .ridesHistory(.delegate(.rideSelected)))):
    // Handled in View via NavigationLink
    return .none
```

В `PassengerAppView.swift` добавь destination:
```swift
.navigationDestination(for: RidesHistoryFeature.State.self) { state in
    RidesHistoryView(store: store.scope(state: \.path[...].ridesHistory, action: ...))
}
```

---

## Проверка

```
xcodebuild -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 16 Pro' build 2>&1 | tail -30
```
