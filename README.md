# RapiDoc Portal documentation

The documentation site for [RapiDoc Portal](https://github.com/rapi-doc/RapiDoc) (`@rapidoc/portal`, `@rapidoc/cli`), built with RapiDoc Portal itself.

## Run it locally

The portal packages are not on npm yet, so this repository runs the CLI from a checkout of the RapiDoc monorepo next to it:

```text
work/
├── rapidoc/    # https://github.com/rapi-doc/RapiDoc (npm install, npm run build:rapidoc)
└── rapi-portal-doc/    # this repository
```

```bash
npm run dev       # http://localhost:4321
npm run build     # static site in dist/
npm run serve     # serves dist/ on http://localhost:8080 (PORT=8090 npm run serve for another port)
npm run preview   # build, then serve
npm run validate  # checks specs/bookstore.yaml
npm run mock      # mock server for the example API on http://localhost:4010
```

Once the packages are published, the scripts become `rapidoc dev`, `rapidoc build`, ... with `@rapidoc/cli` and `@rapidoc/portal` as dev dependencies.

## Layout

```text
rapidoc.config.yaml   # site, header, tabs, sidebar
docs/                 # the pages (Markdown and Markdoc); docs/index.markdoc is the home page
specs/bookstore.yaml  # the example API shown in the "Example API" tab
public/               # logo and other static files
```
