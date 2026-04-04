# Glossary

This glossary defines terms, acronyms, and concepts used throughout the skills framework.

---

## Testing & Development

### TDD (Test-Driven Development)
Development approach where tests are written before implementation code. Mocha's TDD interface uses `suite()`, `test()`, `setup()`, `teardown()`.

```javascript
suite("User tests", () => {
  test("create a user", async () => {
    // Test implementation
  });
});
```

### BDD (Behavior-Driven Development)
Development approach focusing on behavior specifications. Mocha's BDD interface uses `describe()`, `it()`, `before()`, `after()`. Not used in this framework (we use TDD).

### E2E (End-to-End)
Testing that validates complete user workflows through the application, including UI, backend, and database. Uses Playwright to automate browser interactions.

### Unit Test
Test that validates a single function or component in isolation (e.g., testing a store method).

### Integration Test
Test that validates multiple components working together (e.g., API endpoint calling database).

### Fixture
Predefined test data used across multiple tests. Stored in `test/fixtures.js`.

```javascript
export const maggie = {
  firstName: "Maggie",
  lastName: "Simpson",
  email: "maggie@simpson.com",
  password: "secret"
};
```

### Assertion
Statement that verifies expected behavior. Uses Chai library.

```javascript
assert.equal(user.email, "test@example.com");
assert.isDefined(user._id);
```

---

## JavaScript & Node.js

### ESM (ECMAScript Modules)
Modern JavaScript module system using `import`/`export`. Enabled with `"type": "module"` in package.json.

```javascript
import { something } from "./module.js";
export const myFunction = () => {};
```

### CJS (CommonJS)
Older Node.js module system using `require()`/`module.exports`. Not used in this framework.

### `__dirname` in ESM
ESM doesn't provide `__dirname`. Must create it manually:

```javascript
import path from "path";
import { fileURLToPath } from "url";

const __dirname = path.dirname(fileURLToPath(import.meta.url));
```

### `import.meta.url`
ESM variable containing the URL of the current module file.

### Async/Await
Modern syntax for handling asynchronous operations (Promises).

```javascript
const user = await db.userStore.getUserById(id);
```

---

## Database

### ODM (Object-Document Mapping)
Library that maps JavaScript objects to database documents. **Mongoose** is an ODM for MongoDB.

```javascript
const userSchema = new mongoose.Schema({ firstName: String });
const User = mongoose.model("User", userSchema);
```

### ORM (Object-Relational Mapping)
Library that maps JavaScript objects to relational database tables. Examples: Sequelize (SQL), Prisma (SQL). Not used in this framework (we use MongoDB, which uses ODM).

### Schema
Definition of data structure, including field types, validation rules, and relationships.

**Joi Schema** (validation):
```javascript
const UserSpec = Joi.object({
  firstName: Joi.string().required(),
  email: Joi.string().email().required(),
});
```

**Mongoose Schema** (database):
```javascript
const userSchema = new mongoose.Schema({
  firstName: { type: String, required: true },
  email: { type: String, required: true, unique: true },
});
```

### ObjectId
MongoDB's unique identifier type. 24-character hexadecimal string.

```javascript
// Example: "507f1f77bcf86cd799439011"
const id = user._id; // ObjectId
```

### Seed / Seeding
Process of populating database with initial or test data.

```javascript
export const seedData = {
  users: {
    homer: { firstName: "Homer", email: "homer@test.com" },
  },
};
```

### Cascade Delete
Deleting related records when parent is deleted. Example: deleting all playlists when user is deleted.

---

## API & HTTP

### REST (Representational State Transfer)
Architectural style for APIs using HTTP methods and resource-based URLs.

| Method | Operation | Example |
|--------|-----------|---------|
| GET | Read | GET /api/users |
| POST | Create | POST /api/users |
| PUT/PATCH | Update | PUT /api/users/123 |
| DELETE | Delete | DELETE /api/users/123 |

### CRUD (Create, Read, Update, Delete)
Four basic operations for data persistence.

### Status Code
HTTP response code indicating request result.

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Successful GET, PUT |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Missing/invalid auth |
| 404 | Not Found | Resource doesn't exist |
| 500 | Server Error | Unexpected error |

### CORS (Cross-Origin Resource Sharing)
Browser security feature that restricts requests between different origins (domain, port, or protocol).

**Example**: Frontend on `localhost:5173` calling API on `localhost:3000` requires CORS.

```javascript
const server = Hapi.server({
  port: 3000,
  routes: { cors: true },
});
```

### Preflight Request
OPTIONS request sent by browser before actual request to check CORS permissions. Hapi handles automatically when `cors: true`.

### OpenAPI / Swagger
Standard for documenting REST APIs. Provides interactive UI to test endpoints.

```javascript
// View at http://localhost:3000/documentation
await server.register(require("hapi-swagger"));
```

---

## Authentication & Security

### JWT (JSON Web Token)
Compact, URL-safe token for authentication. Contains encoded user information and signature.

**Format**: `header.payload.signature`

```javascript
const token = jwt.sign({ id: user._id }, secret, { algorithm: "HS256" });
// Returns: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### Bearer Token
Authentication scheme where token is sent in `Authorization` header.

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Session
Server-side storage of user authentication state. Uses encrypted cookie with session ID.

```javascript
// Hapi session cookie
server.auth.strategy("session", "cookie", {
  cookie: { name: "sid", password: "secret" },
});
```

### Cookie
Small data stored in browser, sent with every request. Used for session authentication.

### Strategy (Authentication)
Named authentication method in Hapi.

```javascript
// Session strategy (cookie-based)
server.auth.strategy("session", "cookie", { /* ... */ });

// JWT strategy (token-based)
server.auth.strategy("jwt", "jwt", { /* ... */ });
```

### Dual Auth
Using different authentication strategies for web (session) and API (JWT) in same app.

```javascript
// Web route uses session
{ path: "/dashboard", options: { auth: "session" } }

// API route uses JWT
{ path: "/api/users", options: { auth: "jwt" } }
```

### Credentials
User's login information (email + password) or authenticated user object stored in session/token.

### Hash (Password)
One-way encryption of password for secure storage. **Never store passwords in plain text**.

---

## Hapi Framework

### Vision
Hapi plugin for rendering server-side templates (Handlebars, Pug, etc.).

```javascript
import Vision from "@hapi/vision";
await server.register(Vision);
```

### Inert
Hapi plugin for serving static files (images, CSS, JavaScript).

```javascript
import Inert from "@hapi/inert";
await server.register(Inert);
```

### Boom
Hapi library for HTTP errors with proper status codes.

```javascript
import Boom from "@hapi/boom";
return Boom.notFound("User not found");      // 404
return Boom.badRequest("Invalid email");     // 400
return Boom.unauthorized("Invalid token");   // 401
```

### Handler
Function that processes HTTP request and returns response.

```javascript
const handler = async (request, h) => {
  const users = await db.userStore.getAllUsers();
  return h.response(users).code(200);
};
```

### Toolkit (`h`)
Hapi response toolkit passed to handlers. Used to create responses.

```javascript
h.response(data).code(200)  // JSON response
h.view("template", data)    // Render view
h.redirect("/path")         // Redirect
```

### Route Config
Object defining route behavior (path, method, auth, validation, handler).

```javascript
{
  method: "POST",
  path: "/api/users",
  options: {
    auth: "jwt",
    validate: { payload: UserSpec },
    handler: async (request, h) => { /* ... */ },
  },
}
```

### Plugin
Reusable Hapi module that extends server functionality.

```javascript
await server.register(Vision);
await server.register(Cookie);
```

---

## Validation & Schemas

### Joi
Validation library for JavaScript objects. Used for request/response validation.

```javascript
const schema = Joi.object({
  email: Joi.string().email().required(),
  age: Joi.number().min(0).max(120).optional(),
});
```

### Validation
Process of ensuring data meets requirements before processing.

```javascript
const { error, value } = schema.validate(payload);
if (error) return Boom.badRequest(error.message);
```

### Schema Composition
Building complex schemas by extending base schemas.

```javascript
const UserSpec = Joi.object({ /* base fields */ });
const UserSpecPlus = UserSpec.keys({ _id: Joi.string() }); // Extended
```

### `.label()`
Joi method to name schema for Swagger documentation.

```javascript
const UserSpec = Joi.object({ /* ... */ }).label("User");
```

### `.example()`
Joi method to provide sample value for Swagger documentation.

```javascript
firstName: Joi.string().required().example("Homer")
```

---

## Templates & Views

### Handlebars
Template engine for generating HTML from data.

```handlebars
<h1>Welcome, {{user.firstName}}!</h1>
{{#if isAdmin}}
  <a href="/admin">Admin Panel</a>
{{/if}}
```

### Layout
Master template that wraps page content. Contains common elements (header, footer).

### Partial
Reusable template snippet (navbar, card, etc.).

```handlebars
{{> navbar}}  <!-- Includes navbar partial -->
```

### Helper
Custom function used in Handlebars templates.

```javascript
Handlebars.registerHelper("eq", (a, b) => a === b);
```

```handlebars
{{#if (eq currentPath "/dashboard")}}active{{/if}}
```

---

## Development Tools

### Nodemon
Tool that auto-restarts Node.js app when files change.

```bash
npm run dev  # Uses nodemon
```

### ESLint
Tool that analyzes code for errors and style issues.

```bash
npm run lint
```

### Prettier
Code formatter that enforces consistent style.

```bash
npm run format
```

### Playwright
Browser automation tool for E2E testing. Supports Chrome, Firefox, Safari.

```bash
npm run test:e2e
```

---

## Design Patterns

### MVC (Model-View-Controller)
Architectural pattern separating data (Model), presentation (View), and logic (Controller).

```
Request → Route → Controller → Model (Store) → View → Response
```

### Repository Pattern / Store Pattern
Abstraction layer for data access. Allows swapping implementations (mem/json/mongo) without changing business logic.

```javascript
// Same interface, different implementations
const user = await db.userStore.getUserById(id);  // Could be mem, json, or mongo
```

### Factory Pattern
Creating objects based on configuration.

```javascript
db.init("mongo");  // Returns MongoDB stores
db.init("json");   // Returns JSON stores
```

---

## Environment & Configuration

### Environment Variable
Configuration value stored outside code. Set by OS or hosting platform.

```javascript
const port = process.env.PORT || 3000;
```

### `.env`
File containing environment variables for local development. **Never commit to git**.

```env
cookie_password=your_secret_here
db=mongodb://localhost:27017/myapp
```

### `.env.example`
Template showing required environment variables with placeholder values. **Committed to git**.

```env
cookie_password=CHANGE_ME
db=mongodb://localhost:27017/myapp
```

### `dotenv`
Library that loads variables from `.env` file into `process.env`.

```javascript
import "dotenv/config";
console.log(process.env.PORT);
```

---

## Generic Terms (Domain Abstraction)

These terms are used generically in the skills to apply across domains:

### User
Person with an account. Has credentials (email, password).

**Maps to**: Customer, Member, Account, Patient, etc.

### Collection
Container grouping related items. Belongs to a user.

**Maps to**:
- Playlist (music)
- Order (e-commerce)
- Project (task management)
- Category (blog)
- Album (photos)

### Item
Individual element within a collection.

**Maps to**:
- Track (music)
- Product (e-commerce)
- Task (task management)
- Post (blog)
- Photo (photos)

### Domain-Specific Fields
Fields beyond the generic structure that depend on your application domain.

```javascript
// Generic
const ItemSpec = Joi.object({
  title: Joi.string().required(),
  collectionid: Joi.string().optional(),
  // Add domain-specific fields below:
});

// Music domain
artist: Joi.string(),
duration: Joi.number(),

// E-commerce domain
price: Joi.number(),
quantity: Joi.number(),
```

---

## Acronyms Quick Reference

| Acronym | Full Term | Category |
|---------|-----------|----------|
| **API** | Application Programming Interface | Web |
| **BDD** | Behavior-Driven Development | Testing |
| **CJS** | CommonJS | JavaScript |
| **CORS** | Cross-Origin Resource Sharing | Security |
| **CRUD** | Create, Read, Update, Delete | Database |
| **E2E** | End-to-End | Testing |
| **ESM** | ECMAScript Modules | JavaScript |
| **HTTP** | Hypertext Transfer Protocol | Web |
| **JSON** | JavaScript Object Notation | Data |
| **JWT** | JSON Web Token | Security |
| **MVC** | Model-View-Controller | Architecture |
| **ODM** | Object-Document Mapping | Database |
| **ORM** | Object-Relational Mapping | Database |
| **REST** | Representational State Transfer | API |
| **SSR** | Server-Side Rendering | Web |
| **TDD** | Test-Driven Development | Testing |
| **UI** | User Interface | Web |
| **URL** | Uniform Resource Locator | Web |

---

## See Also

- **README.md** - Skill overview and when to use each skill
- **hapi-web/** - Web development fundamentals
- **hapi-api/** - API development and documentation
- **persistence/** - Database and data management
- **testing/** - Testing strategies and tools
- **styling/** - Frontend styling with CSS frameworks
