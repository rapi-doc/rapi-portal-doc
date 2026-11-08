---
title: Single sign-on (SSO) and access
description: Put the portal behind your company's single sign-on (SSO) with OpenID Connect (Okta, Microsoft Entra ID, Google, Keycloak, Auth0), and decide who may read which pages and APIs.
---

Most portals are public. When some guides or APIs are only for partners, customers or your own staff, readers can sign in with your company's single sign-on (SSO): Okta, Microsoft Entra ID (Azure AD), Google Workspace, Keycloak, Auth0, Ping or any other OpenID Connect provider. SAML-only providers work through a broker ([Identity providers](/docs/access/providers/#saml-only-providers)).

Add two keys to the config:

- `auth`: readers sign in through your OpenID Connect provider.
- `access`: rules that decide who may read which pages, APIs and files.

```yaml title="rapidoc.config.yaml"
auth:
  issuer: https://acme.okta.com/oauth2/default
  clientId: portal-web
  scopes: [openid, profile, email, groups]

access:
  default: public
  rules:
    - path: /docs/partners/**
      allow: signed-in
    - path: /apis/billing/**
      allow: { groups: [billing-team] }
```

## Choose your setup

| You need | Setup |
| :--- | :--- |
| Some pages or APIs public, others only for some readers or groups | `auth` and `access`, served by `rapidoc serve` (this page) |
| The whole portal behind SSO, every signed-in employee sees everything | Either `auth` with `access.default: signed-in`, or a plain static build behind your SSO proxy, with no portal server ([Static hosting](/docs/deploy/static-hosting/#the-whole-portal-behind-your-sso)) |

Step by step for one provider: [SSO with Microsoft Entra ID](/docs/access/entra-id/).

## Why a server for per-page rules?

A static site can show a sign-in button and hide links in the browser, but its files are still there for anyone who asks for them: pages, OpenAPI specs, attachments and the search index can be downloaded with their URL, signed in or not. That is fine for a portal whose content is public anyway, and not enough when partner APIs or internal guides must stay private.

`rapidoc serve` checks the reader's session before it sends a protected page, spec, file or search result. Content a reader may not see never leaves the server: protected pages are kept apart from public ones at build time (`dist/protected/`), and search results are filtered per reader. The public part stays plain static files, sent with cache headers so a CDN in front can hold the scripts, styles and fonts.

## How it works

A static host can't check who is asking, so a portal with `auth` is served by `rapidoc serve` (`@rapidoc/server`):

1. `rapidoc build` sorts the output by the rules: `dist/public/` (safe to host anywhere, with no protected content), `dist/protected/`, and `dist/rapidoc-server.json` with the rules.
2. `rapidoc serve dist` serves both, checks the reader's session on every request, and sends readers to your provider to sign in.

The pages stay static. The header's sign-in button and the lock icons ask the server at runtime.

Agents get the same treatment: `rapidoc serve` hosts the APIs as MCP tools at `/mcp`, and a signed-in reader can give their agent a token from the user menu. See [MCP server for agents](/docs/tools/mcp-server/).

```bash
RAPIDOC_SESSION_SECRET=... RAPIDOC_AUTH_CLIENT_SECRET=... \
  npx rapidoc serve dist --port 8080
```

See [Deploy with rapidoc serve](/docs/deploy/rapidoc-serve/).

## The `auth` key

| Key | Meaning |
| :--- | :--- |
| `issuer` | The OpenID Connect issuer URL. `RAPIDOC_AUTH_ISSUER` overrides it. |
| `clientId` | The portal's client ID at the provider. `RAPIDOC_AUTH_CLIENT_ID` overrides it. |
| `scopes` | Scopes to request. Default `openid`, `profile`, `email`. |
| `claims.groups` | The claim with the reader's groups or roles. Default `groups`. Dotted paths work: `realm_access.roles`. |
| `session.maxAge` | How long a sign-in lasts: `30m`, `8h` (default), `7d`. |
| `logout` | `local` (default) signs out of the portal; `idp` also signs out at the provider. |
| `label` | Text of the sign-in button. Default `Sign in`. |
| `audience` | The API the access token is for, sent as `audience` at sign-in (Auth0). `RAPIDOC_AUTH_AUDIENCE` overrides it. |
| `tryIt` | Give **Try** the reader's access token. See [Try with the reader's token](/docs/access/try-it/). |

:::caution[Secrets are never config keys]
The session key and the client secret are environment variables of `rapidoc serve`: `RAPIDOC_SESSION_SECRET` and `RAPIDOC_AUTH_CLIENT_SECRET`. The config is committed to your repository; secrets must not be.
:::

## At your provider

Create a web application (OpenID Connect, authorization code flow) and register the redirect URL:

```text
<portal URL>/_rapidoc/auth/callback
```

Provider notes: [Identity providers](/docs/access/providers/).

## Testing locally

`rapidoc dev` doesn't enforce the rules. To test them, build and serve:

```bash
npx rapidoc build && npx rapidoc serve dist
```
