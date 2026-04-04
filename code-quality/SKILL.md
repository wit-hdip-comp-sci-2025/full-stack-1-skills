---
name: code-quality
description: ESLint, Prettier, and Airbnb style guide configuration for consistent code quality and formatting. Use when setting up linting, formatting, or enforcing code standards.
metadata:
  tags: eslint, prettier, airbnb, linting, formatting, code-quality, code-style
---

## When to use

Use this skill when you need to:

- Set up ESLint with Airbnb style guide for code quality and consistency
- Configure Prettier for automatic code formatting
- Integrate ESLint and Prettier to work together without conflicts
- Add npm scripts for linting and formatting
- Configure IDE integration (VSCode) for automatic formatting

## How to use

- **ESLint setup**: [rules/eslint.md](rules/eslint.md) — ESLint configuration with Airbnb style guide, rule customization, and linting workflows.
- **Prettier setup**: [rules/prettier.md](rules/prettier.md) — Prettier configuration, formatting options, and IDE integration.

Use this when starting a new Node.js project or standardizing code quality across an existing codebase.

## Quick start

```bash
# Install dependencies
npm install --save-dev eslint eslint-config-airbnb-base eslint-config-prettier eslint-plugin-import prettier

# Create .eslintrc.json and .prettierrc.json (see rules/ for configs)

# Format and lint
npm run prettier:fix && npm run eslint:fix
```

## npm scripts

Add to `package.json`:

```json
"scripts": {
  "eslint": "eslint . --ext .js",
  "eslint:fix": "eslint . --ext .js --fix",
  "prettier": "prettier --check .",
  "prettier:fix": "prettier --write ."
}
```

**Usage:**

```bash
npm run prettier:fix  # Format code
npm run eslint:fix    # Fix linting issues
npm run prettier      # Check formatting (CI)
npm run eslint        # Check linting (CI)
```
