---
title: CLI
description: Every rapidoc command and option.
sidebar:
  order: 1
---

```bash
rapidoc <command> [options]
```

| Command | Does |
| :--- | :--- |
| `init [dir]` | Scaffolds a portal in `dir` (default: the current folder), using the OpenAPI files already there. |
| `dev [dir]` | Starts the dev server with live reload on `http://localhost:4321`. |
| `build [dir]` | Builds the static portal into `dist/`. |
| `serve [dist]` | Serves a built portal, with sign-in and access rules. Default folder `dist`. |
| `mock [dir \| spec...]` | Mock server for a portal's APIs, a folder of specs, or spec files, each under `/<api id>`. |
| `validate [spec \| dir]` | Validates one spec, or every spec in a portal's `apis:`. |
| `migrate --from <src>` | Writes `rapidoc.config.yaml` from a `zudoku` or `redocly` project. |
| `--help`, `-h` | Shows help. |
| `--version`, `-v` | Shows the version. |

## Options

| Option | Commands | Meaning |
| :--- | :--- | :--- |
| `--outDir <path>` | build | Output folder. Default `dist`. |
| `--port <number>` | dev, serve, mock | dev: default `4321`; serve: `$PORT` or `8080`; mock: `4010`. |
| `--host <host>` | serve, mock | Address to listen on. Default `$HOST` (serve), else `0.0.0.0`. |
| `--mount-path <path>` | serve | Path requests arrive under, when a proxy strips part of the portal's path. Default `$RAPIDOC_MOUNT_PATH`, else the path of `site.url`. |
| `--issuer <url>` | mock | OpenID Connect issuer whose tokens are accepted. Default `$RAPIDOC_MOCK_ISSUER`. |
| `--audience <id>` | mock | Audience tokens must be issued for. Default `$RAPIDOC_MOCK_AUDIENCE`. |
| `--seed <number>` | mock | Seed for generated data. Default `1`. |
| `--debug` | dev, build | Keep the temporary build workspace and print its path. |
| `--title <title>` | init | Site title. Asked for in a terminal when omitted. |
| `--force`, `-f` | init | Overwrite existing files. |
| `--yes`, `-y` | init | Accept the defaults without asking. |
| `--no-install` | init | Skip `npm install`. |

## Which packages each command needs

The CLI loads the other packages only when a command needs them:

| Command | Needs |
| :--- | :--- |
| `init`, `validate <spec>` | `@rapidoc/cli` only |
| `dev`, `build`, `migrate`, `validate` (portal) | `@rapidoc/portal` |
| `serve` | `@rapidoc/server` |
| `mock` | `@rapidoc/mock` (and `@rapidoc/portal` for a portal folder) |

## Examples

```bash
rapidoc init my-portal --title "Acme Developer Portal"
rapidoc dev
rapidoc build ./my-api-portal
rapidoc serve dist --port 8080
rapidoc mock --issuer https://example.us.auth0.com/ --audience https://api.example.com
rapidoc mock ./specs --port 4011
rapidoc validate ./specs/orders-api.yaml
rapidoc migrate --from zudoku
```
