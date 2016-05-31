---
authors: ["elviro"]
comments: true
date: "2015-09-15"
draft: false
share: true
categories: [English, Software complexity, Functional programming, OOP, Structured programming]
title: "No Country For If Else"
ita: "no-country-for-if-else"

languageCode: "en-US"
type: "post"
aliases:
  - "/no-country-for-if-else-english-version"
---

There is an **unwanted guest** with us as we write code and build software projects: it's the code that's **already written**, and we must take into account its complexity as the code base increases in size. High complexity of the existing code can make the following activities particularly difficult:

- understanding the meaning of old code, written by others or ourselves;
- tracing the causes of bugs, i.e. errors, in code;
- making changes to a certain procedure;
- adding features to existing structures;

Even if we approach the development of new software with [agile methodologies](https://en.wikipedia.org/wiki/Agile_software_development), we always have to deal with the existing code, and to do that we must at least be able to **understand it without overexertion**. So when I talk about *complexity* I am referring in particular to the difficulty with which a programmer can *reason about* the existing code: the preface of the well-known academic textbook [Structure and Interpretation of Computer Programs](http://deptinfo.unice.fr/~roy/sicp.pdf) contains the following sentence:

> programs must be written for people to read, and only incidentally for machines to execute

I couldn't agree more: the time spent by a programmer in writing new code is **only a fraction** of his/her total working time; much of it is spent reading the existing code, to correct, extend, modify or simply understand it. Generating more understandable code, about which it's easier to reason, we can more easily evaluate the **correctness** of what we wrote so far or, if the obtained behavior is not the one expected, find the error. Of course this shouldn't undo the techniques that we routinely use to test the correctness of software *a posteriori*: indeed, no technical or scientific discipline is exempt from empirical tests, and in fact the empirical approach to software engineering is accepted in the academic community, as evidenced by the [existence](http://static.springer.com/sgw/documents/1525357/application/pdf/10664_JournalMetrics_Flyer.pdf) of the [Empirical Software Engineering](http://link.springer.com/journal/10664) journal. My personal position is in the middle: software testing is important, but it's also important to think in advance about its correctness, to avoid causes of **accidental complexity**.

## Index
- [The importance of abstraction](#the-importance-of-abstraction)
- [Measuring complexity](#measuring-complexity)
- [Example: structured solution](#example-structured-solution)
- [Example: OO solution](#example-oo-solution)
- [Example: functional solution](#example-functional-solution)
- [An empirical validation](#an-empirical-validation)
- [Conclusions](#conclusions)

<a name="the-importance-of-abstraction"></a>
## The importance of abstraction

As we've seen in a [previous article](http://engineering.facile.it/programmazione-funzionale-perche-preoccuparsi/), rather than the inherent complexity related to the logic of our software, we must worry about the [accidental complexity](http://shaffner.us/cs/papers/tarpit.pdf) we introduce in the code by adopting **non-optimal, pointlessly complicated solutions**: the excessive use of *if-else-for-switch* control structures tends to make the code hard to understand, maintain and test; see for example the well-known [Arrow Anti-Pattern](http://c2.com/cgi/wiki?ArrowAntiPattern). It should be noted that the introduction of [structured programming](https://en.wikipedia.org/wiki/Structured_programming), i.e. programming based on the aforementioned control structures, was a **major step forward** in software development between the '60s and the '70s: it allowed to program at a higher level of abstraction than the previous style, removing the need to manually manage code execution order with the [infamous](https://www.cs.utexas.edu/users/EWD/ewd02xx/EWD215.PDF) `goto` statement.

Software development paradigms change when programmers find themselves managing larger and more complex projects: to avoid being entangled by the excessive complexity of our own code, or code written by others that we still maintain, we have to work at **higher levels of abstraction**. How much high? A good answer could be the following:

> at a high level of abstraction we can tell the system to do what we want, without specifying how to do it

At an appropriate level of abstraction we can focus on the *what* and let the system decide the *how*. Obviously there's no "maximum" level of abstraction, and as programmers we always find ourselves working **halfway** between the moving electrical charges in a microprocessor, and the needs of our company's business: however, working at a low abstraction level will greatly increase the risk of introducing accidental complexity, and we must be able to identify those cases. For example, when we use a control structure such as *if-else* within a relatively large procedure, we are *manually* making a decision based on the current state of our system: it would be much better to develop a software component, for example an object, which is able to **make that kind of decision for us**; our role, at that point, will be to *declare* the required functionality. Mixing the *what* with the *how* will make it particularly hard to understand what a block of code is going to do, because it can lead to the obfuscation of a procedure's **intent** - connected to the business logic - with the **implementation details**, which are separate concepts, often completely independent (the same intention can be expressed with different lower level implementations).

Here's a simple example. Suppose we have the following function:

```swift
func <A> optionalValue1 (value: A, cond1: Bool, cond2: Bool) -> A? {
	var x: A?
	if cond1 == true {
		if cond2 == true {
			x = nil
		}
		else {
			x = value
		}
	}
	else {
		if cond2 == true {
			x = value
		}
		else {
			x = nil
		}
	}
	return x
}
```

Although the expressed intent is rather simple, what's really going on is not immediately obvious; let's analyze the function step-by-step:

- a variable `x` of type `A?` is declared;
- the first condition is checked:
	- if the first condition is true, then the second condition is checked:
		- if the second condition is true, then the variable is set to `nil`;
		- if the second condition is false, then the variable is set to `value`;
	- if the first condition is false, the second condition is checked:
		- if the second condition is true, then the variable is set to `value`;
		- if the second condition is false, then the variable is set to `nil`;
- the variable `x` is returned;

To think about it, the intent is the following: return `value` if the conditions are different from each other, otherwise return `nil`:

```swift
func <A> optionalValue2 (value: A, cond1: Bool, cond2: Bool) -> A? {
	return cond1 != cond2 ? value : nil
}
```

We have greatly simplified the function, making it easier to understand **just by looking at it** once. But in a more complex case, with many different conditions and possible code routes, it may not suffice to just try and simplify the conditions.

<a name="measuring-complexity"></a>
## Measuring complexity

A unit of measure often taken into account in assessing the complexity of a method or function is the [cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity) (CC): it's defined as the degree of complexity generated by the use of multiple control structures in a procedure. You can calculate the number of CC for a structured procedure by representing it with a [directed graph](https://en.wikipedia.org/wiki/Directed_graph) and counting the number of nodes and arcs. For example, we can represent the `optionalValue1` function with the following graph:

```
[(value,cond1,cond2)]
|
|-> [var x] -> [cond1 == true ?]
               |
               |-y-> [cond2 == true ?]
               |     |
               |     |-y-> [x = nil] --------|
               |     |-n-> [x = value] ------|
               |                             |
               |-n-> [cond2 == true ?]       |
                     |                       |
                     |-y-> [x = value] ------|
                     |-n-> [x = nil] --------|
                                             |-> [return x]
```

For a single connected graph, the number of CC is calculated according to the following formula:

```
CC = [number of arcs] - [number of nodes] + 2
```

For the `defaultValue1` you can see that CC = 4. Apparently the simplified version` defaulValue2` has a lower cyclomatic complexity, but in reality, as indicated in T.J.McCabe's [original article](http://www.literateprogramming.com/mccabe.pdf), CC can only be calculated for a *completely structured* procedure, that is a procedure with a single entry point and a single exit point; CC should also take into account **all the possible cases** for a condition, and `if cond1 != cond2` has precisely four possible cases. A dissertation of possible evolutions of CC, considering multiple entry/exit points for a module, is a available [here](http://www.acis.pamplin.vt.edu/faculty/tegarden/wrk-pap/SQJ.PDF). Thus, we won't keep account of CC in the rest of the article for the following reasons:

- the complexity that interests us is the one related to the "understandability" of a function, and unstructured patterns (such as `guard clause`, early `return` or conditional expressions like the one generated by the ternary operator `?:`) can often make a function or method easier to understand;
- the goal is to write a program by composing many small functions whose complexity is the **minimum possible**, e.g. no control structures, so it will have little value to measure the number of CC for each function;

However, it seemed important to mention cyclomatic complexity in this article, but rather than *measuring* accidental complexity, we are interested in **removing it completely** by replacing conditional statements, often used of structured programming, with something else. Notice that a conditional *expression* is a **simpler** concept than a conditional *statement*: in the former, the entire expression simply returns something based on one or more conditions; in the latter, based on some state, the code could contain various statements, i.e., commands that could be executed or not.

Say you have a function like this:

```swift
func getCorrectValueConsideringConditions <A> (cond1: Condition<A>, cond2: Condition<A>, cond3: Condition<A> ...) -> A? {
	/// do stuff
}
```

In this case we have a large number of conditions attached to a value of type `A`, and we might need to evaluate each of these conditions with several *if-else* and/or nested *switch* statements. One way to simplify this problem could consist in *abstracting* the concept of *condition*, turning it into a *static relationship* between a few parameters and a value of type `A`: we should therefore create a system that is able to *solve* any number of relations, possibly in **any order**, and according to them deduce the resulting value.

Let's see a practical example.

<a name="example-structured-solution"></a>
## Example: structured solution

A person goes to the employment office to find a job: according to certain attributes (job preferences, young/old, number of children younger than 18) the person could be sent to a certain desk to choose a job, or outside in case no job is available for a person with those attributes. Here is the complete procedure (the realism is not important, it's just an example):

A person **p1** comes to the desk **d1** to seek employment; **p1** has a list of job preferences, but could also accept other jobs outside the list; **p1** also has a list of jobs that would certainly never accept;

- if **p1** is young you get from **p1** a list **l1** of preferences;
	- if **l1** is too small, you ask **p1** to enlarge the list with secondary preferences;
	- you match the preferences with the available jobs and get list **l2** of jobs to choose;
	- if **l2** is empty, you ask **p1** if they want the full list of the positions;
	- if **p1** chooses a job, they're sent to desk **d2** to continue the procedure;
- if **p1** is elderly, you ask **p1** to choose from the list **l3** of suitable jobs for older people;
	- if **p1** chooses a job, they're sent to desk **d3** to continue the procedure;
- if **p1** has children, in any case the list of available jobs will be affected by the number of children:
	- if **p1** has one child younger than 18, they will choose a job from the list **l4**;
	- if **p1** has two children younger than 18, they will choose a job from the list **l5**;
	- in the two previous cases, if **p1** chooses a job, **p1** is sent to desk **d4** to continue with the procedure;
	- if **p1** has three or more children younger than 18, **p1** is sent to desk **d5** to continue the procedure;

A "structured" approach to the problem could consist in writing a function in which the entire procedure is expressed with a series of *if-else* statements, or *switch* in the case of the number of children.

The project related to this article is available on [github](https://github.com/broomburgo/NoCountryForIfElse): I strongly recommend to check the code as the article goes on; `.swift` files are just text files, and can be opened with any text editor. In the file `common.swift` you'll find common structures and functions to all the proposed solutions. In particular, we will represent a "person" with `struct Person`, basically a immutable *value object* that collects the various attributes considered in the problem:

```swift
struct Person {
    
    let name: String
    let isYoung: Bool
    let childrenCount: Int
    let likedJobsMain: [String]
    let likedJobsSecondary: [String]
    let dislikedJobs: [String]
}
```

The idea is to write a function called `placeNameForPerson`, returning the next destination's name for the input person: the destination could be another desk `d2, d3, d4, d5`, or `outside` in case the person leaves the employment agency without a job. `placeNameForPerson_structured` returns the name of the destination considering the input `Person` and also requires a second input, a `DeskWithJobs`, i.e. a desk with several lists of available jobs, considering the various cases. In the structured solution, the entire execution logic is written in the function `placeNameForPerson_structured`, and is in fact based upon nested conditional tests.

We can immediately see that it is rather hard to understand what happens in the function just by reading it once: probably, in a "real" software, a function like this would probably be annotated with **several comments**. In addition, it can be easily seen that to trace a bug in a function like this would require a step-by-step debugging procedure, because the large number of conditions makes the function basically not testable in an efficient way. Finally, changing conditions or introducing more of them would probably be really hard.

The underlying problem is actually due to the fact that, with a structured approach, we're **mixing the intent with the implementation**: in this case the intent is to associate groups of conditions to specific results, but these relations are "hidden" within the code and must be deduced from it; for example, if we are asked what is the door for an elderly person with two children, we'd be forced to follow the flow of code:

- because the first condition is related to the number of children, we immediately enter the branch `childrenCount != 0`;
- at this point we enter the `case 2` for the number of children;
- there's a condition for checking that there's a job available;
- a positive choice would lead to the "d4" desk;
- where's the "young/old" condition? Is there a bug? By analyzing the branch `childrenCount == 0` we can verify that this condition is actually checked, but it's simply irrelevant for the other branch;

It is a relatively long way to understand one simple thing, and it is noteworthy that if we wanted to make changes to the procedure, we should always run the entirety of it to make sure we covered all possible cases and there are no conflicts or ambiguities.

We can do better.

<a name="example-oo-solution"></a>
## Example: OO solution

OO design is a often a work of **both reason and creativity**: frequently, a same problem can be solved with multiple OO approaches. As said before, our intent is to *abstract* the concept of *condition*; considering this approach, let's try and establish some foundation, in order to identify the responsibilities and behaviors to be assigned to each class:

- we want to represent each condition as if it were a single *check* separated from the others, e.g. with a `Check` type characterized by methods that verify whether a given person passes a step or not;
- we want to represent separately the checks related to a person's attribute, like the number of children, to those related to the list of jobs to choose from;
- we want to build objects that are able to combine multiple checks into a single check;
- we want to create an object that represents a list of checks, and is able to assess whether the person is going to pass one and only one of those checks, or no check;

The file `oo.swift` contains the complete implementation of the OO solution. We define a `protocol PersonCheckType`, i.e., an interface that declares a `personIsValid` method that returns `true` if a certain `Person` passes the check:

```swift
protocol PersonCheckType {
    func personIsValid(person: Person) -> Bool
}
```

Then some classes implementing the `PersonCheckType` protocol are declared, which individually test the various inherent attributes of a `Person`.

The type `JobsType` simply declares a `availableJobs` property to collect the available jobs: from it more *checks* are derived, also of type `PersonCheckType`, that verify whether a person is valid from the point of view of preferred jobs.

Finally, the type `NextDeskType` declares a `nextDeskNameForPerson` method by which you can define what the name of the next desk for a person will be; the method returns `String?` because maybe there's no desk for a certain person:

```swift
protocol NextDeskType {
    func nextDeskNameForPerson(person: Person) -> String?
}
```

From `NextDeskType` we derive the following classes:

- `CheckNode`, which represents a node in the decision-making structure of the software;
- `CheckStructure`, which represents the entire tree of nodes;

In fact, the implementation of `nextDeskNameForPerson` in `CheckStructure` corresponds exactly to the solution of the problem: find the name of destination desk.

The idea is to create some simple **unit checks**, using various classes of type `PersonCheckType`, and combine those into concrete objects of the class `CheckNode`, thus defining our `CheckStructure`. In "pure" OO style, we defined classes called `ComposedCheck` and `MultipleCheck` to compose more checks into one, and `FailingCheck` to encapsulate a check that is required to fail. None of the implemented methods has conditional structures related to the *business logic*.

As an example, here's the implementation for the class `ChildrenCountCheck`, which validates a person considering the number of children:

```swift
class ChildrenCountCheck: PersonCheckType {
    
    let childrenCount: Int
    init(_ childrenCount: Int) {
        self.childrenCount = childrenCount
    }
    
    func personIsValid(person: Person) -> Bool {
        return person.childrenCount == childrenCount
    }
}
```

The `structure` object defined in `main.swift` contains the list of conditions, in the form of properly configured objects; as you can see, the list is easily readable and the conditions are clear: thus, we can *declare* the intent, separated from the implementation details.

<a name="example-functional-solution"> </a>
## Example: functional solution

In structuring the OO solution we have adopted a **composition principle**: atomic objects with simple behavior, properly prepared, composed with each other in order to obtain more complex structures. In particular, each of our objects represents a specific *atomic behavior*, that's combined with others to obtain a *compound behavior*. **The abstraction holds**, and we can accept the fact that the objects we're using are black boxes that encapsulate a single decision on a person: by creating a larger box with smaller boxes inside, we can define a more complex condition. However there are two problems specific to the OO approach:

- each class requires a lot of code just to build the "decision-in-the-box" abstraction;
- objects' composition is based on the methods of the objects themselves, that results, again, in writing a lot of code, which could obfuscate the intent;

You can get the same result, namely to create a complex decision-making structure from simple components, more clearly by using **functional programming**. The functional code is described in `functional.swift`. We always start by the immutable `struct Person` and create, for convenience, another simple data container, `struct DeskNode`, which simply pairs the name of a node to the name of a optional destination desk. For the rest, we will exclusively define functions, by modeling the entire problem as a series of data transformations.

In this case `PersonCheck` is a **type of function**, which associates a `Person` to a `Bool`. We also define the type of function `PersonNode`, which associates a certain `DeskNode` to a `Person`, and that has basically the purpose that the `nextDeskNameForPerson` method had in the OO solution.

Finally, we represent the various "nodes" with a function of type `JobsNode`, which constructs a `PersonNode` based on a certain `PersonCheck`: in this way, to build a list of `PersonNode` we'll just need to apply the various checks (`PersonCheck`) to the respective nodes (`JobsNode`).

The method most frequently adopted in functional programming to compose functions is to use custom operators. We want, for example, compose the various `PersonCheck` as if we were composing simple `Bool` values, by defining **AND** and **OR** operators. By association with the classic `&&` and `||` we define respectively the operators `<&>` is `<|>`, also copying [precedence and associativity](https://en.wikipedia.org/wiki/Operator_associativity) rules from the standard library. To improve readability and clarity of functions' composition, we also declare a *function application* operator, defined as:

```swift
infix operator <*> {
    associativity left
    precedence 100
}
func <*> <A, B> (left: A -> B, right: A) -> B {
    return left(right)
}
```

We can see from the definition that the operator `<*>` basically applies an argument (positioned to the right of the operator) to a certain function (positioned to the left of the operator): thus, it represents an alternative way to apply a function other than writing its arguments in **parentheses**, and is particularly suitable for improving the readability of the code when we use [higher-order functions](https://en.wikipedia.org/wiki/Higher-order_function). We can see these operators at work in `main.swift`, when we define our nodes, for example:

```swift
node("old", nextDeskName: "d3")
    <*> oldCheck
    <&> mustFail(childrenChecks)
    <&> checkExtendedJobs(availableJobsElderly)
```

The above-mentioned function builds a `PersonNode` named "old" that returns the desk "d3" if the input `Person` complies with following conditions:

- `oldCheck`: the person is elderly;
- `mustFail(childrenChecks)`: the person must not have children;
- `checkExtendedJobs(availableJobsElderly)`: the senior citizens jobs set and the person's favorite jobs set must have some intersection;

At first glance it may seem difficult to understand, but you just need to get the basic abstractions on which functional composition is based in order to appreciate its **power and expressiveness**. The array `nodes`, in `main.swift`, contains all the nodes of the problem, represented as functions: compared to the OO solution contained in `structure`, the functional one is more clear and readable.

<a name="an-empirical-validation"></a>
## An empirical validation

In designing the "OO" and "functional" solutions it was made the specific choice that the order in which the checks occur should be irrelevant: in this way we can remove a known cause of accidental complexity, the one of "control", i.e, the **order** in which the operations are carried out (we talked about it [here](http://engineering.facile.it/programmazione-funzionale-perche-preoccuparsi/#cause-di-complessita)). To do this, you must have **non-ambiguous** checks: at most one check must pass for a certain person. It'd be probably possible to write a formal method to prove that a given set of conditions is non-ambiguous, and then write a unit test based on such proof, but in this case an *empirical approach* seems more suitable and **easier** to implement.

It was therefore choosen to use a [QuickCheck](https://hackage.haskell.org/package/QuickCheck) approach: QuickCheck is a library for the [Haskell](https://www.haskell.org) language, that allows you to automatically test that certain functions satisfy certain properties. The idea is to verify that a particular function always behaves in the same way, testing it with **a large number of random inputs**: in our case, the function is just the list of checks, and the input is a value of type `Person`. To verify that the checks are not ambiguous we can just try them with a large number of randomly generated persons: if for a certain value of `Person` two or more checks are positive, the test stops and the ambiguous checks' names are logged. In the case of two ambiguous checks, to solve the problem it will suffice to make one of them more specific, for example by requiring that the other fails. We will use here the term *QuickCheck* to indicate the empirical test implemented, but in fact the original library does much more: please refer to the [linked](https://hackage.haskell.org/package/QuickCheck) page for all the information.

The functions `quickCheck_functional` and `quickCheck_oo`  implement the required feature: they generate a large number of random `Person`, and verify that the number of passed checks for each person is either 0 or 1. We can try this test by adding a new node. For example, suppose that at some point, after some time that the code was successfully used to solve the problem, you choose to add a new condition: there's a special desk for older people with only one child. Notice that modifying the code of the function `placeNameForPerson_structured` would be rather complicated: we should probably **change more than one decision branch**, adding the same condition at different points, and the likelyhood of error would be high. On the other side, with the other two solutions we just need to add a new node. However, after adding the node, by executing the program we will see that QuickCheck fails, printing the following message:

```
ambiguous nodes: [1 child, 1 child old]
```

QuickCheck is informing us that the nodes called "one child" and "one child old" are **ambiguous**: in fact, if the `Person` has one child and is old, the check for both nodes will pass. To solve the problem it will suffice to introduce another condition in the "one child" node: the seniority check must fail.

In the project you'll also find a `quickCheck_consistency` function, which performs a *sanity check*: it verifies that the target desk for a large number of randomly generated `Person` remains the same in the 3 cases: structured, oo and functional.

<a name="conclusions"></a>
## Conclusions

We have seen three ways to solve the same relatively complex problem, and in two of them we were able to get a **more manageable**, extensible and modifiable code by working at a **higher level of abstraction**. In particular, the "functional" way allowed us to write more compact code, characterized by a more evident intent, at the (low) cost of an initial step of abstraction, represented by custom operators. But the key to the whole process is once again to be found in the aforementioned phrase in *Structure and Interpretation of Computer Programs*, which I quote again:

> programs must be written for people to read, and only incidentally for machines to execute

Adopting a more **declarative** style, where the *business logic* code "declares the intent", describing the "what" and not the "how", you can write more clear and readable software. This software will probably also be more *testable* because it is based on lower level **atomic components**, easy to test individually, which are combined through patterns of composition, also easily testable and provable. The ultimate goal is to build software architectures that are able to **automatically solve problems**, which are described in code by using **basic building blocks**. Finally, we have seen how a mixed "reasoned" and "empirical" approach can give excellent results; *a priori* reasoning and *a posteriori* verification are both useful tools, but their union is particularly powerful: indeed, every scientific discipline requires both the logical-mathematical reasoning for the definition of the theories, and the experimental verification to test their validity.

The tools are there, we just need to **start using them**.
