
# Understanding the Conversation Flow Algorithm

It is important to understand the algorithm that controls the cognitive flow.

Each time a user provides an input, the **conversation flow algorithm** evaluates each step **starting from the beginning** (even if you believe that this step has been completed), until it gets to a step that requires an input for the user or the conversation is complete.

This means, that if it gets a questions step, it checks if the entity is already provided (i.e. `country`) 

* if true => move to the next step
* if false => ask user and stop

However, if the flow algorithm gets to a message step, it automatically prints out the message. This is why you saw the *"Great, let me help you find a car for you."* printing each time you typed a new value.

Most of the time you only want to print out a message in a specific scenario. For example: the initial message *Great, let me help you find a car for you.*, should only be displayed when the user starts the conversation. But once the user provides some data like the **country**, it should stop printing again.

This can be done simply adding a `conditions` array, which could check if the user **has not** provided the **country**.

Update the first message step in the `rent-car` conversation to add a condition:

1. Find the first message step in the `rent-car` conversation
2. Add a `conditions` array => start typing **co** and select `conditions`
3. Add the following condition: `"{{$not ($has country)}}"`

  The step should look like this:

  ```json
{
      "type": "message",
      "messages": [
        "Great, let me help you find a car for you."
      ],
      "conditions": [
        "{{$not ($has country)}}"
      ]
},
```

##### Test

Save and test the `rent-car` conversation.

This time the initial message should show only once.

<!--TODO: update the gif-->
![](./img/flow-has-not-country-demo.gif?raw=true)

