---
title: SSO with Microsoft Entra ID
description: Step by step - put the portal behind Microsoft Entra ID (Azure AD) single sign-on, and give groups of employees access to their APIs with app roles.
---

This guide puts a portal behind your company's Microsoft Entra ID (formerly Azure AD) sign-in: everyone in the company can read the portal, and only the billing team can read the billing API. Other providers work the same way; their differences are in [Identity providers](/docs/access/providers/).

You need a portal that builds (`npx rapidoc build`), the URL it will run at (here `https://developers.acme.com`), and the right to register applications in your Entra tenant.

## 1. Register the portal in Entra ID

In the [Microsoft Entra admin center](https://entra.microsoft.com), open **Identity → Applications → App registrations** and choose **New registration**:

- **Name**: `Developer portal`.
- **Supported account types**: *Accounts in this organizational directory only*.
- **Redirect URI**: platform **Web**, URL `https://developers.acme.com/_rapidoc/auth/callback`.

For local testing, add `http://localhost:8080/_rapidoc/auth/callback` as a second redirect URI under **Authentication**.

From the app's **Overview**, note:

- **Application (client) ID**, for `auth.clientId`.
- **Directory (tenant) ID**, for the issuer: `https://login.microsoftonline.com/<tenant ID>/v2.0`.

Under **Certificates & secrets**, create a **client secret** and copy its value. It is shown only once.

## 2. Create app roles for your groups

Entra ID's `groups` claim holds group IDs, not names, and is left out for users in many groups. App roles avoid both problems.

Under **App roles**, choose **Create app role**:

- **Display name**: `Billing team`
- **Allowed member types**: *Users/Groups*
- **Value**: `billing-team`. This is the name your access rules use.

Then open **Identity → Applications → Enterprise applications → Developer portal → Users and groups**, choose **Add user/group**, and assign the billing team's Entra group (or single users) to the **Billing team** role.

To let only assigned users sign in at all, set **Properties → Assignment required?** to *Yes* on the same enterprise application.

## 3. Configure the portal

```yaml title="rapidoc.config.yaml"
site:
  url: https://developers.acme.com

auth:
  issuer: https://login.microsoftonline.com/<tenant ID>/v2.0
  clientId: <application (client) ID>
  scopes: [openid, profile, email]
  claims:
    groups: roles # app roles arrive in the roles claim

access:
  default: signed-in # every employee who signs in
  rules:
    - path: /apis/billing/**
      allow: { groups: [billing-team] }
    - path: /docs/billing/**
      allow: { groups: [billing-team] }
```

The issuer and client ID are not secrets, so they can live in the committed config. To use one build against a test and a production tenant, set `RAPIDOC_AUTH_ISSUER` and `RAPIDOC_AUTH_CLIENT_ID` instead.

## 4. Build and run

```bash
npx rapidoc build
RAPIDOC_SESSION_SECRET="$(openssl rand -base64 32)" \
RAPIDOC_AUTH_CLIENT_SECRET="<client secret value>" \
RAPIDOC_PUBLIC_URL=https://developers.acme.com \
  npx rapidoc serve dist --port 8080
```

Keep `RAPIDOC_SESSION_SECRET` the same on every instance and across restarts, or everyone is signed out. Store both secrets in your secret manager, not in the repository.

Open the portal: you are sent to the Microsoft sign-in page, then back to the page you asked for. Someone without the **Billing team** role sees an access-denied page under `/apis/billing/`.

To run it in a container or behind a load balancer, see [Deploy with rapidoc serve](/docs/deploy/rapidoc-serve/).

## Troubleshooting

| Problem | Fix |
| :--- | :--- |
| `AADSTS50011`: the redirect URI does not match | The redirect URI in the app registration must equal `<RAPIDOC_PUBLIC_URL>/_rapidoc/auth/callback`, including `https` and any path the portal is served under. |
| `AADSTS7000215`: invalid client secret | Use the secret's **Value**, not its **Secret ID**, and check that it hasn't expired. |
| Signed in, but group rules deny access | Check that the user or their group is assigned to the app role, and that `claims.groups` is `roles`. `rapidoc serve` logs a hint when rules use groups and a sign-in carries none. Role changes apply at the next sign-in. |
| Email rules never match | Entra ID sends `email` only for users with an email address in their profile. Prefer app roles for access. |
