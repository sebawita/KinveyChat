
# Completing a conversation

In order to complete a conversation it is usually best to ask the user to confirm their input, and once they are happy with the result, push the data to your backend.

## Entities Confirmation

Entities confirmation is a crucial step that brings in a lot of value to the conversation:

* it shows the user all the values that they provided
* it allows the user to change any of the values - that is without the need to redo the whole conversation
* it provides a mechanism to confirm the conversation, which allows the chatbot to complete the transaction

There is a special type of a step, `entities-confirmation`, which handles all of the above functionality in a simple, but elegant fashion.

### Add confirmation step

Add a confirmation step to the `rent-car` conversation, which should ask the user to confirm the **startDate**, **endDate**, **city**, **car** and **email** entities. Like this:

1. Go to the end of the `rent-car` conversation
2. Add `entities-confirmation` step => start typing **sten** and select `step-entities-confirmation`. You should get the following code

  ```json
    {
      "type": "entities-confirmation",
      "entity": "result-entity-name",
      "entities": [
        "entity-name"
      ],
      "messages": [
        "Confirm entities?"
      ]
    }
  ```

3. Add the **startDate**, **endDate**, **city**, **car** and **email** entities to the `entities` array
4. Update the `messages` to display a message in the following three lines:
  * *Are these the correct details?*
  * *Press on any of the items you wish to update.*
  * *Press \"Confirm\" to complete the transaction.*
5. Save

### Solution

The whole step should look like this:

```json
{
  "type": "entities-confirmation",
  "entity": "result-entity-name",
  "entities": [
    "startDate",
    "endDate",
    "city",
    "car",
    "email"
  ],
  "messages": [
    [
      "Are these the correct details?",
      "Press on any of the items you wish to update.",
      "Press \"Confirm\" to complete the transaction."
    ]
  ]
}
```

### Test

Now you should be able to test the confirmation step in the `rent-car` conversation.

Try the following steps:

1. Send: *I want to rent a car*
2. Select: **StartDate**, **EndDate**, **Country**, **City**, **Car**, **Email**
3. You should be presented with the confirmation step and the provided entity values
4. Click on the email, and change it
5. You should be presented with the confirmation step again, but the email should be changed
6. Press: **Confirm** 

## Complete with a Webhook

<!--TODO:
Add the explanation of how to send the data with a webhook and add an exercise to send the result to Kinvey and show how to display the data
-->
