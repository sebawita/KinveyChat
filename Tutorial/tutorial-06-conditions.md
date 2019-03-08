
# Conditions

Sometimes when you work on a chatbot conversation, you need an extra control over the flow of the conversation. 
Some steps should be executed only in specific cases (or skipped in specific cases).

For example: the chatbot could ask users if they have a discount-code. If the user says yes, then it should ask for the discount-code. If the user says no, then it should skip the discount-code question.

![](./img/has-code-logic.png?raw=true)

The above scenario can be achieved in two steps:

1. A **confirmation** step - which ask the user a **Yes**/**No** question:

  ```json
  {
      "type": "confirmation",
      "entity": "hasCode",
      "messages": [
        "Do you have a Discount Code?"
      ]
  },
  ```

2. A **question** step with a `conditions` array - which only shows if `hasCode` is true:

  ```json
  {
      "type": "question",
      "entity": "discount-code",
      "entity-type": "",
      "messages": [
        "What is your Discount Code?"
      ],
      "conditions": [
        "{{hasCode}}"
      ]
  }
  ```

## Validation

Conditions can also be used with custom validators, where you can provide a set of expressions to check if the provided user input is valid.
For example, you might not want your users to order more than 5 tickets.

```json
{
  "type": "question",
  "entity": "tickets",
  "entity-type": "Number",
  "messages": [
    "How many tickets would you like?"
  ],
  "reactions": {
    "validations": [
      {
        "type": "custom",
        "parameters": {
          "condition": "{{$lt tickets 6}}"
        },
        "error-message": [
          "Sorry, but you can't buy more than 5 tickets."
        ]
      }
    ]
  }
}
```

## Conditional Operators

There are a number of very useful operators that you could use.

### Operator `$not`

The `$not` operator is a simple negation operator, that changes true to false, and false to true.

Following on the initial **Discount Code** example, you could use it to show a different message when `hasCode` is `false`.

You could do this like this:

```json
{
  "type": "question",
  "entity": "discount-code",
  "messages": [
    "You could sign up to our newsletters to receive a discount code next month."
  ],
  "conditions": [
    "{{$not hasCode}}"
  ]
}
```

### Comparison operators

You could also use a comparison operator to complete the same scenario, and check if:

* `hasCode` **equals** `false`, like this:

  ```json
"{{$eq hasCode false}}"
```

* `hasCode` **not equals** `true`, like this:

  ```json
"{{$ne hasCode true}}"
```


Here is a list of available comparison operators.

* `$eq` - equals
* `$gt` - greater than
* `$gte` - greater or equal
* `$lt` - less than
* `$lte` - less or equal
* `$ne` - not equal

### Operator `$in`

The`$in` operator allows you to check if a value is present in an array.

```json
"{{$in myValue myArray}}"
```

### Boolean `$and` `$or`

You can also use `$and` and `$or` to build more complex conditions, like this:

```json
"{{$and (condition 1) (condition 2)}}"
```

For example when a price should be **less than 20**, but also it should be a **positive number**:

```json
"{{$and  ($lt price 20) ($gt price 0) }}"
```

### Operator `$has`

The `$has` operator allows you to check if a specific entity has been provided by a user. This operator comes in handy in many scenarios.

#### Don't repeat messages

> This might be a good moment to explain the algorithm that controls the cognitive flow:
> 
> Each time user provides an input, the flow algorithm evaluates each step from the beginning, until gets to a step that requires an input for the user.
> 
> This means, that if it gets a questions step, it checks if the entity is already provided: 
> 
> * if true => move to the next step
> * if false => ask user and stop
> 
> However, if the flow algorithm gets to a message step, it automatically prints out the message.

Most of the time you only want to print out a message in a specific scenario. For example: the initial message *Great, let me help you find a car for you.*, should only be displayed when the user starts the conversation.

This can be done simply by checking if the user **has not** provided the **startDate**.

Update the first message step in the `rent-car` conversation, to add a condition that handles that:

1. Find the first message step in the `rent-car` conversation
2. Add a `conditions` array => start typing **co** and select `conditions`
3. Add the following condition: `"{{$not ($has startDate)}}"`

  The step should look like this:

  ```json
{
      "type": "message",
      "messages": [
        "Great, let me help you find a car for you."
      ],
      "conditions": [
        "{{$not ($has startDate)}}"
      ]
},
```

##### Test

Save and test the `rent-car` conversation.

This time the initial message should show only once.

#### Skip Parent Question

Another scenario where the `$has` operator comes in handy, is when looking up an entity requires a two (or more) step process. 

For example, in order to find a city, the chatbot asks for a country first, and then based on the country it provides a list of cities. However, if the user already provided the city, then it would be good to skip the country step.

Please, note that the **value of the country** is used in the validation of the city. So, the city validation should also be updated, so that it only check the city.country when country has been provided.

Follow these steps to update the `rent-car` conversation:

1. Find the `country` question step
2. Add a `conditions` array => start typing **co** and select `conditions`
3. Add the following condition => `"{{$not ($has city)}}"`
4. Find the `city` question step
5. Update `Validations->parameters->condition`

  You need to use an `$or` operator where:
  *  either country is not provided => `($not ($has country))`
  *  or the city.country is the same as the provided country => `($eq city.country country)`

  Like this:

  ```json
"condition": "{{$or ($not ($has country)) ($eq city.country country)}}"
```

  <!--
```json
"condition": "{{$eq  ($has country) ($eq city.country country)}}"
```
-->

##### Test

Save and test the `rent-car` conversation.

Try the following conversations:

1. Send: *I want to rent a car from today to tomorrow*
2. Send: *Berlin*
3. You should be asked to choose a car
4. Select a car
5. Send: *I want to rent a car from today to tomorrow in France* 
6. Send: *Berlin*
7. You should be shown an error message
8. Send: *Paris*
9. You should be asked to choose a car
10. Select a car



