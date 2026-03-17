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

```javascript
// src/api/user-api.js
import Boom from "@hapi/boom";
import { db } from "../models/db.js";
import { UserSpec } from "../models/joi-schemas.js";

export const userApi = {
  find: {
    auth: false,
    handler: async function (request, h) {
      try {
        const users = await db.userStore.getAllUsers();
        return h.response(users).code(200);
      } catch (err) {
        return Boom.serverUnavailable("Database error");
      }
    },
  },

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

  deleteOne: {
    auth: false,
    handler: async function (request, h) {
      try {
        const user = await db.userStore.getUserById(request.params.id);
        if (!user) return Boom.notFound("No User with this id");
        await db.userStore.deleteUserById(request.params.id);
        return h.response().code(204);
      } catch (err) {
        return Boom.serverUnavailable("Database error");
      }
    },
  },

  deleteAll: {
    auth: false,
    handler: async function (request, h) {
      try {
        await db.userStore.deleteAll();
        return h.response().code(204);
      } catch (err) {
        return Boom.serverUnavailable("Database error");
      }
    },
  },
};
```

## Status codes

| Method | Success | Not Found | Invalid Input |
|--------|---------|-----------|---------------|
| GET /api/users | 200 + array | — | — |
| GET /api/users/{id} | 200 + object | 404 | — |
| POST /api/users | 201 + created | — | 400 |
| DELETE /api/users/{id} | 204 | 404 | — |
| DELETE /api/users | 204 | — | — |

## Error handling

- `Boom.notFound("No User with this id")` → 404
- `Boom.badRequest(error.message)` → 400 (validation)
- `Boom.badRequest("Email already registered")` → 400 (MongoDB duplicate key 11000)
- `Boom.serverUnavailable("Database error")` → 500

## API client (tests)

```javascript
// test/api/api-client.js
import axios from "axios";

const baseURL = process.env.API_BASE_URL || "http://localhost:3000";

const client = axios.create({
  baseURL,
  headers: { "Content-Type": "application/json" },
});

export const service = {
  async createUser(user) {
    const response = await client.post("/api/users", user);
    return response.data;
  },

  async getUser(id) {
    const response = await client.get(`/api/users/${id}`);
    return response.data;
  },

  async getAllUsers() {
    const response = await client.get("/api/users");
    return response.data;
  },

  async deleteUser(id) {
    await client.delete(`/api/users/${id}`);
  },

  async deleteAllUsers() {
    await client.delete("/api/users");
  },
};
```

## Integration test pattern (Mocha)

```javascript
// test/api/users-api-test.js
import { assert } from "chai";
import { service } from "./api-client.js";
import { maggie, testUsers } from "../fixtures.js";
import { assertSubset } from "../test-utils.js";

suite("User API Integration", () => {
  setup(async () => {
    await service.deleteAllUsers();
  });

  test("create a user", async () => {
    const input = { firstName: maggie.firstName, lastName: maggie.lastName, email: maggie.email, password: maggie.password };
    const response = await service.createUser(input);
    assertSubset(input, response);
    assert.isDefined(response._id);
  });

  test("get user - not found", async () => {
    try {
      await service.getUser("invalidid");
      assert.fail("Expected 404");
    } catch (err) {
      assert.equal(err.response?.status, 404);
    }
  });

  test("delete user", async () => {
    const created = await service.createUser(maggie);
    await service.deleteUser(created._id);
    try {
      await service.getUser(created._id);
      assert.fail("Expected 404");
    } catch (err) {
      assert.equal(err.response?.status, 404);
    }
  });
});
```

## E2E test pattern (Playwright)

```javascript
// test/e2e/user-api.spec.js
function uniqueEmail() {
  return `api-${Date.now()}-${Math.random().toString(36).slice(2, 9)}@example.com`;
}

test("can create user via API", async ({ request }) => {
  const user = { firstName: "API", lastName: "User", email: uniqueEmail(), password: "secret" };
  const res = await request.post("/api/users", { data: user });
  expect(res.ok()).toBeTruthy();
  expect(res.status()).toBe(201);
  const body = await res.json();
  expect(body).toMatchObject(user);
  expect(body._id).toBeDefined();
});

test("API returns 400 for invalid payload", async ({ request }) => {
  const res = await request.post("/api/users", { data: { firstName: "Only" } });
  expect(res.status()).toBe(400);
});
```

## API–web integration (Playwright)

```javascript
// test/e2e/api-web-integration.spec.js
test("user created via API can login on web", async ({ request, page }) => {
  const email = uniqueEmail();
  const user = { firstName: "API", lastName: "User", email, password: "secret" };
  const createRes = await request.post("/api/users", { data: user });
  expect(createRes.ok()).toBeTruthy();

  await page.goto("/");
  await page.fill('input[name="email"]', email);
  await page.fill('input[name="password"]', "secret");
  await page.click('button:has-text("Log in")');
  await expect(page).toHaveURL("/dashboard");
});
```
