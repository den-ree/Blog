Async/Await

The `async/await` syntax is a cornerstone of Swift Concurrency, allowing to write asynchronous code that looks and behaves like synchronous code.

An `async` **function** is a function that can perform asynchronous work. It allows the function to suspend its execution until an asynchronous operation completes.

The `await` **keyword** is used within an `async` function to pause execution until the asynchronous task completes without blocking the thread on which it's called. This makes asynchronous workflows appear sequential, avoiding the complexity of nested callbacks.

---

#### Best Practices

- Use `async let` to run independent tasks concurrently, but avoid excessive parallelism. If you have dynamic number of tasks, then use `Task Group`.
- Use `throws` in async functions and handle errors with `do-catch` calling `try await` for clean error to propagation.

---

#### Examples of Async/Await

**Sequential Execution**
```swift
let user = try await fetchUser()
let posts = try await fetchPosts(for: user.id)
return UserData(user: user, posts: posts)
```

**Parallel Execution**
```swift
async let first = fetchData1() // begins execution immediately in a child task.
async let second = fetchData2()
let results = await [first, second] // wait until both tasks finished
```

**Error Handling with Async/Await**
```swift
func fetchData() async throws -> Data {
    do {
        return try await URLSession.shared.data(from: URL(string: "https://example.com")!).0
    } catch {
        // Handle errors and propaganate error with defined information
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