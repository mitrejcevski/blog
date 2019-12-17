---
title: "Setting up Koin for Android UI testing"
date: 2019-12-14T09:11:33+01:00
featured_image: "/images/koin/insert_koin.jpg"
---

{{< youtube _BdgqavMj8o >}}

## Problem definition

Recently I've come to a problem running my UI tests because of a `DefinitionOverrideException`. That's an exception thrown by the [Koin](https://insert-koin.io/) library when we have a duplicated definition for a particular type. The way we use [Koin](https://insert-koin.io/) on Android is by starting the container in the `onCreate()` method of the `Application` class and loading the relevant *modules* into the container. Here is how it may look like:

{{< gist mitrejcevski a52a6abf4d3d081bf9a8c5959d09c8c4 >}}

The problem is that the default test runner uses this `Application` class to launch the app for the UI tests. It means that on the application startup, those *modules* will get loaded in the [Koin](https://insert-koin.io/) container. However, in our tests we want to be flexible with the loaded *modules* - don't load the production *modules* and load only *modules* that are relevant for the current test cases. Most of the times we want to use classes different than the real production classes (`Mocks`/`Test Doubles`/`Stubs`/etc.), to avoid making real **network calls**, or real **database manipulation**, and to control the output for a given input. That helps us make the tests reliable and fast, but more on that topic in another blog post. Now, normally we would like to load specific *module* per particular test case. Here is an example test setup:

{{< gist mitrejcevski 4389bba710befc592026502012ab4488 >}}

So, since the *module* in this test contains a definition for a type that was already loaded within the application startup, the [Koin](https://insert-koin.io/) library throws that `DefinitionOverrideException`, meaning that we cannot override the definition that was already loaded.

## Solution

The way to solve this problem is for the tests to use an `Application` class different than the **default** `Application` class. So, we can define a **testing** `Application` class in the UI testing folder, because it will be relevant only for the UI tests. Here is an example of how it may look like:

{{< gist mitrejcevski ade42b20172219f2e0b9667160cc7056 >}}

#### An empty container

In the `onCreate()` method of the `TestRoboApp` class we are starting an empty [Koin](https://insert-koin.io/) container, and we are stopping it in the `onTerminate()` method. When running the tests, for each test case we have an empty container where we can load relevant *modules* that we need, and unload them when we are done. We can do so by using the `@Before` and `@After` annotated methods:

{{< gist mitrejcevski 9410651bd14061b97371de2c509a4ec3 >}}

#### A custom test runner

Next, in order to use the newly created **testing** `Application` class instead of the **default** one, we have to define *custom test runner* and configure it to use the class we want. We can create this *custom test runner* inside the UI testing folder as well:

{{< gist mitrejcevski c5f14b7c1d9412bdbe514a4128b04d81 >}}

All we have to do to configure this *custom test runner* is to override its `newApplication()` method and use the **class name** of our **testing** `Application` class.

#### Replace the default test runner

Ultimately, in our application's `build.gradle` file we need to replace the default test runner
```
testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
```
with our *custom test runner*:

```
testInstrumentationRunner = "nl.jovmit.roboapp.RoboTestRunner"
```

Once we replace, we have to make sure to *sync* the project so the test runner replacement will get recognized. Once the sync is done, we can go ahead and use custom *modules* for particular UI tests.

## Wrap up
We've seen how we can take advantage of a *custom test runner* to make our tests more independent, and how we can benefit from it when using 3rd party libraries like [Koin](https://insert-koin.io/). I hope you find this article useful and applicable in your case. Feel free to [follow me on Twitter](https://twitter.com/jovchem) so you will stay up to date with new content.

{{< twitterfollow account="jovchem">}}
