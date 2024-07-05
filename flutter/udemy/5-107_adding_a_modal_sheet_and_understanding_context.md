# Adding a Modal Sheet and Understanding Context

In `expenses.dart`, we're adding an IconButton as an action in our `AppBar()` that when clicked should open up a bottom sheet.

```
void _openAddExpensesOverlay() {
    showModalBottomSheet(context: context, builder: (BuildContext ctx) {return Text('Bottom Sheet');});
}

@override
Widget build...
...
actions: [
    IconButton(
        icon: const Icon(Icons.add),
        onPressed: _openAddExpenseOverlay,
    )
]
```

What's this? The `showModalBottomSheet` requires a context, and it isn't erroring out, even though we're not passing down context from the parent. What's going on here?

When you're working in a `StatefulWidget`, Flutter will automatically give you a global context to work with. That's why `showModalBottomSheet` is not erroring out -- it's using the global state that is given by Flutter in the background.

## Understanding context

Context is like... a widget's metadata. It contains information on relation to other widgets, most importantly, it contains information on the position of the widgets in the overall UI tree.

Inside of the builder for the `showModalBottomSheet`, we're calling it `ctx` instead of `context` because it's technically a DIFFERENT context that the one used in `showModalBottomSheet`, which is the global one. `ctx` is the bottom sheet's position in the widget tree.
