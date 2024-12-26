Actors and Sendable

---

Actors are designed to ensure thread-safe access to shared mutable state. Distributed actors extend this concept to enable communication between processes or devices, making them ideal for distributed systems.

`actor` is reference type that isolate state, ensuring that only one task can access their mutable state at a time. They simplify concurrency by handling synchronization automatically.

- Prevents data races by isolating state within the actor.
- Only accessible within the actor.
- Ensures only one task interacts with the actor's state at a time.
- Simplifies thread-safe programming by avoiding manual synchronization.
- Tasks must use `await` to interact with an actor's state.
- Do not need `[weak self]` because Swift Concurrency ensures safe and isolated access.
- `[weak self]` is only required in special cases like detached tasks or long-running tasks with non-isolated methods.

---

The `Sendable` protocol ensures that values passed between concurrency domains are thread-safe.

1. Prefer structures against classes 
2. If struct, enum or collection type contains only @Sendable  then it's also sendable
3. Use `@unchecked Sendable` to manually provide safe read/write access.
4. Enforced at compile time, providing strong guarantees.

| **Type**                 | **Conformance Requirements** | 
|--------------------------|------------------------------|
| **Structures/Enums**     | All members and associated values must be `Sendable`.<br>Implicit if frozen, not public, and not `@usableFromInline`.|
| **Actors**               | <br>Implicitly conform.|
| **Classes**              | <br>Must be `final`.<br>Only immutable and sendable stored properties.<br>No superclass or only `NSObject` as a superclass.<br>Classes marked `@MainActor` are implicitly sendable.|
| **Functions/Closures**   | <br>Mark with `@Sendable`.<br>Captured values must be sendable.<br>Captures must be by value.<br>Implicit in contexts like `Task.detached`.<br>Use `@Sendable` in type annotations or before closure parameters.|

#### Best Practices for Actors

- Use actors over manual synchronization methods like locks or semaphores.
- Use `nonisolated` for properties or methods that don’t require actor isolation or don't mutate state.
- Design types and closures to satisfy `Sendable` implicitly whenever possible.
- Ensure manual verification when marking types as `@unchecked` to avoid concurrency errors.
- Always use @Sendable for closures passed to detached tasks or other concurrency operations.
- Use `@MainActor` attribute to ensure that actor methods or properties run on the main thread for UI updates.

---

#### Examples Of Actor usage

**Thread-Safe Data Access**
```swift
actor BankAccount {
    private var balance: Double = 0.0

    func deposit(amount: Double) {
        balance += amount
    }

    func withdraw(amount: Double) throws {
        guard balance >= amount else { throw BankError.insufficientFunds }
        balance -= amount
    }

    func getBalance() -> Double {
        return balance
    }
}

let account = BankAccount()

Task {
    await account.deposit(amount: 100.0)
    let balance = await account.getBalance()
    print("Balance: \(balance)")
}
```

**@Sendable**
```swift 
//@unchecked can be used, but be careful!
class ConcurrentCache<Key: Hashable & Sendable, Value: Sendable>: @unchecked Sendable {
  var lock: NSLock
  var storage: [Key: Value]
}

let lily = Chicken(name: "Lily")
Task.detached {@Sendable in
	lily.feed()
}
```

**Non-isolated**
```swift
actor WeatherService {
    nonisolated let apiEndpoint = "https://api.weather.com"

    func fetchWeather() async -> Weather {
        // Fetch weather data
    }
}
```

**UI Synchronization**:
```swift
@MainActor
actor UIManager {
    func updateLabel(_ text: String) {
        label.text = text
    }
}
```