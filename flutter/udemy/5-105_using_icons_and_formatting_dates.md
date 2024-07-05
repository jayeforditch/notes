# Using Icons and Formatting Dates

Let's update our Expense model to include icons. We can add this code below where we put the enum -- outside of the class.

```
import 'package:flutter/material.dart';
import 'package:uuid/uuid.dart';

const uuid = Uuid();

enum Category { food, travel, leisure, work }

const categoryIcons = {
  Category.food: Icons.lunch_dining,
  Category.travel: Icons.flight_takeoff,
  Category.leisure: Icons.movie,
  Category.work: Icons.work,
};

class Expense {
  Expense({
    required this.title,
    required this.amount,
    required this.date,
    required this.category,
  }) : id = uuid.v4();
  final String id;
  final String title;
  final double amount;
  final DateTime date;
  final Category category;
}
```

This is great so far - we've nicely defined the structure of our expenses, which gives us type-safety and autocompletion tools. But remember, classes can also have methods. Let's add one to handle formatting the date called `getFormattedDate`. But wait, here's a tip!

If you find yourself creating a method that starts with `get`, make it a `getter` instead!

`Getters` are basically "computed properties" => Properties that are dynamically derived, based on other class properties.

Our `getter` will look like this: `String get formattedDate {}`. Note that we SKIPPED adding the parenthesis and only added the curly braces.

Outside of the class, we define our formatter: `final formatter = DateFormat.yMd();`

And we use that in our getter:

```
String get formattedDate {
    return formatter.format(date);
}
```

Then, when we want to use that getter, we can do so like this: `Text(expense.formattedDate)`. Note that we are NOT using parenthesis after `formattedDate` - this is because it is a getter, not a method (which we also could've used).
