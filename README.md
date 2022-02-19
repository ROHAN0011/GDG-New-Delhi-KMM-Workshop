# RSS Reader

<a href="https://youtu.be/6HNUqDL-pI8">Clik here for more information about RSS Reader</a>

## Android and iOS Experiment

<img src="https://github.com/ROHAN0011/Kotlin-Multiplatform-Mobile-KMM/blob/c2d5b3f1703727c3405c8e2453f75f7e6c5571f4/ios+android.png"/>  

## Desktop and Web Experiment

<img src="https://github.com/ROHAN0011/Kotlin-Multiplatform-Mobile-KMM/blob/c2d5b3f1703727c3405c8e2453f75f7e6c5571f4/desktop+web.png"/>

## Project Structure

This repository contains a common Kotlin Multiplatform module, a Android project
and an iOS project. The common module is connected with the Android project via the
Gradle multi-project mechanism. For use in iOS applications, the shared module compiles into a
framework that is exposed to the Xcode project via the internal integration Gradle task. This
framework connects to the Xcode project that builds an iOS application.

You can achieve the same structure by creating a project with
the [KMM Plugin project wizard](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform-mobile)


<img src="https://github.com/ROHAN0011/Kotlin-Multiplatform-Mobile-KMM/blob/c2d5b3f1703727c3405c8e2453f75f7e6c5571f4/basic-structure.png"/>

## Architecture

Kotlin Multiplatform Mobile is a flexible technology that allows you to share only what you want to
share, from the core layer to UI layers.

This sample demonstrates sharing not only the data and domain layers of the app but also the
application state:

<img src="https://github.com/ROHAN0011/Kotlin-Multiplatform-Mobile-KMM/blob/c2d5b3f1703727c3405c8e2453f75f7e6c5571f4/top-level-arch.jpeg"/>

### Shared data and domain layers

There are two types of data sources. The network service is for getting RSS feed updates, while
local storage is for caching the feed, which makes it possible to use the application
offline. [Ktor HTTP Client](https://ktor.io/docs/client.html) is used for making API
requests. [Kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization) is used to
serialize feed data and store it locally
with [MultiplaformSettings](https://github.com/russhwolf/multiplatform-settings). This logic is
organized in the shared module of the `com.github.jetbrains.rssreader.core` package.

### Shared application state

The Redux pattern is used for managing the application state. The simplified Redux architecture is
implemented in the shared module. The `Store` class dispatches the **actions** that can be produced
either by a user or by some async work, and generates the new state. It stores the actual **state**
and facilitates subscription to state updates via Kotlin's `StateFlow`. To provide additional
information about state updates, the `Store` class also produces **effects** that, for example, can
be used to display this information via alerts. This logic is organized in the shared KMM module of
the `com.github.jetbrains.rssreader.app` package.

<img src="https://github.com/ROHAN0011/Kotlin-Multiplatform-Mobile-KMM/blob/c2d5b3f1703727c3405c8e2453f75f7e6c5571f4/arch-details.jpg"/>

### Native UI

The UI layer is fully native and implemented using SwiftUI for iOS, Jetpack Compose for Android,
Compose Multiplatform for Desktop and React.js for web browser.

**On the iOS side,** the `Store` from the KMM library is wrapped into the `ObservableObject` and
implements the state as a `@Published` wrapped property. This publishes changes whenever a
dispatched action produces a new state after being reduced in the shared module. The store is
injected as an `Environment Object` into the root view of the application, and is easily accessible
from anywhere in the application. SwiftUI performs all aspects of diffing on the render pass when
your state changes.

For subscribing to state
updates, [the simple wrapper](https://github.com/Kotlin/kmm-production-sample/blob/master/shared/src/iosMain/kotlin/com/github/jetbrains/rssreader/core/CFlow.kt)
is used. This wrapper allows you to provide a callback that will be called when each new value (the
state in our case) is emitted.

## Multiplatform features used

**✅ Platform-specific API usage.** RSS feeds usually only support the XML format.
The `kotlinx.serialization` library currently doesn't support parsing XML data, but there is no need
to implement your own parser. Instead, platform libraries (`XmlPullParser` for
Android and `NSXMLParser` for iOS) are used. The common `FeedParser` interface
is declared in the `commonMain` source set. Platform implementations are placed in the
corresponding `iOSMain` and `AndroidMain` source sets. They are injected into the
RSSReader class (the KMM module entry point) via the `create` factory method, which is declared in
the [RSSReader class companion object](https://github.com/Kotlin/kmm-production-sample/blob/master/shared/src/androidMain/kotlin/com/github/jetbrains/rssreader/core/RssReader.kt).
