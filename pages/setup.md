author: Wisemen
summary: Wiselab Android 1
id: index
tags:
categories:
environments:
status: Draft

# Wiselab Android 1

## What you'll learn: overview

Duration: 1:00:00

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

### 4. Creating a first screen

* How to use **Figma**
* Write some code!
* Write some more code!
* Create your first **PR** (pull request)

### Let's get started!

![](index/img/intro/android-hello.gif)

## What you need: Prerequisites

Duration: 0:55:00

Before we start, make sure you have everything you need to complete this **WiseLab**. You will need:

### Android Studio

Download the IDE here if you don't have it already!
[Android Studio](https://developer.android.com/studio)

### Figma

Our designers work with **Figma**. You can download it here:

* [Figma](https://www.figma.com/downloads/)

To access the designs you need to log in with your Wisemen account:

* [Figma wireframes](https://www.figma.com/file/hebgv4Qx8VanMAQkO1NFpa/Onboarding-to-do?node-id=407-4095&t=2qdyy89lKwN7dFw3-0)

### BitBucket repository

*ToDo: Add link to BitBucket repository*

### Jira access

*ToDo: Add link to Jira*

## Create the project

Duration: 0:50:00

### 3.1 Create a new project

Open Android Studio and create a new Compose project. You can name it Wiselab_Android_YOURNAME.

![](index/img/setup/new_compose_project.png)

### 3.2 Link your project to BitBucket

You can find the link to your repo on your [personal Confluence](https://appwise.atlassian.net/wiki/home) page.

![](index/img/setup/bitbucket_clone.png)

#### Use the terminal or IDE to link your project to BitBucket

We will show you how to work with the ***terminal***. If you prefer to work with the ***IDE***, you can find the
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

* **master** branch: this branch contains the latest release of the app
* **staging** branch: this branch is used send to the client for testing
* **QA** branch: this branch is used to test the features before they are merged into the staging branch
* **develop** branch: this branch contains the latest development changes
* **feature/...** branches: these branches are used to develop new features for the upcoming release
* **bugfix/...** branches: these branches are used to fix bugs in the app

Make sure you have created all 4 'standard' branches. You can do this by either using the terminal, IDE or a GUI
tool like [SourceTree](https://www.sourcetreeapp.com/) or [Fork](https://git-fork.com/).

Now checkout the **develop** branch and create a new feature branch called **feature/setup-project**.

## Add the Android Core Library

Duration: 0:40:00

Now that you have created your project, it's time to add
our [Android Core Library](https://github.com/appwise-labs/AndroidCore)! This library contains a lot of useful
classes and functions that we use in all our projects. The latest version is also noted in the readme.

### 4.1 Open the **build.gradle** file of your **project**

* Add the following **plugin** to the top of the file:
* Be sure to check for the latest versions!
* This is needed to use the Room dependency for our local database

```gradle.kts
plugins {
    id("com.android.application") version "8.1.0" apply false
    id("org.jetbrains.kotlin.android") version "1.8.10" apply false
    id("com.google.devtools.ksp") version "1.8.10-1.0.9" apply false
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

* Add the following **dependencies**

```gradle
    //Appwise core dependencies
    implementation("com.github.appwise-labs.AndroidCore:core:1.4.3")
    implementation("com.github.appwise-labs.AndroidCore:room:1.4.3")
    implementation("com.github.appwise-labs.AndroidCore:networking:1.4.3")

    //Room
    implementation("androidx.room:room-runtime:2.5.2")
    ksp("androidx.room:room-compiler:2.5.2")
    
    //Koin for dependency injection
    implementation("io.insert-koin:koin-core:3.4.0")
    implementation("io.insert-koin:koin-android:3.4.0")
    implementation("io.insert-koin:koin-androidx-compose:3.4.0")
    
    //Compose navigation
    implementation("io.github.raamcosta.compose-destinations:animations-core:1.8.38-beta")
    ksp("io.github.raamcosta.compose-destinations:ksp:1.8.42-beta")
```

Now you can sync your project and you're ready to go! 🚀
If you have any more questions about the Android Core Library, you can find more information on the link above.

Don't forget to push these changes to your **feature/setup-project** branch with a clear commit message.

## Setup Project

It's time to set some last few things before we can start coding!

### 5.1 Theme

* Go to the figma file under the colors tab and place the colors in the **Color.kt** file under the theme package.
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

### 5.2 TextStyles

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

### 5.3 Spacing

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
// this is a composition local --> you can call it for all composables without the need to pass it all down
val LocalSpacing = compositionLocalOf { Spacing }

val MaterialTheme.spacing: Spacing
    @Composable
    @ReadOnlyComposable
    get() = LocalSpacing.current
```

### 5.3 Check your first preview

* Go to the **MainActivity.kt** file and check for errors.
* Change the previous theme name to the new **AppTheme** name and clear the imports

### 5.3.1 Add a virtual or physical device

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