---
name: hapi-api-development
description: RESTful API endpoints in Hapi with Boom, Joi, and integration tests
metadata:
  tags: api, rest, hapi, boom, joi, axios, endpoints
---

# API development

## Dependencies

```json
"@hapi/boom": "^10.0.1",
"joi": "^17.11.0",
"axios": "^1.13.6"
```

## Project structure

```
src/
├── api-routes.js
└── api/
    └── user-api.js

test/
└── api/
    ├── api-client.js
    └── users-api-test.js
```

## API routes

```javascript
// src/api-routes.js
import { userApi } from "./api/user-api.js";

export const apiRoutes = [
  { method: "GET", path: "/api/users", config: userApi.find },
  { method: "GET", path: "/api/users/{id}", config: userApi.findOne },
  { method: "POST", path: "/api/users", config: userApi.create },
  { method: "DELETE", path: "/api/users/{id}", config: userApi.deleteOne },
  { method: "DELETE", path: "/api/users", config: userApi.deleteAll },
];
```

Register in server: `server.route(apiRoutes)`.

## Handler pattern

Each handler is a Hapi route config object with `auth`, optional `validate`, and `handler`. Two representative examples — the same pattern applies to all CRUD endpoints:

```javascript
// src/api/user-api.js
import Boom from "@hapi/boom";
import { db } from "../models/db.js";
import { UserSpec } from "../models/joi-schemas.js";

export const userApi = {
  findOne: {
    auth: false,
    handler: async function (request, h) {
      try {
        const user = await db.userStore.getUserById(request.params.id);
        if (!user) return Boom.notFound("No User with this id");
        return h.response(user).code(200);
      } catch (err) {
        return Boom.serverUnavailable("Database error");
      }
    },
  },

  create: {
    auth: false,
    validate: {
      payload: UserSpec,
      options: { abortEarly: false },
      failAction: function (request, h, error) {
        return Boom.badRequest(error.message);
      },
    },
    handler: async function (request, h) {
      try {
        const user = await db.userStore.addUser(request.payload);
        return h.response(user).code(201);
      } catch (err) {
        if (err.code === 11000) {
          return Boom.badRequest("Email already registered");
        }
        return Boom.serverUnavailable("Database error");
      }
    },
  },

  // find, deleteOne, deleteAll follow the same structure
};
```

**Key conventions:**
- `find` (GET collection) — return array with 200
- `findOne` (GET by id) — return object with 200, or `Boom.notFound`
- `create` (POST) — validate with Joi, return created object with 201
- `deleteOne` (DELETE by id) — verify exists, then 204
- `deleteAll` (DELETE collection) — 204

## Status codes

| Method | Success | Not Found | Invalid Input |
|--------|---------|-----------|---------------|
| GET /api/users | 200 + array | — | — |
| GET /api/users/{id} | 200 + object | 404 | — |
| POST /api/users | 201 + created | — | 400 |
| DELETE /api/users/{id} | 204 | 404 | — |
| DELETE /api/users | 204 | — | — |

## Error handling

- `Boom.notFound(message)` → 404
- `Boom.badRequest(message)` → 400 (validation or duplicate key)
- `Boom.serverUnavailable(message)` → 503

## API client (tests)

Wrap Axios in a service object so tests read declaratively:

```javascript
// test/api/api-client.js
import axios from "axios";

const client = axios.create({
  baseURL: process.env.API_BASE_URL || "http://localhost:3000",
  headers: { "Content-Type": "application/json" },
});

export const service = {
  async createUser(user) { return (await client.post("/api/users", user)).data; },
  async getUser(id)      { return (await client.get(`/api/users/${id}`)).data; },
  async getAllUsers()     { return (await client.get("/api/users")).data; },
  async deleteUser(id)   { await client.delete(`/api/users/${id}`); },
  async deleteAllUsers() { await client.delete("/api/users"); },
};
```

## Integration test pattern (Mocha)

```javascript
import { assert } from "chai";
import { service } from "./api-client.js";
import { maggie } from "../fixtures.js";
import { assertSubset } from "../test-utils.js";

suite("User API Integration", () => {
  setup(async () => { await service.deleteAllUsers(); });

  test("create a user", async () => {
    const { firstName, lastName, email, password } = maggie;
    const response = await service.createUser({ firstName, lastName, email, password });
    assertSubset({ firstName, lastName, email }, response);
    assert.isDefined(response._id);
  });

  test("get user - not found", async () => {
    try { await service.getUser("invalidid"); assert.fail("Expected 404"); }
    catch (err) { assert.equal(err.response?.status, 404); }
  });
});
```

## E2E test pattern (Playwright)

Playwright's `request` fixture provides direct API access alongside browser tests:

```javascript
test("can create user via API", async ({ request }) => {
  const user = { firstName: "API", lastName: "User", email: `api-${Date.now()}@test.com`, password: "secret" };
  const res = await request.post("/api/users", { data: user });
  expect(res.ok()).toBeTruthy();
  expect(res.status()).toBe(201);
});

test("user created via API can login on web", async ({ request, page }) => {
  const email = `web-${Date.now()}@test.com`;
  await request.post("/api/users", { data: { firstName: "API", lastName: "User", email, password: "secret" } });
  await page.goto("/");
  await page.fill('input[name="email"]', email);
  await page.fill('input[name="password"]', "secret");
  await page.click('button:has-text("Log in")');
  await expect(page).toHaveURL("/dashboard");
});
```

## CORS configuration

When your API is accessed from a different origin (different domain, port, or protocol), browsers enforce CORS (Cross-Origin Resource Sharing). Common scenario: frontend on `localhost:5173`, backend on `localhost:3000`.

### Built-in Hapi CORS (recommended)

Enable CORS for all routes when creating the server:

```javascript
const server = Hapi.server({
  port: process.env.PORT || 3000,
  routes: {
    cors: true,  // Enable CORS for all routes
  },
});
```

This automatically:
- Sets `Access-Control-Allow-Origin` header
- Handles OPTIONS preflight requests
- Works with credentials (cookies, JWT)

### Custom CORS configuration

For production, restrict origins:

```javascript
const server = Hapi.server({
  port: process.env.PORT || 3000,
  routes: {
    cors: {
      origin: process.env.NODE_ENV === "production"
        ? process.env.ALLOWED_ORIGINS?.split(",")  // ["https://example.com"]
        : ["*"],  // Allow all in development
      credentials: true,  // Allow cookies and Authorization headers
    },
  },
});
```

Environment variables:

```env
# .env (development)
NODE_ENV=development

# .env (production)
NODE_ENV=production
ALLOWED_ORIGINS=https://example.com,https://app.example.com
```

### Per-route CORS

Override CORS for specific routes:

```javascript
{
  method: "GET",
  path: "/api/public",
  options: {
    cors: {
      origin: ["*"],  // Public endpoint, allow all origins
    },
    handler: publicHandler,
  },
}
```

### Common pitfall

⚠️ **CORS with credentials**

```javascript
// BAD - allows any origin to send cookies/JWT
routes: {
  cors: {
    origin: ["*"],
    credentials: true,  // Security risk!
  },
}

// GOOD - specific origins only
routes: {
  cors: {
    origin: ["https://example.com"],
    credentials: true,
  },
}
```

## Logging patterns

Proper logging is essential for debugging production issues, monitoring application health, and understanding user behavior. Never rely on `console.log` alone in production.

### Basic logger implementation

Create a logging utility:

```javascript
// src/utils/logger.js
export const logger = {
  info: (message, meta = {}) => {
    console.log(JSON.stringify({
      level: "info",
      message,
      ...meta,
      timestamp: new Date().toISOString(),
    }));
  },

  error: (message, error = {}) => {
    console.error(JSON.stringify({
      level: "error",
      message,
      error: error.message,
      stack: error.stack,
      timestamp: new Date().toISOString(),
    }));
  },

  warn: (message, meta = {}) => {
    console.warn(JSON.stringify({
      level: "warn",
      message,
      ...meta,
      timestamp: new Date().toISOString(),
    }));
  },

  debug: (message, meta = {}) => {
    if (process.env.NODE_ENV === "development") {
      console.debug(JSON.stringify({
        level: "debug",
        message,
        ...meta,
        timestamp: new Date().toISOString(),
      }));
    }
  },
};
```

### Usage in API handlers

```javascript
import { logger } from "../../utils/logger.js";

export const userApi = {
  findOne: {
    auth: false,
    handler: async function (request, h) {
      try {
        logger.debug("Fetching user", { userId: request.params.id });

        const user = await db.userStore.getUserById(request.params.id);

        if (!user) {
          logger.info("User not found", { userId: request.params.id });
          return Boom.notFound("No User with this id");
        }

        logger.info("User retrieved successfully", { userId: user._id });
        return h.response(user).code(200);
      } catch (err) {
        logger.error("Database error in findOne", err);
        return Boom.serverUnavailable("Database error");
      }
    },
  },

  create: {
    auth: false,
    validate: { payload: UserSpec },
    handler: async function (request, h) {
      try {
        const user = await db.userStore.addUser(request.payload);
        logger.info("User created", { userId: user._id, email: user.email });
        return h.response(user).code(201);
      } catch (err) {
        if (err.code === 11000) {
          logger.warn("Duplicate email registration attempt", { email: request.payload.email });
          return Boom.badRequest("Email already registered");
        }
        logger.error("Error creating user", err);
        return Boom.serverUnavailable("Database error");
      }
    },
  },
};
```

### What to log

| Event Type | Level | Include | Example |
|------------|-------|---------|---------|
| User actions | `info` | userId, action, result | User created account |
| API requests | `debug` | method, path, statusCode | GET /api/users → 200 |
| Database errors | `error` | query, error message, stack | MongoDB connection failed |
| Auth failures | `warn` | email (hashed), IP, reason | Invalid password for user@example.com |
| Performance | `debug` | endpoint, duration | /api/users took 234ms |
| Business events | `info` | entityId, event type | Playlist created by user 123 |

### What NOT to log

Security-sensitive information must NEVER appear in logs:

- ❌ **Passwords** (plaintext or hashed)
- ❌ **JWT tokens** or session IDs
- ❌ **Credit card numbers** or payment details
- ❌ **Personal health information** (HIPAA)
- ❌ **API keys** or secrets
- ❌ **Social security numbers** or national IDs

```javascript
// BAD - logs password
logger.info("User login attempt", { email: user.email, password: request.payload.password });

// GOOD - no sensitive data
logger.info("User login attempt", { email: user.email, success: true });
```

### Production logging with Pino

For production, use a proper logging library:

```json
"dependencies": {
  "pino": "^9.6.0",
  "pino-pretty": "^13.0.0"
}
```

```javascript
// src/utils/logger.js
import pino from "pino";

export const logger = pino({
  level: process.env.LOG_LEVEL || "info",
  transport: process.env.NODE_ENV === "development"
    ? { target: "pino-pretty", options: { colorize: true } }
    : undefined,
});
```

Pino features:
- **Fast**: Benchmarked as fastest Node.js logger
- **Structured**: JSON output for log aggregation (ELK, Datadog)
- **Child loggers**: Add context to all logs in a scope
- **Redaction**: Automatically hide sensitive fields

### Child loggers for request context

```javascript
// In handler
const requestLogger = logger.child({ requestId: request.info.id });
requestLogger.info("Processing request");
// Logs: { requestId: "abc123", message: "Processing request" }
```

### Log levels

| Level | When to Use | Production? |
|-------|-------------|-------------|
| `fatal` | Application crash, cannot continue | ✅ Always |
| `error` | Errors that need attention | ✅ Always |
| `warn` | Problems that don't break functionality | ✅ Always |
| `info` | Important business events | ✅ Always |
| `debug` | Detailed diagnostic information | ⚠️ Only if needed |
| `trace` | Very verbose, per-function logging | ❌ Development only |

### Environment configuration

```env
# Development
LOG_LEVEL=debug
NODE_ENV=development

# Production
LOG_LEVEL=info
NODE_ENV=production
```

### Request logging middleware

Log all API requests automatically:

```javascript
// src/utils/request-logger.js
import { logger } from "./logger.js";

export const requestLogger = {
  name: "request-logger",
  version: "1.0.0",
  register: async function (server) {
    server.ext("onPreResponse", (request, h) => {
      const response = request.response;
      const statusCode = response.isBoom ? response.output.statusCode : response.statusCode;

      logger.info("API request", {
        method: request.method.toUpperCase(),
        path: request.path,
        statusCode,
        duration: Date.now() - request.info.received,
      });

      return h.continue;
    });
  },
};

// In app.js
await server.register(requestLogger);
```

### Testing considerations

In tests, you may want to suppress logs:

```javascript
// test setup
import { logger } from "../src/utils/logger.js";

setup(() => {
  // Silence logs during tests
  logger.level = "silent";
});
```

Or capture logs for assertions:

```javascript
import { logger } from "../src/utils/logger.js";

test("logs error on database failure", async () => {
  const logs = [];
  logger.error = (message, error) => logs.push({ message, error });

  // Trigger error
  await db.userStore.addUser({ invalid: "data" });

  assert.equal(logs.length, 1);
  assert.equal(logs[0].message, "Error creating user");
});
```
