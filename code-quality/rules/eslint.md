---
name: eslint
description: ESLint and Airbnb style guide configuration for code quality and consistency
metadata:
  tags: eslint, airbnb, linting, code-quality, code-style
---

# ESLint

## Dependencies

```json
"devDependencies": {
  "eslint": "^8.57.1",
  "eslint-config-airbnb-base": "^15.0.0",
  "eslint-config-prettier": "^10.1.8",
  "eslint-plugin-import": "^2.32.0"
}
```

Install:

```bash
npm install --save-dev eslint eslint-config-airbnb-base eslint-config-prettier eslint-plugin-import
```

**Note:** `eslint-config-prettier` is required to prevent conflicts with Prettier. See [prettier.md](prettier.md) for Prettier setup.

## Configuration

### .eslintrc.json

Create in project root:

```json
{
  "env": {
    "node": true,
    "es2021": true
  },
  "extends": ["airbnb-base", "prettier"],
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "rules": {
    "quotes": ["error", "double"],
    "import/extensions": "off",
    "import/prefer-default-export": "off",
    "object-shorthand": "off",
    "no-unused-vars": "off",
    "no-underscore-dangle": "off",
    "no-param-reassign": "off",
    "no-undef": "off",
    "import/no-unresolved": "off",
    "import/no-extraneous-dependencies": "off",
    "no-console": "off"
  }
}
```

**Key settings:**

- `env: { "node": true }` — Node.js environment (no browser globals)
- `extends: ["airbnb-base", "prettier"]` — Airbnb style guide + Prettier compatibility
- `sourceType: "module"` — ESM support (import/export)
- `ecmaVersion: "latest"` — Modern JavaScript features

**Rule overrides explained:**

- `quotes: ["error", "double"]` — Enforce double quotes (matches Prettier)
- `import/extensions: "off"` — Allow imports without .js extension
- `import/prefer-default-export: "off"` — Allow named exports without default
- `object-shorthand: "off"` — Allow `{ foo: foo }` instead of requiring `{ foo }`
- `no-unused-vars: "off"` — Don't error on unused variables (e.g., `h` toolkit parameter)
- `no-underscore-dangle: "off"` — Allow `_id` (MongoDB), `__dirname`, `__v`
- `no-param-reassign: "off"` — Allow modifying function parameters (common in Hapi handlers)
- `no-undef: "off"` — Don't error on undefined variables
- `import/no-unresolved: "off"` — Don't error on unresolved imports
- `import/no-extraneous-dependencies: "off"` — Allow imports from devDependencies
- `no-console: "off"` — Allow console.log (useful for debugging)

### .eslintignore (optional)

Exclude files from linting:

```
node_modules/
dist/
build/
coverage/
*.min.js
```

## npm scripts

Add to `package.json`:

```json
"scripts": {
  "eslint": "eslint . --ext .js",
  "eslint:fix": "eslint . --ext .js --fix"
}
```

**Usage:**

```bash
npm run eslint        # Check for linting errors
npm run eslint:fix    # Auto-fix linting errors
```

**Combined with Prettier:**

```bash
npm run prettier:fix && npm run eslint:fix
```

See [prettier.md](prettier.md) for Prettier scripts.

## IDE integration (VSCode)

### ESLint extension

Install from VSCode marketplace:

**ESLint** (`dbaeumer.vscode-eslint`)

### Settings (.vscode/settings.json)

Create in project root:

```json
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

This auto-fixes ESLint issues on save.

**Complete setup with Prettier:**

```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

See [prettier.md](prettier.md) for Prettier extension setup.

## Airbnb style guide overview

Key rules from Airbnb JavaScript Style Guide:

### Variables

```javascript
// Use const for all references (don't use var)
const foo = 1;
const bar = 2;

// Use let if you must reassign references
let count = 1;
count += 1;

// One variable per declaration
const a = 1;
const b = 2;
```

### Objects

```javascript
// Use literal syntax
const obj = {};

// Use property shorthand
const name = "Alice";
const user = { name };  // Instead of { name: name }

// Use object spread for shallow copy
const copy = { ...original };
```

### Arrays

```javascript
// Use literal syntax
const arr = [];

// Use Array.push() instead of direct assignment
arr.push(item);

// Use array spreads for copy
const copy = [...original];
```

### Destructuring

```javascript
// Use for objects
const { firstName, lastName } = user;

// Use for arrays
const [first, second] = arr;

// Use in function parameters
function getFullName({ firstName, lastName }) {
  return `${firstName} ${lastName}`;
}
```

### Strings

```javascript
// Use double quotes (with our Prettier config)
const name = "Alice";

// Use template literals for concatenation
const greeting = `Hello, ${name}!`;
```

### Functions

```javascript
// Use arrow functions for anonymous functions
const arr = [1, 2, 3].map((x) => x * 2);

// Use default parameters
function greet(name = "Guest") {
  return `Hello, ${name}`;
}
```

### Modules

```javascript
// Always use import/export (ESM)
import { something } from "./module.js";
export const myFunction = () => {};

// Prefer named exports
export const foo = 1;
export const bar = 2;
```

### Naming conventions

```javascript
// camelCase for variables and functions
const myVariable = 1;
function myFunction() {}

// PascalCase for classes and constructors
class MyClass {}

// UPPERCASE for constants (optional)
const API_KEY = "abc123";
```

## Customizing rules

The default configuration above includes overrides for Hapi + MongoDB development. To add additional rules or modify existing ones, update the `rules` object:

### Stricter unused variables (allow only `h` toolkit)

```json
{
  "rules": {
    "no-unused-vars": ["error", { "argsIgnorePattern": "^h$" }]
  }
}
```

### Enforce import extensions

```json
{
  "rules": {
    "import/extensions": ["error", "always", { "ignorePackages": true }]
  }
}
```

### Require consistent return statements

```json
{
  "rules": {
    "consistent-return": "error"
  }
}
```

### Specific underscore dangle rules

```json
{
  "rules": {
    "no-underscore-dangle": ["error", { "allow": ["_id", "__v", "__dirname"] }]
  }
}
```

### Re-enable console warnings in production

```json
{
  "rules": {
    "no-console": ["warn", { "allow": ["warn", "error"] }]
  }
}
```

**Note:** The default configuration turns off many Airbnb rules for flexibility during development. As your project matures, consider gradually re-enabling rules to enforce stricter code quality.

## Workflows

### Initial setup

```bash
# 1. Install dependencies
npm install --save-dev eslint eslint-config-airbnb-base eslint-config-prettier eslint-plugin-import

# 2. Create .eslintrc.json (see above)

# 3. Add npm scripts to package.json

# 4. Fix existing code
npm run eslint:fix

# 5. Check for remaining issues
npm run eslint
```

### Pre-commit workflow

Before committing code:

```bash
npm run eslint:fix    # Fix auto-fixable issues
npm run eslint        # Check for remaining issues
git add .
git commit -m "feat: add new feature"
```

**With Prettier:**

```bash
npm run prettier:fix  # Format code first
npm run eslint:fix    # Then fix linting
npm run eslint        # Check for remaining issues
```

### CI/CD integration

Add linting check to CI pipeline:

```yaml
# .github/workflows/lint.yml
name: Code Quality

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 20
      - run: npm ci
      - run: npm run eslint
```

**With Prettier:**

```yaml
- run: npm run prettier
- run: npm run eslint
```

## Troubleshooting

### ESLint and Prettier conflicts

**Symptom:** Code formatted by Prettier is flagged by ESLint.

**Solution:** Ensure `eslint-config-prettier` is last in extends array:

```json
{
  "extends": [
    "airbnb-base",
    "prettier"  // Must be last
  ]
}
```

### Import/no-unresolved errors

**Symptom:** ESLint can't resolve imports with `.js` extensions.

**Solution:** Configure import plugin:

```json
{
  "settings": {
    "import/resolver": {
      "node": {
        "extensions": [".js", ".json"]
      }
    }
  }
}
```

Or disable the rule (already in default config):

```json
{
  "rules": {
    "import/no-unresolved": "off"
  }
}
```

### Linting node_modules

**Symptom:** ESLint checks thousands of files in node_modules.

**Solution:** Add `.eslintignore`:

```
node_modules/
```

Or use `--ignore-path`:

```json
{
  "scripts": {
    "eslint": "eslint . --ext .js --ignore-path .gitignore"
  }
}
```

### VSCode not auto-fixing

**Checklist:**

1. ✅ ESLint extension installed
2. ✅ `.vscode/settings.json` configured with `source.fixAll.eslint`
3. ✅ `.eslintrc.json` exists in project root
4. ✅ Restart VSCode

**Test manually:** Right-click → `Fix all auto-fixable problems`

## Best practices

### When to run linting

| Stage | Command | Purpose |
|-------|---------|---------|
| During development | Auto-fix on save (IDE) | Continuous linting |
| Before commit | `npm run eslint:fix` | Fix issues early |
| In CI/CD | `npm run eslint` | Prevent bad code from merging |
| Pre-release | `npm run eslint` | Final quality check |

### Team adoption

1. **Add to package.json** — Ensure everyone uses same ESLint version
2. **Commit .eslintrc.json** — Share ESLint configuration
3. **Document in README** — Explain how to set up IDE
4. **Add to CI** — Enforce on pull requests
5. **Use pre-commit hooks** (optional) — Auto-lint before commits

### Pre-commit hooks with Husky (optional)

Automatically lint staged files before each commit:

```bash
npm install --save-dev husky lint-staged
npx husky init
```

Create `.husky/pre-commit`:

```bash
#!/bin/sh
npx lint-staged
```

Add to `package.json`:

```json
{
  "lint-staged": {
    "*.js": [
      "prettier --write",
      "eslint --fix"
    ]
  }
}
```

Now every commit automatically:
1. Formats staged `.js` files with Prettier
2. Fixes ESLint issues in staged files
3. Adds fixed files back to the commit

**Note:** Commits will fail if ESLint finds unfixable errors.

## Why ESLint?

| Feature | Benefit |
|---------|---------|
| **Static analysis** | Catches errors before runtime |
| **Airbnb style guide** | Industry-standard, consistent code |
| **Auto-fix** | Automatically fixes many issues |
| **Extensible** | Customize rules for your project |
| **IDE integration** | Real-time feedback while coding |

**ESLint vs Prettier:**
- **ESLint** — Catches bugs and enforces best practices (logic and patterns)
- **Prettier** — Handles code formatting (whitespace, quotes, commas)
- **Together** — No overlap, no conflicts when using `eslint-config-prettier`

## See also

- [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
- [ESLint Rules Reference](https://eslint.org/docs/latest/rules/)
- [Prettier setup](prettier.md)
