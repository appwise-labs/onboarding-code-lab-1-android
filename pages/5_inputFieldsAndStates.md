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
