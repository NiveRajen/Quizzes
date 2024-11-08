# Quizzes


# Quiz 1

Swift Quiz - Concurrency
-
Please explain if this function is fully isolated on the MainActor.

```Swift
@MainActor
func someWork() async {
	var someVar = ""
	await otherWork()
	someVar = "Done"
}
```

Answer: We can say someWork() is fully isolated on Main thread if otherWork() is also marked with @Mainactor, else it will run on different thread.

# Quiz 2

Swift Quiz - SwiftUI
-

This code MIGHT have ONE error. IF there is one, can you explain what it is in a short sentence (maximum 128 characters long)? Please answer in the application form.

```Swift
import SwiftUI

class Score: ObservableObject {
   @Published var points = 0
}

struct ScoreView: View {
   @ObservedObject private var score = Score()
   let teamName: String
  var body: some View {
       Button("\(teamName): \(score.points)") { score.points += 1 }
   }
}

struct BasketballGameView: View {
   @State private var quarter = 1
   var body: some View {
       VStack {
           if quarter <= 4 {
               Button(quarter < 4 ? "Next Quarter" : "End Game") { quarter += 1 }
               HStack {
                   ScoreView(teamName: "Team A")
                   ScoreView(teamName: "Team B")
               }
               Text("Quarter: \(quarter)")
           } else {
               Text("Game Ended")
           }
       }
   }
}
```

Answer: Score instance instantiated on every BasketballGameView update. Inject teamAScore,teamBScore to ScoreView, reset on next quarter.

# Quiz 3

Swift Quiz - Thread Safety Quiz
-

What potential issues do you see with this code? How would you improve it?

```Swift
func testThreadSafetyiniOSQuestion() {
    let group = DispatchGroup()
    var sharedResource = 0
    
    for _ in 1...1000 {
        group.enter()
        DispatchQueue.global().async {
            sharedResource += 1
            group.leave()
        }
    }
    
    group.notify(queue: .main) {
        print("Final value: \(sharedResource)")
    }
}

// function call
testThreadSafetyiniOSQuestion()
```

Answer: 
Key issues:
Race Condition: Multiple threads access and modify sharedResource without synchronization. Non-Atomic Operation: The += operation isn't atomic, leading to potential data races. Unpredictable Results: The final value of sharedResource is likely to be less than 1000 and may vary between runs.

Solutions 1. Using Atomic Operations
```Swift
import Foundation

class AtomicInteger {
    private var value: Int32
    private let lock = DispatchSemaphore(value: 1)
    
    init(value: Int32 = 0) {
        self.value = value
    }
    
    func increment() -> Int32 {
        lock.wait()
        defer { lock.signal() }
        value += 1
        return value
    }
}

func improvedThreadSafetyQuestion() {
    let group = DispatchGroup()
    let sharedResource = AtomicInteger()
    
    for _ in 1...1000 {
        group.enter()
        DispatchQueue.global().async {
            _ = sharedResource.increment()
            group.leave()
        }
    }
    
    group.notify(queue: .main) {
        print("Final value: $$sharedResource.increment())")
    }
}
```
Solutions 2. Using Actor (Swift 5.5+)
```Swift
actor SharedResource {
    private(set) var value = 0
    
    func increment() -> Int {
        value += 1
        return value
    }
}

func actorBasedSolution() async {
    let resource = SharedResource()
    await withTaskGroup(of: Void.self) { group in
        for _ in 1...1000 {
            group.addTask {
                await resource.increment()
            }
        }
    }
    print("Final value: $$await resource.value)")
}
```
Solutions 3.Using Serial Queue
```Swift
func serialQueueSolution() {
    let group = DispatchGroup()
    var sharedResource = 0
    let serialQueue = DispatchQueue(label: "com.example.serialQueue")
    
    for _ in 1...1000 {
        group.enter()
        serialQueue.async {
            sharedResource += 1
            group.leave()
        }
    }
    
    group.notify(queue: .main) {
        print("Final value: $$sharedResource)")
    }
}
```

# Quiz 4


Swift Quiz - Closure
-

What would be the output for the below code ?
```Swift
import Foundation

//1. Vehicle Closure
var vehicle = "cars"

let vehicleClosure = { [vehicle] in
    print("I love \(vehicle)")
  }
vehicle = "bikes"
vehicleClosure()

//2. Furit Closure
var fruit = "mangoes" let fruitClosure = {
  print("I love \(fruit)")
  }
fruit = "banana"
fruitClosure ()
```
Answer: 
1. I love cars
2. I love banana


Reason: 
1 is using capture by value so even though variable is updated but it will print the value that it got at time of creation.
In 2 closure is using capture by reference approach so it will have the updated value.


# Quiz 5


Swift Quiz - Thread Safety Quiz
-

// Is this code thread-safe in a concurrent environment? If not, how would you modify it? 
```Swift
class BankAccount {
  private var balance: Double = 0.0 
  
  func deposit(amount: Double) {
    balance += amount 
  }
  
func withdraw(amount: Double) -> Bool {
  if balance >= amount { 
    balance -= amount 
    return true 
  }
  return false
}

func getBalance() -> Double {
  return balance 
} 
```
Issue: Race conditions will happen.
Solution 1: Actors
Solution 2: NSLock
Solution 3: Atomic
Solution 4: Semaphore


# Quiz 6


Swift Quiz - Swift Structs
-
```Swift
struct UserData {
  var username: String
}

var user1 = UserData(username: "Nivedha")
var user2 = user1
```
In Swift, when you copy a value type, like a struct, both copies (e.g., User1 and User2) will point to the same memory location at first. This means they share the same data. However, once one of them is changed or modified, Swift creates a new copy of the data, giving each copy its own separate memory space. This process is called Copy-On-Write (COW). Here the value isn't modified yet.
References: https://www.youtube.com/watch?v=nb3bRQa0iGQ,
https://developer.apple.com/videos/play/wwdc2016/416,
iOS Memory Deep Dive,
Analyze Heap Memory,
Detect and Diagnose Memory Issues.

# Quiz 7

Swift Quiz - struct
-
```Swift
struct Car {
   var name: String
   private var price: Int = 100
}

let car = Car(name: "xyz")
car.name = "abc"
print(car.name)
```
The above snippet can be written as,
```Swift
struct Car {
   var name: String
   private var price: Int = 100

   init(name: String) {
   	self.name = name
   }
}

let car = Car(name: "xyz")
car.name = "abc"
print(car.name)
```


Answer: Compilation error, 1. you cannot assign "abc" to name property as car instance is "let".
2. Price is a private variable which is not accessible outside.
3. Default initializer for all properties of the struct that's an issue because compiler will try to use private property in default initialization.


# Quiz 8

Swift Quiz - Defer
-
What will be the output when this code is executed?
```Swift
//Consider the following code snippet:

class MyClass {

var myProperty: Int = 0 {
  willSet {
           print("Will set to \(newValue)")
	  }
  didSet {
	   print("Did set from \(oldValue) to \(myProperty)")
	 }
 }

 init() {
   defer { myProperty = 1 }
   myProperty = 2
 }
}

let instance = MyClass()
instance.myProperty = 3
```
Output:
Will set to 2
Did set from 0 to 2
Will set to 1
Did set from 2 to 1
Will set to 3
Did set from 1 to 3

Note: Sometimes, Xcode's console might not display all output immediately, especially if the output is rapid. The output might get buffered, leading to some lines being printed while others might not appear immediately.

# Quiz 8

Swift Quiz - Concurrency
-
```swift
//What will be the output?
actor counter {
   var counter = 0

   func increment() {
      counter += 1
   }
}

let counter = Counter()

Task {
   await counter.increment()
}

print(await.counter.count)
```

Answer: Compiler Error.
Reason: Await can only be used within an asynchronous context, like a function marked async or within a Task block.


# Quiz 8

SwiftUI Quiz
-
```swift
//What will the LazyVGrid display when the following code is executed?
struct ContentView: View {
    let items = Array(1...6).map { "Item \($0)" }
    
    let columns = [
        GridItem(.fixed(100)),
        GridItem(.flexible()),
        GridItem(.fixed(100))
    ]
    
    var body: some View {
        LazyVGrid(columns: columns) {
            ForEach(items, id: \.self) { item in
                Text(item)
                    .padding()
                    .background(Color.blue)
            }
        }
        .padding()
    }
}
```
```swift
//Options:
#1. 6 items displayed in 2 rows with uneven column widths
#2. 6 items displayed in 1 column
#3. 6 items displayed in 3 rows with equal column widths
#4. 6 items displayed in 2 rows with equal column widths

Answer:
6 items displayed in 2 rows with uneven column widths
```

# Quiz 9

Swift Quiz
-

```swift
func addTwoNumbers(_ first: Int, _ second: Int) -> Int {
   return first + second
}
```

```swift
#1. let sum = addTwoNumbers()
#2. let sum = addTwoNumbers(first: 4, second: 6)
#3. let sum = addTwoNumbers(4, 6)
#4. let sum = addTwoNumbers(_ 4, _ 6)

Answer:
#3. let sum = addTwoNumbers(4, 6)
```

Here the argumentLabel is omitted. Hence no need to pass parameter name.
