---
name: persistence-mongo
description: MongoDB with Mongoose ODM and pluggable store architecture
metadata:
  tags: mongodb, mongoose, odm, persistence, store-abstraction
---

# Persistence with MongoDB

## Dependencies

```json
"mongoose": "^9.3.0",
"@hapi/boom": "^10.0.1"
```

## Mongoose schemas

```javascript
const userSchema = new mongoose.Schema({
  firstName: { type: String, required: true },
  lastName: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
});

const collectionSchema = new mongoose.Schema({
  title: { type: String, required: true },
  userid: { type: mongoose.Schema.Types.ObjectId, ref: "User", required: true },
});

const itemSchema = new mongoose.Schema({
  title: { type: String, required: true },
  artist: { type: String, required: true },
  duration: { type: Number, default: 0 },
  collectionid: { type: mongoose.Schema.Types.ObjectId, ref: "Collection", required: true },
});

export const User = mongoose.model("User", userSchema);
export const Collection = mongoose.model("Collection", collectionSchema);
export const Item = mongoose.model("Item", itemSchema);
```

## Connection module

```javascript
// models/mongo/connection.js
export async function connect() {
  const uri = process.env.db || "mongodb://localhost:27017/playtime";
  await mongoose.connect(uri);
}

export async function disconnect() {
  await mongoose.disconnect();
}
```

## Mongo store pattern

Implement same interface as mem/json; return `undefined` (not `null`) for not-found:

```javascript
export const userMongoStore = {
  async getUserById(id) {
    if (!mongoose.Types.ObjectId.isValid(id)) return undefined;
    const user = await User.findById(id).lean();
    return user ?? undefined;
  },
  async addUser(user) {
    const newUser = new User(user);
    await newUser.save();
    return newUser.toObject();
  },
  // ... getAllUsers, getUserByEmail, deleteUserById, deleteAll
};
```

## ObjectId handling

- Validate: `mongoose.Types.ObjectId.isValid(id)` before queries
- Use `.lean()` for plain objects
- Return `undefined` for not-found to match mem/json

## db abstraction extension

```javascript
init(storeType) {
  this._storeType = storeType || "mem";
  if (this._storeType === "mongo") {
    this.userStore = userMongoStore;
    this.collectionStore = collectionMongoStore;
    this.itemStore = itemMongoStore;
  }
  // ... json, mem
},
async connect() {
  if (this._storeType === "mongo") await mongoConnection.connect();
},
async disconnect() {
  if (this._storeType === "mongo") await mongoConnection.disconnect();
}
```

## Environment

```
db=mongodb://localhost:27017/playtime
store=mongo
```
