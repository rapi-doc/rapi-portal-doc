---
title: Deploy with rapidoc serve
description: Run a portal with sign-in and access rules on Node or in Docker.
---

A portal with `auth` is served by `rapidoc serve` (`@rapidoc/server`). It serves the build, signs readers in through your provider, and enforces the [access rules](/docs/access/access-rules/) on every page, spec, file and search result.

```bash
npm install -D @rapidoc/cli @rapidoc/portal @rapidoc/server
npx rapidoc build
RAPIDOC_SESSION_SECRET=... RAPIDOC_AUTH_CLIENT_SECRET=... RAPIDOC_PUBLIC_URL=https://developers.acme.com \
  npx rapidoc serve dist --port 8080
```

- **Stateless**: the session is an encrypted cookie, so any number of instances can run behind a load balancer. No database.
- **Small**: it needs only Node, the CLI, `@rapidoc/server` and the build. Not `@rapidoc/portal`.

## Environment variables

| Variable | Required | Meaning |
| :--- | :--- | :--- |
| `RAPIDOC_SESSION_SECRET` | with `auth` | Encrypts the session cookie. 32 or more characters, the same on every instance. Changing it signs everyone out. |
| `RAPIDOC_AUTH_CLIENT_SECRET` | confidential clients | Leave unset for a public client (PKCE only). |
| `RAPIDOC_PUBLIC_URL` | recommended | The portal's external URL: the redirect URL, and whether cookies are `Secure`. Default `site.url`, else the request's host. |
| `RAPIDOC_AUTH_ISSUER`, `RAPIDOC_AUTH_CLIENT_ID`, `RAPIDOC_AUTH_AUDIENCE` | no | Override `auth.issuer`, `auth.clientId`, `auth.audience`. |
| `RAPIDOC_MOUNT_PATH` | no | Path requests arrive under, when a proxy strips part of the portal's path. `--mount-path` wins. Default: the path of `site.url`. |
| `PORT`, `HOST` | no | Default `8080` and `0.0.0.0`. `--port` and `--host` win. |

The server refuses to start, with the reason, when a required value is missing or the provider can't be discovered.

## Behind a proxy that strips part of the path

A portal with `site.url: https://acme.com/sales/developers` is built and served under `/sales/developers`. When a gateway or ingress routes `/sales/*` to the server and strips `/sales`, requests arrive as `/developers/...`. Tell the server:

```bash
npx rapidoc serve dist --mount-path /developers
```

Links, redirects and the sign-in callback keep the full path the browser sees, `/sales/developers/...`. Without a proxy, the server answers under the path of `site.url` by itself.

## Docker

```dockerfile title="Dockerfile"
FROM node:22-alpine AS build
WORKDIR /portal
COPY package*.json ./
RUN npm ci
COPY . .
RUN npx rapidoc build

FROM node:22-alpine
WORKDIR /portal
COPY package*.json ./
RUN npm ci --omit=dev && npm install --no-save @rapidoc/cli @rapidoc/server
COPY --from=build /portal/dist ./dist
ENV PORT=8080
EXPOSE 8080
CMD ["npx", "rapidoc", "serve", "dist"]
```

Health check for load balancers and Kubernetes: `GET /_rapidoc/health`.

## The build output

With `auth`, `rapidoc build` writes:

| Path | Holds |
| :--- | :--- |
| `dist/public/` | Everything anyone may read. Contains no protected content, so it can sit on a CDN. |
| `dist/protected/` | Pages and files behind a rule. |
| `dist/rapidoc-server.json` | The rules and zones the server enforces. |

Responses that depend on the session are `Cache-Control: private, no-store` with `Vary: Cookie`, so a CDN in front never caches a protected page for the wrong reader.

## Without `auth`

`rapidoc serve` also serves a plain static build, which is handy for a local check of the production build:

```bash
npx rapidoc build && npx rapidoc serve dist
```
