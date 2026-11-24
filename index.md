# RapiDoc Portal documentation

> How to build a developer portal from OpenAPI specs and Markdown guides with RapiDoc Portal - configuration, guides, API reference pages, single sign-on (SSO), tools and deployment.

RapiDoc Portal turns your OpenAPI files and Markdown or Markdoc guides into a static developer portal, with search, an API catalog, interactive references and optional single sign-on. This site is built with it. What it offers at a glance is on [RapiDoc Portal's page](https://rapidocui.com/portal/).

```bash
npm create rapidoc@latest my-portal
cd my-portal
npm install
npm run dev
```

The starter is a portal with an example API, a `rapidoc.config.yaml` and two guides, on `http://localhost:4321`. Already have a repository with OpenAPI files? Run `npx @rapidoc/cli init` there instead. [Read the quickstart](/docs/getting-started/quickstart/).

> **Note: RapiDoc Portal 1.0**
> In sync with RapiDoc 15, end to end: the same names, one sidebar and safer defaults across the family. From here on, a change that breaks your portal comes only in a new major version. [See what's new in 1.0](/docs/getting-started/whats-new/).

## Find your way

**[Get started](/docs/getting-started/introduction/)**

What a portal is made of, the quickstart, and the files of a project.

**[Configuration](/docs/configuration/overview/)**

`rapidoc.config.yaml`: the site, theme, header, navigation, APIs, workflows, footer and search.

**[Writing guides](/docs/writing/markdown/)**

Markdown, Markdoc and MDX pages, callouts, live API operations and your own pages.

**[API reference](/docs/api-reference/reference-pages/)**

Reference pages, API versions, the catalog, and RapiDoc's attributes.

**[Single sign-on (SSO)](/docs/access/overview/)**

Sign-in with Okta, Microsoft Entra ID, Google or any OpenID Connect provider, and who may read what.

**[Tools and deploy](/docs/tools/mock-server/)**

A mock for every API, MCP for agents, spec checks, migrating, and hosting the portal.

## A live operation, right in a guide

Guides can embed any operation from your specs. This one comes from the [example Bookstore API](/apis/bookstore/). Its **Try** button sends a real request: run the [mock server](/docs/tools/mock-server/) to get answers before your API exists.

`GET /books` ([Bookstore API](/apis/bookstore))

> **Note: Moving from Zudoku or Redocly?**
> `npx @rapidoc/cli migrate` writes a `rapidoc.config.yaml` from your existing Zudoku or Redocly project. [See migrating](/docs/tools/migrate/).
