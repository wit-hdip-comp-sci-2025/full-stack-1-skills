---
name: hapi-api-swagger
description: OpenAPI (Swagger) documentation for Hapi APIs with Joi schemas and try-it-out
metadata:
  tags: hapi, swagger, openapi, api documentation, joi, validation
---

# Swagger documentation

## Dependencies

```json
"dependencies": {
  "hapi-swagger": "17.3.2"
}
```

Requires **@hapi/vision** and **@hapi/inert** (for Swagger UI). Install if not already present.

## Server registration

Register the plugin after Vision and Inert. Use `createRequire` when in ESM:

```javascript
import { createRequire } from "module";
const require = createRequire(import.meta.url);

await server.register({
  plugin: require("hapi-swagger"),
  options: {
    info: { title: "My API", version: "0.1" },
    documentationPath: "/documentation",
  },
});
```

| Option              | Purpose                    |
| ------------------- | -------------------------- |
| `info.title`        | API name in Swagger UI     |
| `info.version`      | API version string         |
| `documentationPath` | URL path for the docs page |

## Route annotations

Add these to each API route config so they appear in Swagger:

| Property           | Purpose                                |
| ------------------ | -------------------------------------- |
| `tags: ["api"]`    | Groups routes in the UI                |
| `description`      | Short summary (one line)               |
| `notes`            | Longer explanation                     |
| `validate.params`  | Joi schema for path params (e.g. `id`) |
| `validate.payload` | Joi schema for request body (POST/PUT) |
| `response.schema`  | Joi schema for response body           |

Example:

```javascript
import Joi from "joi";
const idParam = Joi.object({ id: IdSpec });

export const userApi = {
  findOne: {
    auth: false,
    tags: ["api"],
    description: "Get user by id",
    notes: "Returns a single user. 404 if not found.",
    validate: { params: idParam },
    response: { schema: UserSpecPlus },
    handler: async function (request, h) {
      // ...
    },
  },
};
```

## Joi schemas for docs

- **Request (payload)**: Use existing Joi schemas (e.g. `UserSpec`, `TrackApiSpec`).
- **Path params**: Define `IdSpec` (e.g. `Joi.alternatives().try(Joi.string(), Joi.object())`) and use in `validate.params`.
- **Response**: Define “Plus” schemas that include `_id` and optional `__v` so Swagger shows the real response shape. Use array schemas for list endpoints: `Joi.array().items(EntitySpecPlus)`.

Register routes **after** registering the Swagger plugin so the plugin can inspect them.

## URL and try-it-out

Users open `http://<host>:<port>/documentation` to view the UI. “Try it out” sends real requests to the same server; ensure CORS or same-origin so requests succeed.
