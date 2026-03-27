# happeo-widget-skill

A [Claude Code](https://claude.ai/code) skill for building Happeo custom widgets with JWT-authenticated API calls.

## What it does

When installed, this skill gives Claude Code the knowledge to scaffold and guide development of Happeo custom widgets — Web Components that fetch live data from authenticated external APIs and render it inside Happeo pages.

Covers:
- Happeo Widget SDK setup (`widgetSDK.api.init`, `getJWT`)
- JWT verification on the server side (`happeoAuth` middleware)
- Local development with `DEV_MODE` (no live Happeo connection needed)
- Building and uploading `bundle.js` to Happeo Admin

## Installation

```bash
claude skills install git@github.com:happeo/happeo-widget-skill.git
```

## Usage

Once installed, trigger the skill by asking Claude Code things like:

- *"Create a Happeo widget that shows data from ServiceNow"*
- *"Build a Happeo custom widget with JWT auth"*
- *"Set up a Happeo widget that calls an external API"*

## Reference implementation

A complete working example lives at **[happeo/happeo-widget-with-auth](https://github.com/happeo/happeo-widget-with-auth)** — an IT Service Catalogue widget with a local Express server, JWT verification, and a searchable tile grid UI.

## Resources

- [Happeo Custom Apps documentation](https://developers.happeo.com/docs/custom-apps)
- [Advanced usage — JWT authentication](https://developers.happeo.com/docs/advanced-usage)
