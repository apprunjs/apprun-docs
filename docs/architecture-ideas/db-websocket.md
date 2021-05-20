---
title: Database-Driven Applications Using WebSockets
published: true
description: This post introduces a new application architecture that allows event handling between the frontend apps and the backend business logic modules without REST API
tags: #AppRun, #JavaScript, #TypeScript, #Framework
---

## Introduction

The database is a specific technology for storing, managing, and processing data. In the real-world, web sites, mobile apps, and business applications that serve dynamic content all have a backend database.

Started being popular in the web and mobile apps, moving to the business applications, nowadays most of the database-driven applications use a REST API based architecture. The REST API provides flexibility, scalability, and simplicity over other traditional web services architectures.

![REST Architecture](https://github.com/yysun/apprun-websockets-sqlite/raw/master/architecture-old.png)

However, the primary purpose of the REST API is to decouple the backend and frontend, which assumes backend and frontend know nothing about each other. Even in case we know and own both backend and frontend, such as in many business applications, we still have to develop the backend API endpoints first. And then, we develop the frontend API clients. Developing backend and frontend separately is tedious and error-prone.

Also, If we want to publish events from the frontend to be handled in the backend business logic modules, we cannot do it directly. Furthermore, the REST API is not a duplex protocol. Only the frontend can call the API. The backend cannot call the frontend. Therefore sometimes, the REST API has become a barrier between frontend and backend that costs us extra time and effort to overcome.

In this post, I will introduce a new application architecture that allows us to send events back and forth between the frontend apps to the backend business logic modules using the [WebSocket API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) and [AppRun](https://github.com/yysun/apprun) without REST API.

> The WebSocket API is a duplex communication channel. It works well with an event-driven framework, such as AppRun.


## The Architecture

The new architecture uses the WebSocket API and AppRun event system.

AppRun has two important functions: _app.run_ and _app.on_. _app.run_ fires events. _app.on_ handles events. E.g.:

Module A handles the _print_ event:

```javascript
import app from 'apprun';
export default () => app.on('print', e => console.log(e));
```
Module B fires the _print_ event:

```javascript
import app from 'apprun';
app.run('print', {});
```

Module B can invoke the function in Module A without knowing Module A. It works when Module A and Module B are both frontend modules. Can the business logic modules behind the webserver also subscribe to the frontend events?

Yes, that's the exact idea of the new architecture.

![AppRun Architecture](https://github.com/yysun/apprun-websockets-sqlite/raw/master/architecture-new.png)

Let's see how it works.

## An Example Application

We will create a database-driven todo application to demonstrate the new architecture. The project has the following files:

![Project Files](https://dev-to-uploads.s3.amazonaws.com/i/ydkewsv89cs1gkxuufde.PNG)

* The database:
  * _db/todo.db_ is a SQLite database
* The public folder has the frontend code:
  * _index.html_
  * _dist/app.js_
* The server folder has the backend code:
  * _db.js_: the business logic
  * _server.js_: the web server using the _express_ and _websocket libraries
* The src folder has the frontend code:
  * _todo.tsx_: the AppRun component for managing the todo list
  * _main.tsx_: the main program


### Send Events to Server Through WebSocket

First, we create a WebSocket in the frontend app (_main.tsx_). Then, We define a special AppRun global event called _//ws:_, which sends the events to the server.

```javascript
const ws = new WebSocket(`wss://${location.host}`);
app.on('//ws:', (event, state) => {
  const msg = { event, state };
  ws.send(JSON.stringify(msg));
});
```

### Receiving Events from Frontend

We create the WebSockets on the webserver side (_index.js_). We listen to the WebSockets messages and convert them to AppRun events. AppRun runs on the webserver. Just like Module A and Module B example above, the AppRun events will be handled in the business logic module (_db.js_).

```javascript
const apprun = require('apprun').app;
require('./db');

const path = require('path');
const express = require('express');
const { createServer } = require('http');
const webSocket = require('ws');
const app = express();

app.use(express.static(path.join(__dirname, '../public')));

const server = createServer(app);
const wss = new webSocket.Server({ server });

wss.on('connection', function(ws) {
  ws.on('message', function (data) {
    try {
      const json = JSON.parse(data);
      console.log('==>', json);
      apprun.run(json.event, json, ws);
    } catch (e) {
      ws.send(e.toString());
      console.error(e);
    }
  });
});

```

Notice the webserver also adds the WebSocket reference, _ws_ as the event parameter for the business logic module.

### Handle Events in Business Logic Module

We handle AppRun events in the business logic module (_db.js_) to complete the CRUD operations against the database.

```javascript
const app = require('apprun').app;
const sqlite3 = require('sqlite3').verbose();
const dbFile = "db/todo.db";

app.on('@get-all-todo', (json, ws) => {
  const sql = 'select * from todo';
  db.all(sql, function (err, rows) {
    json.state = rows || [];
    ws.send(JSON.stringify(json));
  });
});

app.on('@get-todo', (json, ws) => {
});

app.on('@create-todo', (json, ws) => {
});

app.on('@update-todo', (json, ws) => {
});

app.on('@delete-todo', (json, ws) => {
});

app.on('@delete-all-todo', (json, ws) => {
});
```

Once completed the database operations, we use the WebSocket reference, _ws_, to send events back.


### Receiving Events from Backend

Receiving events from the backend in the frontend app (_main.tsx_) is straightforward.

```javascript
const ws = new WebSocket(`wss://${location.host}`);
ws.onmessage = function (msg) {
  const {event, state} = JSON.parse(msg.data);
  app.run(event, state);
}
```

You can see now we have 9 lines of client-side code in _main.tsx_ and 11 lines of server-side code in _index.js_ for transferring AppRun events between frontend and backend through WebSockets.

We also have a business logic module that operates the database using AppRun events.

They are ready to serve the frontend application.

### The Frontend Application

The frontend Todo application is a typical AppRun application that has the Elm inspired architecture (_todo.tsx_). Listed below is the simplified code except.


```javascript
import app, { Component } from 'apprun';

const state = {
  filter: 0,
  todos: []
}

const add = () => {
  app.run('//ws:', '@create-todo', {
    title: document.getElementById('new_todo').value,
    done: 0
  })
};

const toggle = (_, todo) => { app.run('//ws:', '@update-todo', { ... }) };

const remove = (_, todo) => { app.run('//ws:', '@delete-todo', todo) };

const clear = () => { app.run('//ws:', '@delete-all-todo') };

const search = (state, filter) => ({ ...state, filter });

const view = (state) => {...}

const update = {
  '@get-all-todo': (state, todos) => ({ ...state, todos }),

  '@create-todo': (state, todo) => ({ ... }),

  '@update-todo': (state, todo) => { ... },

  '@delete-todo': (state, todo) => { ... },

  '@delete-all-todo': state => ({ ...state, todos: [] })
}

export default new Component(state, view, update);

```

You can see we have _state_, _view_, and _update_ to form an AppRun component.

The local functions handle local events, such as _add_, _toggle_, _remove_, _clear_, and _search_. These functions fire the global event _//ws:_ to the WebSocket.

The _update_ object contains the event handlers for the events fired from the backend.

That's all the implementation plan. For details, please take a look at the live demo and the source code if you like.


### Run the Demo

Live Demo:


https://glitch.com/~apprun-websockets-sqlite

Source Code:

https://github.com/yysun/apprun-websockets-sqlite


## Conclusion

The todo application has demonstrated the architecture of using events through WebSockets. The web server has no REST API endpoints. The frontend has only event handlings and has no REST API calls.

The architecture is useful for database-driven applications, especially business applications.

Furthermore, AppRun events are not limited to frontend and WebSockets.  We can use AppRun events with the [Web Workers API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) explained in the [AppRun Book](https://www.amazon.com/Practical-Application-Development-AppRun-High-Performance/dp/1484240685/). We can also use AppRun in the [Electron Apps](https://github.com/apprunjs/apprun-electron-forge), Firebase, Cloud Pub-Sub, and more ...

Feel the power of event pub-sub pattern and learn more about building applications with AppRun.

