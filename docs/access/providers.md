---
title: Identity providers
description: SSO setup notes for Okta, Microsoft Entra ID (Azure AD), Auth0, Keycloak, Google and others, and SAML-only providers.
---

Single sign-on works with any OpenID Connect provider: Okta, Microsoft Entra ID (Azure AD), Google Workspace, Keycloak, Authentik, Zitadel, Auth0, Ping and AWS Cognito. For a full walkthrough with one provider, see [SSO with Microsoft Entra ID](/docs/access/entra-id/).

Everywhere, create a web application with the authorization code flow and register:

- the redirect URL `<portal URL>/_rapidoc/auth/callback`,
- with `logout: idp`, the portal URL as a post-logout redirect.

A public client (PKCE only) needs no client secret. For a confidential client, set `RAPIDOC_AUTH_CLIENT_SECRET` when you run `rapidoc serve`.

## Groups

[Access rules](/docs/access/access-rules/) by group need the groups in the ID token. Providers send them differently:

| Provider | Setting |
| :--- | :--- |
| Okta | Add the `groups` scope: `scopes: [openid, profile, email, groups]`, and a groups claim on the authorization server. |
| Microsoft Entra ID | Prefer app roles: `claims: { groups: roles }`. The `groups` claim holds IDs, not names, and is dropped for users in many groups. |
| Keycloak | Realm roles: `claims: { groups: realm_access.roles }`. |
| Auth0 | Add the roles in an Action under a namespaced claim, e.g. `claims: { groups: https://acme/roles }`. |

When rules use groups but a sign-in carries none, `rapidoc serve` logs a hint.

## SAML-only providers

Put an OpenID Connect broker in front: Keycloak, Authentik, Zitadel or Ory Polis.

## One build, several providers

`RAPIDOC_AUTH_ISSUER`, `RAPIDOC_AUTH_CLIENT_ID` and `RAPIDOC_AUTH_AUDIENCE` override the config, so the same build can run against a staging and a production tenant.
