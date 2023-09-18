## Navigation

Let's add navigation to our app. We will use
the [raamcosta compose destinations library](https://github.com/raamcosta/compose-destinations) for this. This
library is a wrapper around the navigation component. It makes it easier to use the navigation component with
compose. It also adds some nice features like **animations** and **deep linking**. We already added the dependency
in the setup of the project.

### 8.1 Setup navigation
* Create a new package called **navigation** in the **onboarding** package like this: **com.wiselab.<<name>>.onboarding.navigation**
* Make a new file called **OnboardingNavGraph** in the **navigation** package.
* Add following code to the file:

```kotlin
import com.ramcosta.composedestinations.annotation.NavGraph
import com.ramcosta.composedestinations.annotation.RootNavGraph

@RootNavGraph(start = true)
@NavGraph
annotation class OnboardingNavGraph(
    val start: Boolean = false
)
```

* This is a **NavGraph**. It's a class that contains all the destinations of a navigation graph. The **@RootNavGraph**
  annotation tells the library that this is the root of the navigation graph. The **@NavGraph** annotation tells the
  library that this is a navigation graph. The **start** parameter tells the library that this is the start of the
  navigation graph. This means that this is the first screen that will be shown when the app starts.
* Open the **LandingScreen** file and add the 2 **annotations** to the **LandingScreen** composable function:
  `@Destination` and `@OnboardingNavGraph(start = true)`
* The **@Destination** annotation tells the library that this is a destination. The **@OnboardingNavGraph(start = true)**
  annotation tells the library that this is the start of the navigation graph. This means that this is the first screen
  that will be shown when the app starts.

### 8.2 Integrate in the MainActivity
* Open the **MainActivity** file and remove the **Greeting** composable function with it's preview.
* Change the class so it looks like this:

```kotlin
class MainActivity : ComponentActivity() {
    @OptIn(ExperimentalMaterial3Api::class)
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            AppTheme {
                val navController = rememberNavController()

                Scaffold { paddingValues ->
                    Box(
                        modifier = Modifier
                            .fillMaxSize()
                            .background(MaterialTheme.colorScheme.background)
                            .padding(paddingValues)
                    ) {
                        DestinationsNavHost(
                            navGraph = NavGraphs.onboarding,
                            navController = navController
                        )
                    }
                }
            }
        }
    }
}
```

* We use the **Scaffold** composable to create a basic material design visual layout. We will be using this a lot in our
  projects.
* The Box composable is a container that can contain multiple composable functions. We use it to place the
  **DestinationsNavHost** composable in it.
* The **DestinationsNavHost** composable is a composable function from the destinations library. It's a wrapper around
  the navigation component. It takes a **navGraph** and a **navController** as parameters. The **navGraph** is the
  navigation graph we created in the previous step. The **navController** is a controller that controls the navigation
  between the screens. We will use this later to navigate between screens.

### 8.3 Run your app
In the top right of your IDE you can select the device you want to run your app on. Select a device and run the app
configuration. If you can't find it, try clicking the arrow down to see all configurations like this:

![Run configuration](index/img/buttonsAndScreens/run_app_configuration.png)

You should see the landing screen now! 🙌🏻
Don't forget to commit an push your changes with a proper commit message including the ticket number!

### 8.4 Time for a PR
* Create a pull request from your feature branch to the develop branch.
* Add your buddy as a reviewer.
* Drag your ticket in Jira from **In Development** to **Pull Request**. 🚀
