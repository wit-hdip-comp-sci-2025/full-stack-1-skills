---
name: hapi-web
description: Hapi.js web app with Handlebars views, cookie-based sessions, Joi validation, and static file serving. Use when building the web layer—server setup, templating, validation, auth, or static assets.
metadata:
  tags: hapi, nodejs, handlebars, vision, routes, sessions, cookie, inert, static, joi, validation
---

## When to use

Use this skill when you need to:

- Set up a Hapi.js server with views (Handlebars, layouts, partials)
- Define web routes and controller handlers
- Validate request payloads with Joi schemas (forms, API endpoints)
- Add signup, login, logout, and session-based auth with @hapi/cookie
- Protect routes and redirect unauthenticated users
- Serve static files (images, CSS, JS) via @hapi/inert and a public directory

## How to use

- **Server, views, routes**: [rules/foundations.md](rules/foundations.md) — Hapi + Vision, Handlebars, project structure, route/controller pattern, ESM.
- **Joi validation**: [rules/joi-schemas.md](rules/joi-schemas.md) — Schema composition, validation patterns, request/response validation, custom error messages.
- **Auth and sessions**: [rules/sessions.md](rules/sessions.md) — @hapi/cookie, signup/login/logout, protected routes, validateFunc, user store.
- **Static assets**: [rules/static-resources.md](rules/static-resources.md) — @hapi/inert, public directory, URL mapping, route order.

Refer to the rule that matches your current task; use all when building a full web app (server + validation + auth + static files).
