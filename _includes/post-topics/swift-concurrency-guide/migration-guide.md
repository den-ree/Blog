Migration Guide

Migrating to Swift Concurrency involves transitioning existing asynchronous codebases to use modern features like `async/await`, actors, and task groups.

#### Best Practices for Migration

- Migrate one module or feature at a time.
- Use `@MainActor` for UI-related concurrency.
- Provide examples of the new concurrency approach for your team.
- Replace legacy patterns gradually to minimize disruptions.
- Replace locks (`DispatchQueue.sync`) with `actor` declarations.
- Replace manual thread handling (`DispatchQueue`) with `Task`.
- Test migrated functions with `XCTest` using `async/await`.
- Use Instruments to identify bottlenecks and optimize task execution.

---

#### 1. Identify Asynchronous Workflows

Start by identifying parts of your codebase that rely on:
- Completion handlers.
- Dispatch queues (`DispatchQueue`).
- Legacy threading models like `OperationQueue` or `pthread`.

---

#### 2. Replace Completion Handlers with Async/Await

- Convert legacy completion handler-based code to use `async/await`:

**Before**
```swift
func fetchData(completion: @escaping (Result<Data, Error>) -> Void) {
    URLSession.shared.dataTask(with: URL(string: "https://example.com")!) { data, _, error in
        if let error = error {
            completion(.failure(error))
        } else if let data = data {
            completion(.success(data))
        }
    }.resume()
}
```

**After**
```swift
func fetchData() async throws -> Data {
    try await withCheckedThrowingContinuation { continuation in
        legacyFetchData { data, error in
            if let error = error {
                continuation.resume(throwing: error) // Resume with an error.
            } else if let data = data {
                continuation.resume(returning: data) // Resume with the result.
            } else {
                continuation.resume(throwing: NSError(domain: "UnknownError", code: 0)) // Handle unexpected cases.
            }
        }
    }
}
func legacyFetchData(completion: @escaping (Data?, Error?) -> Void) {
    DispatchQueue.global().async {
        // Simulate some data fetching
        let success = Bool.random()
        if success {
            completion(Data("Fetched data".utf8), nil)
        } else {
            completion(nil, NSError(domain: "ExampleError", code: 1, userInfo: nil))
        }
    }
}
```

--- 

#### 3. Introduce Actors for Shared State

- Replace locks, semaphores, and other manual synchronization mechanisms with actors.

**Before**
```swift
class SharedCounter {
    private var lock = NSLock()
    private var count = 0

    func increment() {
        lock.lock()
        count += 1
        lock.unlock()
    }

    func getCount() -> Int {
        lock.lock()
        defer { lock.unlock() }
        return count
    }
}
```

**After**
```swift
actor Counter {
    private var count = 0

    func increment() {
        count += 1
    }

    func getCount() -> Int {
        return count
    }
}
```

---

#### 4. Adopt Task Groups for Parallel Work

- Task groups are ideal for transitioning batch-processing logic from dispatch queues.

**Before**
```swift
DispatchQueue.global().async {
    let group = DispatchGroup()

    for i in 1...5 {
        group.enter()
        processItem(i) {
            group.leave()
        }
    }

    group.notify(queue: .main) {
        print("All tasks completed")
    }
}
```

**After**
```swift
await withTaskGroup(of: Void.self) { group in
    for i in 1...5 {
        group.addTask {
            await processItem(i)
        }
    }
}
print("All tasks completed")
```

---

#### 5. Migrate to AsyncSequence for Streams

- Transform existing observable patterns into `AsyncSequence`.

**Before**
```swift
class DataStream {
    var callback: ((Int) -> Void)?

    func start() {
        DispatchQueue.global().async {
            for i in 1...5 {
                self.callback?(i)
            }
        }
    }
}
```

**After**
```swift
let stream = AsyncStream(Int.self) { continuation in
    for i in 1...5 {
        continuation.yield(i)
    }
    continuation.finish()
}

for await value in stream {
    print(value)
}
```

---

#### 6. Handle Compatibility Issues

- Older APIs that rely on completion handlers can be bridged using async alternatives.

```swift
func fetchLegacyData() async throws -> Data {
    try await withCheckedThrowingContinuation { continuation in
        legacyFetchData { data, error in
            if let error = error {
                continuation.resume(throwing: error)
            } else if let data = data {
                continuation.resume(returning: data)
            }
        }
    }
}
```
