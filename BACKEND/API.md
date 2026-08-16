# 01 — APIs Deep Dive

## REST, GraphQL, gRPC — Master-Level Understanding

---

### What is an API?

An **API (Application Programming Interface)** defines how software components communicate. In backend development, APIs are the contract between client and server.

```
Client (Browser/Mobile)         Server (Backend)
     │                              │
     │  HTTP Request                │
     │  GET /api/users/123          │
     │  Authorization: Bearer xxx   │
     │─────────────────────────────>│
     │                              │  1. Authenticate
     │                              │  2. Validate input
     │                              │  3. Business logic
     │                              │  4. Query database
     │                              │  5. Format response
     │  HTTP Response               │
     │  200 OK                      │
     │  { "id": 123, "name": "..." }│
     │<─────────────────────────────│
```

---

### REST (Representational State Transfer)

The most common API architecture. Uses HTTP methods to operate on resources.

#### REST Principles

| Principle | Description | Why It Matters |
|-----------|-------------|----------------|
| **Stateless** | Each request contains all info needed | Scalability—any server can handle any request |
| **Client-Server** | Separation of concerns | Independent evolution |
| **Cacheable** | Responses can be cached | Performance |
| **Uniform Interface** | Standard HTTP methods and URLs | Predictability |
| **Layered System** | Client doesn't know about intermediaries | Flexibility |

#### HTTP Methods Deep Dive

```
GET     /users          → Get all users (idempotent, safe)
GET     /users/123      → Get user 123 (idempotent, safe)
POST    /users          → Create new user (not idempotent)
PUT     /users/123      → Replace user 123 entirely (idempotent)
PATCH   /users/123      → Partial update user 123 (not idempotent)
DELETE  /users/123      → Delete user 123 (idempotent)
HEAD    /users/123      → Get headers only (no body)
OPTIONS /users          → Get supported methods (CORS)
```

| Method | Purpose | Idempotent | Safe | Request Body | Response Body |
|--------|---------|------------|------|--------------|---------------|
| GET | Read | ✓ | ✓ | No | Yes |
| POST | Create | ✗ | ✗ | Yes | Yes |
| PUT | Replace | ✓ | ✗ | Yes | Yes |
| PATCH | Partial update | ✗ | ✗ | Yes | Yes |
| DELETE | Remove | ✓ | ✗ | Optional | Optional |

**Idempotent:** Same request multiple times = same result.
**Safe:** Doesn't modify the resource.

#### REST Status Codes (Complete)

```javascript
// 2xx Success
200 OK                    // GET, PUT, PATCH successful
201 Created               // POST successful, resource created
202 Accepted              // Request accepted for async processing
204 No Content            // DELETE successful, no body

// 3xx Redirection
301 Moved Permanently     // Resource permanently moved
302 Found                 // Temporary redirect
304 Not Modified          // Cache is still valid

// 4xx Client Error
400 Bad Request           // Invalid input, validation failed
401 Unauthorized          // Not authenticated (no token/invalid token)
403 Forbidden             // Not authorized (insufficient permissions)
404 Not Found             // Resource doesn't exist
405 Method Not Allowed    // HTTP method not supported
409 Conflict              // Duplicate resource, state conflict
422 Unprocessable Entity  // Validation failed (semantic errors)
429 Too Many Requests     // Rate limited

// 5xx Server Error
500 Internal Server Error // Generic server error
502 Bad Gateway           // Upstream service failed
503 Service Unavailable   // Server overloaded/maintenance
504 Gateway Timeout       // Upstream service timeout
```

#### REST Implementation (Express.js)

```javascript
const express = require('express');
const router = express.Router();

// GET /api/users - List all users with pagination
router.get('/', async (req, res, next) => {
    try {
        const { page = 1, limit = 10, sort = '-createdAt' } = req.query;
        
        const users = await User.find()
            .sort(sort)
            .skip((page - 1) * limit)
            .limit(parseInt(limit));
        
        const total = await User.countDocuments();
        
        res.json({
            data: users,
            pagination: {
                page: parseInt(page),
                limit: parseInt(limit),
                total,
                pages: Math.ceil(total / limit)
            }
        });
    } catch (error) {
        next(error);
    }
});

// GET /api/users/:id - Get single user
router.get('/:id', async (req, res, next) => {
    try {
        const user = await User.findById(req.params.id);
        if (!user) {
            return res.status(404).json({ error: 'User not found' });
        }
        res.json({ data: user });
    } catch (error) {
        next(error);
    }
});

// POST /api/users - Create user
router.post('/', validateUser, async (req, res, next) => {
    try {
        const user = await User.create(req.body);
        res.status(201).json({ data: user });
    } catch (error) {
        if (error.code === 11000) {
            return res.status(409).json({ error: 'Email already exists' });
        }
        next(error);
    }
});

// PUT /api/users/:id - Replace user
router.put('/:id', validateUser, async (req, res, next) => {
    try {
        const user = await User.findByIdAndUpdate(
            req.params.id,
            req.body,
            { new: true, runValidators: true }
        );
        if (!user) {
            return res.status(404).json({ error: 'User not found' });
        }
        res.json({ data: user });
    } catch (error) {
        next(error);
    }
});

// PATCH /api/users/:id - Partial update
router.patch('/:id', async (req, res, next) => {
    try {
        const user = await User.findByIdAndUpdate(
            req.params.id,
            { $set: req.body },
            { new: true, runValidators: true }
        );
        if (!user) {
            return res.status(404).json({ error: 'User not found' });
        }
        res.json({ data: user });
    } catch (error) {
        next(error);
    }
});

// DELETE /api/users/:id - Delete user
router.delete('/:id', async (req, res, next) => {
    try {
        const user = await User.findByIdAndDelete(req.params.id);
        if (!user) {
            return res.status(404).json({ error: 'User not found' });
        }
        res.status(204).send();
    } catch (error) {
        next(error);
    }
});
```

---

### GraphQL

A query language for APIs where the client specifies exactly what data it needs.

#### REST vs GraphQL

```
REST Problem: Over-fetching and Under-fetching

GET /users/123
→ Returns ALL user fields (over-fetching)

GET /users/123
GET /users/123/posts
GET /users/123/followers
→ 3 requests for related data (under-fetching)

GraphQL Solution:

POST /graphql
{
    user(id: 123) {
        name
        email
        posts(limit: 5) { title }
        followersCount
    }
}
→ 1 request, exact data needed
```

| Aspect | REST | GraphQL |
|--------|------|---------|
| Endpoints | Multiple | Single |
| Data fetching | Fixed structure | Client specifies |
| Over-fetching | Common | Eliminated |
| Under-fetching | Common | Eliminated |
| Caching | Easy (HTTP) | Complex |
| File upload | Native | Workaround |
| Real-time | WebSockets | Subscriptions |
| Learning curve | Lower | Higher |
| Error handling | HTTP status codes | Always 200 |

#### GraphQL Schema & Resolvers

```javascript
const { ApolloServer, gql } = require('apollo-server');

// Schema definition
const typeDefs = gql`
    type User {
        id: ID!
        name: String!
        email: String!
        posts: [Post!]!
        followers: [User!]!
    }
    
    type Post {
        id: ID!
        title: String!
        content: String!
        author: User!
        createdAt: String!
    }
    
    type Query {
        user(id: ID!): User
        users(limit: Int, offset: Int): [User!]!
        post(id: ID!): Post
    }
    
    type Mutation {
        createUser(input: CreateUserInput!): User!
        updateUser(id: ID!, input: UpdateUserInput!): User!
        deleteUser(id: ID!): Boolean!
    }
    
    input CreateUserInput {
        name: String!
        email: String!
    }
    
    input UpdateUserInput {
        name: String
        email: String
    }
`;

// Resolvers
const resolvers = {
    Query: {
        user: async (_, { id }) => {
            return await User.findById(id);
        },
        users: async (_, { limit = 10, offset = 0 }) => {
            return await User.find().skip(offset).limit(limit);
        },
    },
    User: {
        posts: async (parent) => {
            return await Post.find({ authorId: parent.id });
        },
        followers: async (parent) => {
            return await User.find({ following: parent.id });
        },
    },
    Mutation: {
        createUser: async (_, { input }) => {
            return await User.create(input);
        },
    },
};

const server = new ApolloServer({ typeDefs, resolvers });
server.listen({ port: 4000 });
```

#### N+1 Problem & DataLoader

```javascript
const DataLoader = require('dataloader');

// Batch function
const postsLoader = new DataLoader(async (userIds) => {
    const posts = await Post.find({ authorId: { $in: userIds } });
    return userIds.map(id => posts.filter(p => p.authorId.toString() === id));
});

const resolvers = {
    User: {
        posts: async (parent) => {
            return postsLoader.load(parent.id.toString());
        },
    },
};
```

---

### gRPC

High-performance RPC framework by Google using Protocol Buffers.

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
| Format | JSON (text) | Protocol Buffers (binary) |
| Speed | Slower | 2-10x faster |
| Streaming | Limited | Full bidirectional |
| Browser support | Native | Needs proxy |
| Code generation | Optional | Built-in |
| Contract | OpenAPI/Swagger | .proto file |

#### Protocol Buffers

```protobuf
// user.proto
syntax = "proto3";
package users;

service UserService {
    rpc GetUser (GetUserRequest) returns (User);
    rpc ListUsers (ListUsersRequest) returns (ListUsersResponse);
    rpc CreateUser (CreateUserRequest) returns (User);
    rpc UpdateUser (UpdateUserRequest) returns (User);
    rpc DeleteUser (DeleteUserRequest) returns (google.protobuf.Empty);
    rpc StreamUsers (StreamUsersRequest) returns (stream User);  // Server streaming
}

message GetUserRequest {
    int32 id = 1;
}

message User {
    int32 id = 1;
    string name = 2;
    string email = 3;
    int64 created_at = 4;
}

message ListUsersRequest {
    int32 limit = 1;
    int32 offset = 2;
}

message ListUsersResponse {
    repeated User users = 1;
    int32 total = 2;
}
```

---

### API Design Patterns

#### HATEOAS (Hyperlinks)

```json
{
    "data": {
        "id": 123,
        "name": "Alice",
        "links": {
            "self": "/api/users/123",
            "posts": "/api/users/123/posts",
            "followers": "/api/users/123/followers"
        }
    }
}
```

#### Bulk Operations

```javascript
// POST /api/users/bulk
{
    "operations": [
        { "method": "POST", "body": { "name": "Alice" } },
        { "method": "POST", "body": { "name": "Bob" } }
    ]
}

// Response
{
    "results": [
        { "status": 201, "data": { "id": 1, "name": "Alice" } },
        { "status": 201, "data": { "id": 2, "name": "Bob" } }
    ]
}
```

#### Long-Running Operations

```javascript
// POST /api/reports/generate → 202 Accepted
{
    "id": "job-123",
    "status": "processing",
    "links": {
        "self": "/api/jobs/job-123",
        "cancel": "/api/jobs/job-123/cancel"
    }
}

// GET /api/jobs/job-123
{
    "id": "job-123",
    "status": "completed",
    "result": { "downloadUrl": "/api/reports/report-123.pdf" }
}
```

---

### Interview Questions

**Q: What is REST and what are its principles?**

A: "REST is an architectural style for APIs using HTTP methods. Key principles: stateless (no server sessions), client-server separation, cacheable responses, uniform interface (standard methods/URLs), layered system. Resources are nouns (/users), methods are verbs (GET, POST)."

**Q: What's the difference between PUT and PATCH?**

A: "PUT replaces the entire resource—you must send all fields. PATCH updates only specified fields. PUT is idempotent (same result if repeated); PATCH is not. PUT /users/123 with {name: 'Alice'} would clear email/age. PATCH only changes name."

**Q: When would you choose GraphQL over REST?**

A: "When clients need flexible data fetching (mobile vs web need different fields), when you have deeply nested related data (avoids N+1 requests), when you want to avoid versioning (schema evolution). Don't use for simple CRUD, file uploads, or when HTTP caching is critical."

**Q: When would you use gRPC over REST?**

A: "For internal microservice communication where performance matters. gRPC uses HTTP/2 and binary Protocol Buffers (2-10x faster). Supports bidirectional streaming. Not browser-friendly—use REST for public APIs, gRPC for internal services."

**Q: What's the N+1 problem in GraphQL?**

A: "Fetching a list with related data triggers N+1 queries. 1 query for users + N queries for each user's posts = N+1 total. Solution: DataLoader batches requests and caches results."

---

*Next: [02 — API Design Best Practices](02-API-Design.md)*
