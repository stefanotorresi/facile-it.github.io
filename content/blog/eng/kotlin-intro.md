---
authors: ["giacomo"]
comments: true
date: "2016-04-08"
draft: true
image: "/images/cover.jpg"
menu: ""
share: true
categories: [English, Java, Kotlin, JVM, Android]
title: "A new hope for JVM: Kotlin"

languageCode: "en-En"
type: "post"


aliases: 
  - "/kotlin-intro"
twitterImage: 'images/kotlin-intro/kotlin_banner.png'
---

![Kotlin](/images/kotlin-intro/kotlin_banner.jpg)

## Premise

Java is an old programming language. Version 1.0 was released in 1996 by Sun Microsystems and despite over the past twenty years it has evolved and grown a lot it is still carrying on some bad design choices such as `null` (ask [Tony Hoare](https://en.wikipedia.org/wiki/Tony_Hoare?section=3#Apologies_and_retractions)), primitive types or lack of a proper function type. With the last version of the language  (Java 8) Java tried to address some of this problems introducing concepts such as [`Optional`](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html) or [lambda expression](http://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html). Even though these additions are clearly  a step forward for the language I still have the feeling that they are only “patches” applied to mitigate problems and not to solve them at their source. For example `Optional` could be used to reduce NPE but it is clearly not designed for [this purpose](https://twitter.com/mariofusco/status/780770300178956289)) and lambda expressions, implemented in Java 8 with SAM types, still force you to write an interface only to define a “function”.

## The Android world

All the above concerns about Java are even more problematic within the Android world, where, due to the notorious fragmentations (a huge amount of devices are stick with an outdated VM), you are forced to target lower Java version (6 and 7).
Google is addressing the problem with its new compiler [Jack](https://source.android.com/source/jack.html) that enables some of the features of Java 8 maintaining backward compatibility with older OS versions. But still it lets us deal with the verbosity of the language and doesn’t really solve the problem.

## A new hope

![Kotlin Hello World!](/images/kotlin-intro/kotlin_helloworld.png)

Back in 2011 the JetBrains team (the guys behind IntelliJ and so Android Studio) unveiled [Kotlin](https://kotlinlang.org/), a new programming language that targets the JVM (and could also be compiled to JavaScript). 
Kotlin is a statically-typed language that combines Object Oriented and functional features enforcing no particular philosophy of programming, and introduces a whole new set of concepts and tools that help to make the code more safe, clean and readable. 
Thanks to its nature it works everywhere in the Java/Android world and it is also interoperable with Java, meaning it will not force you to rewrite the entire codebase to taste it: you can add it to your project a little at a time ([maybe starting with tests](https://medium.com/@sergii/using-kotlin-for-tests-in-android-6d4a0c818776#.lyvd3h43x) 😉).
Here I’ll will rapidly cover two of the main new features of Kotlin:


### Null-safety ([Doc](http://kotlinlang.org/docs/reference/null-safety.html))
In Kotlin a variable cannot be `null`. If you want or need a variable to be *nullable* you have to add `?` to the variable type:
```
val x: Int = null \\ compile error
val y: Int? = null \\ ok
```
Thanks to this information the compiler sees `Int` and `Int?` as two completely different types and can therefore enforce the “null-safety” of your variables.
#### Safe calls
The `?.` allows you to safe call methods on nullable variables without throwing NPE but simply returning null at the end of the  call chain:
```
val x: Int? = null
x?.toString()?.substring(2) \\ no NPE, returns null
```
#### Elvis Operator
The `?:` operator allows you to provide a “default” value for when the variable is `null`:
```
val x: String? = null
(x ?: "hello").substring(2) \\ no NPE, retunrns "llo"
```


### 

# Conclusions

Java has to maintain backward compatibility with previous versions and still has to support the huge amount of developers and codebases present all around the world so it’s natural that every new feature and design change is considered, weighted and reasoned carefully, inevitably slowing down its development. But it does not have to mean that us, as Android developers, “tied” to the JVM should not try more modern and advanced languages such as Kotlin. At bottom, a part (one of the best!) of our job is to try and experiment new technologies, paths and possibilities to learn new concepts and techniques to improve our ability to address problems in the best possible way (and clearly t have some fun 😄). 
Moreover I think that for a software engineer is fundamental to be exposed to more than a single language: learning new patterns, exploring other programming paradigms or simply using and understanding a never-seen syntax has an immeasurable value for our growth and it turns out to be an unexpected usefulness even when developing with ”our” language, so why not do it with a language that allows us to continue to work on running projects targeting our beloved JVM?

#supported by big company