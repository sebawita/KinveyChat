# Entities

So far, you've learned how to add a simple conversation that just responds with a message.

However, a well functioning chatbot should be able to understand the key entities that it should be dealing with, like **Country**, **City**, **Car**, **Insurance** etc.

This is very important when we want the chatbot to understand expressions like:

\<user says>: "I want a car in **London**. I like **Ford Focus**"

Bot should understand:

* Conversation: **rent-car**
* City: **London**
* Car: **Ford Focus**

## Entities - Static Data

There are different kinds of Entity Types. One of the simplest ones is `Keyword` `Static` type.

Using a `Keyword Entity` allows you to train your chatbot with a list of values and synonyms. Your chatbot will look for these keywords in the user input. If it finds a match, it will be able to recognise it as the Entity Value.

When you are dealing with a small set of values that doesn't change too often. You can manually provide the values as a `Static` list.

### Add static data for countries

In the chatbot scenario, we need the chatbot to be able to recognise the names of the countries where our service operates.

It is time for you to create your first entity for **Country**:

1. Open the **Training** tab
2. Press the **Add new** button
3. Set the **name** to: **Country**
4. Keep **Lookup strategy** as **Keyword** and **Traning Data Source** as **Static**
5. Press **Create**

Now you need to populate the Country entity with the values for the countries supported by our car rental company:

1. Press the **Add value** button
2. Set **value** to **Germany**
3. Set **Synonyms** to **Deutschland**
4. Press the **Save** button
5. Repeat for the rest of the countries

Here is the list of countries (with their synonyms) you should support:

* Germany: Deutschland
* France
* Italy: Italia
* Poland: Polska

<!--Add a gif showing how this is done -->

> Note, You need to be careful with short Keywords and short aliases. As sometimes short aliases could match to other words. If you had an alias **FRA** for France, then this could match to Fra in another word. For example:
> 
> <user says> "I need a car in Frankfurt".
>
> The chatbot could match **FRA** in the word **Fra**nkfurt, and match it to **France**.

### Add a conversation step to find the country
<!-- no help with selecting the items -->

Now you can use the Country entity to allow the user to tell the bot in which country they want the car.

At this point, you should have the `rent-car` conversation ready. See homework from [link to homework #homework thing](./tutorial-02-first-conversation.md).

Now, you are going to add a `question` step to the `rent-car` conversation. Question steps are used to capture specific information from the user. The question will ask the user for a Country and save the response for later.

1. Open the **Cognitive Flow** tab 
2. Find the `rent-car` conversation
3. Go to the end of the `steps` array, which should have a message step in there
4. Add a comma at the of the message step and start typing `step` and select the `step-question` snippet. This should output the following code:

  ```json
    {
      "type": "question",
      "entity": "entity-name",
      "entity-type": "",
      "messages": [
        "How to ask for entity?"
      ]
    }
  ```
 *  `entity` - is like a name of a variable where you want to store the result.
 *  `entity-type` - is the type of the entity, it can be one of the [built-in entity types](https://docs.nativechat.com/docs/1.0/nlp-training/entities.html#built-in-entity-types) like `Date`, or it can be a type defined by you. It tells the chatbot what to look for to extract the value
 *  `messages` - contains message prompts asking the user to provide the value

5. Update the above 3 properties to the below values:

 * entity - `"country"`
 * entity-type - `"Country"`
 * messages -  `[ "Which country are you travelling to?" ]`

6. Save

The whole conversation, should look like this:

```json
"rent-car": {
  "type": "goal",
  "steps": [
    {
      "type": "message",
      "messages": [
        "Great, let me help you find a car for you."
      ]
    },
    {
      "type": "question",
      "entity": "country",
      "entity-type": "Country",
      "messages": [
        "Which country are you traveling to?"
      ]
    }
  ]
},
```

<!--gif this is how I did this-->

### Test
<!--Explain the debug console-->
<!--Show two flows:
1. I want to rent a car
2. I want to rent a car in Germany -> show that the data is picked up, but the user needs a small confirmation on it => acknowledgments.
-->

Now is a moment to test the new part of the conversation. 

Open the test window and try the following conversations:

#### Test: Two step conversation

1. Send: *"I want to rent a car"*.
2. Wait for the Country prompt.
3. (Optional) Send: *"I am going to Spain"*
  * the bot should respond with: *"I am not sure I understood what you said."*
4. Send: *"I am going to Germany"*.
5. Expand `Understanding` after the last message in the. It should contain a `Country` object with `value` set to **Germany**, like this:

  ```json
    "Country": [
      {
        "value": "Germany",
        "confidence": 1,
        "_start": 14,
        "_end": 21,
        "_body": "Germany",
        "_entity": "Country"
      }
  ],
  ```
 

#### Test: One step conversation

1. Send: "I want to rent a car in France"
2. Expand `Understanding` after the last message in the console. Now the `Country.value` should be **France**

## Entities: Dynamic Data

You can also populate the entity values from a backend. This is done using a `Keyword` `Dynamic` Entity.

A `Dynamic` entity allows you to train the chatbot based on a data returned from a REST call.
So, instead of typing a value for each item, you get your data from a REST call.

### Backend: Offices

For the Car Rental, we already have a list of office locations stored in a [Kinvey Backend](https://www.progress.com/kinvey) collection.

You can access the data by making a REST call with the following details:

```json
{
  "endpoint": "https://baas.kinvey.com/appdata/kid_r1275v_H4/Offices",
  "method": "GET",
  "headers": {
    "Authorization": "Basic a2lkX3IxMjc1dl9INDo5YWYxOTcxZTFlY2U0NTNhYWUwOTQ2MzZlYmM5MGJlNw=="
  }
}
```

![](./img/kinvey-offices.png?raw=true)

Each record contains **country**, **city**, and **localName** (how the city is called in the local language, which can be used as an alias).

### Add Dynamic Data for Cities

It is time for you to create your second entity for **City**:

1. Open the **Training** tab
2. Press **Add new**
3. Set the `Name` to: **City**
4. Keep the `Lookup strategy` as **Keyword**
5. Set the `Traning Data Source` to **Dynamic**
6. Provide the following configuration:
	* `Endpoint URL`: **https://baas.kinvey.com/appdata/kid_r1275v_H4/Offices**
	* `Headers`:
		* `key`: **Authorization**
		* `value`: **Basic a2lkX3IxMjc1dl9INDo5YWYxOTcxZTFlY2U0NTNhYWUwOTQ2MzZlYmM5MGJlNw==**
	* `Value template`: **{{name}}**
	* `Synonym templates`: **{{localName}}**
7. Press **Test**
  * This should return an array with a list of city names
8. Press **Create**

#### Mustache template

You might be wondering about the syntax used for the Value template: `{{city}}`.
Kinvey Chat uses Mustache template system, which allows mixing text with values returned from an object.

For example, a template `City: {{name}}`, would return an array of items like:

```json
[ "City: Naples", "City: Lyon", "City: Gdansk", ...]
```

You can also use this with any message text returned by the chatbot. For example, you could have a message step like this:

```json
{
  "type": "message",
  "messages": [
    "You picked {{city}} in {{country}}"
  ]
}
```

### Add a conversation step for city

Next, add another question step to the `rent-car` conversation, which should ask the user to select the city. Use the following values:

* Entity: `city`
* Entity Type: `City`,
* Message: `In which city are you looking for a car?`


### Test


You should test the new conversation part. Try these conversations:

#### Step by step

1. Send: *I want to rent a car*.
2. Wait for the Country prompt.
3. Send: *France*.
4. Wait for the City prompt.
5. Send: **Lyon**
6. Check the `Understanding` for both the Country and the City in the console.

#### All at once

1. Send: *I want to rent a car in Deutschland, Berlin*.
2. Check the `Understanding` for both the Country and the City in the console.

### Retraining the chatbot with a Dynamic Entity

It wouldn't be surprising at all if the rental company decided to expand and add more cities to their offering. This would however mean that the chatbot wouldn't be able to recognise the new cities.

This is a quite common scenario where the dynamic data changes. When that happens you just need to retrain the chatbot with the new data.

You could really easily trigger the update process manually:

1. Open the **Training** tab
2. Open the dynamic entity that needs updating, for example **City**
3. Press **Update** and confirm the action with another **Update**
4. Give it a bit of time and you are good to go.

## Regex data

There are also scenarios where you want to be able to match to an entity like a driving license number or a car registration plate, but obviously you cannot list every single driving license number (plus this would make a very slow process of trying to match millions of potential values).

This can be done by defining an entity type as a regular expression.

### Driving License Number

The chatbot should be able to ask for the users Driving License Number. For the sake of simplicity, lets say that the license number is made of:

* 3 letters from the name
* 4 digits from the month and year of birth
* 2 letters from the country code

For example:
* Stefan => STE
* born in Dec 1986 => 1286
* from Germany => DE

The full license # should be: **STE1286DE**

A regular expression for this would look like this: `[A-Za-z]{3}[0-9]{4}[A-Za-z]{2}`

> You can learn more about regular expressions from [MDN web docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions).

### Add regex entity Driving License Number

Now you will add a third entity type to your chatbot training.

1. Open the **Training** tab
2. Press **Add new**
3. Set the `Name` to: **DrivingLicenseNumber**
4. Set the `Lookup strategy` to **Regex**
5. Set the `Pattern` to **[A-Za-z]{3}[0-9]{4}[A-Za-z]{2}**
6. Press **Create**

### Test

We don't need a question step to test if an Entity Type works.
The chatbot engine always makes an attempt to match to any of the available Entity Types.

You can easily test the following expressions:
 
* *EVA1093FR* => should match: EVA1093FR
* *My license is MAT1001UK* => should match: MAT1001UK
* *This license AAA123BB* => shouldn't match anything

And for each, expand the **Understanding**. If a match occurs, you should find an object like:

```json
"DrivingLicenseNumber": [
  {
    "value": "EVA1093FR",
    "confidence": 1,
    "_start": 14,
    "_end": 23,
    "_body": "EVA1093FR",
    "_entity": "DrivingLicenseNumber"
  }
]
```

## Homework

As a homework add a Dynamic Entity Type for a car.
This is so that the chatbot could recognise expressions like: *I want to rent a* **Ford KA**.

### Part 1

* The type `Name` should be called `Car`.
* The data should be loaded from:

  ```json
{
      "endpoint": "https://baas.kinvey.com/appdata/kid_r1275v_H4/Cars",
      "method": "GET",
      "headers": {
        "Authorization": "Basic a2lkX3IxMjc1dl9INDo5YWYxOTcxZTFlY2U0NTNhYWUwOTQ2MzZlYmM5MGJlNw=="
      }
}
```

* The `value` should come from **name**
* The `synonym` should come from **short-name**

### Part 2

Add a question step to `rent-car` to ask for a Car.
