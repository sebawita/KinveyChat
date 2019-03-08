# Conversational UI

In the previous chapter, you've learned how to deal with data and how to allow the user to pass in specific pieces of data. However, this can get quite tricky, when we expect a limited number of answers. For example, if the car rental service only operates in four countries, we shouldn't let the user guess, which countries are available. Instead, we should use conversational UI to guide the user through the process.

With Conversational UI we not only provide a list of available values, but we can also provide additional information that goes with each item. When listing the available cars, we can also provide the price at the same time.

In this chapter, you will enhance the existing questions steps for **Country**, **City** and **Car** with Conversational UI to improve the booking process.

## Quick Reply

The simplest of Conversational UI elements is the **Quick Reply** component.

It displays available answers as a set of buttons. It allows the user to click on one of the buttons, or type the answer.

![quick-reply-example](./img/quick-reply-example.png?raw=true)

### Add Quick Reply to the Country Step

Add **Quick Reply** to the **Country** step, like this:

1. Find the **Country** step and go to the end of the step
2. Add `display` => start typing **di** and select `display`
3. Add `type` => start typing **ty** and select `type
4. Add `quick-reply` => start typing **qui** and select `quick-reply`

You should get something like this:

```json
"display": {
  "type": "quick-reply"
}
```

> Note: always use code snippets, they will save you a lot of keystrokes
>
> ![](./img/quick-reply-example.gif?raw=true)

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
  }
},
```

The Chatbot will use all the entries allocated to `"entity-type": "Country"`, and display each item as a button.

> Note: **Quick Reply** is great when we are dealing with a small list of items. For anything larger than 10 items it is best to display as a list picker (see **Single Select**).

### Test

Save the cognitive flow, and ask the bot to *"rent a car"*.

Now the **Country step** should look like this:

![](./img/quick-reply-demo.gif?raw=true)

## Single Select

The next Conversational UI component is **Single Select**.

Single Select displays a list of available items in a modal window.

### Add Single Select to the City Step

Add **Single Select** to the **City** step, like this:

1. Find the **City** step and go to the end of the step
2. Add `display` => start typing **di** and select `display`
3. Add `type` => start typing **ty** and select `type
4. Add `single-select` => start typing **sin** and select `single-select `
5. Add `title` => start typing **ti** and select `title` then add `Find a city` text
6. Add `button-text` => start typing **bu** and select `button-text` then add `Select this city` text

You should get something like this:

```json
"display": {
  "type": "single-select",
  "title": "Find a city",
  "button-text": "Select this city"
},
```

* `title` - is used for the text of the button that opens the list
* `button-text` - is used for the text of the button that confirms the items selection

The whole **City** step should look like this:

```json
{
  "type": "question",
  "entity": "city",
  "entity-type": "City",
  "messages": [
    "In which city are you looking for a car?"
  ],
  "display": {
    "type": "single-select",
    "title": "Find a city",
    "button-text": "Select this city"
  }
},
```

### Test

Save the cognitive flow, and ask the bot to *"rent a car in France"*.

Now the **City step** should look like this:

![](./img/single-select-demo.gif?raw=true)

## Display data with data-source

Displaying items as in a list is very simple, however, the current workflow has a small flaw. The current list allows the user to select a city from any country, which doesn't make sense if they chose France as the country.

The solution to this is quite simple. Instead of getting the data directly from the Entity, you can make a query and display only the returned items.

### Make City Single Select display correct records
 
To make the Single Select return specific items, you need to add two pieces of configuration to the `"display"` object:

* `data-source` - consist of the REST command to query the data
* `template` - instructs the chatbot on how to display the items

#### Step 1 - add data-source

In the configuration of the City Entity you used this endpoint - `https://baas.kinvey.com/appdata/kid_r1275v_H4/Offices` - to get all office locations. In order to filter it by country, you can provide a query object requesting a specific country, like this:

`https://baas.kinvey.com/appdata/kid_r1275v_H4/Offices?query={"country": "France"}`

To make it more dynamic, you can use the Mustache syntax, and replace `France` with `{{country}}`, like this:

`https://baas.kinvey.com/appdata/kid_r1275v_H4/Offices?query={"country":"{{country}}"}`

Finally, you need to escape the double quotes with a backward slash (`"` => `\"`):
`"https://baas.kinvey.com/appdata/kid_r1275v_H4/Offices?query={\"country\": \"{{country}}\"}"`

Add the `"data-source"` object to the `"display"` object:

```json
"data-source": {
  "endpoint": "https://baas.kinvey.com/appdata/kid_r1275v_H4/Offices?query={\"country\": \"{{country}}\"}",
  "method": "GET",
  "headers": {
    "Authorization": "Basic a2lkX3IxMjc1dl9INDo5YWYxOTcxZTFlY2U0NTNhYWUwOTQ2MzZlYmM5MGJlNw=="
  }
}
```

#### Step 2 - add template

Next, in order for the chatbot to understand how to display the results of the query, you need to add a `template` object to the `display` object.

```json
"template": {
  "title": "{{city}}",
  "subtitle": "{{country}}",
}
```

A template is made of: 

* `title` - field to display, but also the value returned from the selection
* (optional) `subtitle` - the text displayed under the item
* (optional) `value` - the value that should be returned for the selected item
* (optional) `image` - the field to use for an image

### Understanding `title` vs `value`.**

It is important to understand that the value returned from the selection, must be matched to the items defined in the Entity (in the training tab). So, if you change the title to be: 

`"title": "Office in {{city}}"`

Then selecting **Office in Berlin** will fail, as the City Entity, doesn't contain such record.

In this case, you need to use the `value`, which should provide the property that the chatbot could match to, like this:

```json
"template": {
  "title": "Office in {{city}}",
  "subtitle": "{{country}}",
  "value": "{{city}}"
}
```

### Add image to the template

To make the list fancier you can add an image with a flag. Although the records don't return a URL for an image, you can use [https://www.countries-ofthe-world.com/flags-normal/flag-of-france.png](https://www.countries-ofthe-world.com/flags-normal/flag-of-france.png). Just use the Mustache syntax and replace `france` with `{{country}}`, like this:

```json
"template": {
  "title": "Office in {{city}}",
  "subtitle": "{{country}}",
  "value": "{{city}}"
  "image": "https://www.countries-ofthe-world.com/flags-normal/flag-of-{{country}}.png"
}
```

The whole **City** step should look like this:

```json
{
  "type": "question",
  "entity": "city",
  "entity-type": "City",
  "messages": [
    "In which city are you looking for a car?"
  ],
  "display": {
    "type": "single-select",
    "title": "Find a city",
    "button-text": "Select this city",
    "data-source": {
      "endpoint": "https://baas.kinvey.com/appdata/kid_r1275v_H4/Offices?query={\"country\": \"{{country}}\"}",
      "method": "GET",
      "headers": {
        "Authorization": "Basic a2lkX3IxMjc1dl9INDo5YWYxOTcxZTFlY2U0NTNhYWUwOTQ2MzZlYmM5MGJlNw=="
      }
    },
    "template": {
      "title": "{{city}}",
      "subtitle": "{{country}}",
      "image": "https://www.countries-ofthe-world.com/flags-normal/flag-of-{{country}}.png"
    }
  }
},
```

### Data Source result as an Array

The Data Source always expects an array. 

However, some services return results as a JSON object. For example, you could get a response like this:

```json
{
  code: 200,
  result: {
    cities: [ 'Berlin', 'Paris', 'Rome', 'Warsaw']
  }
}
```

The data that we need is under `result.cities`. In this case, you can use a `selector` property and provide a [JSONPath](https://goessner.net/articles/JsonPath/index.html) to the array. 

So, the solution to the above example would be:

```json
"data-source": {
  "endpoint": "https://another.api.com/cities",
  "method": "GET",
  "selector": "$.result.cities[:]"
},
```

### Test

Save the cognitive flow, and ask the bot to *"rent a car"* and then select a country.

Now the conversation should look like this:

![](./img/single-select-dynamic-demo.gif?raw=true)

## Carousel

The next the Conversational UI reply templates and perhaps the most elegant one is the **Carousel**. Which allows you to display items in a fancy carousel. This component is best used when you have a few options supported with nice images.

### Add Carousel to the Car Step

In the previous chapter, in the homework section, you were asked to add a Car Entity and a Question for a Car.

Now, you will add a **Carousel** Component to the Car step.

The configuration is identical to that of a **Single Select** Component. The only difference is the Display type:

```json
"display": {
  "type": "carousel",
  ...
}
```

### Challenge

Can you update the step so that it:

* gets the cars from: `"endpoint": "https://baas.kinvey.com/appdata/kid_r1275v_H4/Cars"`
* displays each item:
    * `name` as the main item,
    * `price` as a subtitle
    * `imageUrl` as an image

> Hint: You can use a `$currency` formatter to display the price in a nice format. For example:
> 
> `{{$currency price 'USD'}}`.
> 
> You can read more about formatters in the [documentation](https://docs.nativechat.com/docs/1.0/cognitive-flow/formatting.html).

### Solution

Here is how the full step should look like:

```json
{
  "type": "question",
  "entity": "car",
  "entity-type": "Car",
  "messages": [
    "What car would you like?"
  ],
  "display": {
    "type": "carousel",
    "title": "Find a car",
    "button-text": "Select this car",
    "data-source": {
      "endpoint": "https://baas.kinvey.com/appdata/kid_r1275v_H4/Cars",
      "method": "GET",
      "headers": {
        "Authorization": "Basic a2lkX3IxMjc1dl9INDo5YWYxOTcxZTFlY2U0NTNhYWUwOTQ2MzZlYmM5MGJlNw=="
      }
    },
    "template": {
      "title": "{{name}}",
      "subtitle": "{{$currency price 'USD'}}",
      "image": "{{imageUrl}}"
    }
  }
}
```

### Test

Save the cognitive flow, and ask the bot to *"rent a car"* and go through the whole flow.

The conversation should look like this:

![carousel](./img/carousel-dynamic-demo.gif?raw=true)

## Pickers

Kinvey Chat also offers location and date pickers to help users provide a location or a date in an interactive manner. The pickers not only make it easier, but they also make sure that the data is provided in the right format.

> Note, the pickers extend the functionality of a `question` step.

## Date Picker

The date picker allows the user to select a date on an interactive calendar, which returns the date in a specific date format.

<!--### Add steps to capture the pick-up and drop-off dates-->

To complete the booking process you also need to ask the customer for the pick-up and drop-off dates.

To do that you need to add two question steps.

Add these steps before the **Country** step, so that the chatbot would start by asking for the dates.

> By now, you should be well versed with adding question steps. Try to implement these steps without copy & pasting the solution.

### Add the pick-up Date question
For the first question step, set:

* **Entity** to `startDate`,
* **Entity Type** to `Date`
* The **Prompt message** to `When would you like to pick up the car?`

Then add a display configuration like this one:

```json
"display": {
  "type": "date-picker",
  "title": "Find pick-up date",
  "button-text": "Select"
}
```

* `title` - the button text that opens the calendar
* `button-text` - the button text that confirms the calendar choice 

### Add the drop-off Date question
Add the second question step after the first one, set:

* The **Entity** to `endDate`
* The **Prompt message** to `When would you like to drop off the car?`
* The **Button text to show the calendar** to `Find drop-off date`

### Solution

The two steps should look like this:

```json
{
  "type": "question",
  "entity": "startDate",
  "entity-type": "Date",
  "messages": [
    "When would you like to pick up the car?"
  ],
  "display": {
    "type": "date-picker",
    "title": "Find pick-up date",
    "button-text": "Select"
  }
},
```

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
  }
},
```

### Test

Now you should be able to test the new flow of the `rent-car` conversation.

Try the following conversations:

> Remember, you can send the *restart* command, each time you want to try a different conversation.

#### Pickers only

1. Send: *I want to rent a car*
2. Open the calendar and select a pick-up date
3. Open the calendar and select a drop-off date

#### Mixed

1. Send: *I want to rent a car*
2. Send: *I need a car tomorrow*
3. Open the calendar and select a drop-off date

#### Start with the Intro conversation

1. Send: *I want to rent a car today*
2. Open the calendar and select a drop-off date

<!--TODO: Need a gif for this-->

## Location Picker

The Location Picker is a really useful widget that allows users to select a specific location on a map. The Location Picker returns a `Location` object, which contains `latitude` and `longitude` properties.

The location picker is very easy to configure. You just need a question step with the `"entity-type": "Location"`, and a `display` object of `"type": "location-picker"`, like this:

```json
{
  "type": "question",
  "entity": "location",
  "entity-type": "Location",
  "messages": [
    "Where is it?"
  ],
  "display": {
    "type": "location-picker",
    "title": "Pick a location",
    "button-text": "The car is here"
  }
},
```

## Homework

The car rental company offers a special service, where the customers can drop off their car anywhere in the city. The chatbot needs a new conversation that handles this specific scenario.

Add a new conversation called `drop-off-location`, which should be triggered with one of these expressions:

* *I dropped off the car*,
* *Can you pick up the car?*,
* *The car is ready for pickup*

The `drop-off-location` conversation should work in two steps:

### Step 1

In the first step, ask the user about the location of the car:

* *Where did you leave the car*
* *Where can we find the car?*,
* *Can you show us on the location of the car on the map?*

Then add a location picker, which should show when the user clicks on *Pick a location* and saves when the user clicks on *The car is here*.

### Step 2

In the second step, just print the following message in 3 lines:

* *Thank you*
* *We will pick up the car from: {{$json location}}*
* *Have a nice day*


### Solution

Here is the full code for the `drop-off-location` conversation:

```json
"drop-off-location": {
  "type": "goal",
  "steps": [
    {
      "type": "question",
      "entity": "location",
      "entity-type": "Location",
      "messages": [
        "Where did you leave the car?",
        "Where can we find the car?",
        "Can you show us on the location of the car on the map?"
      ],
      "display": {
        "type": "location-picker",
        "title": "Pick a location",
        "button-text": "The car is here"
      }
    },
    {
      "type": "message",
      "messages": [
        [
          "Thank you.",
          "We will pick up the car from: {{$json location}}",
          "Have a nice day"
        ]
      ]
    }
  ]
},
```
