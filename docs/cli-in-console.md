# AppRun CLI in Console

We have been using the command-line interface (CLI) in the terminal window and the command prompt. Have you thought of a CLI in the console of the browser's developer tool?

![CLI in the console](https://thepracticaldev.s3.amazonaws.com/i/khumq8np94i5uwo9bwn1.png)

## How does it work?

In the console of the browser's developer tool (F12), you can type the command.

```javascript
_apprun `<command> [options]`
```

Just like many other CLI, the *help* command lists all available commands. You can see three commands in the screenshot*components*, *events* and *log*.

## Why do we need a CLI in the console?

CLI in the console is convenient for watching runtime data. For example, during application development, we often need to debug and exam the internal data of the application. Using the *console.log* function is the easiest yet very powerful way to display the data because the console lets us drill down into the nested array and object structure.

![Drilldown](https://thepracticaldev.s3.amazonaws.com/i/fq37a5rjfoz4pqsi0f05.png)

With a CLI in the console, The app codebase stays clear of *console.log*. The CLI provides a non-destructive way of watching the runtime data. We can include the CLI script in the development environment and remove it from the production environment.

#How is it made?

It is relatively easy to create a CLI in the console than to create a dev-tool as the browser extension. It is based on the JavaScript [tagged templates](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#Tagged_templates).

We create the *_apprun* function in the *window* object.

```javascript
window['_apprun'] = (strings) => { }
```

The *_apprun* function is called when we type the AppRun commands in the console. The command and the command parameters are passed into the *_apprun* function as the function parameter *strings*, which we can parse and then invoke the command functions.

```javascript
window['_apprun'] = (strings) => {
  const [cmd, ...p] = strings[0].split(' ').filter(c => !!c);
  const command = window[`_apprun-${cmd}`];
  if (command) command[1](...p);
  else window['_apprun-help'][1]();
}
```
It has an extensive architecture. We create the AppRun commands in the *window* object. The AppRun command is a tuple that includes the description of the command and the implementation function of the command. E.g. the help command look like this:

```javascript
window['_apprun-help'] = ['', () => {
  Object.keys(window).forEach(cmd => {
    if (cmd.startsWith('_apprun-')) {
      cmd === '_apprun-help' ?
        console.log('AppRun Commands:') :
        console.log(`* ${cmd.substring(8)}: ${window[cmd][0]}`);
    }
  });
}];
```
The *help* command searches for the tuples stored in the *window* object and prints the description of other AppRun commands.

That's all the infrastructure code we need to create CLI commands in the console. Let's see an example.


# Live Demo

The AppRun CLI in the console is one of the developer tools includes in the AppRun library. You can visit the AppRun RealWorld Example App https://gothinkster.github.io/apprun-realworld-example-app to see the CLI in actions.

* The *components* command logs the DOM elements that have AppRun components
* The *events* command logs the event subscription of the app
* The *log* command logs the runtime events publication of the app
* The *create-event-tests* command creates unit tests for the app
* The *create-state-tests* command creates Jest snapshot tests for the app

# Conclusion

Developers like CLI. CLI in the console is useful for getting runtime events and messages hard for the traditional CLI in the terminal. The AppRun CLI in the console even extended the CLI beyond watching the data to generate tests. Thus, it increases the development productivity for debugging and testing.




