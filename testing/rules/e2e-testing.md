---
name: testing-e2e-strategies
description: Mocha/Chai unit tests and Playwright E2E tests for a Hapi app
metadata:
  tags: testing, mocha, chai, playwright, e2e, unit-tests, fixtures
---

# E2E and unit testing

## Dependencies

```json
"mocha": "^11.7.5",
"chai": "^6.2.2",
"@playwright/test": "^1.58.2"
```

## Test structure

```
test/
├── fixtures.js
├── test-utils.js
├── models/
│   ├── user-model-test.js
│   ├── collection-model-test.js
│   └── item-model-test.js
└── e2e/
    ├── auth.spec.js
    ├── collection.spec.js
    └── item.spec.js
```

## Fixtures and assertSubset

```javascript
// fixtures.js
export const maggie = { firstName: "Maggie", lastName: "Simpson", email: "maggie@simpson.com", password: "secret" };
export const sampleCollection = { title: "Beethoven Sonatas" };
export const sampleItem = { title: "Piano Concerto No. 1", artist: "Beethoven", duration: 35 }; // fields after title are domain-specific

// test-utils.js
export function assertSubset(subset, superset) {
  for (const key of Object.keys(subset)) {
    assert.property(superset, key);
    assert.equal(superset[key], subset[key]);
  }
}
```

## Unit test pattern (Mocha TDD)

```javascript
suite("User Model tests", () => {
  setup(async () => {
    db.init();
    await db.userStore.deleteAll();
  });

  test("create a user", async () => {
    const newUser = await db.userStore.addUser(maggie);
    assertSubset(maggie, newUser);
    assert.isDefined(newUser._id);
  });
});
```

## Setup order for dependent models

- **User**: `deleteAll` users only
- **Collection**: `deleteAll` collections and users, then `addUser(maggie)` for testUser
- **Item**: create user, create collection, `deleteAll` items

## Playwright config

```javascript
export default {
  testDir: "./test/e2e",
  webServer: { command: "npm start", url: "http://localhost:3000", reuseExistingServer: !process.env.CI },
  use: { baseURL: "http://localhost:3000" },
};
```

## E2E auth and isolation

- Use unique emails per test: `maggie_${Date.now()}@test.com`
- Wait for redirects: `await expect(page).toHaveURL("/dashboard")`
- Use mem store (`store=mem`) or `fullyParallel: false` if tests share state

## In-process API testing with server.inject()

Use the `createServer()` pattern (see [hapi-web foundations](../../hapi-web/rules/foundations.md)) to test API handlers without an HTTP listener:

```javascript
import { createServer } from "../../src/app.js";

let server;

setup(async () => {
  server = await createServer();
  await db.userStore.deleteAll();
});

test("create user via API", async () => {
  const res = await server.inject({
    method: "POST",
    url: "/api/users",
    payload: maggie,
    headers: { "Content-Type": "application/json" },
  });
  assert.equal(res.statusCode, 201);
  assertSubset(maggie, JSON.parse(res.payload));
});
```

## Coordinating server + tests with start-server-and-test

For integration tests that need a running HTTP server:

```json
"devDependencies": {
  "start-server-and-test": "^2.0.12"
}
```

```json
"scripts": {
  "test:api": "start-server-and-test start http://localhost:3000 test:integration"
}
```

This starts the server, waits for the URL to respond, runs the test command, then shuts down.

## NPM scripts

```json
"test": "mocha --ui tdd test/models/**/*.js",
"test:e2e": "npx playwright test",
"test:e2e:ui": "npx playwright test --ui",
"test:api": "start-server-and-test start http://localhost:3000 test:integration"
```
