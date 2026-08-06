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

*Next: [02 — API Integration](02-API-Integration.md)*
