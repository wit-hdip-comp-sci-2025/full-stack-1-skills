---
name: joi-schemas
description: Joi validation schemas and composition patterns for request/response validation
metadata:
  tags: joi, validation, schemas, composition
---

# Joi Schemas

## Dependencies

```json
"joi": "^17.11.0"
```

Install:

```bash
npm install joi
```

## Basic validation

### Simple schema

```javascript
import Joi from "joi";

const userSchema = Joi.object({
  firstName: Joi.string().required(),
  lastName: Joi.string().required(),
  email: Joi.string().email().required(),
  password: Joi.string().min(6).required(),
});

const { error, value } = userSchema.validate(payload, { abortEarly: false });
if (error) return h.view("signup-view", { errors: error.details, user: payload });
```

**Validation options:**

- `abortEarly: false` — Return all errors, not just the first one
- `stripUnknown: true` — Remove fields not in schema
- `convert: true` — Type coercion (default, e.g., "123" → 123)

### Inline validation in routes

```javascript
{
  method: "POST",
  path: "/signup",
  options: {
    validate: {
      payload: Joi.object({
        firstName: Joi.string().required(),
        lastName: Joi.string().required(),
        email: Joi.string().email().required(),
        password: Joi.string().min(6).required(),
      }),
      options: { abortEarly: false },
      failAction: function (request, h, error) {
        return h.view("signup-view", { errors: error.details }).takeover();
      },
    },
    handler: accountsController.signup,
  },
}
```

**failAction** — Custom error handling:
- `(request, h, error)` — Receives request, toolkit, validation error
- `takeover()` — Return response immediately (skip handler)

## Schema composition

Build related schemas by extending base schemas to avoid duplication:

### User schemas

```javascript
// models/joi-schemas.js
import Joi from "joi";

// Base schema - for creation (no DB fields)
export const UserSpec = Joi.object({
  firstName: Joi.string().required().example("Homer"),
  lastName: Joi.string().required().example("Simpson"),
  email: Joi.string().email().required().example("homer@example.com"),
  password: Joi.string().min(6).required().example("secret123"),
}).label("User");

// Extended schema - for responses (includes DB fields)
export const UserSpecPlus = UserSpec.keys({
  _id: Joi.alternatives().try(Joi.string(), Joi.object()).optional(),
  __v: Joi.number().optional(),
}).label("UserDetails");

// Array schema - for list endpoints
export const UserArray = Joi.array()
  .items(UserSpecPlus)
  .label("UserArray");

// Credentials subset - for login (only what's needed)
export const UserCredentialsSpec = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().required(),
}).label("UserCredentials");
```

**Usage in routes:**

```javascript
import { UserSpec, UserSpecPlus, UserArray, UserCredentialsSpec } from "../models/joi-schemas.js";

// Create user - use base schema (no _id)
{
  method: "POST",
  path: "/api/users",
  options: {
    validate: { payload: UserSpec },
    response: { schema: UserSpecPlus },  // Response includes _id
  },
}

// Authenticate - credentials only
{
  method: "POST",
  path: "/api/users/authenticate",
  options: {
    validate: { payload: UserCredentialsSpec },
  },
}

// Get all - array schema
{
  method: "GET",
  path: "/api/users",
  options: {
    response: { schema: UserArray },
  },
}
```

**Benefits:**

- Single source of truth for field definitions
- Changes to base schema propagate automatically
- Clear relationship between schemas (UserSpec → UserSpecPlus)
- Swagger documentation more consistent
- Less code duplication

### Collection and Item schemas

**Pattern for other entities:**

```javascript
// Collection schemas
export const CollectionSpec = Joi.object({
  title: Joi.string().required(),
  userid: Joi.string().optional(),  // Set by server
}).label("Collection");

export const CollectionSpecPlus = CollectionSpec.keys({
  _id: Joi.alternatives().try(Joi.string(), Joi.object()).optional(),
  __v: Joi.number().optional(),
}).label("CollectionDetails");

export const CollectionArray = Joi.array()
  .items(CollectionSpecPlus)
  .label("CollectionArray");

// Item schemas (adapt fields to your domain)
export const ItemSpec = Joi.object({
  title: Joi.string().required(),
  collectionid: Joi.string().optional(),
  // Add domain-specific fields here (e.g., artist, duration for music)
}).label("Item");

export const ItemSpecPlus = ItemSpec.keys({
  _id: Joi.alternatives().try(Joi.string(), Joi.object()).optional(),
  __v: Joi.number().optional(),
}).label("ItemDetails");

export const ItemArray = Joi.array()
  .items(ItemSpecPlus)
  .label("ItemArray");
```

## Schema extension methods

### .keys() - Add or override fields

```javascript
const BaseSchema = Joi.object({
  name: Joi.string().required(),
});

const ExtendedSchema = BaseSchema.keys({
  age: Joi.number().optional(),
  email: Joi.string().email().optional(),
});
```

### .label() - Name for documentation

```javascript
const UserSpec = Joi.object({ /* ... */ }).label("User");
// Swagger will show "User" as the model name
```

### .example() - Sample values

```javascript
const UserSpec = Joi.object({
  firstName: Joi.string().required().example("Homer"),
  lastName: Joi.string().required().example("Simpson"),
  email: Joi.string().email().required().example("homer@simpson.com"),
});
// Swagger documentation shows examples
```

### .description() - Field documentation

```javascript
const UserSpec = Joi.object({
  email: Joi.string()
    .email()
    .required()
    .description("User's email address for login"),
  password: Joi.string()
    .min(6)
    .required()
    .description("Password must be at least 6 characters"),
});
```

## Common validation patterns

### ObjectId (MongoDB)

```javascript
_id: Joi.alternatives().try(Joi.string(), Joi.object()).optional()
```

Accepts both:
- String: `"507f1f77bcf86cd799439011"` (JSON representation)
- Object: `ObjectId("507f1f77bcf86cd799439011")` (MongoDB internal)

### Email validation

```javascript
email: Joi.string().email().required()
```

Built-in email format validation.

### Password requirements

```javascript
password: Joi.string()
  .min(8)
  .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
  .required()
  .description("Password must be at least 8 characters with uppercase, lowercase, and number")
```

### Optional with default

```javascript
role: Joi.string().valid("user", "admin").default("user")
```

### Date validation

```javascript
createdAt: Joi.date().iso().default(() => new Date())
```

### Number ranges

```javascript
age: Joi.number().min(0).max(120).required()
price: Joi.number().positive().precision(2).required()
```

### Conditional validation

```javascript
const schema = Joi.object({
  type: Joi.string().valid("personal", "business").required(),
  taxId: Joi.when("type", {
    is: "business",
    then: Joi.string().required(),
    otherwise: Joi.forbidden(),
  }),
});
```

### Array validation

```javascript
// Array of strings
tags: Joi.array().items(Joi.string()).min(1).max(10)

// Array of objects
items: Joi.array().items(ItemSpec).required()

// Unique array values
emails: Joi.array().items(Joi.string().email()).unique()
```

## Validation in controllers

### Web route validation (form submission)

```javascript
// accounts-controller.js
export const accountsController = {
  signup: {
    handler: async function (request, h) {
      const userSchema = Joi.object({
        firstName: Joi.string().required(),
        lastName: Joi.string().required(),
        email: Joi.string().email().required(),
        password: Joi.string().min(6).required(),
      });

      const { error, value } = userSchema.validate(request.payload, { abortEarly: false });

      if (error) {
        return h.view("signup-view", {
          errors: error.details,
          user: request.payload,
        });
      }

      const user = await db.userStore.addUser(value);
      return h.redirect("/login");
    },
  },
};
```

**Error details structure:**

```javascript
error.details = [
  {
    message: '"firstName" is required',
    path: ["firstName"],
    type: "any.required",
  },
  {
    message: '"email" must be a valid email',
    path: ["email"],
    type: "string.email",
  },
]
```

### API validation (JSON)

```javascript
// user-api.js
import Boom from "@hapi/boom";
import { UserSpec } from "../models/joi-schemas.js";

export const userApi = {
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
        return Boom.serverUnavailable("Database error");
      }
    },
  },
};
```

## Custom error messages

### Per-field messages

```javascript
const schema = Joi.object({
  email: Joi.string()
    .email()
    .required()
    .messages({
      "string.empty": "Email is required",
      "string.email": "Please enter a valid email address",
    }),
  password: Joi.string()
    .min(6)
    .required()
    .messages({
      "string.empty": "Password is required",
      "string.min": "Password must be at least 6 characters",
    }),
});
```

### Global messages

```javascript
const schema = Joi.object({ /* ... */ }).messages({
  "any.required": "{{#label}} is required",
  "string.empty": "{{#label}} cannot be empty",
  "string.email": "{{#label}} must be a valid email",
});
```

## File organization

```
src/
├── models/
│   ├── joi-schemas.js          # All Joi schemas
│   ├── db.js                    # Store abstraction
│   └── mongo/
│       ├── user.js              # Mongoose models
│       └── playlist.js
├── api/
│   ├── user-api.js              # Import schemas for validation
│   └── playlist-api.js
└── controllers/
    └── accounts-controller.js   # Import or define inline
```

**Centralized schemas** (recommended):

```javascript
// models/joi-schemas.js
export const UserSpec = Joi.object({ /* ... */ });
export const UserSpecPlus = UserSpec.keys({ /* ... */ });
export const PlaylistSpec = Joi.object({ /* ... */ });
// ...
```

**Inline schemas** (for simple, route-specific validation):

```javascript
// In route definition
validate: {
  payload: Joi.object({
    email: Joi.string().email().required(),
    password: Joi.string().required(),
  }),
}
```

## Joi vs Mongoose schemas

| Feature | Joi | Mongoose |
|---------|-----|----------|
| **Purpose** | Request/response validation | Database schema + validation |
| **Location** | Routes, controllers, API handlers | MongoDB models |
| **When** | Before data enters system | Before data enters database |
| **Validation** | HTTP payloads, query params | Database documents |
| **Output** | `{ error, value }` | Throws ValidationError |

**Use both:**

```javascript
// Joi validates HTTP request
{
  method: "POST",
  path: "/api/users",
  options: {
    validate: { payload: UserSpec },  // Joi
    handler: async (request, h) => {
      const user = await db.userStore.addUser(request.payload);  // Mongoose validates here
      return h.response(user).code(201);
    },
  },
}

// Mongoose validates before save
const userSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true },
});
```

**Why both?**

- **Joi** — Validates incoming data, provides user-friendly errors
- **Mongoose** — Final validation before database, enforces data integrity

## Testing Joi schemas

```javascript
// test/models/schemas-test.js
import { assert } from "chai";
import { UserSpec } from "../../src/models/joi-schemas.js";

suite("Joi Schema tests", () => {
  test("valid user passes validation", () => {
    const user = {
      firstName: "Homer",
      lastName: "Simpson",
      email: "homer@simpson.com",
      password: "secret123",
    };

    const { error } = UserSpec.validate(user);
    assert.isUndefined(error);
  });

  test("missing required field fails", () => {
    const user = {
      firstName: "Homer",
      lastName: "Simpson",
      // Missing email and password
    };

    const { error } = UserSpec.validate(user);
    assert.isDefined(error);
    assert.equal(error.details.length, 2);
  });

  test("invalid email format fails", () => {
    const user = {
      firstName: "Homer",
      lastName: "Simpson",
      email: "not-an-email",
      password: "secret123",
    };

    const { error } = UserSpec.validate(user);
    assert.isDefined(error);
    assert.include(error.message, "email");
  });
});
```

## Best practices

1. **Centralize schemas** — Define in `models/joi-schemas.js`
2. **Use composition** — Build Base → Plus → Array schemas
3. **Add labels** — For Swagger documentation
4. **Add examples** — For Swagger sample requests
5. **Fail early** — Validate at route level, not in handlers
6. **Reuse schemas** — Same schema for web and API routes
7. **Custom messages** — User-friendly error messages for forms
8. **Test schemas** — Unit test validation rules

## Common pitfalls

### ❌ Duplicating schemas

```javascript
// Bad - same validation in multiple places
const webSchema = Joi.object({ email: Joi.string().email() });
const apiSchema = Joi.object({ email: Joi.string().email() });
```

**✅ Solution:**

```javascript
// Good - single source of truth
export const UserSpec = Joi.object({ email: Joi.string().email() });
```

### ❌ Forgetting .label()

```javascript
// Bad - Swagger shows generic names
const UserSpec = Joi.object({ /* ... */ });
```

**✅ Solution:**

```javascript
// Good - Swagger shows "User"
const UserSpec = Joi.object({ /* ... */ }).label("User");
```

### ❌ Not handling validation errors

```javascript
// Bad - error crashes app
const user = await db.userStore.addUser(request.payload);
```

**✅ Solution:**

```javascript
// Good - validate first
const { error, value } = UserSpec.validate(request.payload);
if (error) return h.view("signup", { errors: error.details });
```

## See also

- [Joi API Reference](https://joi.dev/api/)
- [Hapi Validation](https://hapi.dev/tutorials/validation/)
- [Swagger integration](../hapi-api/rules/swagger.md)
