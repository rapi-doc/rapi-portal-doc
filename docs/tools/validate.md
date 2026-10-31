---
title: Validate
description: Check your OpenAPI specs against the schema, locally and in CI.
---

```bash
npx rapidoc validate                       # every local spec in apis:
npx rapidoc validate ./specs/orders.yaml   # one file
```

`rapidoc validate` checks specs against the OpenAPI 3.0 or 3.1 schema. Relative `$ref`s to other files resolve. It also warns about operations without an `operationId`, whose links change when their summary is edited.

It exits with code `1` when a spec has errors, so it can gate a pull request:

```yaml title=".github/workflows/docs.yml"
# in a job's steps:
- run: npm ci
- run: npx rapidoc validate
- run: npx rapidoc build
```

Projects created with `npm create rapidoc` have it as `npm run validate`.

Validating one file needs only `@rapidoc/cli`. Validating a whole portal also reads the config, so it needs `@rapidoc/portal`.
