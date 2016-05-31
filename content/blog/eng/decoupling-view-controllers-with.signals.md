---
authors: ["elviro"]
comments: true
date: "2016-03-15"
draft: false
share: true
categories: [English, Swift, Functional programming, Signal]
title: "Decoupling view controllers with Signals"

languageCode: "en-US"
type: "post"
aliases:
  - "/decoupling-view-controllers-with-signals"
images: ['/images/logo.png']

---

[Last time](http://engineering.facile.it/code-reuse-a-primer/) we looked at the **Signal** class, that is, a simple, reusable way of encapsulating the [observer pattern](https://en.wikipedia.org/wiki/Observer_pattern). There are many use cases for a signal, and I'm going to show one possible application, spawned from a real-world problem. View controllers' composition and decoupling is **hard**: we often need an input from a view controller, that has to send its input back to its creator, while handling the back navigation somehow. We often find ourselves in a situation in which several different responsibilities are all expressed in a single view controller, with the effect of creating a gigantic class, full of entangled imperative statements, hard-to-understand sequencing and general complexity. We'll use the `Signal` class to assign the various responsibilities to different classes, and write cleaner, more declarative code. The core of this architectural pattern lies in inverting the way in which objects communicate, view controller or other: instead of asking objects to do things, we're going to **observe** what objects are doing, and **react** accordingly. *Observe* and *React* are the cornerstones of the programming paradigm known as [functional reactive programming(FRP)](https://en.wikipedia.org/wiki/Functional_reactive_programming); the present article is not going to talk about FRP as a whole, nor to present shared FRP techniques; the point is to discuss an architectural pattern for decoupling view controllers from responsibilities not strictly related to user interaction, by leveraging some basic FRP tools. 

The example project for this article is available in [GitHub](https://github.com/broomburgo/SignalViewControllers/): I'm going to paste some code examples, but it's recommended to check and test the entire project while reading the article. What follows is the full implementation of `Signal`, and its public interface for sending new values, called `Emitter`:

```Swift
import Foundation

public enum Persistence {
  case Stop
  case Continue
}

public final class Signal<Subtype> {
  typealias Observation = Subtype -> Persistence

  private var observations: [Observation] = []

  public init() {}

  public func onReception (observeFunction: Subtype -> Persistence) -> Signal {
    observations.append(observeFunction)
    return self
  }

  public func map<OtherSubtype>(transform: Subtype -> OtherSubtype) -> Signal<OtherSubtype> {
    let mappedSignal = Signal<OtherSubtype>()
    onReception {
      mappedSignal.send(transform($0))
      return .Continue
    }
    return mappedSignal
  }

  public func flatMap<OtherSubtype>(transform: Subtype -> Signal<OtherSubtype>) -> Signal<OtherSubtype> {
    let mappedSignal = Signal<OtherSubtype>()
    onReception {
      transform($0).onReception {
        mappedSignal.send($0)
        return .Continue
      }
      return .Continue
    }
    return mappedSignal
  }

  public func filter(predicate: Subtype -> Bool) -> Signal {
    let filteredSignal = Signal<Subtype>()
    onReception {
      if predicate($0) {
        filteredSignal.send($0)
      }
      return .Continue
    }
    return filteredSignal
  }

  public func unionWith (otherSignal: Signal<Subtype>) -> Signal {
    let unifiedSignal = Signal<Subtype>()
    let observeFunction = { (value: Subtype) -> Persistence in
      unifiedSignal.send(value)
      return .Continue
    }
    onReception(observeFunction)
    otherSignal.onReception(observeFunction)
    return unifiedSignal
  }
}

public func + <Subtype> (left: Signal<Subtype>, right: Signal<Subtype>) -> Signal<Subtype> {
  return left.unionWith(right)
}

extension Signal {
  private func send (value: Subtype) {
    var newObservations: [Observation] = []
    while observations.count > 0 {
      let observe = observations.removeFirst()
      let persistence = observe(value)
      switch persistence {
      case .Continue:
        newObservations.append(observe)
      case .Stop: break
      }
    }
    observations = newObservations
  }
}

public final class Emitter<Subtype> {
  public let signal = Signal<Subtype>()

  public func emit(value: Subtype) {
    signal.send(value)
  }
}
```

## The megacontroller

Suppose we need to create a simple app to leave a feedback for a movie we just watched; the feedback will be divided in two categories:

- *average*: the movie was *good*, *bad* or *so and so*;
- *polarized*: the movie was *very bad* or *really good*;

The app is going to present a page in which the user can select an *average* or a *polarized* feedback, and can tap a button that will present the user the available choices, i.e., respectively *good*, *son and so*, *bad*, or *really* and *very bad*. The app is also going to thank the user if they've just left a positive feedback, but only if the previously left feedback was of lower value. The example is a little contrived but instructive, as we're going to see. An **easy, familiar approach** to the problem would be to start with the design of the two view controllers involved:

- the `MainPage` will let the user select if the feedback is average or polarized, and will present a button to show the page where the actual feedback value will be selected; the `MainPage` will also show the currently selected feedback (or none) in a label;
- the `SelectionPage` is going to present several buttons with the various feedback values (different based on the feedback category), and at the tap of a button the page should actually pop from the navigation stack, going back to the now-updated `MainPage`.

Armed with our **imperative mind** we would probably start by adding a sequence of instructions to the `MainPage`, that we'll probably perceive as the "main" controller of our app. But soon, we would probably notice that a lot of **clearly separated** responsibilities are being collected in the `MainPage` class:

- the `SelectionPage` has to be constructed and initialized, and by doing this in `MainPage` we are imposing a strict **dependency** between the two classes;
- after its initialization, the `SelectionPage` must be presented to the user: we would probably embed `MainPage` in a `UINavigationController`, then push the `SelectionPage` from the `MainPage`, resulting in a strict coupling of the two classes with a particular navigation and presentation strategy (i.e., using a `UINavigationController`) that is going to cause headaches if the use cases change (for example, in developing the **iPad version** of the app);
- there's a feedback value **stored somewhere**, because we need to know if the user selected `average` or `polarized`, and actually which was the selected feedback, because its value will be shown on the main page; the simplest solution seems to be just storing the selected feedback value in the `MainPage` class;
- the act of "collecting the feedback" is a responsibility in itself, because the collected value is probably going to be used somewhere (for example, posting it to a web service);

We basically identified 4 different responsibilities, each of which is probably going to need its own class, but instead of thinking about the methods that need to be called on each one of them, we're going to try and think with signals.

## The model

For example, if the model changes we need to update the text shown on the `MainPage`: a possible way to manage this could be to create a `ModelController` class, which holds the model, and *emits a signal* each time the model changes. Then, the `MainPage` could *react* to this signal, and change the UI accordingly. A nice to way to do this is injecting the `ModelController` in the `MainPage` constructor, so that `MainPage` can establish the appropriate bindings:

```Swift
/// MainPage initializer
init(feedbackModelController: ModelController<FeedbackModel>) {
  super.init(nibName: nil, bundle: nil)
  feedbackModelController.updateSignal.onReception § eachTime § updateViewsWithFeedbackModel
  viewReadyEmitter.signal.onReception § eachTime § feedbackModelController.notify
}
```

The `§` operator and the `eachTime` function are just helpers to make the **functional composition** easier: as it often happens with functional programming, or declarative programming in general, we can infer the meaning of an expression just by reading it; in fact, `onReception § eachTime § updateViewsWithFeedbackModel` means that when the signal is received, the `MainPage` will update the view every time according to the new `FeedbackModel`: `eachTime` means that every time the signal triggers, so will the update; this is in contrast with the `once` function, that makes the object listen only to the first signal trigger. This is  related to the `Persistence` of a `Signal` observation, that is, if the object should continue listen to a signal or not: `Persistence` is a single `enum` with two values, `Continue` and `Stop`.

An important characteristic of the view controllers is the fact that the views are not yet initialized in the constructor, so we often need to memorize some data and use it in the `viewDidLoad` method, that is called by the framework when all the views are loaded, and can consequently be manipulated; but we used a signal to express the update logic directly in the constructor, where the `ModelController` is available: the line `viewReadyEmitter.signal.onReception § eachTime § feedbackModelController.notify` means that when the `viewReadyEmitter`, i.e., the emitter that will send a signal when the view is ready, is triggering, the model controller has to *notify* its current value to all the observers; this way we don't need to manually update the views in the `viewDidLoad` method: everything is connected, and the signals will **propagate** according to the declarative bindings.

## Handling page creation and presentation

Transitioning between pages is one the key points in iOS programming: page presentation has to be **predictable and smooth**, to avoid a confusing user experience. One of the main premises of the `UIViewController` class was and still is the creation of a modular hierarchy of views, that's independent from the way it's presented to the user: that way we can easily reuse **the same** view controller, for example, in full screen on the iPhone, or as a child view controller on a more complex hierarchy on iPad. The problem is that, if we write down the navigation and presentation logic *inside* the view controller class, we are going to establish tight dependencies between the view controller and its presentation, thus going against the premise.

A possible solution would be to use a `NavigationHandler`, that will handle the transition between pages, along with the initial presentation during the application startup. The strategy here is to use a class that knows **when and how** to present pages, oblivious of what pages actually do. For example, on receiving the appropriate signal from `MainPage`, the `NavigationHandler` will present the `SelectionPage`. Where the `SelectionPage` comes from? We should certainly avoid to burden the `NavigationHandler` with pages' initialization responsibilities,  in fact we identified the class creation as a responsibility in itself. A frequent pattern used to handle the initialization of objects is the [Factory Pattern](http://www.oodesign.com/factory-pattern.html): a `PageFactory` will construct our pages with the options passed to the factory constructor. Thus, `NavigationHandler` will ask the `PageFactory` for pages, through the methods `makeMainPage` and `makeSelectionPage`.

But we incur in a problem: `makeSelectionPage` will just return a `SelectionPage` object, but we don't know if it's *always* the same instance, or a new instance each time. In fact, `PageFactory` doesn't *promise* always the same page. This is important because the `NavigationHandler` will only take care of page presentation, but the very same page instance has to be considered, for example, for collecting the feedback. A possible strategy would be to *cache* the `SelectionPage`, but one of the main points of functional programming is **avoiding state**: we don't want to burden ourselves with the responsibility of managing mutable state, which is one of the [main causes of complexity](http://shaffner.us/cs/papers/tarpit.pdf) in software development. Instead, we will once again leverage signals to handle the situation. `PageFactory` exposes two signals that are triggered at page creation; `NavigationHandler`  uses those signal to bind its actions to the initialized pages:

```Swift
pageFactory.signalMakeMainPage
  .flatMap { $0.signalLeaveFeedback }
  .onReception § eachTime § inAnyCase § presentSelectionPage
  
pageFactory.signalMakeSelectionPage
  .flatMap { $0.signalSelection }
  .onReception § eachTime § inAnyCase § popTopPage
```

In the code just shown, the signals that are triggered when the pages are initialized are *flatMapped* to the respective interaction signals, that is, the observed signal is a signal that will trigger when the second signal is triggered, but the *second* signal will be available only when the *first* signal is triggered: the `flatMap` method will let us reference a signal that is not yet available. The `inAnyCase` function is used because `NavigationHandler` is not interested in the *content* of the signals, but just the fact that they are triggered; the content is going to be handled by another class: `FeedbackCollector`.

## Composing signals

`FeedbackCollector` has the responsibility of *collecting the whole feedback*: this seems tricky, because the creation of a new feedback is not a synchronous procedure, and it's the result of **many different interactions** from the user in different contexts. As we saw, a signal is an abstraction over asynchronous programming: we declare the signal bindings for actions that will trigger at a certain point in time. Thanks to signals we can express the logic for collecting feedback in a single function call, executed during the app startup phase, even if:

- at the app startup the pages are not yet initialized;
- to actually collect the feedback various interactions in multiple pages will be needed;

Usually these consideration would lead to **stateful** computations, where state would be expressed in various points, and mutated. But the class `FeedbackCollector` can generate a signal triggered each time a new feedback is collected with a single expression:

```Swift
func collectFeedbackModelChange() -> Signal<FeedbackModelChange> {
  return pageFactory.signalMakeMainPage
    .flatMap { $0.signalPolarizedChanged }
    .map(FeedbackModel.transformWithPolarized)
    + pageFactory.signalMakeSelectionPage
      .flatMap { $0.signalSelection }
      .map(FeedbackModel.transformWithFeedback)
}
```

The `+` operator will literally *add* 2 signals together, producing a signal that is triggered each time either of the signals is triggered. The app uses the `Signal<FeedbackModelChange>` produced by the `collectFeedbackModelChange()` to update the model in the `ModelController`.

Another example of asynchronous reaction to signals is shown in the `AppDelegate`'s method `handleGoodFeedbacksWithAlert()`; here's the logic: if the user leaves a positive feedback, that is better than the previously left feedback, the app will show an alert, thanking the user. The entire logic for this is handled in the following expression:

```Swift
feedbackModelController.deltaSignal
  .filter { $0.feedback.rawValue < $1.feedback.rawValue}
  .filter { $1.feedback == .Good || $1.feedback == .ReallyGood }
  .onReception § eachTime § inAnyCase § showThankYouAlert
```

`ModelController`'s `deltaSignal` is triggered each time the model changes, and contains both the old and the new value: in the shown expression a signal is created by *filtering* the `deltaSignal`, because we need a signal that is triggered only when the new feedback is different from the previous and positive.

## Conclusion

A clean and decoupled architecture is hard to get right, and can be obtained in different ways. Using signals will help us achieve the following:

- we can work on **a single piece at a time**, thinking about what *values* are exchanged between classes, instead of interfaces and abstractions;
- we'll be able to focus on what a view controller, or any class, really has to *produce*, that is, what value is going to yield in the economy of the app;

Signals are not application-wide events: we don't need a router or a dispatcher (like 	`NSNotificationCenter`) to properly handle them, because each object will declare its observation of specific signals. Also, because signals are easily transformed and composed, we can declaratively express our bindings without worrying about *when* and *how* things are going to happen, but just *what* we need. Of course our architecture has to take into account the fact that getting a *precise sequence of events* is basically impossible when working with signals: actually, *sequencing* is not even considered in declarative programming, because, along with *state*, is a great cause of complexity. In the end, a declarative architecture, like everything, has its tradeoffs, but I still think it's worth trying.
