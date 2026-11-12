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
| The whole portal behind SSO, every signed-in reader sees everything | Either `auth` with `access.default: signed-in`, or a plain static build behind your SSO proxy, with no portal server ([Static hosting](/docs/deploy/static-hosting/#the-whole-portal-behind-your-sso)) |

Step by step for one provider: [SSO with Microsoft Entra ID](/docs/access/entra-id/).

:::caution[Who counts as "signed in"]
`signed-in` lets in everyone your provider lets sign in, not only your employees. Depending on how the provider is set up, that can include anyone with an account: self sign-up in Auth0 or Cognito, any Google account when the client isn't limited to your Workspace domain, other organizations in a multi-tenant Entra ID app, and guest users. Limit sign-in at the provider (in Entra ID, set **Assignment required?** to *Yes*; in Okta, assign the app to groups; turn off self sign-up), or use [group rules](/docs/access/access-rules/) instead of `signed-in`.
:::

## Why a server for per-page rules?

A static site can show a sign-in button and hide links in the browser, but its files are still there for anyone who asks for them: pages, OpenAPI specs, attachments and the search index can be downloaded with their URL, signed in or not. That is fine for a portal whose content is public anyway, and not enough when partner APIs or internal guides must stay private.

`rapidoc serve` checks the reader's session before it sends a protected page, spec, file or search result. The contents of protected pages, specs and files never leave the server for a reader who may not see them: they are kept apart from public ones at build time (`dist/protected/`), and search results are filtered per reader. The public part stays plain static files, sent with cache headers so a CDN in front can hold the scripts, styles and fonts.

What stays visible: the titles and paths of protected pages. Locked links keep their place in the header, sidebars and catalog, with a lock icon, so readers know there is more behind sign-in. Don't put anything confidential in a page title, a sidebar label or a URL.

## How it works

A static host can't check who is asking, so a portal with `auth` is served by `rapidoc serve` (`@rapidoc/server`):

1. `rapidoc build` sorts the output by the rules: `dist/public/` (no protected content), `dist/protected/`, and `dist/rapidoc-server.json` with the rules.
2. `rapidoc serve dist` serves all three, checks the reader's session on every request, and sends readers to your provider to sign in.

:::danger[Never upload all of dist/ to a static host]
A static host serves every file it has, so `dist/protected/` would be readable by anyone. Run `rapidoc serve` on the whole `dist/`, or put only `dist/public/` on a static host or CDN.
:::

The pages stay static. When a page loads, the header's sign-in button and the lock icons ask the server who the reader is.

`rapidoc serve` also hosts your APIs as tools for AI agents, at `/mcp` ([Model Context Protocol](/docs/tools/mcp-server/)). It is always on. An agent without a token sees only the tools of public APIs. A signed-in reader can copy a token for their agent from the user menu: with it, the agent sees the tools of the APIs that reader may see, and calls them as that reader. The token lasts as long as a sign-in (`session.maxAge`). To revoke all tokens, change `RAPIDOC_SESSION_SECRET`, which also signs everyone out.

```bash
# Secrets come from your secret manager or the host's environment settings, never from the command line
npx rapidoc serve dist   # port 8080, or $PORT
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
The session secret and the client secret are environment variables of `rapidoc serve`, because the config is committed to your repository and secrets must not be:

- `RAPIDOC_SESSION_SECRET` (required): 32 characters or more, for example from `openssl rand -base64 32`. It encrypts the session cookie. Generate it once and keep it the same on every instance and across restarts; a new value signs everyone out.
- `RAPIDOC_AUTH_CLIENT_SECRET`: only for a confidential client. A public client (PKCE only) has none ([Identity providers](/docs/access/providers/)).
:::

## At your provider

Create a web application (OpenID Connect, authorization code flow) and register the redirect URL:

```text
<portal URL>/_rapidoc/auth/callback
```

Provider notes: [Identity providers](/docs/access/providers/).

## Testing locally

`rapidoc dev` doesn't enforce the rules: it shows every page, with no sign-in button and no lock icons. To test sign-in and the rules, build the portal and serve it with `rapidoc serve`:

1. **At your provider**, add `http://localhost:8080/_rapidoc/auth/callback` as a redirect URL of the portal's application. With `logout: idp`, also add `http://localhost:8080/` as a post-logout redirect.
2. **Put the secrets in a `.env` file** in the project folder, and add `.env` to `.gitignore` so it is never committed:

   ```bash title=".env"
   RAPIDOC_SESSION_SECRET=<output of: openssl rand -base64 32>
   # Only for a confidential client:
   RAPIDOC_AUTH_CLIENT_SECRET=<client secret>
   RAPIDOC_PUBLIC_URL=http://localhost:8080
   ```

   `RAPIDOC_PUBLIC_URL` is the address the provider sends readers back to. Here it overrides `site.url`, which points at your production site.
3. **Build and serve**, reading the `.env` file:

   ```bash
   npx rapidoc build
   node --env-file=.env node_modules/.bin/rapidoc serve dist
   ```

4. **Check** at `http://localhost:8080`:
   - Signed out, public pages open; a protected page sends you to your provider's sign-in, then back to that page.
   - Signed in as a reader the rules allow, the protected page opens and the header shows your name.
   - Signed in as a reader they don't allow, you get an "access denied" page.

Plain `http://localhost` works: the session cookie is marked secure only on `https` addresses.
