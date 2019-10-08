---
title: "Android notifications — An elegant way to build and display"
date: 2017-08-26T07:20:21+02:00
featured_image: "/images/notifications/notifications.jpg"
---

{{< youtube LTqvVWvmACY >}}

Almost all of the Android apps I’ve been working on were commercial apps, and almost all of them are using the push notifications capabilities. Push notifications are a nice way to keep your users engaged, and from the business perspective, taking the advantages they provide is crucial. Of course, there are situations when push notifications doesn’t apply or make sense, but we’re not gonna talk about that now.

Displaying notifications on Android is fairly easy. The [documentation](https://developer.android.com/guide/topics/ui/notifiers/notifications.html) is very nice and clean, and describes the best practices and requirements to make the UX compliant to the standards. Nova days, the standard way to send a push to a device is by using the [Firebase](https://firebase.google.com/). If there are no customizations or custom payload, the effort on the Android side in oder to display a notification, branded specifically for the app is matter of defining some properties (like icon etc) in xml. However, this post is about receiving push notifications with custom payload, and display different notification based on that payload. The way I solve this I find pretty handy and I think it deserves to be shared, so other people may find it useful too, or they may get inspired to write a better one.

## Let’s dig into the code
First of all, we have to define the dependencies for google play services and Firebase. It’s very well explained [here](https://firebase.google.com/docs/cloud-messaging/android/client) what are the dependencies and their latest versions, so we will skip that step and start with defining services in our `AndroidManifest.xml` file:

{{< gist mitrejcevski c4a57ed65aaa8fbc74a18b0da3739419 >}}

The first one is the one in charge of receiving the events when Firebase is refreshing the device token, and the second one is about receiving the push notifications coming from Firebase. When Firebase refreshes the token, we have to send it to our backend so it gets updated and the device will keep receiving pushes. The implementation is rather pretty straight forward:

{{< gist mitrejcevski f37d8318d6ef8c2e43aa41717dac9c8c >}}

And the second service is the one that receives the push notifications. All it does is just deliver the received data into the PushNotification class that knows how to display a notification. Here is how it looks like:

{{< gist mitrejcevski 3a13f0cb5cc88cb719c9c907b6ab107c >}}

## The interesting part
It’s important to mention that the data that is received in the remote message is a `Map`, and one of key-value pairs inside is an `id`. This id is very important, because by using that id we will distinguish different types of push notifications, and ultimately we will display different notification. Let’s take a look at the implementation of the `PushNotification` class:

{{< gist mitrejcevski a2d810a707d2249883d09cc74aede9f7 >}}

To start of, let’s get through the class dependencies:

 - `NotificationManager` is the Android’s SDK notification manager that is used for showing the notifications by it’s `notify()` method.

 - `NotificationItemResolver` is an interface that provides a contract for resolving the received data into a type that will be used to build a displayable notification

 - `NotificationBuilder` is an interface that provides a contract for making `android.app.Notification` out of the type that was resolved by the resolver.

And here is how they are constructed

{{< gist mitrejcevski 4c721c37080a91d07ba747c06881e079 >}}

Since the `NotificationManager` is pretty straight forward, let’s take a look at the `NotificationItemResolver` and it’s concrete implementation:

{{< gist mitrejcevski 3741815cde5f8097de5cdd3822b008f1 >}}

The implementation of the interface is basically a factory that produces different types of `PushNotificationItem` based on the `id` value that comes from Firebase that we discussed before. When we would like to add new type, we will have to add new implementation of `PushNotificationItem` (which we will check in a moment) and define the mapping in the `resolve()` method. The `PushNotificationItem` is an interface that defines the basics of the notification that will be displayed:

{{< gist mitrejcevski 28c2f9546b4ab529833205af16b9b230 >}}

When targeting O, the API requires us to provide a channel in order to display a notification. The channel is sort of a group, and the user could enable/disable different channels. Therefore, the channel has an `id` and a `title`, and this is what it looks like:

{{< gist mitrejcevski 13eb9936ad0c325fa64c2fc0c878baff >}}

Next, the `PushNotificationItem` interface defines `title`, `message`, `smallIcon`, and `pendingIntent`. When the notification will be displayed, this are the items that are going to be displayed. A notification can have many other different things for content, but for this example we take only the basics. Also, a very important thing is the `PendingIntent` which describes what is going to be opened in the app upon click on the notification.

So far, we have defined two `PushNotificationItem` types that are resolved in our `PushNotificationItemResolver` implementation, mainly the `PushNotificationItem.Empty`, and `FriendSuggestionNotification`. Let’s get through the second one:

{{< gist mitrejcevski d177815a9f00cc7050767c19376b17c6 >}}

The empty one is quite same, just with different values correspondingly. And it just creates pending intent that opens the app’s main screen.

So far we’ve seen the mechanism behind mapping and preparing the notification data based on the received data from Firebase. We’ve seen that adding new type is as easy as defining a new class that derives from the `PushNotificationItem` and defining it in the resolver. If it has to go to a new channel that doesn’t exist, we will also have to define new channel class that derives from `PushNotificationChannel` and that’s it.

Now, let’s take a look at the definition and the implementation of the `NotificationBuilder` that actually creates a notification that would be actually displayed:

{{< gist mitrejcevski 3dcf75066ea640ed95fe2f5adee86737 >}}

As we may see, the implementation here is fairly simple. It just maps the values from the `PushNotificationItem` into an `android.app.Notification` using the `NotificationCompat.Builder`

Getting back to the `show()` method of the `PushNotification` class where we started off, it does the things pretty straight forward: first maps the data into a `PushNotificationItem`, and if the current build version is O, it also creates a channel. Then it triggers a `notify()` on the manager with the `android.app.Notification` that got built out of the `PushNotificationItem`

As we’ve seen so far, this solution is quite well separated and easy to deal with. It’s easy to provide translation for the notification texts, as all that has to be done is defining translated `strings.xml` without a need to do any changes on the notifications themselves. Defining a new type of notification requires the backend to send different `id` for that type, and a new class for it on the Android side. Changing the way the data is mapped, or extending the functionality is as easy as replacing the implementations of the interfaces. And ultimately, and very importantly — this solution is testable. It’s easy to write unit tests for the way the `NotificationItemResolver` does resolve the data and if it does properly, as well as writing tests for the actual `PushNotificationItem` types, and check if they return the desired values.

## Wrap up
That’s it, folks, I hope (and I’d be very glad) if somebody finds this useful and handy, and of course, I’m ready to get into some more explanations or general discussions if required.
