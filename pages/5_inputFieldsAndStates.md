## Working with states

Now that you know how to make your first screen, we can start working with states. To work with states we first need to understand how Composable
functions work.

### 9.1 Composition and recomposition

A basic **Composable function** with no parameters will never change how it is drawn. This implies that the view is never redrawn, unless the parent
Composable function is redrawn.
When a Composable function is redrawn, it is called **recomposition**. One way to trigger recomposition is triggered is when a Composable function is
called with different parameters.
For instance if we have a Composable function that takes a String as a parameter, and we call it with the String "Hello World", and then we call it
again with the String "Hello Wisemen", the Composable function will be recomposed.
If we want to trigger recomposition without changing the parameters, to add increased functionality, we need to use **state**.

### 9.2 State

A State is a way to **store data** in a Composable function. When the state changes, the Composable function is recomposed. A basic way to create a
state is:

```kotlin
var state by remember { mutableStateOf("Hello World") }
```

Here we created a state that holds a String. The state is initialized with the value "Hello World". The state is **mutable**, which triggers
recomposition of the views using this value as parameter.
The **remember** function is used to make sure the state is not recreated every time the Composable function is recomposed. This is important because
if the state is recreated, the value is reset to the initial value.

### 9.3 TextField

The best way to learn how to work with states is to work with a ``TextField``. Like we did for the buttons, we will create a new file called **
EditText.kt**. Here you can create your basic edit texts that can be reused in the future.
Create a basic EditText like this:

```kotlin
@Composable
fun EditText(
    modifier: Modifier = Modifier
) {
    var input = ""

    TextField(
        value = input,
        onValueChange = { input = it },
        modifier = modifier
    )
}
```

When you run this preview and try to type in this textField, you see that nothing happens. This is because the parameters of the EditText are never
changed and the composable function is not smart enough to know that the input variable is changed.
To fix this we need to use state. Change the EditText to this:

```kotlin
@Composable
fun EditText(
    modifier: Modifier = Modifier
) {
    var input by remember { mutableStateOf("") }

    TextField(
        value = input,
        onValueChange = { input = it },
        modifier = modifier
    )
}
```

Now when you run the preview and type in the TextField, you see that the text is updated. This is because the state is changed and the Composable
function is recomposed. **IT LIVES** 🎉

### 9.4 Stateless first and state hoisting

When working with states, it is important to keep your Composable functions stateless as much as possible. As mentioned earlier, a Composable function
is recomposed when the state changes.
If your UI gets more complex, and you use more function hierarchies with all their different states, it can become hard to keep track of all the
states. And also it can lead to **unwanted recompositions** an because of that a **bad performance**.
This also breaks the "single source of responsibility" principle which can lead to bug-prone code.
To prevent this, we can use state hoisting. State hoisting is a way to move the state to the parent Composable function.

Let's improve our EditText with state hoisting so it follows this principle.

```kotlin
@Composable
fun EditText(
    modifier: Modifier = Modifier,
    input: String,
    onInputChanged: (String) -> Unit
) {
    TextField(
        value = input,
        onValueChange = onInputChanged,
        modifier = modifier
    )
}


@Composable
fun ComposableUsingEditText() {
    var input by remember { mutableStateOf("") }

    EditText(
        input = input,
        onInputChanged = { input = it }
    )
}
```

Now the EditText is stateless and the state is hoisted to the parent Composable function.
If you do this for all your Composable functions, you will have a better performance and bugs will be easier to find.

### 9.5. Exercise: Finishing the component library

Now that you know how to style your Composables and how to work with states, you can finish the component library.

Keep these principles in mind:

- Make sure to create a new file for every type of component you create. This way you can reuse them in the future and you don't have to restyle them
  every time.
- Remember to add the modifier parameter to your custom views so they are easier to lay out in the future.
- Make sure to use state hoisting when working with states.
- Use previews to test your components handling of different states.

After you are finished, make a new PR and ask your buddy to review it. 🚀

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
    saveStateHandle: SavedStateHandle
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
