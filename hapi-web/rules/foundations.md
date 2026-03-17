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
├── server.js           # Hapi init, Vision, views config
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

## ESM notes

- Use `"type": "module"` in package.json
- `__dirname` via: `path.dirname(fileURLToPath(import.meta.url))`
