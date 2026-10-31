---
title: Access rules
description: Decide who may read which pages, APIs and files - by sign-in, group or email.
---

```yaml title="rapidoc.config.yaml"
access:
  default: public
  rules:
    - path: /docs/partners/**
      allow: signed-in
    - path: /apis/billing/**
      allow: { groups: [billing-team] }
    - path: /docs/internal/**
      allow: { emails: ['*@acme.com'] }
```

## Rules

Rules are checked top to bottom, and **the first rule whose path matches decides**. Put specific rules before general ones.

`path` is a URL path pattern:

- `*` matches one path segment: `/docs/*/setup` matches `/docs/orders/setup`.
- `**` matches any number of segments: `/docs/internal/**` matches everything under `/docs/internal/`.

`default` decides for everything no rule matches. It is `public` unless you set it.

## Who: `allow`

| `allow` | Who may read |
| :--- | :--- |
| `public` | Everyone, signed in or not. |
| `signed-in` | Anyone who has signed in. |
| `{ groups: [a, b] }` | Signed-in readers in any of these groups. |
| `{ emails: ['*@acme.com', 'jo@partner.io'] }` | Signed-in readers whose **verified** email matches any of these. |

`groups` and `emails` can be combined; a reader who matches either may read.

Groups come from the claim named in `auth.claims.groups` (default `groups`). See [Identity providers](/docs/access/providers/) for how each provider sends them.

## APIs and their specs

An API's reference page, tag pages, spec file and embedded operations all follow `/apis/<id>`, so one rule covers all of them:

```yaml title="rapidoc.config.yaml"
access:
  rules:
    - path: /apis/billing/**
      allow: { groups: [finance] }
```

This covers every [version](/docs/api-reference/versions/) of the API too. To protect one version only, write a rule for its URLs, such as `/apis/billing/v4-beta/**`: each version's spec follows that version's pages.

## What is protected

- Pages, specs, and files from `public/`.
- Search results: the search indexes are split per rule, so readers only find what they may read.
- The sitemap.

Locked links stay visible in menus and the catalog with a lock icon, so readers know there is more behind sign-in.

Always public: the site's scripts, styles and fonts, the RapiDoc bundle, the logo and the favicon. `/_rapidoc/` is reserved for the portal.

## What readers get

| Request | Allowed | Signed out | Signed in, not allowed |
| :--- | :--- | :--- | :--- |
| Page | 200 | Redirect to sign-in, then back | 403 "access denied" page |
| Spec or other file | 200 | 401 | 403 |

## Build checks

- A rule that matches no page or file **fails the build**. A typo in a path would otherwise leave the content public.
- A protected API whose spec is a URL is a warning: the portal can't protect a file it doesn't serve. Protect it where it is hosted.
- A public page that embeds an operation of a protected API is a warning.
