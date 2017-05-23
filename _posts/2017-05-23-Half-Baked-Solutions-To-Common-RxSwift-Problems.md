---
title:  "Half-Baked Solutions to Common RxSwift Problems"
---

I've been using [RxSwift](https://github.com/ReactiveX/RxSwift) in some of my latest iOS projects. While I feel that it's provided elegant UI binding and a natural feel to asynchronous programming, I have to admit that it came with numerous growing pains. I struggled to find documented solutions to some of the challenges I was facing, as RxSwift is not as widely adopted as its more affluent Java and .Net counterparts. Here are some of those hurdles that I faced, along with my solutions.

## Why can’t I access the tap method of my UIButton?

One of the greatest benefits of Rx is its UI binding capabilities. So you must imagine how peeved off I was when I couldn't do something as simple as bind some logic to taps on a UIButton. Luckily, the solution is just as simple as the problem.

```
import UIKit
import RxSwift
import RxCocoa

let button = UIButton()
button.rx.tap.bind {
    print("That wasn't too hard, was it?")
}
```

See that sneaky `import RxCocoa` at the top there? RxSwift functionality relating to UIKit is found in the RxCocoa module. The `rx` property on UIButton that I'm using in the snippet comes from RxCocoa. You can find it on most UIKit objects. It's what powers RxSwift's interaction with UIButton, UITableView, UIImageView, and all of the other usual suspects. You can find an RxSwift extension on most UIKit objects - just look out for the object plus `Rx.swift` - e.g. `UITableView+Rx.swift`. 

## How do I combine the output from multiple observables into one?

If you have data from multiple sources, and want to treat them as one, you need to merge them.

```
let cars = Observable.from(["Nissan", "Ford", "BMW"])
let tanks = Observable.from(["Tiger II", "T-44", "Panther"])
let bikes = Observable.from(["Ducati", "Honda", "Kawasaki"])

let landVehicles = Observable.of(cars, tanks, bikes).merge()

landVehicles.subscribe(onNext: { print($0) })

/* Output:
Nissan
Ford
Tiger II
BMW
T-44
Ducati
Panther
Honda
Kawasaki
*/
```

First, we combine the three observables using `of`. You can use the `of` operator on any list of items to create an Observable from said items. Used here, it gives us an Observable of Observables.

The `merge` operator is used to flatten out an Observable of Observables. We use it to create the `landVehicles` observable, which is an Observable of Strings.

## How do I make one observable depend on another?

I have one observable that initiates a service call when I subscribe to it. However, I only want to execute that service call if my view passes validation successfully. I have another observable that returns an `error` event if validation fails. If the validation observable submits an `error` event, the service call observable should not execute. 

Here's the solution that I came up with:

```
func submit(number: Int) {
    let validateAndPerformServiceCallIfSuccessful =
        performValidation(numberThatShouldBeEven: number)
            .flatMap { () -> Observable<Void> in // 1
                print("Validated successfully")
                return performServiceCall() // 2
    }
    
    validateAndPerformServiceCallIfSuccessful.subscribe(onNext: { _ in
        print("Service call was a success!")
    }, onError: { _ in
        print("Something went wrong.")
    })
}

private func performValidation(numberThatShouldBeEven: Int) -> Observable<Void> {
    if numberThatShouldBeEven % 2 == 0 {
        print("Validation succeeded.")
            return Observable.just(())
    } else {
        print("Validation failed.")
        let error = NSError(domain: "com.thecodedself.rxswift",
                            code: 0,
                            userInfo: nil)
        return Observable.error(error)
    }
}

private func performServiceCall() -> Observable<Void> {
    print("Performing service call")
    return Observable.just(()) 
}
```

If an even number is passed to the `submit` function, the service call is executed. If the validation fails, an error message is printed on the screen. 

Here's an explanation of what's happening:

1. `flatMap` over the `performValidation` observable. When you flatmap an observable, any time the observable sends a `next` event, your closure will execute and you'll return another observable. If the observable that you're flatmapping sends a `completed` or `error` event, the observable is finished executing - that means no service call.
2. Any time the `performValidation` observable sends a `next` event, return the `performServiceCall` observable. What this means is that the `validateAndPerformServiceCallIfSuccessful` observable will send an `error` or `completed` event if either performValidation or performServiceCall send an error or competed event. The meat of it lies in the `flatMap` closure - if the performValidation observable sends a `next` event, the performServiceCall observable gets to do its thing.

This pattern does seem complex, but it can be insanely powerful once you learn it and use it to your advantage.

## Testing with RxSwift is horrible. How can I make it less horrible?

Well, that's a very valid point. Testing against the RxSwift framework can be difficult on its own, involving asynchronicity and a lot of complex logic in your tests. Luckily, we have *RxBlocking*.

RxBlocking is a library in the same GitHub repository as RxSwift. If you're using Cocoapods, just add `pod 'RxBlocking'` to your test target.

No dealing with `TestSchedulers` and `XCTestExpectations`. If you can work with an array, you can work with RxBlocking.

```
class FibonacciFinderTests: XCTest {
    
func testThatThreeFollowingFibonacciNumbersAreReturned() {
    let expectedNumbers = [13, 21, 34]
    
    guard let actualNumbers = try? FibonacciFinder().findThreeFollowing(number: 9)
        .toBlocking()
        .toArray() else {
            XCTFail()
            return
    }
    
    XCTAssertEqual(expectedNumbers, actualNumbers)
    }
}
```

The above snippet tests the `findThreeFollowing` function on the `FibonacciFinder` class, which, given a number, should return the first three numbers that follow it in the fibonacci sequence.

Calling `toBlocking` on an observable returns a *Blocking Observable*, which is an observable with added functionality to block your thread of execution so that you can test it without having to worry about asynchronicity. We then call `toArray`, which handles subscribing to the observable and returning any `next` events neatly packaged in an array for us for easy of testing. Note that `toArray` can throw exceptions, which is why we have the try statement and a guard clause.

# Still a lot more to learn

As I write more RxSwift code, and occasionally run into barriers, I learn a little bit more about the library and improve the quality and readability of my reactive solutions. It's tough in the beginning, but as you get more comfortable, RxSwift starts to show its utility. Don't be afraid to dive in and read the implementation code and try different things.

Enjoy your Swifting!
