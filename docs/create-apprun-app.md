# Create AppRun App

The _npm create apprun-app_ command line tool creates a new AppRun project. It can scaffold a new project with build tools and a development server.

To create a project, run:

```sh
npx create-apprun-app my-app
cd my-app
npm start
```

You can select one of the following project templates:

```
? Select a template › - Use arrow-keys. Return to submit.
    HTML/JS
    HTML/JS - Web Component
    Blank App
    Signle Page App
❯   AppRun Site (default)
```

You can also select a build tool:

```
? Select a compiler › - Use arrow-keys. Return to submit.
    esbuild
    webpack
❯   vite
```

Then, you can choose to add Jest and git repo:

```
? Add Jest? … No / Yes
? Add git repo? › No / Yes
```

Finally, it creates a new project for you:

```
Project created in:  ....../my-app
Please go to the project directory and run:

        npm start

And then, you can visit the project at: http://localhost:8080
```

You can read more about the project templates in the following sections.

* [Single Page App](spa.md)
* [AppRun Site](apprun-site.md)