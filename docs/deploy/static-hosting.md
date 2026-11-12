---
title: Static hosting
description: Deploy the built portal to GitHub Pages, Netlify, Vercel, Cloudflare Pages or any web server.
---

Without `auth`, `rapidoc build` writes a plain static site to `dist/`. Host it anywhere that serves files.

```bash
npx rapidoc build                 # dist/
npx rapidoc build --out-dir public # another folder
```

Set `site.url` to the portal's public URL, for canonical links and the sitemap. To host the portal in a folder of another site, put the folder in the URL (`https://acme.com/developers`): links are built with `/developers` in front, and you upload `dist/` into that folder ([Serving under a path](/docs/configuration/site/#serving-under-a-path)).

## GitHub Pages

```yaml title=".github/workflows/deploy.yml"
name: Deploy portal
on:
  push:
    branches: [main]
permissions:
  contents: read
  pages: write
  id-token: write
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - run: npm ci
      - run: npx rapidoc validate
      - run: npx rapidoc build
      - uses: actions/upload-pages-artifact@v3
        with:
          path: dist
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
    steps:
      - uses: actions/deploy-pages@v4
```

The portal's links start at `/`, so serve it from the root of a domain: a custom domain, or a `<user>.github.io` repository.

## Netlify, Vercel, Cloudflare Pages

| Setting | Value |
| :--- | :--- |
| Build command | `npx rapidoc build` |
| Output folder | `dist` |
| Node.js version | 22 or later |

## Any web server

Copy `dist/` to the server's web root (Nginx, Apache, Caddy, S3 with CloudFront). Every page is a folder with an `index.html`, so the server needs no rewrite rules.

## Need sign-in (SSO)?

A static host can't check who is reading. There are two ways to put a portal behind your company's single sign-on.

If only calls to your API need a sign-in, and the documentation stays public, you need neither: readers sign in to your API from the API page itself, in the browser. See [OAuth 2 sign-in for Try](/docs/api-reference/reference-pages/#oauth-2-sign-in-for-try).

### The whole portal behind your SSO

When every signed-in employee may read everything, keep the plain static build and let the layer in front of it handle sign-in. No portal server runs:

- **Cloudflare Access** in front of Cloudflare Pages or any origin.
- **AWS**: an Application Load Balancer with OpenID Connect authentication in front of the files, or CloudFront with an authenticating edge function.
- **Your own server**: [oauth2-proxy](https://oauth2-proxy.github.io/oauth2-proxy/) in front of Nginx or Caddy.
- **Your host's own protection**, such as Vercel or Netlify on plans that offer SSO for deployments.

Leave `auth` and `access` out of the config: the portal doesn't know about the sign-in, so it shows no sign-in button or user menu, Try doesn't get the reader's token, and there is no MCP endpoint for agents.

### Some pages for some readers

For per-page and per-group rules, a sign-in button, Try with the reader's own token and MCP tools under the same rules, configure [`auth` and `access`](/docs/access/overview/) and deploy with [`rapidoc serve`](/docs/deploy/rapidoc-serve/). Public pages can still be cached by a CDN in front of it.
