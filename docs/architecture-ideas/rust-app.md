---
title: Rust WebAssembly and AppRun
published: true
description: This post describes how to create a WebAssembly package using Rust and use it in the applications
tags: #apprun #rust #webassembly #javascript
---
## Introduction

[WebAssembly](https://webassembly.org/) has many different definitions on the Internet. I like the one from [MDN](https://developer.mozilla.org/en-US/docs/WebAssembly) the most, which says WebAssembly is a new binary assembly-like language that can run in the modern web browsers at near-native speed. There are [many tools](https://github.com/appcypher/awesome-wasm-langs) to compile code written in C/C++, Rust, Go, C#, etc. to be WebAssembly. It tells us that we can create high-performance code, but not using JavaScript/TypeScript

I decided to play with Rust. [Rust](https://www.rust-lang.org/what/wasm) is another hot buzzword. It is a relatively new programming language focused on performance and safety, especially safe concurrency. -- Wikipedia

This post describes how to create a WebAssembly package using Rust and use it in the [AppRun](https://apprun.js.org) applications from a JavaScript/TypeScript developer point of view. You will see the minimum steps of adding and using WebAssembly into your JavaScript/TypeScript project.

## Setup

First, you will need the Rust toolchain, including rustup, rustc, and cargo for compiling Rust code, and [wasm-pack](https://rustwasm.github.io/wasm-pack/) for building, testing and publishing Rust-generated WebAssembly.

### Install Rust

To install Rust on Mac/Linux, run the following command in the terminal.
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
On Windows, I enabled the [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10) and used Rust in the Linux terminal.

### Install wasm-pack

Once installed Rust, run the following command in the terminal.
```
cargo install wasm-pack
```
Believe it or not, that's all you need to create WebAssembly. Let's go back to the JavaScript/TypeScript world.

* If you start from scratch, follow the next section to create an AppRun project.
* If you already have an existing project, jump to the section of Create WebAssembly Project.


### Create AppRun Project

Run the commands to create an AppRun project:

```
mkdir your-app-name
cd your-app-name
npx apprun -i
```

Wait a few minutes for installing the npm packages, and then run the npm command:

```
npm start
```

You will see a hello world application running.

![AppRun Hello World](https://dev-to-uploads.s3.amazonaws.com/i/0v0lkfuto7d4lxh1639i.png)
<figcaption>AppRun Hello World</figcaption>

Next, we will add WebAssembly to this project.

### Create WebAssembly Project

Let's create a Rust project by running the following command:

```
cargo new wasm --lib
```

The command creates a folder called _wasm_ and two files under the folder _your-app-name/wasm_: _Cargo.toml_ and _src/lib.rs_.

It is a regular Rust project, not a WebAssembly yet. You will need to add _wasm-bindgen_ as the dependency to make it target WebAssembly. Open _Cargo.toml_ and add the following sections.

```
[lib]
crate-type = ["cdylib"]

[dependencies]
wasm-bindgen = "0.2.60"
js-sys = "0.3.37"
```

> [wasm-bindgen](https://rustwasm.github.io/) is a Rust library that facilitates high-level interactions between wasm modules and JavaScript. [js-sys](https://crates.io/crates/js-sys) is the waw bindings to JS global APIs for projects using wasm-bindgen.

Now, you can use wasm-pack to build a WebAssembly.

```
cd wasm
wasm-pack build
```

### Use WebPack

Since the AppRun project is a WebPack project, we can use the [wasm-pack-plugin](https://github.com/wasm-tool/wasm-pack-plugin) to unify the build process that creates the WebAssembly and JavaScript code at the same time. Go ahead to add the package:

```
npm i @wasm-tool/wasm-pack-plugin -D
```

And add the wasm-pack-plugin into the _webpack.config.js_.

```js
const WasmPackPlugin = require("@wasm-tool/wasm-pack-plugin");
module.exports = {
  ...
  plugins: [
    new WasmPackPlugin({
      crateDirectory: path.resolve(__dirname, ".")
    }),
  ]
  ...
}
```

Also, because the wasm-pack-plugin generates the _dynamic import_ module, you need to modify _tsconfig.json_ file to set the module to be _esnext_.

```json
{
  "compilerOptions": {
    ...
    "module": "esnext",
    ...
  }
}
```

Finally, the npm scripts: _npm start_ and _npm run build_ will build the TypeScript code as well the Rust code.

Let's write some Rust code.


## WebAssembly and AppRun

We will demonstrate two interactions between the WebAssembly and the AppRun application.

* Call the WebAssembly from the AppRun application
* Call the AppRun application from the WebAssembly

### Call WebAssembly

First, we create a Rust function in the _wasm/src/lib.rs_ file.

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn add(a: i32, b: i32) -> i32 {
  a + b
}
```

This function adds two numbers. We can make a counter application from it. Here is the counter application in AppRun.

```js
import app from 'apprun';

let wasm;
import('../wasm/pkg').then(module => wasm = module);

const state = {
  title: 'Hello world - AppRun !',
  count: 0
}

const add = (state, num) => ({
  ...state,
  count: wasm.add(state.count, num)
});

const view = ({ title, count }) => <>
  <h1>{title}</h1>
  <h1>{count}</h1>
  <button $onclick={[add, -1]}>-1</button>
  <button $onclick={[add, +1]}>+1</button>
</>;

app.start(document.body, state, view);
```

You can see from the code above:

* Wasm-pack has created a JavaScript module that we can import dynamically.
* We can call the WebAssembly function just like a regular JavaScript function from a module.

Running the application, we have a counter that uses the WebAssembly function.

![Counter with WASM](https://dev-to-uploads.s3.amazonaws.com/i/dmy2qkqqtlyxed4qovlf.png)
<figcaption>Counter with WASM</figcaption>

Next, let's see how does the WebAssembly function call AppRun functions.

### Call the AppRun

Open _wasm/src/lib.rs_ file and add the following functions.

```rust
#[wasm_bindgen]
extern "C" {
  #[wasm_bindgen(js_namespace = app)]
  fn run(event: &str, p: &str);
}

#[wasm_bindgen(start)]
pub fn start() {
  run("@hello", "hello world from rust");
}
```

* The first function named _run_ binds to the AppRun _app.run_ function.
* The second function named _start_ runs automatically when the WebAssembly is loaded.
* The _start_ function calls the _run_ function to send a '@hello' event to AppRun.

Back to AppRun code, we will handle the '@hello' event.

```js
import app from 'apprun';

let wasm;
import('../wasm/pkg').then(module => wasm = module);

const state = {...}

const add = (state, num) => ({...});

const view = ({ title, count }) => <>...</>;

const update = {
  '@hello': (state, title) => ({...state, title})
}

app.start(document.body, state, view, update);
```

Now, when the application starts, it displays the messages sent from the WebAssembly.

![Message from WASM](https://dev-to-uploads.s3.amazonaws.com/i/atljaqot7q08qcu9rk9m.png)
<figcaption>Message from wasm</figcaption>

We have successfully made two-way interactions between the WebAssembly and the AppRun application.

## Souce Code

You can run the live demo: https://yysun.github.io/apprun-rust.

Or visit the source.

{% github yysun/apprun-rust %}

You also can use this project as an AppRun application template. Run the command to create your application.

```
npx degit yysun/apprun-rust my-app
```

## Conclusion

This post should give you a quick start to use Rust/WebAssembly in the AppRun applications. The demo project shows the two technologies interact with each other very well. You can use the demo project as a template.

We have now opened the door to a new world. There is much more potential to explore.
