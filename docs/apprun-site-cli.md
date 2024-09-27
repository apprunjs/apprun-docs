# AppRun-Site Command Line

The AppRun-site command line has the following commands:

```
Usage: apprun-site [options] [command]

Options:
  -h, --help                  display help for command

Commands:
  init [destination]        Initialize a new project from a template
  build [options] [source]  build site
  serve [options] [source]  launch preview server, live reload is optional
  dev [options] [source]    launch development server, watch and live reload
  help [command]            display help for command
```

## Build Command Options

You can use the following command-line options for the _build_ command.

```sh
Usage: apprun-site build [options] [source]

build site

Options:
  -c, --clean            clean the output directory (default: false)
  -w, --watch            watch the directory (default: false)
  -o, --output [output]  output directory (default: "public")
  -p, --pages [pages]    pages directory (default: "pages")
  -s --server-only       build server app (default: false)
  -b --client-only       build server app (default: false)
  -r --render            pre-render pages (default: false)
  --no-csr               no client side routing
  -h, --help             display help for command
```

## Dev Command Options

```sh
Usage: apprun-site dev [options] [source]

launch development server, watch and live reload

Options:
  -o, --output [output]  output directory (default: "public")
  -p, --pages [pages]    pages directory (default: "pages")
  --no-watch             no watching the directory
  --no-live_reload       no live reload
  --no-csr               no client side routing
  -h, --help             display help for command
```

## Serve Command Options

```sh
Usage: apprun-site serve [options] [source]

launch development server, live reload is optional

Options:
  -o, --output [output]  output directory (default: "public")
  --no-ssr               disable server side rendering
  --no-save              disable auto save of side rendered pages
  -h, --help             display help for command
```

## AppRun-Site Config File

In addition, you can also customize the AppRun-Site in the `apprun-site.config.js` config file.

```js
export default {
  port: 8080,
  site_url: '/',
  static_pages: ['/products/1', '/products/2', '/products/3'],
}

```

WIP, more options to come.