# State Management

The State is the application state at any given time of your applications. The State is the data flow between Update and View. It acts as the data transfer object (DTO) in traditional multilayered application architecture, where the DTO is an object that carries data between logical and physical layers.

The benefits of using events and DTO like the state is that there are no dependencies between the _view_ and _update (event handlers)_. It makes the AppRun applications easier to develop, test, and maintain. You can get more information about [unit testing](11-unit-testing) later.


## State History

The state can be stored in state history by AppRun. Once the state history is enabled, we can travel through the history back and forth to get the previous and next state. See the [Counter example](https://apprun.js.org/#counters) and [Todo - undo-redo example](https://apprun.js.org/#todo).

