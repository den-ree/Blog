AsyncSequence for Async Streams

---

`AsyncSequence` is a protocol introduced in Swift Concurrency to handle asynchronous streams of values. It is analogous to the `Sequence` protocol but works asynchronously, enabling developers to process data that arrives over time, such as network streams or real-time updates.

It produces a sequence of values asynchronously. Consumers use the `for await` loop to retrieve these values one at a time, pausing execution until the next value is available.

- Asynchronous Iteration
- Values are generated only when requested.
- Automatically handles slower consumers without overloading resources.

**Declaring an AsyncSequence**

You can create your own `AsyncSequence` by conforming to the protocol and implementing the `AsyncIterator` type.

---

`AsyncStream` is a built-in utility for creating and consuming asynchronous sequences. It is especially useful for bridging asynchronous data sources into the `AsyncSequence` paradigm.

---

#### **Best Practices for AsyncSequence**

- Prefer `AsyncStream` when handling event-based or dynamic data (updates for real-time data)
- Implement proper error propagation in custom sequences.
- Use appropriate buffering policies to avoid excessive memory usage.
- Always ensure tasks and streams respect cancellation signals.

#### Example of AsyncSequence & AsyncStream:

**Consuimng AsyncSequence**
```swift
let counter = Counter()

for await number in counter {
    print("Number: \(number)")
}
```

**Creating an AsyncStream**
```swift
let stream = AsyncStream(Int.self) { continuation in
    for i in 1...10 {
        continuation.yield(i)
    }
    continuation.finish()
}

// Consuming an AsyncStream
for await value in stream {
    print("Value: \(value)")
}
```

**Creating a custom AsyncSequence**
```swift
struct Counter: AsyncSequence {
    typealias Element = Int

    struct AsyncIterator: AsyncIteratorProtocol {
        var current = 0

        mutating func next() async -> Int? {
            guard current < 10 else { return nil }
            defer { current += 1 }
            return current
        }
    }

    func makeAsyncIterator() -> AsyncIterator {
        return AsyncIterator()
    }
}
```

**Error Handling**:
```swift
struct FaultyCounter: AsyncSequence {
    typealias Element = Int

    struct AsyncIterator: AsyncIteratorProtocol {
       var current = 0

    mutating func next() async throws -> Int? {
        guard current < 5 else { throw CustomError.limitReached }
        defer { current += 1 }
           return current
        }
   }

    func makeAsyncIterator() -> AsyncIterator {
        return AsyncIterator()
    }
}
```

**Buffered AsyncStream**:
```swift
let bufferedStream = AsyncStream(Int.self, bufferingPolicy: .bufferingOldest(5)) { continuation in
    for i in 1...100 {
        continuation.yield(i)
    }
    continuation.finish()
}
```

**Cancellation**
```swift
Task {
    for await value in stream {
       print("Processing \(value)")
        if value == 5 { break }
    }
}
```

**Real-Time Updates**
```swift
func fetchLivePrices() -> AsyncStream<Double> {
    AsyncStream { continuation in
        Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { timer in
            continuation.yield(Double.random(in: 100...200))
        }
    }
}
```