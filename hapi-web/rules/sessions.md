---
name: hapi-web-sessions
description: Cookie-based authentication with signup, login, logout, and protected routes in Hapi
metadata:
  tags: hapi, authentication, cookies, sessions, signup, login, protected-routes
---

# Hapi sessions and auth

## Dependencies

```json
"@hapi/cookie": "^12.0.1"
```

## Auth flow

| Action        | Route         | Method | Auth |
| ------------- | ------------- | ------ | ---- |
| Show login    | /             | GET    | no   |
| Show signup   | /signup       | GET    | no   |
| Process signup| /register     | POST   | no   |
| Process login | /authenticate | POST   | no   |
| Logout        | /logout       | GET    | yes  |
| Dashboard     | /dashboard    | GET    | yes  |

## Cookie plugin setup

```javascript
import Cookie from "@hapi/cookie";

await server.register(Cookie);

server.auth.strategy("session", "cookie", {
  cookie: {
    name: "sid",
    password: process.env.cookie_password, // 32+ chars
    isSecure: false, // true in production
    isSameSite: "Lax",
  },
  validateFunc: async (request, session) => {
    const user = await db.userStore.getUserById(session.id);
    return { valid: !!user, credentials: user };
  },
});

server.auth.default("session");
```

## Protecting routes

```javascript
{
  method: "GET",
  path: "/dashboard",
  options: {
    auth: { mode: "required", redirectTo: "/" },
    handler: dashboardHandler,
  },
}
```

## Signup handler

1. Validate input; check duplicate email via `db.userStore.getUserByEmail(email)`
2. Create user: `db.userStore.addUser({ firstName, lastName, email, password })`
3. Set session: `request.cookieAuth.set({ id: user._id })`
4. Redirect: `h.redirect("/dashboard")`

## Login and logout

```javascript
// Login: compare password, then request.cookieAuth.set({ id: user._id })
// Logout:
request.cookieAuth.clear();
return h.redirect("/");
```

## User store interface

Stores must implement: `addUser`, `getUserById`, `getUserByEmail`, `deleteAll`. Use uuid for `_id` in mem store.
