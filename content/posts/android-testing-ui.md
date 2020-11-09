---
title: "Testing Android UI with pleasure"
date: 2018-01-28T07:19:48+02:00
featured_image: "/images/ui_testing/espresso.png"
tags: ["Android", "Testing"]
---

## Intro
Although it should be very clear by now, testing is a very important part of the software development. Tests are, in a way, a construction around the software, which provides confidence for altering its structure (mostly to improve it) while being sure the behaviour stays the same.

This article is focused mainly on the UI tests in an Android application. Traditionally, testing the UI was done by running a mocked backend on the same machine where the device/emulator is exercising the UI tests. The mocked backend would provide the test-doubles for the running tests. However, this approach is quite tricky. The reason is that, in order to run the tests, there is an external dependency — a running backend. Therefore, beside the problem with the network latency, there is even bigger problem to run the tests in a CI or CD systems which are not maintained by ourselves. So in order to run the tests under those cloud based systems, the app would have to either reach out to real backend, or a backend that would mock the real one, and that would require quite some maintenance. On the other hand, **the goal is only to test the UI, not the network or the backend**. So how could this get solved?

Thinking about that, the test should make sure the UI acts properly. For a given input, the test should exercise if the proper output is being displayed, or with other words, should make sure the UI is changed as expected. Therefore, there is an idea to keep the test-doubles in the source code of the app. That way, the app will be independent in terms of having a mocked backend and it would be able to run the tests anywhere. Furthermore, the tests would be rock-solid and would run very fast, without any idling resources required. Let’s have a look how could this be achieved.

## Details
For the purposes of this article, I’ve created a [simple demo](https://github.com/mitrejcevski/ui-testing) that would help sorting out the ideas and the approach this article describes. *The demo is a login screen that shows error in case the username or the password is empty, or the auth is incorrect, otherwise it would open the main screen*. In order to make it as simple as possible, I decided to use only espresso, and no other 3rd party libraries. The project has 2 flavours **mock** and **prod**, and there in an implementation of the data source in each of them. The **prod** flavor has an implementation that does the real work, and the **mock** flavour has an implementation that provides the test-doubles. That is the key point in this approach. Since we know exactly what we could expect from the backend for a given input, we could mock that data and return back towards the UI instantly, instead of making any actual calls. So the **mock** would be used in order to run the UI tests, and the **prod** for everything else (like unit tests, or making release builds). As mentioned before, instead of adding flavours, Dagger would have done a great job too, but the idea is to avoid any additional setups, and get to the point in a very simple way. Let’s take a closer look:

{{< imgpreview src="/images/ui_testing/login_home.svg" caption="The login and the main screen" >}}

Initially, the app opens the login screen. As described before, if the username or the password is empty, the screen should show an error. Also, we need to have tests that will make sure for given incorrect auth an error is being shown, and for given correct auth, the main screen is open. Here is the test case:

{{< gist mitrejcevski ec7f9ddddfae52946e89ed046cbd9c9f >}}

The test case should provide enough information about what is expected from the login screen. Since I wanted to put the focus on the UI tests, I made the `Activity` to talk directly to a data source without adding `Presenter` or `ViewModel`. There is a [branch](https://github.com/mitrejcevski/ui-testing/tree/full) with that implementation in the [project](https://github.com/mitrejcevski/ui-testing), and that is discussed later. For now, the `Activity` talks to the data source interface which looks like this:

{{< gist mitrejcevski 0a31140caead3bbbcd46adb3680b8cbd >}}

This interface has 2 implementations, one for each flavour respectively. The code structure looks like this:

{{< highlight text >}}
src/mock/java/nl/jovmit/login/RemoteLoginDataSource.kt
src/prod/java/nl/jovmit/login/RemoteLoginDataSource.kt
{{< /highlight >}}

This way, the data source implementation would be chosen by the selected flavor. For the sake of the simplicity, we would take a look only in the mock implementation:

{{< gist mitrejcevski 653ac65a9cf7f415914881ddbdbcd221 >}}

As we could see, the implementation just provides the desired test-doubles based on the input, so we could directly focus on the UI tests. This way, there is no need to reach out to a real backend or any other mocking backend. We know what the backend would return and in which way it would return it, so we could simply mock its response and have it in the source code. Ultimately, the code in the `Activity` is rather naive and quite out of real world, but for the sake of the example it does the job:

{{< gist mitrejcevski 12f8f04a742f891684f588b7a0752727 >}}

Now, by selecting the **mock** flavour and executing the test case, all tests would pass. The tests are quite solid and stable and more importantly they can run wherever we want, without thinking about having an up and running backend (real or mocked). The rest of the code can be easily unit tested, and those tests are included in the repository. Just they are out of the scope of this article to be discussed here. The implementation in the **prod** flavour on the other hand, would do the real work and make call to real backend. As we could conclude, it would be pointless to make calls if the username or the password is empty. Therefore, that case can be and should be covered by the unit tests for the **prod** implementation.

## The other branch
As I mentioned earlier, the [project](https://github.com/mitrejcevski/ui-testing) also contains a [branch](https://github.com/mitrejcevski/ui-testing/tree/full) named **full** which includes the Android Architecture Components ([LiveData](https://developer.android.com/topic/libraries/architecture/livedata.html) and [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel.html)) and it makes a little bit more sense from a real-world perspective. There is not a single change in the UI tests, and they still pass. However, there is some tweaks in the implementation details underneath. Mainly, the branch introduces a `LoginViewModel` that takes the responsibility of performing the login and exposing live data to be observed by the `LoginActivity`. The `LoginViewModel` then talks to the data source. The `LoginViewModel` and the `RemoteLoginDataSource` are both unit-tested.

## Wrap up
Using this approach for a while now, and I found it as a very nice way to test the UI. The tests are very solid and stable. Having both UI and unit tests brings so much confidence that makes the development very joyful and fun. No matter if we are about to run the tests in the local development machine, a CI server or a test lab, tests would run nicely. And to bring the whole thing a step further, we could deliver the testing report in a Slack channel once the tests are done, so we got an immediate feedback and we know if everything’s fine.

In this particular case, going by the flavours approach, there would be 2 commands to run the UI and the unit tests, one for each respectively:

`./gradlew testProdDebug` to run the unit tests. We want to run the unit tests agains the production implementation, because that’s the one providing the relevant business logic.

`./gradlew connectedMockDebugAndroidTest` to run the UI tests. We want to run the UI tests agains the mock implementation, so it will use the test-doubles for fast response, and performing the UI tests reliably.

Espresso is an amazing framework and it does an incredibly good job. In comparison to the old android instrumentation framework, it’s way more superior and lot more stable and reliable.

## Having different and/or better ideas?
If you know a better or nicer or simpler way for doing this, please feel free to open a PR or just another branch, or at least an issue so we could discuss, share knowledge and learn from each other.
