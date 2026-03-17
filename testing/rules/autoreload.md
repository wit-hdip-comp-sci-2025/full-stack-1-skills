---
name: testing-autoreload
description: Restart Node.js app on file changes with nodemon for development
metadata:
  tags: nodemon, dev, autoreload, development, productivity, node
---

# Development auto-reload (nodemon)

## Dependencies

Add nodemon as a **devDependency**:

```json
"devDependencies": {
  "nodemon": "^3.1.14"
}
```

## NPM scripts

| Script  | Command                 | Use case                  |
| ------- | ----------------------- | ------------------------- |
| `start` | `node src/server.js`    | Production                |
| `dev`   | `nodemon src/server.js` | Development (auto-reload) |

```json
"scripts": {
  "start": "node src/server.js",
  "dev": "nodemon src/server.js"
}
```

## Nodemon config

Create `nodemon.json` in the project root so restarts are predictable and you avoid loops (e.g. lowdb writing `*.json`):

```json
{
  "watch": ["src"],
  "ext": "js",
  "ignore": ["*.json", "test/**", "node_modules/**"],
  "delay": "500"
}
```

| Option   | Purpose                                                                      |
| -------- | ---------------------------------------------------------------------------- |
| `watch`  | Directories to watch (e.g. `["src"]`; omit to watch cwd)                     |
| `ext`    | File extensions that trigger restart (default includes `js,mjson`)           |
| `ignore` | Glob patterns to ignore; exclude data files and tests to avoid restart loops |
| `delay`  | Debounce restarts (ms) so one save doesn't trigger multiple restarts        |

## Avoiding restart loops

- **Ignore JSON data files**: If the app writes to `src/models/json/*.json` or similar, add `*.json` or the data path to `ignore` so writes don't trigger a restart.
- **Ignore test output**: Add `test/**` if test runners write under `test/`.
- **Optional**: Use `"delay": "500"` to coalesce rapid saves.

## Usage

```bash
npm run dev
```

Start the app with nodemon; edit files under `src/` (or whatever `watch` includes) and the server restarts automatically.
