## Create your first components and screen

Start creating some UI components and a screen to show them on. Starting with
the [landing screen](https://www.figma.com/file/hebgv4Qx8VanMAQkO1NFpa/Onboarding-to-do?type=design&node-id=455-5009&mode=design&t=a3AEuYp0dvS3MRfT-4)
where we can register or log in. We always name our branches to the Jira ticket. The first ticket we will work on
is **OB-1**. Create a new branch called `feature/ob-1--landing-screen`

* Drag the Jira ticket to the **In Development** column.

### 7.1 Create some buttons

* Open the figma file and check the **components used** page. This is where you can find all
  the [components used](https://www.figma.com/file/hebgv4Qx8VanMAQkO1NFpa/Onboarding-to-do?type=design&node-id=877-1328&mode=dev) in the design.
* Create a new package called **shared**. The full path should be  **com.wiselab.<<name>>.ui.shared**.
* Create a new Kotlin file named **Buttons** in the **Shared** package to create our different buttons
* Create the following buttons:
    * PrimaryButton
    * SecondaryButton
    * TertiaryButton
    * OutlinedButton
* Show a preview of the buttons like this:

<img width="400" src="index/img/setup/button_preview.png" />

Note that you can see the preview when clicking on the <img width="50" src="index/img/buttonsAndScreens/ide_preview_button.png" /> button in the top
right corner of the IDE.

* Here's a bit of code to get you going:

```kotlin
@Preview
@Composable
fun ButtonsPreview() {
    AppTheme {
        Column(
            modifier = Modifier
                .fillMaxWidth()
                .background(MaterialTheme.colorScheme.background)
                .padding(Spacing.medium),
            verticalArrangement = Arrangement.spacedBy(Spacing.medium)
        ) {
            Buttons.Primary(
                text = "Primary",
                Modifier.fillMaxWidth()
            ) {}
        }
    }
}

object Buttons {

    @Composable
    fun Primary(
        text: String,
        modifier: Modifier = Modifier,
        onClick: () -> Unit
    ) {
        Button(
            modifier = modifier,
            shape = RoundedCornerShape(5.dp),
            colors = ButtonDefaults.buttonColors(
                containerColor = Color.PeriWinkel,
                contentColor = Color.Black
            ),
            onClick = onClick
        ) {
            Text(text = text)
        }
    }
}
```

* Let's explain what's going on here:

first off we create the object Buttons. This is a container for all our buttons. We do this so we can easily find
all our buttons in the project.
Then we create a composable function for each button. This way we can easily reuse the buttons in our project.
The Primary Button has 3 parameters:

* text: String
* modifier: Modifier
* onClick: () -> Unit

The text and onClick parameters speak for themselves. The modifier is a bit more complicated. It's a way to
add styling to our button like positioning, padding etc..., but more about this later.

* Next the **preview** is created, we always place these above the object:

Make sure to check the modifiers we use here. The `fillMaxWidth` modifier makes sure the button takes up the full
width of the screen. The `background` modifier makes sure the background color is the same as the background color
of the app. The `padding` modifier makes sure there is some space between the buttons. The `verticalArrangement`
makes sure the buttons are spaced out vertically.

* Now create the other buttons and show them in the preview.
* Hint for the disabled state: Check the opacity/alpha value in Figma and use ``Color.SanJuan.copy(alpha = ...)``.
* Commits linked to Jira tickets should always contain the ticket number. In this case **OB-1**. The commit message
  should look something like this: **OB-1: Create button components**.

### 7.2 Create a screen

The task manager is seen as 2 Epics, onboarding and todo's. create a new package **com.wiselab.<<name>>.feature**.
Also create a new package called **onboarding** in the **feature** package. The full path should be **com.wiselab.<<name>>.feature.onboarding**.
In this package create a new package called **landing** for the landing screen. The full path should be **com.wiselab.<<name>>.feature.onboarding.landing**.

* Create a new Kotlin file called **LandingScreen** in the **landing** package.
* We like to split up our screens in minimal 3 parts, we go in more detail about this in a future section. For now
    * **Layout**: this is where we place all the content of the screen
    * **Screen**: this is where we link our data and state to the layout
    * **Preview**: this is where we show the layout in a preview
* This is also the order we create them in. Note that this is **not** the order in the file. We **always place the
  preview above the screen and the screen above the layout**.

* This is the preview we are working towards:

<img width="400" src="index/img/buttonsAndScreens/landing_preview.png" />

#### 7.2.1 Create the Layout

* Create a composable function called **LandingLayout**. This is where we place all the content of the screen.
* Because there are 2 buttons vertically aligned we will be using the **Column** Composable as previously explained.
* The function will look something like this:

```kotlin
@Composable
fun LandingLayout() {
    Scaffold {
        Column(
            modifier = Modifier
                .fillMaxWidth()
                .wrapContentHeight()
                .padding(horizontal = Spacing.medium)
        ) {
            Buttons.Primary(text = "Create an account", Modifier.fillMaxWidth()) {}
            Buttons.Secondary(
                text = "Log in",
                Modifier
                    .fillMaxWidth()
                    .padding(bottom = 58.dp)
            ) {}
        }
    }
}
```

We use the **fillMaxWidth** modifier to make sure the buttons take up the full width of the screen.
We use the **wrapContentHeight** modifier to make sure the column is only as high as it's content.
We use the **padding** modifier to add some space between the buttons and the edge of the screen. and padding under the secondary button.

Some explanation about **Scaffold** : The **Scaffold** is a Composable that implements the basic material design visual layout.
It has a top bar, bottom bar and a floating action button. We will be using this a lot in our projects. It is like a structured container for our
screen.
At This moment the Scaffold is still experimental, that's why we need to add the `@OptIn(ExperimentalMaterial3Api::class)`

#### 7.2.2 Scaffold Padding

* There is still an issue. The Scaffold must always pass a padding value to it's content. This is because the Scaffold
  has a top bar, bottom bar and a floating action button. We need to make sure the content is not hidden behind these.
* We can fix it like this:

```kotlin 
@Composable
fun LandingLayout() {
    Scaffold { contentPadding ->
        Column(
            modifier = Modifier
                .fillMaxWidth()
                .wrapContentHeight()
                .padding(contentPadding)
                .padding(horizontal = Spacing.medium)
        ) {
            Buttons.Primary(text = "Create an account", Modifier.fillMaxWidth()) {}
            Buttons.Secondary(
                text = "Log in",
                Modifier
                    .fillMaxWidth()
                    .padding(bottom = 58.dp)
            ) {}
        }
    }
}
```

Note that the Scaffold padding is always the first padding that is applied. A modifier is always applied from **top to bottom**.

#### 7.2.3 Create the screen

* Now create the **LandingScreen** composable. This is where we place the layout in a screen, this is the parent.
* Here is where we will link our data to our view. We will explain this in a future section.
* For now we will just call our LandingLayout function.
* The function will look something like this:

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun LandingScreen() {
    LandingLayout()
}

```

#### 7.2.4 Create the preview

* Now create the **LandingScreenPreview** composable. This is where we show the screen in a preview. If you forgot how to create a preview, check
  the **ButtonsPreview**.
* As you will see the buttons are not yet aligned at the bottom. We can fix this by adding a **Spacer** above the buttons like this:

```kotlin
@Composable
fun LandingLayout() {
    Scaffold { contentPadding ->
        Column(
            modifier = Modifier
                .fillMaxWidth()
                .wrapContentHeight()
                .padding(contentPadding)
                .padding(horizontal = Spacing.medium)
        ) {
            Spacer(modifier = Modifier.weight(1f))
            Buttons.Primary(text = "Create an account", Modifier.fillMaxWidth()) {}
            Buttons.Secondary(
                text = "Log in",
                Modifier
                    .fillMaxWidth()
                    .padding(bottom = 58.dp)
            ) {}
        }
    }
}
```

The spacer will take up all the available space and push the buttons to the bottom. The [**
weight** modifier](https://developer.android.com/jetpack/compose/modifiers#weight-in-row-and-column) is used to make sure the spacer takes up all the
available space.

#### 7.2.5 String extraction

* There is still 1 bad practise. Strings should never be hardcoded. We should always use a string resource. We can fix this by adding a string
  resource to the **strings.xml** file.
  The easiest way to do this is to right click on the string and select **Show Context Actions** and then **Extract string resource**. Do this for
  both buttons.
* Use command + click (or windows + click) on the string to navigate to the string resources. Here you can change, add, delete and even translate
  strings.
* Don't forget to commit your changes with a proper commit message!
