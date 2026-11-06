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
| `preview <spec>` | Shows one spec (a file or URL) in `<rapi-doc>` on `http://localhost:4000`, reloading when a file in its folder changes. No portal needed. |
| `mock [dir \| spec...]` | Mock server for a portal's APIs, a folder of specs, or spec files, each under `/<api id>`. |
| `validate [spec \| dir]` | Validates one spec (a file or URL), or every spec and workflow file in a portal ([Validate](/docs/tools/validate/)). |
| `mcp export <spec>` | Writes an MCP server project for the spec ([MCP server](/docs/tools/mcp-server/#from-the-command-line)). |
| `mcp serve <spec>` | Runs that MCP server over stdio, without writing files. |
| `migrate [--from <src>]` | Writes `rapidoc.config.yaml` from a `zudoku` or `redocly` project (found in the folder when `--from` is left out). |
| `doctor [dir]` | Checks Node, the packages' versions, the portal config and the browser bundle; `--format json` for bug reports. |
| `help [command]` | Shows help, or one command's options and examples (same as `rapidoc <command> --help`). |
| `--version`, `-v` | Shows the version. |

Options and arguments can come in any order (`rapidoc build --out-dir public ./portal`). A misspelled command or option is an error with a suggestion (`Unknown option --outdir. Did you mean --out-dir?`).

## Options

| Option | Commands | Meaning |
| :--- | :--- | :--- |
| `--out-dir <path>` | build | Output folder. Default `dist`. |
| `--port <number>` | dev, serve, mock, preview | dev: default `4321`; serve: `$PORT` or `8080`; mock: `4010`; preview: `4000`. |
| `--host <host>` | serve, mock, preview | Address to listen on. Default `$HOST` (serve), `localhost` (preview), else `0.0.0.0`. |
| `--open` | dev, preview | Open the page in the browser. |
| `--theme <name>` | preview | `<rapi-doc>` theme. |
| `--mount-path <path>` | serve | Path requests arrive under, when a proxy strips part of the portal's path. Default `$RAPIDOC_MOUNT_PATH`, else the path of `site.url`. |
| `--issuer <url>` | mock | OpenID Connect issuer whose tokens are accepted. Default `$RAPIDOC_MOCK_ISSUER`. |
| `--audience <id>` | mock | Audience tokens must be issued for. Default `$RAPIDOC_MOCK_AUDIENCE`. |
| `--seed <number>` | mock | Seed for generated data. Default `1`. |
| `--debug` | dev, build | Keep the temporary build workspace and print its path. |
| `--title <title>` | init | Site title. |
| `--spec <file>` | init | An OpenAPI file to include; repeat for more. Default: every one found in the folder. |
| `--starter-spec` | init | Use the starter spec instead of the OpenAPI files found. |
| `--force`, `-f` | init | Overwrite existing files (in a terminal, lists them and asks first). |
| `--no-install` | init | Skip `npm install`. |
| `--format <format>` | validate, doctor | `pretty` (default), `json`, or `sarif` (validate). |
| `--strict` | validate | Fail on warnings too. |
| `--remote` | validate, mcp | Also fetch remote specs and remote `$ref`s. |
| `--out-dir <path>` | mcp export | Where to write the project. Default `mcp-server`. |
| `--force`, `-f` | mcp export | Write into a folder that isn't empty. |
| `--arazzo <file>` | mcp | An Arazzo file whose workflows become tools too. |
| `--server-name <name>` | mcp | The MCP server's name. Default: from the API title. |
| `--base-url <url>` | mcp serve | Call the API here instead of the spec's server (for example `rapidoc mock`). |
| `--help`, `-h` | all | Show the command's help instead of running it. |
| `--quiet`, `-q` | all | Print errors and warnings only. |
| `--yes`, `-y` | all | Don't ask anything: take the default answer to every question. |

## Questions

In a terminal, commands ask for what their options don't say, and `rapidoc` alone shows a menu of the common commands. When the output is piped, `CI` is set, or `--yes` is given, they never ask: they use the defaults, or stop and name the option to pass. Every question has an option, so any run can be scripted.

| Command | Asks | Option that answers it |
| :--- | :--- | :--- |
| `init` | The site title, which of the OpenAPI files found to include, whether to run `npm install`. Then prints the command that does the same without questions. | `--title`, `--spec` / `--starter-spec`, `--no-install` |
| `init --force` | Whether to replace the existing files it lists. | `--yes` |
| `migrate` | Which project to migrate, when `--from` is left out (it looks for `redocly.yaml` and `zudoku.config.*`); whether to replace an existing `rapidoc.config.yaml`. | `--from`, `--yes` |
| `dev`, `serve`, `mock` | Whether to use the next free port when the port is taken. Without a terminal, `serve` and `mock` stop (`dev` moves to a free port by itself). | `--port`, `--yes` |

`Ctrl+C` on a question cancels the command (exit code `130`).

## Output and exit codes

On a terminal the CLI prints icons; when its output is piped or `NO_COLOR` is set, it prints plain text, with `error:` and `warning:` in front of problems. Errors and warnings go to stderr, everything else to stdout.

| Exit code | Means |
| :--- | :--- |
| `0` | Success. |
| `1` | A problem in the input: an invalid spec or config, a failed build, a missing package. |
| `2` | A wrong command line: an unknown command or option, a missing or malformed value. |
| `130` | Cancelled at a question (`Ctrl+C`). |

## Which packages each command needs

The CLI loads the other packages only when a command needs them:

| Command | Needs |
| :--- | :--- |
| `init`, `validate <spec>` | `@rapidoc/cli` only |
| `dev`, `build`, `migrate`, `validate` (portal) | `@rapidoc/portal` |
| `serve` | `@rapidoc/server` |
| `mock` | `@rapidoc/mock` (and `@rapidoc/portal` for a portal folder) |
| `preview`, `mcp export`, `doctor` | `@rapidoc/cli` only |
| `mcp serve` | `@modelcontextprotocol/sdk` |

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
