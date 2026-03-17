---
name: hapi-web
description: Hapi.js web app with Handlebars views, cookie-based sessions, and static file serving. Use when building the web layer—server setup, templating, auth, or static assets.
metadata:
  tags: hapi, nodejs, handlebars, vision, routes, sessions, cookie, inert, static
---

## When to use

Use this skill when you need to:

- Set up a Hapi.js server with views (Handlebars, layouts, partials)
- Define web routes and controller handlers
- Add signup, login, logout, and session-based auth with @hapi/cookie
- Protect routes and redirect unauthenticated users
- Serve static files (images, CSS, JS) via @hapi/inert and a public directory

## How to use

- **Server, views, routes**: [rules/foundations.md](rules/foundations.md) — Hapi + Vision, Handlebars, project structure, route/controller pattern, ESM.
- **Auth and sessions**: [rules/sessions.md](rules/sessions.md) — @hapi/cookie, signup/login/logout, protected routes, validateFunc, user store.
- **Static assets**: [rules/static-resources.md](rules/static-resources.md) — @hapi/inert, public directory, URL mapping, route order.

Refer to the rule that matches your current task; use all three when building a full web app (server + auth + static files).
