# Brand Tokens Reference

On Pages v2 pages, Happeo sets the page group's branding as CSS variables with the `--happeo-` prefix. Web Component widgets render without Shadow DOM, so they inherit these variables and can match the page without knowing its colours or fonts.

## Contract

- The tokens below are contract version `1`. They are never renamed or removed, and their meaning doesn't change. New tokens may be added.
- Always write `var(--happeo-x, <fallback>)`. Tokens are missing outside Pages v2 (channels, Pages v1) and in iframe widgets, and the fallback keeps the widget usable there.
- Don't use Happeo's internal variables (`--pages-v2-*`, `--accent`, `--primaryColor`, …). They can change at any time.

## Tokens

| Token | Use for |
| --- | --- |
| `--happeo-color-primary` | The organisation's primary brand colour |
| `--happeo-color-secondary` | The organisation's secondary colour |
| `--happeo-color-accent` | Calls to action and highlights |
| `--happeo-color-on-accent` | Text and icons on an accent fill |
| `--happeo-color-surface` | Inner card or panel background (follows the colour scheme) |
| `--happeo-color-surface-alt` | Subtle alternate background (follows the colour scheme) |
| `--happeo-color-text` | Body text (follows the colour scheme) |
| `--happeo-color-heading` | Heading text (follows the colour scheme) |
| `--happeo-color-link` | Links (follows the colour scheme) |
| `--happeo-font-heading` | Heading font family |
| `--happeo-font-body` | Body font family |
| `--happeo-font-h1`, `--happeo-font-h2`, `--happeo-font-h3` | `font` shorthand for headings, including mobile sizes |
| `--happeo-font-body-text` | `font` shorthand for body text |
| `--happeo-radius` | Corner radius for inner cards |

"Follows the colour scheme" means the token holds a light and a dark value, and the browser picks one from the widget's inherited `color-scheme`. A widget on a dark section, or with a dark style option, gets light text and dark surfaces without any code.

## Rules

1. Use `var(--happeo-*, <fallback>)` for every colour and font. Don't hard-code brand hex values.
2. Don't paint the widget's root background. Happeo paints the widget box to match the page. Use `--happeo-color-surface` or `--happeo-color-surface-alt` only for inner cards.
3. Put text and icons on accent fills in `--happeo-color-on-accent`.
4. Support both colour schemes: use the text and surface tokens, or `light-dark()`, and never assume a white background.
5. Don't load fonts yourself. The page already loads the style's fonts.
6. In JS, read values on mount (see below). CSS updates on its own when an admin edits the style.

## Example

```css
.mw-card {
  background: var(--happeo-color-surface, #ffffff);
  color: var(--happeo-color-text, #2b313b);
  border-radius: var(--happeo-radius, 0px);
  font: var(--happeo-font-body-text, 400 16px/1.5 sans-serif);
}

.mw-card h2 {
  color: var(--happeo-color-heading, #2b313b);
  font: var(--happeo-font-h2, 700 32px/1.5 sans-serif);
}

.mw-cta {
  background: var(--happeo-color-accent, #23cfb1);
  color: var(--happeo-color-on-accent, #2b313b);
}

.mw-tile:hover {
  background: var(--happeo-color-surface-alt, #f5f6fa);
}
```

For a colour with no token, derive it from one: `color-mix(in srgb, var(--happeo-color-accent, #23cfb1) 15%, transparent)`.

## Reading values in JS

Plain tokens resolve directly:

```js
const accent = getComputedStyle(el).getPropertyValue("--happeo-color-accent").trim();
```

Tokens that follow the colour scheme (`text`, `heading`, `link`, `surface`, `surface-alt`) return an unresolved `light-dark(…)` expression from `getPropertyValue`. Read the resolved colour through a real property instead, e.g. for a chart library that needs a colour string:

```js
el.style.color = "var(--happeo-color-text)";
const textColor = getComputedStyle(el).color; // e.g. "rgb(43, 49, 59)"
```

Read the current colour scheme with `getComputedStyle(el).colorScheme`.

## Previewing with real branding

The tokens only exist on a Happeo page. To preview a widget locally or in an artifact with an organisation's real branding, call the Happeo MCP tool `get-page-branding` when it is connected. It returns every token resolved for a page group (or the organisation's active style), with light and dark values, and a `:root { … }` CSS block to paste into the preview. Use those values only in the preview, never in the widget code.

Without the MCP tool, preview with a `:root` block of the fallback values, and switch `color-scheme` between `light` and `dark` to check both:

```css
:root {
  color-scheme: light;
  --happeo-color-accent: #23cfb1;
  --happeo-color-on-accent: #2b313b;
  --happeo-color-surface: light-dark(#ffffff, #09221f);
  --happeo-color-surface-alt: light-dark(#f5f6fa, #204440);
  --happeo-color-text: light-dark(#2b313b, #ffffff);
  --happeo-color-heading: light-dark(#2b313b, #ffffff);
  --happeo-color-link: light-dark(#0075db, #ffffff);
}
```
