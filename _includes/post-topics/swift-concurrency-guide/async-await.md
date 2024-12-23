Async/Await in Swift Concurrency

---

The `async/await` syntax is a cornerstone of Swift Concurrency, allowing developers to write asynchronous code that looks and behaves like synchronous code.

---

### Async Functions & Await keyword
An `async` function is a function that can perform asynchronous work. It allows the function to suspend its execution until an asynchronous operation completes.

The `await` keyword is used within an `async` function to pause execution until the asynchronous task completes without blocking the thread on which it's called. This makes asynchronous workflows appear sequential, avoiding the complexity of nested callbacks.

---

### Examples of Async/Await

**Sequential Execution**
```swift
let user = try await fetchUser()
let posts = try await fetchPosts(for: user.id)
return UserData(user: user, posts: posts)
```

**Parallel Execution**
```swift
async let first = fetchData1() // begins execution immediately as a child task.
async let second = fetchData2()
let results = await [first, second] // wait until both tasks finished
```

**Error Handling with Async/Await**
```swift
func fetchData() async throws -> Data {
    do {
        return try await URLSession.shared.data(from: URL(string: "https://example.com")!).0
    } catch {
        throw CustomError.networkError
    }
}
```

**Use MainActor for UI Updates**
```swift
Task {
    let data = await fetchData() 
    await MainActor.run {
        updateUI(with: data) // data should be @Sendable
    }
}

Task { @MainActor in
    updateUI()
}
```

---

### Best Practices

- Avoid using `sleep` or heavy computations in async functions.
- Avoid excessive parallelism of tasks (for complex usage use Task Group)
- Use `async let` to run independent tasks concurrently.
- Use `throws` in async functions to handle errors cleanly.
- Use `do-catch` and `try` with `await` for clean error propagation.

### Limitations of Async/Await

- Only available in Swift 5.5+.
- Legacy APIs with completion handlers need migration.
- Async functions can only be called from other async functions or `Task`.

---

## **Advanced Async/Await Features**

1. **Task Cancellation**:
   - Use `Task.isCancelled` to check for cancellation and handle cleanup.
   ```swift
   func performTask() async {
       while !Task.isCancelled {
           // Perform work
       }
   }
   ```

2. **Task Priority**:
   - Set task priority using `Task(priority: .high)`.

---

## **Debugging Async/Await**

Use Xcode's async call stack to trace async function calls. Instruments can also profile and debug tasks using the Swift Concurrency Instrument.

---
