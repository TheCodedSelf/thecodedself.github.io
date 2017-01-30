The adapter pattern and the wrapper pattern each solve common but distinct problems. Their common usage and similarities in implementation, however, can lead to confusion. It also doesn't help that both seem to be thrown around as buzz words. The adapter pattern and wrapper pattern are two very useful tools and you can benefit from having them properly labeled in your toolbox.

## Definition
**Adapter:** An adapter allows code that has been designed for compatibility with one interface to be compatible with another. An adapter accomplishes this by transforming the input meant for Interface A into compatible input for Interface B. It is a bridge between two existing interfaces.

**Wrapper:** A wrapper encapsulates and hides the complexity of other code. Third-party code can be hard to use due to the fact that the exposed interface is made to accommodate many use cases. When you are only concerned about a subset of the features or the exposed interface, or you find that using the library is hard or tedious, then what you need is to wrap it in a simpler, more constrained interface.

## Differences
- **Intention:** The end product may look similar but the intention is different. A wrapper is intended to simplify an interface to an external library. An adapter is intended to bridge the disconnect between one interface and another. You may look at a new library that you wish to use and write a wrapper to simplify and streamline its use. You may look at an interface, internal or external, that your existing code needs to conform to, and write an adapter to do that.

- **Composition:** A wrapper contains another object and wraps around it. It has the the sole responsibility of moving data to and from the wrapped object. An adapter doesn't necessarily contain or simplify an object, although this can be a secondary benefit of using adapters. An adapter transforms input to make it match the input required by another interface. It adapts input to that other interface.

- **Problem space:** An adapter solves a problem of incompatibility, while a wrapper fulfills the need of a simplified and specific programming interface.

## Example
Imagine inserting a 2-prong plug into a 3-prong wall socket by using an adapter. That plug adapter is akin to the adapter pattern, whereas the actual plug is a wrapper around the live wires inside.

The live wires have all of the functionality you need, but the **wrapper** - the plug - simplifies things. It has all of the functionality with none of the danger.

Your 2-prong plug isn't compatible with the 3-prong wall socket, however. You need an **adapter** to map the 2-prong plug's interface to what is expected by the 3-prong socket's interface.

### Let's see some code!
Imagine a new feature of your app needs to integrate with social media to get some basic user info. A FacebookConnector class would be a wrapper around Facebook. It facilitates your connection to Facebook. Rather than talk to Facebook's SDK which exposes many different use cases, you create a class that talks to the SDK for you, and only exposes what you need. I have a Facebook wrapper that I used for a Swift app [on my Github profile](https://github.com/TheCodedSelf/iOS-Social-Integration-Example/blob/master/SocialIntegrationPOC/FacebookConnector.swift).

What if you know that you need some particulars from social media - a user's email, a full name, perhaps a profile picture or a personal description? You don't know what particular social media that you'd like to integrate with. Quite frankly, you don't care as long as you can get the info that you need. In this scenario, you'd make an ISocialIntegrator interface, defined as following in C#:

```C#
interface ISocialIntegrator
  {
  void Connect();
  string Email { get; }
  string FullName { get; }
  string Description { get; }
  }
```

Notice how the actual social network is not mentioned. The interface is agnostic of whatever social network is being used under the hood.

Now that you've defined the interface that the rest of your code will interact with, you can start developing against the interface without worrying about its actual implementation. After a two or three well-earned beers, once you make the choice of the particular social media that you'd like to use, you can create an adapter between your SocialIntegrator interface and the social media that you've chosen. You can create a FacebookAdapter, LinkedInAdapter, etc. which will conform to the SocialIntegrator interface and bridge the gap between the generic info that you need and the actual API of the social network that you've chosen.
## Summary
Being cognizant of what problem you are trying to solve and what pattern you are using to solve that problem will help you to keep your code clean and focused on a singular purpose.

Use the wrapper pattern to simplify code, encapsulate third-party dependencies, and eliminate repetition.

Use the adapter pattern to allow yourself to swap out third-party dependencies at will by interacting with your own interface, and then making adapters that map from your own interface to the third party code.

