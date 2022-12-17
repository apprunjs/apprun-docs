# AppRun-Site Command Line

The AppRun-site command line has the following commands:

```
Usage: apprun-site [options] [command]

Options:
  -h, --help                  display help for command

Commands:
  init [options] [targetDir]  initialize a new app
  build [options] [source]    build site
  serv [options] [source]     launch development server, watch and no live reload
  dev [options] [source]      launch development server, watch and live reload
  help [command]              display help for command
```

## Build Command Options

You can use the following command-line options for the _build_ command.

```sh
Usage: apprun-site build [options] [source]

build site

Options:
  -c, --clean            clean the output directory (default: false)
  -w, --watch            watch the directory (default: false)
  -r, --render           pre-render html pages (default: false)
  -o, --output [output]  output directory (default: "public")
  -p, --pages [pages]    pages directory (default: "pages")
  -h, --help             display help for command
```

## Dev Command Options

```sh
Usage: apprun-site dev [options] [source]

launch development server, watch and live reload

Options:
  -o, --output [output]  output directory (default: "public")
  -p, --pages [pages]    pages directory (default: "pages")
  -n, --no_ssr           disable server side rendering (default: false)
  -h, --help             display help for command
➜  apprun git:(master) ✗ npx apprun-site@latest serve -h
```

## Serv Command Options

```
Usage: apprun-site serv [options] [source]

launch development server, watch and no live reload

Options:
  -o, --output [output]  output directory (default: "public")
  -p, --pages [pages]    pages directory (default: "pages")
  -n, --no_ssr           disable server side rendering (default: false)
  -h, --help             display help for command
```

## AppRun-Site Config File

In addition, you can also customize the AppRun-Site in the `apprun-site.yml` config file.

```yml
site_name: AppRun Site
site_url: /

no-startup: true   # don't inject startup code for dynamic routing
no-sss: true       # don't use server-side-rendering

dev-server:
  port: 8080

static-pages:     # additional pages to pre-render for the static site
  - /products/1
  - /products/2
  - /products/3
```

WIP, more options to come.