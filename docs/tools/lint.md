---
title: Lint
description: Check what your specs need to make a good portal, beyond being valid OpenAPI.
---

```bash
npx rapidoc lint                          # every API in rapidoc.config.*
npx rapidoc lint ./specs/orders.yaml      # one file (or a URL)
npx rapidoc lint --strict --format sarif > rapidoc.sarif
```

A valid spec can still make a poor portal: operations without an `operationId` get page URLs that change when their summary is edited, bodies without examples leave **Try** and the mock server with nothing to show, tags that aren't declared have no description for their sidebar group. `rapidoc lint` finds those. It runs [`validate`](/docs/tools/validate/)'s check first, so one CI step covers both; an invalid spec is not linted further.

Each finding names the file, line and column, and the rule:

```text
🔍 orders: specs/orders.yaml
   ❌ 1 error(s):
      specs/orders.yaml:37:7  operationId "createOrder" is also used by POST /orders  [operation-operationId-unique]  (#/paths/~1orders~1{id}/delete/operationId)
   ⚠️ specs/orders.yaml:11:5  GET /orders has no summary  [operation-summary]  (#/paths/~1orders/get)
   ⚠️ specs/orders.yaml:22:11  application/json request body has no example  [media-example]  (#/paths/~1orders/post/requestBody/content/application~1json)
```

## Rules

Each rule has a severity: `error` fails the run, `warn` is reported (and fails with `--strict`), `off` is skipped. The defaults lean on `warn`, so the first run on an existing spec doesn't fail CI. `rapidoc lint --rules` prints this list.

| Rule | Checks | Why it matters | Default |
| :--- | :--- | :--- | :--- |
| `operation-operationId` | Every operation has an `operationId`. | Page URLs are built from it (otherwise from the summary, which changes); so are MCP tool names. | warn |
| `operation-operationId-unique` | No two operations share one. | They would share a page URL and a tool name. | error |
| `operation-operationId-url-safe` | Letters, digits, `_`, `-` and `.` only. | It appears in URLs. | warn |
| `operation-summary` | Every operation has a summary. | The sidebar label and the MCP tool's title. | warn |
| `operation-description` | Every operation has a description. | Readers, and the MCP tool's description. | off |
| `operation-tag-defined` | Tags used are in the top-level `tags` list. | The sidebar group's description and order come from there. | warn |
| `tag-description` | Every top-level tag has a description. | It introduces the group on the API page. | off |
| `operation-2xx-response` | A 2xx or 3xx response is documented. | Try has nothing to show as the expected result. | warn |
| `operation-4xx-response` | A 4xx response is documented. | Readers see the error shapes. | off |
| `security-defined` | POST, PUT, PATCH and DELETE have a security requirement, their own or the document's. `security: []` opts out. | Try and the mock server need it; MCP marks them as consequential. | warn |
| `security-scheme-defined` | Every scheme a requirement names is declared. | Try's authentication panel is built from the declarations. | error |
| `no-unused-components` | Every component is referenced. | Documentation of nothing. | warn |
| `media-example` | Request and response bodies have an example, on the media type or its schema (or on every property). | Try fills the request with it; the mock server answers with it. | warn |
| `path-kebab-case` | Path segments other than parameters are kebab-case. | Naming. | off |
| `no-server-example-com` | Servers aren't example.com or localhost. | Try would call nowhere. | warn |
| `info-contact` | `info.contact` is set. | Shown in the API page header. | off |
| `info-license` | `info.license` is set. | Shown in the API page header. | off |

## Settings

Most specific wins: `--rule` on the command line, then the API's entry under `lint.apis`, then `lint.rules`, then the defaults above.

```yaml title="rapidoc.config.yaml"
lint:
  rules:
    operation-description: warn
    no-unused-components: off
  apis:
    legacy: # an API that can't meet a rule yet
      operation-operationId: off
```

```bash
npx rapidoc lint --rule operation-description=warn --rule path-kebab-case=error
```

The `lint` key is read straight from the file, so linting one spec needs no other package. A misspelled rule name is an error with a suggestion.

## Options

| Option | Meaning |
| :--- | :--- |
| `--rule <name>=<error \| warn \| off>` | Set a rule's severity; repeat for more. |
| `--rules` | List the rules, their defaults and what they check. |
| `--format pretty \| json \| sarif` | `pretty` (default) for the terminal; `json` for scripts; `sarif` for GitHub code scanning, with every rule described. |
| `--strict` | Fail on warnings too. |
| `--remote` | Also fetch remote specs in `apis:` and remote `$ref`s. |

## In CI

Exit code `1` on errors (or warnings with `--strict`), `2` for a wrong rule name or severity:

```yaml title=".github/workflows/docs.yml"
# in a job's steps:
- run: npx rapidoc lint --strict
```

Upload a SARIF report to see the findings on the pull request, as the [Validate](/docs/tools/validate/#in-ci) page shows.

## Which packages it needs

One spec, a URL, or a folder of OpenAPI files: `@rapidoc/cli` alone. A portal folder (with `rapidoc.config.*`): `@rapidoc/portal`, to read the `apis:` list.
