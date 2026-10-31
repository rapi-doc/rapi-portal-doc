---
title: Introduction
description: What RapiDoc Portal is, what it builds, and which package does what.
---

RapiDoc Portal is a docs-as-code developer portal framework. You keep your OpenAPI specs and your guides in a repository, describe the site in one `rapidoc.config.yaml`, and the `rapidoc` command builds a static portal from them.

A portal has:

- **A home page** from `docs/index.md`, `.mdx` or `.markdoc`.
- **Guides** under `/docs/...`, written in Markdown, MDX or Markdoc.
- **An interactive API reference** for each spec at `/apis/<id>`, rendered by the [RapiDoc](https://rapidocweb.com) web component.
- **An API catalog** at `/catalog`, with every API grouped by category.
- **Full-text search** across guides and API operations.
- **A header with tabs**, where each documentation tab has its own sidebar, plus a footer, a banner and light and dark themes.

The output is plain HTML, CSS and JavaScript. Host it on GitHub Pages, Netlify, Vercel, Cloudflare Pages, S3 or any web server. If some pages or APIs must stay private, add sign-in and access rules and serve the build with `rapidoc serve`.

## The packages

| Package | What it does |
| :--- | :--- |
| `@rapidoc/cli` | The `rapidoc` command: `init`, `dev`, `build`, `serve`, `mock`, `validate`, `migrate`. The only command you run. |
| `@rapidoc/portal` | The engine behind `rapidoc dev` and `rapidoc build`: config, layout, Markdoc tags, search. |
| `@rapidoc/server` | Behind `rapidoc serve`: serves a built portal with OpenID Connect sign-in and access rules. Optional. |
| `@rapidoc/mock` | Behind `rapidoc mock`: a mock server for the portal's APIs, with request validation and token checks. Optional. |
| `create-rapidoc` | `npm create rapidoc` scaffolds a new portal project. |
| `rapidoc` | The `<rapi-doc>` web component that renders each API reference. |

The CLI loads `@rapidoc/portal`, `@rapidoc/server` and `@rapidoc/mock` only for the commands that need them. `rapidoc init` and `rapidoc validate <spec>` work without `@rapidoc/portal` installed.

## Why another portal framework?

- **Docs as code.** Everything is a file in your repository: review it in pull requests, build it in CI.
- **Static by default.** No database and no runtime to operate. Sign-in is the one feature that needs a server, and it's opt-in.
- **Strict config.** An unknown key fails the build with its path and a hint, so a typo never disappears silently.
- **Accessible.** The layout is tested against WCAG 2.1 AA in both themes and at phone width, and accent text colors are derived from your brand color so they keep 4.5:1 contrast.

Next: [build your first portal](/docs/getting-started/quickstart/).
