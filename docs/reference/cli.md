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
| `validate [spec \| dir]` | Validates one spec (a file or URL), or every spec and workflow file in a portal ([Validate](/docs/tools/validate/)). |
| `migrate --from <src>` | Writes `rapidoc.config.yaml` from a `zudoku` or `redocly` project. |
| `help [command]` | Shows help, or one command's options and examples (same as `rapidoc <command> --help`). |
| `--version`, `-v` | Shows the version. |

Options and arguments can come in any order (`rapidoc build --out-dir public ./portal`). A misspelled command or option is an error with a suggestion (`Unknown option --outdir. Did you mean --out-dir?`).

## Options

| Option | Commands | Meaning |
| :--- | :--- | :--- |
| `--out-dir <path>` | build | Output folder. Default `dist`. |
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
| `--format <format>` | validate | `pretty` (default), `json` or `sarif`. |
| `--strict` | validate | Fail on warnings too. |
| `--remote` | validate | Also check remote specs and remote `$ref`s. |
| `--help`, `-h` | all | Show the command's help instead of running it. |
| `--quiet`, `-q` | all | Print errors and warnings only. |

## Output and exit codes

On a terminal the CLI prints icons; when its output is piped or `NO_COLOR` is set, it prints plain text, with `error:` and `warning:` in front of problems. Errors and warnings go to stderr, everything else to stdout.

| Exit code | Means |
| :--- | :--- |
| `0` | Success. |
| `1` | A problem in the input: an invalid spec or config, a failed build, a missing package. |
| `2` | A wrong command line: an unknown command or option, a missing or malformed value. |

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
