---
permalink: /functions-deserve-injection/
title: "Functions Deserve Injection, Too"
---

Lately, I've been taking advantage of Swift's functional abilities where it makes sense to help me write concise and clear code that's easy to test. I'd like to share one technique that has helped me to eliminate repetition and breakages of encapsulation in tests: **function injection**.

As a code base evolves over time, a lot of small, focused utilities are added. Business rules are clearly defined and abstracted away, helper classes are created to cut down on the code repetition. 

Here's one small utility function that I use quite often:

```
extension String {
    func isNonEmpty() -> Bool {
        return !trimmingCharacters(in: .whitespacesAndNewlines).isEmpty
    }
}
```

This eliminates the verbosity of checking if a string is empty or contains only whitespace. It is, understandably, quite easy to test as well.

```
    func testNoCharactersReturnsTrue() {
        XCTAssertFalse("".isNonEmpty())
    }
    
    func testOnlyBlankSpacesReturnsTrue() {
        XCTAssertFalse("         ".isNonEmpty())
    }
    
    func testNewlineReturnsTrue() {
        XCTAssertFalse("  \n   ".isNonEmpty())
    }
    
    func testNonEmptyStringReturnsFalse() {
        XCTAssertTrue("Hello there".isNonEmpty())
    }
```

As an example, imagine that we're using this in an app for a clothings store. When you walk into the store, you should get a push notification from the app with a greetings message containing the user's name. If there's something wrong with the user's name - if it's empty or whitespace - we'll throw an error.

```
struct Greeter {
    func greet(name: String) throws -> String {
        guard name.isNonEmpty() else { throw GreetingError.invalidName }
        return "Welcome, \(name)! Have you seen the specials on offer?"
    }
}
```

Let's write some tests for this function. 

We want to test two things:
- The returned message is correct when the name is valid
- An error is thrown when the business logic determines that a name is invalid

```
    func testGreetingWithValidNameReturnsGreetingString() {
        let expected = "Welcome, Caroline! Have you seen the specials on offer?"
        let actual = try? Greeter().greet(name: "Caroline")
        XCTAssertEqual(expected, actual)
    }
    
    func testGreetingWithEmptyNameThrowsInvalidNameError() {
        verifyGreetingErrorIsThrown(whenGiven: "")
    }
    
    func testGreetingWithBlankSpaceNameThrowsInvalidNameError() {
        verifyGreetingErrorIsThrown(whenGiven: "   ")
    }
    
    func testGreetingWithNewlineCharacterNameThrowsInvalidNameError() {
        verifyGreetingErrorIsThrown(whenGiven: "  \n ")
    }
    
    func verifyGreetingErrorIsThrown(whenGiven name: String) {
        do {
            try Greeter().greet(name: name)
            XCTFail("An error should have been thrown")
        } catch GreetingError.invalidName {
            return
        } catch {
            XCTFail("Incorrect error type was thrown")
        }
    }
```

Now, that's a lot of tests for such a small function...

![So many tests? Interesting.]({{ site.url }}/assets/images/FunctionInjection/interesting.gif){: .align-center}

## The Problem

If you look closely, you'll see that we're basically repeating all of the negative tests of the `isNonEmpty` function. If we follow this approach, we'll be repeating the tests at every use site of `isNonEmpty`. That breaks the DRY principle, and it could be much worse if you had more complex logic than simply checking if a user typed a bunch of spaces for their username.

When we're testing the `greet` function, we don't actually want to be testing what it means for a string to be empty or not. That's already been defined somewhere else in the code base. We simply want to test what the function does when it works, and how it fails when it doesn't.

When you're testing a component that depends on another, we make use of Dependency Injection to pass the dependence in. That way we can control the behavior. It keeps tests short and focused.

We could refactor the `isNonEmpty` function to its own class, and inject it either by using a dependency container or as a parameter to the `greet` function, but that seems like overkill. This is where the functional nature of Swift comes in. Rather than creating a class to pass as a parameter, we can **apply dependency injection by injecting the function itself.**

## The Solution

Rather than directly calling isNonEmpty in the greet function, we'll pass it in as a parameter.

```
    func greet(name: String,
               isNonEmpty: (String) -> Bool = { $0.isNonEmpty() == false })
        throws -> String {
            guard isNonEmpty(name) else { throw GreetingError.invalidName }
            return "Welcome, \(name)! Have you seen the specials on offer?"
    }
```

The new parameter has a default value set to call the intended closure. We can use it in production code without worrying about the extra parameter: 

`greeter.greet(name: "Batman")`

This adheres to the "Clarity at the point of use" section of the [Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/#fundamentals). 

Now, when testing, we don't need to check every permutation of the isNonEmptyFunction. We just inject a closure that returns true for testing the happy path, and we return false when testing the failure case.

```
    func testGreetingReturnsGreetingStringWhenNotEmpty() {
        let expected = "Welcome, Caroline! Have you seen the specials on offer?"
        let actual = try? Greeter().greet(name: "Caroline", isNonEmpty: { _ in true })
        XCTAssertEqual(expected, actual)
    }
    
    func testGreetingThrowsInvalidNameErrorWhenEmpty() {
        do {
            try Greeter().greet(name: "asdf", isNonEmpty: { _ in false })
            XCTFail("An error should have been thrown")
        } catch GreetingError.invalidName {
            return
        } catch {
            XCTFail("Incorrect error type was thrown")
        }
    }
```

From four tests down to two! The tests are now focused solely on the purpose of the greet function and no longer leak the implementation of isNonEmpty.

## Please Share Your Thoughts

I've put the complete example code in a Playground [on my GitHub account](https://github.com/TheCodedSelf/FunctionInjection). I like how straightforward testing is when following this approach in the right scenario. I do feel that it can sometimes add complexity to the production code solely for the sake of the tests, which is a caveat that means I use this pattern somewhat sparingly and only when it makes sense. The most common use case for me is small utility functions that build on other utility functions.

What do you think? Do you have any alternatives? Let me know!  

Have fun, and happy Swifting!
