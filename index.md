# Developer portals from OpenAPI

> Turn OpenAPI specs and Markdown guides into a developer portal. Docs as code, static output, optional single sign-on (SSO).

## What you get

**[Interactive API reference](/docs/api-reference/reference-pages/)**

Every OpenAPI 3.0, 3.1 or Swagger 2.0 spec gets a reference page rendered by RapiDoc, with Try, code samples and a sidebar that stays in view.

**[Guides as code](/docs/writing/markdown/)**

Markdown, MDX and Markdoc pages in a `docs/` folder. Callouts, tabs, steps, cards, and live API operations inside a guide.

**[One config file](/docs/configuration/overview/)**

`rapidoc.config.yaml` describes the header, tabs, sidebars, theme and footer. An unknown key fails the build with its path and a hint.

**[Search built in](/docs/configuration/docs-search-redirects/)**

Full-text search across guides and API operations, indexed at build time. No search service to run.

**[Single sign-on (SSO)](/docs/access/overview/)**

Optional SSO with Okta, Microsoft Entra ID, Google or any OpenID Connect provider. Decide who may read which pages, APIs and files, by group or email.

**[Mock server](/docs/tools/mock-server/)**

Serve your specs with their examples, validate requests, and check OAuth tokens and scopes, so you can use Try before your API is deployed.

## From zero to portal

```bash
npm create rapidoc@latest my-portal
cd my-portal
npm install
npm run dev
```

The starter is a portal with an example API, a `rapidoc.config.yaml` and two guides, running on `http://localhost:4321`. Then point it at your own spec. Already have a repository with OpenAPI files? Run `npx @rapidoc/cli init` there instead. [Read the quickstart](/docs/getting-started/quickstart/).

## A live operation, right in a guide

Guides can embed any operation from your specs. This one comes from the [example Bookstore API](/apis/bookstore/). Its **Try** button sends a real request: run the [mock server](/docs/tools/mock-server/) to get answers before your API exists.

`GET /books` ([Bookstore API](/apis/bookstore))

> **Note: Moving from Zudoku or Redocly?**
> `npx @rapidoc/cli migrate` writes a `rapidoc.config.yaml` from your existing Zudoku or Redocly project. [See migrating](/docs/tools/migrate/).
