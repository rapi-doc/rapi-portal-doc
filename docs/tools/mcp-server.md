---
title: MCP server for agents
description: Every portal build carries its APIs and workflows as MCP tools; rapidoc serve hosts them at /mcp for AI agents, with the reader's own access.
---

AI agents can use your APIs the way readers do. `rapidoc build` turns every local spec and every workflow into [MCP](https://modelcontextprotocol.io) tool manifests, and `rapidoc serve` hosts them at `/mcp`: one endpoint for the whole portal, with the same access rules as the pages.

```text
dist/mcp/<api id>.json              the API's operations as tools, named <api id>_<operation>
dist/mcp/workflows/<id>.json        each workflow of the file as one tool, its inputs as the arguments
```

Nothing to configure: a build has the manifests whenever it has specs on disk (a spec that is a URL is left out).

## Connect an agent

With [`rapidoc serve`](/docs/deploy/rapidoc-serve/), the portal's MCP endpoint is `https://<your portal>/mcp` (Streamable HTTP). A client that speaks stdio only, such as Claude Desktop, connects through [mcp-remote](https://www.npmjs.com/package/mcp-remote):

```json title="claude_desktop_config.json"
{
  "mcpServers": {
    "acme-portal": {
      "command": "npx",
      "args": ["mcp-remote", "https://developers.acme.com/mcp"]
    }
  }
}
```

Without a token, the agent sees the public APIs and calls them without credentials.

## Agents that sign in

On a portal with [sign-in](/docs/access/overview/), a signed-in reader opens the user menu and picks **Connect an AI agent**. **Get token** mints a token for their agents and shows the command that uses it:

```bash
npx mcp-remote https://developers.acme.com/mcp --header "Authorization: Bearer <token>"
```

With the token, the agent:

- sees the tools of every API the reader may see, and the workflows whose APIs they may all see;
- with [`auth.tryIt`](/docs/access/try-it/), calls the APIs with the reader's own access token, so the API's permission check applies to the agent as it does to **Try**.

The token is the reader's session (sealed like the cookies, never a copy of them) and lasts as long as the session. It opens exactly what the reader can see; a reader can't give an agent more than they have.

## Credentials of other kinds

The reader's access token fills `oauth2`, `openIdConnect` and HTTP `bearer` schemes. For an API that takes an API key or HTTP Basic, set the credential in the environment of `rapidoc serve`, named like the exported project's: the scheme id in upper snake case (`PARTNER_KEY`), `<SCHEME>_USERNAME` and `<SCHEME>_PASSWORD` for Basic, or `API_KEY` for any scheme. Every agent then calls that API with the same key.

## From the command line

For an API that isn't in a portal, or to try its tools before building one, the CLI turns any spec into an MCP server:

```bash
npx rapidoc mcp export openapi.yaml --out-dir orders-mcp   # writes the server project
npx rapidoc mcp serve openapi.yaml                         # runs it over stdio, no files
```

`mcp serve` is what an agent's config can run directly (it needs `@modelcontextprotocol/sdk` installed next to the CLI):

```json
{
  "mcpServers": {
    "orders": { "command": "npx", "args": ["rapidoc", "mcp", "serve", "./specs/orders.yaml"] }
  }
}
```

Credentials come from the environment, one variable per security scheme as in the exported project (`BEARER_AUTH` for a scheme `BearerAuth`). `--base-url http://localhost:4010/orders` sends the calls to [`rapidoc mock`](/docs/tools/mock-server/) instead of the spec's server, and `--arazzo flows.arazzo.yaml` adds the workflows as tools.

## Exported servers

Readers can also download a standalone MCP server for one API from its reference page (**Export MCP server**, with `rapidoc.mcp: true`): a small stdio server with the same manifest and runtime, for agents that run where the portal can't be reached.
