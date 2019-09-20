---
title: "Add offline support using Room"
date: 2017-11-23T07:20:04+02:00
---
{{< imgpreview src="/images/offline_support/arch-components.png" caption="developer.android.com" >}}

Recently, Google has announced the stable version of the [architecture components](https://developer.android.com/topic/libraries/architecture/index.html), which arguably were quite stable before getting to 1.0.0. This library contains 4 parts: [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel.html), [LiveData](https://developer.android.com/topic/libraries/architecture/livedata.html), [Room](https://developer.android.com/topic/libraries/architecture/room.html), and the [Paging Library](https://developer.android.com/topic/libraries/architecture/paging.html). This parts work great together because as one may assume — they are designed to do so. The architecture components could drastically influence and change the traditional way of developing Android apps, so they are potential game changer. This article is focused on the Room part of the architecture components, used for data persistence in order to enable the offline support of an app.

## Intro
Room is a library that simplifies the database manipulation by using annotation processing. The purpose of this article is to describe the way Room is used to add an offline support to an [existing project](https://github.com/mitrejcevski/room_example). It’s a very basic example of fetching a profile, and there are 2 implementations of the `ProfileDataSource` interface — one using `RxJava` and another one using the Kotlin’s `Coroutines`. There are those 2 implementations just to demonstrate that the offline support can be added in either case.

## Data
To begin with, the implementation just makes a call to remote to fetch a profile, and displays the results into a `TextView`. Let’s take a look at the structure.

##### The data source interface
{{< gist mitrejcevski 67754769fb27d298e11603040d5432bf >}}

##### The implementation using RxJava
{{< gist mitrejcevski ed7866833197b5e23ed396ff7e6daf12 >}}

An interesting thing to put attention on in this implementation is the way the data is set to the `LiveData` object — by using the `postValue()` method.

##### The implementation using Kotlin coroutnes
{{< gist mitrejcevski 8fd53e9e7d7dd2e0a19fc1d57982fbd7 >}}

Similarly, when using coroutines, the data is set by using the `postValue()` method. The reason behind is the fact that the live data value is being set from a background thread. `LiveData` does not allow the `setValue()` method to be called by background thread.

## ViewModel
The next part is the `ViewModel` that uses the `ProfileDataSource` and here is what it looks like:
{{< gist mitrejcevski 6a137cc6b4aff161f6808b4b23c6a9d1 >}}

The `ViewModel` injects the data source that is provided by `Dagger` (it could be either the one using `RxJava`, or the other one), and it exposes a function to fetch a profile, and the resulting `LiveData` object. Let’s check the different parts of it, and how it works:

 - The `inputLiveData` is being updated each time the `fetchProfile()` function is called with new value.

 - The `profileResult` is initialised by use of the `switchMap` transformation, which basically converts one type of `LiveData` into another.

 - The `profile` function just exposes the `profileResult` live data so it can be observed for updates.

Calling the `fetchProfile()` function will update the `inputLiveData` which in turn will cause a call to the `dataSource` to fetch the profile. The `profileResult` variable is being initialised right away, because the `dataSource` does return a `LiveData` object instantly. Once the background job is done, only it’s value is being updated and it will be delivered to the active observers of the `LiveData` object.

## Presentation
Finally, here is how the `Activity` that makes use of that `ViewModel` is implemented:

{{< gist mitrejcevski 32535c2e5df8f6134613925ee4c51f1b >}}

It just makes call to the `fetchProfile()` function of the `ViewModel` and observes for the updates. Once an update arrives, it will display it into the `TextView`.

## Adding offline support
Next, let’s go ahead and add an offline support to this solution that by now should be clear what it is all about. First, the `Profile` data class has to be changed by adding an entity annotation:

{{< gist mitrejcevski e97859d3bd65322bb6bdf8f982867784 >}}

By adding the `@Entity` annotation, this class is capable to be used by Room. Room will use the class name to create a table in the database from it. If a different name is desired, the annotation has a `tableName` method that allows doing so (`@Entity(tableName = “nameForTable”)`). Also, Room requires at least one field annotated with the `@PrimaryKey` annotation.

## DAO
The `DAO` (Data Access Object) has to be defined together with the `Entity` object in order for Room to be able to perform the database operations. The DAO should define the desired operations.

{{< gist mitrejcevski cc71fba975e5106f4e6439a55b64967b >}}

## Database
Eventually, the database is defined by adding an abstract class that extends from the `RoomDatabase` class, and adding a `@Database` annotation. In this case there is one entity annotated class (`Profile`) that should be put in the entities array. The class has also defined an abstract function that exposes the `ProfileDao` interface defined in the previous step.

{{< gist mitrejcevski df1ff40bf2e724850b44a924f60839a9 >}}

## Providing database instance
Once this all things are being defined, the project has to be re-build, so an actual implementations of the annotated classes is going to be generated. Afterwards, an instance from the database can be created. In this case, there is a singleton database instance provided by using `Dagger`

{{< gist mitrejcevski 34d053dd88c7d027dc97cc42317e2976 >}}

That should conclude the adding database phase, and at this point the database is ready to be used.

## The offline support
The concept behind adding offline support is to load the data that exists in the database, and make a network call that will update the value in the database. It means that the `LiveData` that is going to be observed by the observers is being provided by the database. Then, the update of the database is going to be propagated to the observers of the `LiveData` provided earlier. It is fairly simple. Let’s go ahead and take a look at the changes needed to be done:

{{< gist mitrejcevski 87b2b58afba9cf96144febd710d1aa50 >}}

The first change done in the data source that uses `RxJava` is the `AppDatabase` instance in the constructor. Then, the `fetchProfile()` function returns the live data provided by the database (the DAO), with applied transformation. The transformation had to be applied here in order to keep the `ViewModel` same and not change anything in there. So, the transformation maps the database’s provided `LiveData<Profile>` into `LiveData<ProfileResponse>`. Similarly, the data source implementation based on coroutines has to be changed in the same fashion:

{{< gist mitrejcevski 1b73472e404c307d5dd0c5264f7e85c6 >}}

By doing this, the app has an offline support. The way it is going to perform could be described with the following flow:

 - The `Activity` makes call to the `ViewModel` and observes the resulting `LiveData` that it exposes.

 - The `ViewModel` triggers call to the `DataSource` function that return a `LiveData` provided from the database.

 - Because this flow attached an observer to the `LiveData` provided from the database, a query to the database is triggered to find a `Profile` with the `profileId` provided in the function argument.

 - Then, if there is such a record, the database will set the value to the `LiveData` it returned before with that record, or with null otherwise. That’s why in the map function, there is a null check `Transformations.map(source) { ProfileResponse.Success(it ?: Profile())}` that will instead of null apply an empty `Profile`.

 - At the time when the `fetchProfile()` function of the `DataSource` is called, both implementations are making a call to the server, and they insert the response in the database. Once inserted, the `LiveData` provided by the database is automatically updated, and the observers will get the new value.

## Error handling
One thing to be noticed after making this changes is the fact that the error handling that was there before is lost. In case there is no record in the database with the asked `profileId` an empty `Profile` object is delivered to the `LiveData` observers (UI), and if an error happens while executing the network call, the UI is not going to be notified which is a bad thing. Fortunately, there are ways to solve this, and one of them is by using the `MediatorLiveData`.

## MediatorLiveData for the rescue
The mediator live data can be used as a merger for 2 (or more) `LiveData` sources. At this point, there could be 2 sources considered: the network response and the database. Again, the changes should be applied only in the data source implementation:

{{< gist mitrejcevski a99a1507cf1265bbe5c548e7e7b0db5b >}}

Or if the implementation of the data source is the one using the coroutines, the resulting outcome would be like this:

{{< gist mitrejcevski 91c404a6839b34e17d8a5d30657ceca3 >}}

Arguably, this is a quite naive and very simple example, but it should serve it’s purpose to explain the basic concept of adding offline support. Since it is quite flexible, there could be various different ways to change this implementation, and one great point to add is that it could be nicely and easily unit tested.

Adding offline support is not always required, especially when the app is targeted for markets with great internet coverage. However, there are lots of countries worldwide with a quite poor not only coverage but internet speed as well, so adding an offline support could drastically improve the usability and the engagement with the users.
