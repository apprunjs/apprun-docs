## Introduction

The recently updated [AppRun Docs Site](https://apprun.js.org/docs) has made the code snippets in the documents runnable and editable, making the technical documentation interactive and much more fun to use.

The site is built with [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/), a beautiful and powerful tool for building technical documentation sites. We extended it by adding a web component built with [AppRun](https://apprun.js.org) to deliver interactive experiences.

In this post, I Will explain how it's made. Let's start with reviewing the user experience.

## User Experience

Technical documents usually have code snippets. Often the code has syntax highlighted for easy reading. However, users usually can only see screenshots but not live results of the code. Screenshots have limitations. For example, when describing how to make animation, a static screenshot is not helpful. We need a way to display the live code execution results.

### See the Results

You can visit the [AppRun Docs Page](https://apprun.js.org/docs/directive/#custom-directive) to see a live animation.

![run code](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1da6f59c33jatqyo0imc.png)

### Try the Code

Furthermore, users might have been inspired by the code examples and want to try different ideas. Traditionally, they could copy and paste the code to run it in their code editors. It would be nice for users to edit the code right on the doc site and see the results.

You can click the "Try the Code" button of the [AppRun Docs Page](https://apprun.js.org/docs/directive/#custom-directive). It opens the AppRun Playground with an editor and preview pane to play the code.


![edit code](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y0dc7t2lw0txqab2vus7.png)

The user experiences have improved with the capabilities of seeing the code results and trying the code in technical documents.


## Author Experience

Not only is it much more attractive to the readers, but also the authors will feel it is much more enjoyable when writing the documents.

### Present the Live Code

Traditionally, authors copy and paste the code snippets from their testing projects into the markdown documents as code blocks. The limitation is that they can only present the code but not the running code. Sometimes, it would be hard to describe the code behavior. For example, describing a calculator could need a long text, but it could be easier to present the calculator for users to click.

You can visit the [AppRun Docs Page](https://apprun.js.org/docs/architecture-ideas/state-machine/#model-a-calculator) to see a running calculator.

![hide source](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yx244b2z9vkjooyl1f7i.png)

All we need to do is to add a web component, called _apprun-play_ under the code blocks.


```` markdown
```js
// code snippets
```
<apprun-play></apprun-play>
```
````

### Control the Presentation

You probably have noticed that the page shows only the results but not the source code. It is because we can control whether to show the source code. We can also decide whether to see the "Try the Code" button.

```` markdown
```js
// code snippets
```
<apprun-play hide_src="true" hide_button="true"></apprun-play>
````

You can visit the [AppRun Docs Page](https://apprun.js.org/docs/architecture/#ceremony-vs-essence) to see an example of only displaying the running results.

![hide try-the-code button](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ldvvrmu1juty4jt7ezvo.png)


We can present the code snippets, but we can also embed whole applications because the _apprun-play_ web component supports HTML.

![Embed Apps](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nfwlatbkx39bpijwihhl.png)

We can use the [embedding external files](https://squidfunk.github.io/mkdocs-material/reference/code-blocks/#embedding-external-files) feature of [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/). This way, the markdown document does not include the source code and can remain simple and clean.
Automatic Test of the Code

When it displays the code result automatically means automatic testing of the code, which tells the author if the code works as expected.

Also, while writing, the authors can come up with new ideas. With _apprun-play_ web component, they can edit the code and see the live results. Once it's done, they can copy and paste the code back into the document.

Overall, the _apprun-play_ web component is a helpful tool for the document author.

## How It's Made

Web components/custom elements are safe in the markdown documents. We can build web components out of the [AppRun Components](https://apprun.js.org/docs/component/) quickly.

The _apprun-play_ web component is an AppRun component that gets the source code from its previous sibling element, a _textarea_, or a _div_ with highlighted code. Then, the _apprun-play_ web component creates an iframe for the code.

You can find the [source code here](https://github.com/yysun/apprun/blob/master/src/apprun-play.tsx).

You can [download compiled the _apprun-play_ web component](https://raw.githubusercontent.com/yysun/apprun/master/dist/apprun-play.js) and add it to the configuration file of Material for MkDocs, _mkdocs.yml_

```yml
extra_css:
  - assets/vendor/codemirror/codemirror.css

extra_javascript:
  - assets/vendor/codemirror/codemirror.js
  - assets/vendor/codemirror/mode/javascript/javascript.js
  - assets/vendor/codemirror/mode/xml/xml.js
  - assets/vendor/codemirror/mode/jsx/jsx.js
  - assets/apprun-play.js
```

That's it. The _apprun-play_ web component is ready for use in all the markdown documents.

Finally, the AppRun Docs Site Github project is: https://github.com/apprunjs/apprun-docs/

Please enjoy and send pull requests.
