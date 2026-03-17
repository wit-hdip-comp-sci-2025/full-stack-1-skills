---
name: persistence
description: Store abstraction (mem, JSON, Mongo), Joi validation, env config, and MongoDB seeding. Use when building persistence layers, adding Mongo/Mongoose, or seeding sample data.
metadata:
  tags: persistence, lowdb, joi, dotenv, mongodb, mongoose, seeding, store-abstraction
---

## When to use

Use this skill when you need to:

- Implement in-memory or JSON file stores and a pluggable store abstraction (db.init)
- Add Joi validation, dotenv, and cascade delete for mem/JSON
- Integrate MongoDB with Mongoose and extend the store abstraction
- Seed MongoDB with development/test data using reference syntax (`->collection.key`) and production guards

## How to use

- **Fundamentals**: [rules/fundamentals.md](rules/fundamentals.md) — Mem/store abstraction, lowdb JSON stores, Joi validation, dotenv, data models, cascade delete.
- **Mongo**: [rules/mongo.md](rules/mongo.md) — Mongoose schemas, connection, mongo store pattern, ObjectId handling, db extension.
- **Seed**: [rules/seed.md](rules/seed.md) — Seed data structure, seeder implementation, app wiring, tests, E2E/scripts.

Refer to the rule that matches your task; use all three for a full Mongo-backed app with seeded data.
