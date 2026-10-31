---
title: Static hosting
description: Deploy the built portal to GitHub Pages, Netlify, Vercel, Cloudflare Pages or any web server.
---

Without `auth`, `rapidoc build` writes a plain static site to `dist/`. Host it anywhere that serves files.

```bash
npx rapidoc build                 # dist/
npx rapidoc build --outDir public # another folder
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

## Need sign-in?

A static host can't check who is reading. With `auth` and `access`, deploy with [`rapidoc serve`](/docs/deploy/rapidoc-serve/) instead.
