## Create your first components and screen

Start creating some UI components and a screen to show them on.

### 6.1 Create some buttons

* Create a new package called **shared** in the **ui** package. (Not in the **theme** package)
* Create a new Kotlin file named **Buttons** to create our different buttons
* Create the following buttons:
    * PrimaryButton
    * SecondaryButton
    * TertiaryButton
    * OutlinedButton
* Show a preview of the buttons like this:

<img width="400" src="index/img/setup/button_preview.png" />

* Here's a bit of code to get you going:

```kotlin
@Preview
@Composable
fun ButtonsPreview() {
  AppTheme {
    Column(
      //...
    ) {
      Buttons.Primary(
        text = "Primary",
        Modifier.fillMaxWidth()
      ) {}

      //...
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
                //...,
                //...
            ),
            onClick = onClick
        ) {
            Text(text = text)
        }
    }
}
```
* Don't forget to use the **Spacing** you set up in the previous step!

### 6.2 Create a screen
Because the task manager is seen as 1 Epic, create a new package called **taskManager** in the main package. In this package create a new package called **landing** for the landing screen.
