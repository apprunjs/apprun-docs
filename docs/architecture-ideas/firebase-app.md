---
title: Serverless App Using Firebase and AppRun
published: true
description: This post introduces a serverless application architecture using Firebase and AppRun.
tags: #apprun #serverless #firebase #javascript
---

## Introduction

I have been writing about the application architecture without REST, which includes the [underlying architecture using WebSockets](https://dev.to/yysun/create-a-phoenix-liveview-like-app-in-js-with-apprun-dc8) and the [database-driven architecture](https://dev.to/yysun/database-driven-applications-using-websockets-2b9o). In this post, I will continue the journey to make a serverless application architecture using Firebase and [AppRun](https://apprun.js.org).

You will see how easy it is to use AppRun's event system with the Firebase stack to develop applications that have the full business logic process capabilities, such as authentication, authorization, request logging, and real-time database, and without REST layer.

Finally, we can make the application a serverless deployment to Firebase.

## The Architecture

The example application uses the following technologies:

* Firebase Cloud Firestore as the backend database
* Firebase Cloud Functions for business logic process
* Firebase Hosting to host the frontend
* Firebase Authentication

> [Firebase](https://firebase.google.com) is Google's mobile platform that helps you quickly develop high-quality apps and grow your business.

I will focus on the architecture instead of step-by-step instructions. If you are not familiar with the Firebase suite of products, please search google for the tutorials.

The architecture can be summarized in the diagram below.

![Architecture](https://github.com/yysun/apprun-firebase/raw/master/architecture.png)
<figcaption>Figure 1. Architecture Diagram</figcaption>

Let's get into the details.

### Event Pub-Sub Using FireStore

The center of the architecture is the Firebase Cloud Firestore. Firestore is a real-time database that keeps your data in-sync across client apps. When one client saves the data, FireStore pushes the data to all other clients.

In the AppRun applications, we use _app.on_ to publish events. If we save the events to FireStore, the events can be handled by other applications. It is the step (1) shown in Figure 1 above.

Firestore also triggers Cloud Functions.

### Business Logic Process Using Cloud Functions

Cloud Functions is Google Cloud's serverless compute platform. It runs on the server, not in the client apps. Therefore it is the best technology for business logic processing, authentication, and authorization. Functions are serverless. Functions run on Google's server, so we don't need to provision, manage, or upgrade the server.

The Functions are event-driven (the magic word, I love). Firestore can trigger Functions upon data updates. When we save the events into FireStore, FireStore triggers the Function to handle the events automatically. It is the step (2) in Figure 1.

### Real-Time Data Sync Using FireStore.

During the Functions event handling, it writes the updated data back to FireStore (step (3) in Figure 1). FireStore pushes the update to the frontend applications (step (4) in Figure 1). The frontend application listens to FireStore changes and publishes AppRun events for the frontend logic process to run.

Now, the event handling cycle is completed. Let's see it in action with an example.

## Example

The example is a ToDo application.

![Todo App](https://dev-to-uploads.s3.amazonaws.com/i/siavm31erdel8ea0asgj.png)
<figcaption>Figure 2. ToDo Application</figcaption>

### Save Events to FireStore

As usual, in the AppRun applications, we convert the DOM events into AppRun events. E.g., When users click the _add_ button, we publish the //: event.

```js
// in JSX
<button $onclick={[add]}>Add</button>

const add = () => {
  app.run('//:', '@create-todo', {
    title: (document.getElementById('new_todo').value,
    done: 0
  })
}
```

The //: event handler saves the event into FireStore.

```js
const db = firebase.firestore();
app.on('//:', (event, data = {}) => {
  db.collection(`events`).add({ uid, event, data })
});
```

There is a top-level collection, called _events_ in FireStore. We save the user id (obtained using Firebase anonymous authentication), event name (@create-todo), and event parameters (the new to-do item).

FireStore triggers our Function, which is monitoring the _events_ collection.

### Handle Events in Functions

```js
exports.updateTodo = functions.firestore.document('events/{Id}')
  .onWrite((change, context) => {
    const dat = change.after.data() as any;
    const { uid, event, data } = dat;
    const db = admin.firestore();
    const todos = db.collection('/users/' + uid + '/todos');
    switch (event) {
      case '@create-todo': return todos.add(data);
      case '@update-todo': ...
      case '@delete-todo': ...
      case '@delete-all-todo': ...
      default: return;
    }
});
```

The Function destructs the user id, event name, and event parameters and handles it accordingly, e.g., it adds a new Todo item data into FireStore upon the '@create-todo' event. And so on so forth.

FireStore then pushes the data change to the frontend.

### Real-Time Data in Frontend

In the frontend, we subscribe to the _onSnapshot_ of FireStore and publish the AppRun event, '@show-all'.

```js
const db = firebase.firestore();
db.collection(`users/${uid}/todos`).onSnapshot(snapshot => {
  app.run('@show-all',
    snapshot.docs.map(d => ({ id: d.id, ...d.data() })))
});
```

Now, we are back to our AppRun application world, in which you can see the three familiar parts: _state_, _view_, and _update_.

```js
import app, { Component } from 'apprun';

const state = {
  filter: 0,
  todos: []
}

const add = () => {
  app.run('//:', '@create-todo', {
    title: (document.getElementById('new_todo').value,
    done: 0
  })
};
const toggle = (_, todo) => { app.run('//:', '@update-todo', { ...todo, done: !todo.done }) };
const remove = (_, todo) => { app.run('//:', '@delete-todo', todo) };
const clear = () => { app.run('//:', '@delete-all-todo') };

const view = ({todos}) => {...}

const update = {
  '@show-all': (state, todos) => ({ ...state, todos })
}
```

The Firebase ToDo application shares the same architecture as in the [Database-Driven Application Post](https://dev.to/yysun/database-driven-applications-using-websockets-2b9o). They are only different in events. The Firebase ToDo application saves the events to FireStore. The Database-Driven Application sends and receives the events through the WebSockets.

> If you are new to AppRun, read the [AppRun Book](https://www.amazon.com/Practical-Application-Development-AppRun-High-Performance/dp/1484240685/) or visit [AppRun Docs](https://apprun.js.org).

## Live Demo and Source Code

You can play with the live demo at https://apprun-demo.firebaseapp.com.

Source Code: https://github.com/yysun/apprun-firebase

## Conclusion

The AppRun event pub-sub pattern looks so simple (just _app.run_ and _app.on_), yet so powerful. It is not only useful inside the frontend app. It shines more in crossing process boundaries, such as in the cases of [WebSockets](https://dev.to/yysun/create-a-phoenix-liveview-like-app-in-js-with-apprun-dc8), [Web Workers](https://github.com/yysun/apprun-apress-book/tree/master/Chapter_05), [Electron Apps](https://github.com/apprunjs/apprun-electron-forge), Firebase of course, and more ...



