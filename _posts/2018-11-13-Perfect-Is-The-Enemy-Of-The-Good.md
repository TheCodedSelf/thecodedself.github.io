---
title: "Perfect is the Enemy of the Good"
---

<h3 style="margin-top: 0px; color: #777777;">
<em>How My Architecture Crippled Development</em>
</h3>

---

A couple of years ago, I was working on a side project with a few friends. We thought that it would be the next big thing. We put our collective best efforts into it; I worked long, hard hours fleshing out the scaffolding of the perfect architecture. Little did I know that my effort would doom the project to join the abyss of failed projects as quickly as it had begun.

## We had the best tools

![All the best iOS Swift tools]({{ site.url }}/assets/images/PerfectEnemyGood/toolbox.png){: .align-center}

Since this was *The Next Big Thing*™, we used everything at our disposal. 

- RxSwift for reactive programming
- VIPER for our architecture pattern
- Swinject for dependency injection
- Cuckoo for mocking
- Flow Operations for managing navigation

The **Flow Operations** were a particularly interesting concept, inspired by the [Advanced NSOperations](https://developer.apple.com/videos/play/wwdc2015/226/) session from WWDC 2015. We used Flow Operations to manage navigation in the app. For instance, if you wanted to register a new user, you'd invoke a `RegisterFlowOperation`.

A FlowOperation was a subclass of Operation:

```swift
class FlowOperation: Operation
```

The [Operation](https://developer.apple.com/documentation/foundation/operation) class represents the code and data for a task of your choosing. It also handles concurrency and dependencies. So, our Flow Operations represented the task of flowing from one screen to another in an app.

Operations can be dependent on each other - for instance, the `EditProfileFlowOperation` is dependent on the `SignInFlowOperation`. If you've already signed in, you can edit your profile, but if you haven't, then you'll be directed to sign in if you invoke the `EditProfileFlowOperation`.

## How does it work?

There was a lot of hidden complexity in the Flow Operation system, and some trickiness that you wouldn't notice until you started using it. I'll briefly go over some of the code.

You could sell products in this app we were building. This is how you'd start the Sell Flow:

```swift
private func startSellFlow() {
  DispatchQueue.global().async { [navController = navController] in
    let flow = SellFlowOperation(navigationController: navController)
    flow.beginFlow()
    flow.waitUntilFinished()
  }
}
```

You create the flow operation, call `beginFlow` to navigate to the first screen, and then `waitUntilFinished` to prevent other flows from executing until this one is done. 

But, don't forget to put it on a background queue, or the app will hang. Also, don't forget to call `waitUntilFinished`, or nothing interesting will happen at all. If another flow was already executing, yours wouldn't run until that one finished. I shudder when I think back on the hours of debugging issues like this. 💀

A flow can have one or more ViewControllers to display in succession. How? With a bunch of complicated RxSwift code, of course.

```swift
func present(viewControllers: Observable<UIViewController>,
             finishWhenDone: Bool = false) {
  viewControllers.observeOn(MainScheduler.instance)
    .subscribe { [weak self] event in
      if case .next(let viewController) = event {
        self?.pushViewController(viewController)
      } else if finishWhenDone {
        self?.finish()
      }
    }.addDisposableTo(disposeBag)
}
```

It subscribes to an RxSwift Observable of UIViewControllers that come from the Presenter portion of the VIPER architecture. When a new ViewController is received from the FlowAction, it's pushed onto the navigation stack. When there are no more ViewControllers, the FlowAction finishes (if `finishWhenDone` is true).

A Router object ➡️  invokes the Flow Operation ➡️  which gets View Controllers from a Presenter ➡️  and then pushes them onto the navigation stack.

## Technically...

It's a great system. It's a complex, well-thought out architecture. Dependencies are managed elegantly and each part of the system has its own single responsibility.

Sounds good, right?

## In practice...

It's a mess. The architecture is extremely hard to use. You need to remember to:

- Invoke the FlowOperation on the background thread, or you'll cause a deadlock.
- Call `beginFlow` to start the flow, or nothing will happen.
- Supply ViewControllers from a Presenter or no new screens will display.
- Call `finish` when you're done to allow the next flow to start.
- Don't create dependency cycles between flows or the app will crash.

Complex architectures create complex problems. Building the right architecture for the right app is extremely important. I'm not advocating for spaghetti code. But when you build unnecessary complexity into an architecture like the one I just described, you're asking for trouble.

Developers follow the **path of least resistance**. A hard to use architecture causes a lot of resistance. When no one can figure out how to put a simple ViewController on the screen, they'll just opt for MVC instead. If you want your team using something other than Massive View Controllers, you need to provide something that's easy to use, or it won't be used at all.

With a hard-to-use system like the Flow Operations, it's probably better that they follow the path of least resistance and default to MVC anyways. A system with so many 'gotchas' makes it extremely easy to write buggy code. *Remember the background thread. Remember to avoid dependency cycles.* The more tricky bits you need to remember, the more likely you are to create bugs and spend more time in debugging than building actual features.

# A better approach

What could have changed in the Flow Operation architecture that might have led the project to better success?

### Easier

Your architecture shouldn't make life harder. If you want things to be hard, program in assembly. I'd much rather build my apps in a way that's easy to build now, and easy to maintain later.

### Enabler

The structure you have in place is not meant to be a barrier that keeps you from moving forward. Following VIPER or MVVM provides you with some scaffolding, but building something new shouldn't be a complicated process that's prone to errors. These architectural patterns are there to make development faster, in the long run. 

When you have to keep a list of tricky problems in your head just to push a new ViewController onto the screen, then something's gone wrong.

### Don't make me think

How easy is the system to understand? How easy was it to write? Will you be able to comprehend things 6 months from now? What about a new developer on the team: will they be able to ramp up without too many headaches in the process?

This is why I've fallen out of love with frameworks like RxSwift. It makes code hard to understand, and harder for team members that are less versed in the library. I love the benefits of reactive programming, but I'd rather take a simpler approach such as using [property observers](https://medium.com/the-andela-way/property-observers-didset-and-willset-in-swift-4-c3730f26b1e9).

# Getting there

There are a few things that I like to keep in mind when building an architecture. This works for scaffolding a project, building a class interface, or choosing third-party frameworks.

### The right tool for the job

Complexity has its time and its place. Techniques that flourish in a large team of ten or more developers aren't necessarily the right approach for your side projects. A complicated architecture shouldn't be the starting point - add in the extra structure when your team grows and your needs evolve.

### Rule of Three

Preventing code duplication seems like the #1 rule of clean programming. While removing code duplication is a commendable goal, **don't do it** at the cost of creating an incorrect abstraction. When you're looking at duplicated code, do the two pieces in question share a similar purpose? Or is their similarity merely coincidental?

Code duplication is bad. Creating leaky and incorrect abstractions is worse. They will mutate your code base into an abstract mess that requires a lexicon to make sense of.

### YAGNI

You Ain't Gonna Need It. For every possible future scenario that you can think of, 90% of them won't happen. Don't spend your time preparing for the possibility that a project manager might have a change of heart six months down the line.

Rather write clean, flexible code that can adapt to change. Write code that's well tested and easy to change when the time comes. When a change of plans presents itself, you'll be ready. But if you try and prepare for every possible future outcome? YAGNI.

# The result

The unnecessary complication that I introduced to the side project mentioned earlier made development a pain. It was hard to build, and hard to use. The other developers on the project battled against the Flow Operations. RxSwift was a huge learning barrier for those that didn't know it. And, for a small app and team like ours, VIPER was most likely overkill.

If, instead, we followed the principles mentioned in this post, we could have built something that was easy to understand. The less you have to think when figuring out how to use it, the better. It should be easy to use. An enabler to your programming, not a barrier.

The system should be only as complex as it needs to be to solve the problem at hand. If the problem changes, the complexity of the system can change to follow suit. 

# Please share your thoughts

Do you have any horror stories from failed projects? Do you have any thoughts to share on this advice, or some of your own? Please leave it in the comments below!
