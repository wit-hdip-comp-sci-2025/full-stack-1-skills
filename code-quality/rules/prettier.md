---
name: prettier
description: Prettier configuration for automatic code formatting
metadata:
  tags: prettier, formatting, code-style
---

# Prettier

## Dependencies

```json
"devDependencies": {
  "prettier": "^3.6.2"
}
```

Install:

```bash
npm install --save-dev prettier
```

## Configuration

### .prettierrc.json

Create in project root:

```json
{
  "trailingComma": "all",
  "singleQuote": false,
  "printWidth": 10000
}
```

**Settings explained:**

- `trailingComma: "all"` — Add trailing commas wherever possible (ES5+)
- `singleQuote: false` — Use double quotes (default, matches ESLint)
- `printWidth: 10000` — Effectively disables line wrapping (keeps code on single lines)

### printWidth options

This configuration prioritizes horizontal scrolling over automatic line breaks. Adjust `printWidth` based on team preference:

```json
{
  "printWidth": 80    // Standard (wraps at 80 chars)
  "printWidth": 100   // Moderate
  "printWidth": 120   // Wide
  "printWidth": 10000 // No wrapping (default in this framework)
}
```

**Why disable line wrapping?**

The high `printWidth` value prevents Prettier from breaking long lines. This works well when:
- Using wide monitors
- Preferring horizontal scrolling
- Keeping related code on one line

**When to use lower values:**

- Code reviews on mobile/narrow screens
- Strict 80-character line length policies
- Working with deeply nested code

### Other common options

```json
{
  "semi": true,              // Add semicolons (default)
  "tabWidth": 2,             // Use 2 spaces for indentation (default)
  "useTabs": false,          // Use spaces, not tabs (default)
  "bracketSpacing": true,    // { foo: bar } not {foo: bar} (default)
  "arrowParens": "always"    // (x) => x not x => x (default)
}
```

See [Prettier Options](https://prettier.io/docs/en/options.html) for full list.

### .prettierignore (optional)

Exclude files from formatting:

```
node_modules/
dist/
build/
coverage/
package-lock.json
*.min.js
```

## npm scripts

Add to `package.json`:

```json
"scripts": {
  "prettier": "prettier --check .",
  "prettier:fix": "prettier --write ."
}
```

**Usage:**

```bash
npm run prettier      # Check formatting (no changes)
npm run prettier:fix  # Apply formatting (write changes)
```

**Combined with ESLint:**

```bash
npm run prettier:fix && npm run eslint:fix
```

Format first (Prettier), then lint (ESLint). See [eslint.md](eslint.md) for ESLint setup.

## IDE integration (VSCode)

### Prettier extension

Install from VSCode marketplace:

**Prettier - Code formatter** (`esbenp.prettier-vscode`)

### Settings (.vscode/settings.json)

Create in project root:

```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

This auto-formats with Prettier on save.

**Complete setup with ESLint:**

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

See [eslint.md](eslint.md) for ESLint extension setup.

### Workspace vs User settings

**Project settings** (`.vscode/settings.json`):
- Committed to git
- Shared across team
- Project-specific formatting rules

**User settings** (VSCode preferences):
- Not committed to git
- Personal preferences
- Apply globally across all projects

Prefer project settings for consistency.

## Workflows

### Initial setup

```bash
# 1. Install Prettier
npm install --save-dev prettier

# 2. Create .prettierrc.json (see above)

# 3. Add npm scripts to package.json

# 4. Format existing code
npm run prettier:fix

# 5. Verify formatting
npm run prettier
```

### Pre-commit workflow

Before committing code:

```bash
npm run prettier:fix  # Format code
git add .
git commit -m "feat: add new feature"
```

**With ESLint:**

```bash
npm run prettier:fix  # Format first
npm run eslint:fix    # Then fix linting
npm run eslint        # Check for remaining issues
```

### CI/CD integration

Add formatting check to CI pipeline:

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
      - run: npm run prettier
```

**With ESLint:**

```yaml
- run: npm run prettier
- run: npm run eslint
```

## Troubleshooting

### VSCode not formatting on save

**Checklist:**

1. ✅ Prettier extension installed
2. ✅ `.vscode/settings.json` configured
3. ✅ `.prettierrc.json` exists in project root
4. ✅ Restart VSCode

**Test manually:** Right-click → Format Document (Shift+Option+F / Shift+Alt+F)

### Prettier formats but file reverts

**Symptom:** File formats on save but immediately reverts to original.

**Cause:** Another extension (e.g., ESLint) is also formatting.

**Solution:** Set Prettier as default formatter:

```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

### Prettier ignores .prettierrc.json

**Symptom:** Configuration file exists but Prettier uses defaults.

**Causes:**
1. File is in wrong location (must be in project root)
2. JSON syntax error in config file
3. VSCode using global Prettier instead of project Prettier

**Solution:** Check file location and validate JSON:

```bash
# Validate JSON
cat .prettierrc.json | jq .

# Ensure project Prettier is used
npm run prettier:fix
```

### Prettier and ESLint conflict

**Symptom:** Prettier formats code, then ESLint flags formatting errors.

**Solution:** Install `eslint-config-prettier` to disable conflicting ESLint rules:

```bash
npm install --save-dev eslint-config-prettier
```

Update `.eslintrc.json`:

```json
{
  "extends": ["airbnb-base", "prettier"]
}
```

See [eslint.md](eslint.md) for full ESLint setup.

## Best practices

### When to run formatting

| Stage | Command | Purpose |
|-------|---------|---------|
| During development | Auto-format on save (IDE) | Continuous formatting |
| Before commit | `npm run prettier:fix` | Ensure consistent formatting |
| In CI/CD | `npm run prettier` | Prevent unformatted code from merging |
| Pre-release | `npm run prettier` | Final consistency check |

### Team adoption

1. **Add to package.json** — Ensure everyone uses same Prettier version
2. **Commit .prettierrc.json** — Share formatting configuration
3. **Commit .vscode/settings.json** — Enable auto-format on save for all
4. **Document in README** — Explain how to set up IDE
5. **Add to CI** — Enforce formatting on pull requests
6. **Use pre-commit hooks** (optional) — Auto-format before commits

### Pre-commit hooks with Husky (optional)

Automatically format staged files before each commit:

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

**Note:** Prettier rarely fails (it always formats), but ESLint may block commits if it finds unfixable errors.

## Prettier vs ESLint

| Tool | Purpose | Example Rules |
|------|---------|---------------|
| **Prettier** | Code formatting (whitespace, quotes, commas) | Single/double quotes, trailing commas, line width |
| **ESLint** | Code quality (logic, patterns, bugs) | Unused variables, missing imports, consistent returns |

**Use both:**
- Prettier formats code automatically
- ESLint catches bugs and enforces best practices
- `eslint-config-prettier` prevents conflicts

**Why separate?**
- Prettier is opinionated (few options, zero debates)
- ESLint is configurable (hundreds of rules)
- Together: format first, then lint

## Format on save: Prettier vs ESLint

**Recommended:** Use Prettier for formatting, ESLint for linting:

```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

**What happens on save:**
1. Prettier formats the file (quotes, commas, spacing)
2. ESLint fixes auto-fixable issues (unused vars, import order)

**Why not use ESLint for formatting?**
- Prettier is faster
- Prettier handles edge cases better
- ESLint formatting rules conflict with Prettier

## Why Prettier?

| Feature | Benefit |
|---------|---------|
| **Opinionated** | No style debates, just format |
| **Fast** | Formats entire codebase in seconds |
| **Automatic** | Set and forget (format on save) |
| **Consistent** | Same output every time |
| **Zero config** | Works out of the box |

**Prettier = Zero Style Decisions**

Instead of debating:
- Single vs double quotes
- Trailing commas or not
- Line width 80 vs 100

Just run Prettier and move on to building features.

## See also

- [Prettier Options](https://prettier.io/docs/en/options.html)
- [Prettier Playground](https://prettier.io/playground/) — Test configuration
- [ESLint setup](eslint.md)
