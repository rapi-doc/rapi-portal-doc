---
title: Project structure
description: Where the config, specs, guides and static files go, and how files become URLs.
---

```text
my-portal/
├── rapidoc.config.yaml     # Site, header, navigation tabs, theme, APIs
├── public/                 # Static files served from / (logo, favicon, images)
├── specs/                  # OpenAPI 3.0 / 3.1 / Swagger 2.0 files (YAML or JSON)
│   └── orders.yaml
└── docs/                   # Guides: .md, .mdx, .markdoc (sub-folders become URL segments)
    ├── index.markdoc       # the home page (optional)
    ├── getting-started.md
    └── guides/
        └── checkout.mdx
```

## `rapidoc.config.yaml`

The one config file. The engine also reads `rapidoc.config.yml`, `.json`, `.js`, `.mjs` and `.ts`. See [Configuration](/docs/configuration/overview/).

## `docs/`

Every file is a page. The path decides the URL:

| File | URL |
| :--- | :--- |
| `docs/index.markdoc` | `/` (the home page) |
| `docs/getting-started.md` | `/docs/getting-started` |
| `docs/guides/checkout.mdx` | `/docs/guides/checkout` |

In the config, a page is named by its path without the extension: `guides/checkout`. Without a `docs/index.*`, `/` opens the first tab.

Change the folder with `docs.dir` in the config. If both `page.md` and `page.mdx` exist, the `.mdx` one is used.

## `specs/`

OpenAPI files, referenced from `apis:` in the config. Relative `$ref`s to other files resolve. A spec can also be a URL. Each API is published at `/specs/<api id>.<ext>`, so readers can download it.

## `public/`

Copied to the site root as-is: `public/logo.svg` is `/logo.svg`. Put the logo, favicon and images for guides here.

## What a build leaves behind

Only the output folder, `dist/`. While it runs, a build works in a temporary folder `.rapidoc-temp/` inside the project and removes it at the end, even when the build fails. Add both to `.gitignore` (the starter does):

```text
node_modules/
dist/
.rapidoc-temp/
```
