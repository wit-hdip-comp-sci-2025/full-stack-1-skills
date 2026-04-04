---
name: hapi-api-jwt
description: JWT authentication for Hapi REST API with Bearer tokens and dual auth (session + JWT)
metadata:
  tags: jwt, api, authentication, hapi, hapi-auth-jwt2, bearer
---

# JWT API authentication

JWT authentication for API routes while keeping cookie-based session auth for the web layer.

## Dependencies

```json
"hapi-auth-jwt2": "^10.0.0",
"jsonwebtoken": "^9.0.2"
```

## 1. JWT utilities

Create `src/utils/jwt-utils.js`:

- **Secret**: Read at runtime (e.g. `process.env.cookie_password || "secret"`) so tests can override.
- **createToken(user)**: `jwt.sign({ id: user._id, email: user.email }, secret, { algorithm: "HS256", expiresIn: "1h" })`.
- **decodeToken(token)**: `jwt.verify(token, secret, { algorithms: ["HS256"] })`; return `{}` on catch.
- **validate(decoded)**: For hapi-auth-jwt2. Look up user by `decoded.id` (e.g. `db.userStore.getUserById(decoded.id)`). Return `{ isValid: true, credentials: user }` or `{ isValid: false }`.

Use a **getSecret()** helper so the secret is read when the function runs, not at module load (helps tests that set `process.env.cookie_password` before creating the server).

## 2. Authenticate endpoint

- **Path**: e.g. `POST /api/users/authenticate`.
- **Auth**: `auth: false` (public).
- **Payload**: e.g. `{ email, password }`; validate with Joi (e.g. `UserCredentialsSpec`).
- **Handler**: Look up user by email; if not found or password mismatch, return `Boom.unauthorized(...)`. Otherwise `const token = createToken(user)` and `return h.response({ success: true, token }).code(201)`.

Register the authenticate route **before** other user routes so it is matched first.

## 3. Protecting API routes

- **Protected routes**: Set `auth: { strategy: "jwt" }` on each handler (find, findOne, deleteOne, deleteAll, and equivalent for playlists/tracks).
- **Public routes**: Keep `auth: false` for signup (e.g. `POST /api/users`) and for `POST /api/users/authenticate`.

Default auth can stay **session** for the app so web routes are unchanged.

## 4. Server setup (Hapi)

1. **Register plugin**: `await server.register(require("hapi-auth-jwt2"))` (use `createRequire(import.meta.url)` if in ESM).
2. **JWT strategy**:
   - `key`: same secret as in jwt-utils (e.g. `process.env.cookie_password || "secret"`).
   - `validate`: the `validate` function from jwt-utils (async, receives `decoded`, returns `{ isValid, credentials }`).
   - `verifyOptions`: `{ algorithms: ["HS256"] }`.
   - **tokenType: "Bearer"** so the plugin extracts the token from `Authorization: Bearer <token>` (required for hapi-auth-jwt2).
3. **Session strategy**: Keep existing cookie/session strategy; set `server.auth.default("session")` so web routes still use session.

## 5. Swagger (if used)

- **securityDefinitions**: `jwt: { type: "apiKey", name: "Authorization", in: "header" }`.
- **security**: `[{ jwt: [] }]` so the UI shows Authorize and lock icons for protected operations.

## 6. Client / tests

- **API client**: Store token after calling authenticate; send `Authorization: Bearer <token>` on every request (e.g. axios interceptor or per-request headers).
- **Integration tests**: In setup, create a user, call authenticate, then use the token for subsequent API calls. Clear or re-authenticate when the user is deleted (token becomes invalid once validate looks up the user and they no longer exist).
- **Handler/unit tests**: Use `server.inject(..., { headers: { Authorization: "Bearer " + createToken(user) } })`. Ensure the same secret is used (e.g. set `process.env.cookie_password = "secret"` in setup before creating the server).

## 7. Route order and dual auth

- **Authenticate** route must be registered before other user routes (e.g. before `GET /api/users`) so `POST /api/users/authenticate` is matched.
- **Web**: continues to use session (cookie); no change to login/signup pages.
- **API**: clients get a JWT from `POST /api/users/authenticate` and send it in the `Authorization` header; API routes use `auth: { strategy: "jwt" }`.

## Checklist

- [ ] Add `hapi-auth-jwt2` and `jsonwebtoken` to dependencies
- [ ] Add `src/utils/jwt-utils.js` (createToken, decodeToken, validate with runtime secret)
- [ ] Add `POST /api/users/authenticate` (auth: false, return `{ success, token }`)
- [ ] Register authenticate route before other user API routes
- [ ] Set `auth: { strategy: "jwt" }` on protected API handlers; keep `auth: false` on signup and authenticate
- [ ] Register hapi-auth-jwt2 and jwt strategy with **tokenType: "Bearer"**, key, validate, verifyOptions
- [ ] Swagger: securityDefinitions.jwt and security: [{ jwt: [] }]
- [ ] Tests: authenticate in setup where needed; send Bearer token in headers; align JWT secret in tests (e.g. `process.env.cookie_password`)
