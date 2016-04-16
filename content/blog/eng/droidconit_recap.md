---
authors: ["giacomo"]
comments: true
date: "2016-04-18"
draft: true
image: "/images/droidconit-recap/logo_droidcon_it.png"
menu: ""
share: true
categories: [English, Android, Conferences]
title: Droidcon It 2016

languageCode: "en-En"
type: "post"

---
![Droidcon It](/images/droidconit-recap/logo_droidcon_it.png)


The third edition of [Droidcon IT](http://it.droidcon.com/2016/) was, as expected, a great conference, full of interesting talks and people coming from all over the world. We saw a lot of GDE ([Google Developer Expert](https://developers.google.com/experts/all/technology/android)) and also some Developer Advocates from Google, although it was not organized directly from the company. Back in March the Android team surprisingly released the brand new **N Developer Preview** earlier than expected, so this year we were already able to talk about the new features in Android N and analyze them. Furthermore there has been talk of **Kotlin**, **RxJava** and a lot of other useful and interesting topics.

In this post I will recap and make some considerations about the most interesting (and funny!) talks.

# Day 1

### Keynote - *[Wojtek Kalicinski](https://twitter.com/wkalic)*
The day 1 keynote was all about the N Developer Preview and was given by Wojtek Kalicinski, a developer advocate directly from Google.
Wojtek went through alle the new improvements made to the platform, both from the user and the developer point of view. Here’s a short summary of the most relevant:

- **Multi-window support**: probably the most requested user feature. I’ve always thought that its implementation would have been relatively straightforward, given the already responsive UI of Android, and in fact the code changes needed to support this feature are minimal. Just make sure you don’t lock screen orientation using `android:screenOrientation` in your manifest.
- **Better notifications**: in addition to a visual change of the notification panel, apps will now be able to let the user interact with bundled notifications individually or directly reply from the notification; and if the app already supports Android Wear notifications, most of this will come for free, since the APIs are the same.
- **Doze improvements**: now the battery saving mode Doze works not only when the phone is stationary (in a less aggressive variant) but also when the phone is in the user pocket or hands. Wojtek said that most of the apps should continue to work without any change -or eventually [minimal](http://developer.android.com/intl/ko/training/monitoring-device-state/doze-standby.html#assessing_your_app)-, but he strongly encouraged to test apps against this new feature.
- **JAVA 8 features!**: finally Google will allow developers to use a limited set of features of the last (but already 2 years old) version of the language, including *lambdas* (🎉) and *default and static interface methods* (TODO: add DP2 apis) . All of this is possibile thanks to the Jack[^1] toolchain which is able to compile Java 8 syntax to Java 6 compatible bytecode supporting in this way old phones.

Other changes include improvements to [Project Svelte](http://developer.android.com/intl/ko/preview/api-overview.html#background_optimizations), a new [Data Saver](http://developer.android.com/intl/ko/preview/api-overview.html#data_saver) feature, the addition of the [Quick Settings Tile API](http://developer.android.com/intl/ko/preview/api-overview.html#tile_api) and [many more](http://developer.android.com/intl/ko/preview/api-overview.html).

The most interesting and encouraging aspect coming out from the keynote and the N Preview release itself is the fact that Google is -finally- trying to fix the famous issue with Android updates. 
![N Preview Updates](/images/droidconit-recap/n-preview-updates.png)
Releasing the new OS preview six months before its public release is not only really useful for third party developers to update apps to support new APIs, but hopefully will help phone manufactures to keep their custom Android implementations more aligned with the stock version release cycle. This will result in a better user experience and, more important for us, a better developer experience, making it closer to the iOS world where developers don’t have to support five years old OS versions.

### #PERFMETTERS for Android - *[Hasan Hosgel](https://twitter.com/alosdev)*

[Slides](https://speakerdeck.com/alosdev/perfmatters-for-android-droidcon-turin-2016)

The talk was all about performances in Android. The speaker analyzed why #**PERMETTERS** on Android: the mobile user is generally *impatient* and *intolerant* and also a couple of seconds during the start-up could make the difference in making the user interacting with the app. He went through a lot of interesting tips and trick to improve performance, such as avoiding multi-pass nested layout -a topic discussed in more detail by Huyen Tue Dao in her [talk](#lean-layouts)-, avoiding memory leaks or having a cache on disk for data. 

The performance are crucial to engage the user and I think the speaker made a great job summarizing useful tips to improve it. Moreover most of them doesn’t alter the readability or the maintainability of the code, as is often the case for performance driven code changes.

### Let it flow! Unidirectional data flow architecture in Android - [*Benjamin Augustin*](https://twitter.com/dorvaryn)

[Slides](https://speakerdeck.com/dorvaryn/let-it-flow)

The use of patterns to structure the presentation layer (UI) in Android are still unfrequent. During the last couple of year we have seen a lot of discussions about **MVC**/**MVP**/**MVVM** patterns within the Android developers community and some implementations of them. 
During the talk the speaker presented a new approach that takes some concepts from the web development world like [*Flux*](https://facebook.github.io/flux/) and [*Redux*](http://redux.js.org). The concept in a nutshell is that the UI, in addition to being as dumb as possible, should also be **stateless**. In other words the UI listens to state changes pushed by the business logic and update itself accordingly.
Benjamin concentrated less on the abstract concepts and so used Kotlin and RxJava to build an [app](https://github.com/Dorvaryn/unidirectionalDataFlow) the demonstrate it practically.
 
I personally like this approach since it has a lot of advantages. It makes the UI state completely **predictable** and **reproducible**: it allows to represent an app state without going through all the steps needed to reach it, for example to reproduce a bug or test a particular situations. The problem is that, as you can see from the demo app, it adds a lot of boilerplate code and it is also a lot more verbose and difficult to implement it without tools like RxJava or Kotlin, that may be a concern to you.

### Play everywhere: providing a consistent experience across very different devices - [*Alessandro Bizzarri*](http://it.droidcon.com/2016/speakers/alessandro-bizzarri/) & [*Dima Kunin*](http://it.droidcon.com/2016/speakers/dima-kunin/)

The two speakers showed a high-level tour of how at **Spotify** they  are able to provide a consistent experience across different devices -defined as the combination of the its physical form and its OS-. 
They created their own **design language** to supports almost every platform, that defines almost everything from color palette and buttons style to animation and sounds. 
They explained how they use a **C++ library** common to all platforms to reuse code; in particular on Android they use it through the NDK, a toolset to include C or C++ code in your app and call it from Java code.
It was a light talk but still it was interesting to see how such a big company like Spotify finds solutions to scale and maintain a consistent and engaging user experience.

### Loving lean layouts - [*Huyen Tue Dao*](https://twitter.com/queencodemonkey) <a name="lean-layouts"></a>

[Slides](https://speakerdeck.com/queencodemonkey/droidcon-italy-2016-loving-lean-layouts)

The speaker gave a really great talk about one of the best way to keep an app smooth and responsive: **optimize layouts** to be as flat as possible. Basically Android will call `measure()` and 	`layout()` methods -two times in the case of `RelativeLayout`- through all the hierarchy of views to be able to draw the layout; so, for example, having a useless root level will  
result in more measure/layout passes and so poor performances.

The talk was very easy to follow and really useful: most of the advices covered are often ignored because singularly they have less impact, but, if taken as good habits and all together, they could really do the difference in maintain the UI smooth and lag-free.

# Day 2

### Android reactive programming with RxJava - [*Ivan Morgillo*](https://twitter.com/hamen)
RxJava has been echoing all over the Android developers community during last couple of years and given its huge popularity I was surprised to see that there was only one talk about it. Ivan Morgillo, however, is a topic expert -he also wrote a [book](https://www.packtpub.com/application-development/rxjava-essentials) about it- and during the talk he went through some of the basics of reactive programming (`Observables`, `Observer`, `Subscription`…) and a lot of useful operators, which are one of the strength of RxJava and Reactive extensions (Rx*) in general.

I actually was hoping for a more theoretical talk with an analysis of the advantages and disadvantages of embracing the reactive pattern in our code. It was instead a lot practical and it covered a great number of common use cases where reactive programming could really help to make code mode readable and maintainable.

### You can do better with Kotlin - [*Svetlana Isakova*](https://github.com/svtk)

[Slides](https://speakerdeck.com/svtk/you-can-do-better-with-kotlin)

Kotlin has recently reached [**1.0 version**](https://blog.jetbrains.com/kotlin/2016/02/kotlin-1-0-released-pragmatic-language-for-jvm-and-android/) and it is gaining a lot of attention within the Java community and especially whiten the Android one, and a lot of developers are hoping that Google will make it the default programming language for its mobile OS.

Kotlin is a **modern**, **pragmatic** and **Android-friendly** programming language with robust supporting tools since it is developed from the people behind IntelliJ (and so Android Studio). It could be mixed with Java code -making the switch painless- since there is no Kotlin SDK but just JDK plus extensions, and it includes a lot of modern concept such as data classes, null-safety, extension functions and lambdas.

The speaker showed also ANKO, a DSL written in Kotlin, that uses *lambdas with receivers*, to create layouts and views and allows to write code like:
```
verticalLayout {
    val name = editText()
    button("Say Hello") {
        onClick { toast("Hello, ${name.text}!") }
    }
}
```

This talk was one of the most interesting for me, as it persuaded me to try and deepen my knowledge of Kotlin. Svetlana was very good at showing the main features of the JetBrains language, and at explaining how they convert in Java code, an aspect that I found very useful to mutate my mindset to the new syntax. 

### fun things (you: CanDo <in Kotlin> ) = … - [*Maciej Górski*](https://github.com/mg6maciej)

[Code](https://github.com/mg6maciej/fun-things-you-CanDo-in-Kotlin)

The second Droidcon talk about Kotlin was a very fun one, with the speaker acting a funny scene where a nephew try to explain to granpa why to use **Kotlin instead of Java** with a live coding session.
<iframe width="560" height="315" src="https://www.youtube.com/embed/RVu_0TIiH8Q" frameborder="0" allowfullscreen></iframe>

The second part of the talk was still in live programming (without slides) and showed some Kotlin feature in practice. It covered some functional programming, operator overloading, infix functions, laziness and more.

It was funny and informative to watch a live coding talk and for sure it made me more curious about Kotlin.


# Conclusions

[^1]: [Jack](https://source.android.com/source/jack.html) is the new Google compiler which has the goal to speed up the compilation process, compiling Java .class files directly to .dex files (Android bytecode) 