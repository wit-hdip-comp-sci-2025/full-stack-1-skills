---
name: persistence-mongo-seed
description: Seed MongoDB with development/test data and arrow-reference syntax
metadata:
  tags: mongodb, mongoose, seeding, seed data, development, reference syntax
---

# MongoDB seeding

Covers seed-data structure, seeder implementation, and app.js wiring.

## 1. Seed data structure

Create a module (e.g. `src/models/seed-data.js`) that exports a single object keyed by **collection name**:

- Each **collection** has **`_model`** (e.g. model name string) for documentation/tests.
- Each **item** (document) in a collection is keyed by a **stable key** so other items can reference it. Skip the `_model` key when iterating.
- **Reference syntax**: any field whose value is a string starting with `->` is a reference. Format: `"->collection.key"` (e.g. `"->users.bart"`). Resolved to the created item's `_id` after that item is inserted.

Insert **order matters**: process collections in dependency order—a collection before any collection whose items reference it (e.g. parent collection first, then collections that reference it).

```javascript
// Example: three collections; items in later collections reference items in earlier ones
export const seedData = {
  users: {
    _model: "User",
    homer: { firstName: "Homer", email: "homer@example.com", password: "secret" },
    bart: { firstName: "Bart", email: "bart@example.com", password: "secret" },
  },
  items: {
    _model: "Item",
    mozart: { title: "Mozart Favourites", userid: "->users.bart" },
  },
  entries: {
    _model: "Entry",
    entry_1: { title: "Concerto No. 1", artist: "Mozart", duration: 15, itemId: "->items.mozart" },
  },
};
```

## 2. Seeder implementation

Create a **seed** module (e.g. `src/models/seed.js`) that:

- **Production guard**: If `process.env.NODE_ENV === "production"`, log a warning and return without seeding (return empty refMap).
- **Store guard**: Only run when the app is using Mongo (e.g. `db._storeType === "mongo"`). Otherwise return empty.
- **Options**: Support `{ dropCollections: true }` (default). When true, clear collections in **reverse dependency order** (dependents first, then collections they reference) before inserting.
- **Reference resolution**: Maintain a **refMap** keyed by collection name, then by item key → created `_id` (e.g. `{ users: { homer: id, bart: id }, items: { mozart: id }, entries: { entry_1: id } }`). For each item, replace any `"->collection.key"` value with `refMap[collection][key]` before insert. Invalid refs throw.
- **Insert order**: Loop each collection in dependency order; for each collection, skip `_model`, then for each item resolve refs and insert via the existing store API for that collection. After each insert, set `refMap[collection][itemKey] = created._id`.
- **Return value**: Return the refMap so tests can assert created IDs.

Helper pattern: `isRef(value)` → `typeof value === "string" && value.startsWith("->")`. `resolveRef(value, refMap)` → parse `"->collection.key"`, return `refMap[collection][key]` or throw. `resolveDocument(obj, refMap)` → copy object with all refs resolved.

## 3. Wiring into the app

- After **db.init(storeType)** and, when `storeType === "mongo"`, after **db.connect()**, call **await seed({ dropCollections: true })**.
- Keep seeding in the same process as the server (e.g. in `createServer()` in app.js), not in a separate script, so one startup = one seed when using Mongo.

## 4. Tests

- **Seed data structure**: Unit tests that assert `_model` per collection, required fields on sample items, and that reference fields use the `"->collection.key"` form and point at valid keys.
- **Seed function**:
  - With mem (or non-mongo): assert `seed()` returns without inserting (e.g. empty refMap).
  - With mongo: assert seed clears when `dropCollections: true`, inserts expected counts per collection, resolves refs (each item's reference fields point at existing collection keys), and returns created IDs. Use `test.beforeEach` / `test.skip` or `store !== "mongo"` to skip mongo-only tests when not running against Mongo.
- **API / E2E**: Tests that expect seeded data should run only when the server was started with Mongo (so seed ran). Use env (e.g. `store=mongo`) for the server and skip those tests when `process.env.store !== "mongo"` (e.g. Playwright `testInfo.skip()` in a `beforeEach`).

## 5. E2E and scripts

- **Playwright**: If the config passes `store` into the webServer env, use `process.env.store || "mem"` so that when you run with `store=mongo`, the started server uses Mongo and seeds. Seeded E2E tests can then assert on seeded items.
- **Scripts**: e.g. `test:api:mongo` and `test:e2e:mongo` that set `store=mongo` (and `db=mongodb://...`) so the server seeds; run API or E2E tests that rely on seeded data.

## Checklist

- [ ] Seed data module: `_model` per collection, named keys per item, `"->collection.key"` for refs; collections in dependency order.
- [ ] Seed module: production and mongo-only guards; dropCollections option; ref resolution; insert per collection via existing stores; return refMap of created IDs.
- [ ] App: call `await seed({ dropCollections: true })` after mongo connect when using Mongo.
- [ ] Tests: structure tests; seed function tests (mongo tests skipped when store !== mongo); API/E2E for seeded data run only with mongo.
- [ ] E2E: webServer env respects `process.env.store`; seeded specs skip when `store !== "mongo"`.
