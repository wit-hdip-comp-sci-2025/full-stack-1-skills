---
name: persistence-fundamentals
description: In-memory and JSON file persistence with lowdb and env config
metadata:
  tags: persistence, lowdb, dotenv, store-abstraction, mem-store, json-store
---

# Persistence fundamentals

## Dependencies

```json
"dotenv": "^17.3.1",
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
  async getUserById(id) { return db.data.users.find(u => u._id === id) ?? undefined; },
  // ... getUserByEmail, getAllUsers, deleteUserById, deleteAll
};
```

## Environment configuration

### File structure

```
.env                 # Local development (gitignored - NEVER commit)
.env.example         # Template with placeholder values (committed)
.env.test            # Test environment (optional)
.gitignore           # Must include .env
```

### .env.example (template)

Commit this to git as documentation:

```env
# Server
NODE_ENV=development
PORT=3000

# Database
db=mongodb://localhost:27017/myapp
store=mongo

# Session Security
cookie_name=myapp-session
cookie_password=CHANGE_ME_32_CHARS_MINIMUM_SECRET

# JWT (if using JWT auth)
jwt_secret=CHANGE_ME_ANOTHER_SECRET_KEY
jwt_expiry=1h

# External Services (examples)
# SMTP_HOST=smtp.example.com
# SMTP_USER=your_email
# API_KEY=your_api_key
```

### .env (actual secrets - NEVER commit)

```env
NODE_ENV=development
PORT=3000
db=mongodb://localhost:27017/myapp-dev
store=mongo
cookie_password=a8d7f6e5c4b3a2d1f9e8d7c6b5a4f3e2
jwt_secret=9f8e7d6c5b4a3f2e1d9c8b7a6f5e4d3c
```

### .gitignore

Always exclude environment files:

```
.env
.env.local
.env.*.local
node_modules/
```

### Loading configuration

```javascript
// Load dotenv at app entry point
import "dotenv/config";
```

Or create a centralized config module:

```javascript
// src/config.js
import "dotenv/config";

export const config = {
  env: process.env.NODE_ENV || "development",
  port: parseInt(process.env.PORT, 10) || 3000,

  db: process.env.db || "mongodb://localhost:27017/myapp",
  store: process.env.store || "mem",

  cookie: {
    name: process.env.cookie_name || "session",
    password: process.env.cookie_password,
    isSecure: process.env.NODE_ENV === "production",
  },

  jwt: {
    secret: process.env.jwt_secret,
    expiry: process.env.jwt_expiry || "1h",
  },
};

// Validate required secrets
if (!config.cookie.password || config.cookie.password.length < 32) {
  throw new Error("cookie_password must be at least 32 characters");
}

if (!config.jwt.secret) {
  throw new Error("jwt_secret is required");
}
```

Usage:

```javascript
// src/app.js
import { config } from "./config.js";

const server = Hapi.server({
  port: config.port,
  routes: { cors: true },
});
```

### Environment-specific settings

| Setting | Development | Test | Production |
|---------|-------------|------|------------|
| `NODE_ENV` | `development` | `test` | `production` |
| `store` | `mem` or `json` | `mem` | `mongo` |
| `cookie.isSecure` | `false` | `false` | `true` (HTTPS only) |
| `db` | Local MongoDB | Test DB | Production DB URL |
| `LOG_LEVEL` | `debug` | `silent` | `info` or `warn` |
| `PORT` | `3000` | `3001` | Set by host |

### Security checklist

**Never**:
- ❌ Commit `.env` to git (use `.env.example` instead)
- ❌ Hardcode secrets in source code
- ❌ Use same secrets in development and production
- ❌ Share secrets via email, Slack, or chat
- ❌ Use short or weak secrets (minimum 32 characters)
- ❌ Store secrets in public repositories

**Always**:
- ✅ Add `.env` to `.gitignore`
- ✅ Use environment variables for all secrets
- ✅ Rotate secrets regularly (every 90 days)
- ✅ Use different secrets per environment
- ✅ Generate secrets with `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`

### Deployment

Set environment variables in your hosting platform:

```bash
# Heroku
heroku config:set cookie_password=your_secret_here
heroku config:set db=mongodb+srv://user:pass@cluster.mongodb.net/myapp

# Vercel
vercel env add cookie_password production
vercel env add db production

# Docker
docker run \
  -e NODE_ENV=production \
  -e cookie_password=your_secret_here \
  -e db=mongodb://mongo:27017/myapp \
  myapp

# Railway, Render, Fly.io
# Use web dashboard to set environment variables
```

### Testing configuration

In tests, override environment variables:

```javascript
// test setup
process.env.NODE_ENV = "test";
process.env.cookie_password = "test_secret_32_characters_long!";
process.env.jwt_secret = "test_jwt_secret_also_32_chars!";
process.env.store = "mem";  // Fast in-memory store for tests
process.env.LOG_LEVEL = "silent";  // Suppress logs during tests
```

## Data models

**User**: _id, firstName, lastName, email, password  
**Collection**: _id, title, userid  
**Item**: _id, title, collectionid, plus domain-specific fields (e.g. `artist`, `duration` for a music app — adapt to your domain)  

## Cascade delete

When deleting a collection, delete its items. When deleting a user, delete their collections and items.
