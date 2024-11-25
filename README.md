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

# Quiz 9

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


# Quiz 10

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

# Quiz 11

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


# Quiz 12

Swift Quiz
-

```swift
//What is the value of number after executing this code?
extension Int {
    mutating func square() {
        self = self * self
    }
}

var number = 2
number.square()
```
Options:
1. This code generates a compilation error
2. 8
3. 4
4. 2

Output:
4

# Quiz 13

Combine Quiz
-
```swift
//What will be the output?
let value = [1,2,3,4,5,6,7,8,9]
let result = value
		.lazy
		.filter { value in
			value % 2 != 0
		}
		.map { value in
			value * value
		}
		.prefix(3)
```
Output: [1,9,25]


# Quiz 14

Swift Quiz
-
```swift
if (1, "banana") < (2, "apple") {
    print("Alpha")
}
if (2, "mango") < (2, "peach") {
    print("Beta")
}
if (3, "avacado") < (2, "guava") {
    print("Delta")
}
```
Answer: 
Alpha
Beta

# Quiz 15

Swift Quiz
-

Which of the following is not a valid control transfer statement in swift?

1. break
2. goto
3. continue
4. return

Answer: Goto

# Quiz 16

Swift Quiz
-
Given the code snippet below, which aims to create an array of tuples containing the index and value of each element in an array, rewrite it in a more "Swifty" and efficient way.

Before:
```swift
let list = [Int](1...5)
var arrayOfTuples = [(Int, Int)]()

for (index, element) in list.enumerated() {
    arrayOfTuples += [(index, element)]
}

print(arrayOfTuples)
// Expected output: [(0, 1), (1, 2), (2, 3), (3, 4), (4, 5)]
```

After:
```swift
let list = [Int](1...5)
let arrayOfTuples = Array(list.enumerated())
print(arrayOfTuples)
// Expected output: [(0, 1), (1, 2), (2, 3), (3, 4), (4, 5)]
```

# Quiz 17

Swift Quiz
-
Select the correct statement that creates a dictionary wuth a key type of Int and a value type of String

```swift
1. var dict = [1, "Tokyo"]()
2. var dict[Int, String] = [1, "Tokyo"]
3. var dict = [1, "Tokyo"]
4. var dict = [1: "Tokyo"]
```
Answer: 4

# Quiz 18

Swift Quiz
-
What is the value of variable greeting, after executing the following code?

```swift
let firstName: String?
let middleName: String? = nil
let lastName = "Jackson"

let greeting = "Hi, \(firstName ?? middleName ?? lastName)"
```

```swift
1. nil
2. Jackson
3. This code generates a compliation error
4. Hi, Jackson
```

Answer: Option 3, firstName used before being initialized. 

# Quiz 19

Swift Coding Quiz
-
The goal of this exercise is to check the presence of a number in an array.

**Specifications:**
* The items are integers arranged in ascending order.
* The array can contain up to 1 million items
  
Implement the function existsInArray(_numbers: [Int], k: Int) so that it returns true if k belongs to numbers, otherwise the function should return false.
 
**Example:**
let numbers = [-9, 14, 37, 102]
existsInArray(numbers, 102) returns true
existsInArray(numbers, 36) returns false

**Note:** Try to save CPU cycles

```swift
//using swift operator
func existsInArray(_numbers: [Int], k: Int) -> Bool {
    return _numbers.contains(k)
}

//using Binary search
func existsInArrayUsingBinarySearch(_numbers: [Int], k: Int) -> Bool {
    var left = 0
    var right = _numbers.count - 1 
    
    
    
    while left <= right {// 0 <= 0
        let mid = left + (right - left) / 2 
        
        if _numbers[mid] == k { 
            return true
        } else if _numbers[mid] < k { 
            left = mid + 1
        } else {
            right = mid - 1 
        }
    }
    
    return false
}
```

# Quiz 20

Swift Quiz
-
Select the correct statement which can be used to identify if a given variable is an array of Int elements.

```swift
1. if variable == Array {}
2. if variable is [Int] {}
3. if variable == [Int] {}
4. if variable is Array {}
```

**Answer**: 3 


# Quiz 21

Swift Quiz
-
Select the invalid statement from the following answers

```
1. var name: String? = "John"
2. var age: Int = nil
3. var average: Double? = 75
4. let score: Float = 86.7
```
**Answer**: 2


# Quiz 22

Swift Quiz
-

What are the scores for Dave after executing following code?

var scores = ["Dave": [75, 86], "Steve": [94, 78]]

scores["Dave"]?[0] += 1
scores["Brian"]?[0] = 60

**Options:**
1. [94,78]
2. [76,86]
3. [75,86]
4. Compilation error

**Answer**: 2

# Quiz 23

Swift Quiz
-
Select the correct function that takes an argument of a generic type.

**Options:**
1. func function<T>(argument: T)()
2. func function<T>(argument)()
3. func function(argument)()
4. 2. func function(argument: <T>)()

**Answer**: 1

# Quiz 24

Swift Quiz
-
What is the generic class that implements all the basic behaviour of a Core Data model object?
1. Core Data
2. NSManagedObject
3. NSManagedObjectContext
4. SQLite
   
**Answer**: 2
NSManagedObject is a generic class that implements all the basic behavior required of a managed object. You can create custom subclasses of NSManagedObject , although this is often not required.

# Quiz 25

Swift Quiz
-
Select the correct syntax for iterating through the keys and values of a Dictionary type variable called scores.
1. for (key,value) in scores {}
2. for (key,value) in (scores.keys, scores.values) {}
3. for (key,value) in enumerate(scores) {}
4. or value in scores.key {}
   
**Answer**: 1
