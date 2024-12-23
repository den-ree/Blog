
Actors

---

Actors are a key component of Swift Concurrency, designed to ensure thread-safe access to shared mutable state. Distributed actors extend this concept to enable communication between processes or devices, making them ideal for distributed systems.

---

## **What are Actors?**

Actors are reference types that isolate state, ensuring that only one task can access their mutable state at a time. They simplify concurrency by handling synchronization automatically.

### Features of Actors:
1. **State Isolation**:
   - Prevents data races by isolating state within the actor.
2. **Serialization**:
   - Ensures only one task interacts with the actor's state at a time.
3. **Thread Safety**:
   - Simplifies thread-safe programming by avoiding manual synchronization.

---

## **Declaring and Using Actors**

### Declaring an Actor
Actors are defined using the `actor` keyword.

#### Example:
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
```

### Interacting with Actors
You must use `await` to interact with actor methods or properties.

#### Example:
```swift
let account = BankAccount()

Task {
    await account.deposit(amount: 100.0)
    let balance = await account.getBalance()
    print("Balance: \(balance)")
}
```

---

## **Actor Isolation**

Actor isolation ensures that:
1. **Mutable State is Private**:
   - Only accessible within the actor.
2. **Thread-Safe Access**:
   - Tasks must use `await` to interact with an actor's state.

---

## **Nonisolated Properties and Methods**

Use `nonisolated` for properties or methods that don’t require actor isolation.

#### Example:
```swift
actor WeatherService {
    nonisolated let apiEndpoint = "https://api.weather.com"

    func fetchWeather() async -> Weather {
        // Fetch weather data
    }
}
```

---

## **MainActor**

The `@MainActor` attribute ensures that actor methods or properties run on the main thread, making it ideal for UI updates.

#### Example:
```swift
@MainActor
actor UIUpdater {
    func updateUI() {
        // Perform UI updates
    }
}
```

---

## **What are Distributed Actors?**

Distributed actors extend actors to support communication across process or network boundaries. They provide a foundation for building distributed systems.

### Key Features:
1. **Location Transparency**:
   - The caller doesn’t need to know if the actor is local or remote.
2. **Automatic Serialization**:
   - Parameters and return values are serialized for remote communication.
3. **Fault Tolerance**:
   - Handle failures gracefully in distributed environments.

---

## **Declaring Distributed Actors**

Use the `distributed actor` keyword for distributed actors.

#### Example:
```swift
import Distributed

distributed actor ChatService {
    func sendMessage(_ message: String) async throws {
        print("Sending message: \(message)")
    }
}
```

---

## **Common Patterns with Distributed Actors**

1. **Remote Procedure Calls (RPC)**:
   - Invoke methods on distributed actors as if they were local.

#### Example:
```swift
distributed actor Calculator {
    func add(_ a: Int, _ b: Int) -> Int {
        return a + b
    }
}
```

2. **Error Handling**:
   - Handle communication failures.
   ```swift
   do {
       let result = try await remoteCalculator.add(3, 5)
       print("Result: \(result)")
   } catch {
       print("Failed to communicate with the remote actor: \(error)")
   }
   ```

---

## **Actor Best Practices**

1. **Use Nonisolated Methods Sparingly**:
   - Only for properties or methods that don’t mutate state.

2. **Prefer Actors for Shared State**:
   - Use actors over manual synchronization methods like locks or semaphores.

3. **Leverage Distributed Actors for Scalability**:
   - Use distributed actors to build systems that scale across multiple devices or processes.

---

## **Debugging Actors**

1. **Log Access**:
   - Add logs in actor methods to trace execution.

2. **Use Instruments**:
   - Profile actor interactions and detect contention points.

---

## **Common Use Cases**

1. **Thread-Safe Data Access**:
   - Use actors to manage shared mutable state.
   ```swift
   actor DataStore {
       private var items: [String] = []

       func addItem(_ item: String) {
           items.append(item)
       }

       func getItems() -> [String] {
           return items
       }
   }
   ```

2. **UI Synchronization**:
   - Use `@MainActor` to perform UI updates safely.
   ```swift
   @MainActor
   actor UIManager {
       func updateLabel(_ text: String) {
           label.text = text
       }
   }
   ```

3. **Distributed Systems**:
   - Build a distributed chat application.
   ```swift
   distributed actor ChatRoom {
       func sendMessage(_ message: String) {
           print("Message: \(message)")
       }
   }
   ```

---

## **Conclusion**

Actors and Distributed Actors are powerful constructs in Swift Concurrency that simplify thread-safe programming and enable distributed computing. By isolating mutable state and leveraging actor features like `@MainActor` and `distributed`, developers can build scalable, robust applications with ease.



Actors prevent data races by isolating mutable state.

### Example: Counter Actor
```swift
actor Counter {
    private var value = 0

    func increment() { value += 1 }
    func getValue() -> Int { value }
}

let counter = Counter()
Task {
    await counter.increment()
    print("Counter value: \(await counter.getValue())")
}
```

Sendable

1. Prefer structures against classes 
2. If struct, enum or collection type contains only @Sendable  then it's also sendable
3. Use @unchecked @Sendable for classes to manually provide safe read/write access

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
