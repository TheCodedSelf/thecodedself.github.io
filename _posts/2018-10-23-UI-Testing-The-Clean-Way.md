---
title: "UI Testing the Clean Way"
---

I write software, and sometimes I write bugs. Sometimes I write a _lot_ of bugs. But, when I do, I catch them early with testing.

I do manual tests before I commit, I write unit tests as I write my code. Lately, I've also been getting into writing UI tests.

One critique of UI testing is how messy and brittle it can be. Building the tests can be complicated. You might spend hours on a test just for a design to change that breaks the test and ruins all your hard work.

So, I want to tell you about some problems you might face in keeping your UI tests clean and readable, and how we can leverage Swift for this endeavour. 

I'll walk you through my approach to UI testing with everyone's favorite example project - a To-Do list app. I know, boring, right? Well, it works perfectly for the examples we'll be looking at, so... 

![Deal with it]({{ site.url }}/assets/images/UITestingClean/dealwithit.png){: .align-center}

You can download the [example project here](https://github.com/TheCodedSelf/POP-Approach-To-UI-Testing). It's a simple app that allows us to create, edit, and delete a to-do. 

![Walking through the app]({{ site.url }}/assets/images/UITestingClean/walkthru-small.gif){: .align-center}

# Just press record

UI testing in Xcode is easy. Click inside the body of an empty UI test, click the Record button, and you're up and running. 

![Record UI Test Button]({{ site.url }}/assets/images/UITestingClean/PressRecord.png){: .align-center}

Let's start off by recording a UI test to add a new to-do.

![Recording test]({{ site.url }}/assets/images/UITestingClean/Record.gif){: .align-center}

That was easy! And it sure did create a lot of code. That must be good, right?

```swift
func testExample() {
  
  let app = XCUIApplication()
  app.navigationBars["Todo List"].buttons["Add"].tap()
  
  let textField = app.otherElements.containing(.navigationBar, identifier:"New Todo")
    .children(matching: .other).element.children(matching: .other).element
    .children(matching: .other).element.children(matching: .textField).element

  textField.tap()
  textField.tap()
  
  let datePickersQuery = app.datePickers
  datePickersQuery.pickerWheels["October"]/*@START_MENU_TOKEN@*/.press(forDuration: 0.6);/*[[".tap()",".press(forDuration: 0.6);"],[[[-1,1],[-1,0]]],[0]]@END_MENU_TOKEN@*/
  datePickersQuery.pickerWheels["9"]/*@START_MENU_TOKEN@*/.press(forDuration: 0.5);/*[[".tap()",".press(forDuration: 0.5);"],[[[-1,1],[-1,0]]],[0]]@END_MENU_TOKEN@*/
  app.buttons["Done"].tap()
  
}
```

I'll be honest, most of that is unreadable to me. Run the test, and you'll see it doesn't work, either. It completely missed the name of the to-do that we typed in, and the interaction with the date pickers doesn't work either.

It's clear that this approach won't work for a large suite of UI tests. It's a good learning experience to figure out what XCTest has to offer, but code like that shouldn't be part of your code base.

# Not as easy as we thought

There are clearly some problems with the recording approach to UI testing.

## Complicated setup

Each of your tests will start on the main screen of your app. What if you want to test the editing of a to-do? Your test will start by opening a to-do, tapping the Edit button, and then proceed to the actual meat of the test. What if you need to sign in with a particular username and password? Each of your tests will need to do that, as well.

## Duplicated code

There's no code reuse when recording your tests. You can't build any meaningful abstractions, or create helper functions that make the code easier to maintain and understand.

## Hard to read

With lines like this in your code base:

```swift
let textField = app.otherElements.containing(.navigationBar, identifier:"New Todo")
      .children(matching: .other).element.children(matching: .other)
      .element.children(matching: .other).element.children(matching: .textField)
      .element
```

You won't be able to figure out what's going on in your tests a month from now. Enough said.

# A better approach
There are many ways to improve the code that we got from recording a test, but since we're using Swift, why not make use of protocol-oriented programming?

We'll attempt to tackle the three problems of setup, reusability, and readability by using three families of protocols.

## -Starting for setup
Remember the example of tests that start on the Edit screen of our to-do app, or a test that needs the user to be signed in? 

When we need to specify where the test should start, we'll create a `-Starting` protocol to carry that setup code. For instance, `EditScreenStarting`. By conforming to this protocol, we declare that a particular test case starts on the Edit screen. Anyone looking at the protocol conformances of the test case knows exactly what screen will be tested.

## -Interacting for reusability

Recording your tests can spit out some complicated code that you'd need to copy-paste everywhere you want to perform the same action. 

If we want to tap on a to-do named _Buy Groceries_, we'll need to do something like this:

```swift
XCUIApplication().cells
  .containing(.staticText, identifier: "Buy Groceries")
  .firstMatch
  .tap()
```

It requires knowledge that the list of to-dos is a table view with cells that contain a label with the title. It's not that complicated, but if the structure of your app changes and you have this code copy-pasted throughout your test suite, you're gonna have a bad time.

Instead, we'll wrap that into a protocol called `TodoListInteracting` later on. Any test case that is concerned with manipulating to-do list items will have to make its intention clear by conforming to `TodoListInteracting`.

## -Verifying for readability

Tapping on the Plus button in the navigation bar should take you to the Add To-Do screen. This is how we can verify that the user is indeed on the Add screen:

```swift
let app = XCUIApplication()
XCTAssert(app.navigationBars["New Todo"].exists
 && app.staticTexts["Todo Title:"].exists
 && app.buttons[DoneButtonIdentifier].exists)
```

It does the job, but it's not very clear what 'the job' is. Rather than seeing that everywhere we want to verify that we're on the Add screen, we'll just put it into a `AddScreenVerifying` protocol instead. 

When we want to verify that we're on the Add screen, we can just do something like `XCTAssert(addScreenIsShowing())`. Any file that wants to test aspects of the Add screen will have to conform to the `AddScreenVerifying` protocol, making it clear what the test is actually testing.

# Let's get started!

We'll dive straight in and test the deleting of a to-do. Here's the completed test running:

![Delete To-do Test]({{ site.url }}/assets/images/UITestingClean/DeleteTodo4.gif){: .align-center}

In the test, we delete a to-do titled 'Go to Gym'. To delete the to-do, and verify that it was deleted, you need to perform these steps:

- Tap on the to-do titled 'Go to Gym' to show the View To-do screen
- Tap on the delete button
- Ensure that the View To-do screen is no longer showing
- Ensure that no to-do exists with the title 'Go to Gym'

Great, with that item deleted, I'll finally have time for some Netflix! Here's what the test class looks like:

```swift
class ViewTodoTests: UITestCase, ViewScreenStarting, TodoListInteracting, ViewTodoScreenVerifying {
  
  let titleOfTodoForTest = "Go to Gym"
  
  func testCanDeleteTodo() {
    
    // 1
    let delete = XCUIApplication().buttons[Accessibility.View.DeleteButton]
    
    // 2
    delete.tap()
    
    // 3
    XCTAssertFalse(viewTodoScreenIsShowing(forTodoTitled: titleOfTodoForTest))
    
    // 4
    XCTAssertFalse(todo(titled: titleOfTodoForTest).waitForExistence(timeout: 1))
  }
  
}
```

## Body of the test

Most of the magic is being done by protocols, so the actual test is very light: only four lines.

Let's walk through it:

### 1. Finding the delete button
First, we need a handle on the delete button of the View To-do screen. To access a UI element from the tests, you generally use its `accessibilityIdentifier` property. 

Using the accessibilityIdentifier is something that recording your tests won't do. If you have a 'Submit' button for a form, then recorded tests will look for a button titled 'Submit'. If the button title changes, the tests will fail. Using the accessibilityIdentifier is a way to get a handle on the button even if something like its title changes.

To find the delete button, we just look for a button with the `Accessibility.View.DeleteButton` identifier. I've created an `Accessibility` enum to hold the identifiers for our elements. It looks like this:

```swift
enum Accessibility {
  enum View {
    static let DeleteButton = "Delete Todo"
  }
}
```

This enum is a member of the UI Testing target as well as the app target, and the `DeleteButton` identifier is set on the the actual delete button in the app, in the ViewController for the View To-do screen:

```swift
deleteButton.accessibilityIdentifier = Accessibility.View.DeleteButton
```

## 2. Tapping on the button

Now that we have a handle on the delete button, tap it.

```swift
delete.tap()
```

This is the same as physically tapping the delete button on the View To-do screen.

## 3. Verify that the View screen is dismissed

Now that the delete button has been tapped, the View To-do screen should pop and navigate back to the main screen. What do you think this line does?

```swift
XCTAssertFalse(viewTodoScreenIsShowing(forTodoTitled: titleOfTodoForTest))
```

We're asserting that the View To-do screen is not showing for the to-do that we're testing. It reads quite well, doesn't it? 

`viewTodoScreenIsShowing` is defined in the `ViewTodoScreenVerifying` protocol to which this class conforms. 

```swift
protocol ViewTodoScreenVerifying {
    func viewTodoScreenIsShowing(forTodoTitled: String) -> Bool
}

extension ViewTodoScreenVerifying {
    func viewTodoScreenIsShowing(forTodoTitled title: String) -> Bool {
        let app = XCUIApplication()
        let dateHeader = app.staticTexts["Is Scheduled For:"]
        let titleLabel = app.staticTexts[title]
        return dateHeader.exists && titleLabel.exists
    }
}
```

To see if we're on the View To-do screen, we look for a couple of known elements on that screen. There should be a label titled 'Is Scheduled For:', and a label with the title of the to-do. If the composition of the View To-do screen changes, we only need to update the body of this function.

Any test case that is verifying an aspect of the View To-do screen will conform to the ViewTodoScreenVerifying protocol. ViewTodoTests conforms to ViewTodoScreenVerifying, so we know that it's a test case that is verifying aspects of the View To-do screen.

## 4. Verify that the to-do is deleted

In the previous step, we verified that we're no longer on the View To-do screen. Next, we need to make sure that the deleted to-do doesn't exist any more.

```swift
XCTAssertFalse(todo(titled: titleOfTodoForTest).waitForExistence(timeout: 1))
```

We're looking for a to-do with the same title as the one we're testing with `todo(titled: titleOfTodoForTest)`. 

This test needs to interact with an element on the list of to-dos from the main screen, so we conform to TodoListInteracting to make use of this function.

`todo(titled:)` is a convenience function to make the task of finding a to-do a little bit easier and more readable:

```swift
protocol TodoListInteracting {
    func todo(titled: String) -> XCUIElement
}

extension TodoListInteracting {
    func todo(titled title: String) -> XCUIElement {
        return XCUIApplication().cells
            .containing(.staticText, identifier: title)
            .firstMatch
    }
}
```

`waitForExistence` gives the app some time to wait and see if the element comes into existence. We're navigating back to the main screen from the View To-do screen, so we want to wait a bit to make sure that the navigation has completed to verify that the to-do doesn't exist anymore. 

All together now: `todo(titled: titleOfTodoForTest).waitForExistence(timeout: 1)` will wait 1 second, and return false if the to-do does not exist.

# Wait a second...

I've established that the test starts on the View To-Do screen. How did we get there in the first place? 

Let's take another look at the protocol conformance of `ViewTodoTests`.

```swift
class ViewTodoTests: UITestCase, ViewScreenStarting, ViewTodoScreenVerifying
```

The class conforms to `ViewScreenStarting`, indicating that all tests start on the View To-Do screen. Let's take a closer look at `ViewScreenStarting`.

```swift
protocol ViewScreenStarting: TodoListInteracting {
  var titleOfTodoForTest: String { get }
  func startViewScreen()
}

extension ViewScreenStarting {
  func startViewScreen() {
    XCUIApplication().launch()
    todo(titled: titleOfTodoForTest).tap()
  }
  
  func configureStartup() {
    startViewScreen()
  }
}
```

The magic happens inside `startViewScreen`.

First, we launch the app with `XCUIApplication().launch()`. In a UI test, the test starts with the app closed.

ViewScreenStarting also conforms to TodoListInteracting, so it can use the `todo(titled:)` function that we used in the body of our test. We need a to-do to open, so ViewScreenStarting has a `titleOfTodoForTest` property that the test case provides. Inside ViewTodoTests, we set the title of the to-do that we're testing:

```swift
let titleOfTodoForTest = "Go to Gym"
```

Now `startViewScreen` will know which to-do item to tap.

### Running code with StartupConfigurable

`startViewScreen` needs to run when the test starts. To do so, we use `StartupConfigurable`. ViewScreenStarting conforms to StartupConfigurable. It's a simple protocol:

```swift
protocol StartupConfigurable {
    func configureStartup()
}
```

ViewScreenStarting calls `startViewScreen` from this function.

```swift
func configureStartup() {
  startViewScreen()
}
```

In turn, we have a base test class that will call `configureStartup`:

```swift
class UITestCase: XCTestCase {
  override func setUp() {
    super.setUp()
    (self as? StartupConfigurable)?.configureStartup()
  }
}
```

Remember, ViewTodoTests conforms to ViewScreenStarting, which conforms to StartupConfigurable. This means that ViewTodoTests conforms to StartupConfigurable as well. 

When one of the tests from ViewTodoTests start, the `setUp` method in UITestCase will run the necessary startup work.

## Tying it all together

To summarise the `ViewTodoTests` class:

It is a subclass of `UITestCase`, which allows it to run any setup code specified by conformance to the `StartupConfigurable` protocol. Tests will start on the View To-do screen, because it conforms to `ViewScreenStarting`. The boilerplate of the test is cleaned up by `ViewTodoScreenVerifying` and `TodoListInteracting`.

It can be a lot to take in, but through this approach, we gain a very simple and easy to read test:

```swift
func testCanDeleteTodo() {
  let delete = XCUIApplication().buttons[Accessibility.View.DeleteButton]
  delete.tap()
  
  XCTAssertFalse(viewTodoScreenIsShowing(forTodoTitled: titleOfTodoForTest))
  XCTAssertFalse(todo(titled: titleOfTodoForTest).waitForExistence(timeout: 1))
}
```

## What else can we do?

For tests on the Edit screen, you need to navigate to the View screen and then tap the edit button. You could create an `EditScreenStarting` protocol that inherits its functionality from `ViewScreenStarting`:

```swift
protocol EditScreenStarting: ViewScreenStarting {
  func startEditScreen()
}

extension EditScreenStarting {
  
  func startEditScreen() {
    // Navigate to the view screen
    startViewScreen()
    
    // Tap on the edit button
    XCUIApplication().buttons[Accessibility.View.EditButton].tap()
  }
  
  func configureStartup() {
    startEditScreen()
  }
}
```

`-Starting` protocols can cascade - in this example, EditScreenStarting builds off of ViewScreenStarting. You can very easily write tests for a deep and complex ViewController hierarchy in this manner. 

This does a lot to simplify the setup of tests. Combined with the `-Verifying` and `-Interacting` protocols, building UI tests is like working with building blocks - just combine the protocols you want, and the tests are easy!

# Please share your thoughts

What do you think of this approach? It's definitely a work in progress, but I've found it to be useful in my tests. I don't put everything into a protocol, but rather just the repetitive boilerplate, or repeated code that deserves a higher level of abstraction.

Do you have something to share? Please leave it in the comments below!
