# JWT Flow Reference

## Full Token Lifecycle

1. Widget calls `widgetSDK.api.init(id)` — must be first; `id` (the `uniqueId` attribute) scopes the SDK instance so multiple copies of the widget on the same page don't conflict.
2. Widget calls `api.getJWT()` — returns a short-lived JWT Happeo signs with the **Shared Secret** from Custom Apps admin.
3. Widget attaches JWT to every server request: `Authorization: Bearer <token>`.
4. Server's `happeoAuth` middleware calls `jwt.verify(token, SHARED_SECRET)`.
5. Decoded payload is attached to `res.locals.user` for route handlers.
6. The SDK automatically refreshes the JWT before expiry — no manual refresh needed.

## JWT Payload Shape

```json
{
  "user": {
    "id": "user-id",
    "email": "user@company.com",
    "firstName": "Jane",
    "lastName": "Doe"
  },
  "organisation": {
    "id": "org-id"
  },
  "iat": 1700000000,
  "exp": 1700003600
}
```

After verification, `happeoAuth` flattens this to:
```js
res.locals.user = {
  ...decoded.user,           // id, email, firstName, lastName
  organisationId: decoded.organisation?.id
}
```

## Widget SDK API Methods

All methods are available on the `api` object returned by `widgetSDK.api.init(id)`.

| Method | Purpose |
|---|---|
| `api.getJWT()` | Auth token for server requests |
| `api.getCurrentUser()` | Full current user profile |
| `api.getUser(id)` | Fetch a specific user |
| `api.searchUsers()` | Search users with pagination |
| `api.getContext()` | Widget location (channel, page, render mode) |
| `api.getSettings()` / `api.setSettings()` | Widget-level key-value config (not search-indexed) |
| `api.getContent()` / `api.setContent()` | Widget content (search-indexed) |
| `api.declareSettings(config, callback)` | Register configurable UI settings shown in edit mode |

## Configuring the Shared Secret

1. Happeo Admin → Custom Apps → your app → **Shared Secret** (generate or set).
2. Copy the value to `server/.env` as `SHARED_SECRET=<value>`.
3. The same secret is used in `jwt.verify()` — they must match exactly.

## DEV_MODE Bypass

When `DEV_MODE=true` in `server/.env`, `happeoAuth` skips verification and injects:
```js
{ id: "dev-user", email: "dev@example.com", organisationId: "dev-org" }
```

The local webpack config sets `MOCK_WIDGET_SDK=true`, which mocks `widgetSDK.api.init()` and `getJWT()` so the widget loads at `http://localhost:8080` with no Happeo connection.

**Never deploy with `DEV_MODE=true`.**

## Server-Side Validation Checklist

- `SHARED_SECRET` set in environment ✓
- `jwt.verify()` called (not `jwt.decode()`) ✓
- 401 returned on any verification failure ✓
- `DEV_MODE` warning logged at startup ✓
- `SHARED_SECRET` absence causes `process.exit(1)` in production ✓
