# Full-Stack Foundations

Use this README to understand the skill portfolio and choose which domain skill applies to your task.

## Available Skills

| Skill | Domain | Use when |
|-------|--------|----------|
| [**hapi-web**](hapi-web/SKILL.md) | Server, routes, Handlebars, Joi, sessions, static files | Setting up Hapi, views, validation, auth, static assets. See `rules/` for foundations, joi-schemas, sessions, static-resources. |
| [**persistence**](persistence/SKILL.md) | Mem, JSON, Mongo, env, seeding | Store abstraction, Mongoose, seeding. See `rules/` for fundamentals, mongo, seed. |
| [**testing**](testing/SKILL.md) | Mocha, Playwright, nodemon | Unit tests, E2E specs, fixtures, dev auto-reload. See `rules/` for e2e-testing, autoreload. |
| [**styling**](styling/SKILL.md) | Front-end styling | Layouts, forms, navbar; options include [bulma-css](styling/rules/bulma-css.md). |
| [**hapi-api**](hapi-api/SKILL.md) | REST API, Boom, Joi, Swagger, JWT | API endpoints, docs, JWT auth. See `rules/` for api-development, swagger, jwt. |
| [**code-quality**](code-quality/SKILL.md) | ESLint, Prettier, Airbnb | Code linting, formatting, style consistency. See `rules/` for eslint, prettier. |


## Usage

Skills are project-scoped (`.cursor/skills/`). Cursor applies them when prompts match the skill descriptions. Reference explicitly: "Using hapi-web, add login." or "Using hapi-api (jwt), secure the API."

## Structure

Each skill has:
- `SKILL.md`: entrypoint with metadata, when-to-use, and instructions
- Optional `rules/*.md` for detailed content (e.g. **hapi-web** has `rules/foundations.md`, `rules/joi-schemas.md`, `rules/sessions.md`, `rules/static-resources.md`; **hapi-api** has `rules/api-development.md`, `rules/swagger.md`, `rules/jwt.md`; **persistence** has `rules/fundamentals.md`, `rules/mongo.md`, `rules/seed.md`; **testing** has `rules/e2e-testing.md`, `rules/autoreload.md`; **styling** has `rules/bulma-css.md`; **code-quality** has `rules/eslint.md`, `rules/prettier.md`)

## Prerequisites

~1 year programming: JavaScript/Node.js, basic HTTP, databases, web concepts.

## Glossary

New to the terminology? See **[GLOSSARY.md](GLOSSARY.md)** for definitions of:
- Acronyms (TDD, E2E, JWT, CORS, ESM, ODM, etc.)
- Technical terms (Bearer tokens, Schema composition, Fixtures, etc.)
- Framework-specific terms (Vision, Inert, Boom, Toolkit, etc.)
- Generic domain terms (Collection, Item, User)
