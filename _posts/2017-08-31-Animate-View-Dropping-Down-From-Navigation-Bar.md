---
title: "iOS: Animating your own toast view"
---

A toast view is a small, short-lived popup provides a small bite of information (see what I did there?) to the user. It's an Android paradigm, but if you're working on an iOS app that has an Android component, the chances are high that you have been or will be asked to implement one at some point. So you might as well learn how to make one simply without having to pull in a 3rd-party dependency, right? Well, let's get started then.

I'll focus on helping you get the basics of the animation down. The actual look of the view can be left to more talented designers than myself.

# Animating a drop down view from the navigation bar

We're going to animate the toast coming down from the navigation bar, but the lessons learnt needn't be tied to the exact implementation. Create your animations as you wish. This is the final result that we're hoping to achieve:

<p style="text-align:center"><img alt="Toast animation from UINavigationBar" src="{{ site.url }}/assets/images/ToastAnimation/example.gif"/></p>

## Animating the transform
It's generally a bad idea to modify a view's frame just for the purpose of animation. We'll animate the [transform](https://developer.apple.com/documentation/uikit/uiview/1622459-transform) instead. The responsibility of a view's `frame` is setting the view's place in the view hierarchy. The responsibility of the `transform` is to add any modifications to how the view should look to the user. You can modify the view's transform to rotate it, scale it, or reposition it. We're going to use it to reposition the view on the screen and animate that change.

```
@IBAction private func startToastAnimation(_ sender: Any) {
    
    let toastView = createToastView()
    view.addSubview(toastView)
    animate(toastView: toastView)
}

private func createToastView() -> UIView {
    // 1.
    let toastViewHeight = CGFloat(80)
    let toastView = UIView(frame: CGRect(x: view.frame.origin.x,
                                         y: -toastViewHeight,
                                         width: view.frame.width,
                                         height: toastViewHeight))
    toastView.backgroundColor = .green
    return toastView
}

private func animate(toastView: UIView) {
    
    UIView.animate(withDuration: 1.0, animations: {
        // 2.
        toastView.transform = toastView.transform
            .translatedBy(x: 0, y: toastView.frame.height)
    }, completion: { _ in
        // 3.
        UIView.animate(withDuration: 1.0, delay: 1.0, animations: {
            toastView.transform = CGAffineTransform.identity
        }, completion: { _ in
            // 4.
            toastView.removeFromSuperview()
        })
    })
}
```

### 1.

Firstly, create the toast view. We position it above the current view by setting the `y` position of its frame to be the negative of the view's height. This will put it behind the navigation bar where the user can't see it. It's sneakily hiding behind the scenes until we animate it in.

### 2.

Translate the view's transform by its height. A *translation* is a change in position, similar to how a *rotation* is a change in angle and a *scale* is a change in size. You can do translation, rotation, and scale animations using the following three methods on the transform property:
- `translatedBy(x:y:)`
- `rotated(by:)`
- `scaledBy(x:y:)`

In the first step, we positioned the toast to be above the view by exactly its height. By adding a transform to move it downwards by its full height, we'll bring the toast into position and make it appear as if it's dropping out from behind the navigation bar.

### 3.

Once the animation to the view's transform is completed, reverse the animation. It's incredibly easy to reverse any animation on a view's transform - just set the transform back to `CGAffineTransform.identity`. This is the default, unmodified transform that all views are created with. After playing around with scales, rotations, and translations, you can get a view back into its comfort zone by resetting the transform back to `CGAffineTransform.identity`.

### 4. 

Once the reversal of the animation is completed, remove the view from the hierarchy. Depending on exactly what you're animating, this step may not be necessary. For a toast view however, the view is meant to be transient, so I figured it would be best to respect that mental model of a transient toast and remove it from the view.

# Where to go from here?

Play around with the timing of your animations using [UIViewAnimationOptions](https://developer.apple.com/documentation/uikit/uiviewanimationoptions). You can make an animation ease in or out, or set it to repeat and run in reverse.

And, when you feel ready to continue your journey to animation wizardry, start exploring [UIViewPropertyAnimator](https://developer.apple.com/documentation/uikit/uiviewpropertyanimator). You can easily and concisely customise your animation's timing, dynamically change its direction, and even make it user-interactable.

Have fun, and happy Swifting!
