# 01 — APIs Deep Dive

## REST, GraphQL, gRPC — Complete Guide with 30+ Interview Questions

---

### REST (Representational State Transfer)

#### Core Principles

| Principle | Description | Example |
|-----------|-------------|---------|
| **Stateless** | Each request contains all info needed | No server sessions |
| **Client-Server** | Separation of concerns | Frontend/backend independent |
| **Cacheable** | Responses can be cached | GET responses cacheable |
| **Uniform Interface** | Standard HTTP methods | GET, POST, PUT, DELETE |
| **Layered System** | Client doesn't know about intermediaries | Load balancer transparent |

#### HTTP Methods

```javascript
GET     /users          // Get all users (idempotent, safe)
GET     /users/123      // Get user 123 (idempotent, safe)
POST    /users          // Create user (not idempotent)
PUT     /users/123      // Replace user 123 (idempotent)
PATCH   /users/123      // Partial update (not idempotent)
DELETE  /users/123      // Delete user 123 (idempotent)
```

#### Status Codes

```javascript
// 2xx Success
200 OK                    // GET, PUT, PATCH successful
201 Created               // POST successful
204 No Content            // DELETE successful

// 4xx Client Error
400 Bad Request           // Invalid input
401 Unauthorized          // Not authenticated
403 Forbidden             // Not authorized
404 Not Found             // Resource doesn't exist
409 Conflict              // Duplicate resource
422 Unprocessable Entity  // Validation failed
429 Too Many Requests     // Rate limited

// 5xx Server Error
500 Internal Server Error // Server crash
502 Bad Gateway           // Upstream service failed
503 Service Unavailable   // Server overloaded
```

---

### GraphQL

#### Schema Definition

```graphql
type User {
    id: ID!
    name: String!
    email: String!
    posts: [Post!]!
}

type Post {
    id: ID!
    title: String!
    author: User!
}

type Query {
    user(id: ID!): User
    users(limit: Int): [User!]!
}

type Mutation {
    createUser(name: String!, email: String!): User!
}
```

#### Resolver Implementation

```javascript
const resolvers = {
    Query: {
        user: async (_, { id }) => await User.findById(id),
        users: async (_, { limit = 10 }) => await User.find().limit(limit),
    },
    User: {
        posts: async (parent) => await Post.find({ authorId: parent.id }),
    },
};
```

---

### gRPC

#### Protocol Buffer Definition

```protobuf
service UserService {
    rpc GetUser (GetUserRequest) returns (User);
    rpc ListUsers (ListUsersRequest) returns (stream User);
}

message GetUserRequest {
    int32 id = 1;
}

message User {
    int32 id = 1;
    string name = 2;
    string email = 3;
}
```

---

### Interview Questions (30+)

#### Conceptual Questions

**Q1: What is REST?**
A: "REST is an architectural style for APIs using HTTP methods to operate on resources. Key principles: stateless, client-server separation, cacheable, uniform interface. Resources are nouns (/users), methods are verbs (GET, POST)."

**Q2: What's the difference between PUT and PATCH?**
A: "PUT replaces the entire resource—you must send all fields. PATCH updates only specified fields. PUT is idempotent (same result if repeated); PATCH is not. Example: PUT /users/123 with {name: 'Alice'} would clear email. PATCH only changes name."

**Q3: What does idempotent mean?**
A: "Same request multiple times = same result. GET, PUT, DELETE are idempotent. POST is not (creates new resource each time). Important for retry logic—if request fails, can safely retry idempotent operations."

**Q4: What's the difference between 401 and 403?**
A: "401 Unauthorized: not authenticated—you don't know who I am. 403 Forbidden: not authenticated—I know who you are, but you don't have permission. 401 means 'log in'; 403 means 'you can't do this'."

**Q5: When would you use REST vs GraphQL?**
A: "REST: simple CRUD, public APIs, caching important, file uploads. GraphQL: complex data needs, mobile apps (different fields than web), multiple related resources in one request. REST is simpler; GraphQL eliminates over/under-fetching."

**Q6: When would you use gRPC over REST?**
A: "For internal microservice communication where performance matters. gRPC uses HTTP/2 and binary Protocol Buffers (2-10x faster). Supports bidirectional streaming. Not browser-friendly—use REST for public APIs, gRPC for internal services."

**Q7: What's the N+1 problem in GraphQL?**
A: "Fetching a list with related data triggers N+1 queries. 1 query for users + N queries for each user's posts = N+1 total. Solution: DataLoader batches requests and caches results."

**Q8: Explain the REST maturity model (Richardson).**
A: "Level 0: Single endpoint, HTTP as transport. Level 1: Multiple endpoints for resources. Level 2: HTTP methods (GET, POST, PUT, DELETE). Level 3: HATEOAS—hyperlinks in responses. Most APIs are Level 2."

**Q9: What is HATEOAS?**
A: "Hypermedia As The Engine Of Application State. Responses include links to related actions. Client discovers capabilities from responses, not hard-coded URLs. Makes API self-documenting."

**Q10: How do you handle file uploads in REST vs GraphQL?**
A: "REST: multipart/form-data with POST request. GraphQL: typically use a separate REST endpoint for uploads, or use libraries like graphql-upload. REST is simpler for file uploads."

#### Implementation Questions

**Q11: How would you design a REST API for a blog?**
A: "Resources: /users, /posts, /comments. Nested: /users/123/posts, /posts/456/comments. Methods: GET (list/get), POST (create), PUT (update), DELETE (delete). Pagination, filtering, sorting via query params."

**Q12: How do you handle API versioning?**
A: "URL path (/v1/users)—most common, explicit. Query param (?version=1)—clean URLs. Header (Accept: application/vnd.api.v1+json)—hidden. I'd use URL path for public APIs."

**Q13: How do you implement pagination?**
A: "Offset-based: ?limit=10&offset=20 (simple but slow for large offsets). Cursor-based: ?cursor=abc&limit=10 (consistent, efficient). Use cursor for infinite scroll, offset for admin dashboards."

**Q14: How do you handle errors in APIs?**
A: "Consistent format: { error: { code, message, details } }. Use appropriate HTTP status codes. Create custom error classes. Global error handler middleware. Never expose stack traces in production."

**Q15: How do you implement rate limiting?**
A: "Track requests per client (IP or API key) in Redis. Use sliding window or token bucket algorithm. Return 429 with Retry-After header. Different limits for different endpoints."

#### Advanced Questions

**Q16: What's the difference between authentication and authorization?**
A: "Authentication: 'Who are you?'—verify identity (JWT, OAuth). Authorization: 'What can you do?'—check permissions (RBAC, ABAC). Authentication happens first, then authorization."

**Q17: How do you handle CORS?**
A: "Cross-Origin Resource Sharing. Browsers block cross-domain requests. Use cors middleware with specific origins, methods, headers. Don't use wildcard (*) in production."

**Q18: What is API gateway pattern?**
A: "Single entry point for all API requests. Handles routing, authentication, rate limiting, logging, caching. Examples: Kong, AWS API Gateway, Express gateway. Simplifies client, centralizes cross-cutting concerns."

**Q19: How do you document APIs?**
A: "OpenAPI/Swagger for REST—auto-generate from code. GraphQL: schema is self-documenting. Tools: Swagger UI, Postman, GraphQL Playground. Include examples, error codes, authentication."

**Q20: What's the difference between REST and RPC?**
A: "REST: resource-oriented (nouns), HTTP methods (verbs), stateless. RPC: action-oriented (verbs), single endpoint, procedure calls. REST for CRUD, RPC for specific operations."

**Q21: How do you handle long-running operations?**
A: "Return 202 Accepted with job ID. Client polls status endpoint. Or use WebSockets/SSE for real-time updates. Or use webhooks for completion notification."

**Q22: What is API throttling vs rate limiting?**
A: "Rate limiting: cap on requests per time window. Throttling: slow down requests when limit approached. Rate limiting returns 429; throttling delays responses."

**Q23: How do you handle API backward compatibility?**
A: "Versioning (v1, v2). Add new fields without removing old. Deprecation headers. Sunset dates. Maintain old versions for 6-12 months."

**Q24: What's the difference between REST and WebSocket?**
A: "REST: request-response, stateless, HTTP. WebSocket: bidirectional, persistent connection, real-time. REST for CRUD; WebSocket for chat, live updates, gaming."

**Q25: How do you test APIs?**
A: "Unit tests for handlers. Integration tests with supertest. Contract tests (Pact). Load testing (k6, Artillery). Security testing (OWASP ZAP). Manual testing with Postman."

**Q26: What is API composition pattern?**
A: "Aggregate data from multiple services into single response. API gateway or dedicated composition layer. Reduces client-side complexity. Example: user profile + posts + followers in one response."

**Q27: How do you handle API pagination with large datasets?**
A: "Cursor-based pagination (efficient, consistent). Keyset pagination (WHERE id > last_id). Avoid offset for large datasets (slow). Use database indexes on cursor column."

**Q28: What's the difference between REST and GraphQL subscriptions?**
A: "REST: polling or WebSockets. GraphQL subscriptions: built-in real-time via WebSocket. GraphQL subscriptions are typed and part of the schema."

**Q29: How do you handle API idempotency?**
A: "Idempotency keys for POST requests. Client generates unique key, server deduplicates. Redis to track processed keys. Important for payment processing, retry logic."

**Q30: What is API chaining?**
A: "Client makes sequential requests, each using data from previous. Problem: multiple round trips. Solution: GraphQL (single request), BFF (Backend for Frontend), API composition."

---

### REST Implementation Patterns (Complete)

#### Express.js REST API (Production-Ready)

```javascript
const express = require('express');
const router = express.Router();
const { body, param, query, validationResult } = require('express-validator');

// Validation middleware
const validate = (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
        return res.status(400).json({ error: { code: 'VALIDATION_ERROR', details: errors.array() } });
    }
    next();
};

// GET /api/users - List with pagination, filtering, sorting
router.get('/', [
    query('page').optional().isInt({ min: 1 }),
    query('limit').optional().isInt({ min: 1, max: 100 }),
    query('sort').optional().isIn(['name', '-name', 'createdAt', '-createdAt']),
    query('search').optional().isString()
], validate, async (req, res, next) => {
    try {
        const { page = 1, limit = 10, sort = '-createdAt', search } = req.query;
        const filter = {};
        
        if (search) {
            filter.$or = [
                { name: { $regex: search, $options: 'i' } },
                { email: { $regex: search, $options: 'i' } }
            ];
        }
        
        const [users, total] = await Promise.all([
            User.find(filter).sort(sort).skip((page - 1) * limit).limit(parseInt(limit)).lean(),
            User.countDocuments(filter)
        ]);
        
        res.json({
            data: users,
            pagination: {
                page: parseInt(page),
                limit: parseInt(limit),
                total,
                pages: Math.ceil(total / limit),
                hasNext: page * limit < total,
                hasPrev: page > 1
            }
        });
    } catch (error) {
        next(error);
    }
});

// GET /api/users/:id - Get single resource
router.get('/:id', [
    param('id').isMongoId()
], validate, async (req, res, next) => {
    try {
        const user = await User.findById(req.params.id).lean();
        if (!user) {
            return res.status(404).json({ error: { code: 'NOT_FOUND', message: 'User not found' } });
        }
        res.json({ data: user });
    } catch (error) {
        next(error);
    }
});

// POST /api/users - Create resource
router.post('/', [
    body('name').trim().isLength({ min: 2, max: 50 }),
    body('email').isEmail().normalizeEmail(),
    body('age').optional().isInt({ min: 0, max: 150 })
], validate, async (req, res, next) => {
    try {
        const user = await User.create(req.body);
        res.status(201)
            .header('Location', `/api/users/${user._id}`)
            .json({ data: user });
    } catch (error) {
        if (error.code === 11000) {
            return res.status(409).json({ error: { code: 'CONFLICT', message: 'Email already exists' } });
        }
        next(error);
    }
});

// PUT /api/users/:id - Full update
router.put('/:id', [
    param('id').isMongoId(),
    body('name').trim().isLength({ min: 2, max: 50 }),
    body('email').isEmail().normalizeEmail()
], validate, async (req, res, next) => {
    try {
        const user = await User.findByIdAndUpdate(req.params.id, req.body, { new: true, runValidators: true });
        if (!user) {
            return res.status(404).json({ error: { code: 'NOT_FOUND', message: 'User not found' } });
        }
        res.json({ data: user });
    } catch (error) {
        next(error);
    }
});

// PATCH /api/users/:id - Partial update
router.patch('/:id', [
    param('id').isMongoId(),
    body('name').optional().trim().isLength({ min: 2, max: 50 }),
    body('email').optional().isEmail().normalizeEmail()
], validate, async (req, res, next) => {
    try {
        const user = await User.findByIdAndUpdate(
            req.params.id,
            { $set: req.body },
            { new: true, runValidators: true }
        );
        if (!user) {
            return res.status(404).json({ error: { code: 'NOT_FOUND', message: 'User not found' } });
        }
        res.json({ data: user });
    } catch (error) {
        next(error);
    }
});

// DELETE /api/users/:id - Delete resource
router.delete('/:id', [
    param('id').isMongoId()
], validate, async (req, res, next) => {
    try {
        const user = await User.findByIdAndDelete(req.params.id);
        if (!user) {
            return res.status(404).json({ error: { code: 'NOT_FOUND', message: 'User not found' } });
        }
        res.status(204).send();
    } catch (error) {
        next(error);
    }
});
```

---

### GraphQL Implementation (Complete)

```javascript
const { ApolloServer, gql, AuthenticationError, ForbiddenError } = require('apollo-server-express');
const DataLoader = require('dataloader');

// Schema
const typeDefs = gql`
    type User {
        id: ID!
        name: String!
        email: String!
        role: Role!
        posts: [Post!]!
        followers: [User!]!
        followersCount: Int!
        createdAt: String!
    }
    
    type Post {
        id: ID!
        title: String!
        content: String!
        published: Boolean!
        author: User!
        comments: [Comment!]!
        createdAt: String!
    }
    
    type Comment {
        id: ID!
        text: String!
        author: User!
    }
    
    enum Role {
        USER
        ADMIN
        MODERATOR
    }
    
    type Query {
        user(id: ID!): User
        users(limit: Int, offset: Int, role: Role): [User!]!
        post(id: ID!): Post
        posts(limit: Int, offset: Int, published: Boolean): [Post!]!
        me: User
    }
    
    type Mutation {
        createUser(input: CreateUserInput!): User!
        updateUser(id: ID!, input: UpdateUserInput!): User!
        deleteUser(id: ID!): Boolean!
        createPost(input: CreatePostInput!): Post!
        publishPost(id: ID!): Post!
    }
    
    input CreateUserInput {
        name: String!
        email: String!
        password: String!
    }
    
    input UpdateUserInput {
        name: String
        email: String
    }
    
    input CreatePostInput {
        title: String!
        content: String!
    }
    
    type Subscription {
        postCreated: Post!
        postPublished: Post!
    }
`;

// DataLoader for N+1 problem
const createLoaders = () => ({
    userLoader: new DataLoader(async (userIds) => {
        const users = await User.find({ _id: { $in: userIds } });
        return userIds.map(id => users.find(u => u._id.toString() === id));
    }),
    postsLoader: new DataLoader(async (userIds) => {
        const posts = await Post.find({ authorId: { $in: userIds } });
        return userIds.map(id => posts.filter(p => p.authorId.toString() === id));
    })
});

// Resolvers
const resolvers = {
    Query: {
        user: async (_, { id }, { loaders }) => loaders.userLoader.load(id),
        users: async (_, { limit = 10, offset = 0, role }) => {
            const filter = role ? { role } : {};
            return User.find(filter).skip(offset).limit(limit);
        },
        me: async (_, __, { user }) => {
            if (!user) throw new AuthenticationError('Not authenticated');
            return User.findById(user.id);
        }
    },
    User: {
        posts: async (parent, _, { loaders }) => loaders.postsLoader.load(parent.id),
        followersCount: async (parent) => User.countDocuments({ following: parent.id })
    },
    Mutation: {
        createUser: async (_, { input }) => User.create(input),
        updateUser: async (_, { id, input }, { user }) => {
            if (!user) throw new AuthenticationError('Not authenticated');
            if (user.role !== 'ADMIN' && user.id !== id) throw new ForbiddenError('Not authorized');
            return User.findByIdAndUpdate(id, input, { new: true });
        }
    }
};

const server = new ApolloServer({
    typeDefs,
    resolvers,
    context: ({ req }) => ({
        user: req.user,
        loaders: createLoaders()
    })
});
```

---

### gRPC Implementation (Complete)

```protobuf
// user.proto
syntax = "proto3";
package users;

import "google/protobuf/empty.proto";
import "google/protobuf/timestamp.proto";

service UserService {
    rpc GetUser (GetUserRequest) returns (User);
    rpc ListUsers (ListUsersRequest) returns (ListUsersResponse);
    rpc CreateUser (CreateUserRequest) returns (User);
    rpc UpdateUser (UpdateUserRequest) returns (User);
    rpc DeleteUser (DeleteUserRequest) returns (google.protobuf.Empty);
    rpc StreamUsers (StreamUsersRequest) returns (stream User);
}

message GetUserRequest {
    int32 id = 1;
}

message User {
    int32 id = 1;
    string name = 2;
    string email = 3;
    string role = 4;
    google.protobuf.Timestamp created_at = 5;
}

message ListUsersRequest {
    int32 limit = 1;
    int32 offset = 2;
    string role = 3;
}

message ListUsersResponse {
    repeated User users = 1;
    int32 total = 2;
}

message CreateUserRequest {
    string name = 1;
    string email = 2;
    string password = 3;
}

message UpdateUserRequest {
    int32 id = 1;
    string name = 2;
    string email = 3;
}

message DeleteUserRequest {
    int32 id = 1;
}

message StreamUsersRequest {
    string role = 1;
}
```

```javascript
// gRPC Server Implementation
const grpc = require('@grpc/grpc-js');
const protoLoader = require('@grpc/proto-loader');

const packageDef = protoLoader.loadSync('user.proto');
const proto = grpc.loadPackageDefinition(packageDef);

const server = new grpc.Server();

server.addService(proto.users.UserService.service, {
    GetUser: async (call, callback) => {
        try {
            const user = await User.findById(call.request.id);
            if (!user) {
                return callback({ code: grpc.status.NOT_FOUND, message: 'User not found' });
            }
            callback(null, user);
        } catch (error) {
            callback({ code: grpc.status.INTERNAL, message: error.message });
        }
    },
    
    ListUsers: async (call, callback) => {
        try {
            const { limit = 10, offset = 0, role } = call.request;
            const filter = role ? { role } : {};
            const [users, total] = await Promise.all([
                User.find(filter).skip(offset).limit(limit),
                User.countDocuments(filter)
            ]);
            callback(null, { users, total });
        } catch (error) {
            callback({ code: grpc.status.INTERNAL, message: error.message });
        }
    },
    
    StreamUsers: async (call) => {
        try {
            const { role } = call.request;
            const filter = role ? { role } : {};
            const users = User.find(filter).cursor();
            
            users.on('data', (user) => call.write(user));
            users.on('end', () => call.end());
            users.on('error', (err) => call.destroy(err));
        } catch (error) {
            call.destroy(error);
        }
    }
});

server.bindAsync('0.0.0.0:50051', grpc.ServerCredentials.createInsecure(), () => {
    console.log('gRPC server running on port 50051');
});
```

---

### WebSocket Implementation

```javascript
const WebSocket = require('ws');
const { v4: uuidv4 } = require('uuid');

const wss = new WebSocket.Server({ port: 8080 });

// Connection handling
wss.on('connection', (ws, req) => {
    const userId = authenticateUser(req);
    ws.userId = userId;
    ws.isAlive = true;
    
    // Heartbeat
    ws.on('pong', () => { ws.isAlive = true; });
    
    // Message handling
    ws.on('message', (data) => {
        try {
            const message = JSON.parse(data);
            handleMessage(ws, message);
        } catch (error) {
            ws.send(JSON.stringify({ error: 'Invalid message format' }));
        }
    });
    
    // Disconnect
    ws.on('close', () => {
        console.log(`User ${userId} disconnected`);
    });
});

// Heartbeat interval
setInterval(() => {
    wss.clients.forEach((ws) => {
        if (!ws.isAlive) return ws.terminate();
        ws.isAlive = false;
        ws.ping();
    });
}, 30000);

// Broadcast to all clients
function broadcast(message) {
    wss.clients.forEach((client) => {
        if (client.readyState === WebSocket.OPEN) {
            client.send(JSON.stringify(message));
        }
    });
}

// Send to specific user
function sendToUser(userId, message) {
    wss.clients.forEach((client) => {
        if (client.userId === userId && client.readyState === WebSocket.OPEN) {
            client.send(JSON.stringify(message));
        }
    });
}
```

---

### Server-Sent Events (SSE)

```javascript
// Express SSE endpoint
app.get('/api/events', (req, res) => {
    res.writeHead(200, {
        'Content-Type': 'text/event-stream',
        'Cache-Control': 'no-cache',
        'Connection': 'keep-alive'
    });
    
    const sendEvent = (data) => {
        res.write(`data: ${JSON.stringify(data)}\n\n`);
    };
    
    // Send heartbeat every 30 seconds
    const heartbeat = setInterval(() => {
        res.write(': heartbeat\n\n');
    }, 30000);
    
    // Subscribe to events
    const onEvent = (event) => sendEvent(event);
    eventEmitter.on('update', onEvent);
    
    // Cleanup on disconnect
    req.on('close', () => {
        clearInterval(heartbeat);
        eventEmitter.off('update', onEvent);
    });
});
```

---

### Additional Interview Questions (20+)

**Q31: How do you handle API authentication in REST?**
A: "JWT in Authorization header (Bearer token). API keys in header (X-API-Key). OAuth 2.0 for third-party access. Session cookies for web apps. Choose based on use case."

**Q32: What is BFF (Backend for Frontend) pattern?**
A: "Separate backend for each frontend type (mobile, web, desktop). Each BFF optimized for its frontend's needs. Reduces over-fetching. API gateway pattern variant."

**Q33: How do you handle API caching?**
A: "HTTP caching headers (Cache-Control, ETag, Last-Modified). CDN caching. Application-level caching (Redis). Cache invalidation strategies (TTL, manual, event-based)."

**Q34: What is API contract testing?**
A: "Verify API matches documentation. Tools: Pact, Dredd. Consumer-driven contracts. Ensures backward compatibility. Run in CI/CD pipeline."

**Q35: How do you handle API monitoring?**
A: "Log all requests (method, path, status, duration). Track error rates. Monitor response times. Set up alerts for anomalies. Tools: Datadog, New Relic, Prometheus."

**Q36: What is API composition pattern?**
A: "Aggregate data from multiple services into single response. API gateway or dedicated composition layer. Reduces client-side complexity."

**Q37: How do you handle API pagination with cursors?**
A: "Encode cursor as base64 of last seen ID. Client sends cursor in next request. Server uses WHERE id > cursor. More efficient than offset for large datasets."

**Q38: What is API chaining?**
A: "Client makes sequential requests, each using data from previous. Problem: multiple round trips. Solution: GraphQL (single request), BFF, API composition."

**Q39: How do you handle API errors globally?**
A: "Error handling middleware. Custom error classes. Consistent error format. Log errors server-side. Return appropriate status codes."

**Q40: What is API gateway vs load balancer?**
A: "API gateway: application-level routing, auth, rate limiting. Load balancer: network-level traffic distribution. API gateway operates at Layer 7; load balancer at Layer 4/7."

**Q41: How do you implement API retries?**
A: "Exponential backoff with jitter. Retry on 5xx errors and timeouts. Idempotency keys for safe retries. Maximum retry count. Circuit breaker for persistent failures."

**Q42: What is API versioning strategy?**
A: "URL path (/v1/users) for public APIs. Header versioning for internal. Deprecation headers. Sunset dates. Maintain old versions for 6-12 months."

**Q43: How do you handle API rate limiting per user?**
A: "Track requests per user ID in Redis. Different limits for different tiers. Sliding window algorithm. Return 429 with Retry-After header."

**Q44: What is API throttling?**
A: "Slow down requests when approaching rate limit. Delays responses instead of rejecting. Use token bucket algorithm. More graceful than hard rate limiting."

**Q45: How do you implement API search?**
A: "Full-text search with Elasticsearch. Autocomplete with prefix matching. Faceted search. Filtering and sorting. Pagination of results."

**Q46: What is API webhook pattern?**
A: "Server pushes events to client URL. Client registers webhook URL. Server sends HTTP POST on events. Retry on failure. Verify signatures."

**Q47: How do you handle API file downloads?**
A: "Stream large files. Set Content-Disposition header. Support range requests for resumption. Use cloud storage with signed URLs."

**Q48: What is API pagination best practice?**
A: "Cursor-based for large datasets. Offset for admin dashboards. Include total count. Provide next/prev links. Consistent response format."

**Q49: How do you test GraphQL APIs?**
A: "Unit test resolvers. Integration test with Apollo Server. Test queries and mutations. Mock data loaders. Test error handling."

**Q50: What is API gateway pattern benefits?**
A: "Single entry point. Centralized auth. Rate limiting. Load balancing. Logging. Caching. Simplifies client. Cross-cutting concerns."

---

*Next: [02 — API Design Best Practices](https://github.com/Aditya-myst/Interview-PlayBook/blob/main/BACKEND/API_DESIGN.md)*
