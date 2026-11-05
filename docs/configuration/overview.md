---
title: Configuration overview
description: How rapidoc.config.yaml is organized, its conventions, and editor autocomplete.
---

Everything about the site lives in `rapidoc.config.yaml` at the project root. Only `site.title` is required.

```yaml title="rapidoc.config.yaml"
# yaml-language-server: $schema=./node_modules/@rapidoc/portal/schema.json
site:        # title, logo, favicon, banner
theme:       # accent color, light/dark mode, fonts
header:      # row 1: links and menus right of the search
navigation:  # row 2: one tab per entry; documentation tabs have a sidebar
apis:        # your OpenAPI specs
docs:        # docs folder, table of contents, edit links
footer:      # call to action, columns, social links, newsletter
rapidoc:     # <rapi-doc> attributes for API reference pages
search:      # full-text search on or off
redirects:   # old URLs to new ones
auth:        # OpenID Connect sign-in (optional)
access:      # who may read what (optional)
```

## Conventions

The same three words mean the same thing everywhere:

- `label` is the text of anything clickable.
- `href` is any URL: a path on this site (`/docs/intro`) or a full URL.
- `items` are children: dropdown items, mega-menu links, sidebar groups.

A navigation item's kind comes from its keys: `slug` (or a plain string), `href`, `items`, `autogenerate`, `api` or `catalog`.

## Strict by design

The schema rejects unknown keys. A typo fails the build with its path and, where it can, the key you meant:

```text
navigation[0].items[2]: unknown key "path" (use href (links) or slug (doc pages))
```

## Autocomplete in your editor

The first line of the file points editors that understand YAML schemas (VS Code with the YAML extension, JetBrains IDEs) at the JSON Schema that ships as `@rapidoc/portal/schema.json`:

```yaml title="rapidoc.config.yaml"
# yaml-language-server: $schema=./node_modules/@rapidoc/portal/schema.json
```

You get completion, descriptions on hover, and errors as you type.

## Other formats

The engine also reads `rapidoc.config.yml`, `.json`, `.js`, `.mjs` and `.ts`. A JavaScript or TypeScript config default-exports the same object.

## The sections

- [Site](/docs/configuration/site/): title, logo, favicon, URL, banner
- [Theme](/docs/configuration/theme/): accent color, mode, fonts
- [Header](/docs/configuration/header/): links, dropdowns, mega-menus
- [Navigation](/docs/configuration/navigation/): tabs and sidebars
- [APIs](/docs/configuration/apis/): specs, ids, categories
- [Workflows](/docs/configuration/workflows/): Arazzo files, drawn and runnable in the docs
- [Footer](/docs/configuration/footer/)
- [Docs, search and redirects](/docs/configuration/docs-search-redirects/)
- [Sign-in and access](/docs/access/overview/)

Every key in one table: [Configuration keys](/docs/reference/config/).
