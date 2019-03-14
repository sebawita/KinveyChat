
# Completing a conversation

In order to complete a conversation it is usually best to ask the user to confirm their input, and once they are happy with the result, push the data to your backend or trigger some other process outside of the chatbot domain.

## Entities Confirmation

Entities confirmation is a crucial step that brings in a lot of value to the conversation:

* it shows the user all the values that they provided
* it allows the user to change any of the values - that is without the need to redo the whole conversation
* it provides a mechanism to confirm the conversation, which allows the chatbot to complete the transaction

There is a special type of a step `entities-confirmation`, which handles all of the above functionality in a simple, but elegant fashion.

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

![Demo](./img/finishing-confirmation-flow-demo.gif?raw=true)

## Complete the conversation with a Webhook

Once the user completes the conversation and confirms their choice, all you are left with is to take the date collected in the conversation and send it to your backend.
This would usually mean triggering a process to execute on the agreed transaction. For example:

1. user: orders a pizza via your chatbot
2. chatbot: sends the order to your backend
3. payment system: charge the provided payment card
4. kitchen: cooks a pizza
5. delivery: delivers the pizza

In order to send the data to your backend you can use a `webhook` step. Its purpose is simple, execute an HTTP command described in the `data-source` property and display the provided **message** once that is complete.

Here is what the `step-webhook` code snippet generates:

```json
{
  "type": "webhook",
  "data-source": {
    "endpoint": "https://",
    "method": "POST",
    "headers": {
      "header name": "header value"
    },
    "payload": {
      "key": "value"
    }
  }
}
```

## Conversation Flow for Webhook Steps

A **Webhook Step** behaves just like a **Message Step** in the sense that the **Conversation Flow Algorithm** will execute it every time it steps over it. So, if you follow the pizza order webhook step with a satisfaction question, your customer might get charge twice and will receive two pizza orders.

This can be easily avoided. You can add an `entity` property, which is used to store the result returned from executing the webhook. You can use the entity content to display it to the user (more on that in the bonus section). But more importantly in this case, the **Conversation Flow Algorithm** will see that the webhook already contains a result, and therefore, it will not execute it again.

## Send the Order to the backend

To finalise the car renting conversation, you need to send the Order to the backend, and then provide the user with a confirmation message.

### Backend

For the purposes of the Car Rental service, there is a Cloud Function called **BookCar**, that allows you to complete the booking process.

**BookCar** can be reached with the following parameters:

```json
"endpoint": "https://baas.kinvey.com/rpc/kid_r1275v_H4/custom/BookCar",
"method": "POST",
"headers": {
  "Authorization": "Basic a2lkX3IxMjc1dl9INDo5YWYxOTcxZTFlY2U0NTNhYWUwOTQ2MzZlYmM5MGJlNw=="
},
```

The booking details should provided as the request body (`payload` in the chatbot configuration). For example:

```json
{
  "pickUpDate": "24 Mar 2019",
  "dropOffDate": "30 Mar 2019",
  "car": "Smart"
  "price": "200",
  "city": "Berlin",
  "email": "me@email.com"
}
```

In the real world application the **BookCar** function should perform some extra validations, save the new order and trigger additional processes (like an email confirmation etc.).

In the context of this tutorial, **BookCar** only saves the provided data in a collection and returns a unique booking reference, i.e. `{ bookingRef: BER01234 }`
Additionally, in order to avoid storing sensitive data, the provided email always gets replaced with `sample@email.com`.

### Add a Webhook step to call BookCar 

In order to complete the transaction with a call to the **BookCar** cloud function, follow these steps:

1. Got to the end of the `rent-car` conversation
2. Add a `webhook` step => start typing **stw**
3. Configure the `data-source` to call `BookCar` (see the [configuration above](./tutorial-07-finishing-touches.md#backend)),
4. Configure the `payload`, use the following fields
  * `pickUpDate` -> `startDate` 
  * `dropOffDate` -> `endDate`
  * `car` -> `car`
  * `price` -> `car.price`
  * `city` -> `city`
  * `email` -> `email` (although you can ignore this field, as it is not used)

    Additionally, you should format the dates, so that they are passed to the backend in a specific format. This can be done with the `$date` formatter, like this: `{{$date startDate 'D MMM YYYY'}}`.
    
5. Your `data-source` should look like this:

  ```json
  "data-source": {
      "endpoint": "https://baas.kinvey.com/rpc/kid_r1275v_H4/custom/BookCar",
      "method": "POST",
      "headers": {
        "Authorization": "Basic a2lkX3IxMjc1dl9INDo5YWYxOTcxZTFlY2U0NTNhYWUwOTQ2MzZlYmM5MGJlNw=="
      },
      "payload": {
        "pickUpDate": "{{$date startDate 'D MMM YYYY'}}",
        "dropOffDate": "{{$date endDate 'D MMM YYYY'}}",
        "car": "{{car}}",
        "price": "{{car.price}}",
        "city": "{{city}}",
        "email": "{{email}}"
      }
  },
  ```

6. Add an `entity` property (preferably, just before the `data-source` configuration) => start typing **en** and select `entity` and call it `order-response`. This will allow you to capture the response returned by **BookCar**.
7. Add a `massages` array (preferably, at the end of the step configuration)  => start typing **me** and select `messages`. 

  Then add this message: *Your car is booked. Your booking reference is {{order-response.bookingRef}}.*

8. Save

#### Solution

The whole step should look like this:

```json
{
  "type": "webhook",
  "entity": "order-response",
  "data-source": {
    "endpoint": "https://baas.kinvey.com/rpc/kid_r1275v_H4/custom/BookCar",
    "method": "POST",
    "headers": {
      "Authorization": "Basic a2lkX3IxMjc1dl9INDo5YWYxOTcxZTFlY2U0NTNhYWUwOTQ2MzZlYmM5MGJlNw=="
    },
    "payload": {
      "pickUpDate": "{{$date startDate 'D MMM YYYY'}}",
      "dropOffDate": "{{$date endDate 'D MMM YYYY'}}",
      "car": "{{car}}",
      "price": "{{car.price}}",
      "city": "{{city}}",
      "email": "{{email}}"
    }
  },
  "messages": [
    "Your car is booked. Your booking reference is {{order-response.bookingRef}}"
  ]
}
```

#### Test

Now you should be able to test the full conversation `rent-car` conversation.

Try the following steps:

1. Send: *I want to rent a car*
2. Select: **StartDate**, **EndDate**, **Country**, **City**, **Car**, **Email**
3. Press: **Confirm**
4. You should receive a message with a unique **bookingRef**.
git

![Full Demo](./img/finishing-full-flow-demo.gif?raw=true)


## Congratulations

And just like that you have created a fully functioning chatbot. That can communicate using Natural Language Processing, communicate with the backend to display data, validate user input and save the transaction to the backend.

There is still more to learn. You can find a lot more info the [documentation](https://docs.nativechat.com/).
