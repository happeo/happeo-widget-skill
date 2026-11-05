---
name: happeo-widget
description: This skill should be used when the user asks to "create a Happeo widget", "build a Happeo custom widget", "add a widget to Happeo", "create a widget that calls an API", "set up JWT auth for a Happeo widget", "build a widget with server authentication", "create a Happeo custom app", or wants to integrate external data (ITSM, ServiceNow, Freshservice, Zendesk, etc.) into Happeo pages as a widget.
version: 0.3.0
---

# Happeo Custom Widget with JWT Auth

Build Happeo custom widgets (Web Components) that call authenticated server-side APIs. Keep widgets simple — only add complexity when the use case requires it. The canonical example: https://github.com/happeo/happeo-widget-with-auth

## Architecture

```
Happeo Page
  └── Widget (bundle.js uploaded to Happeo)
        │  widgetSDK.api.init(id)
        │  api.getJWT()  →  JWT signed with SHARED_SECRET
        ▼
      Server (Express, localhost:3001 or deployed)
        │  happeoAuth middleware verifies JWT
        │  decodes { user, organisationId } from token
        ▼
      External API / mock data
```

The JWT is the critical link: Happeo signs it with a **Shared Secret** configured in Happeo Admin → Custom Apps. The server verifies it with the same secret, confirming requests come from authenticated Happeo users.

## Minimal File Structure

```
/src
  index.js       Web component registration + slug
  Widget.js      SDK init, getJWT(), fetch, render

/server
  index.js       Express app + routes
  middleware/
    happeoAuth.js  JWT verification
  .env.example   SHARED_SECRET, DEV_MODE, PORT

webpack.config.js         Production build → dist/bundle.js
webpack.config.local.js   Local dev (mock SDK, port 8080)
```

Add more files only when the widget genuinely needs them.

## Key Constants to Configure

| File | What to change |
|---|---|
| `src/index.js` | `slug` — from Happeo Admin → Custom Widgets |
| `server/.env` | `SHARED_SECRET` — from Happeo Admin → Custom Apps |
| Build command | `API_URL=https://your-server.com npm run build` |

## Step-by-Step: Creating a New Widget

### 1. Happeo Admin Setup

1. Go to **Happeo Admin → Custom Apps** → create a new app → copy the **Shared Secret**.
2. Within the app, create a **Custom Widget** (Web Component type) → copy the **slug**.

### 2. Widget Front End (`src/`)

**`src/index.js`** — register the web component:
```js
import React from "react";
import { createRoot } from "react-dom/client";
import Widget from "./Widget";

class MyHappeoWidget extends HTMLElement {
  connectedCallback() {
    const uniqueId = this.getAttribute("uniqueId") || "";
    const mode = this.getAttribute("mode") || "";
    const location = this.getAttribute("location") || "";
    this._root = createRoot(this);
    this._root.render(<Widget id={uniqueId} editMode={mode === "edit"} location={location} />);
  }
  disconnectedCallback() { this._root?.unmount(); }
}

const slug = "your-widget-slug"; // ← from Happeo Admin
window.customElements.get(slug) || window.customElements.define(slug, MyHappeoWidget);
```

Happeo passes three attributes: `uniqueId`, `mode` (`"edit"` or `""`), and `location` (page/channel context).

**`src/Widget.js`** — the core auth + data fetch pattern:
```js
import React, { useState, useEffect } from "react";
import widgetSDK from "@happeo/widget-sdk";

const API_URL = process.env.API_URL || "http://localhost:3001";

const Widget = ({ id, editMode }) => {
  const [items, setItems] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const init = async () => {
      try {
        const api = await widgetSDK.api.init(id);
        const jwt = await api.getJWT();
        const response = await fetch(`${API_URL}/api/data`, {
          headers: { Authorization: `Bearer ${jwt}` },
        });
        if (!response.ok) throw new Error(`Server error: ${response.status}`);
        const { items } = await response.json();
        setItems(items);
      } catch (err) {
        setError(err.message);
        if (!editMode) api?.reportError(err); // report to Happeo outside edit mode
      } finally {
        setLoading(false);
      }
    };
    init();
  }, [id]);

  if (loading) return <p>Loading…</p>;
  if (error) return <p>Error: {error}</p>;
  return <ul>{items.map(item => <li key={item.id}>{item.name}</li>)}</ul>;
};

export default Widget;
```

This is intentionally minimal. Add state, components, and styling only as the use case requires.

### 3. Server (`server/`)

**`server/index.js`**:
```js
require("dotenv").config();
const express = require("express");
const cors = require("cors");
const { happeoAuth } = require("./middleware/happeoAuth");

const app = express();
const DEV_MODE = process.env.DEV_MODE === "true";

if (!DEV_MODE && !process.env.SHARED_SECRET) {
  console.error("FATAL: SHARED_SECRET is not set");
  process.exit(1);
}

app.use(express.json());
app.use(cors());

app.get("/health", (_req, res) => res.json({ status: "ok" }));

app.get("/api/data", happeoAuth, (req, res) => {
  const { user } = res.locals; // { id, email, organisationId }
  res.json({ items: getData(user) });
});

function getData(_user) {
  return []; // ← replace with real API call
}

app.listen(process.env.PORT || 3001);
```

**`server/middleware/happeoAuth.js`**:
```js
const jwt = require("jsonwebtoken");
const DEV_MODE = process.env.DEV_MODE === "true";

const happeoAuth = (req, res, next) => {
  if (DEV_MODE) {
    res.locals.user = { id: "dev-user", email: "dev@example.com", organisationId: "dev-org" };
    return next();
  }
  const token = (req.headers.authorization || "").replace("Bearer ", "");
  if (!token) return res.status(401).json({ error: "Missing token" });
  try {
    const decoded = jwt.verify(token, process.env.SHARED_SECRET);
    res.locals.user = { ...decoded.user, organisationId: decoded.organisation?.id };
    next();
  } catch {
    res.status(401).json({ error: "Invalid or expired token" });
  }
};

module.exports = { happeoAuth };
```

**`server/.env.example`**:
```
SHARED_SECRET=replace-with-your-shared-secret
DEV_MODE=true
PORT=3001
```

### 4. Webpack — inject API_URL

```js
// webpack.config.js (production)
new webpack.DefinePlugin({
  "process.env.API_URL": JSON.stringify(process.env.API_URL || "http://localhost:3001"),
  "process.env.MOCK_WIDGET_SDK": JSON.stringify(false),
})

// webpack.config.local.js (local dev)
new webpack.EnvironmentPlugin({ MOCK_WIDGET_SDK: true, API_URL: "http://localhost:3001" })
```

### 5. Local Development

```bash
# Terminal 1 — server
cd server && cp .env.example .env
npm install && npm start        # http://localhost:3001

# Terminal 2 — widget
npm install && npm run dev      # http://localhost:8080
```

### 6. Build & Deploy

```bash
API_URL=https://your-server.example.com npm run build
# Upload dist/bundle.js to Happeo Admin → Custom Widgets
```

## Matching the Page's Branding

On Pages v2 pages the widget inherits the page group's branding as `--happeo-*` CSS variables. Style the widget with them so it matches every page it's placed on:

```css
.mw-card {
  background: var(--happeo-color-surface, #ffffff);
  color: var(--happeo-color-text, #2b313b);
  border-radius: var(--happeo-radius, 0px);
  font: var(--happeo-font-body-text, 400 16px/1.5 sans-serif);
}
.mw-cta {
  background: var(--happeo-color-accent, #23cfb1);
  color: var(--happeo-color-on-accent, #2b313b);
}
```

- Always write a fallback. The tokens are missing outside Pages v2 and in iframe widgets.
- Don't hard-code brand hex values, don't paint the widget's root background, and don't load fonts.
- Text and surface tokens switch between light and dark on their own. Never assume a white background.
- To preview with an organisation's real branding, call the Happeo MCP tool `get-page-branding` if it's connected and paste its `:root` CSS block into the preview only.

Full token list, rules and the JS pattern for resolved colours: **`references/brand-tokens.md`**.

## Testing JWT Auth Locally

```js
// server/generate-test-token.js
require("dotenv").config();
const jwt = require("jsonwebtoken");
console.log(jwt.sign(
  { user: { id: "test", email: "test@example.com" }, organisation: { id: "test-org" } },
  process.env.SHARED_SECRET,
  { expiresIn: "1h" }
));
```

```bash
DEV_MODE=false node generate-test-token.js
curl -H "Authorization: Bearer <token>" http://localhost:3001/api/data
```

## Common Pitfalls

- **`slug` mismatch** — must exactly match the value in Happeo Admin.
- **`SHARED_SECRET` mismatch** — server and Happeo Admin must share the same value.
- **Hard-coded colours** — a widget with fixed hex colours clashes with other page styles and turns unreadable on dark sections. Use the `--happeo-*` tokens.
- **CSS hover/focus states** — inline styles can't express `:hover` or `:focus`. Inject a scoped `<style>` tag for those; prefix all class names to avoid bleeding into Happeo's page styles.

## Additional Resources

- **`references/jwt-flow.md`** — JWT payload shape, full SDK method list, DEV_MODE details
- **`references/brand-tokens.md`** — `--happeo-*` brand tokens, styling rules, light and dark mode, previews
- **`references/ui-patterns.md`** — scoped styles, valid list HTML, connecting a real API
- **Reference implementation** — https://github.com/happeo/happeo-widget-with-auth
- **Happeo Custom Apps docs** — https://developers.happeo.com/docs/custom-apps
- **Advanced usage / JWT** — https://developers.happeo.com/docs/advanced-usage
