# Combine-Concept-Guide
This will show the usage of combine framework
---
## 🧩 Advanced Topics
---
### Explain Combine framework
Combine is Apple’s reactive programming framework.
---

```swift
import Combine

class ViewModel: ObservableObject {
    @Published var text = ""
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        $text.sink { print("Text changed: \($0)") }.store(in: &cancellables)
    }
}
```
---

# 📘 Combine Framework --- Interview Ready Guide

## What is Combine?

Combine is Apple's reactive programming framework for handling
asynchronous events and data streams. It provides **Publishers**,
**Subscribers**, and **Operators**. Think of it as a modern alternative
to Delegates, Closures, and NotificationCenter.


## Basic Terms

  **Publisher**         Emits a sequence of values over time (e.g., API
                        responses, text field updates).

  **Subscriber**        Receives and reacts to those emitted values.

  **Operator**          Modifies values between publisher and subscriber
                        (e.g., `map`, `filter`, `debounce`).

  **Cancellable**       A token to stop a subscription when you no longer
                        need updates.

## Basic Example --- Just Publisher

``` swift
import Combine

let publisher = Just("Hello Combine")
let subscriber = publisher.sink { value in
    print("Received:", value)
}
```

**Explanation:** - `Just` is a simple publisher that emits one value and
finishes. - `sink` is a subscriber that receives that value. - Output:
`Received: Hello Combine`

------------------------------------------------------------------------

## Combine with Multiple Operators

``` swift
import Combine

let numbers = [1, 2, 3, 4, 5]
let publisher = numbers.publisher

let subscriber = publisher
    .filter { $0 % 2 == 0 }   // Take only even numbers
    .map { $0 * 10 }          // Multiply each by 10
    .sink { value in
        print("Received:", value)
    }
```

**Explanation:** - The array becomes a publisher. - `filter` passes only
even numbers. - `map` modifies them. - Output:
`Received: 20   Received: 40`

------------------------------------------------------------------------

## Combine with URLSession (Most Common Interview Example)

``` swift
import Combine
import Foundation

struct User: Codable {
    let id: Int
    let name: String
}

class NetworkManager {
    var cancellables = Set<AnyCancellable>()
    
    func fetchUsers() {
        guard let url = URL(string: "https://jsonplaceholder.typicode.com/users") else { return }
        
        URLSession.shared.dataTaskPublisher(for: url)
            .map { $0.data }
            .decode(type: [User].self, decoder: JSONDecoder())
            .receive(on: DispatchQueue.main)
            .sink(
                receiveCompletion: { completion in
                    switch completion {
                    case .finished:
                        print("✅ Finished fetching users")
                    case .failure(let error):
                        print("❌ Error:", error)
                    }
                },
                receiveValue: { users in
                    print("👤 Users:", users)
                }
            )
            .store(in: &cancellables)
    }
}
```

**Explanation:** - `dataTaskPublisher` publishes (data, response) from a
network call. - `.map` extracts the data. - `.decode` converts JSON to
`[User]`. - `.receive(on:)` ensures UI updates happen on the main
thread. - `.sink` handles completion and values. - `.store(in:)` keeps
the subscription alive until you cancel or the object deallocates.

✅ **Interview Tip:** You can explain that Combine simplifies async data
handling and replaces old patterns like Delegates and Closures.

------------------------------------------------------------------------

## Combine with @Published and ViewModel (MVVM Integration)

``` swift
import Combine
import Foundation

class CounterViewModel: ObservableObject {
    @Published var count = 0
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        $count
            .filter { $0 % 2 == 0 }
            .sink { value in
                print("Even count:", value)
            }
            .store(in: &cancellables)
    }
    
    func increment() {
        count += 1
    }
}
```

**Explanation:** - `@Published` automatically creates a publisher for
the property. - `$count` allows Combine to observe changes. - `.sink`
reacts whenever `count` changes. - This pattern is heavily used in
**SwiftUI MVVM**.

------------------------------------------------------------------------

## Important Operators for Interview

  -------------------------------------------------------------------------------------------------------------
  Operator               Description                   Example
  ---------------------- ----------------------------- --------------------------------------------------------
  `map`                  Transforms data               `.map { $0 * 2 }`

  `filter`               Filters emitted values        `.filter { $0 > 10 }`

  `debounce`             Waits before emitting         `.debounce(for: .seconds(1), scheduler: RunLoop.main)`

  `merge`                Combines multiple publishers  `publisher1.merge(with: publisher2)`

  `combineLatest`        Emits when any of multiple    `combineLatest(textPublisher, sliderPublisher)`
                         publishers emit               
  -------------------------------------------------------------------------------------------------------------

## Common Interview Questions

1.  What is Combine used for?
2.  Difference between Combine, async/await, and Delegates?
3.  What is the role of Cancellable?
4.  Explain `map`, `filter`, and `decode`.
5.  How to handle networking with Combine?
6.  How to cancel a subscription?
7.  How is Combine used with SwiftUI's `@Published` properties?

---
# Role of `Cancellable` in Combine (Swift)
---

## 🧩 What is `Cancellable`?

`Cancellable` is a **protocol** in Combine that represents **a
connection (subscription)** between a **Publisher** and a
**Subscriber**.\
When you subscribe to a publisher, Combine returns a `Cancellable`
object --- typically stored in a variable like `AnyCancellable`.\
If you cancel it, the publisher stops sending values, freeing memory and
resources.

------------------------------------------------------------------------

## 💡 Why It's Needed

Combine creates **long-living data streams** (e.g., network requests, UI
bindings).\
If these aren't stopped manually, they keep emitting data --- leading to
**memory leaks** or **unnecessary background work**.

`Cancellable` lets you: - Stop the subscription anytime\
- Prevent memory leaks\
- Control when data flow should end

------------------------------------------------------------------------

## 🧠 Example

``` swift
import Combine

class ExampleViewModel {
    var cancellable: AnyCancellable?

    func fetchData() {
        cancellable = URLSession.shared.dataTaskPublisher(for: URL(string: "https://api.example.com")!)
            .map(\.data)
            .sink(receiveCompletion: { completion in
                print("Completed:", completion)
            }, receiveValue: { data in
                print("Received \(data.count) bytes")
            })
    }

    func cancelRequest() {
        cancellable?.cancel()
        print("Request cancelled")
    }
}

let viewModel = ExampleViewModel()
viewModel.fetchData()

// Later when not needed
viewModel.cancelRequest()
```

### 🧾 Output

    Request cancelled

Here: - `sink` returns an `AnyCancellable` - When you call `cancel()`,
the subscription stops --- Combine will not send more values or
completion events.

------------------------------------------------------------------------

## 🧱 Common Patterns

### 1. Store multiple cancellables

``` swift
var cancellables = Set<AnyCancellable>()

publisher
    .sink { value in print(value) }
    .store(in: &cancellables)
```

### 2. Automatic cancellation

When the object (e.g., `ViewModel`) deinitializes, all cancellables in
its set are automatically cancelled.

------------------------------------------------------------------------

## ⚙️ Summary

  Aspect           Description
  ---------------- -----------------------------------------------------
  **Type**         Protocol
  **Purpose**      Manage and cancel Combine subscriptions
  **Prevents**     Memory leaks, unnecessary background work
  **Common Use**   Stored in `AnyCancellable` or `Set<AnyCancellable>`
  **Analogy**      Like `Task.cancel()` in `async/await`

------------------------------------------------------------------------

## 🔁 Comparison Tip

-   **Combine's `Cancellable`** → Cancels data streams (reactive
    pipelines).\
-   **`Task` in async/await** → Cancels concurrent async work.\
    Both are mechanisms for **cleaning up async operations** safely.

---
# Difference between Combine, async/await, and Delegates in Swift
---

## 🧩 Overview Table

  --------------------------------------------------------------------------------
  Feature          Combine                 async/await          Delegates
  ---------------- ----------------------- -------------------- ------------------
  **Introduced     iOS 13                  iOS 15               iOS 2 (very old)
  In**                                                          

  **Type**         Reactive Framework      Structured           Design Pattern
                                           Concurrency          

  **Paradigm**     Publisher--Subscriber   Sequential,          One-to-one
                   (Reactive)              Synchronous-like     communication
                                           async code           

  **Best For**     Handling multiple async Simple async tasks   Passing
                   data streams or event   (network calls, file data/events
                   chains                  IO, etc.)            between two
                                                                objects

  **Data Flow**    One-to-many (multiple   One-time async       One-to-one
                   subscribers)            response             

  **Complexity**   High                    Medium               Low

  **Memory         Uses `AnyCancellable`   Automatic (`Task` &  Uses `weak`
  Handling**                               structured           delegate reference
                                           concurrency)          

------------------------------------------------------------------------

## 🧠 1. Delegates

**Oldest and simplest** pattern for one-to-one communication between
objects.

### ✅ Use When

-   You need to send data/events **from one object to another**
    (e.g. from a child view controller to parent).
-   Example: TableView delegates, custom callbacks.

### 🧩 Example

``` swift
protocol DataDelegate: AnyObject {
    func didReceiveData(_ data: String)
}

class Sender {
    weak var delegate: DataDelegate?
    
    func fetchData() {
        delegate?.didReceiveData("Hello from Delegate!")
    }
}

class Receiver: DataDelegate {
    func didReceiveData(_ data: String) {
        print(data)
    }
}

let sender = Sender()
let receiver = Receiver()
sender.delegate = receiver
sender.fetchData()
// Output: Hello from Delegate!
```

------------------------------------------------------------------------

## ⚡️ 2. Combine

**Reactive programming** framework by Apple.\
Used for chaining and transforming multiple asynchronous streams (like
network responses, UI updates, notifications).

### ✅ Use When

-   You need **reactive streams** (multiple async updates over time).
-   Example: Combine network publisher → decode → update UI.

### 🧩 Example

``` swift
import Combine
import Foundation

class APIService {
    var cancellable: AnyCancellable?

    func fetchData() {
        cancellable = URLSession.shared.dataTaskPublisher(for: URL(string: "https://api.example.com")!)
            .map(\.data)
            .decode(type: [String].self, decoder: JSONDecoder())
            .sink(receiveCompletion: { print($0) },
                  receiveValue: { print("Data: \($0)") })
    }
}
```

------------------------------------------------------------------------

## 🕒 3. async/await

Introduced in Swift 5.5 (iOS 15).\
Makes asynchronous code look **synchronous and linear**, improving
readability.

### ✅ Use When

-   You have **one-time async calls** like fetching data, writing files,
    etc.
-   You want **clean, modern async syntax** without callbacks or
    publishers.

### 🧩 Example

``` swift
func fetchData() async throws -> [String] {
    let url = URL(string: "https://api.example.com")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode([String].self, from: data)
}

Task {
    do {
        let result = try await fetchData()
        print("Data:", result)
    } catch {
        print("Error:", error)
    }
}
```

------------------------------------------------------------------------

## 🧭 When to Use Which

  -----------------------------------------------------------------------
  Scenario                        Best Option
  ------------------------------- ---------------------------------------
  One-to-one communication        **Delegate**
  between objects (simple)        

  Handling continuous event/data  **Combine**
  streams                         

  Simple, single async task (like **async/await**
  network calls)                  

  Complex pipelines (e.g. data →  **Combine**
  transform → UI update →         
  logging)                        

  Migrate legacy code             Start moving **Delegates →
                                  async/await** or **Combine**
  
------------------------------------------------------------------------

### 🚀 Tip

-   **Delegates** are great for UIKit patterns and simple callbacks.\
-   **async/await** is best for modern async logic (clean and
    readable).\
-   **Combine** is powerful for complex reactive scenarios (but overkill
    for simple async calls).

---
 # 🧩 Combine Framework – Advantages, Keywords, and Usage
---

## 🚀 Advantages of Combine

Combine provides a **declarative, consistent, and type-safe** way to handle asynchronous data  
(events, network calls, user input) with less boilerplate and better readability.

Use Combine for **reactive programming in Swift** — such as:

- Binding UI to data  
- Handling network responses  
- Form validation  
- Search debounce  
- Chaining async operations in a clean pipeline  

---

## 🧠 Keywords and Usage

### 🔹 `cancellables`
A collection that stores subscriptions (`AnyCancellable`) to keep Combine pipelines alive until explicitly cancelled or deallocated.

### 🔹 `debounce`
Delays publishing values until no new values are received for a specified time interval, useful for reducing rapid updates (e.g., typing).

### 🔹 `removeDuplicates`
Prevents publishing of consecutive duplicate values, ensuring downstream only receives changes.

### 🔹 `assign(to:)`
Automatically assigns published values to a property, keeping it updated with the latest emitted value.

### 🔹 `sink`
Subscribes to a publisher and allows you to handle received values (and optionally completion events) manually.

### 🔹 `CombineLatest`
Watches multiple publishers and emits combined values whenever **any** of them changes.

### 🔹 `Merge`
Combines multiple publishers so that any of them can emit a value independently.

---

## 🧩 Example 1 – Using CombineLatest

```swift
Publishers.CombineLatest($email, $password)
    .map { email, password in
        return email.contains("@") && password.count >= 6
    }
    .assign(to: &$isValid)
```

### 🧾 Explanation
- `$email` and `$password` are `@Published` properties — publishers that emit values whenever they change.  
- `Publishers.CombineLatest` takes these two publishers and **combines** them.  
- Every time either `email` or `password` changes, it gives the latest values of both.  
- `.map { email, password in ... }` checks validation logic.  
- `.assign(to: &$isValid)` automatically updates the `isValid` property with the result.

✅ So, `isValid` becomes `true` only if the email and password are valid.

---

## 🧩 Example 2 – Using Merge

```swift
Publishers.Merge($email, $password)
    .sink { [weak self] _ in
        guard let self = self else { return }
        self.showMessage = false
        self.isUserFound = false
    }
    .store(in: &cancellables)
```

### 🧾 Explanation
- `Publishers.Merge($email, $password)` emits values **whenever any one** of them changes.  
- `.sink { ... }` subscribes to the publisher and reacts to changes.  

When the user types in either field:
- `showMessage = false` → hides the “User Found / Not Found” message.  
- `isUserFound = false` → resets the login result.  
- `.store(in: &cancellables)` keeps the subscription alive; otherwise, it cancels immediately.

---

## 🧠 Summary

| Concept | Description |
|----------|--------------|
| Combine | Declarative framework for managing async data |
| cancellables | Holds Combine subscriptions |
| debounce | Delays emissions to prevent rapid updates |
| CombineLatest | Combines latest values of multiple publishers |
| Merge | Emits values when any publisher changes |
| sink | Subscriber to handle values manually |

---
