---
title: "Writing Screenshot Tests with Paparazzi"
date: 2023-08-14T00:26:55+02:00
featured_image: "/images/paparazzi/paparazzi.png"
tags: ["Android", "Testing", "Screenshot"]
---
## Introduction
Screenshot testing is one of the most powerful ways to make sure your UI remains looking the same over time. With screenshot testing, we aren't only checking if the UI elements are present on the screen, but also if they are positioned well, and if they look as they should.

What's good about the Screenshot testing lately is that it gets fast, stable and reliable way to test UI. In terms of Android, a bunch of frameworks are available lately, and one of them is [Paparazzi](https://github.com/cashapp/paparazzi)

In this article we'll look into how to set it up and use it.

## Setup
Probably the most important thing to keep in mind before approaching Paparazzi is that _it must be setup in a module other than your main app module_. The problem is in the way and the order in which things are being compiled in Android, and {{< twitterfollow account="jrodbx">}} explained it well, so if you are looking for the details - John got them all.

That's in fact a good thing because it forces us to move the composables into a different module, and make them stateless. In general, that is a good practice. So go ahead and create a new module (if you haven't one already) for composables and their states.

### Dependencies
When it comes to the dependencies, there is only a single plugin dependency that we need to bring in.

#### Plugin Application
In your project level `build.gradle` add the following:

{{< highlight kotlin >}}
buildscript {
  repositories {
    mavenCentral()
    google()
  }
  dependencies {
    classpath "app.cash.paparazzi:paparazzi-gradle-plugin:version"
  }
}
{{< /highlight >}}
And then in your app level `build.gradle` add:

{{< highlight kotlin >}}
apply plugin: "app.cash.paparazzi"
{{< /highlight >}}

#### Plugins DSL
If you are using plugins DSL, add the following:
{{< highlight kotlin >}}
plugins {
  id "app.cash.paparazzi" version "<paparazzi_version>"
}
{{< /highlight >}}

#### Version Catalog
Ultimately, if you are using Version Catalog (which you should), do this:

In your `libs.versions.toml` in the `plugins` section, add:

{{< highlight kotlin >}}
paparazzi = { id = "app.cash.paparazzi", version.ref = "paparazzi-version" }
{{< /highlight >}}

Next, in your root `build.gradle` plugins add:

{{< highlight kotlin >}}
plugins {
 ...
 alias libs.plugins.paparazzi apply false
}
{{< /highlight >}}

And ultimately, in your module's `build.gradle`, in the plugins, add:

{{< highlight kotlin >}}
plugins {
    ...
    alias libs.plugins.paparazzi
}
{{< /highlight >}}

That is enough for the setup. BTW, if you need help introducing version catalog in your app - comment below, and I'll write a tutorial.

## Usage
Now we can go ahead and start writing screenshot tests. If you are using `Previews` for your composables, you are already in a great position, because writing screenshot tests for your composables will be as effortless as a copy-paste.
> Note: Paparazzi works with the traditional View system just as well. However, in this post we only talk about Composables.

As established before, we should be in a module other than the main app module. It can be a single module where we host all our composable and their states, or we might have feature modules holding their own composables. Both are equally fine.

### Writing Tests
The best thing about Paparazzi is that we write unit tests that run very fast. Plus they are stable and reliable. A Paparazzi test is very similar to a regular unit test. First we need to get the Paparazzi rule:

{{< highlight kotlin >}}
@get: Rule
val paparazzi = Paparazzi(
    deviceConfig = DeviceConfig.PIXEL_5
)
{{< /highlight >}}

Then we can use it to write a test:

{{< highlight kotlin >}}
@Test
fun loginScreenSnapshot() {
  paparazzi.snapshot {
    LoginScreen(...) //<- The composable to make a screenshot from.
  }
}
{{< /highlight >}}

That's all! Now we can run this test. As we can examine from the example above, the API is super simple and easy to use. The rule allows us to create a configuration that suits us. All we need to do is wrap our composables in the `snapshot` lambda.

### Taking Screenshots
Once we write a test and run it - it will pass. But it won't make a screenshot. It will pass because there was no screenshot created before. We need to generate screenshots from our tests initially, so then when we run the tests again, they will be checking against something.

To create the screenshots, in the terminal, we need to run
{{< highlight kotlin >}}
./gradlew module:recordPaparazziDebug
{{< /highlight >}}
Running this command will generate the screenshots in the tests directory `snapshots` folder.

Then, for subsequent runs, we can use
{{< highlight kotlin >}}
./gradlew module:verifyPaparazziDebug
{{< /highlight >}} to check if we happened to break any of the composables.

### Leveraging Previews
A very good and common practice in a typical Jetpack Compose codebase is writing `previews` for the composables. It allows us and the other developers to see what a composable is rendering, and how the UI it spits out looks like. When that is the case, writing the Paparazzi tests boils down to copy-pasting the preview into the `paparazzi.snapshot {}` lambda.

## Next Steps
If you are looking into more resources to learn Jetpack Compose and Android Development, [join my free training here](https://www.skool.com/android-devs?invite=e7235a1900bf49cb89c043e0fb9753d3).

Happy Coding!