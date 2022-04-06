# AppRun-Site Configurations

There are several options and configurations for AppRun-Site.

## Build Command Options

You can use the following command line options for the _build_ command.

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

## Dev Server Options

You can customize dev server in the `apprun-site.yml` config file.

```yml
site_name: AppRun Site
site_url: /

no-startup: true   # don't inject start up code for dynamic routing

dev-server:
  port: 8080

static-pages:     # additional pages to pre-render for the static site
  - /products/1
  - /products/2
  - /products/3
```

WIP, more options to come.