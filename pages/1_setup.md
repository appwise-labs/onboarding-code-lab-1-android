author: Wisemen
summary: Wiselab Android 1
id: index
tags:
categories:
environments:
status: Draft

# Wiselab Android 1

## What you'll learn: overview

<img width="200" src="index/img/intro/wisemen_logo_acid.png" />

Hello fellow developer! Welcome to **Wisemen**, we are happy to have you on board! We want to make sure you are
soon up to speed with the core of Android development and our way of working.
Therefore we created this awesome series of **CodeLabs** for you. These are a constant work in progress, so please
let us know if you have any feedback or suggestions!
This is what you will learn in this CodeLab:

### 1. Creating a new project with our Android Core Library

We have our own Android Core Library that we use in all our projects. This library contains a lot of useful classes
and functions that we use in all our projects.

### 2. Folder structure

We have a specific folder structure that we use in all our projects. This is to make sure that we can easily find
all the files we need.

### 3. AGILE

1. Work with **Jira**
2. Using **Bitbucket** for version control
3. Branching strategy
4. **Pull Requests**

### 4. Creating your first app

You will be creating a simple To-Do app. This app will have a few features that are written out in Jira.

We are currently in a transition phase from **XML** to **Jetpack Compose**. Therefore we will be using **Jetpack
Compose** for this app.
Jetpack Compose is a new way of writing UI for Android apps in a more declarative way. You can find more
information about Jetpack Compose [here](https://developer.android.com/jetpack/compose).
Our Legacy projects are still written in XML, but as time comes you'll get to learn that as well! Our new projects
are written in Jetpack Compose, therefore we will be using that for this Wiselab.

### Let's get started!

![](index/img/intro/android-hello.gif)

## What you need: Prerequisites

Before we start, make sure you have everything you need to complete this **WiseLab**. You will need:

### Android Studio

Download the IDE here if you don't have it already!
[Android Studio](https://developer.android.com/studio)

### Figma

Our designers work with **Figma**. You can download it here:

* [Figma](https://www.figma.com/downloads/)

To access the designs you need to log in with your Wisemen account:

* [Figma wireframes](https://www.figma.com/file/hebgv4Qx8VanMAQkO1NFpa/Onboarding-to-do?node-id=407-4095&t=2qdyy89lKwN7dFw3-0)

### GitHub Repository

* You can find the link to your repo on your [personal Confluence](https://appwise.atlassian.net/wiki/home) page.

### Jira access

*ToDo: Add link to Jira*

## Create the project

### 3.1 Create a new project

Open Android Studio and create a new Compose project. You can name it Wiselab_Android_<<Name>>.

![](index/img/setup/new_compose_project.png)

Make sure your package name is: **com.wiselab.wiselab_android_<<name>>**

### 3.2 Link your project to GitHub

You can find the link to your repo on your [personal Confluence](https://appwise.atlassian.net/wiki/home) page.

![](index/img/setup/bitbucket_clone.png)

#### Use the terminal or IDE to link your project to BitBucket

We recommend to use a GIT GUI like [SourceTree](https://www.sourcetreeapp.com/) or [Fork](https://git-fork.com/).
As backup we will show you how to work with the ***terminal***. If you prefer to work with the ***IDE***, you can
find the
instructions [here](https://www.jetbrains.com/help/idea/set-up-a-git-repository.html#add-remote).

* Open the terminal in Android Studio (bottom left corner)
* add remote origin to project
* ```shell
  git init
  git remote add origin <your repo url>
  ```

From here on you can choose to use the terminal or the IDE to work with Git.

### 3.3 Our Branching strategy

We use the **Gitflow** branching strategy. You can find more information about this
strategy [here](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow).

* **develop** branch: this branch contains the latest development changes
* **QA** branch: this branch is used to test the features before they are merged into the staging branch
* **staging** branch: this branch is used send to the client for testing
* **master** branch: this branch contains the latest release of the app

* **feature/...** branches: these branches are used to develop new features for the upcoming release and are only
  pushed to develop
* **bugfix/...** branches: these branches are used to fix bugs in the app and are only pushed to develop

Make sure you have created all 4 'standard' branches. You can do this by either using the terminal, IDE or a GUI
tool like [SourceTree](https://www.sourcetreeapp.com/) or [Fork](https://git-fork.com/).

Now checkout the **develop** branch and create a new feature branch called **feature/setup-project**.

## Add the Android Core Library

Now that you have created your project, it's time to add
our [Android Core Library](https://github.com/appwise-labs/AndroidCore)! This library contains a lot of useful
classes and functions that we use in all our projects. The latest version is also noted in the readme.

### 4.1 Open the **build.gradle** file of your **project**

* Add the following **plugins** to the top of the file:
* Be sure to check for the latest versions on the links above the plugins!
* This is needed to use the Room dependency for our local database

```gradle.kts
plugins {
    // https://mvnrepository.com/artifact/com.android.application/com.android.application.gradle.plugin?repo=google
    id("com.android.application") version "<<version>>" apply false
    
    // https://plugins.gradle.org/plugin/org.jetbrains.kotlin.android
    id("org.jetbrains.kotlin.android") version "<<version>>" apply false
    
    //https://github.com/google/ksp/releases
    id("com.google.devtools.ksp") version "<<version>>" apply false
}
```

### 4.2 Open the **settings.gradle** file of your **project**

* Under the **repositories** tag in the **dependencyResolutionManagement** block, add the following:

```gradle.kts
repositories {
    google()
    mavenCentral()
    
    maven("maven.google.com")
    maven("https://maven.fabric.io/public")
    maven("https://jitpack.io")
    maven("https://plugins.gradle.org/m2/")
}
```

### 4.3 Open the **build.gradle** file of your **app**

* Make sure you have following **plugins**

```gradle.kts
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("com.google.devtools.ksp")
}
```

* Add the following **dependencies**. The versions can be found in the url above the dependencies.

```gradle
    //Appwise core dependencies
    //https://jitpack.io/#appwise-labs/AndroidCore
    implementation("com.github.appwise-labs.AndroidCore:core:<<version>>")
    implementation("com.github.appwise-labs.AndroidCore:room:<<version>>")
    implementation("com.github.appwise-labs.AndroidCore:networking:<<version>>")

    //Room
    //https://developer.android.com/jetpack/androidx/releases/room
    implementation("androidx.room:room-runtime:<<version>>")
    ksp("androidx.room:room-compiler:<<version>>")
    
    //Koin for dependency injection
    //https://insert-koin.io/docs/setup/koin/
    implementation("io.insert-koin:koin-core:<<version>>")
    implementation("io.insert-koin:koin-android:<<version>>")
    implementation("io.insert-koin:koin-androidx-compose:<<version>>")
    
    //Compose navigation
    //https://github.com/raamcosta/compose-destinations
    implementation("io.github.raamcosta.compose-destinations:animations-core:<<version>>")
    ksp("io.github.raamcosta.compose-destinations:ksp:<<version>>")
```

Now you can sync your project and you're ready to go! 🚀
If you have any more questions about the Android Core Library, you can find more information on the link above.

Don't forget to push these changes to your **feature/setup-project** branch with a clear commit message.

## Setup Project

It's time to set some last few things before we can start coding!

In case you
forgot: [Figma wireframes](https://www.figma.com/file/hebgv4Qx8VanMAQkO1NFpa/Onboarding-to-do?node-id=407-4095&t=2qdyy89lKwN7dFw3-0)

### 5.1 App class

Let's start by creating the **App** class. This is the first class that is called when the app starts. We will use
this class to set up our dependency injection and navigation in the future.
Create a new class in the main package called **App** and add the following code:

```kotlin

class App : Application() {

    companion object {
        lateinit var instance: App
            private set

        fun isProxymanEnabled() = BuildConfig.DEBUG && BuildConfig.BUILD_TYPE != "release"
    }

    override fun onCreate() {
        super.onCreate()

        instance = this
    }
}
```

Let's explain this bit of code. The **App** class extends the **Application** class. This is the first class that
is called when the app starts. We use the **companion object** to create a singleton of the **App** class. This way
we can access the **App** class from anywhere in the app. We also use this class to check if we are in debug mode
and if we are not in release mode. This is used to enable the Proxyman interceptor for debugging purposes. The
instance of the **App** class is set in the **onCreate** function.

#### 5.1.1 Init Core

* Create a new private function called initCore to set up the Android Core Library. This function will be called in
  the **onCreate** function.
* Add the following code to the **initCore** function:

```kotlin
CoreApp.init(this)
    .apply {
        if (BuildConfig.DEBUG) {
            initializeErrorActivity(true)
        }
    }
    .initializeLogger(getString(R.string.app_name), BuildConfig.DEBUG)
    .build()
```

This code initializes the Android Core Library. We also initialize the error activity and the logger. The logger is
used to log messages to the console. The error activity is used to show errors in the app. This is only enabled in
debug mode.

* Don't forget to add the **initCore** function to the **onCreate** function.

#### 11.1.2 Init Koin
[Koin](https://github.com/InsertKoinIO/koin) is our [dependency injection](https://developer.android.com/training/dependency-injection) library.
We use this library to inject our viewmodels and repositories in our composables. This way we can easily test our code.

Create a new package called **com.wiselab.<<name>>.data.di**. This is where we will put all our dependency injection related classes.

* Create a new file called **AppModule** with following code:
* We will add to this in the future when we need to inject more dependencies.

```kotlin
val appModule = module { }
```
Here we create a module that will be used to inject our dependencies. We create a single instance of our database here.

* In the **App** class: create a new private function called initKoin to set up Koin. This function will be called in the **onCreate** function.
* Add the following code to the **initKoin** function:

```kotlin
startKoin {
  androidLogger(Level.DEBUG)
  //this is so when you want to inject app context this determines where to get it
  androidContext(this@App)
  modules(appModule)
}
```
Here we initialize Koin and add the appModule we created earlier.

### 5.2 Theme

* Go to the figma file under the colors tab and place the colors in the **Color.kt** file in the theme package. Place these colors in an **Object**.
* Change the colors of the **LightColorScheme** in the theme.kt file, delete the **DarkColorScheme** because we are
  not going to implement a dark theme.
* Check the [Material3](https://m3.material.io/styles/color/the-color-system/key-colors-tones) guidelines for the
  key colors and Color roles.
* Change the **Theme**-function to the most basic version like this:

 ```kotlin
    @Composable
fun AppTheme(
    content: @Composable () -> Unit
) {
    val colors = LightColorScheme

    MaterialTheme(
        colorScheme = colors,
        content = content
    )
}
```

### 5.3 TextStyles

* Add an **object**-file named **TextStyles** to preset our different text styles. 🔤
* Use the [Material3](https://m3.material.io/styles/typography/applying-type) guidelines for the different text
  styles.
* Create the following styles in this file:
    * HeadlineLarge
    * HeadlineMedium
    * HeadlineSmall
    * Subtitle
    * LabelLarge
    * LabelSmall
    * BodyLarge
    * BodySmall
    * Button

* Let's create the first style together: Look at the figma design for the largest headline: "Mijn to do's". Click
  on it until it is the only component selected. You can see the text style in the right panel. Use these values to
  create the **HeadlineLarge** style:

```kotlin

object TextStyles {

    val headlineLarge = TextStyle(
        fontSize = 34.sp,
        fontWeight = FontWeight.W700,
        lineHeight = 41.sp,
        letterSpacing = 0.37.sp
    )
}
```

* Note that we use the **Sp** unit for the font size, line height and letter spacing. This is because we want to
  use the **Sp** unit for all our text styles. This way the text will scale with the system font size. Don't mind
  the fontFamily, SF Pro Display is the default font on iOS. The color is not needed because we will set the color
  in the theme.

### 5.4 Spacing

* Add an **object**-file named **Spacing** to preset our different spacing values. 📏
* Use the [Material3](https://m3.material.io/layout/spacing) guidelines for the different spacing values.
* Use this as a reference for the different spacing values:

```kotlin
package com.wiselab.onboarding_compose.ui.theme

import androidx.compose.material3.MaterialTheme
import androidx.compose.runtime.Composable
import androidx.compose.runtime.ReadOnlyComposable
import androidx.compose.runtime.compositionLocalOf
import androidx.compose.ui.unit.Dp
import androidx.compose.ui.unit.dp

object Spacing {
    val default: Dp = 0.dp
    val extraSmall: Dp = 4.dp
    val small: Dp = 8.dp
    val medium: Dp = 16.dp
    val large: Dp = 24.dp
    val extraLarge: Dp = 40.dp
    val huge: Dp = 64.dp
}
```

### 5.5 Check your first preview

* Go to the **MainActivity.kt** file and check for errors.
* Change the previous theme name to the new **AppTheme** name and clear the imports

### 5.5.1 Add a virtual or physical device

If you don't have a virtual or physical device, you can create one by following these steps: 📱

**Physical device**

* Go to your phone settings and enable **Developer options**. If you don't know how to do this, you can find more
  information [here](https://developer.android.com/studio/debug/dev-options).
* Connect you phone to your computer with either USB or Wireless debugging. You can find more
  information [here](https://developer.android.com/tools/adb#connect-to-a-device-over-wi-fi).

**Virtual device**

* Go to the **AVD Manager** in Android Studio and create a new virtual device. You can find more
  information [here](https://developer.android.com/studio/run/managing-avds).

### 5.3.2 Run the preview

In the **MainActivity.kt** file, click on the **Preview** button next to the **GreetingPreview** function. This
will open a preview of your app in the IDE. If you have a virtual or physical device connected, you can also run
the app on your device.

![](index/img/setup/preview_example.png)

Now you can push your changes to your **feature/setup-project** branch with a clear commit message and create a
pull request to merge your branch into the **develop** branch. Don't forget to add your buddy as a reviewer! 🕵️
