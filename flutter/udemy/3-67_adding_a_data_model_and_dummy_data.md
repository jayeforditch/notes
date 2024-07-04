# Section 3 - Video 67: Adding a Data Model & Dummy Data

Models are classes that act as blueprints for specific objects.

Objects help with organizing data and separating logic.

Create a new folder within the `lib` folder called `models`.

```
class QuizQuestion {
    const QuizQuestion(this.text, this.answers);
    final String text;
    final List<String> answers;
}
```

```
const questions = [
    QuizQuestion('What are the main building blocks of Flutter UIs?',
    [
        'Widgets',
        'Components',
        'Blocks',
        'Functions',
    ],
    )
]
```
