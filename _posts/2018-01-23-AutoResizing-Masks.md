---
permalink: /autoresizing-masks/
title: "AutoResizing Masks and You"
---

You're an Autolayout Wizard. You know Interface Builder like the back of your hand. Then, one day, you create a simple UIView, add it as a subview with some elegantly crafted constraints, and.. it blows up in your face. What gives?

Take a look at this code example that tries to create a UILabel and center it in the view.

```
let centerLabel = UILabel()
centerLabel.text = "Perfectly centered!"
view.addSubview(centerLabel)

NSLayoutConstraint.activate([
  centerLabel.centerXAnchor.constraint(
    equalTo: view.centerXAnchor, constant: 0),
  centerLabel.centerYAnchor.constraint(
    equalTo: view.centerYAnchor, constant: 0)
  ])
```

Try this, and you'll be thoroughly disappointed. The perfectly centered label is nowhere to be found. 

![A disappointing result]({{ site.url }}/assets/images/AutoResizing/empty.png){: .align-center}

Luckily, the console output provides a vital clue. 

```
[LayoutConstraints] Unable to simultaneously satisfy constraints.
	Probably at least one of the constraints in the following list is one you don't want. 
	Try this: 
		(1) look at each constraint and try to figure out which you don't expect; 
		(2) find the code that added the unwanted constraint or constraints and fix it. 
	(Note: If you're seeing NSAutoresizingMaskLayoutConstraints that you don't understand, refer to the documentation for the UIView property translatesAutoresizingMaskIntoConstraints) 
(
    "<NSAutoresizingMaskLayoutConstraint:0x60400009cb60 h=--& v=--& UILabel:0x7f927f2021b0'Perfectly centered!'.midY == 0   (active)>",
    "<NSLayoutConstraint:0x608000099140 UILabel:0x7f927f2021b0'Perfectly centered!'.centerY == UIView:0x7f927f1017b0.centerY   (active)>",
    "<NSLayoutConstraint:0x60400009cd90 'UIView-Encapsulated-Layout-Height' UIView:0x7f927f1017b0.height == 568   (active)>",
    "<NSAutoresizingMaskLayoutConstraint:0x60400009ce30 h=-&- v=-&- 'UIView-Encapsulated-Layout-Top' UIView:0x7f927f1017b0.minY == 0   (active, names: '|':UIWindow:0x7f927bd073a0 )>"
)

Will attempt to recover by breaking constraint 
<NSLayoutConstraint:0x608000099140 UILabel:0x7f927f2021b0'Perfectly centered!'.centerY == UIView:0x7f927f1017b0.centerY   (active)>
```

Looks like it's a problem with Autolayout constraints.

`
[LayoutConstraints] Unable to simultaneously satisfy constraints.
`

It goes on to say:

`
(Note: If you're seeing NSAutoresizingMaskLayoutConstraints that you don't understand, refer to the documentation for the UIView property translatesAutoresizingMaskIntoConstraints) 
`

Looking at the conflicting constraints in the console output, we're getting the `NSAutoresizingMaskLayoutConstraints` that were just mentioned. 

`
<NSAutoresizingMaskLayoutConstraint:0x6080000993c0 h=--& v=--& UILabel:0x7ff037604a20'Perfectly centered!'.midY == 0   (active)>
`

So, it's finally time to ask:

## What the heck is an Autoresizing Mask?

Before Auto Layout, the current layout system on iOS, views were resized using the *springs and struts* layout system. A view’s width and height are its *springs* which can stretch. The *struts* are the view’s distance to its container. You can imagine that the view is ‘sitting’ on the struts, held in place, to determine the view’s position. The view’s springs can stretch as necessary to determine the size and shape of the view.

The *autoresizing mask* determines which springs and struts can change to position the view in different layouts. 

The autoresizing mask can be any combination of these values: 
* `UIViewAutoresizingFlexibleBottomMargin`: Allows the bottom strut (the distance between the bottom of the view and its container) to change
* `UIViewAutoresizingFlexibleTopMargin`: Allows the top strut (the distance between the top of the view and its container) to change
* `UIViewAutoresizingFlexibleLeftMargin`: Allows the left strut (the distance between the left of the view and its container) to change
* `UIViewAutoresizingFlexibleRightMargin`: Allows the right strut (the distance between the right of the view and its container) to change
* `UIViewAutoresizingFlexibleHeight`: Allows the height spring to change
* `UIViewAutoresizingFlexibleWidth`: Allows the width spring to change

This is a lot easier to visualize by looking at the example at Interface Builder gives you. Drag any view into a nib or storyboard. Before adding any constraints, choose the Size Inspector tab in the right pane (Alt-Cmd-5) and play with the values in the Autoresizing section.

![Autoresizing example]({{ site.url }}/assets/images/AutoResizing/Autoresizing-Example.gif){: .align-center}

## Why does it cause conflicting constraints?

At the introduction of Autolayout, a lot of views were still laid out using springs and struts, both internal to UIKit as well as in third-party apps. For this reason, UIKit creates constraints automatically to adapt the springs and struts layout system to Autolayout.

If the `translatesAutoresizingMaskIntoConstraints` property of your UIView is set to true, UIKit will create Autolayout constraints that best fit your view's autoresizing mask. 

`translatesAutoresizingMaskIntoConstraints` is automatically set to false when you create a view inside Interface Builder and start to add your own constraints. You don't have this luxury if you're laying out your view in code, and you'll have to set the property manually.

So, if you add constraints to a view created in code, you'll most likely see conflicting constraints until you set `translatesAutoresizingMaskIntoConstraints` to false.

Making that change in the previous example will cause the label to display correctly.

```
let centerLabel = UILabel()

// Don't forget this!
centerLabel.translatesAutoresizingMaskIntoConstraints = false

centerLabel.text = "Perfectly centered!"
view.addSubview(centerLabel)

NSLayoutConstraint.activate([
  centerLabel.centerXAnchor.constraint(
    equalTo: view.centerXAnchor, constant: 0),
  centerLabel.centerYAnchor.constraint(
    equalTo: view.centerYAnchor, constant: 0)
  ])
```

![Autoresizing example]({{ site.url }}/assets/images/AutoResizing/perfectly-centered.png){: .align-center}

Looking good!

# You probably won't need the autoresizing mask.

You'll almost always be using Autolayout when laying out your views. Interface Builder sets `translatesAutoresizingMaskIntoConstraints` to false for you, and you'll need to do it yourself when creating views in code.

I hope this post has been useful to you! You might already know to flip the `translatesAutoresizingMaskIntoConstraints` switch when you're seeing strange behavior, but it's helpful to get some background to help you remember. 

Thanks for taking the time to read this. If you enjoyed it, please share it with your friends. I'd also love to hear your suggestions in the comments below. 
{: .notice--info}
