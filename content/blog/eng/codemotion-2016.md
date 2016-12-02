---
authors: ["engineering"]
comments: true
date: "2016-11-30"
draft: true
share: true
categories: [English, Conferences]
title: "Codemotion Milan 2016 in review"

languageCode: "en-EN"
type: "post"
toc: true
---

*«Learn or die»*. These are the words with which [**Codemotion Milan 2016**](http://milan2016.codemotionworld.com/) started on 25th November. Most of our development team attended this tech conference like the [previous year](http://engineering.facile.it/blog/ita/facile-it-devs-codemotion-milan-2015/). Moreover [Facile.it](http://www.facile.it) was present at the event as a sponsor with his stand looking for new talent.

During the event we also officially announced the second [**#FacileHack**](http://hackathon.facile.it/) planned for the next year.

This post would be a short recap of some talks in chronological order. The choice of the talks is based on the personal taste of the various members of the editorial staff of Facile.it Engineering blog.

![A part of Facile Dev Team](/images/codemotion-2016/codemotion_2016.jpg)

# Day 1

## The new features of PHP 7
 * Enrico Zimuel ([@ezimuel](https://twitter.com/ezimuel))
 * 11:30 – 12:10 #programming ([slides](http://zimuel.it/slides/codemotion2016/))

Enrico Zimuel, Software Engineer at [Zend Technologies](http://www.zend.com/), the company that is behind PHP, talks about some of the new features of PHP 7: the scalar type and return type declaretions, the spaceship and null coalescing operators, the anonymous classes, the consistent 64-bit support, etc. Futhermore he explains why PHP 7 is twice as fast compared to the previous version and it has 30% lower memory consumption.

## Kubernetes and lastminute.com: our course towards better scalability and processes
 * Michele Orsi ([@micheleorsi](https://twitter.com/micheleorsi))
 * 11:30 – 12:10 #devops ([slides](http://www.slideshare.net/micheleorsi/kubernetes-and-lastminutecom-our-course-towards-better-scalability-and-processes))

Lastminute.com recently started a migration from a monolithic app to a microservice approach, leveraging the power of kubernetes, the OSS lead by Google for container orchestration. This was quite the effort, and unveiled a great deal of advantages and challenges.

## Continuous budgeting
 * Francesco Fullone ([@fullo](https://twitter.com/fullo))
 * 14:10 - 15:50 #inspirational
 
This talk is about being agile in the entrepeneur side of our profession: how to plan and adapt to change when deciding how and how much to spend in our coding ventures; re-evaluating is key, and the only way to success is a delicate balance between maintenance and improvement.

## To ∞ (~65K) and beyond!
 * Sebastiano Gottardo ([@rotxed](https://twitter.com/rotxed))
 * 16:10 - 16:50 #mobile ([slides](https://speakerdeck.com/dextor/to-65k-and-beyond))

A lot of old -but still widely used- Android devices are natively limited to run apps with less than 65K method references (due to the Dalvik JVM). For many apps this could be a serious problem. During the talk Sebastiano, an Android engineer at Musixmatch, gave a very comprehensive and clear explanation of the problem and went through all the possible solutions showing some really useful tips to deal or better *avoid* it.

## Functional Reactive Programming with Kotlin on Android
 * Giorgio Natili ([@giorgionatili](https://twitter.com/giorgionatili))
 * 17:10 - 17:50 #mobile ([slides](https://drive.google.com/file/d/0BxCm4NRlzb3PWjNNaG1KS0Utckk/view))

The title of this talk was very interesting considering the fact that both the functional paradigm and the Kotlin language are used within the Facile.it Android app. But actually I have to say that I found a 40 minutes talk not suitable as a format to address these two large topics together (72 slides!). I would have preferred two separate talks, but it was nevertheless an interesting presentation full of useful tips and advices about Kotlin and FRP.

# Day 2

## Coding Culture
 * Sven Peters ([@svenpet](https://twitter.com/svenpet))
 * 10:30 - 11:10 #inspirational ([slides](http://www.slideshare.net/svenpeters/coding-culture))

In his keynote Sven Peters, Evangelist for [Atlassian](https://www.atlassian.com/), explains - in a very geek style - what is *company culture* particularly referring to his company. He says that when people work in a great coding culture they are motivated and they are mostly very productive, but also more responsibility for the features they are developing. Developer will have more freedom to change their roles and try out new stuff, this is the right way to make better products with happier developers. He concluded his speech with these words: *«Products may change, but never forget values, cultivate your culture!»*.

## MicroMonolith - Top anti-patterns of adopting distributed system
 * Michal Franc ([@francmichal](https://twitter.com/francmichal))
 * 12:30 – 13:10 #architectures
 
In this talk, Michal reports his (and his team) experience in the infamous transistion from a monolithic app to a microservice oriented architecture. Michal works at [@JustGiving](https://twitter.com/JustGiving), which mainly uses .NET, but he didn't focus on the specific technology, but rather on the generic problems and pitfalls that this kind of transition brings to the table. During his tale, he laid to us a lot of useful suggestions and common issues to look for.

## Develop applications in Big Data Era with Scala and Spark
 * Mario Cartia ([@mariocartia](https://twitter.com/mariocartia))
 * 12:30 – 13:10 #cloud #bigdata

Large amount of data are produced everyday and Big Data is a concept increasingly trendy. In his talk Mario Cartia presented Scala and Spark,two powerful tools suitable to Big Data Processing.  Scala is a General purpose programming language that combines the benefits of OOP and functional programming. Apache Spark is an open source framework and it's a fast engine for big data processing, based on Scala language.

## Gang of Four Patterns in a Functional Light
 * Mario Fusco ([@mariofusco](https://twitter.com/mariofusco))
 * 14:10 – 14:50 #programming ([code](https://github.com/mariofusco/from-gof-to-lambda))

In this live-coding talk, Mario Fusco uses Java (and new Java 1.8 features) to translate the wide-known GoF design patterns into functional programming, making the code a lot less verbose, reducing the noise of boilerplate over the really important code.

Also, kittens:

<blockquote class="twitter-tweet" data-cards="hidden" data-lang="it"><p lang="it" dir="ltr">&quot;In programmazione funzionale, quando lanci un&#39;eccezione muore un gattino, se ritorni null muore la mamma&quot; by <a href="https://twitter.com/mariofusco">@mariofusco</a> <a href="https://twitter.com/CodemotionIT">@CodemotionIT</a> <a href="https://t.co/40XErlWFbX">pic.twitter.com/40XErlWFbX</a></p>&mdash; Giulio Santoli (@gjuljo) <a href="https://twitter.com/gjuljo/status/802507849494654976">26 novembre 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>
