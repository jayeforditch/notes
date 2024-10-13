# Getting started with theming

To use theming, we have to customize the ThemeData in `main.dart`. The problem with doing it like we have below is that we're basically telling Flutter that we're making our own theme completely from scratch. So we should technically fill out all the like 50 arguments you can pass to ThemeData.

```
void main() {
    runApp(
        MaterialApp(
            theme: ThemeData(),
            home: const Expenses(),
        ),
    );
}
```

Instead, we should edit the theme like this:

```
void main() {
    runApp(
        MaterialApp(
            theme: ThemeData().copyWith(),
            home: const Expenses(),
        ),
    );
}
```

By using `.copyWith()` and passing our arguments there, we keep all the default styling provided by Material 3 and then just change the parts of the theme that we pass.
