---
name: persistence-fundamentals
description: In-memory and JSON file persistence with lowdb, Joi, and env config
metadata:
  tags: persistence, lowdb, joi, dotenv, store-abstraction, mem-store, json-store
---

# Persistence fundamentals

## Dependencies

```json
"dotenv": "^17.3.1",
"joi": "^17.11.0",
"lowdb": "^7.0.1"
```

## Store abstraction (db.js)

```javascript
export const db = {
  userStore: null,
  collectionStore: null,
  itemStore: null,

  init(storeType) {
    switch (storeType) {
      case "json":
        this.userStore = userJsonStore;
        this.collectionStore = collectionJsonStore;
        this.itemStore = itemJsonStore;
        break;
      default:
        this.userStore = userMemStore;
        this.collectionStore = collectionMemStore;
        this.itemStore = itemMemStore;
    }
  },
};
```

Call `db.init(process.env.store || "mem")` at startup.

## Mem store pattern

Simple in-memory array; use uuid for `_id`. Implement: `addUser`, `getUserById`, `getUserByEmail`, `getAllUsers`, `deleteUserById`, `deleteAll`.

## JSON store pattern (lowdb)

```javascript
import { Low } from "lowdb";
import { JSONFile } from "lowdb/node";

const adapter = new JSONFile(join(process.cwd(), "src/models/json/users.json"));
const db = new Low(adapter, { users: [] });
await db.read();

export const userJsonStore = {
  async addUser(user) {
    user._id = uuid();
    db.data.users.push(user);
    await db.write();
    return user;
  },
  async getUserById(id) { return db.data.users.find(u => u._id === id) ?? null; },
  // ... getUserByEmail, getAllUsers, deleteUserById, deleteAll
};
```

## Joi validation

```javascript
const userSchema = Joi.object({
  firstName: Joi.string().required(),
  lastName: Joi.string().required(),
  email: Joi.string().email().required(),
  password: Joi.string().min(6).required(),
});

const { error, value } = userSchema.validate(payload, { abortEarly: false });
if (error) return h.view("signup-view", { errors: error.details, user: payload });
```

## Environment

```javascript
import "dotenv/config";
```

`.env`: `cookie_password=32_char_minimum_secret`, `store=mem|json`. Exclude `.env` from git.

## Data models

**User**: _id, firstName, lastName, email, password  
**Collection**: _id, title, userid  
**Item**: _id, title, artist, duration?, collectionid  

## Cascade delete

When deleting a collection, delete its items. When deleting a user, delete their collections and items.
