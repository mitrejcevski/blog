---
title: "Refactoring Legacy Code"
date: 2020-01-10T14:27:01+02:00
draft: true
---

## Intro

{{< youtube 1ap11RbVGik >}}

Quite often in software development, we come to a point where we have to alter some code, whether to make it easier to read and understand or to add a new feature.

Recently a friend of mine posted a piece of code in a public Slack channel, and he asked for resources that would help him improve not only that particular code but any legacy code in general. I'm very thankful that he asked that because I took it as a motivation to record a series of screencast videos where I explain and demonstrate a bunch of techniques that everyone can learn and use in their daily job.

## Books
The techniques I am working with while refactoring the code in the screencast are heavily based on the following books:
 - [Working effectively with legacy code]("https://bit.ly/2R8lkVM") by [Michael Feathers]("https://twitter.com/mfeathers")
 - [Refactoring]("https://amzn.to/2ucyGaB") by [Martin Fowler]("https://twitter.com/martinfowler")
I find those books very valuable when it comes to software development, and I would strongly recommend everyone to read them.

## Code Refactoring
Code refactoring is a process of changing an existing computer code in such a way that it does not alter the external behavior of the code while it improves its internal structure.

#### Automated Refactoring (Supported by the IDE)
The IDEs that are widely used today, especially the ones for the strongly-typed programming languages, provide a variety of automated refactoring, often even suggesting possible refactoring that would improve the code. It's safe to perform such refactoring because the behavior of the code it is getting applied to is going to stay the same. In that sense, the more we use the automated refactoring provided by the IDE, the better. Minimizing manual refactoring is always a good idea.

#### Frequent commits
Generally, making frequent commits are a valuable consideration to be taken, especially when it comes to refactoring legacy code. It's quite common to make a mistake when working with code so that we have to revert the latest changes. By committing frequently, we are minimizing the changes we might have to undo, and we make the actual revert easy to be done. Given that the legacy code could be quite complex, having the ability to make easy revert is becoming very valuable.

One may argue that creating too many commits might become problematic in some cases, and if that's the case, we can easily squash them into fewer before merging into the targeting branch. However, I will advocate for keeping the full history for better evidence.

#### Test Harness
In the "Working effectively with legacy code" book, the term **test harness** refers to a testing code that we write to exercise a piece of software and the code that we use to run it. In that sense, when working with legacy code, the tests that we write together with the running set up to cover a piece of functionality is called **test harness**. Once we have the code under test harness, we have the safety and the confidence to start refactoring.

#### Code Coverage
The code coverage tool comes to great help when refactoring legacy code. It plays an important role when putting code under test harness by allowing us to see which parts of that code are getting executed. Once we see the parts which are getting covered, we can focus on a particular region and understand what it is doing. An essential note at that point is not to get tricked if a line of code is covered. A covered line of code means that the line is getting executed, but it does not mean that its behavior is preserving. We need additional checks to make sure that we are maintaining the side effect of its execution. That's where the **defect injection** comes into play.

#### Defect Injection
The defect injection is a technique that we use to make sure a particular test is failing for the right reason. As mentioned previously, a covered line of code is not the same as preserved behavior. Usually, the test that makes sure a block of code is covered is different than the test that checks its behavior. It is not impossible, but often, those are separate tests. So, to prove a behavior is remaining the same, we can inject a defect in a block of code that we are exercising, so that the test that penetrates that code will fail. To do so, we can comment out a line of code, or make a small change that would make the test fail. Then we would observe if the test fails because of the defect we have injected. Once we have the confidence that the behavior is not changed, we can undo the injected defect and continue.

#### Test Data Builders
A good test is the well-focused one, the one that clearly describes its intention and involves only the relevant details. Very often, we need to involve data in a test. Thus, we might end up initializing and arranging not particularly relevant data structures into its setup. In those situations, we can make use of a data builder - a helper used in the tests that leverage the builder pattern. Let's take a look at a concrete example. Here is a DTO class with a handful of properties:

{{< gist mitrejcevski eff3e55a81f6551acd13ea2a0bd132c7 >}}

and a test where we are checking the result of a search call:

{{< gist mitrejcevski 6ef41a6f04090e338305d59d2e6bf4a8>}}

In the test setup, we are initializing a new user object, and doing that, besides the value for the `userId`, we have to provide values for the `firstName`, `lastName`, `age`, and `location`. However, all the values in that test except the `userId` are **completely irrelevant**. By using a data builder, we can significantly improve the readability and the intention of the test.

{{< gist mitrejcevski 380433c58d9688d877db6137952b5fc6 >}}

Now, we have dropped the irrelevant details from the test, and we ended up with a more focused test.

#### Seam
When refactoring legacy code, often there are parts of the code involving static calls, calls that are expensive to be made (like querying database), or usage of constant fields. Putting those parts under test harness is not a straight forward task. By extracting such a call into another function, we are opening an ability to replace it with something that makes more sense for the test harness. That newly extracted function is called **seam**.

#### Subclass to Test
To make use of the seam, we need to create a **subclass** and override the function, so that it will call or return whatever we need in the particular test. This subclass is relevant only in the tests, and we are calling it **subclass to test** or a **testing subclass**. Let's take a look at a concrete example. Here is a piece of code that we want to put under a test harness:

{{< gist mitrejcevski cbe0b4a1ecae6619753f13be575620b7 >}}

There is a static call to find trips `TripsDao.findTripsFor(userId)`, so if we call this function in a test, it might try to make a real query to the database, which in turn will take a lot of time, it might throw an exception, but there is no way for us to know the exact return result. Furthermore, making real calls to a database from a test will lead to flaky tests and might corrupt the database records. To get over that problem, we can create a seam, so that we will allow ourselves to put this code under test harness easily:

{{< gist mitrejcevski 9e90b88381003d81c8af61b8106f21e1 >}}

It's essential to note that by creating this seam, we are not changing the behavior of the code. We only delegate the call to the new function we created.

In languages like [Kotlin]("https://kotlinlang.org/"), where the classes and functions are implicitly `final`, we need to make them `open` to be able to override them, but more about that in the [Complexity](#complexity) section.

Once we have created the seam, in the tests, we can easily create a **testing subclass** where we can override the function and return a result that we can control.

{{< gist mitrejcevski 1295abb3ee3a602adc1ea35ea7a12697 >}}

#### Complexity


## Conclusion
Improving legacy code is not a trivial task and it always takes a lot of effort an energy. It's very crucial to have a good arsenal of tools available to fight against it, and to know how to use them properly. Practicing is important, the more we use them the more confident we become and we build experience that helps us pick the right tool for particular case faster.
