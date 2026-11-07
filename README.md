# RapiDoc Portal documentation

The documentation site for [RapiDoc Portal](https://github.com/rapi-doc/RapiDoc) (`@rapidoc/portal`, `@rapidoc/cli`), built with RapiDoc Portal itself.

## Run it locally

Node.js 22.12 or later. The site uses the published packages (`@rapidoc/cli`, `@rapidoc/portal`, `@rapidoc/server`, `@rapidoc/mock`):

```bash
npm install
npm run dev       # http://localhost:4321
npm run build     # static site in dist/
npm run serve     # serves dist/ on http://localhost:8080 (PORT=8090 npm run serve for another port)
npm run preview   # build, then serve
npm run validate  # checks specs/bookstore.yaml
npm run mock      # mock server for the example API on http://localhost:4010
```

Pushes to `main` deploy the site to https://portal.rapidocui.com (`.github/workflows/deploy.yml`).

To try unreleased portal changes, install from a checkout of the [RapiDoc monorepo](https://github.com/rapi-doc/RapiDoc) next to this one, e.g. `npm install -D ../rapidoc/packages/rapidoc-portal`, and don't commit that change.

## Layout

```text
rapidoc.config.yaml   # site, header, tabs, sidebar
docs/                 # the pages (Markdown and Markdoc); docs/index.markdoc is the home page
specs/bookstore.yaml  # the example API shown in the "Example API" tab
public/               # logo and other static files
```
