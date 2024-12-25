Tasks and Task Groups

---

`Tasks` and `Task Groups` to enable structured and unstructured concurrent operations. These tools allow developers to execute multiple tasks in parallel, manage their lifetimes, handle errors gracefully, and work efficiently with system resources.

---

### Tasks

A Task represents an isolated, concurrent unit of work. It is lightweight compared to threads and provides structured handling of asynchronous operations.

- Child Tasks automatically linked to the parent task and inherit its context (priority, cancellation).
- Detached Tasks operate independently of any parent task, useful for heavy computations or isolated work.
- Tasks suspend at `await` points, freeing threads to perform other work.
- Use `Task.yield()` to voluntarily pause execution, allowing other tasks to progress.
- Pass lightweight state to tasks within the same hierarchy using `@TaskLocal`.

**Retaining Cycle**
- With `actor`:
    - Do not need `[weak self]` because Swift Concurrency ensures safe and isolated access.
	- `[weak self]` is only required in special cases like detached tasks or long-running tasks with non-isolated methods.
- With `class`:
	- Use `[weak self]` to avoid strong reference cycles and retain cycles, especially for UI-related objects or classes.

| **Task Priority**   | **Description**   |
|----------------|-------------------|
| `.high`        |  Tasks that must execute immediately & for critical user interactions *(UI animations, handle immediate touch gestures)*|
| `.medium`      | <br> Tasks that are important but user isn’t actively waiting for *(loading visible images, data needed soon but not immediately)*|
| `.low`         | <br> Non-urgent tasks that can wait if higher-priority tasks are queued *(prefetching data that may be used later, preparing non-critical updates or UI elements)*|
| `.background`  | <br> Long-running or non-visible tasks that can operate in the background without impacting the user experience *(back up data, synchronize large datasets, index or archiving files)*|
| `.default`     | <br> General-purpose tasks that do not require specific prioritization *(fetch data for standard use, perform common computations)*|


#### Examples of Task Usage

**Basic Task Creation**
```swift
Task {
    print("This is a basic task.")
}

Task(priority: .high) {
    print("High-priority task running.")
}

Task.detached {
    print("Detached task running.")
}
```

**Checking for Cancellation**
```swift
Task {
    for i in 0..<10 {
        if Task.isCancelled {
            print("Task cancelled.")
            return
        }
        print("Processing \(i)")
    }
}
```

---

### Task Groups

A `TaskGroup` enables managing multiple concurrent tasks within a single scope. It provides structured concurrency by ensuring all tasks complete before proceeding.

| Type                     | Description                                                                 | Use Case                                                                |
|--------------------------|-----------------------------------------------------------------------------|-------------------------------------------------------------------------|
| **TaskGroup**            | For tasks with results that don’t throw errors.                             | Use when you need results from multiple tasks, and none can throw errors. |
| **ThrowingTaskGroup**    | For tasks that can throw errors, propagating them to the parent context.    | Use when tasks can throw errors, and you need to handle or propagate those errors. |
| **DiscardingTaskGroup**  | For tasks with results that are discarded after execution.                  | Use for side-effect tasks where results are unnecessary (e.g., sending notifications). |
| **ThrowingDiscardingTaskGroup** | For tasks that can throw errors, with results discarded after execution. | Use for side-effect tasks that can throw errors (e.g., long-running cleanup operations). |

--- 

#### Best Practices for Tasks and Task Groups

- Regularly check `Task.isCancelled` to terminate unnecessary work and stop early.
- Use `Task.detached` sparingly to prevent loss of parent context (e.g., task-local values, cancellation).
- Assign appropriate priorities to ensure critical tasks execute promptly.
- Use `@TaskLocal` for lightweight, scoped state-sharing.
- Replace threads `sleep` with `Task.sleep` for non-blocking delays. 
- Move heavy computations in detached task async functions.
- Use `ThrowingTaskGroup` or `ThrowingDiscardingTaskGroup` for tasks that might fail, and catch errors where needed.
- Use `try await` to wait for a task’s result, handling any potential errors that might be thrown.


---

### Examples of Task Group Usage

**Processing Results with `for await`**
```swift
await withTaskGroup(of: Int.self) { group in
    for i in 1...5 {
        group.addTask { i * i }
    }
    for await result in group {
        print("Result: \(result)")
    }
}
```

**Handling Errors in a `ThrowingTaskGroup`**
```swift
try await withThrowingTaskGroup(of: String.self) { group in
    group.addTask { try await fetchData(from: "https://example.com") }
    group.addTask { throw URLError(.badURL) }

    do {
        for try await data in group {
            print(data)
        }
    } catch {
        print("Error: \(error)")
    }
}
```

**Side-Effect Operations with a `DiscardingTaskGroup`**
```swift
await withDiscardingTaskGroup(of: Void.self) { group in
    for _ in 1...5 {
        group.addTask {
            print("Side effect performed.")
        }
    }
}
```

---

### Common Use Cases

**1. Parallel Data Fetching**
```swift
try await withThrowingTaskGroup(of: Data.self) { group in
    for url in urls {
        group.addTask {
            try await fetchData(from: url)
        }
    }
    for try await data in group {
        print(data)
    }
}
```

**2. Batch Processing**
```swift
await withTaskGroup(of: String.self) { group in
    for file in files {
        group.addTask {
            processFile(file)
        }
    }
    for await result in group {
        print("Processed: \(result)")
    }
}
```

**3. Real-Time Updates**
```swift
await withTaskGroup(of: Void.self) { group in
    for stream in streams {
        group.addTask {
            await handleStream(stream)
        }
    }
}
```

**4. Processing Tasks One by One**
```swift
try await withThrowingTaskGroup(of: String.self) { group in
    group.addTask { try await fetchData(from: "https://example1.com") }
    group.addTask { try await fetchData(from: "https://example2.com") }

    while let result = try await group.next() {
        print("Fetched result: \(result)")
    }
}
```

**5. Continue execution in case if task in group failed**
```swift
try await withThrowingTaskGroup(of: String.self) { group in
    group.addTask { try await fetchData(from: "https://example1.com") }
    group.addTask { throw URLError(.badURL) } // Simulate an error

    do {
        for try await result in group {
            print("Fetched result: \(result)")
        }
    } catch {
        print("Error: \(error)")
        while let result = try await group.next() {
            print("Processing remaining task result: \(result)")
        }
    }
}
```