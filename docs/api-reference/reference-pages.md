---
title: Reference pages
description: What each API's reference page shows, its URLs, and how to link to an operation.
---

Every API in `apis:` gets an interactive reference page, rendered by the [RapiDoc](https://rapidocweb.com) web component. See one: [the example Bookstore API](/apis/bookstore/).

## URLs

| URL | Shows |
| :--- | :--- |
| `/apis/<id>` | The API's overview: title, description, servers, authentication. |
| `/apis/<id>/<tag>` | The operations of one tag. One page per tag keeps big APIs fast. |
| `/apis/<id>/<tag>#<operation>` | One operation. The anchor is the `operationId` in kebab-case: `createOrder` is `#create-order`. |
| `/apis` | Redirects to the [catalog](/docs/api-reference/catalog/). |
| `/specs/<id>.yaml` | The spec itself (or `.json`, the file's own extension). |

Tag slugs are the tag names in kebab-case: the tag `Shipment Management` is `/apis/shipments/shipment-management`.

An API with [versions](/docs/api-reference/versions/) has these URLs for its default version, and the same below `/apis/<id>/<version>` for each other version.

## The page

- **A version picker** at the top of the sidebar, when the API has [several versions](/docs/api-reference/versions/).
- **A sidebar** on the left lists the tags and their operations, with the HTTP method of each. It is static HTML, so it shows at once, before the reference has loaded, and it stays in view while the page scrolls. Its filter box narrows the list as you type.
- **The reference** shows each operation's parameters, request body, responses and examples, with schemas as an expandable tree or a table.
- **Try** sends a real request to the spec's server from the reader's browser and shows the response.
- **Authentication** lets readers enter an API key, a bearer token, or sign in with OAuth 2 before they try.

The page follows the portal: same header, footer, theme and accent color, and light or dark with the rest of the site.

## Linking to an operation

Give every operation an `operationId`. The anchor is then stable:

```markdown
[Create a shipment](/apis/shipments/shipments#create-shipment)
```

Without an `operationId`, the anchor is built from the summary and changes when the summary is edited, so `rapidoc dev` and `rapidoc build` warn about such operations. Every operation can also be linked by its method and path, which only changes with the endpoint: `#get-orders-orderid`.

## OAuth 2 sign-in for Try

RapiDoc's **Authorize** button sends readers to your OAuth provider and back to `/oauth-receiver.html`, a page every portal build has. Register `<portal URL>/oauth-receiver.html` as a redirect URL at the provider: one URL for the whole portal.

With [portal sign-in](/docs/access/overview/), readers don't need to authorize separately: see [Try with the reader's token](/docs/access/try-it/).

## Customizing

Change how RapiDoc renders, such as the schema style, the code sample languages or the font size, with the `rapidoc:` key. See [RapiDoc attributes](/docs/api-reference/rapidoc-attributes/).
