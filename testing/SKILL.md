---
name: testing
description: Unit tests (Mocha/Chai), Playwright E2E, fixtures, and development auto-reload with nodemon. Use when writing tests, configuring Playwright, or adding a dev workflow with file watching.
metadata:
  tags: testing, mocha, chai, playwright, e2e, nodemon, autoreload, fixtures
---

## When to use

Use this skill when you need to:

- Write unit tests for models/stores (Mocha/Chai) and E2E specs (Playwright)
- Configure Playwright, fixtures, test isolation, and NPM test scripts
- Run the app in development with automatic restart on file changes (nodemon)
- Avoid restart loops and keep a clear dev vs start split

## How to use

- **E2E and unit testing**: [rules/e2e-testing.md](rules/e2e-testing.md) — Mocha/Chai, Playwright config, fixtures, assertSubset, setup order, E2E auth and isolation, `server.inject()` for in-process API tests, `start-server-and-test` for integration tests.
- **Auto-reload**: [rules/autoreload.md](rules/autoreload.md) — nodemon, dev vs start scripts, nodemon.json, avoiding restart loops.

Refer to the rule that matches your task.
