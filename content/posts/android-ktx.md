---
title: "Android KTX"
date: 2018-06-29T10:25:45+02:00
featured_image: "/images/ktx/android_ktx.png"
tags: ["Android"]
---
## Intro
[Android KTX](https://developer.android.com/kotlin/ktx) is an open source library or set of functionalities designed to make the Android development with Kotlin even more pleasant. The abbreviation KTX stands for Kotlin Extensions, so this library is basically a set of extension functions, extension properties and other top-level functions. In this article, we take a look at what's inside this library and how we can take advantage of it. This library's goal is not to add new features to the existing Android APIs, but rather make those APIs easier to use by leveraging the features of the Kotlin language.

## Structure
A very important thing to note at the beginning is that [Android KTX](https://developer.android.com/kotlin/ktx) provides functionalities which would be added to many individual projects by the developers most of the time anyway. Arguably, there are also many things that will get included by adding the dependency, which are not going to be used. Thanks to the [ProGuard](https://developer.android.com/studio/build/shrink-code), all the unused code will get stripped out, so there should be no worry regarding the library footprint. Now, let's take a look at some of the very important and crucial concepts of the Kotlin language which are used for building the [Android KTX](https://developer.android.com/kotlin/ktx).

### Extension functions
An extension function is one of the Kotlin's features that allows us to add a functionality to an existing class without modifying it directly. By adding extension functions to a class, they are represented as simple static methods on bytecode level. Then, we can call those functions on the objects of that class type, just as they were part of the class's API initially. Here is an example to give a little better insight into the way it's done. Say, we want to add an extension function to the `String` class to print itself. Here is how we can do it:

{{< highlight kotlin >}}
fun String.printSelf() {
    println(this)
}
{{< /highlight >}}

Inside the extension function, we can use `this` to refer to the current object on which the function is being executed. Then, this function becomes available to be called on any object of `String` type. So we can do:
{{< highlight kotlin >}}
fun usage() {
    "Kotlin Rocks".printSelf()
}
{{< /highlight >}}

### Extension properties
Similarly to extension function, Kotlin supports extension properties. Also, the way we define them is quite the same:

{{< highlight kotlin >}}
val String.isLongEnough: Boolean
    get() = this.length > 5
{{< /highlight >}}

Then, we can use this extension property on any object of `String` type:

{{< highlight kotlin >}}
"Kotlin".isLongEnough
{{< /highlight >}}

The behavior of the extension property can only be defined by explicitly providing getter (plus setter for `var`s). Initializing those properties the ordinary way doesn't work. The reason behind this is that an extension property does not insert members into the type, so there is no efficient way for it to have a [backing field](https://kotlinlang.org/docs/reference/properties.html#backing-fields).

### Top level functions
In Kotlin, a function is a first-class citizen. We are allowed to define functions in Kotlin files (.kt) which could afterward be accessed and used from other Kotlin files. This is a very powerful concept. If we define a file inside a package `com.example` and define functions in it, they can be used simply by importing them into the usage side. Here is an example:

{{< highlight kotlin >}}
package com.example

fun sum(x: Int, y: Int): Int = x + y
{{< /highlight >}}

now this function can be accessed and used from any other Kotlin file by simply importing it like this:

{{< highlight kotlin >}}
import com.example.sum

fun test() {
    println(sum(1, 2))
}
{{< /highlight >}}

An important note here is the access modifier. In the example above, the function `sum` does not have defined an access modifier, and in Kotlin it's `public` by default. Being public, it makes the `sum` function accessible from any other Kotlin file. Kotlin also has an `internal` access modifier keyword, which would make this function accessible only in the module where this file exists (a [module](https://kotlinlang.org/docs/reference/visibility-modifiers.html#modules) could contain many different packages). Ultimately, the function could also be `private` which will make it accessible only from inside the file where it is defined.

### Default arguments
Most probably, you are familiar with the overloading concept like in Java already. Overloading is a concept that allows us to define constructors or methods with the same signature, only differing in their parameter list (both types and number of parameters could be different). In Kotlin, this concept is taken a step further, so we could achieve the same result by defining a single function, by specifying default values to some or all of the arguments. Eventually, it boils down to the approach used in Java again. Let's take a look at the following example:

{{< highlight kotlin >}}
fun greet(firstName: String, lastName: String = "") {
    println("Hello $firstName $lastName")
}
{{< /highlight >}}

In this example, the function takes two arguments, but the `lastName` is optional because by default its value is going to be an empty `String`. So, when calling this function we are only required to supply a `firstName`, and we can call it this way:

{{< highlight kotlin >}}
greet("John")
{{< /highlight >}}

Kotlin also supports named arguments, so we could call a function supplying the arguments by their names:

{{< highlight kotlin >}}
greet("John", lastName = "Doe")
{{< /highlight >}}

This is particularly useful when we have a function with multiple optional arguments and we want to call it with supplying only specific ones, or in a different order.

## Deep Dive
Now, as we know what the [Android KTX](https://developer.android.com/kotlin/ktx) library is based on, let's dig and observe some of the most common extension functions.

#### Converting URL to URI
To begin with, there is a very simple extension function on the `Uri` class. Many times in Android, we need to convert a `String` URL into a `Uri` object, for instance when creating an `Intent` with data etc. The way we are usually doing this is by calling `Uri.parse("string_url")`. [Android KTX](https://developer.android.com/kotlin/ktx) defines an extension function that does the job. Here is how it looks like:

{{< highlight kotlin >}}
inline fun String.toUri(): Uri = Uri.parse(this)
{{< /highlight >}}

so we can use it by calling `toUri()` on any string, like this:

{{< highlight kotlin >}}
"any_sting_url".toUri()
{{< /highlight >}}

#### Editing shared preferences

Next, let's take a look at an extension function defined on the `SharedPreferences` interface. The usual way of putting values into the `SharedPreferences` in Android is to call `edit()` in order to obtain the `SharedPreferences.Editor` instance. Then, we can insert the values by calling `editor.putType("key", typeValue)`. After that, it is very important to call `apply()` or `commit()` on the editor, in order for the values to be stored in the `SharedPreferences`. Many times we forget doing so, and we waste some time debugging until we notice what is happening. A usual example of storing values in the `SharedPreferences` looks like this:

{{< highlight kotlin >}}
val editor = sharedPreferences.edit()
editor.putString("key", value)
editor.apply()
{{< /highlight >}}

By using the [Android KTX](https://developer.android.com/kotlin/ktx) extension on the `SharedPreferences` the code shortens and simplifies quite a lot, and it becomes:

{{< highlight kotlin >}}
sharedPreferences.edit {
    putString("key", value)
}
{{< /highlight >}}

The relevant extension function looks like this:

{{< highlight kotlin >}}
inline fun SharedPreferences.edit(
    commit: Boolean = false,
    action: SharedPreferences.Editor.() -> Unit
) {
    val editor = edit()
    action(editor)
    if (commit) {
        editor.commit()
    } else {
        editor.apply()
    }
}
{{< /highlight >}}

The first parameter to this function is a `Boolean` value that controls the call to the editor, whether it would use the `commit()` or the `apply()` call. Clearly, by default this value is set to `false` which means by default the function will call `apply()` on the editor. A more interesting parameter is the second one. It's a function literal with a receiver, and the receiver is of type `SharedPreferences.Editor`. It means that when calling this function, we can use lambda over the receiver, so we can directly call functions that are exposed by the receiver type without additional qualifiers. This is shown with the `putString` call.

#### Operating on view before drawing

Most of the apps we are using every day are having some sort of lists where some images are being loaded. Often, those images are of a different size, and the image sizes are usually provided in the response. Since the images are normally loaded in the background, we want to allocate the space required for the image to be displayed, and once it's loaded we already have the space allocated, so we would avoid UI expanding when the image is being displayed, which prevents the UI flickering effect. This is usually done by using a `ViewTreeObserver` which provides an `OnPreDrawListener`. This listener has a callback that is being called before the view drawing. For our images example, usually we set the view sizes that are provided in the response inside this callback, and we apply some default background (for example gray). That is one of the many use cases of the `ViewTreeObserver` observer and its `OnPreDrawListener`. Here is a snippet that shows the way we normally approach it:

{{< highlight kotlin >}}
view.viewTreeObserver.addOnPreDrawListener(
    object : ViewTreeObserver.OnPreDrawListener {
        override fun onPreDraw(): Boolean {
            viewTreeObserver.removeOnPreDrawListener(this)
            performSomethingOverTheView()
            return true
        }
    })
{{< /highlight >}}

[Android KTX](https://developer.android.com/kotlin/ktx) has defined an extension function on the `View` type named `doOnPreDraw()` that simplifies the above snippet, so it would become:

{{< highlight kotlin >}}
view.doOnPreDraw {
     performSomethingOverTheView()
}
{{< /highlight >}}

There are also some other nice extensions defined on the `View` type which are working with the view visibility, updating view padding or layout params etc.

#### Working with bundle

Bundles are a very common thing in Android, but working with bundles is often quite a boilerplate. The way we normally compose a bundle looks like this:
{{< highlight kotlin >}}
val bundle = Bundle()
bundle.putString("key", "value")
bundle.putString("keyBoolean", true)
bundle.putString("keyInt", 1)
{{< /highlight >}}

[Android KTX](https://developer.android.com/kotlin/ktx) defines a top-level function named `bundleOf()`, and by using it, composing bundle becomes a lot nicer:

{{< highlight kotlin >}}
val bundle = bundleOf("key" to "value", "keyBoolean" to true, "keyInt" to 1)
{{< /highlight >}}

There is also a `persistableBundleOf()` function for creating a `PersistableBundle` but it's available for API version 21 and up. Similarly, there is a `contentValuesOf()` function that could be used in the same way as the functions for creating bundle above, and it returns a `ContentValues` object.

#### Iteration over view group
Working with [ViewGroup](https://developer.android.com/reference/android/view/ViewGroup) in Android is quite a common thing.  The [ViewGroup](https://developer.android.com/reference/android/view/ViewGroup) is a kind of container that could contain other views called children. Many times we need to loop through its children, but the traditional way to do so could be quite a mess. [Android KTX](https://developer.android.com/kotlin/ktx) has defined an extension property that exposes its children, and here is how it looks like:

{{< highlight kotlin >}}
val ViewGroup.children: Sequence<View>
    get() = object : Sequence<View> {
        override fun iterator() = this@children.iterator()
    }
{{< /highlight >}}

As we can see, the children property is returning a sequence of child views, and it allows us to write very concise loops over any [ViewGroup](https://developer.android.com/reference/android/view/ViewGroup) type:

{{< highlight kotlin >}}
viewGroup.children.forEach {
    doSomething(it)
}
{{< /highlight >}}

#### Displaying toast

One of the most common ways of displaying some sort of short info to the user is a `Toast`. When displaying a toast by using the standard API, we have to pass the `Context`, the actual message that we want to be displayed, whether it is a `String` or a resource reference to load value from the `strings.xml`, and the toast duration. Since we cannot display a toast without having a `Context`, and since most of the time the toast duration is its `Toast.LENGTH_SHORT` constant, it would be great to have an option to display a toast by simply calling a function and passing the actual message argument. A common way to achieve this is by defining some sort of base class that defines such method, and then it would be called from the subclasses. However, [Android KTX](https://developer.android.com/kotlin/ktx) defines an extension functions to the `Context` type for displaying toasts. It takes 2 arguments, the message and the duration, while the duration is being set to `Toast.LENGTH_SHORT` by default. So we can call to display toast wherever we have a `Context` by simply passing the message argument, whether it is a `String` type or a reference to the string resources:

{{< highlight kotlin >}}
toast("Hello")
toast(R.string.hello)
{{< /highlight >}}

## Wrap Up
[Android KTX](https://developer.android.com/kotlin/ktx) is a very nice part of the [Android JetPack](https://developer.android.com/jetpack/) project, and as we've seen it contains quite some nice ways to improve the Android development we are used to. As told before, it doesn't provide any new functionalities, but rather simplifies the APIs which are already provided by the Android SDK. [Here](https://www.youtube.com/watch?v=st1XVfkDWqk) is a nice talk from the [Google I/O 2018](https://events.google.com/io/recap/) by [Jake Wharton](https://jakewharton.com/), where he elaborates more on the [Android KTX](https://developer.android.com/kotlin/ktx). This article only scratches the surface, and there are many more fun things to be revealed inside the [Android KTX](https://developer.android.com/kotlin/ktx), related to `animation`, `database`, `location`, `graphics`, `text` and so on. Also, the team is welcoming new contributions, which is great, so if you have an idea that is not there yet, feel free to submit it.
