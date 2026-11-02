# UI Patterns Reference

## Styling in Widgets

Widgets render into a custom element without Shadow DOM, so styles affect the whole page. Two approaches:

**Inline styles** — safe and simple. Works for all static styles. Use for most things.

**Injected `<style>` tag** — needed only for things inline styles cannot express: `:hover`, `:focus`, `@keyframes`, `::before`/`::after`. Inject once on mount and prefix all class names to avoid bleeding into Happeo's page styles:

```js
useEffect(() => {
  const styleId = "my-widget-styles";
  if (!document.getElementById(styleId)) {
    const el = document.createElement("style");
    el.id = styleId;
    el.textContent = `
      .mw-tile:hover { background-color: var(--happeo-color-surface-alt, #f5f6fa); }
      .mw-input:focus { outline: none; border-color: var(--happeo-color-accent, #23cfb1); }
    `;
    document.head.appendChild(el);
  }
}, []);
```

Apply the prefix class to the root element and use it as a scope: `.mw-tile` not `.tile`.

### Brand colours and fonts

Use the `--happeo-*` brand tokens for every colour and font, always with a fallback: `var(--happeo-color-accent, #23cfb1)`. They match the widget to the page's branding. See **`references/brand-tokens.md`** for the token list, rules and examples.

### Light and dark mode

Pages widgets have light and dark modes, set automatically from the background the widget is rendered on (a dark section or a dark style option). The widget inherits the CSS `color-scheme` property. Don't paint the widget's root background; Happeo already paints it. Use the text and surface tokens, which follow `color-scheme` on their own, or `light-dark()` for colours of your own, and never assume a white background.

## Valid HTML in Lists

When rendering a list of links, `<a>` must live **inside** `<li>`, not wrap it:

```jsx
// ✓ Valid
<li><a href={item.url}>...</a></li>

// ✗ Invalid HTML
<a href={item.url}><li>...</li></a>
```

## Connecting to a Real API

Replace the stub in `server/index.js`:

```js
async function getData(user) {
  // user.organisationId scopes the query to the right tenant
  const response = await fetch(`${process.env.EXTERNAL_API_URL}/items`, {
    headers: { Authorization: `Bearer ${await getToken(user)}` },
  });
  return response.json();
}
```

The `user` object decoded from the Happeo JWT contains `{ id, email, organisationId }`.
