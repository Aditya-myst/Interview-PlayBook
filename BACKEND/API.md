# 01 — APIs Deep Dive

## REST, GraphQL, gRPC — The Foundation

---

### What is an API?

An **API (Application Programming Interface)** defines how software components communicate. In backend development, APIs are the contract between client and server.

```
Client (Browser/App)          Server (Backend)
     │                              │
     │  HTTP Request (GET /users)   │
     │─────────────────────────────>│
     │                              │  Process request
     │  HTTP Response (200 OK)      │  Query database
     │<─────────────────────────────│
     │  { "users": [...] }          │
```

---

### REST (Representational State Transfer)

The most common API architecture. Uses HTTP methods to operate on resources.

#### REST Principles   

| Principle | Description |
|-----------|-------------|
| **Stateless** | Each request contains all info needed (no server sessions) |
| **Client-Server** | Separation of concerns |
| **Cacheable** | Responses can be cached |
| **Uniform Interface** | Standard HTTP methods and URLs |
| **Layered System** | Client doesn't know if talking to end server or intermediary |

#### HTTP Methods

```
GET     /users          → Get all users
GET     /users/123      → Get user 123
POST    /users          → Create new user
PUT     /users/123      → Update user 123 (full)
PATCH   /users/123      → Update user 123 (partial)
DELETE  /users/123      → Delete user 123
```

| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| GET | Read resource | ✓ | ✓ |
| POST | Create resource | ✗ | ✗ |
| PUT | Replace resource | ✓ | ✗ |
| PATCH | Partial update | ✗ | ✗ |
| DELETE | Remove resource | ✓ | ✗ |

**Idempotent:** Same request multiple times = same result.
**Safe:** Doesn't modify the resource.

#### REST Example (Express.js)

```javascript
// GET /users
app.get('/users', async (req, res) => {
    const users = await db.query('SELECT * FROM users');
    res.json({ data: users });
});

// GET /users/:id
app.get('/users/:id', async (req, res) => {
    const user = await db.query('SELECT * FROM users WHERE id = ?', [req.params.id]);
    if (!user) return res.status(404).json({ error: 'User not found' });
    res.json({ data: user });
});

// POST /users
app.post('/users', async (req, res) => {
    const { name, email } = req.body;
    const user = await db.query('INSERT INTO users (name, email) VALUES (?, ?)', [name, email]);
    res.status(201).json({ data: user });
});

// PUT /users/:id
app.put('/users/:id', async (req, res) => {
    const { name, email } = req.body;
    await db.query('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, req.params.id]);
    res.json({ data: { id: req.params.id, name, email } });
});

// DELETE /users/:id
app.delete('/users/:id', async (req, res) => {
    await db.query('DELETE FROM users WHERE id = ?', [req.params.id]);
    res.status(204).send();
});
```

#### REST Status Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Not authenticated |
| 403 | Forbidden | Not authorized |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate resource |
| 422 | Unprocessable | Validation failed |
| 429 | Too Many Requests | Rate limited |
| 500 | Internal Server Error | Server crash |

---

### GraphQL

A query language for APIs where the client specifies exactly what data it needs.

#### REST vs GraphQL

```
REST: Multiple endpoints, fixed response
GET /users/123                    → Full user object
GET /users/123/posts              → User's posts
GET /users/123/followers          → User's followers
= 3 requests for related data

GraphQL: Single endpoint, flexible query
POST /graphql
{
    user(id: 123) {
        name
        posts { title }
        followers { name }
    }
}
= 1 request for exactly what you need
```

| Aspect | REST | GraphQL |
|--------|------|---------|
| Endpoints | Multiple | Single |
| Data fetching | Fixed structure | Client specifies |
| Over-fetching | Common | Eliminated |
| Under-fetching | Common (multiple requests) | Eliminated |
| Caching | Easy (HTTP caching) | Complex |
| File upload | Native | Workaround needed |
| Learning curve | Lower | Higher |

#### GraphQL Example

```graphql
# Schema
type User {
    id: ID!
    name: String!
    email: String!
    posts: [Post!]!
}

type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
}

type Query {
    user(id: ID!): User  
    users(limit: Int, offset: Int): [User!]!
    post(id: ID!): Post
}

type Mutation {
    createUser(name: String!, email: String!): User!
    updateUser(id: ID!, name: String, email: String): User!
    deleteUser(id: ID!): Boolean!
}
```

```javascript
// Resolver
const resolvers = {
    Query: {
        user: (_, { id }) => users.find(u => u.id === id),
        users: (_, { limit = 10, offset = 0 }) => users.slice(offset, offset + limit),
    },
    User: {
        posts: (parent) => posts.filter(p => p.authorId === parent.id),
    },
    Mutation: {
        createUser: (_, { name, email }) => {
            const user = { id: generateId(), name, email };
            users.push(user);
            return user;
        },
    },
};
```

#### N+1 Problem in GraphQL

```graphql
# This query triggers N+1 database queries:
{
    users {
        name
        posts { title }  # For each user, fetch posts separately
    }
}
# 1 query for users + N queries for each user's posts = N+1 queries!
```

**Solution: DataLoader (batching & caching)**

```javascript
const DataLoader = require('dataloader');

const postsLoader = new DataLoader(async (userIds) => {
    const posts = await db.query('SELECT * FROM posts WHERE authorId IN (?)', [userIds]);
    return userIds.map(id => posts.filter(p => p.authorId === id));
});

const resolvers = {
    User: {
        posts: (parent) => postsLoader.load(parent.id),  // Batched!
    },
};
```

---

### gRPC

A high-performance RPC framework by Google, using Protocol Buffers.

#### When to Use gRPC

| Use Case | Best Choice |
|----------|-------------|
| Browser clients | REST or GraphQL |
| Mobile apps | REST or GraphQL |
| Microservice-to-microservice | gRPC |
| Real-time streaming | gRPC |
| High performance needed | gRPC |
| Public API | REST |

#### gRPC vs REST

| Aspect | REST | gRPC |
|--------|------|------|
| Protocol | HTTP/1.1 | HTTP/2 |
| Format | JSON | Protocol Buffers (binary) |
| Speed | Slower | 2-10x faster |
| Streaming | Limited | Full bidirectional |
| Browser support | Native | Needs proxy |
| Code generation | Optional | Built-in |

#### Protocol Buffers (Protobuf)

```protobuf
// user.proto
syntax = "proto3";

service UserService {
    rpc GetUser (GetUserRequest) returns (User);
    rpc ListUsers (ListUsersRequest) returns (stream User);  // Server streaming
    rpc CreateChat (stream Message) returns (ChatResponse);  // Client streaming
}

message GetUserRequest {
    int32 id = 1;
}

message User {
    int32 id = 1;
    string name = 2;
    string email = 3;
}

message ListUsersRequest {
    int32 limit = 1;
    int32 offset = 2;
}

message Message {
    string content = 1;
    string sender = 2;
}
```

#### gRPC Service Implementation (Node.js)

```javascript
const grpc = require('@grpc/grpc-js');
const protoLoader = require('@grpc/proto-loader');

const packageDef = protoLoader.loadSync('user.proto');
const proto = grpc.loadPackageDefinition(packageDef);

const server = new grpc.Server();

server.addService(proto.UserService.service, {
    GetUser: (call, callback) => {
        const user = users.find(u => u.id === call.request.id);
        callback(null, user);
    },
    
    ListUsers: (call) => {
        const { limit = 10, offset = 0 } = call.request;
        users.slice(offset, offset + limit).forEach(user => {
            call.write(user);
        });
        call.end();
    },
});

server.bindAsync('0.0.0.0:50051', grpc.ServerCredentials.createInsecure(), () => {
    console.log('gRPC server running on port 50051');
});
```

---

### REST vs GraphQL vs gRPC — When to Use What

| Scenario | Best Choice | Why |
|----------|-------------|-----|
| Public web API | REST | Simple, well-understood, cacheable |
| Mobile app with varying data needs | GraphQL | Client specifies exact data |
| Internal microservice communication | gRPC | Fast, typed, streaming |
| Simple CRUD API | REST | Overkill for GraphQL/gRPC |
| Real-time data streaming | gRPC | Bidirectional streaming |
| Browser real-time | GraphQL Subscriptions or WebSocket | Browser support |

---

### Interview Questions

**Q: What is REST?**

A: "REST is an architectural style for APIs using HTTP methods (GET, POST, PUT, DELETE) to operate on resources identified by URLs. Key principles: stateless, uniform interface, cacheable, client-server separation. Resources are nouns (/users, /posts), methods are verbs (GET, POST)."

**Q: What's the difference between PUT and PATCH?**

A: "PUT replaces the entire resource—you must send all fields. PATCH updates only the specified fields. PUT /users/123 with {name: 'Alice'} would set name to Alice and clear all other fields. PATCH /users/123 with {name: 'Alice'} only changes the name."

**Q: When would you use GraphQL over REST?**

A: "When clients need flexible data fetching—mobile apps that need different fields than web, or pages that need related data from multiple resources. GraphQL eliminates over-fetching (getting unused data) and under-fetching (needing multiple requests). The trade-off: more complex server implementation, harder caching."

**Q: What's the N+1 problem in GraphQL?**

A: "When fetching a list with related data, GraphQL resolvers might query the database once for the list, then once for each item's related data. For 100 users with posts: 1 query for users + 100 queries for posts = 101 queries. Solution: DataLoader batches and caches queries."

**Q: When would you use gRPC over REST?**

A: "For internal microservice communication where performance matters. gRPC uses HTTP/2 and binary Protocol Buffers—2-10x faster than JSON over HTTP/1.1. It supports bidirectional streaming and has built-in code generation. The trade-off: not browser-friendly, harder to debug."

---

*Next: [02 — API Design & Best Practices](02-API-Design.md)*
