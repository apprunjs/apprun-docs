# Single Page Apps

## Create Project

You can initialize a project using the `npm init apprun-app` command and select the `Single Page App` option.

```sh
npm init apprun-app [my-app]
```

## SPA Architecture

AppRun SPA usually includes an HTML file, the main program that renders the screen layout, and page components that render the pages.

![](imgs/Figure_7-2.png)

AppRun SPA uses the events to route user interaction to the components. Treating routing like other web events is the smart idea of AppRun. All web events are unified under the event pub-sub pattern. Routing does not require special treatment.

AppRun components are modularized using the ECMAScript module standard. We can import the modules statically and dynamically. We
can also use the native module from modern browsers.

```
.
├─ dist/
├─ src/
│  ├─ About.tsx
│  ├─ Contact.tsx
│  ├─ Home.tsx
│  ├─ Layour.tsx
│  └─ main.tsx
└─ index.html
```

