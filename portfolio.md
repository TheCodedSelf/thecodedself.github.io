---
layout: single
title: Portfolio
permalink: /portfolio/
header_alignment: center
gallery_traverse:
  - url: /assets/images/Portfolio/traverse1.jpg
    image_path: assets/images/Portfolio/traverse1.jpg
    alt: "Traverse iOS App"
  - url: /assets/images/Portfolio/traverse2.jpg
    image_path: assets/images/Portfolio/traverse2.jpg
    alt: "Traverse iOS App"
gallery_raywenderlich:
  - url: /assets/images/Portfolio/rw-logo.png
    image_path: assets/images/Portfolio/rw-logo.png
    alt: "raywenderlich.com, the most popular iOS tutorial site on the Internet"
gallery_zenbreath:
  - url: /assets/images/Portfolio/zenbreath1.jpg
    image_path: assets/images/Portfolio/zenbreath1.jpg
    alt: "Zen Breath iOS App"
  - url: /assets/images/Portfolio/zenbreath2.jpg
    image_path: assets/images/Portfolio/zenbreath2.jpg
    alt: "Zen Breath iOS App"
gallery_hushtime:
  - url: /assets/images/Portfolio/hushtime1.png
    image_path: assets/images/Portfolio/hushtime1.png
    alt: "Hush Time macOS App"
  - url: /assets/images/Portfolio/hushtime2.png
    image_path: assets/images/Portfolio/hushtime2.png
    alt: "Hush Time macOS App"
gallery_colorgame:
  - url: /assets/images/Portfolio/colorgame1.png
    image_path: assets/images/Portfolio/colorgame1.png
    alt: "Color Game iOS App"
  - url: /assets/images/Portfolio/colorgame2.png
    image_path: assets/images/Portfolio/colorgame2.png
    alt: "Color Game iOS App"
  - url: /assets/images/Portfolio/colorgame3.png
    image_path: assets/images/Portfolio/colorgame3.png
    alt: "Color Game iOS App"
---

<br/>

# Traverse: Explore Your World

{% include gallery id="gallery_traverse" %}

<ul id="menu">
  <li><a target="_blank" rel="noopener" href="https://apps.apple.com/us/app/traverse-explore-your-world/id1369515409">App Store</a></li>
  <li><a target="_blank" rel="noopener" href="http://www.thecodedself.com/gettraverse/">Website</a></li>
</ul>

Traverse lets you collect your runs, cycles, and explorations on the world map. Tracking a new area reveals it on the map and lets you know where you still need to explore. Fully written in Swift.

Uses the following:
- CoreGraphics for masking unexplored areas
- MapKit for the world map
- Strava API for syncing activities
- CoreLocation for activity tracking
- In-App Purchases for monetization

# raywenderlich.com

{% include gallery id="gallery_raywenderlich" %}

<ul id="menu">
  <li><a target="_blank" rel="noopener" href="https://www.raywenderlich.com/5542-enum-driven-tableview-development">Enum-Driven TableView Development
</a></li>
  <li><a target="_blank" rel="noopener" href="https://www.raywenderlich.com/8164-push-notifications-tutorial-getting-started">Push Notifications: Getting Started
</a></li>
  <li><a target="_blank" rel="noopener" href="https://www.raywenderlich.com/u/keeganrush">All Tutorials</a></li>
</ul>

I write for raywenderlich.com, the most popular iOS tutorial site on the Internet.

# Zen Breath

{% include gallery id="gallery_zenbreath" %}

<ul id="menu">
  <li><a target="_blank" rel="noopener" href="https://itunes.apple.com/us/app/zen-breath/id1071857375?ls=1&mt=8">App Store</a></li>
  <li><a target="_blank" rel="noopener" href="https://github.com/TheCodedSelf/zen-habits-reader">GitHub</a></li>
</ul>

An iOS client for a blog named Zen Habits. Written in Objective-C and published to the iOS App Store.

Uses the following:
- Core Data to store posts
- Cocapods for third-party dependencies
- In-App Purchases and Google AdMob for monetization
- Google Analytics to provide user insights

<br/>

# Hush Time

{% include gallery id="gallery_hushtime" %}

<ul id="menu">
  <li><a target="_blank" rel="noopener" href="{{ site.url }}/Hush-Time">Website</a></li>
  <li><a target="_blank" rel="noopener" href="https://github.com/TheCodedSelf/Hush-Time">GitHub</a></li>
</ul>

A macOS app written in Swift. Select the apps you'd like to close, choose a duration for your focused block of time, and hit Start.

Uses the following:
- AppleScript to close apps
- An NSOpenPanel to select a list of apps
- Simple reactive concepts to manage the current state

<br/>

# SwiftCron

<ul id="menu">
  <li><a target="_blank" rel="noopener" href="https://github.com/TheCodedSelf/SwiftCron">GitHub</a></li>
</ul>

An open-sourced cron expression parser library written in Swift. Cron expressions are a highly flexible means of scheduling tasks to run periodically at fixed times, dates, or intervals. 

SwiftCron is 90% covered by unit tests and works anywhere Swift works, including Linux.

Uses the following:
- Heavy use of the Apple's date and time APIs for parsing cron expressions
- Cocoapods, Carthage, and Swift Package Manager for installation
- Travis CI and Codecov for automated builds and testing

<br/>

# Color Game

{% include gallery id="gallery_colorgame" %}

<ul id="menu">
  <li><a target="_blank" rel="noopener" href="https://github.com/TheCodedSelf/Color-Game">GitHub</a></li>
</ul>

An iOS game demonstrating the [Stroop Effect](https://en.wikipedia.org/wiki/Stroop_effect) written in Swift. The name of a color flashes on the screen, printed in a different color (e.g. the word "Green", but colored "Red"). The user taps a button corresponding to the *color of the word* (Red). The rounds get faster with each win, so stay sharp.

Uses the following:
- A CAShapeLayer and UIBezierPath to animate the 'time remaining' ring
- Consumable In App Purchases and iAd video adverts for monetisation
