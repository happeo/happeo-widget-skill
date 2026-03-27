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
      .mw-tile:hover { background-color: #f0f6ff; }
      .mw-input:focus { outline: none; border-color: #1d4ed8; }
    `;
    document.head.appendChild(el);
  }
}, []);
```

Apply the prefix class to the root element and use it as a scope: `.mw-tile` not `.tile`.

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
