I’ve recently dipped my toes into Android development to see the differences in environment and tooling as compared to iOS development. While I haven’t done much as of yet, I hope to ship some Android apps in the future alongside some iOS ones. I figured it was time to understand the platform so that I can better relate to the woes of Android development. 

### Disclaimer
Note that this is entirely opinion based, and these are only initial opinions. Understanding of any language or SDK fleshes out over time, and I’m looking forward to see how my understanding and opinions change as I write more and more Android code.

## The Findings
I expected to see much more boilerplate and repetitive code as opposed to iOS, where we have the ability for frameworks to be more powerful and simple at the same time by tapping into the Objective-C runtime. While this was true to some extent, I’d like to talk more about some of the other differences I noticed:

### IDE
If you’re ever used a Jetbrains IDE or plugin, you’ll love Android Studio. It’s such a pleasure having the platform’s standard IDE being one by Jetbrains. I’ve been burned by Xcode too many times, namely due to bugs and lack of refactoring tools, and I for one am quite pleased with Android Studio.

### Activities vs ViewControllers
Where you’d use a UIViewController in iOS, you’re using Activities in Android. While passing data through Intents is not perfect in my eyes, I prefer it to using the `prepareForSegue` method on iOS to see if the segue identifier matches the one you expect and then configuring the destination view controller. However, if you don’t need to pass data around, I must say that handling all your segues in a Storyboard without writing any code is very convenient.

### Supporting different platforms and devices
I'm starting to see that we have it so easy on iOS when it comes to supporting other devices. There’s an almost unlimited number of hardware and screen size combinations for Android. Many iOS developers can name almost every possible device that their apps can run on out of the top of their heads, and that’s not something to take for granted. On Android I can see a need for a few if statements like the following:

```
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB)
```

On iOS, the amount of versioning checks is considerably less due to the practice of only supporting version N-1 (meaning today, I would support iOS 10 and iOS 9, but drop any support for iOS 8). This is a luxury that Android does not have.

### Localization
Android specifies string resources in XML. Specific locales are XML files with the ISO country code in the name. iOS has a similar structure but with a nasty `.strings` file instead of XML where standard practice is to specify strings like the following:
`“home.loggedin.title” = “Hello, World!”;`
or even worse:
`“Hello, World!” = “Hello, World!”;`

In use:

```
myLabel.text = NSLocalizedString(“Hello, World!", comment: “")
```

What I like about Android is that it’s strongly typed.

```
textView.setText(R.string.hello_world);
```

### Navigation
The first project that I built while trying to learn Android was an activity with a text field and a button. 

![Android First Activity]({{ site.url }}/assets/images/AndroidForiOSDevelopers/Android First Activity.png "Android First Activity")

When you tapped on the button, you’d be presented with another activity with a label that contained the text from the previous activity’s text field.

![Android Second Activity]({{ site.url }}/assets/images/AndroidForiOSDevelopers/Android Second Activity.png "Android Second Activity")

I was quite pleased with myself until I realized that I had forgotten to put in a navigation bar to get back to the first activity. Ugh, now I have to figure out how to embed everything in a navigation bar. How do you even make a navigation bar in Android? It took me a few seconds before I remembered that Android devices always have a back button. Lo and behold, a functional navigation hierarchy without implementing a UINavigationBar! Feel my power!

![]({{ site.url }}/assets/images/AndroidForiOSDevelopers/Feel My Power.jpg)

### Dynamic Views
Activities can contain Fragments, which is sort of like a ViewController containing multiple ViewControllers as children and receiving all lifecycle calls that the Activity receives. You can easily add Fragments to an Activity in XML or in code using the `FragmentManager` class.  No messing with Storyboards and Nibs and making sure that you correctly tie everything up!

{:.center}
![Dynamic Views on Android]({{ site.url }}/assets/images/AndroidForiOSDevelopers/homer_simpson_woohoo-300x190.jpeg "Dynamic Views on Android")

The Android Developer training documentation shows how you can get the behavior of a UISplitViewController that shows two views on a tablet, but uses a navigation hierarchy to navigate between the two views on smaller devices:

![Android Fragment UISplitViewController comparison](https://developer.android.com/images/training/basics/fragments-screen-mock.png)

Except with Fragments, you’re not limited to that layout.

### Storing Data
You may whine about Core Data now, but there’s nothing in Android stopping you from executing arbitrary SQL queries. You have the `SQLiteOpenHelper` class to guide you but other than that it’s pretty much up to you to manage your database. At least we now have [Realm](https://realm.io/products/realm-mobile-database/) on both platforms to help us overcome the database nightmare.

## Android Development Going Forward

Why would an iOS developer want to learn Android development? 

- They capture the majority of the market, especially in developing countries
- Users expect apps to be available on both iOS and Android
- It’s a natural way to expand your skill set: a new language and SDK, but still the same mobile paradigm

I’m excited to take on more Android development going forward.