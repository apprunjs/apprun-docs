# Installation

## npm

AppRun is distributed on npm. To get it, run:

```sh
npm install apprun
```

## Create AppRun App

The _npm create apprun-app_ command line tool creates a new AppRun project. It can scaffold a new project with build tools and a development server.

To create a project, run:

```sh
npx create-apprun-app my-app
cd my-app
npm start
```

## No build step

You don't need npm at all to try AppRun. You can reference it from a CDN and write your
views with the `html` template function — no compiler required. See
[Use AppRun in the browser](esm.md).

## TypeScript and JSX

When you use JSX, configure your compiler to use AppRun's JSX factory. In `tsconfig.json`:

```json
{
  "compilerOptions": {
    "jsx": "react",
    "jsxFactory": "app.h",
    "jsxFragmentFactory": "app.Fragment"
  }
}
```

The `create-apprun-app` templates set this up for you. For strongly-typed components and
events, see [Strong Typing](strong-typing.md).



