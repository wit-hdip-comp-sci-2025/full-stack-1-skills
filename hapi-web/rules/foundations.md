---
name: hapi-web-foundations
description: Hapi.js server setup with Handlebars templating, routes, layouts, and partials
metadata:
  tags: hapi, vision, handlebars, routes, server, views
---

# Hapi foundations

## Dependencies

```json
"@hapi/hapi": "^21.4.7",
"@hapi/vision": "^7.0.3",
"handlebars": "^4.7.8",
"uuid": "^13.0.0"
```

## Project structure

```
src/
├── app.js              # Server config (createServer)
├── server.js           # Entry point (start)
├── web-routes.js       # Route definitions
├── controllers/        # Request handlers
└── views/
    ├── layouts/
    ├── partials/
    └── *.hbs
```

## Server setup

```javascript
import Hapi from "@hapi/hapi";
import Vision from "@hapi/vision";
import Handlebars from "handlebars";

const server = Hapi.server({ port: 3000, host: "127.0.0.1" });
await server.register(Vision);

server.views({
  engines: { hbs: Handlebars },
  relativeTo: __dirname,
  path: "./views",
  layoutPath: "./views/layouts",
  partialsPath: "./views/partials",
  layout: true,
  isCached: false,
});

server.route(webRoutes);
await server.start();
```

## Route and controller pattern

```javascript
// web-routes.js
export const webRoutes = [
  { method: "GET", path: "/", handler: homeHandler },
  { method: "GET", path: "/about", handler: aboutHandler },
];

// controllers/about-controller.js
export const aboutController = {
  index: { handler(request, h) { return h.view("about-view", { title: "About" }); } },
};
```

## Handlebars helpers

Register for comparisons (e.g. active nav):

```javascript
Handlebars.registerHelper("eq", (a, b) => a === b);
```

Usage: `{{#if (eq currentPath "/")}}is-active{{/if}}`

## Separating configuration from startup (app.js / server.js)

Split server setup into two files so tests can obtain a fully configured server without starting an HTTP listener:

```javascript
// src/app.js — configure and return the server
import Hapi from "@hapi/hapi";
import Vision from "@hapi/vision";
import Handlebars from "handlebars";
import { webRoutes } from "./web-routes.js";
import { apiRoutes } from "./api-routes.js";
import { db } from "./models/db.js";

export async function createServer() {
  const server = Hapi.server({ port: 3000, host: "127.0.0.1" });
  await server.register(Vision);
  server.views({ engines: { hbs: Handlebars }, /* ... */ });
  db.init();
  server.route(webRoutes);
  server.route(apiRoutes);
  return server;
}
```

```javascript
// src/server.js — start only
import { createServer } from "./app.js";

const server = await createServer();
await server.start();
console.log(`Server running on ${server.info.uri}`);
```

This enables `server.inject()` in unit tests (see [testing skill](../../testing/rules/e2e-testing.md)) and `start-server-and-test` for integration tests without duplicating configuration.

## ESM notes

- Use `"type": "module"` in package.json
- `__dirname` via: `path.dirname(fileURLToPath(import.meta.url))`
