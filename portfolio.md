---
layout: single
title: Portfolio
permalink: /portfolio/
header_alignment: center
gallery_zenbreath:
  - url: /assets/images/Portfolio/zenbreath1.jpg
    image_path: assets/images/Portfolio/zenbreath1.jpg
    alt: "Zen Breath"
  - url: /assets/images/Portfolio/zenbreath2.jpg
    image_path: assets/images/Portfolio/zenbreath2.jpg
gallery_hushtime:
  - url: /assets/images/Portfolio/hushtime1.png
    image_path: assets/images/Portfolio/hushtime1.png
    alt: "Hush Time"
  - url: /assets/images/Portfolio/hushtime2.png
    image_path: assets/images/Portfolio/hushtime2.png
    alt: "Hush Time"
---

# Hush Time


<ul id="menu">
  <li><a href="{{ site.url }}/Hush-Time">Website</a></li>
  <li><a href="https://github.com/TheCodedSelf/Hush-Time">GitHub</a></li>
</ul>

A macOS app written in Swift. Select the apps you'd like to close, choose a duration for your focused block of time, and hit Start.

Uses the following:
- AppleScript to close apps
- An NSOpenPanel to select a list of apps
- Simple reactive concepts to manage the current state

{% include gallery id="gallery_hushtime" %}

# Zen Breath

<ul id="menu">
  <li><a href="https://itunes.apple.com/us/app/zen-breath/id1071857375?ls=1&mt=8">App Store</a></li>
  <li><a href="https://github.com/TheCodedSelf/zen-habits-reader">GitHub</a></li>
</ul>

An iOS client for a blog named Zen Habits. Written in Objective-C and published to the iOS App Store.

Uses the following:
- Core Data to store posts
- Cocapods for third-party dependencies
- Google AdMob for advertising
- In App Purchases for monetisation
- Google Analytics to provide user insights

{% include gallery id="gallery_zenbreath" %}
