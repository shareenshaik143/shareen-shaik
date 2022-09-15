# shareen-shaik
# LocationFetcher

[![Download](https://img.shields.io/maven-central/v/app.freel/locationfetcher)](https://search.maven.org/artifact/app.freel/locationfetcher)

> :warning: If you're using Jetpack Compose, see [this README](locationfetcher-compose/README.md) instead.

Simple location fetcher for Android Apps built with Kotlin and Coroutines.

Building location-aware Android apps can be a bit tricky. This library makes it as simple as:

```kotlin
class MyActivity : ComponentActivity() {

    private val locationFetcher = locationFetcher({ getString(R.string.location_rationale) }) {
        interval = 5.seconds
        priority = LocationRequest.PRIORITY_HIGH_ACCURACY
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        locationFetcher.location
            .onEach { errorsOrLocation ->
                errorsOrLocation.tap { location ->
                    // Got location
                }.tapLeft { errors ->
                    // Handle errors (no permission/location settings disabled).
                    // Note that this library will automatically try to resolve errors.
                }
            }
            .flowWithLifecycle(Lifecycle.State.STARTED)
            .launchIn(this)
    }
}
```

This library provides a simple location component, `LocationFetcher`, requiring only an instance of either `ComponentActivity`, `Fragment` or `Context`, to make your Android app location-aware.

If the device's location services are disabled, or if your app is not allowed location permissions by the user, this library will automatically ask the user to enable location services in settings or to allow the necessary permissions as soon as you start collecting the `LocationFetcher.location` flow.


## Installation with Gradle

### Setup Maven Central on project-level build.gradle

This library is hosted in Maven Central, so you must set it up for your project before adding the module-level dependency.

The new way to install dependencies repositories is through the `dependencyResolutionManagement` DSL in `settings.gradle(.kts)`.

Kotlin or Groovy:
```kotlin
dependencyResolutionManagement {
    repositories {
        mavenCentral()
    }
   }


### Add dependency

On app-level `build.gradle`, add dependency:

```kotlin
dependencies {
  implementation("app.freel:locationfetcher:9.0.0")
}
```

## Usage

### Instantiating

On any `ComponentActivity`, `Fragment`, or `Context` class, you can instantiate a `LocationFetcher` by calling the extension functions on `ComponentActivity`, `Fragmnet`, or `Context`:

```kotlin
locationFetcher({ getString(R.string.location_rationale) }) {
    // configuration block
}
```

Alternatively, there are some `LocationFetcher()` method overloads. You can see all public builders [in here](https://github.com/psteiger/LocationFetcher/blob/master/locationfetcher/src/main/java/com/freelapp/libs/locationfetcher/Builders.kt).

If `LocationFetcher` is created with a `ComponentActivity` or `Fragment`, it will be able to show dialogs to request the user to enable permission in Android settings and to allow the app to obtain the device's location. If `LocationFetcher` is created with a non-UI `Context`, it won't be able to show dialogs.

### Options

`LocationFetcher` supports the following configurations for location fetching when creating the component:

(Note: for GPS and Network providers, only `interval` and `smallestDisplacement` are used. If you want to use all options, limit providers to `LocationRequest.Provider.Fused`, which is the default)

```kotlin
locationFetcher("We need your permission to use your location for showing nearby items") {
    fastestInterval = 5.seconds
    interval = 15.seconds
    maxWaitTime = 2.minutes
    priority = LocationRequest.PRIORITY_HIGH_ACCURACY
    smallestDisplacement = 50f
    isWaitForAccurateLocation = false
    providers = listOf(
        LocationRequest.Provider.GPS,
        LocationRequest.Provider.Network, 
        LocationRequest.Provider.Fused
    )
    numUpdates = Int.MAX_VALUE
    debug = false
}
```

Alternatively, you might prefer to create a standalone configuration instance. It is useful, for example, when sharing a common configuration between multiple `LocationFetcher` instances:

```kotlin
val config = LocationFetcher.Config(
    rationale = "We need your permission to use your location for showing nearby items",
    fastestInterval = 5.seconds,
    interval = 15.seconds,
    maxWaitTime = 2.minutes,
    priority = LocationRequest.PRIORITY_HIGH_ACCURACY,
    smallestDisplacement = 50f,
    isWaitForAccurateLocation = false,
    providers = listOf(
        LocationRequest.Provider.GPS,
        LocationRequest.Provider.Network,
        LocationRequest.Provider.Fused
    ),
    numUpdates = Int.MAX_VALUE,
    debug = true
)
val locationFetcher = locationFetcher(config)
```
