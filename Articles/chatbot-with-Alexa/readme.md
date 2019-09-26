## Intro

In the last few years, voice assistants changed the way we interact with our devices. To some, this was a somewhat strange experience, but many more felt that this was a more natural way of interacting with most services provided by our phones and computers.

Many companies took this as an opportunity to enhance the experience for their customers and provided some of their core functionality via a set of voice commands, thus becoming more customer-friendly and strengthening their brand.

Voice chatbots are not only popular because of the convenience of talking to a device, but also they come very *handy*, when your hands are busy (i.e. when you are cooking ) or more importantly when your eyes should be *focused* elsewhere (i.e. when you are driving).

In this article, you will learn how to build your own voice assistant. You will use [Kinvey Chat](https://www.progress.com/kinvey/chat) to build an **Alexa skill** in a few easy steps.

## Scenarios

To build something worthwhile we need a *"real"* customer and a set of *"real"* scenarios.

A telecommunications company called **C-Mobile** provides various services to its customers. They identified a need to assist their pay-as-you-go customers with various services around topping up their accounts and buying data packages.

### Scenario #1 — check my balance

As a customer, I want to be able to ask my Alexa device to check my balance.

Alexa should respond with the amount of credit I have left on my account together with the remaining minutes, texts and data.

### Scenario #2 — top-up

As a customer, I want to be able to top up my account.

Alexa should guide me through the required steps, provide with a list of options, and at the end provide the updated credit on the account.

### Scenario #3 — buy a data package

As a customer, I want to be able to buy additional data packages.

Alexa should guide me through the required steps, provide me with a list of data packages, which should be paid for with the credits on the account.

### Assumptions / simplification of the scenario

Usually, when you build applications for services like this, you will have a functionality that allows your user to log in to their specific account. In the case of voice assistants, this usually is handled through the admin portal, which then automatically associates the user account with their app.

This is a whole topic on its own, which I am planning to cover in another blog post. Instead, I will ask you to hardcode the user id in the project.

## API

**C-Mobiles** provides you with a very simple API, which allows you to manage individual accounts.

You can access all of the calls from the following https://demoapis.com/cmobile/<function-name>:

Please note that all API calls expect an **id** parameter. Please pick a number 9-11 digits long, and use it for all of your calls. This way when you top up an account and then ask for the account status, you will see the balance updated.

Also, if you happen to use a number that someone else is also using (this is not very likely unless you use the demo id), just pick a different number and you should be fine.

Here is the list of available functions:

### getAccountStatus

`getAccountStatus` allows you to ask for the Account balance, minutes, SMS, and data.

**Signature:**

```
/availableCards/:id
```

You need to provide your selected account **id**.

**Example:**

Here is an example of how to get the account balance for id **012345678**

[https://demoapis.com/cmobile/getAccountStatus/012345678](https://demoapis.com/cmobile/getAccountStatus/012345678)

**Result:**

```json
{
  "balance": 12,
  "minutes": 100,
  "SMS": 200,
  "megabytes": 29,
  "data": "29 MB"
}
```

### topUp

`topUp` allows you to top up your account balance, as a result, the balance will increase and you will receive an updated account status.

**Signature:**

```
/topUp/:id?val={amount}&card={4-digits}
```

You need to provide your selected account **id**.

Additionally, you need to provide two query params:

- `val` — with the amount you want to top up
- `card` — last 4 digits of the card to use, this param is not used for anything, but it is here to simulate the scenario.

**Example:**

Here is an example of how to top the account id **012345678** with **10** credits using the card ending with **9876** (*don't worry no card payment will actually happen* 😉).

[https://demoapis.com/cmobile/topUp/012345678?val=10&card=9876](https://demoapis.com/cmobile/topUp/012345678?val=10&card=9876)

**Result:**

```json
{
  "SMS": 200,
  "balance": 22,
  "minutes": 100,
  "data": "29 MB",
  "megabytes": 29
}
```

### availableCards/:id

`availableCads` allows you to look up all associated payment cards with the account.

**Signature:**

```
/availableCards/:id
```

You need to provide your selected account **id**.

**Example:**

Here is an example of how to check what payment cards are associated with the account id **012345678**.

[https://demoapis.com/cmobile/availableCards/012345678](https://demoapis.com/cmobile/availableCards/012345678)

**Result:**

```json
[
  {
    "card": "6575",
    "val": 6575
  },
  {
    "card": "3688",
    "val": 3688
  }
]
```

### resetAccount

`resetAccount` allows you to reset your account. This can come useful when you want to start from scratch.

**Signature:**

```
/resetAccount/:id
```

You need to provide your selected account **id**.

**Example:**

Here is an example of how to reset the account id **012345678**.

[https://demoapis.com/cmobile/resetAccount/012345678](https://demoapis.com/cmobile/resetAccount/012345678)

**Result:**

```json
{
  "balance": 12,
  "minutes": 100,
  "SMS": 200,
  "megabytes": 29,
  "data": "29 MB"
}
```

### dataPackages

`dataPackages` allows you to view available data packages for you to purchase.

For each package, you will receive a name, formatted data size, megabytes, and a price.

**Signature:**

```
/dataPackages
```

No parameters are required.

**Example:**

Here is an example of how to view available data packages.

[https://demoapis.com/cmobile/dataPackages](https://demoapis.com/cmobile/dataPackages)

**Result:**

```json
[
  {
    "name": "lite",
    "data": "512 MB",
    "megabytes": 512,
    "price": 7
  },
  {
    "name": "small",
    "data": "1 GB",
    "megabytes": 1024,
    "price": 10
  },
  {
    "name": "medium",
    "data": "3 GB",
    "megabytes": 3072,
    "price": 20
  },
  {
    "name": "big",
    "data": "5 GB",
    "megabytes": 5120,
    "price": 25
  }
]
```

### buyData

`buyData` allows you to purchase more data for your account. As a result, more data will be added to the account, and the price of the package will be deducted from the balance.

**Signature:**

```
/buyData/:id?dataPack={name}
```

You need to provide your selected account **id**.

Additionally, you need to provide a query param `dataPack` with the name of the package that you want to purchase.

**Example:**

Here is an example of how to buy a **medium** data pack for the account id **012345678**.

[https://demoapis.com/cmobile/buyData/012345678?dataPack=tiny](https://demoapis.com/cmobile/buyData/012345678?dataPack=tiny)

**Result:**

```json
{
  "SMS": 200,
  "balance": 5,
  "minutes": 100,
  "data": "0.5 GB",
  "megabytes": 541
}
```

## The flow of the tutorial

We will tackle this project in two rounds.

- **Phase 1** — Getting Started — You will build a smart part of the conversation (scenario: *Check my balance* ) and then test it as part of an Alexa skill. This part focuses mostly on making sure that your environment is set up correctly, and on getting some response from an Alexa Skill.
- **Phase 2** — Add more conversations — You will add the remaining conversations. This part focuses more on managing the conversation flow and completing the provided scenarios.

## Phase 1 — Getting started

The getting started is as simple as 1,2,3, which can be done in these really easy steps.

### Step 1 — Create a new Kinvey Chat project

First, you need to sign in to [Kinvey Chat Console](https://bots.kinvey.com/). 

>  If you don't have an account yet, you can create it [here](https://console.kinvey.com/sign-up). Don't worry, you can get started for free.

After you sign in you can create a new chatbot. <br />Press the **[+ New bot]** button:

<img src="./img/new-bot.png?raw=true" width="400">

Call the project **cmobile**.

Keep the **Bot Language** as **English**, and **Start with** as **Blank bot**.

Press the **[Create]** button.

<img src="./img/new-bot-config.png?raw=true" width="400">

This will create a simple chatbot, with a couple of super basic conversations.

#### Testing

To test it, press the **[Test]** button. Then when the chatbot loads:

- type: **hi**
- the chatbot should respond with a welcome message
- click one of the buttons or type: **Conversation 1**
- the chatbot should respond with something like: *This is conversation 1*

### Step 2 — Create your first conversation — check my balance

Now that you have a working chatbot, it is time for you to implement the first conversation, which is for the **check my balance** scenario.

When the user says *"Check my balance"*, the chatbot should call the `getAccountStatus` function, and then print out the message with the status returned from the API.

Are you ready? Let's go.

#### Add a new conversation

In the **Cognitive Flow** tab (which is the tab you arrive by default), on the right-hand side, there is a list of all conversations in this project. Click on the **conversationTwo**, which will take you to that conversation.

At the end of the `json` containing `conversationTwo` add a comma, and in the new line start typing `con`, this should trigger a little pop-up with a list of code snippets.

>  The code snippets are really useful in the process of constructing your chatbot, as they help you generate the JSON in the correct format, plus they save you a lot of time.

Select the `conversation-goal` snippet and press enter. This should output the following code:

```json
"conversation-name": {
  "type": "goal",
  "steps": [

  ]
}
```

Change `"conversation-name"` to `"check-my-balance"`.

#### Add a response message  to the conversation

Next, you need to add a **message** step to the conversation.

> Hint: If you press **tab**, the cursor will jump inside the `steps[ ]` array.

From inside the `steps[ ]` array, start typing `step` (or `stme` for *step message*) and select the `step-message` snippet. This should produce the following code:

```json
{
  "type": "message",
    "messages": [
      "Your message"
  ]
}
```

Change `"Your message"` to `"I am checking your balance"`.

The `check-my-balance` conversation should look like this:

```json
"check-my-balance": {
  "type": "goal",
  "steps": [
    {
      "type": "message",
      "messages": [
        "I am checking your balance"
      ]
    }
  ]
}
```

**Here is how to do it:**

<img src="./img/check-my-balance-1.gif?raw=true">

#### Train the chatbot to know when to use your new conversation

Next, you need to train the chatbot to understand when to trigger this conversation.

Navigate to the **Training** tab, click on **Conversation built-in**, and then press the **[Add value]** button.

Set the `value` to `"check-my-balance"` — this how the chatbot engine knows which conversation to trigger

Add the following `expressions` — this how you tell the chatbot engine when to trigger this conversation:

- *Check my balance*
- *What is my balance?*

Press the **[Save]** button.

#### Test

To test this new conversation, press the **[Test]** button and type: **Check my balance**

The chatbot should respond with your message.

**Here is how to do it:**

<img src="./img/check-my-balance-2.gif?raw=true">

#### Update the conversation to return the account status

Now that you have the conversation working, it is time to change it so that it returns *real* data.

#### Update the conversation to call the API and print a message

Remove the message step, so that the `"steps": [ ]` array is empty.

Start typing `stwe` and select `step-webhook`. Remove all the properties in the `data-source`, but keep the `endpoint`.

Update `endpoint` to `"https://demoapis.com/cmobile/getAccountStatus/012345678"`

> Note: please use a different id instead of **012345678**, or you might be clashing with other people using the same number.

Add an `"entity"` property (it is best to place it below `"type": "webhook",`) and call it `"status"` — the result of the API call will be stored in `status`.

Finally, you need to add a `"messages"` property to this step, which will contain the message to be printed when the API returns the result, like this:

```
"messages": [
  "Let's have a look at your account. You have {{status.balance}} dollars left."
]
```

> Hint: You could also, make the response multiline — which is not very important with voice chatbots, as chatbots tend to read the whole output, as one line, but it looks better when we test it in the console.
>
> To add a multiline response, we need to use an array of arrays.
>
> Here are two examples:
>
> **One Array**
>
> ```json
> "messages": [
>   "Line 1",
>   "Line 2",
>   "Line 3"
> ]
> ```
>
> This will result in one of the lines printed (or spoken) at random.
>
> **Two Arrays**
>
> ```json
> "messages": [
>   [
>     "Line 1",
>     "Line 2",
>     "Line 3"
>   ]
> ]
> ```
>
> This will result in all of the lines printed (or spoken).

Here are `messages` containing all of the data returned from the `getAccountStatus` API call:

```
"messages": [
  [
    "Let's have a look at your account. You have:",
    "{{$currency status.balance 'USD'}}",
    "{{status.minutes}} minutes, {{status.SMS}} text messages",
    "and {{status.data}} left"
  ]
]
```

The whole `check-my-balance` conversation should look like this:

```json
"check-my-balance": {
  "type": "goal",
  "steps": [
    {
      "type": "webhook",
      "entity": "status",
      "data-source": {
        "endpoint": "https://demoapis.com/cmobile/getAccountStatus/012345678"
      },
      "messages": [
        [
          "Let's have a look at your account. You have:",
          "{{$currency status.balance 'USD'}}",
          "{{status.minutes}} minutes, {{status.SMS}} text messages",
          "and {{status.data}} left"
        ]
      ]
    }
  ]
}
```

#### Test

To test this new conversation, press the **[Test]** button and type: *What is my balance?*

The chatbot should respond with your message.

**Here is how to do it:**

<img src="./img/check-my-balance-3.gif?raw=true">

#### (Optional) Tidy up the initial messages

Before you move on, it is a good idea to tidy up the initial messages.

Your chatbot project has 3 support conversations:

- `welcome` — this conversation is triggered when you start talking with your chatbot. This conversation triggers the `help` conversation.
- `help` — this conversation is triggered when the chatbot believes that you might need help
- `restart` — this conversation is triggered when you type/say *"restart"*. This conversation triggers the `welcome` conversation

**Update the welcome message**

Find the `welcome` conversation and change: `"This is a welcome conversation for your chatbot"` text to `"Welcome to the c-mobile self service."`

**Update the help message and the list of proposed actions**

Find the `help` conversation and remove the line `"If you get stuck, you can always restart our conversation by typing 'restart'",`. This way the intro conversation will be a lot shorter.

Additionally, you should change the `display` options from `"Conversation 1", "Conversation 2"` to `"Check my balance", "Top up", "Buy data"`. You will implement `Top up` and `Buy data` later, but it is good to already have these options.

Your `welcome` and `help` conversations should look like this:

```json
"welcome": {
  "type": "support",
  "steps": [
    {
      "type": "message",
      "messages": [
        "Welcome to the c-mobile self-service."
      ]
    },
    {
      "type": "conversation",
      "conversation": "help",
      "conditions": [
        "{{$not ($has conversation) }}"
      ]
    }
  ]
},
"help": {
  "type": "support",
  "steps": [
    {
      "type": "message",
      "messages": [
        [
          "Here is what I can do for you:"
        ]
      ],
      "display": {
        "type": "quick-reply",
        "data": [
          "Check my balance",
          "Top up",
          "Buy data"
        ]
      }
    }
  ]
},
```

#### Test

To test this new configuration, press the **[Test]** button and type: **Hi**

The welcome message should be different 

#### Remove Conversations One/Two

Additionally, you can remove `ConversationOne` and `ConversationTwo`, as these are not needed anymore.

This should be done in two steps:

- Remove the `json` from the **Cognitive Flow** — this can be done by clicking the little `-` sign next to the conversation you want to remove, then highlight that line and the one below and delete.
- Remove the phrases from the **Conversation Training** — open the **Training** tab, go to `Conversation` then press the **trash can** icon next to `ConversationOne` and `ConversationTwo`.

#### The code

The whole cognitive flow should look like this:

```json
{
  "conversations": {
    "welcome": {
      "type": "support",
      "steps": [
        {
          "type": "message",
          "messages": [
            "Welcome to the c-mobile self service."
          ]
        },
        {
          "type": "conversation",
          "conversation": "help",
          "conditions": [
            "{{$not ($has conversation) }}"
          ]
        }
      ]
    },
    "help": {
      "type": "support",
      "steps": [
        {
          "type": "message",
          "messages": [
            [
              "Here is what I can do for you:"
            ]
          ],
          "display": {
            "type": "quick-reply",
            "data": [
              "Check My Balance",
              "Top up",
              "Buy more data"
            ]
          }
        }
      ]
    },
    "restart": {
      "type": "support",
      "steps": [
        {
          "type": "message",
          "messages": [
            "Your conversation is restarted."
          ]
        },
        {
          "type": "conversation",
          "conversation": "welcome"
        }
      ]
    },
    "check-my-balance": {
      "type": "goal",
      "steps": [
        {
          "type": "webhook",
          "entity": "status",
          "data-source": {
            "endpoint": "https://demoapis.com/cmobile/getAccountStatus/012345678"
          },
          "messages": [
            [
              "Let's have a look at your account. You have:",
              "{{$currency status.balance 'USD'}}",
              "{{status.minutes}} minutes, {{status.SMS}} text messages",
              "and {{status.data}} left"
            ]
          ]
        }
      ]
    }
  },
  "settings": {
    "invalid-replies": [
      "I am not sure I understood what you said."
    ],
    "general-failure": [
      "We are experiencing technical difficulties at this moment."
    ],
    "previous-conversation-messages": [
      "I am going back to the {{ conversationDisplayName }} now."
    ]
  },
  "commands": {
    "NEXT-PAGE": [
      "Next 5"
    ],
    "RESTART": [
      "restart"
    ]
  }
}
```

### Step 3 — Connect with the Alexa Console

Now that you have a working chatbot, it is time to start talking to it with actual voice commands. The best thing is that you don't even need an Alexa device to do that.

All you need is a configured [Alexa Developer Console](https://developer.amazon.com/alexa/console/ask) and the [Kinvey Chat Proxy skill](https://alexa.amazon.co.uk/spa/index.html?#skills/dp/B07W4QRF28).

#### Kinvey Chat Proxy skill

**Kinvey Chat Proxy** is a skill, which is designed to easily connect with your Kinvey Chat chatbots without having to publish a new skill. It works like a sandbox skill, which relays all messages from the user to the connected Kinvey Chat chatbot.

Go to your Alexa app, find and install the **Kinvey Chat Proxy** skill.

Alternatively, you can install it using the Alexa web interface. Log in to your Amazon account from the [US Portal](https://alexa.amazon.com/), [UK Portal]( https://alexa.amazon.co.uk/) (or your country-specific portal). Then go to **Skills**, find and install the **Kinvey Chat Proxy** skill.

#### Amazon Alexa Console

Next, you need to go to the [Alexa Developer Console](https://developer.amazon.com/alexa/console/ask) and open the Alexa Test simulator.

To open the simulator you need to use an Alexa skill. If you already have one, you can just go to the **Test** tab and you are good to go.

If you don't have an Alexa skill yet, here is what you need to do:

In the [Alexa Developer Console](https://developer.amazon.com/alexa/console/ask), press the **Create Skill** button.

> Note, you could use this skill later when you are ready to publish your chatbot as an Alexa skill.

Call the skill **C Mobile**, and select **Custom** model and **Provision your own** hosting method, and press the **Create skill** button.

Then, select the **Start from scratch** template, and press the **Choose** button.

Open the **JSON Editor**, and find **"Amazon.FallbackIntent"** and set the **samples** to *"Sample"* (at this stage it could be any word, we just need something in there), it should look like this:

```
{
    "name": "AMAZON.FallbackIntent",
    "samples": [
        "Sample"
    ]
},
```

Then, press the **Build Model** button and wait for a little bit.

Finally, open the **Test** tab, and switch the skill testing from **Off** to **Development**.

**Here is how to do it:**

> HERE SHOULD BE THE VIDEO FROM [./video/Alexa-new-skill.mp4](https://github.com/sebawita/KinveyChat/blob/master/Articles/chatbot-with-Alexa/video/Alexa-new-skill.mp4?raw=true)

<video>
  <source src="./video/Alexa-new-skill.mp4?raw=true">
</video>

#### Use proxy

Now, to complete the loop, you just need to get the **proxy id** and pass it to the **Kinvey Chat Proxy** skill.

In **Kinvey Chat**, expand the **Test** options, and choose **Test in Alexa Developer Console**. This will provide you with instructions on how to make it work.

You should something like this:

<img src="./img/Kinvey-Chat-Proxy-Screen.png?raw=true" width="400">

Copy the last message that contains the **proxy id**. (hint. you can click on the little icon next to it).

Go back to the **Alexa Developer Console**:

Type: **kinvey chat proxy** and press Enter.

Then, paste the **proxy command** and press enter.

And voila, your chatbot is ready to listen and to speak back to you.

**Here is how to do it:**

> HERE SHOULD BE THE VIDEO FROM [./video/Connecting-Proxy-to-Alexa.mp4](https://github.com/sebawita/KinveyChat/blob/master/Articles/chatbot-with-Alexa/video/Connecting-Proxy-to-Alexa.mp4?raw=true)

<video>
  <source src="./video/Connecting-Proxy-to-Alexa.mp4?raw=true">
</video>

#### Talk to me

There are two ways that you can interact your chatbots in **Alexa Developer Console**.

- You can type everything, just like you would in a text-based interface.
- You can talk to it, just like you would expect to use the real device, except you don't have to say the **"Alexa"** keyword. To do that click and hold on the microphone icon, and release when you finish the command.

> Note, if you reload the Alexa Developer Console page, or close it and open it again, then the console will disconnect from the Kinvey Chat Proxy skill. All you need to do is say: *"Kinvey Chat Proxy"*, and you will be back where you left it (no need to provide the Proxy ID again).

#### Test

To hear the welcome message, say: **"hi"**

Then say: **"Check my balance"**

And just like that, you have just talked to your first Kinvey Chat chatbot through an Alexa skill.

Well done, you deserve a pat on the back:

<img src="https://media.giphy.com/media/cLYubXxF0P55MO1IUY/giphy.gif" alt="pat on the back">

#### What is happening here?

The way this works is pretty straight forward.

Whenever you speak, Alexa will use voice recognition to convert your speech to text.

It then sends it to Kinvey Chat — via the Kinvey Chat Proxy skill.

Then Kinvey Chat runs its algorithms to decide on a response, which is sent back to Alexa.

Finally, Alexa reads out loud the message and waits for the user response.

## Speaking Challenges

One of the trickiest challenges you will face is with voice recognition.

There might be cases when your users will try to say things like: *"Pay now"*, while Alexa might understand it as *"Play now"*. This can be quite problematic when your chatbot expects a specific item.

To work around that, you should train your chatbot to understand different ways of saying the same thing — this includes different phrases, but also different words that sound similar — so that it would respond with the same action regardless of how someone might pronounce their commands.

#### Debugging conversations

In the case of this chatbot, a user could say: *"Check my balance"*, which Alexa might understand as *"Shake my balance"*. This is still close enough, so Kinvey Chat might accept it.

<img src="./img/Shake-my-balance.png?raw=true" width="300">

However, if you are struggling with some voice commands, you can investigate what messages are sent to Kinvey Chat, and how Kinvey Chat interpreted them. This will come especially useful when you start using your chatbots outside of the Alexa Developer Console.

In Kinvey Chat portal, navigate to the **History** tab. Find the conversation that was causing trouble and click on it.

> hint: you can add a filter to show only conversations for specific dates, or Alexa only.

From here, you can see all the messages sent and received. If you click on the user input, you will see how this was interpreted by Kinvey Chat. In the case of *"Shake my balance"*, Kinvey Chat was 75% sure that you wanted to check your balance, so all is good.

<img src="./img/Shake-my-balance-understanding.png?raw=true">

To make it 100%, you would need to add *"Shake my balance"* to the Conversation training for **check-my-balance**.

## Finishing off the app

Now that you have your chatbot connected with Alexa, you can proceed with implementation of the remaining conversations, "Top up" and "Buy data".

This time, you will be able to edit the conversations and test them straight away with voice commands. No need to configure the proxy id, or anything else. All changes will be available in Alexa as soon as you save them.

## Conversation: Top-up

In this conversation the chatbot should ask the user:

- for the **amount** they want to top up with
- which payment **card** they want to use
- to **confirm** their selection

Then finally, it should either execute the top up (if confirmed) or cancel the process (if confirmation rejected).

### API

For this conversation, you should use the following API calls:

- `availableCards` — used to get a list of payment cards associated with the account
  - requires `account id`
  - example: [https://demoapis.com/cmobile/availableCards/012345678](https://demoapis.com/cmobile/availableCards/012345678)
- `topUp` — used to perform the top up 
  - requires `account id` and `card number`
  - example: [https://demoapis.com/cmobile/topUp/012345678?val=10&card=9876](https://demoapis.com/cmobile/topUp/012345678?val=10&card=9876)

> **Reminder:** to avoid conflicting with other people using this API, please use a different account id to the one used in the example (012345678).

### Adding a new conversation

First, you need to create a new conversation called `top-up`, which is done just like you did it with the `check-my-balance`.

#### Add check-my-balance

Go to the **Cognitive Flow** tab, and find the end of the  `"check-my-balance"` code.

Add a comma, and add a new line.

Start typing `con`, and select the `conversation-goal` snippet and press enter. 

Change `"conversation-name"` to `"top-up"`.

**Add a sample step**

Then, just to see that the conversation works, you should add a sample message step.

In the `steps` array, start typing `step` and select the `step-message` snippet.

Set the message to `"top-up works"`.

The conversation should look like this:

```json
"top-up": {
  "type": "goal",
  "steps": [
    {
      "type": "message",
      "messages": [
        "top-up works"
      ]
    }
  ]
}
```

**Train the chatbot to know when to use your new conversation**

Finally, train the chatbot to understand when to trigger this conversation.

Navigate to the **Training** tab, click on **Conversation built-in**, and then press the **[Add value]** button.

Set the `value` to `"top-up"`

Add the following expressions:

- *Top up*
- *Add more funds*

Press the **[Save]** button.

**Test**

To test this, go to **Alexa Developer Console**, and say **Top up** or **Add more funds**

Alexa should respond with your message.

> **Note #1**
>
> If Alexa doesn't understand your expressions, like when you say *"top up"* it might understand it as *"top pop"* then you can add this expression to the training.

> **Note #2**
>
> You should be able to continue to use your chatbot through the proxy app. But if not just say "Kinvey Chat Proxy".

### Implement the conversation

Now, that you have the `top-up` conversation wired up, it is time to implement it with the proper logic.

Note, you should test the conversation each time you add a new step.

**Clean up**

Start by removing the sample message step, so that the `steps` array is empty, like this

```json
"top-up": {
  "type": "goal",
  "steps": [
    
  ]
}
```

#### Ask for top-up-value

The first step of this conversation is to ask the user how much they would like to top up.

In the `steps` array, start typing `step` and select the `step-question` snippet.

You should be presented with the following code:

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

Here is what his all means:

- `entity` — the name of the variable where the user answer is going to be stored. Note this is not going to be the whole expression that the user will say, but the actual value that you are after
  - set it to: **"top-up-value"**
- `entity-type` — the type of the entity that you are after, this is how the chatbot will know what to find the value that you are after
  - set it to: **"Number"**
- `messages` — the expression your chatbot will use to ask for user input
  - set it to: **"How much would you like to top up?"**

Your step should look like this:

```json
{
  "type": "question",
  "entity": "top-up-value",
  "entity-type": "Number",
  "messages": [
    "How much would you like to top up?"
  ]
}
```

Press the **[Save]** button and test.

**Test**

Now, when you say:

- **"Top up"**
- and then **"Make it twenty five"**

Alexa will not say anything after you provide the answer.

Don't worry, that is OK, as you don't need to confirm the answers provided to the chatbot questions.

**Adding a reaction**

Note, that the chatbot can recognise entity values at any time during the conversation.

For example, when you say **"Top up twenty five"**, the chatbot will understand that:

- you want to `top-up`
- the `top-up-value` is`25`

In this case, the chatbot should respond with a confirmation that it understood the extra entity value.

This is done with **acknowledgements**.

After the end of messages array, add a comma, then add the following code:

```json
"reactions": {
  "acknowledgements": [
    "I understand that you want to top up {{$currency top-up-value 'USD'}}"
  ]
}
```

Press the **[Save]** button and test.

**Test**

Now, when you say: **"Top up twenty five"**

Alexa should respond with: **"I understand that you want to top up $25.00"**

**The full step should look like this:**

```json
{
  "type": "question",
  "entity": "top-up-value",
  "entity-type": "Number",
  "messages": [
    "How much would you like to top up?"
  ],
  "reactions": {
    "acknowledgements": [
      "I understand that you want to top up {{$currency top-up-value 'USD'}}"
    ]
  }
},
```

#### Ask for the card number

The next step should be to ask the user to provide the payment card they want to use.

In the case of the c-mobile API, you only need to provide the last 4 digits of the card, which is already stored in the system. You can see all available cards by calling: [https://demoapis.com/cmobile/availableCards/012345678](https://demoapis.com/cmobile/availableCards/012345678).

Go to the **Cognitive Flow** tab, and find the end of the `top-up-value` question step.

Add a comma, and start typing `stq` and select the `step-question` snippet.

Set the properties to:

- `entity`: **card**
- `entity-type`: **Number**
- `messages`: **Which card would you like to use?**

**Make it explicit**

Additionally, to avoid the number in the card ending clashing with the `top-up-value`, you need to make this question explicit. This means that when the user is asked for the card number, no other entities of type Number can be updated.

Add a new line after `entity-type` and add  `"is-explicit": true,`

**Provide possible values**

To help the user choose the right value, you can call `availableCards`, and provide the users with the possible values.

After the `messages` array, add a comma and then add the following code:

> **Reminder:** to avoid conflicting with other people using this API, please change account id

```json
"display": {
  "type": "quick-reply",
  "data-source": {
    "endpoint": "https://demoapis.com/cmobile/availableCards/012345678"
  },
  "template": "ending **{{card}}"
}
```

Press the **[Save]** button and test.

**Test**

Now try to say:

- **"Top up 20"**
- **"use card ending 1234"**

**Alexa truncating leading zeroes**

Note, when you tell Alexa a number that starts with leading zeroes, Alexa will skip these digits.

For example, input: **Use card 0020**, will be captured as **Use card 20**.

Make sure to be aware of this behaviour, so that when you expect a 4 digits card or PIN, and if you receive less, you should be safe to assume that the missing digits are zeroes.

**(Optional) Validation**

If you would like to make this conversation more robust and make sure that the user always provides a card number that is actually stored on the system, you could use a validation.

There are various types of validations, which allow you to validate that a provided value is a telephone number, or that it matches a specific regular expression, and many more. You can read all about validation rules in the [docs](https://docs.nativechat.com/docs/1.0/cognitive-flow/validation.html).

The validation that you need to use in this scenario is a `custom` `webhook` validation. This validation allows you to execute a query, then validate that the entity value matches the `_response` returned from the query.

Add the following code after the `display` section:

> **Reminder**: remember to update your account id

```json
"reactions": {
  "validations": [
    {
      "type": "custom",
      "parameters": {
        "data-source": {
          "endpoint": "https://demoapis.com/cmobile/availableCards/012345678",
          "selector": "$[:].val"
        },
        "condition": "{{$in card _response}}",
      },
      "error-message": [
        "Card number {{card}} not found on the system."
      ]
    }
  ]
}
```

Here is how to understand this code:

- `data-source`

  - `endpoint`: contains the URL with a query

  - `selector`: parses the response to a specific format, it uses [JSON Path Expressions](https://goessner.net/articles/JsonPath/index.html#e2) to do the job.

    In this case, the response from **/availableCards** query will look something like this:

    ```json
    [
      {
        "card": "6575",
        "val": 6575
      },
      {
        "card": "3688",
        "val": 3688
      }
    ]
    ```

    While you just want to check if the `card` value matches one of the `val` values.

    To do that `$.[:]` will give you all objects from the response, while `.val` will return all values of `val`. As a result `$:[:].val` will return:

    ```json
    [
      6575,
      3688
    ]
    ```

- `condition`: the expression to test the validation.

  `$in card _response` checks if the value of `card` exists in the parsed `_response`

- `error-message`: the error message, you can use `{{card}}` to tell the user what card number doesn't seem to work.

Press the **[Save]** button and test.

**Test**

Now try to say:

- **"Top up 20"**
- **"use card ending** <u>[wrong number]</u>**"**  
  - the chatbot should respond with an error message, and ask for input again
- **"use card ending** <u>[correct number]</u>**"**
  - the chatbot should continue

**The full step should look like this:**

```json
{
  "type": "question",
  "entity": "card",
  "entity-type": "Number",
  "is-explicit": true,
  "messages": [
    "Which card would you like to use? "
  ],
  "display": {
    "type": "quick-reply",
    "data-source": {
      "endpoint": "https://demoapis.com/cmobile/availableCards/012345678"
    },
    "template": "ending **{{card}}"
  },
  "reactions": {
    "validations": [
      {
        "type": "custom",
        "parameters": {
          "condition": "{{$in card _response}}",
          "data-source": {
            "endpoint": "https://demoapis.com/cmobile/availableCards/012345678",
            "selector": "$[:].val"
          }
        },
        "error-message": [
          "Card number {{card}} not found on the system."
        ]
      }
    ]
  }
}
```

#### Ask to confirm the values

Before, the chatbot performs the top-up operation, it should ask the user to confirm the **top-up-value** and the **card**.

This can be done with a **Confirmation** step.

Go to the end of the card question step, add a comma and start typing `stco` and select the `step-confirmation` snippet. The snippet should look like this.

```json
{
  "type": "confirmation",
  "entity": "result-entity-name",
  "messages": [
    "Confirm action?"
  ]
}
```

Set the properties to:

- `entity`: **top-up-confirmation**,
- `messages`: **Just to confirm. Do you want to top up {{$currency top-up-value 'USD'}} with the card ending with {{card}}?**

Press the **[Save]** button and test.

**Test**

Now try to say:

- **"Top up 20"**
- **"use card ending** <u>[correct number]</u>**"**
  - the chatbot should ask the confirmation question including the `top-up-value` and the `card` number.
- **"Yes"**

**The full step should look like this:**

```json
{
  "type": "confirmation",
  "entity": "top-up-confirmation",
  "messages": [
    "Just to confirm. Do you want to top up {{$currency top-up-value 'USD'}} with the card ending with {{card}}?"
  ]
}
```

#### Display cancelation message

If the user rejects the confirmation, then the chatbot should respond with a message confirming the choice.

This can be done by simply adding a new message step.

However, you want this step to only trigger when `top-up-confirmation` is `false`. This can be done with the help of the `conditions` property.

You can learn more about **Conditions** in the [docs](https://docs.nativechat.com/docs/1.0/cognitive-flow/conditions.html).

Add a new message step at the end of the conversation, start typing `stme` and select the `step-message` snippet.

Set the `messages` to **Top-up has been cancelled**.

Add the `conditions` property with the following (self-explanatory) condition:

```json
"conditions": [
  "{{$not top-up-confirmation}}"
]
```

Press the **[Save]** button and test.

**Test**

Start the chat, and say **"No"** when you get asked to confirm.

The chatbot should respond with the cancellation message.

**The full step should look like this:**

```json
{
  "type": "message",
  "messages": [
    "Top-up has been cancelled"
  ],
  "conditions": [
    "{{$not top-up-confirmation}}"
  ]
},
```

#### Execute the top-up

The final step is to execute the top-up call, by calling the cmobile API `topUp` function.

This operation can be done with a **Webhook** step.

However, once again you don't want this step to be executed every time. You want this step to be executed when `top-up-confirmation` is `true`.

Add a new **Webhook** step at the end of the conversation, start typing `stwe` and select the `step-webhook` snippet. The snippet should look like this:

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

**Data Source**

You don't need the `method`, `headers` and `payload` properties, so just delete them.

Then update the `endpoint` to call the `topUp` API function and pass `top-up-value` as `val`, like this:

```json
"data-source": {
  "endpoint": "https://demoapis.com/cmobile/topUp/012345678?val={{top-up-value}}"
},
```

**Condition**

To make sure that this step is only triggered when the user confirms they want to proceed.

Add the following `conditions` property to the step (not inside the `data-source`):

```json
"conditions": [
  "{{top-up-confirmation}}"
]
```

**Confirmation message**

Then finally, you need to display a message confirming that the transaction is complete.

To do that you need to add two properties: `entity` and `messages`.

Add an `entity` property called **top-up-response** (it is best to add it below `type`)

```json
"entity": "top-up-response",
```

Add a `messages` property with the following message (it is best to add it below `conditions`:

```json
"messages": [
  "Your new balance is {{$currency top-up-response.balance 'USD'}}"
]
```

**The full step should look like this:**

```json
{
  "type": "webhook",
  "entity": "top-up-response",
  "data-source": {
    "endpoint": "https://demoapis.com/cmobile/topUp/012345678?val={{top-up-value}}"
  },
  "conditions": [
    "{{top-up-confirmation}}"
  ],
  "messages": [
    "Your new balance is {{$currency top-up-response.balance 'USD'}}"
  ]
}
```

Press the **[Save]** button and test.

**Test**

Go through the chat conversation, and say **"Yes"** when you get asked to confirm.

The chatbot should execute the **Top-Up** API call and respond with updated balance value. It should be increased by the amount specified in the first step.

### The code

The whole **top-up** conversation should look like this:

```json
"top-up": {
  "type": "goal",
  "steps": [
    {
      "type": "question",
      "entity": "top-up-value",
      "entity-type": "Number",
      "messages": [
        "How much would you like to top up?"
      ],
      "reactions": {
        "acknowledgements": [
          "I understand that you want to top up {{$currency top-up-value 'USD'}}"
        ]
      }
    },
    {
      "type": "question",
      "entity": "card",
      "entity-type": "Number",
      "is-explicit": true,
      "messages": [
        "Which card would you like to use? "
      ],
      "display": {
        "type": "quick-reply",
        "data-source": {
          "endpoint": "https://demoapis.com/cmobile/availableCards/012345678"
        },
        "template": "ending **{{card}}"
      },
      "reactions": {
        "validations": [
          {
            "type": "custom",
            "parameters": {
              "data-source": {
                "endpoint": "https://demoapis.com/cmobile/availableCards/012345678",
                "selector": "$[:].val"
              },
              "condition": "{{$in card _response}}"
            },
            "error-message": [
              "Card number {{card}} not found on the system."
            ]
          }
        ]
      }
    },
    {
      "type": "confirmation",
      "entity": "top-up-confirmation",
      "messages": [
        "Just to confirm. Do want to top up {{$currency top-up-value 'USD'}} with the card ending with {{card}}?"
      ]
    },
    {
      "type": "message",
      "messages": [
        "Top up has been cancelled"
      ],
      "conditions": [
        "{{$not top-up-confirmation}}"
      ]
    },
    {
      "type": "webhook",
      "entity": "top-up-response",
      "data-source": {
        "endpoint": "https://demoapis.com/cmobile/topUp/012345678?val={{top-up-value}}"
      },
      "conditions": [
        "{{top-up-confirmation}}"
      ],
      "messages": [
        "Your new balance is {{$currency top-up-response.balance 'USD'}}"
      ]
    }
  ]
}
```

## Conversation: Buy data

In this conversation the chatbot should:

- list available **data-packs** and ask the user which **data-pack** they want to buy
- then, it should execute the data purchase, and provide the updated data

### API

For this conversation, you should use the following API calls:

- `dataPackages` — used to get a list of available data packs
  - example: [https://demoapis.com/cmobile/dataPackages](https://demoapis.com/cmobile/dataPackages)
- `buyData` — used to purchase the selected data pack 
  - requires `account id` and `data-pack`
  - example: [https://demoapis.com/cmobile/buyData/012345678?dataPack=small](https://demoapis.com/cmobile/buyData/012345678?dataPack=small)

> **Reminder:** to avoid conflicting with other people using this API, please use a different account id to the one used in the example (012345678).

### Adding a new conversation

First, you need to create a new conversation called `buy-data`, which is done just like you did it with the `top-up`.

#### Add buy-data

Go to the **Cognitive Flow** tab, and find the end of the  `"top-up"` code.

Add a comma, and add a new line.

Start typing `con`, and select the `conversation-goal` snippet and press enter. 

Change `"conversation-name"` to `"buy-data"`.

This time leave the `"steps"` array empty.

**Train the chatbot to know when to use your new conversation**

Finally, train the chatbot to understand when to trigger this conversation.

Navigate to the **Training** tab, click on **Conversation built-in**, and then press the **[Add value]** button.

Set the `value` to `"buy-data"`

Add the following expressions:

- *Buy data*
- *Add data*

Press the **[Save]** button.

### Adding DataPack entity

For the chatbot to allow the user to select a **Data Pack**, you need to train it, so that it understands what Data Packs are available, and how to extract it from a sentence.

For example, when a user says: *"I want a small data pack"*

The chatbot should understand that **small**, is the name of the required **Data Pack**.

**Add DataPack to the Entity training list**

Creating new entities is quite simple. 

Open the **Training** tab and press the **[Add new]** button and set:

- the `name` to **DataPack** - this will be the name of your new Entity Type
- the `Lookup Strategy` to **Keyword** - this means that there is a specific list of expected values
- the `Training data source` to **Dynamic** - this means that the list of values can be loaded dynamically
- the `Endpoint URL` to **https://demoapis.com/cmobile/dataPackages** - this is the URL where the data is coming from
- the `Value template` to **{{name}}** - this indicates which field should be used for training of the Entity Type

Press the **[Test]** button, which should display a list of possible values.

If everything looks fine, then press the **[Create]** button.

Kinvey Chat will pull the data, and train the data model to understand the values for **DataPack**.

### Implement the conversation

Now, that you have the `buy-data` conversation wired up, and a data model for **DataPack**, it is time to implement the logic.

#### Ask for data-pack

The first step should be to provide the user with the list of available data packs and ask them to select one.

In the `steps` array, start typing `stq` and select the `step-question` snippet. Set:

- the `entity` to **data-pack**
- the `entity-type` to **DataPack**
- the `messages` to **"Which data pack would you like?"**

At this point, your code should look like this:

```json
{
  "type": "question",
  "entity": "data-pack",
  "entity-type": "DataPack",
  "messages": [
    "Which data pack would you like?"
  ]
},
```

Which should be enough to let the user choose a Data Pack, however, we should let user what Data Packs are available.

**Provide possible values**

To help the user choose a DataPack, you can call `dataPackages`, and provide the users with the possible values.

After the `messages` array, add a comma and then add the following code:

```json
"display": {
  "type": "quick-reply",
  "data-source": {
    "endpoint": "https://demoapis.com/cmobile/dataPackages"
  },
  "template": "{{name}} {{data}} for {{$currency price 'USD'}}"
}
```

Press the **[Save]** button and test.

**The full step should look like this:**

```json
{
  "type": "question",
  "entity": "data-pack",
  "entity-type": "DataPack",
  "messages": [
    "Which data pack would you like?"
  ],
  "display": {
    "type": "quick-reply",
    "data-source": {
      "endpoint": "https://demoapis.com/cmobile/dataPackages"
    },
    "template": "{{name}} {{data}} for {{$currency price 'USD'}}"
  }
},
```

#### Execute the buy data

The final step is to execute the buy data call, by calling the cmobile API `buyData` function.

This operation can be done with a **Webhook** step. Add a new **Webhook** step at the end of the conversation, start typing `stwe` and select the `step-webhook` snippet. 

**Data Source**

You don't need the `method`, `headers` and `payload` properties, so just delete them.

Then update the `endpoint` to call the `buyData` API function and pass `data-pack` as `dataPack`, like this:

```json
"data-source": {
  "endpoint": "https://demoapis.com/cmobile/buyData/012345678?dataPack={{data-pack}}"
},
```

**Confirmation message**

Then finally, you need to display a message confirming that the transaction is complete.

To do that you need to add two properties: `entity` and `messages`.

Add an `entity` property called **buy-data-response** (it is best to add it below `type`)

```json
"entity": "buy-data-response",
```

Add a `messages` property with the following message (it is best to add it below `conditions`:

```json
"messages": [
  "Your purchase is complete. You now have {{buy-data-response.data}}. Your balance is {{$currency buy-data-response.balance 'USD'}}"
]
```

**The full step should look like this:**

```json
{
  "type": "webhook",
  "entity": "buy-data-response",
  "data-source": {
    "endpoint": "https://demoapis.com/cmobile/buyData/012345678?dataPack={{data-pack}}"
  },
  "messages": [
    "You now have {{buy-data-response.data}}. Your balance is {{$currency buy-data-response.balance 'USD'}}"
  ]
}
```

Press the **[Save]** button and test.

**Test**

Say: **"Add Data"**

The chatbot should list the available packages.

Say: **"Lite"**

The chatbot should execute the **Buy Data** API call and respond with updated data and balance values.

### The code

The whole **buy-data** conversation should look like this:

```json
"buy-data": {
  "type": "goal",
  "steps": [
    {
      "type": "question",
      "entity": "data-pack",
      "entity-type": "DataPack",
      "messages": [
        "Which data pack would you like?"
      ],
      "display": {
        "type": "quick-reply",
        "data-source": {
          "endpoint": "https://demoapis.com/cmobile/dataPackages"
        },
        "template": "{{name}} {{data}} for {{$currency price 'USD'}}"
      }
    },
    {
      "type": "webhook",
      "entity": "buy-data-response",
      "data-source": {
        "endpoint": "https://demoapis.com/cmobile/buyData/012345678?dataPack={{data-pack}}"
      },
      "messages": [
        "You now have {{buy-data-response.data}}. Your balance is {{$currency buy-data-response.balance 'USD'}}"
      ]
    }
  ]
}
```

## Congratulations

<img src="https://media.giphy.com/media/3oz8xAFtqoOUUrsh7W/giphy.gif">

And just like that, you have created a fully functioning Alexa Skill. It can communicate using Natural Language Processing, communicate with the cmobile API to provide the users with the ability to check their account status, top up and buy more data.

There is still more to learn. You can learn more from the [Kinvey Chat documentation](https://docs.nativechat.com/).