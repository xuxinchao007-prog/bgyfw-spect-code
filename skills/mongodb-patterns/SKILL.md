---
name: mongodb-patterns
description: MongoDB database patterns for query optimization, data modeling, indexing, and security. Based on official MongoDB documentation.
---

# MongoDB Patterns

Quick reference for MongoDB best practices. For detailed guidance, refer to official MongoDB documentation at https://www.mongodb.com/docs/

## When to Activate

- Designing MongoDB schemas and data models
- Writing aggregation pipelines or queries
- Optimizing query performance with indexes
- Implementing security and authentication
- Troubleshooting slow operations
- Planning sharding strategies

## Quick Reference

### Index Cheat Sheet

| Query Pattern | Index Type | Example |
|--------------|------------|---------|
| `{ field: value }` | Single Field | `db.collection.createIndex({ score: 1 })` |
| `{ a: x, b: y }` | Compound | `db.collection.createIndex({ userid: 1, score: -1 })` |
| `{ field: { $in: [...] } }` | Multikey (automatic) | Index on array field |
| `{ $text: { $search: "..." } }` | Text | `db.collection.createIndex({ content: "text" })` |
| `{ location: { $near: [...] } }` | 2dsphere | `db.collection.createIndex({ loc: "2dsphere" })` |
| Hashed sharding key | Hashed | `db.collection.createIndex({ _id: "hashed" })` |

**Index Best Practices:**
- Limit collections to maximum 50 indexes
- Place equality columns first, then range columns in compound indexes
- Use covered queries to avoid document lookups
- Monitor index usage with `$indexStats`

### Data Modeling Quick Reference

| Relationship Type | Recommendation | Cardinality |
|------------------|----------------|-------------|
| One-to-Few | Embed | ≤100 items |
| One-to-Many | Embed or Reference | 100-1000 items |
| One-to-Squillions | Reference | Unlimited |
| Frequent access together | Embed | Any |
| Independent access | Reference | Any |

**Core Principle:** Data that is accessed together should be stored together

### Common Patterns

**Compound Index Order:**
```javascript
// Equality columns first, then range columns
db.orders.createIndex({ status: 1, createdAt: -1 })

// Works for: { status: "pending", createdAt: { $gt: date } }
// Also works for: { status: "pending" } (leftmost prefix)
```

**Covering Query:**
```javascript
// Index covers all fields in query and projection
db.users.createIndex({ email: 1, name: 1, createdAt: 1 })

// Covered query - no document lookup needed
db.users.find(
  { email: "user@example.com" },
  { name: 1, createdAt: 1, _id: 0 }
)
```

**Embedding Pattern:**
```javascript
// Good: One-to-few relationship with frequent access together
{
  _id: 1,
  name: "John Doe",
  address: {
    street: "123 Main St",
    city: "New York",
    zip: "10001"
  },
  phoneNumbers: ["212-555-0153", "646-555-0199"]
}
```

**Referencing Pattern:**
```javascript
// Good: One-to-many with independent access
// Order document
{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  customerId: ObjectId("507f191e810c19729de860ea"),
  total: 100,
  status: "completed"
}

// Customer document (separate collection)
{
  _id: ObjectId("507f191e810c19729de860ea"),
  name: "John Doe",
  email: "john@example.com"
}
```

**Aggregation Pipeline:**
```javascript
// Stage order matters: $match early to filter documents
db.orders.aggregate([
  { $match: { status: "active" } },      // Filter first
  { $group: {                            // Then aggregate
    _id: "$category",
    total: { $sum: "$amount" }
  }},
  { $sort: { total: -1 } },              // Sort results
  { $limit: 10 }                         // Limit output
])
```

**Pagination (Cursor-based):**
```javascript
// Efficient pagination using _id
db.products.find({ _id: { $gt: lastId } })
  .sort({ _id: 1 })
  .limit(20)

// Compound key pagination
db.orders.find({
  _id: { $gt: lastId },
  createdAt: { $gt: lastDate }
})
.sort({ createdAt: 1, _id: 1 })
.limit(20)
```

**Time-Series Bucket Pattern:**
```javascript
// For time-series data, group measurements into buckets
{
  _id: ObjectId("..."),
  sensorId: 123,
  date: "2024-01-15",
  measurements: [
    { timestamp: ISODate("2024-01-15T10:00:00"), value: 25.3 },
    { timestamp: ISODate("2024-01-15T10:05:00"), value: 25.5 }
  ]
}
```

### Performance Monitoring

**Explain Plan Analysis:**
```javascript
// Basic explain
db.collection.find({ quantity: { $gte: 100 } }).explain()

// Execution stats
db.collection.find({ quantity: { $gte: 100 } }).explain("executionStats")

// Key metrics:
// - stage: "IXSCAN" (good) vs "COLLSCAN" (bad)
// - totalDocsExamined: Should equal nReturned
// - totalKeysExamined: Should equal nReturned
// - executionTimeMillis: Query execution time
```

**Profiling:**
```javascript
// Enable profiling (0=off, 1=slow only, 2=all)
db.setProfilingLevel(1, { slowms: 100 })

// View slow operations
db.system.profile.find().sort({ millis: -1 }).limit(10)

// Check index usage
db.collection.aggregate([{
  $indexStats: {}
}])
```

**Current Operations:**
```javascript
// View running operations
db.currentOp({
  "op": { "$ne": "command" },
  "active": true
})

// Kill long-running operation
db.killOp(opId)
```

### Anti-Pattern Detection

**Find Missing Indexes (COLLSCAN):**
```javascript
// Find collection scans (no index used)
db.system.profile.find({
  "command.query": { $exists: true },
  "stage": "COLLSCAN"
})

// Check with explain
db.collection.find({ field: "value" }).explain("executionStats")
// Look for stage: "COLLSCAN" in executionStages
```

**Find Large Documents:**
```javascript
// Find documents approaching 16MB limit
db.collection.aggregate([
  {
    $project: {
      name: 1,
      size: { $strLenCP: { $toString: "$$ROOT" } }
    }
  },
  { $sort: { size: -1 } },
  { $limit: 10 }
])

// Get collection stats
db.collection.stats()
```

**Find Unused Indexes:**
```javascript
// Check index usage statistics
db.collection.aggregate([{
  $indexStats: {}
}])
// Look for indexes with low usage.count
```

**Find Inefficient Queries:**
```javascript
// Queries examining many documents but returning few
db.system.profile.find({
  "docsExamined": { $gt: 1000 },
  "nreturned": { $lt: 10 }
}).sort({ "docsExamined": -1 })
```

**Find N+1 Problems:**
```javascript
// Multiple $lookup operations indicate potential issue
db.system.profile.find({
  "command.pipeline": {
    $elemMatch: { "$lookup": { $exists: true } }
  }
})
```

### Configuration Template

```javascript
// mongod.conf configuration

// Storage
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true
  wiredTiger:
    engineConfig:
      cacheSizeGB: 4
    collectionConfig:
      blockCompressor: snappy

// Network
net:
  port: 27017
  bindIp: 127.0.0.1

// Security
security:
  authorization: enabled
  javascriptEnabled: false

// Operation Profiling
operationProfiling:
  mode: slowOp
  slowOpThresholdMs: 100
  slowOpSampleRate: 1.0

// Logging
systemLog:
  destination: file
  path: /var/log/mongodb/mongod.log
  logAppend: true
```

### Security Best Practices

**Enable Authentication:**
```javascript
// Enable in configuration or command line
security:
  authorization: enabled

// Create admin user
use admin
db.createUser({
  user: "admin",
  pwd: passwordPrompt(),
  roles: [ { role: "root", db: "admin" } ]
})
```

**Role-Based Access Control:**
```javascript
// Create roles
db.createRole({
  role: "readWriteOwnData",
  privileges: [{
    resource: { db: "app", collection: "" },
    actions: ["find", "insert", "update", "remove"]
  }],
  roles: []
})

// Create user with role
db.createUser({
  user: "appUser",
  pwd: passwordPrompt(),
  roles: [ { role: "readWrite", db: "appDatabase" } ]
})
```

**Field-Level Security:**
```javascript
// Schema validation with encryption (MongoDB 4.2+)
db.runCommand({
  collMod: "users",
  validator: {
    $jsonSchema: {
      bsonType: "object",
      properties: {
        ssn: {
          encrypt: {
            keyId: [UUID("...")],
            algorithm: "AEAD_AES_256_CBC_HMAC_SHA_512-Random"
          }
        }
      }
    }
  }
})
```

**Network Security:**
```javascript
// Enable TLS/SSL
net:
  ssl:
    mode: requireSSL
    PEMKeyFile: /path/to/pem
    CAFile: /path/to/ca

// Restrict bindIp
net:
  bindIp: 127.0.0.1,private_ip
```

## Related

- Official MongoDB Documentation: https://www.mongodb.com/docs/
- MongoDB University: https://university.mongodb.com/
- Skill: `postgres-patterns` - PostgreSQL patterns
- Skill: `mysql-patterns` - MySQL patterns
- Skill: `backend-patterns` - API and backend patterns

---

*Based on [MongoDB Reference Manual](https://www.mongodb.com/docs/manual/)*
