# Understanding Keys -- Setup

So what are keys, those things we're always using in stateful widgets?

```
class Expenses extends StatefulWidget {
  const Expenses({super.key});
  @override
  ...
}
```

# Which Problem Do Keys Solve?

Let's say we have a todo app where you can sort asc or desc based on priority. The first time the build method is called, you have three todos -- 1, 2, and 3. When you press the button to re-sort them, the new order is 3, 2, 1. But what is happening behind the scenes?

The first time the build method is called, the widget tree and the element tree are copies.

| Widget Tree                                                               | Element Tree                                      |
| ------------------------------------------------------------------------- | ------------------------------------------------- |
| Defines user interface & widget relations -> build() is called frequently | Created behind the scenes -> elements are re-used |
| Column()                                                                  | Column Element                                    |
| TodoItem('1')                                                             | <- TodoItem Element                               |
| TodoItem('2')                                                             | <- TodoItem Element                               |
| TodoItem('3')                                                             | <- TodoItem Element                               |

When we press the button to re-sort, the Element skeleton actually stays the same -- the same things are still on the screen, the placement just changed. What actually changes is which element refrences which widget. This is done for performance reasons. Even though something changed with the widgets, it's still the same amount of widgets and the same types of widgets. I don't need to re-build and scrap everything that's there, I just need to change the references.

But things start to break if we change the todos and make them checkable (add a checkbox to each of them). At first glance, everything seems fine. You click the first one and it checks correctly. Second one, still works. But if you then re-sort them, the first todo will still be checked, even though the todo that was there before is now the third one. Broken!

Even though Flutter is keeping track of the references and stuff, state is managed differently. It is kept independent from the widgets and is connected to the elements.

| Widget Tree                                                               | Element Tree                                      |         |
| ------------------------------------------------------------------------- | ------------------------------------------------- | ------- |
| Defines user interface & widget relations -> build() is called frequently | Created behind the scenes -> elements are re-used |         |
| Column()                                                                  | Column Element                                    |         |
| TodoItem('1')                                                             | <- TodoItem Element ->                            | State 1 |
| TodoItem('2')                                                             | <- TodoItem Element ->                            | State 2 |
| TodoItem('3')                                                             | <- TodoItem Element ->                            | State 3 |

So if we have three todo widgets and they're connected to their respective elements by references, the states of those objects DON'T move which causes this problem.

| Widget Tree                                                               | Element Tree                                      |         |
| ------------------------------------------------------------------------- | ------------------------------------------------- | ------- |
| Defines user interface & widget relations -> build() is called frequently | Created behind the scenes -> elements are re-used |         |
| Column()                                                                  | Column Element                                    |         |
| TodoItem('3')                                                             | <- TodoItem Element ->                            | State 1 |
| TodoItem('2')                                                             | <- TodoItem Element ->                            | State 2 |
| TodoItem('1')                                                             | <- TodoItem Element ->                            | State 3 |

This problem can be solved by using keys.

# Understanding & Using Keys

Flutter determines whether an element can be reused or not, simply by taking a look at the type of that element. For example, TodoItem. If the type on the element tree matches what's in that same position on the widget tree, then the widget is re-used and just the reference is updated.

But, if we add keys to our TodoItems, then that will be checked as well as the type. So when checking to see if anything needs to be rebuilt, Flutter first checks the types on the widget and element trees. Those are the same. But wait, there are keys! Let's check those too. Ahhh, the keys are out of alignment. Ok, let's actually move the element. And when the element moves, so does the state that's connected to it!

So what should we use as the value for key? The first option is a `ValueKey()` which comes with Flutter. `ValueKey()` wants a value, but that value should be unique between the other todo values. So we can use `ValueKey(todo.text)` because that should be unique. Ideally, we'd use an id.

An alternative to the `ValueKey()` would be to use an `ObjectKey()`, in which case we could use the entire todo object as a value, like this: `ObjectKey(todo)`

```
for (final todo in _orderedTodos)
    CheckableTodoItem(
        key: ValueKey(todo.text),
        todo.text,
        todo.priority,
    )
```

# Mutating Values in Memory & Making Sense of var, final & const

```
final numbers = [1,2,3];
numbers.add(4);
```

Why does this work? Isn't `numbers` final? What final actually means is that you can't assign a new value to it. So this doesn't work:

```
final numbers = [1,2,3];
numbers = [4,5,6]; <- this line throws an error
numbers.add(10); <- this just adds a new item to the list. It's not re-assigning the variable, it's updating the object in memory. Using the `=` sign creates a new object in memory.
```

If I create numbers with `var` instead of `final`, I can reassign the value.

If I create `numbers` with `const`, I'll get an error when I try to add a new value -- `Cannot add to an unmodifiable list`. `numbers` isn't just `final`, it also can't be manipulated.
