---
authors: ["jean"]
comments: true
date: "2016-05-11"
draft: true
share: true
categories: [English, TDD, PHP]
title: "How TDD & PHP7 helped me being a dad and a developer at the same time"

languageCode: "en-US"
type: "post"
---
I worked as a professional developer for more than five year since now, and nearly half a year ago I became a dad.
It has been great, and you also get some unexpected perks! For example, my colleagues got me this gift for my son:

(elePHPant photo) 

So, we can say that his future is pretty clear... But don't say this to my wife! 

During the pregnancy, many of my friends and fellow parents warned me, half jokingly, about one thing: *"sleep now, you'll be deadly tired after!"*. I can't deny that, having a child has a toll on your sleep schedule... Even if, as in my case, having a 9 to 6 office job, my wonderful wife does all the parenting heavy lifting (and I consider myself pretty lucky for having her!). 

# The enemies of programming
As many of you will agree with me, **sleep deprivation** is the enemy of programming. Maybe we fear only one thing more than that: **being interrupted**.

(comic img) 

While writing code we have to think really hard, we use complex abstractions, we go through long business workflows and so on... fatigue and interruptions are the main enemies of those in this line of work.

# My experience 
On my daily job, I do all this mental juggling on a pretty big project, which is based on PHP 5.5, Symfony 2.8, Doctrine and so on; in this project we luckily use a good deal of **good practices**, and **automated sofware testing** is one of those. I actually switched to this job to learn about doing automatic testing, continuous integration and other best practices.

A few months after my son was born I also had the opportunity to start **a new, fresh project**. To be completly honest, it was not actually fresh, but it was a **complete rewrite** of an internal service that is used to manage invoices for multiple business units inside our company. I knew pretty well the old system that had to be replaced, so I was put in charge of redoing it from scratch.

One of the issue with the old system was **maintainability**: we had no tests, we hadn't a proper dev environment, and the design was not great; also, the bureaucracy and invoicing are the core domain of the system, so it was not so easy and linear in its processes. This created the perfect environment to see the [Broken windows theory](https://en.wikipedia.org/wiki/Broken_windows_theory) in action, and the code base got worse over time, one patch, copy paste or fast fix at a time. 

Obviously, as any tech passionate would do, I took the opportunity to use a lot of new shiny tools! I picked **PHP 7**, which was just released, and started the project with something familiar to me but still pretty new and cool, **Symfony 3.0**.

# What I found useful
I rambled and thought about this project a lot in the past months with my colleagues, because the old system was costing us a lot of overhead in usage, and we had a pretty clear idea of what its problems were, so I hadn't to study a lot before start writing the first classes.

So, I had to spend *some* time thinking about an object oriented design for my project, but I was rapidly able to dive right into writing code, and be really confident about it. In the end, a lot of this confidence came from a few choices that I pursued during the development of this project.

## TDD and high coverage
I already knew the advantages of doing automated testing and Test Driven Development, but in the previous project that practice was introduced after some time, so not all the codebase was covered, and we couldn't (or wouldn't?) do TDD 100% of the times.

Instead, in this case **I wanted to write nearly everything with TDD**, and have a **very high threeshold for the minimum coverage** that I would mantain. Right now I'm proud to have a ~92% test coverage. This wasn't a mere "let's hit 100%!" mindless objective ([since it's pointless]({{< ref "software-testing-coverage-vs-efficacia.md" >}})), but instead it fueled **a positive feedback cycle**: the more I wrote new classes using TDD, the more the coverage rose and stayed high; at the same time, I found myself inspecting the coverage report to find missing spots, and that lead multiple times highlighting some edge-case that I didn't test, and I really needed to.

Of course, I still leaved some part uncovered, since it's pointless to test them (e.g. Doctrine entities), and I covered some parts multiple times, since they were **critical paths** inside my application.

## Unit tests to the rescue!
Last but not least, the main critical advantage that TDD gave me was **focus** even on strained days: I wrote the classes starting from unit tests, without having to bear in mind all the project in its complex design.
 
I then wrote some functional test to assure that the **collaboration between my unit-tested objects** was fine, and this later step was also useful to delay the definition of the classes as services inside the Symfony's DI container. I was also **able to change my mind** on some details of the design a few times without suffering mental confusion or having to rewrite too much code.

## PHP 7: scalar and return types declarations
Two of the main reasons behind the choice of PHP 7 as the language version for this project were the [two main new features](http://php.net/manual/en/migration70.new-features.php) introduced in that version: **scalar types** and **return type declarations**. 

(RT, RT everywhere meme)

Before Facile.it, I worked as C++ developer, and oh boy! did I really missed scalars and return types! 

*"I came onboard of the PHP community right in time"*, I thought... So I exploited this situation to start using all this new features. I started to enjoy having again the possibility of typehint string and integers, and I discovered how return types declaration enforces really well the cohesion of your objects, making it **rightly painful returning different types of data from the same method**, or mixing a type with null.

Return types also demonstrated to be a **double edged sword** in some cases, especially on Doctrine entities: they are really useful to enforce consistency in your values, since they trigger a `\TypeError` each time you call a getter method on a wrongly empty property, but **you can't obviously use them on nullable fiels**, since it will break your application at any time.

On the other end, having return types declared on your business-logic classes it's pretty useful, and more over when used in conjunction with TDD: every time you define a mock you are forced to declare expectations and predictions with the right type values, hence it **indirectly helps mantaining the collaboration contract between objects**, without too much hassle. If I changed the signature of a method that was mocked somewhere, the mock would break the test, **highlighting the issue and making the test (and the high coverage) even more valuable**.

# Conclusions
I hope that those little life/programming lessons that I learned in these months will be valuable to other people like me. Stressful events are unavoidable, and testing and other best practices are there also to protect our work and its quality even from the inevitable days when we can't be at the top of our game.
