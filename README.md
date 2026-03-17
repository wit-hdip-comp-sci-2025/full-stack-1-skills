# Full-Stack Foundations

Use this README to understand the skill portfolio and choose which domain skill applies to your task.

## Available Skills

| Skill | Domain | Use when |
|-------|--------|----------|
| [**hapi-web**](hapi-web/SKILL.md) | Server, routes, Handlebars, sessions, static files | Setting up Hapi, views, auth, static assets. See `rules/` for foundations, sessions, static-resources. |
| [**persistence**](persistence/SKILL.md) | Mem, JSON, Mongo, Joi, env, seeding | Store abstraction, Mongoose, seeding. See `rules/` for fundamentals, mongo, seed. |
| [**testing**](testing/SKILL.md) | Mocha, Playwright, nodemon | Unit tests, E2E specs, fixtures, dev auto-reload. See `rules/` for e2e-testing, autoreload. |
| [**ux**](ux/SKILL.md) | Front-end styling | Layouts, forms, navbar; options include [bulma-css](ux/rules/bulma-css.md). |
| [**hapi-api**](hapi-api/SKILL.md) | REST API, Boom, Joi, Swagger, JWT | API endpoints, docs, JWT auth. See `rules/` for api-development, swagger, jwt. |


## Usage

Skills are project-scoped (`.cursor/skills/`). Cursor applies them when prompts match the skill descriptions. Reference explicitly: "Using hapi-web, add login." or "Using hapi-api (jwt), secure the API."

## Structure

Each skill has:
- `SKILL.md`: entrypoint with metadata, when-to-use, and instructions
- Optional `rules/*.md` for detailed content (e.g. **hapi-web** has `rules/foundations.md`, `rules/sessions.md`, `rules/static-resources.md`; **hapi-api** has `rules/api-development.md`, `rules/swagger.md`, `rules/jwt.md`; **persistence** has `rules/fundamentals.md`, `rules/mongo.md`, `rules/seed.md`; **testing** has `rules/e2e-testing.md`, `rules/autoreload.md`; **ux** has `rules/bulma-css.md`)

## Prerequisites

~1 year programming: JavaScript/Node.js, basic HTTP, databases, web concepts.
