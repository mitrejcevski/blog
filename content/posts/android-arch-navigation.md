---
title: "Android Navigation Arch Component — A Curious Investigation"
date: 2018-05-23T07:19:26+02:00
draft: true
---
![https://www.pexels.com/](/images/navigation.jpeg)

## About
On the Google IO 2018 event, there was an announcement of JetPack — a set of libraries to help and boost the Android development. These libraries are bringing the development of the essential parts of an application to a completely new level. By essential parts I mean things like handling configuration change, persisting data, executing operations in the background and so on. Some of those libraries were announced at Google IO 2017, and by now are having a stable release, and some of them are completely new. One of the new libraries is the Navigation.

## What is Navigation
The Navigation component is a new architecture component that is supposed to simplify the implementation of the navigation in an application. At the very beginning, it’s important to mention that the Navigation component is scoped to a single Activity. It means that it embraces the idea of using single activity with multiple fragments for navigating through different parts of the application. So, an Activity(preferably an entry point into the application) would contain a Navigation component. Then, one of the fragments of this Activity could potentially point to another Activity, which would have to define its own Navigation scope. Now, this does sound a little bit tricky because of the well-known problems with the fragments and the fragment manager, but the good news is that this Navigation component is there to do these things for us. So the fragment management is done by itself.

## Why Navigation
This is one of the first questions that come to mind when thinking about the Navigation component. As mentioned earlier, there was a well-known problem with the fragments and many developers were trying to avoid using fragments, or at least avoid working with the `FragmentManager`. Turns out, the need of such component lays in the fact that the activities are having some limitations too.

{{< tweet 996426634541522944 >}}

Fair enough, the arguments that [Ian Lake](https://twitter.com/ianhlake) points out in this tweet are more than enough to think about making more robust and easy to use solution. The Navigation component is not only helping to sort out those problems mentioned by [Ian Lake](https://twitter.com/ianhlake) in that tweet, but it brings some awesome features like safe arguments passing, default arguments, handling deep links, automatic setup with side [NavigationView](https://developer.android.com/reference/android/support/design/widget/NavigationView) and with the [BottomNavigationView](https://developer.android.com/reference/android/support/design/widget/BottomNavigationView). Furthermore, there is also a visual editor where all these things can be wired up together, and the developers can take an overview of what’s going around and understand and manage the structure lot easier. Let’s take a closer look.

## Getting Started
I would highly recommend the official guide for the new Navigation component, as well as its documentation to get rolling with the implementation.

{{< figure src="https://developer.android.com/images/topic/libraries/architecture/navigation-graph_2x-callouts.png" link="https://developer.android.com/topic/libraries/architecture/navigation/navigation-implementing" caption="Get started with the Navigation component" >}}

I’ve created a simple playground project where I was trying various things with the Navigation component. Keep in mind that I am using the latest version of [Android Studio](https://developer.android.com/studio/preview/) from the Canary Channel.

{{< figure src="https://avatars1.githubusercontent.com/u/1390865?s=460&v=4" link="https://github.com/mitrejcevski/jetpack-playground" >}}

To begin with, we have to declare the Navigation component dependencies in our build file, followed by sync: (note: the versions of the dependencies are aligned with the time this article is written)

{{< highlight groovy >}}
implementation 'android.arch.navigation:navigation-fragment-ktx:1.0.0-alpha01'
implementation 'android.arch.navigation:navigation-ui-ktx:1.0.0-alpha01'
{{< /highlight >}}
