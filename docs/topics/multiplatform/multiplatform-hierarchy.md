[//]: # (title: Hierarchical project structure)

Since Kotlin 1.6.20, every new multiplatform project comes with hierarchical project structure. It means that source
sets form a hierarchy for sharing the common code among several targets. It opens up a variety of opportunities,
including usage of platform-dependent libraries in common source sets and sharing code when creating multiplatform
libraries.

To get default hierarchical project structure in your
projects, [update to the latest release](releases.md#update-to-a-new-release). If you need to stay on the version
previous to 1.6.20, you can still enable this feature manually. For this, add the following to you `gradle.properties`:

```properties
kotlin.mpp.enableGranularSourceSetsMetadata=true
kotlin.native.enableDependencyPropagation=false
```

## For multiplatform project authors

With the new hierarchical project structure support, you can share code among some, but not
all [targets](multiplatform-dsl-reference.md#targets) in a multiplatform project.

You can also use platform-dependent libraries, such as `UIKit`, and `posix` in source sets shared among several native
targets. One popular case is having access to iOS-specific dependencies like `Foundation` when sharing code across all
iOS targets. New structure helps you share more native code without being limited by platform-specific dependencies.

By using the hierarchical structure along with platform-dependent libraries in shared source sets, you can eliminate the
need to use certain workarounds to get IDE support for sharing source sets among several native targets, for
example `iosArm64` and `iosX64`:

```kotlin
kotlin {
    // workaround 1: select iOS target platform depending on the Xcode environment variables
    val iOSTarget: (String, KotlinNativeTarget.() -> Unit) -> KotlinNativeTarget =
        if (System.getenv("SDK_NAME")?.startsWith("iphoneos") == true)
            ::iosArm64
        else
            ::iosX64

    iOSTarget("ios")
}
```

```bash
# workaround 2: make symbolic links to use one source set for two targets
ln -s iosMain iosArm64Main && ln -s iosMain iosX64Main
```

Instead of doing this, you can create a hierarchical structure
with [target shortcuts](multiplatform-share-on-platforms.md#use-target-shortcuts)
available for typical multi-target scenarios, or you can manually declare and connect the source sets. For example, you
can create two iOS targets and a shared source set with the `ios()` shortcut:

```kotlin
kotlin {
    ios() // iOS device and simulator targets; iosMain and iosTest source sets
}
```

The Kotlin toolchain will provide the correct default dependencies and find the API surface area available in the shared
code. This prevents such cases as, for example, the use of a macOS-specific function in the code shared for Windows.

## For library authors

A hierarchical project structure allows reusing code in similar targets, as well as publishing and consuming libraries
with granular APIs targeting similar platforms.

The Kotlin toolchain will automatically figure out the API available in the consumer source set while checking the
unsafe usages, like using an API meant for the JVM in JS code.

* Libraries published with the hierarchical project structure are compatible only with projects that have hierarchical
  project structure. To enable compatibility with non-hierarchical projects, add the following to
  the `gradle.properties` file in your library project:

   ```properties
   kotlin.mpp.enableCompatibilityMetadataVariant=true
   ```

* Libraries published without the hierarchical project structure can’t be used in a shared native source set. For
  example, users with `ios()` shortcuts in their `build.gradle.(kts)` files won’t be able to use your library in their
  iOS-shared code.

See [Compatibility](#compatibility) for more details.

## Compatibility

The compatibility between multiplatform projects and libraries is as follows:

| Library with hierarchical project structure | Project with hierarchical project structure | Compatibility                                            |
|---------------------------------------------|---------------------------------------------|----------------------------------------------------------|
| Yes                                         | Yes                                         | ✅                                                        |
| Yes                                         | No                                          | Need to enable with `enableCompatibilityMetadataVariant` |
| No                                          | Yes                                         | Library can't be used in a shared native source set      |
| No                                          | No                                          | ✅                                                        |

## How to opt-out

To disable hierarchical structure support, set the following option to `false` in gradle.properties:

```properties
kotlin.mpp.hierarchicalStructureSupport=false
```

As for the `kotlin.mpp.enableCompatibilityMetadataVariant` option that enables compatibility of libraries published with
the hierarchical project structure and non-hierarchical projects, you should disable it separately:

```properties
kotlin.mpp.enableCompatibilityMetadataVariant=false
```