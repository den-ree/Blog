Tasks and Task Groups

---

**Tasks** and Task Groups are fundamental constructs in Swift Concurrency, enabling developers to run and manage concurrent operations effectively. 

---

A *Task* represents an isolated, concurrent unit of work. Tasks can be grouped into hierarchies and are lightweight compared to threads.

---

## **Task Hierarchy**

### Child Tasks
- Child tasks are automatically linked to their parent task.
- If a parent task is canceled, its child tasks are also canceled.

#### Example:
```swift
Task {
    // Parent task
    await withTaskGroup(of: Void.self) { group in
        for i in 0..<5 {
            group.addTask {
                // Child task
                print("Task \(i)")
            }
        }
    }
}
```

### Detached Tasks
Detached tasks are independent and do not inherit the parent task's cancellation or priority.

#### Example:
```swift
Task.detached {
    print("Detached task running")
}
```

---

## **Task Cancellation**

Tasks can check for cancellation requests using `Task.isCancelled`. It allows tasks to gracefully handle cancellation and perform cleanup.

#### Example:
```swift
Task {
    for i in 0..<10 {
        if Task.isCancelled {
            print("Task was canceled")
            break
        }
        print("Processing \(i)")
    }
}
```

---

## **Task Priority**

Swift Concurrency allows you to assign priority to tasks, influencing their execution order.

### Priority Levels:
- `.high`
- `.medium`
- `.low`
- `.background`
- `.default`

#### Example:
```swift
Task(priority: .high) {
    print("High-priority task")
}
```

---

## **Task Group Patterns**

Task groups provide structured concurrency for managing multiple tasks in a single scope.

### Dynamic Task Group
Add tasks dynamically during execution.

#### Example:
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

### Handling Errors in Task Groups
Use `withThrowingTaskGroup` to handle errors in a group.

#### Example:
```swift
try await withThrowingTaskGroup(of: String.self) { group in
    group.addTask { try await fetchData(from: "https://example.com") }
    group.addTask { throw URLError(.badURL) }
    do {
        for try await data in group {
            print(data)
        }
    } catch {
        print("Error occurred: \(error)")
    }
}
```

---

## **Task-Local Values**

Task-local values allow you to pass values between tasks within the same task hierarchy.

#### Example:
```swift
@TaskLocal static var userID: String?

Task {
    Task.userID = "12345"
    await printUserID()
}

func printUserID() async {
    print(Task.userID ?? "No user ID")
}
```

---

## **Task Traces**

Swift Concurrency provides tools to trace task execution using Instruments.

### Task Tracing Features:
1. **Task Hierarchy Visualization**:
   - View relationships between tasks and their parents.
2. **Execution Timelines**:
   - Analyze task execution times.
3. **State Transitions**:
   - Trace state changes (e.g., pending, running, suspended).

---

## **Best Practices for Tasks and Task Groups**

1. **Manage Cancellation**:
   - Regularly check `Task.isCancelled` to allow early exits.

2. **Avoid Overusing Detached Tasks**:
   - Use only when isolation from the parent task is necessary.

3. **Combine Task Groups with Async/Await**:
   - Use task groups for parallel execution and result aggregation.

4. **Leverage Task Priority**:
   - Assign appropriate priority to critical tasks.

5. **Use Task-Local Values for Scoped State**:
   - Store lightweight state accessible across a task hierarchy.

6. **Other** 
    - Excessive use of await can serialize otherwise parallelizable tasks, leading to performance bottlenecks.
	- Be Intentional About Task Cancellation:
	- Avoid using await if the task is likely to be canceled before completion.
	- Respect UI Responsiveness:
	- Avoid await on long-running operations on the main actor. Offload such work to a background task.
    
---

## **Debugging and Optimization**

1. **Use Instruments for Tracing**:
   - Profile task execution to identify bottlenecks.

2. **Log Task Events**:
   - Add custom logs to trace task creation and completion.

3. **Aggregate Task Metrics**:
   - Monitor task group execution time and resource usage.

---

## **Common Use Cases**

1. **Parallel Data Fetching**:
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

2. **Batch Processing**:
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

3. **Real-Time Updates**:
   ```swift
   await withTaskGroup(of: Void.self) { group in
       for stream in streams {
           group.addTask {
               await handleStream(stream)
           }
       }
   }
   ```

---

## **Conclusion**

Tasks and Task Groups are powerful tools that form the backbone of Swift Concurrency. By understanding their properties, patterns, and advanced features like task-local values and task traces, developers can build efficient, robust, and maintainable concurrent applications.

----
