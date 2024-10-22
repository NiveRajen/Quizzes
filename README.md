# Quizzes


#Quiz 1
Please explain if this function is fully isolated on the MainActor.

@MainActor
func someWork() async {
	var someVar = ""
	await otherWork()
	someVar = "Done"
}



#Quiz 2

This code MIGHT have ONE error. IF there is one, can you explain what it is in a short sentence (maximum 128 characters long)? Please answer in the application form.

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

#Quiz 3

What potential issues do you see with this code? How would you improve it?


// iOS Interview Question #16
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

Answer: 
Key issues:
Race Condition: Multiple threads access and modify sharedResource without synchronization. Non-Atomic Operation: The += operation isn't atomic, leading to potential data races. Unpredictable Results: The final value of sharedResource is likely to be less than 1000 and may vary between runs.

Solutions 1. Using Atomic Operations

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

Solutions 2. Using Actor (Swift 5.5+)

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

Solutions 3.Using Serial Queue

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


# Quiz 4


Swift Quiz - Closure
-

What would be the output for the below code ?

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

Issue: Race conditions will happen.
Solution 1: Actors
Solution 2: NSLock
Solution 3: Atomic
Solution 4: Semaphore
