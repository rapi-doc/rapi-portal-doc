---
title: Theme
description: Accent color, light and dark mode, and fonts.
---

```yaml title="rapidoc.config.yaml"
theme:
  primary: '#f76b39'
  mode: auto
  fonts:
    sans: { family: Inter }
    mono: { family: JetBrains Mono }
```

## Accent color

`primary` (default `#f76b39`) colors links, buttons, the active tab and sidebar item, and the API reference.

You pick one color, and the portal keeps it readable. At build time it derives the text variants from it so that accent text keeps WCAG AA contrast (4.5:1) on both the light and the dark background. Text on accent-filled buttons is black or white, whichever reads better. So a light brand color like yellow still gives readable links.

## Mode

`mode` is the theme a visitor sees until they pick one with the theme switcher:

- `auto` (default) follows the operating system.
- `light` or `dark` forces one.

Hide the switcher with `header.themeSwitcher: false`.

## Fonts

```yaml title="rapidoc.config.yaml"
theme:
  fonts:
    sans: { family: Inter }                                  # loaded from Google Fonts, weights 400-700
    mono: { family: JetBrains Mono, url: /fonts/mono.css }   # your own stylesheet
```

A `family` alone loads from Google Fonts. Add `url` to load it from another CDN or from a CSS file in `public/`. `sans` is the text font; `mono` is used for code.

## Your own CSS

```yaml title="rapidoc.config.yaml"
theme:
  css: ./styles/portal.css   # or a list of files
```

The files load on every page after the portal's own styles, so plain selectors win without `!important`. Paths are relative to `rapidoc.config.yaml`, and `rapidoc dev` reloads when they change.

The supported way in is the portal's CSS variables, for example:

```css title="styles/portal.css"
:root {
  --portal-radius: 0.75rem;   /* corners of cards, callouts and code blocks */
}
```

Other selectors work too, but the portal's markup can change between releases.

## API reference colors

The API reference pages follow the portal theme: the same accent color, and light or dark with the rest of the page. To change more of RapiDoc's look, see [RapiDoc attributes](/docs/api-reference/rapidoc-attributes/).
