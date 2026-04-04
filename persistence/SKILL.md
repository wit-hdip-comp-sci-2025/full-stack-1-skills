---
name: persistence
description: Store abstraction (mem, JSON, Mongo), env config, MongoDB seeding, and Cloudinary image storage. Use when building persistence layers, adding Mongo/Mongoose, seeding sample data, or implementing cloud-based image uploads.
metadata:
  tags: persistence, lowdb, dotenv, mongodb, mongoose, seeding, store-abstraction, cloudinary, image-upload
---

## When to use

Use this skill when you need to:

- Implement in-memory or JSON file stores and a pluggable store abstraction (db.init)
- Configure environment variables with dotenv and implement cascade delete for mem/JSON
- Integrate MongoDB with Mongoose and extend the store abstraction
- Seed MongoDB with development/test data using reference syntax (`->collection.key`) and production guards
- Upload, store, and manage images using Cloudinary cloud storage

## How to use

- **Fundamentals**: [rules/fundamentals.md](rules/fundamentals.md) — Mem/store abstraction, lowdb JSON stores, dotenv, data models, cascade delete.
- **Mongo**: [rules/mongo.md](rules/mongo.md) — Mongoose schemas, connection, mongo store pattern, ObjectId handling, db extension.
- **Seed**: [rules/seed.md](rules/seed.md) — Seed data structure, seeder implementation, app wiring, tests, E2E/scripts.
- **Cloudinary**: [rules/cloudinary.md](rules/cloudinary.md) — Cloud image storage, upload/delete/retrieve operations, multipart routes, ImageStore utility.

Refer to the rule that matches your task; use all three core rules for a full Mongo-backed app with seeded data. Add Cloudinary for cloud-based image management.
