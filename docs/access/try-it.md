---
title: Try with the reader's token
description: Let Try call your API with the signed-in reader's access token.
---

With `auth.tryIt`, a signed-in reader can press **Try** on any operation and the request carries their own access token. Your API's own permission check answers: `200` for a reader who may call it, `403` for one who may not. Readers see exactly what their account can do, with no token to copy and paste.

```yaml title="rapidoc.config.yaml"
auth:
  issuer: https://acme.us.auth0.com/
  audience: https://api.acme.com
  tryIt: true
```

## How it works

1. At sign-in, `rapidoc serve` keeps the reader's access token in its own encrypted, `HttpOnly` cookie. The ID and refresh tokens are not kept at all.
2. Reference pages and embedded operations ask the server for the token and fill it into the Authentication section of the spec's first `oauth2` or `openIdConnect` scheme, or else its first HTTP `bearer` scheme.
3. **Try** sends `Authorization: Bearer <token>` to the API.

The page keeps the token in memory only, never in local storage. It is handed only to pages of the portal itself (same origin), but every script running on those pages can fetch it, including analytics or other scripts you add with [`head`](/docs/configuration/site/#head-tags). Add only scripts you trust to a portal with `tryIt`.

The token expires on the provider's schedule, often after an hour, which can be well before the sign-in ends (`session.maxAge`). From then on, **Try** sends no token until the reader signs in again. A shorter `session.maxAge` keeps the two closer.

## The audience

The access token has to be for your API, not for the portal:

- **Auth0**: set `audience` to the API identifier.
- **Okta**: use the authorization server of your API as the `issuer`, and leave `audience` out.
- **Microsoft Entra ID**: request your API's scopes in `scopes`, and leave `audience` out.

## Try it with the mock server

Before the API exists, point the spec's server at [`rapidoc mock`](/docs/tools/mock-server/) and start it with your issuer and audience. The mock checks each token's signature, audience and scopes the way your API would, so you can show who may call what.
