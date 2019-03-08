# Intro

In this tutorial you will learn how to build a chatbot using [Kinvey Chat](https://www.progress.com/kinvey/chat). You will follow a series of steps, which will guide you through various aspects of building a fully featured chatbot.

## Background

You will build a chatbot for a car rental company. We will make a number of simplifications, for example: we will assume that each city can only have one rental office. However, the general idea should be fairly accurate with what you would expect from a real chatbot.

As the backend, we will use an instance of the [Progress Kinvey serverless backend](https://www.progress.com/kinvey), which will come with some data and cloud functions that will aid you with the process. 

### Postman

The tutorial will introduce each data collection and cloud function as we go.

If you are curious and would like to preview the data, you need to make a **GET** request and pass in an **authentication** key, which means that you can't just paste the URL in your browser to get the data.

In order to get a glimpse of the data or test out the cloud functions, you could use an application like [Postman](https://www.getpostman.com/). Here is a [getting started guide for Postman](https://learning.getpostman.com/getting-started/).

## The Structure of the tutorial

This tutorial is structured in a slightly different way to how you would actually go about building a chatbot. Usually you would add all the required features each time you add a new conversation step.

However, in order to make things simpler and easier to understand, the tutorial is divided in a number of chapters where each introduces a new concept and expands on the existing chatbot project.

## Type Type Type

It is important that you **type each piece of code yourself** and **don't copy & paste solutions** unless you are really stuck.

Typing will help you get used to the process, syntax and the structure of the project. While copy and paste is good to see how something works, before you forget it all. So, if you want to really learn: type, type, type it all.

Even as you progress through the tutorial and feel like you have already completed a similar task and that you can just copy & paste and tweak what needs changing, even (or especially) then try to go through the process and build each piece of the chatbot.

## Login/Registration

First you need to sign in with your **Kinvey Account** to [Kinvey Chat Portal](https://bots.kinvey.com/login).

If you don't have a **Kinvey Account** yet, you can [create one here](https://console.kinvey.com/sign-up). 

## Creating a project

From the Chat portal, create a new project (by pressing the big plus button), select a **Blank bot** template, call it **Car Rental**, and press **Create**.

![Create a bot](./img/create-bot.png?raw=true)

### Understanding a conversation structure

Although the template you used is the **Blank bot**, you will find that there is already some code in the **Cognitive Flow** tab.

You will probably notice that the code is in a JSON format. That is correct, Kinvey Chat uses a declarative approach, where you describe what you want the chatbot to handle, and the chatbot engine manages the process for you.

The chatbot contains configurations: 

* `welcome` - what to do when a user starts a new session,
* `help` - what to do when a user types **help** or when it gets stuck
* `restart` - what to do after the user asks to **restart**
* `conversationOne` and `conversationTwo` - two super basic conversations
* `settings` - general project settings - currently used for default messages,
* `commands` - commands that allow the user to restart a conversation or to ask for the next batch of items.

Let's have a look at **conversationOne**:

```json
"conversationOne": {
  "type": "goal",
  "display-name": "conversation 1",
  "steps": [
    {
      "type": "message",
      "messages": [
        [
          "This is conversation 1"
        ]
      ]
    }
  ]
},
```

Whenever this conversation kicks in, the chatbot responds with a simple text "This is conversation 1" and then it completes.

## Testing the default conversation

This is enough to test the chatbot. 

1. Press the **Test** button, which will open a chatbot in a new window.
2. Type: **Conversation 1** and press enter
3. See the response
4. See the conversation log, which shows the understanding from the chatbot engine - we will dive deeper into it later
5. Type **restart** and try again...

![Test](./img/test-conversation-1.png?raw=true)

> Note, that you don't have to restart close this window and open it again each time you make changes to your chatbot. Usually it is enough to **save** the chatbot and type **restart** for the chatbot to use the latest update.
