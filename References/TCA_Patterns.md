---
last_updated: 2026-04-22
---

# TCA Patterns — BIRGE Reference

> Готовые паттерны для копирования. Antigravity: читай этот файл при генерации TCA кода.

## Feature Scaffold (шаблон)

```swift
import ComposableArchitecture

@Reducer
struct RideFeature {
    @ObservableState
    struct State: Equatable {
        var rideID: String = ""
        var status: RideStatus = .requested
        var isLoading: Bool = false
        var error: String? = nil
    }

    enum Action: Sendable {
        case rideStarted(String)
        case statusUpdated(RideStatus)
        case cancelTapped
        case rideCancelled
        case errorOccurred(String)
    }

    @Dependency(\.rideClient) var rideClient
    @Dependency(\.webSocketClient) var webSocketClient

    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case let .rideStarted(id):
                state.rideID = id
                state.isLoading = true
                return .run { send in
                    for await event in webSocketClient.connect(rideID: id) {
                        await send(.statusUpdated(event.status))
                    }
                }
                .cancellable(id: RideWebSocketID.self)

            case let .statusUpdated(status):
                state.status = status
                state.isLoading = false
                return .none

            case .cancelTapped:
                return .run { [id = state.rideID] send in
                    do {
                        try await rideClient.cancel(id)
                        await send(.rideCancelled)
                    } catch {
                        await send(.errorOccurred(error.localizedDescription))
                    }
                }

            case .rideCancelled:
                return .cancel(id: RideWebSocketID.self)

            case let .errorOccurred(msg):
                state.error = msg
                state.isLoading = false
                return .none
            }
        }
    }
}

private enum RideWebSocketID: Hashable {}
```

## Dependency Registration

```swift
// 1. Define the interface
struct RideClient {
    var cancel: @Sendable (String) async throws -> Void
    var fetchStatus: @Sendable (String) async throws -> RideStatus
}

// 2. Register with DependencyValues
extension DependencyValues {
    var rideClient: RideClient {
        get { self[RideClient.self] }
        set { self[RideClient.self] = newValue }
    }
}

// 3. Live implementation
extension RideClient: DependencyKey {
    static var liveValue: RideClient {
        RideClient(
            cancel: { id in
                let _ = try await URLSession.shared.data(
                    for: URLRequest(url: URL(string: "/rides/\(id)/cancel")!)
                )
            },
            fetchStatus: { id in
                // ...
            }
        )
    }
    
    // 4. Test implementation (автоматически используется в тестах)
    static var testValue: RideClient {
        RideClient(
            cancel: { _ in },
            fetchStatus: { _ in .requested }
        )
    }
}
```

## Navigation Pattern (NavigationStack + TCA)

```swift
@Reducer
struct AppFeature {
    @ObservableState
    struct State: Equatable {
        var path = StackState<Path.State>()
        var isAuthenticated: Bool = false
    }
    
    enum Action {
        case path(StackActionOf<Path>)
        case authSucceeded
    }
    
    @Reducer
    enum Path {
        case home(HomeFeature)
        case rideDetail(RideFeature)
    }
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .authSucceeded:
                state.isAuthenticated = true
                return .none
            case .path:
                return .none
            }
        }
        .forEach(\.path, action: \.path)
    }
}
```

## Background Effect (GPS)

```swift
case .startLocationTracking:
    return .run { send in
        for await location in LocationClient.shared.locationStream() {
            await send(.locationUpdated(location))
        }
    }
    .cancellable(id: LocationTrackingID.self)

case .stopLocationTracking:
    return .cancel(id: LocationTrackingID.self)

private enum LocationTrackingID: Hashable {}
```

## Unit Test Pattern

```swift
@MainActor
final class OTPFeatureTests: XCTestCase {
    func test_verifyOTP_success_navigatesToHome() async {
        let store = TestStore(initialState: OTPFeature.State()) {
            OTPFeature()
        } withDependencies: {
            $0.authClient.verifyOTP = { _, _ in
                JWT(accessToken: "mock", refreshToken: "mock", expiresIn: 3600)
            }
        }
        
        await store.send(.otpCodeChanged("123456")) {
            $0.otpCode = "123456"
        }
        await store.send(.verifyTapped) {
            $0.isLoading = true
        }
        await store.receive(.verificationSucceeded) {
            $0.isLoading = false
        }
    }
}
```
