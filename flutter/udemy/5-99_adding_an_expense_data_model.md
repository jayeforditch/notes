# Adding an Expense Data Model with a Unique ID and Exploring Initializer Lists

- Create a folder in `lib` called `models`, then create a file called `expense.dart`. The purpose of this file is to describe the structure that the expenses in this app should have.
- Each of our expenses should have a title, amount, and date. They should also have a uuid which will be generated each time the Expense class is instantiated. We can do this using an initializer list, which is the `: id = uuid.v4()` part of the code.

- In Dart, "Initializer Lists" can be used to initialize class properties (like "id") with values that are NOT received as constructor function arguments.

```
import 'package:uuid/uuid.dart';

const uuid = Uuid();

class Expense {
  Expense({
    required this.title,
    required this.amount,
    required this.date,
  }) : id = uuid.v4();
  final String id;
  final String title;
  final double amount;
  final DateTime date;
}
```
