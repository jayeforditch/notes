# Introducing Enums

We've made our model for our expenses, but there's a problem - we forgot to add a 'category' field.\

We COULD just go into our class and add `final String category;` but then category could be anything. `leisure` would be different than `lesure` because typos happen. Instead, let's restrict the options.

`enum` is a keyword that lets us make our own custom types, for example `enum Category`. We don't create enums within a class. Instead, they go outside, on their own.

```
enum Category {food, travel, leisure, work}

class Expense {
    Expense({...})
}
```

Interestingly, you don't wrap the enum values in quotations or anything - you just write them as is.

With that, we can use the new `Category` type in our model: `final Category category;`
