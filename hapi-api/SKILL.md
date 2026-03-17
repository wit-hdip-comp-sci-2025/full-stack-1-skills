---
name: hapi-api
description: Hapi REST API with endpoints, Boom/Joi, OpenAPI (Swagger) docs, and optional JWT auth. Use when building API routes, documenting the API, or securing it with Bearer tokens.
metadata:
  tags: api, rest, hapi, boom, joi, swagger, openapi, jwt, axios
---

## When to use

Use this skill when you need to:

- Add REST API endpoints with correct status codes and error handling (@hapi/boom, Joi)
- Implement CRUD handlers, API client patterns, and integration/E2E tests (axios, Playwright)
- Expose interactive OpenAPI (Swagger) documentation and try-it-out
- Secure API routes with JWT (authenticate endpoint, Bearer tokens) while keeping session auth for the web

## How to use

- **API development**: [rules/api-development.md](rules/api-development.md) — Routes, handler pattern, Boom/Joi, status codes, API client, Mocha/Playwright tests.
- **Swagger documentation**: [rules/swagger.md](rules/swagger.md) — hapi-swagger registration, route annotations, Joi schemas for docs.
- **JWT API auth**: [rules/jwt.md](rules/jwt.md) — JWT utilities, authenticate endpoint, protecting routes, dual auth (session + JWT), Swagger security, tests.

Refer to the rule that matches your task; use all three for a full documented, JWT-secured API.
