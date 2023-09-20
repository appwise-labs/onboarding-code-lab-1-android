## MVVM and screen structure

Before we start building our screens, we need to understand how we structure our screens and how we use **MVVM**.
More complex screens that need to fetch data from the backend or database and manipulate it, need to be structured in a way that is easy to understand
and maintain.

### 10.1 MVVM

MVVM stands for **Model-View-ViewModel**. It consists of three parts:

- **Model**: The data model that is used in the screen. For this we use repositories as a data source. More on this later when we start with
  networking.
- **View**: The UI of the screen. This is the Composable function that is used to draw the screen. This you know already.
- **ViewModel**: The logic of the screen. This is the class that holds the logic of the screen. It can be used to fetch data from the backend or
  database, manipulate the data and pass it to the view.

### 10.2 ViewModel

As mentioned above, this is the class that holds the **logic** of the screen.
Since we want to keep the **data manipulation and the UI separate**, we have some rules for the communication between the ViewModel and the View.

- The ViewModel should never know about the View. This means that the ViewModel should not have any references to the View.
- Any data that needs to be manipulated should be managed here, not in the View.
- The data that is passed to the View should be **immutable**. This means that the View should not be able to change the data.
- If the data needs to be changed, it should be done in the ViewModel.

### 10.3 Screen structure

Lets look at the image below to see the whole picture of the communication between the View and the ViewModel.

![](index/img/buttonsAndScreens/mvvm_structure.png)

We know, the whole picture is a little overwhelming. But don't worry, we will go through it step by step.

For naming, keep in mind, all these classes work in tandem to create a screen. So we will name them after the screen they are used in.
For instance if we create a login screen, we will have a LoginScreen, LoginViewModel, LoginUiState, LoginActions and so on.

#### 10.3.1 Screen.kt (View)

Here we have the View layer of our screen. This consist of at least 3 composable functions with their own responsibilities: Screen, Layout and Preview

```kotlin
@Composable
fun Screen(
    navController: NavController,
    viewModel: ViewModel = koinViewModel()
) {
    // Collect data from the ViewModel to populate the layout
    val uiState by viewModel.uiState.collectAsState()

    // Establish links between UI events and the navController
    LaunchedEffect(viewModel) {
        viewModel.eventFlow.collect { event: UiEvent ->
            when (event) {
                is UiEvent.NavigateToNext -> navController.navigate(NextDestination)
                is UiEvent.ShowDialog -> // show dialog
            }
        }
    }

    // Link the uiState and action handler to the layout
    Layout(
        state = uiState,
        onAction = viewModel::onAction,
    )
}
```

The `Screen` function is a crucial part of your UI layer (View) in Jetpack Compose.
It serves as the entry point for rendering your screen's content. Here are the key aspects of this function:

- Parameters
    - `NavController`: The navigation controller used for navigating between screens.
    - `ViewModel`: An instance of the ViewModel associated with this screen. We use the [dependency injection](https://developer.android.com/training/dependency-injection) library Koin to inject the ViewModel.
- Function
    - Collects data from the ViewModel to populate the layout.
    - Establishes links between UI events and the `navController`.
    - Links actions of the ViewModel to the layout components.

```kotlin
@Composable
fun Layout(
    state: UiState,
    onAction: (Action) -> Unit
) {
    // Layout components
}
```

The `Layout` function is responsible for defining the structure of your screen's layout. It plays a central role in rendering the UI. Here's what you
need to know about it:

- Parameters
    - `state: UiState`: Represents the current state of the screen's UI.
    - `onAction: (Action) -> Unit`: A function used to handle UI actions.
- Function
    - Displays the UI components based on the provided `state`.
    - Triggers actions based on user interactions.
    - Also used for preview and testing purposes.

```kotlin
@Preview
@Composable
fun Preview() {
    AppTheme {
        Layout(
            state = UiState(),
            onAction = {}
        )
    }
}
```

The `Preview` function provides a simple, placeholder implementation of the `Layout` with a default state. It's useful for previewing your layout
during development.

#### 10.3.2 ViewModel.kt (ViewModel)

The `ViewModel` class is a critical component in managing the state and behavior of your UI.

```kotlin
class ViewModel(
    private val repository: Repository,
    savedStateHandle: SavedStateHandle
) : ViewModel() {

    // Extracts navigation arguments from the SavedStateHandle to receive data from the previous screen
    private val args = savedStateHandle.navArgs<NavArgs>()

    // The single source of the truth for the UI state
    var state by mutableStateOf(UiState())
        // The setter is private to prevent external modification
        private set

    // A channel for sending events to the UI
    private val eventChannel = Channel<UiEvent>()

    // The channel is exposed as a flow to expose a read-only interface
    val eventFlow = eventChannel.receiveAsFlow()

    // The entry point for UI actions to be handled by the ViewModel
    fun onAction(action: Action) {
        when (action) {
            is Action.OnNextClicked -> eventChannel.trySend(UiEvent.NavigateToNext)
            is Action.OnTextChanged -> state = state.copy(text = action.text)
            is Action.Save -> repository.save(state.text)
        }
    }
}
```

##### **Constructor**

- Repositories are injected into the `ViewModel` via Koin.
- `savedStateHandle`: An instance of `SavedStateHandle` is injected via Koin, allowing for the preservation of UI state during configuration changes.

##### **private val args = savedStateHandle.navArgs<NavArgs>()**

- This code snippet extracts navigation arguments from the `SavedStateHandle` to facilitate data exchange between screens during navigation.

##### **var state by mutableStateOf(UiState())**

- This variable is the single source of truth for the current UI state.
- It is read-only for external sources, ensuring data consistency and reducing bugs.

##### **private val eventChannel = Channel<UiEvent>()**

- An event channel for sending UI events.
- Exposes `eventFlow` to collect navigation events triggered by the ViewModel.

##### **fun onAction(action: Action)**

- Handles UI actions triggered by **user interactions**.
- Updates the UI state as necessary.
- Communicates changes to the Data layer when needed.
- Sends **UI events** through the event channel.

#### 10.3.3 UiState.kt

```kotlin
data class UiState(
    val text: String = "",
    val isLoading: Boolean = false,
    val error: String = "",
)
```

- `UiState` represents the current and immutable state of the screen.
- **Immutability** is crucial for ensuring the **reliability and predictability** of the Compose composition process.
- When you need to update the state, create a new state instance based on the current one rather than modifying the fields directly.

#### 10.3.4 UiEvent.kt

```kotlin
sealed class UiEvent {
    object NavigateToNext : UiEvent()
    data class ShowDialog(val message: String) : UiEvent()
}
```

- `UiEvent` represents various types of events that need to be handled at the View layer.
- These events are often used to trigger navigations, animations, or display dialogs.
- Fired by the ViewModel and handled by the UI.

#### 10.3.5 Action.kt

```kotlin
sealed class Action {
    object OnNextClicked : Action()
    data class OnTextChanged(val text: String) : Action()
    object Save : Action()
}
```

- `Action` represents user-initiated actions from the UI layer.
- These actions are fired by the UI and handled by the ViewModel.

#### 10.3.6 NavArgs.kt

```kotlin
data class NavArgs(
    val id: String = "-1"
)
```

- `NavArgs` is a typed class that holds data passed between screens during navigation.
- This data can be extracted from the `SavedStateHandle` within the ViewModel.

### 10.4 Screen implementation
Time to create the login screen! 🎉

#### 10.4.1 Create the screen
You should be able to create the screen yourself now. If you need help, you can always look at the previous screens and components.
Make sure you create a new package called **login** in the **onboarding** package. The full path should be **com.wiselab.<<name>>.feature.onboarding.login**.

#### 10.4.2 Create the ViewModel
Create a new Kotlin file called **LoginViewModel** in the **login** package.

```kotlin
class LoginViewModel : BaseViewModel()
```

* We extend the **BaseViewModel**. This is a class that we created to make it easier to create ViewModels. It contains some boilerplate code that we
  don't want to write every time we create a new ViewModel.

#### 10.4.3 Create the UiState
Create a new Kotlin file called **LoginUiState** in the **login** package.
Here you place the data that is needed to draw the screen. This is the data that is passed from the ViewModel to the View.

```kotlin
data class LoginUiState (
    val email: String = "",
    val password: String = "",
    val isPasswordVisible:Boolean = false
)
```

* Add this state to the **LoginViewModel**:

```kotlin
class LoginViewModel : BaseViewModel() {

    var state by mutableStateOf(LoginUiState())
        private set
}
```

#### 10.4.4 Create the UiAction
Create a new Kotlin file called **LoginUiAction** in the **login** package.
Here you place the actions that can be triggered from the View. This is the data that is passed from the View to the ViewModel.

```kotlin
sealed class LoginUiAction {
    data object OnLoginClicked : LoginUiAction()
    data object OnRegisterClicked : LoginUiAction()
    data object OnForgotPasswordClicked : LoginUiAction()
    data object OnPasswordVisibilityClicked : LoginUiAction()
    data class OnEmailChanged(val email: String) : LoginUiAction()
    data class OnPasswordChanged(val password: String) : LoginUiAction()
}
```

A **sealed class** is a class that can only be **extended in the same file**. This means that we can only use these actions in the Login screen.

* Add this action to the **LoginViewModel**:

```kotlin
class LoginViewModel : BaseViewModel() {

    var state by mutableStateOf(LoginUiState())
        private set

    fun onAction(action: LoginUiAction) {
        when (action) {
            is LoginUiAction.OnLoginClicked -> {}
            is LoginUiAction.OnRegisterClicked -> {}
            is LoginUiAction.OnForgotPasswordClicked -> {}  
            is LoginUiAction.OnPasswordVisibilityClicked -> {}
            is LoginUiAction.OnEmailChanged -> state = state.copy(email = action.value)
            is LoginUiAction.OnPasswordChanged -> {}
        }
    }
}
```

The `onEmailChanged` action is given as an example. The other actions you have to fill in yourself in a couple of moments.

#### 10.4.5 Implement in the Layout
Before we can use the viewModel in the Layout, we need to add it to the app module. Open the **appModule.kt** file in the **di** package.
Add the following code to the file:

```kotlin
val appModule = module {
    viewModel { LoginViewModel() }
}
```

* This way we can inject the viewModel in the Layout. Open the **LoginScreen** file and add the `viewModel: LoginViewModel = koinViewModel()` parameter to the **Screen** composable function.
* In the layout composable add 2 parameters:
  * `state: LoginUiState`
  * `onAction: (LoginUiAction) -> Unit = {}`

* In the screen composable add the following code:

```kotlin
@Composable
@Destination
@OnboardingNavGraph
fun LoginScreen(
    viewModel: LoginViewModel = koinViewModel()
) {
    LoginLayout(
      state = viewModel.state,
      onAction = viewModel::onAction
  )
}
```

* Now you can trigger the onEmailChanged action in the **LoginLayout** composable. Add this to your `EditText` composable: `onInputChange = { onAction(LoginUiAction.EmailChanged(it)) }`
* Do the same for the other password and passwordVisibility action. You can use the `onAction` function to trigger the actions in the ViewModel.
* Hint: For the **passwordVisibility** you can check the `onClickIcon` parameter of the `EditText` composable.

#### 10.4.6 Events
Now we need to add the events to the ViewModel.

* First create a new Kotlin file called **LoginUiEvent** in the **login** package.
* Add the following code to the file:

```kotlin
sealed class LoginUiEvent {
    data object Back : LoginUiEvent()
    data object NavigateToRegister : LoginUiEvent()
    data object NavigateToForgotPassword : LoginUiEvent()
}
```

* We will use the `eventChannel` for this. Add the following code to the **LoginViewModel**:

```kotlin
class LoginViewModel : BaseViewModel() {

    var state by mutableStateOf(LoginUiState())
        private set

    private val eventChannel = Channel<LoginUiEvent>()
    val eventFlow = eventChannel.receiveAsFlow()

    fun onAction(action: LoginUiAction) {
        when (action) {
            is LoginUiAction.OnLoginClicked -> {}
            is LoginUiAction.OnRegisterClicked -> eventChannel.trySend(LoginUiEvent.NavigateToRegister)
            is LoginUiAction.OnForgotPasswordClicked -> eventChannel.trySend(LoginUiEvent.NavigateToForgotPassword)
            is LoginUiAction.OnPasswordVisibilityClicked -> {}
            is LoginUiAction.OnEmailChanged -> state = state.copy(email = action.value)
            is LoginUiAction.OnPasswordChanged -> {}
        }
    }
}
```

* Now we can use the `eventFlow` in the **LoginScreen** composable. Add the following code to the **LoginScreen** composable:

```kotlin
@Composable
@Destination
@OnboardingNavGraph
fun LoginScreen(
    navController: NavController,
    viewModel: LoginViewModel = koinViewModel()
) {
    LoginLayout(
        state = viewModel.state,
        onAction = viewModel::onAction
    )

    LaunchedEffect(viewModel) {
        viewModel.eventFlow.collect { event: LoginUiEvent ->
            when (event) {
                is LoginUiEvent.Back -> navController.popBackStack()
                is LoginUiEvent.NavigateToRegister -> {}    //navController.navigate(RegisterScreenDestination) as example when the RegisterScreen is created
                is LoginUiEvent.NavigateToForgotPassword -> {}
            }
        }
    }
}
```

Now you're set to add **actions** and **events** to your `LandingScreen` composable to fix the navigation.
Don't forget to commit and push your changes. Create a pull request so your code can be reviewed! 🙌🏻
