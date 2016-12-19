Superfluous code is code that is written unnecessarily. It is code that has all the added complexity of valuable code, but it adds no value of its own. If you were to remove it, the product would behave exactly the same. I can think of a few causes of superfluous code:

##### Improper understanding of the toolset
If a developer doesn't properly understand the language or the library that is being used, how can he be certain that the code he writes is suited to the task at hand? Oftentimes code is written that duplicates functionality included in the standard library. Now the same functionality exists in two different places: in your code base, and in your library. It has to be maintained twice. And tested twice. 

For complex algorithms, you're losing the benefit of years and years of bug fixes and performance improvements to the implementation that's available to you through your toolset. Take the time to understand what is available to you. If it suits your needs, use it. Don't write superfluous code when you can write no code and gain the same value from your platform's SDKs. But, if you don't know what is available to you, you're doomed to repeat the bugs of the past.

I recently came across the following Swift code:

```Swift
import Foundation

class Foo {
    var isRegisteredForNotifications = false
    
    init() {
        registerForNotifications()
    }
    
    func registerForNotifications() {
        isRegisteredForNotifications = true
        NotificationCenter.default.addObserver(forName:Notification.Name(rawValue:"MyNotification"),
               object:nil, queue:nil,
               usingBlock:notificationWasFired)
    }
    
    func notificationWasFired(notification: Notification) {
        /*
        .
        .
        */
    }
    
    deinit {
        if isRegisteredForNotifications {
            NotificationCenter.removeObserver(self)
        }
    }
}
```

NotificationCenter is a class in Swift's Foundation library that implements the Observer pattern. You add an object as an observer for a particular notification and tell it what code to call when that notification is fired. The `Foo` class is registering for the `MyNotification` notification and asking NotificationCenter to call `notificationWasFired` when someone fires the notification.

The developer added a flag to the class which he sets to true when the notification is registered, and he then removes the instance as an observer in `deinit`, as one should. All good, but developers that have used NotificationCenter will know that removing an observer is a safe operation. For instance, one could do the following:

```Swift
NotificationCenter.removeObserver(self)
NotificationCenter.removeObserver(self)
NotificationCenter.removeObserver(self)
```
Without any negative impact. It's known that removing an observer is safe, even if the observer is not registered for any notifications. There's no reason to include the overhead of the extra flag which has to be set, checked, and maintained.

##### Changing requirements
Requirements change as businesses evolve. It's one of the reasons our jobs are so hard. Sometimes, you write code that was useful yesterday but it has no functionality today. It may no longer be called anywhere. Or you may be calling a function whose result is no longer used. If the function's return value is never accessed, and the function produces no other side effects, why keep it? If you need it later, your source control will remember it. 

##### False sense of safety

```C
var factory = new WidgetFactory();
if (factory != null) {
	Console.WriteLine(factory.name);
} else {
	return;
}

var config = new WidgetConfiguration();
if (factory != null) {
	factory.createWidgetWithConfiguration(config);
}
```

Null checks are good. NullReferenceExceptions are bad. But is that second null check really necessary? If nothing is modifying `factory`, don't check it twice. And if your application is a tangled ball of threads, and you have no idea if `factory` will still be a valid pointer at the time you need it, then maybe those race conditions are a more important problem to solve.

##### Premature optimization
Writing more performant code usually leads to less readable code. That's why you'll hear the advice time and time again that you should only optimize parts of the application that has proven to be a problem through the inspection of metrics. Any optimization that is done without the inspection of current performance should be seen as superfluous, as it does not satisfy any requirement of the application.

## Why superfluous code is a problem
When you come across code like this, whether it's fresh code in a pull request, or in a legacy class that you're factoring, it's your duty to remove it. Why?

##### It breaks convention.
There's a superfluous null check before accessing variable `foo`. What about the other location where `foo` was accessed? What about when we access `bar`? What about other classes that access objects of the same type as `foo`? Was `foo` a special case? Or is this now best practice? **Too many questions!** Luckily, you can remove all ambiguity by removing the superfluous code.

##### Extra code means extra capacity for bugs
We need to write code because that's how we build shippable features. Code adds value but more code means more complexity and more places things can go wrong. If the code you're writing doesn't add value or perform a necessary function, it only serves to increase the capacity for bugs.

##### The code that isn't written has 100% test coverage, now and into the future
Someone might argue that the code is safe from bugs if it's covered by tests. But is the code covered by the right tests? Have all possible permutations been covered? What about future changes? You can't know about future changes. There can't be any guarantee that the code will be tested six months from now. But I can guarantee you that 100% of the code that I *didn't* write is still 100% tested.

##### Extra code means extra cognitive load
Open a class in your project. Any class. Now select everything in the body of the class and delete it. It's much easier to understand something that's empty. As we add more functionality to the class, there's more that a reader needs to take in to fully understand that class. We are increasing the cognitive load when we add to the class, but it's a small price to pay for the obvious gain: a (hopefully) valuable product. However, when you add superfluous code, you're increasing the cognitive load without adding value. The class is now harder to read while providing zero extra gain.
## Summary
More code means more complexity. More complexity means a heightened possibility for bugs, and more time to understand the intention of the code. Every time we add code we are making a tradeoff: more complexity for more value. I want the code that adds value. But if I'm making a tradeoff where I add complexity and gain no value, that's not a deal that I want to be a part of.


## P.S.
Thank you for listening to my ramblings on my first ever blog post. Please leave a comment or send me a mail at thecodedself@gmail.com with any feedback you have!
