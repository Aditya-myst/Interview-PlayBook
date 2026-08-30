# 01 — Client-Server Architecture

## Understanding the Full Picture

---

### What is Client-Server Architecture?

A model where **clients** (browsers, mobile apps) request resources from **servers** (backend applications) that process requests and return responses.

```
┌─────────────┐         HTTP Request          ┌─────────────┐
│             │  GET /api/users HTTP/1.1       │             │
│   Client    │  Host: api.example.com         │   Server    │
│  (Browser)  │──────────────────────────────>│  (Backend)  │
│             │                                │             │
│             │  HTTP Response                 │             │
│             │  200 OK                        │             │
│             │  { "users": [...] }            │             │
│             │<──────────────────────────────│             │
└─────────────┘                                └─────────────┘
```

---

### The Request Lifecycle

When you type `https://example.com` in your browser:

```
1. DNS Resolution
   Browser → DNS Server: "What's the IP of example.com?"
   DNS Server → Browser: "93.184.216.34"

2. TCP Connection
   Browser → Server: SYN
   Server → Browser: SYN-ACK
   Browser → Server: ACK
   (Three-way handshake)

3. TLS Handshake (HTTPS)
   Browser → Server: "Let's establish secure connection"
   Server → Browser: "Here's my certificate"
   Browser: Verifies certificate
   Both: Agree on encryption keys

4. HTTP Request
   GET / HTTP/1.1
   Host: example.com
   Accept: text/html
   Cookie: session=abc123

5. Server Processing
   - Parse request
   - Route to handler
   - Query database
   - Generate response

6. HTTP Response
   HTTP/1.1 200 OK
   Content-Type: text/html
   <html>...</html>

7. Browser Rendering
   - Parse HTML → DOM tree
   - Parse CSS → CSSOM
   - Combine → Render tree
   - Layout → Paint
```

---

### Frontend Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Frontend                              │
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │
│  │    HTML      │  │    CSS      │  │  JavaScript │   │
│  │  (Structure) │  │  (Style)    │  │  (Behavior) │   │
│  └─────────────┘  └─────────────┘  └─────────────┘   │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              React / Next.js                     │   │
│  │  - Components (UI building blocks)               │   │
│  │  - State Management (data flow)                  │   │
│  │  - Routing (page navigation)                     │   │
│  │  - API Calls (fetch/axios)                       │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

### Backend Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Backend                               │
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │
│  │   Routes     │  │  Middleware │  │ Controllers │   │
│  │ (URL mapping)│  │  (pipeline) │  │ (handlers)  │   │
│  └─────────────┘  └─────────────┘  └─────────────┘   │
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │
│  │  Services   │  │   Models    │  │  Database   │   │
│  │ (business   │  │ (data       │  │ (PostgreSQL │   │
│  │  logic)     │  │  structure) │  │  MongoDB)   │   │
│  └─────────────┘  └─────────────┘  └─────────────┘   │
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │
│  │   Cache     │  │  Queue      │  │  External   │   │
│  │  (Redis)    │  │  (Kafka)    │  │  APIs       │   │
│  └─────────────┘  └─────────────┘  └─────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

### Full Stack Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Client    │────>│   Server    │────>│  Database   │
│  (React)    │     │  (Node.js)  │     │ (PostgreSQL)│
└─────────────┘     └─────────────┘     └─────────────┘
       │                    │                    │
       │                    │                    │
       ▼                    ▼                    ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   CDN       │     │   Cache     │     │   Storage   │
│  (CloudFront│     │  (Redis)    │     │  (S3)       │
└─────────────┘     └─────────────┘     └─────────────┘
```

---

### Express.js Backend Structure

```
src/
├── index.js              # Entry point
├── config/
│   └── database.js       # DB connection
├── middleware/
│   ├── auth.js           # Authentication
│   ├── errorHandler.js   # Error handling
│   └── rateLimiter.js    # Rate limiting
├── routes/
│   ├── auth.js           # /api/auth/*
│   ├── users.js          # /api/users/*
│   └── posts.js          # /api/posts/*
├── controllers/
│   ├── authController.js
│   ├── userController.js
│   └── postController.js
├── services/
│   ├── authService.js
│   ├── userService.js
│   └── postService.js
├── models/
│   ├── User.js
│   └── Post.js
└── utils/
    ├── logger.js
    └── helpers.js
```

```javascript
// index.js - Entry point
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const { errorHandler } = require('./middleware/errorHandler');
const authRoutes = require('./routes/auth');
const userRoutes = require('./routes/users');

const app = express();

// Middleware
app.use(helmet());           // Security headers
app.use(cors());             // Cross-origin requests
app.use(express.json());     // Parse JSON bodies

// Routes
app.use('/api/auth', authRoutes);
app.use('/api/users', userRoutes);

// Error handling
app.use(errorHandler);

app.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();
const { authenticate } = require('../middleware/auth');
const userController = require('../controllers/userController');

router.get('/', userController.getAllUsers);
router.get('/:id', userController.getUserById);
router.post('/', authenticate, userController.createUser);
router.put('/:id', authenticate, userController.updateUser);
router.delete('/:id', authenticate, userController.deleteUser);

module.exports = router;
```

```javascript
// controllers/userController.js
const userService = require('../services/userService');

exports.getAllUsers = async (req, res, next) => {
    try {
        const users = await userService.getAllUsers(req.query);
        res.json({ data: users });
    } catch (error) {
        next(error);
    }
};

exports.getUserById = async (req, res, next) => {
    try {
        const user = await userService.getUserById(req.params.id);
        if (!user) {
            return res.status(404).json({ error: 'User not found' });
        }
        res.json({ data: user });
    } catch (error) {
        next(error);
    }
};
```

---

### Interview Questions

**Q: Walk me through what happens when you type a URL in the browser.**

A: "1) DNS resolution converts domain to IP. 2) TCP connection established (three-way handshake). 3) TLS handshake for HTTPS. 4) HTTP request sent. 5) Server processes request (routing, middleware, handler, database). 6) HTTP response returned. 7) Browser parses HTML, CSS, JavaScript. 8) DOM built, render tree created, painted to screen."

**Q: What's the difference between frontend and backend?**

A: "Frontend: runs in browser, handles UI/UX, uses HTML/CSS/JavaScript/React. Backend: runs on server, handles business logic, database operations, authentication, uses Node.js/Python/Java. Frontend sends requests; backend processes them and returns responses."

**Q: What is middleware in Express.js?**

A: "Functions that execute during the request-response cycle. They can modify request/response objects, execute code, end the cycle, or call the next middleware. Used for authentication, logging, error handling, rate limiting."

---

### Additional Interview Questions (30+)

**Q1: What is the difference between HTTP and HTTPS?**
A: "HTTP: unencrypted, data sent in plain text. HTTPS: encrypted with TLS/SSL, data encrypted in transit. HTTPS prevents eavesdropping, man-in-the-middle attacks. Use HTTPS for all production applications."

**Q2: What are HTTP methods and when would you use each?**
A: "GET: retrieve data (idempotent, safe). POST: create new resource (not idempotent). PUT: replace entire resource (idempotent). PATCH: partial update (not idempotent). DELETE: remove resource (idempotent)."

**Q3: What is the difference between HTTP/1.1 and HTTP/2?**
A: "HTTP/1.1: one request per connection (or pipelining). HTTP/2: multiplexing (multiple requests over single connection), header compression, server push. HTTP/2 is faster for multiple requests."

**Q4: What is REST and what are its principles?**
A: "REST: Representational State Transfer. Principles: stateless, client-server separation, cacheable, uniform interface, layered system. Resources are nouns, methods are verbs."

**Q5: What is the difference between server-side rendering (SSR) and client-side rendering (CSR)?**
A: "SSR: HTML generated on server, sent to browser. Faster initial load, better SEO. CSR: JavaScript runs in browser, renders UI. Better for dynamic apps. Next.js supports both."

**Q6: What is a reverse proxy?**
A: "Server that sits between clients and backend servers. Handles load balancing, SSL termination, caching, compression. Examples: Nginx, HAProxy, AWS ALB."

**Q7: What is load balancing?**
A: "Distribute requests across multiple servers. Algorithms: round robin, least connections, IP hash. Health checks to remove unhealthy servers. Improves availability and scalability."

**Q8: What is the difference between TCP and UDP?**
A: "TCP: reliable, ordered, connection-oriented. UDP: unreliable, unordered, connectionless. TCP for web, email. UDP for gaming, video streaming, DNS."

**Q9: What is DNS and how does it work?**
A: "Domain Name System: translates domain names to IP addresses. Process: browser cache → OS cache → recursive resolver → root server → TLD server → authoritative server."

**Q10: What is a CDN and how does it help?**
A: "Content Delivery Network: caches content at edge locations worldwide. Reduces latency, improves load times. Examples: CloudFront, Cloudflare, Akamai."

**Q11: What is CORS and why is it needed?**
A: "Cross-Origin Resource Sharing. Browsers block requests to different domains by default. CORS allows servers to specify which origins can access their resources."

**Q12: What is the difference between cookies, localStorage, and sessionStorage?**
A: "Cookies: sent with every request, 4KB limit, can be httpOnly. localStorage: persists until cleared, 5-10MB, same origin. sessionStorage: cleared on tab close, 5-10MB, same origin."

**Q13: What is WebSocket and when would you use it?**
A: "Full-duplex communication over single TCP connection. Persistent connection. Use for real-time: chat, live updates, gaming, notifications. Unlike HTTP, server can push to client."

**Q14: What is Server-Sent Events (SSE)?**
A: "Server pushes updates to client over HTTP. Unidirectional (server to client). Use for real-time notifications, live feeds. Simpler than WebSocket for one-way communication."

**Q15: What is the difference between synchronous and asynchronous communication?**
A: "Synchronous: client waits for response (HTTP, REST). Asynchronous: client continues, processes response later (message queues, WebSockets). Sync for queries, async for events."

**Q16: What is API gateway?**
A: "Single entry point for all API requests. Handles routing, authentication, rate limiting, logging, caching. Simplifies client, centralizes cross-cutting concerns."

**Q17: What is microservices architecture?**
A: "Application as suite of small services. Each service independently deployable. Communication via APIs/events. Benefits: scalability, flexibility, technology diversity."

**Q18: What is the difference between monolith and microservices?**
A: "Monolith: single codebase, single deployment. Microservices: multiple services, independent deployment. Monolith simpler initially; microservices better for large teams and scaling."

**Q19: What is containerization?**
A: "Package application with dependencies into container. Consistent across environments. Lightweight compared to VMs. Docker is the most popular container platform."

**Q20: What is the difference between Docker and virtual machines?**
A: "Docker: shares host OS kernel, lightweight, fast startup. VMs: full OS, heavier, slower startup. Docker for application isolation; VMs for OS-level isolation."

**Q21: What is CI/CD?**
A: "Continuous Integration: auto test on push. Continuous Deployment: auto deploy when tests pass. Benefits: catch bugs early, faster releases, consistent deployments."

**Q22: What is infrastructure as code?**
A: "Define infrastructure in code. Tools: Terraform, CloudFormation, Pulumi. Version control, reproducible environments, automated provisioning."

**Q23: What is the difference between SQL and NoSQL databases?**
A: "SQL: relational, fixed schema, ACID, complex queries. NoSQL: document/key-value/graph, flexible schema, horizontal scaling. SQL for structured data; NoSQL for flexible data."

**Q24: What is database indexing?**
A: "Data structure for fast lookups. Speeds up SELECT, slows down INSERT/UPDATE. Choose indexes based on query patterns. B-tree for range queries, hash for equality."

**Q25: What is caching and why use it?**
A: "Store frequently accessed data in faster storage. Reduces database load, improves response times. Redis for distributed cache, CDN for static assets."

**Q26: What is the difference between authentication and authorization?**
A: "Authentication: 'Who are you?'—verify identity. Authorization: 'What can you do?'—check permissions. Auth happens first, then authorization."

**Q27: What is JWT and how does it work?**
A: "JSON Web Token: stateless authentication. Header (algorithm), payload (claims), signature. Server creates JWT, client sends in Authorization header."

**Q28: What is OAuth 2.0?**
A: "Delegated authorization. User redirected to provider, approves, gets auth code, exchanges for access token. Used for social login."

**Q29: What is the difference between session and token-based authentication?**
A: "Session: server stores session data, client has session ID. Token: client stores token (JWT), server validates signature. Session can be invalidated; token valid until expiry."

**Q30: What is rate limiting?**
A: "Limit requests per client per time window. Prevents abuse, ensures fair usage. Implement with Redis. Return 429 when limit exceeded."

---

*Next: [02 — API Integration](02-API-Integration.md)*
