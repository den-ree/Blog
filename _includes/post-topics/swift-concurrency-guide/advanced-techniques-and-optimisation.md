Advanced Techniques

Swift Concurrency provides advanced techniques to fine-tune and extend concurrency for specialized use cases. These include features like detached tasks, task-local values, atomic values, sendable protocol, and handling advanced synchronization patterns.

---

#### Best Practices

- Avoid overusing detached tasks as they lack the parent task's context.
- Store lightweight, task-specific data.
- Ensure all shared data conforms to `Sendable`.
- Regularly check for cancellation in long-running tasks.

---

#### Detached Tasks

Detached tasks are independent units of work that do not inherit context like priority or task-local values from their parent.

- Runs independently of the task hierarchy.
- Ideal for background work that does not need context from the current task.

```swift
Task.detached {
    let result = await performBackgroundComputation()
    print(result)
}
```

**Use Cases**
- Offloading computationally intensive tasks to the background.
- Performing work that does not depend on the current task's context.

---

#### Task-Local Values

Task-local values provide a way to store lightweight state that can be accessed by the current task and its child tasks.

```swift
@TaskLocal static var userID: String?

Task {
    Task.userID = "12345"
    print(Task.userID ?? "No user ID")
}
```

**Benefits**
- Pass context-sensitive values like authentication tokens or user IDs across task hierarchies.

---

#### Atomic Values

Atomic values ensure thread-safe updates to shared state without using locks.

```swift
actor Counter {
    private var value = 0

    func increment() {
        value += 1
    }

    func getValue() -> Int {
        return value
    }
}
```

**Benefits**
- Simplifies concurrent state management.
- Avoids pitfalls like deadlocks and data races.

---

#### Advanced Synchronization Patterns

Swift Concurrency enables advanced synchronization using actors and task groups.

```swift
actor SharedData {
    private var data: [String] = []

    func addItem(_ item: String) {
        data.append(item)
    }

    func getData() -> [String] {
        return data
    }
}

let sharedData = SharedData()

await withTaskGroup(of: Void.self) { group in
    for i in 1...10 {
        group.addTask {
            await sharedData.addItem("Item \(i)")
        }
    }
}

let results = await sharedData.getData()
print(results)
```

---

#### Handling Cancellation Gracefully

Tasks in Swift can respond to cancellation signals, allowing developers to clean up resources or stop unnecessary work.

```swift
Task {
    for i in 0...100 {
        guard !Task.isCancelled else {
            print("Task was cancelled")
            break
        }
        print("Processing item \(i)")
    }
}
```

**Benefits**
- Improves resource management.
- Prevents unnecessary computations.

---

#### Avoid Main Actor Blocking

The `@MainActor` ensures that tasks run on the main thread, which is critical for UI updates. However, long-running tasks on the main actor can block the UI, making the app unresponsive.

**Solution**
- Offload heavy computations to background tasks.
- Use `Task` or `Task.detached` to perform work outside the main thread.

#### Example:
```swift
@MainActor
func updateUI() {
    Task {
        let data = await fetchData()
        display(data)
    }
}
```

---

#### Reduce Actor Contention

Actors serialize access to their state, which can lead to contention when multiple tasks compete for access.

**Solution**
- Minimize shared state.
- Use multiple actors to distribute the workload.

#### Example:
```swift
actor Logger {
    private var logs: [String] = []

    func addLog(_ log: String) {
        logs.append(log)
    }
}
```

---

#### Use Task Priorities Wisely

Assign higher priority to UI-critical tasks and lower priority to background computations.

```swift
Task(priority: .high) {
    await performCriticalTask()
}
```

---

#### Optimize Task Group Usage

Task groups enable efficient parallel execution, but improper usage can lead to wasted resources.

**Tips**
1. Limit the number of tasks in a group to avoid overwhelming the thread pool.
2. Aggregate results efficiently to reduce memory usage.

```swift
await withTaskGroup(of: Int.self) { group in
    for i in 1...5 {
        group.addTask { i * i }
    }
    for await result in group {
        print(result)
    }
}
```

---


#### Combine Concurrency Patterns

Combine tools like async/await, task groups, and actors for more efficient workflows.

```swift
actor DataProcessor {
    private var processedData: [Int] = []

    func process(data: [Int]) async {
        await withTaskGroup(of: Void.self) { group in
            for item in data {
                group.addTask {
                    self.processedData.append(item * item)
                }
            }
        }
    }
}
```

---

#### Distributed Actors

Distributed actors extend actors to support communication across process or network boundaries. They provide a foundation for building distributed systems.

1. **Location Transparency**:
   - The caller doesn’t need to know if the actor is local or remote.
2. **Automatic Serialization**:
   - Parameters and return values are serialized for remote communication.
3. **Fault Tolerance**:
   - Handle failures gracefully in distributed environments.

Use the `distributed actor` keyword for distributed actors.

```swift
import Distributed

distributed actor ChatService {
    func sendMessage(_ message: String) async throws {
        print("Sending message: \(message)")
    }
}
```