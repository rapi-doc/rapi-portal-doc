---
title: Validate
description: Check your OpenAPI specs against the specification, locally and in CI.
---

```bash
npx rapidoc validate                       # every local spec in apis:, and the workflow files
npx rapidoc validate ./specs/orders.yaml   # one file
npx rapidoc validate https://example.com/openapi.json
```

`rapidoc validate` checks specs strictly against the OpenAPI Specification (2.0, 3.0, 3.1 and 3.2):

- **Required fields and types**, as the specification's schema for that version defines them: `info.version`, a response's `description`, `type` values, and so on.
- **Path parameters**: every `{name}` in a path must be defined as a `path` parameter, on the path or on each operation (a `$ref` to a parameter counts).
- **`$ref`s**: each must point to something. Files reached through a relative `$ref` are checked too.

Each error names the file, line and column, and the JSON pointer of what's wrong; an error inside a file reached through `$ref` points into that file:

```text
🔍 orders: specs/orders.yaml
   ❌ 2 error(s):
      specs/orders.yaml:3:1  must have required property 'version'  (#/info)
      specs/shared/responses.yaml:12:3  must have required property 'description'  (#/NotFound)
```

With `@rapidoc/portal` installed it also warns about operations without an `operationId` (their links change when their summary is edited), and checks that the steps of each workflow file resolve against your APIs.

## Options

| Option | Meaning |
| :--- | :--- |
| `--format pretty \| json \| sarif` | `pretty` (default) for the terminal; `json` for scripts; `sarif` for GitHub code scanning. `json` and `sarif` print one document on stdout and nothing else. |
| `--strict` | Fail on warnings too. |
| `--remote` | Also fetch and check remote specs in `apis:` and remote `$ref`s. Without it they are skipped (with a warning for `$ref`s). A spec given as a URL is always fetched. |

## In CI

It exits with code `1` when a spec has errors (or warnings, with `--strict`), so it can gate a pull request:

```yaml title=".github/workflows/docs.yml"
# in a job's steps:
- run: npm ci
- run: npx rapidoc validate
- run: npx rapidoc build
```

To see the problems on the pull request itself, upload a SARIF report to GitHub code scanning:

```yaml title=".github/workflows/docs.yml"
permissions:
  security-events: write
# in a job's steps:
- run: npx rapidoc validate --format sarif > rapidoc.sarif || true
- uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: rapidoc.sarif
```

Projects created with `npm create rapidoc` have it as `npm run validate`.

Validating one file needs only `@rapidoc/cli`. Validating a whole portal also reads the config, so it needs `@rapidoc/portal`.
