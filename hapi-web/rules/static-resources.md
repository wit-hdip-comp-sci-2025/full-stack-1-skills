---
name: hapi-web-static-resources
description: Serve static files (images, CSS, JS) in a Hapi app using @hapi/inert
metadata:
  tags: hapi, inert, static files, assets, public, images, css, js
---

# Static resources with Inert

## Dependencies

```json
"@hapi/inert": "^7.1.0"
```

## Project structure

```
public/
├── images/
├── css/
└── js/

src/
├── app.js    # Register Inert, add static route
└── views/
```

## Server setup

1. **Register Inert** (before routes that use it):

```javascript
import Inert from "@hapi/inert";

await server.register(Inert);
```

2. **Add static route last** so web and API routes take precedence. Use a catch-all path and the `directory` handler:

```javascript
import path from "path";
import { fileURLToPath } from "url";

const __dirname = path.dirname(fileURLToPath(import.meta.url));
const publicPath = path.join(__dirname, "..", "public");

server.route({
  method: "GET",
  path: "/{param*}",
  handler: {
    directory: {
      path: publicPath,
      listing: false, // no directory listing (security)
      index: false,   // no default index.html for directories
    },
  },
  options: { auth: false },
});
```

3. **Route order**: Register `webRoutes` and `apiRoutes` before this static route. The first matching route wins; more specific paths (e.g. `/dashboard`, `/api/users`) are defined earlier, so `/{param*}` only handles requests that did not match.

## URL mapping

Files under `public/` are served at the root:

| File on disk             | URL                |
| ------------------------ | ------------------ |
| `public/images/logo.png` | `/images/logo.png` |
| `public/css/style.css`   | `/css/style.css`   |
| `public/js/app.js`       | `/js/app.js`       |

## Using assets in views

Reference by URL path (no `public` in the path):

```html
<img src="/images/logo.png" alt="Logo" />
<link rel="stylesheet" href="/css/style.css" />
<script src="/js/app.js"></script>
```

## Handler options

| Option           | Purpose                                     |
| ---------------- | ------------------------------------------- |
| `path`           | Absolute path to the directory to serve     |
| `listing: false` | Disable directory listing (recommended)     |
| `index: false`   | Do not serve index files for directory URLs |
