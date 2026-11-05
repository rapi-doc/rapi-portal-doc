---
title: Mock server
description: Serve your specs with examples and generated data, validate requests, and check real OAuth scopes.
---

`rapidoc mock` serves every local spec of the portal on one port, each under `/<api id>` (a version other than the default under `/<api id>/<version>`). Readers can press **Try** before the API exists, and you can demo who may call what with real tokens.

```bash
npm install -D @rapidoc/mock
npx rapidoc mock
```

```text
http://localhost:4010/bookstore   → specs/bookstore.yaml
```

List the mock as a server in the spec, and the reference pages' **Try** calls it:

```yaml title="specs/bookstore.yaml"
servers:
  - url: http://localhost:4010/bookstore
    description: Local mock
```

This site's [example API](/apis/bookstore/) does exactly that: run `npm run mock` in its repository and **Try** answers. **Run workflow** on a [workflow](/docs/configuration/workflows/#run-against-the-mock) diagram uses the same server, so whole workflows run with no backend.

## What it answers

- **Examples first**: the response's example from the spec, else data generated from the schema (patterns, formats, enums, ranges). Generated data is the same on every run for a given `--seed`.
- **Validated requests**: parameters and JSON, form and multipart bodies are checked against the spec. A bad request gets `400` with the list of problems.
- **CORS** for any origin, so a portal on another port can call it.

## Choosing a response

| Request header | Effect |
| :--- | :--- |
| `Prefer: code=404` | The documented `404` response (else `4XX`, else `default`). |
| `Prefer: example=empty` | The media type's example named `empty`. |
| `Prefer: dynamic=true` | Generated data even when there is an example. |

Applied preferences come back in `Preference-Applied`. `GET /` lists the APIs served.

## Real token checks

Each operation's `security` is enforced:

| Scheme | Check |
| :--- | :--- |
| OAuth 2, OpenID Connect, HTTP bearer | The token must be a JWT signed by your issuer, unexpired and for your audience, else `401`. It must grant the operation's scopes in its `scope`, `scp` or `permissions` claim, else `403` with `WWW-Authenticate: Bearer error="insufficient_scope"`. |
| API key | The header, query parameter or cookie must be present. |
| HTTP basic | Credentials must be present. |
| `security: []` or none | Open. |

```bash
npx rapidoc mock --issuer https://acme.us.auth0.com/ --audience https://api.acme.com
```

Without `--issuer`, any bearer token passes and scopes aren't checked. Combined with [Try with the reader's token](/docs/access/try-it/), signed-in readers see `200` or `403` depending on their own permissions.

## Options

| Option | Variable | Meaning |
| :--- | :--- | :--- |
| `--issuer <url>` | `RAPIDOC_MOCK_ISSUER` | OpenID Connect issuer whose access tokens are accepted. |
| `--audience <id>` | `RAPIDOC_MOCK_AUDIENCE` | `aud` the tokens must have. |
| `--port <number>` | | Default `4010`. |
| `--host <host>` | | Default `0.0.0.0`. |
| `--seed <number>` | | Seed for generated data. Default `1`. |

## Without a portal

The mock also works on plain spec files:

```bash
npx rapidoc mock ./specs                      # every OpenAPI file in the folder
npx rapidoc mock orders.yaml billing.json     # these files
```

Each spec is served under its file name: `orders-api.yaml` is `/orders-api`. The mapping is printed at start. This needs only `@rapidoc/cli` and `@rapidoc/mock`, not the portal.
