
# Reactions

So far you have built a functional chatbot that allows a customer to select the booking dates, country, city and a car, which is supported with a set of handy UI widgets. This is great and very powerful stuff.

However, the chatbot could be even more proactive when interacting with the user to:

* keep the user well informed on what the bot understands,
* validate user input,
* resolve ambiguous input statements,
* provide suggestions based on history.

All of these interactions can be achieved with the help of **Reactions**, which allow you to expand the behavior of each question step. Depending on the user's input, the chatbot engine might trigger one of the configured reactions.

To add a **reaction** to a question step, you just need to add a `reactions` object with the right configuration, which instructs the chatbot on how to react to the user's input.

## Acknowledgments

The most commonly used reaction is an **acknowledgment**, which acknowledges to the user that the chatbot captured an entity value. This is very useful when a user provides data to the chatbot outside of the regular flow. For example:

```
<user>: I want to rent a car in **Spain**
```

Here the user provides the Country, without the chatbot having to ask for it. So, the natural reaction should be something like:

```
<bot>: Let's find you a car in **Spain**.
<bot>: In which city are you looking for a car?
```

However, when the user provides the value, as an answer to chatbots request, like this:

```
<bot>: In which city are you looking for a car?
<user>: Madrid
<bot>: What car would you like?
```
then, there is no need to acknowledge the City, as that is already expected.

### Add an acknowledgment for Country and City

In this exercise, you need to update the Country and City steps, to provide acknowledgments of receiving the values.

To do that, you need to add `"reactions"` object with an `"acknowledgments"` property to the Country step, and then provide expressions used for the acknowledgment. Like this:

```json
"reactions": {
  "acknowledgements": [
    "Cool, I see you picked {{country}}.",
    "I understand that you need a car in {{country}}."
  ]
}
```

> Note, you can use the code snippets once again. Just type `rea` and then `ack`, and you should be able to pick the right options.

The whole step should look like this:

```json
{
  "type": "question",
  "entity": "country",
  "entity-type": "Country",
  "messages": [
    "Which country are you traveling to?"
  ],
  "display": {
    "type": "quick-reply"
  },
  "reactions": {
    "acknowledgements": [
      "Cool, I see you picked {{country}}.",
      "I understand that you need a car in {{country}}."
    ]
  }
},
```

Now, try to do the same in the City step with the following expressions:

* *I see you want one of our cars in CITY_NAME*,
* *We should have a car for you in CITY_NAME*

### Test

To test it, try this conversation:
<!--#### Provide Country and City out of order-->

1. Send: *I want to rent a car in France*
2. The bot should acknowledge **France**
3. Send: *Oh in Paris*
4. The bot should acknowledge **Paris**

## Validations

<!--You can also configure your chatbot to validate user input and -->
Another type of reaction is **User Input Validation**, which allows the chatbot to validate the input value, and display an error message if required.

The validation can be as simple as a format check of the input, using: **phone**, **email** or **regex** templates.
These are used to make sure that the user gave us something that at least **looks like** a real **phone** or **email** or other value.

```json
"reactions": {
  "validations": [
    {
      "type": "phone",
      "error-message": [
        "This doesn't look like a valid phone number. Can you type it again?"
      ]
    }
  ]
}
```

<!--

You could also require the input to have specific **length** constraints (number of characters) or the value to be in a specific **range** or even **match** one of the provided acceptable answers.
These can be used to make sure that the values match 

```json
"reactions": {
  "validations": [
    {
      "type": "length",
      "parameters": {
        "min": 8,
        "max": 12
      },
      "error-message": [
        "The registration plate should be between 8 and 12 characters. Please try again."
      ]
    }
  ]
}
```

When dealing with pictures, you could use **image** validation, which allows you to specify a list of tags that you expect the picture to **contain**. The chatbot will analyze the picture provided by the user, generate a list of tags. The validation will fail if none of the tags matches those in the **contains list**.

For example: 

```json
"reactions": {
  "validations": [
    {
      "type": "image",
      "error-message": [
        "This doesn't look like a receipt. Please upload a photo with the receipt that you received from our agent."
      ],
      "parameters": {
        "contains": [
          "receipt", "document"
        ]
      }
    }
  ]
}
```
-->

You can read more about [User Input Validation in the docs](https://docs.nativechat.com/docs/1.0/cognitive-flow/validation.html). 

### Custom validation: startDate vs endDate 

As it is, the user could ask to rent a car from **4-Mar** to **1-Mar**, which is not right, as you cannot drop off a car in the past.

You need to add a validation to ensure that the drop-off date (`endDate`) is greater than the pick-up date (`startDate`). To do that you need to use a **custom** validation, follow these steps:

1. Find the question step with the `endDate` entity
2. Add a `reactions` object => using code snippets => start typing **rea** and select `reactions`,
3. Add a `validations` array => using code snippets => start typing **val** and select `validations`,
4. Add a `custom` validation object => using code-snippets => start typing **vacu** and select `validation-custom`, your code should look like this:

  ```json
    "reactions": {
      "validations": [
        {
          "type": "custom",
          "parameters": {
            "condition": "{{}}"
          },
          "error-message": [
            "Invalid value message"
          ]
        }
      ]
    }
  ```

5. Set the `condition` to check that the endDate > startDate => using : `"condition": "{{$gt endDate startDate}}"`
  * we will dive deeper into conditions in another chapter
6. Set the error message to say: *"The drop-off date ({{endDate}}) cannot be before the pick-up date ({{startDate}})."*

#### Solution

The whole step should look like this:

```json
{
  "type": "question",
  "entity": "endDate",
  "entity-type": "Date",
  "messages": [
    "When would you like to drop off the car?"
  ],
  "display": {
    "type": "date-picker",
    "title": "Find drop-off date",
    "button-text": "Select"
  },
  "reactions": {
    "validations": [
      {
        "type": "custom",
        "parameters": {
          "condition": "{{$gt endDate startDate}}"
        },
        "error-message": [
          "Invalid value message"
        ]
      }
    ]
  }
},
```

#### Test

Now you should be able to test date validation of the `rent-car` conversation.
Just try to provide different dates pick-up and drop-off dates and see if you get an error message when the drop-off date is not greater than the pick-up date.

### Validation: with Entity Properties

In the current flow, the user can select a country and then it is provided with a list of cities for those countries. However, the user can still type any city, which might result in a request for **Paris** in **Italy**.

<!--
If you look at the City Entity, you will notice that each record contains a `country`, `city` and `localName` properties.
-->
When a user types `rome`, the chatbot doesn't just take it as a `"rome"` string, but it actually matches the full object (from the trained City Entity collection) together with all its attributes, like this:

```json
{
  "country": "Italy",
  "city": "Rome",
  "localName": "Roma"
}
```

You can create a condition that could compare `city.country` to the `country` entity, like this:

```json
"condition": "{{$eq city.country country}}"
```

Add a custom validation - with the above condition - to the **city** reaction.

#### Solution

> Before, you copy this solution. Did you try to implement yourself? Remember, practice will help you learn.

The full reaction code should look like this:

```json
"reactions": {
  "acknowledgements": [
    "I see you want one of our cars in {{city}}",
    "We should have a car for you in {{city}}"
  ],
  "validations": [
    {
      "type": "custom",
      "parameters": {
        "condition": "{{$eq city.country country}}"
      },
      "error-message": [
        "{{city}} is not in {{country}}. Please select a city in {{country}}."
      ]
    }
  ]
}
```

#### Test

Now you should be able to test city validation of the `rent-car` conversation.

Note, you need to provide the dates before the city validation gets triggered.

### Query based validation: Available Cars

You can also validate based on the result of a dynamic query. 
This very useful where the list of valid options depends on the current state of the backend.

This is also the case for the Car Rental business. Not all cars are always available. As they might be already booked for a specific date or city.

#### Backend - Cloud Function: GetAvailableCars

The backend provides you with a cloud function called `GetAvailableCars`, which expects you to provide a `city` and in response, it returns a simplified list of available cars.

> Note, this cloud function is very basic and always returns, the same cars for these cities:
> 
> * Berlin, Warsaw, Paris, Rome => "Kia Sorento", "Mazda MX-5", "Mercedes S-Class Cabriolet", "Smart"
> * Hanover, Gdansk, Lyon, Venice => "BMW 5 Series", "Ford KA", "Mercedes S-Class Cabriolet", "Smart"
> * Marseille, Cracow, Naples, Munich => "BMW 5 Series", "Ford KA", "Kia Sorento", "Mazda MX-5"
> 
> In a real-life scenario, this function should take both dates and the location into consideration.

Here is the required configuration to call this cloud function:

```json
"endpoint": https://baas.kinvey.com/rpc/kid_r1275v_H4/custom/GetAvailableCars
"method": POST
"headers": {
  "Authorization": "Basic a2lkX3IxMjc1dl9INDo5YWYxOTcxZTFlY2U0NTNhYWUwOTQ2MzZlYmM5MGJlNw=="
},
"body": { "city": "Berlin" }
```

Here is an example on how to call this function in Postman (the image doesn't show the Headers config):

![](./img/Postman-GetAvailableCars.png?raw=true)

#### Add Custom Validation for Cars

Implementing this validation is quite simple, you need to:

1. Find the `car` step
2. Add `Reactions` => start typing **rea** and select `reactions`
3. Add `Validations` => start typing **val** and select `validations`
4. Add `Webhook Validation` => start typing **vawe** and select `validation-custom-webhook`
5. Your code should look like this:

  ```json
  "reactions": {
      "validations": [
        {
          "type": "custom",
          "parameters": {
            "condition": "{{}}",
            "data-source": {
              "endpoint": "https://"
            }
          },
          "error-message": [
            "Invalid value message"
          ]
        }
      ]
    }
  ```
6. Update the `data-source` object using the configuration provided above.

  > Note, that the `body` should be passed as a `payload` object.
  
  Like this:

  ```json
  "data-source": {
      "endpoint": "https://baas.kinvey.com/rpc/kid_r1275v_H4/custom/GetAvailableCars",
      "method": "POST",
      "headers": {
        "Authorization": "Basic a2lkX3IxMjc1dl9INDo5YWYxOTcxZTFlY2U0NTNhYWUwOTQ2MzZlYmM5MGJlNw=="
      },
      "payload": {
        "city": "{{city}}"
      }
    }
  ```

7. Narrow down the returned result from `GetAvailableCars` - to return carIDs.
  
  If you have a look at the result returned from `GetAvailableCars` (see the Postman screenshot above), you will notice that it is a JSON object, with a `cars` array where each contains a `carID` property.
  
  ```json
  {
      "cars": [
        {
          "carID": "car03",
          "model": "Kia Sorento",
          "price": 45,
          "class": "Standard"
        },
        {
          "carID": "car04",
          "model": "Mazda MX-5",
          "price": 53,
          "class": "Luxury"
        }
      ]
    }
    ```
  
  You need to extract all `carID` values. This is done with a `selector`, like this:
  
  ```json
  "selector": "$.cars[:].carID"
  ```
  
  Here is how it works:
  
  * first drill in to the `cars` property => `$.cars`
  * then get all properties from each car => `[:]`
  * and narrow down the properties to `carID` only => `.carID`

  Add the following `selector` to your `data-source` => `"$.cars[:].carID"`.
  
  Here is how the `data-source` should look like:
  
  ```json
  "data-source": {
      "endpoint": "https://baas.kinvey.com/rpc/kid_r1275v_H4/custom/GetAvailableCars",
      "method": "POST",
      "headers": {
        "Authorization": "Basic a2lkX3IxMjc1dl9INDo5YWYxOTcxZTFlY2U0NTNhYWUwOTQ2MzZlYmM5MGJlNw=="
      },
      "payload": {
        "city": "{{city}}"
      },
      "selector": "$.cars[:].carID"
    }
  ```
 
8. Update the condition to check if the selected **car** `id` is in the filtered response array.

  > Note, it is `car.id`, because we are checking the `id` property of the selected `car` object.

  ```json
"condition": "{{$in car.id _response}}",
```

9. Update the `error-message`

  ```json
  "Unfortunatelly, {{car}} is not available in {{city}}. Try another car."
  ```
  
10. Here is how the whole reaction should look like:

  ```json
    "reactions": {
      "validations": [
        {
          "type": "custom",
          "parameters": {
            "condition": "{{$in car.id _response}}",
            "data-source": {
              "endpoint": "https://baas.kinvey.com/rpc/kid_r1275v_H4/custom/GetAvailableCars",
              "method": "POST",
              "headers": {
                "Authorization": "Basic a2lkX3IxMjc1dl9INDo5YWYxOTcxZTFlY2U0NTNhYWUwOTQ2MzZlYmM5MGJlNw=="
              },
              "payload": {
                "city": "{{city}}"
              },
              "selector": "$.cars[:].carID"
            }
          },
          "error-message": [
            "Unfortunatelly, {{car}} is not available in {{city}}. Try another car."
          ]
        }
      ]
    }
  ```
    
#### Test

Now you should be able to test car validation of the `rent-car` conversation.

> To skip the steps for dates, you can start with an expression like: *"Rent a car from today to tomorrow"*

1. Send: *Rent a car from today to tomorrow in Germany*
2. Select **Berlin**
3. Select **BMW 5 Series** or **Ford KA** => you should get an error message.
4. Select any other car => this should work fine.

![](./img/car-validation-demo.gif?raw=true)

<!--
##### Venice

1. Send: *Rent a car from today to tomorrow in Italy*
2. Select **Venice**
3. Select **Kia Sorento** or **Mazda MX-5** => you should get an error message.
4. Select any other car => it should work fine.
-->

## Ambiguities

The daily language we speak is full of ambiguous statements. Often you will come out from a meeting wondering wether the other person meant one thing or the opposite.

Similarly, chatbots suffer from a level of confusion with ambiguous statements. They can even get confused when multiple domain entities are used in a sentence. For example when a user says:

*"I want to rent a car. I am traveling between France and Germany"*

The chatbot will recognise two countries: **France** and **Germany**. It is not immediately obvious which country should be picked.

Scenarios like this can be resolved with an **Ambiguities Reaction**. This reaction allows the chatbot to ask about the ambiguous entity, and allows the user to select a specific value.

### Add ambiguities resolver to Country

The Car Rental chatbot should be able to handle ambiguity around the selection of a country. Add a **Ambiguities Resolver** to the **Country**, like this:

1. Find the Country step
2. Add `ambiguities` to the `reactions` => start typing **am** and select `ambiguities`
3. Add these values:
  * `"I am not sure, which country do you need a car for. Can you select one of the following?"`
  * `"You mentioned these countries. Which country do you need a car for?"`


#### Solution

The whole reaction should look like this:

```json
"reactions": {
  "acknowledgements": [
    "Cool, I see you picked {{country}}.",
    "I understand that you need a car in {{country}}."
  ],
  "ambiguities": [
    "I am not sure, which country do you need a car for. Can you select one of the following?",
    "You mentioned these countries. Which country do you need a car for?"
  ]
}
```
  
#### Test

Now you should be able to test some statements that mention more than one country.

Try starting the conversation with one of these:

* *I want to rent a car. I am traveling between France and Germany*
* *I am flying from France to Germany, I want to rent a car*

![](./img/country-validation-demo.gif?raw=true)


## Suggestions

Chatbots are most suitable to automate repetitive business processes for users. The expectation is that a chatbot learns your preferences over time, and gives you an option to just confirm or update the existing info.

This is done with a **Suggestions Reaction**. This reaction allows the chatbot to take note of the most recent values provided by the same user, and provide a suggestion instead of a query. This is especially useful when the answer has to be typed like an address, or a phone number.

It takes for the user to use the same value twice in a row for the value to be used as a valid suggestion.
Additionally, when suggestions are used together with **Quick Reply** buttons. Then the most recently value will be displayed first. Meaning:

* choose a value once => next time that value will come first
* choose the same value again => next time that value will be used for a suggestion question

### Add suggestions for country

The Car Rental chatbot should take note of the country each user tends to travel to, and if they keep choosing the same country, then the chatbot should be able to suggest the country instead of asking to choose one.

Add a **Suggestion** to the **Country** **reactions**, like this:

1. Find the Country step
2. Add `suggestions ` to the `reactions` => start typing **su** and select `suggestions`
3. Add these values:
  * `"I see you tend to rent cars in {{country}}. Do you need a car in {{country}}?"`
  * `"Based on your previous choices. Would you like a car in {{country}}?"`

#### Solution

The whole reaction should look like this:

```json
"reactions": {
  "acknowledgements": [
    "Cool, I see you picked {{country}}.",
    "I understand that you need a car in {{country}}."
  ],
  "ambiguities": [
    "I am not sure, which country do you need a car for. Can you select one of the following?",
    "You mentioned these countries. Which country do you need a car for?"
  ],
  "suggestions": [
    "I see you tend to rent cars in {{country}}. Do you need a car in {{country}}?",
    "Based on your previous choices. Would you like a car in {{country}}?"
  ]
}
```

#### Test

Now you should be able to test country suggestions.

In order to test the suggestions, you need to use the same test window for the whole test. This is because each time you open a new test window, the chatbot will treat it as a new user. This is different for a fully deployed chatbot instance, as there is a mechanism in place to recognise a returning user.

Additionally, you need to complete the conversation twice, in order to test suggestions. Just selecting a value and then calling **restart** will not work.

Try the following steps:

1. Send: *Rent a car from today to tomorrow*
2. Select **Poland** - note this should appear as the last option in the list
3. Send: *Warsaw*
4. Select: **Mercedes**
5. Send: *Rent a car from today to tomorrow*
6. Select **Poland** - this time it should appear as the first option
7. Send: *Warsaw*
8. Select: **Mercedes**
9. Send: *Rent a car from today to tomorrow*
10. You should be presented with a suggestion message plus **Yes** and **No** buttons
11. Select: **Yes** - this should result in **Poland** as an output
12. Keep going

![](./img/country-suggestions-demo.gif?raw=true)

## Homework

Before you move on to the next chapter, here a few challenges for you:

### Add new email step to the `rent-car` conversation

Add a question step to collect an email address with:

* acknowledgment message
* email validation
* suggestions

### Advanced: Add picture capture to the `drop-off-location` conversation

Add a new step to the `drop-off-location` conversation, which should allow users to submit a picture of where they dropped off the car.

Like this:

```json
{
  "type": "question",
  "entity": "carPicture",
  "entity-type": "File",
  "messages": [
    "Can you take a picture of where you parked the car?"
  ]
},
```

Now add a picture validation that checks if the picture contains a **car**.

For more info see [the documentation](https://docs.nativechat.com/docs/1.0/cognitive-flow/validation.html#image).
