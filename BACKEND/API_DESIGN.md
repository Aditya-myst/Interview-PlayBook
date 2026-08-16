# 02 — API Design Best Practices

## Versioning, Pagination, Error Handling, Rate Limiting — 25+ Interview Questions

---

### API Versioning Strategies

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| **URL Path** | `/v1/users` | Explicit, easy to route | URL changes |
| **Query Parameter** | `/users?version=1` | Clean URLs | Less visible |
| **Header** | `Accept: application/vnd.api.v1+json` | Clean URLs | Hidden |
| **Content Negotiation** | `Accept: application/json;version=1` | RESTful | Complex |

```javascript
// URL path versioning
app.use('/v1/users', usersV1Router);
app.use('/v2/users', usersV2Router);

// Deprecation headers
app.use('/v1/*', (req, res, next) => {
    res.set('Sunset', 'Sat, 01 Jan 2027 00:00:00 GMT');
    res.set('Deprecation', 'true');
    res.set('Link', '</v2/users>; rel="successor-version"');
    next();
});
```

---

### Pagination

```javascript
// Offset-based
GET /api/users?limit=10&offset=20

// Cursor-based
GET /api/users?cursor=eyJpZCI6MTAwfQ==&limit=10

// Response format
{
    "data": [...],
    "pagination": {
        "total": 1000,
        "limit": 10,
        "offset": 20,
        "hasMore": true,
        "next": "/api/users?limit=10&offset=30",
        "prev": "/api/users?limit=10&offset=10"
    }
}
```

---

### Error Handling

```javascript
class AppError extends Error {
    constructor(message, statusCode, code, details = null) {
        super(message);
        this.statusCode = statusCode;
        this.code = code;
        this.details = details;
    }
}

class ValidationError extends AppError {
    constructor(details) {
        super('Validation failed', 400, 'VALIDATION_ERROR', details);
    }
}

class NotFoundError extends AppError {
    constructor(resource, id) {
        super(`${resource} with ID ${id} not found`, 404, 'NOT_FOUND');
    }
}

// Global error handler
app.use((err, req, res, next) => {
    const status = err.statusCode || 500;
    res.status(status).json({
        error: {
            code: err.code || 'INTERNAL_ERROR',
            message: err.message || 'An unexpected error occurred',
            ...(err.details && { details: err.details }),
            ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
        }
    });
});
```

---

### Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const Redis = require('ioredis');

const redisClient = new Redis();

const limiter = rateLimit({
    store: new RedisStore({ sendCommand: (...args) => redisClient.call(...args) }),
    windowMs: 15 * 60 * 1000,
    max: 100,
    standardHeaders: true,
    message: { error: { code: 'RATE_LIMITED', message: 'Too many requests' } }
});

const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 5,
    skipSuccessfulRequests: true
});

app.use('/api/', limiter);
app.use('/api/auth/login', authLimiter);
```

---

### Request Validation

```javascript
const Joi = require('joi');

const userSchema = Joi.object({
    name: Joi.string().min(2).max(50).required(),
    email: Joi.string().email().required(),
    age: Joi.number().integer().min(0).max(150).optional(),
    role: Joi.string().valid('user', 'admin').default('user')
});

function validate(schema) {
    return (req, res, next) => {
        const { error, value } = schema.validate(req.body, { abortEarly: false });
        if (error) {
            const details = error.details.map(d => ({
                field: d.path.join('.'),
                message: d.message
            }));
            throw new ValidationError(details);
        }
        req.body = value;
        next();
    };
}
```

---

### Interview Questions (25+)

**Q1: How do you version your APIs?**
A: "URL path versioning (/v1/users) is most common—explicit and easy to route. Include deprecation headers (Sunset, Deprecation) when retiring old versions. Maintain old versions for 6-12 months."

**Q2: What's the difference between offset and cursor pagination?**
A: "Offset: skip N records, simple but inconsistent with concurrent changes and slow for large offsets. Cursor: use unique identifier, consistent and efficient. Use offset for admin dashboards, cursor for infinite scroll."

**Q3: How do you handle API errors consistently?**
A: "Standard error format: { error: { code, message, details } }. Use appropriate HTTP status codes. Create custom error classes (ValidationError, NotFoundError). Global error handler middleware. Never expose stack traces in production."

**Q4: How do you implement rate limiting?**
A: "Track requests per client (IP or API key) in Redis. Use sliding window or token bucket algorithm. Return 429 with Retry-After header. Different limits for different endpoints (stricter for auth)."

**Q5: How do you validate API requests?**
A: "Use validation libraries (Joi, Yup, Zod). Validate at middleware level. Return detailed validation errors with field names. Validate both request body and query parameters."

**Q6: What's the difference between filtering, sorting, and searching?**
A: "Filtering: exact match on fields (?status=active). Sorting: order by field (?sort=name:asc). Searching: text search (?search=john). Implement with query parameters."

**Q7: How do you handle API pagination with large datasets?**
A: "Cursor-based pagination (efficient, consistent). Keyset pagination (WHERE id > last_id). Avoid offset for large datasets (slow). Use database indexes on cursor column."

**Q8: What is API throttling vs rate limiting?**
A: "Rate limiting: cap on requests per time window. Throttling: slow down requests when limit approached. Rate limiting returns 429; throttling delays responses."

**Q9: How do you handle API backward compatibility?**
A: "Versioning (v1, v2). Add new fields without removing old. Deprecation headers. Sunset dates. Maintain old versions for 6-12 months."

**Q10: What is HATEOAS and when would you use it?**
A: "Hypermedia As The Engine Of Application State. Responses include links to related actions. Client discovers capabilities from responses. Use for complex APIs where client navigation changes frequently."

**Q11: How do you document APIs?**
A: "OpenAPI/Swagger for REST—auto-generate from code annotations. Include examples, error codes, authentication. Tools: Swagger UI, Postman, Redoc. Keep documentation in sync with code."

**Q12: How do you handle file uploads in APIs?**
A: "Use multipart/form-data for REST. Set appropriate size limits. Validate file types. Store in cloud storage (S3, GCS). Return URL in response. For GraphQL, use separate REST endpoint."

**Q13: What is API composition pattern?**
A: "Aggregate data from multiple services into single response. API gateway or dedicated composition layer. Reduces client-side complexity. Example: user profile + posts + followers in one response."

**Q14: How do you handle long-running operations?**
A: "Return 202 Accepted with job ID. Client polls status endpoint. Or use WebSockets/SSE for real-time updates. Or use webhooks for completion notification."

**Q15: What's the difference between REST and RPC?**
A: "REST: resource-oriented (nouns), HTTP methods (verbs), stateless. RPC: action-oriented (verbs), single endpoint, procedure calls. REST for CRUD, RPC for specific operations."

**Q16: How do you test APIs?**
A: "Unit tests for handlers. Integration tests with supertest. Contract tests (Pact). Load testing (k6, Artillery). Security testing (OWASP ZAP). Manual testing with Postman."

**Q17: What is API gateway pattern?**
A: "Single entry point for all API requests. Handles routing, authentication, rate limiting, logging, caching. Examples: Kong, AWS API Gateway, Express gateway. Simplifies client, centralizes cross-cutting concerns."

**Q18: How do you handle API idempotency?**
A: "Idempotency keys for POST requests. Client generates unique key, server deduplicates. Redis to track processed keys. Important for payment processing, retry logic."

**Q19: What's the difference between REST and WebSocket?**
A: "REST: request-response, stateless, HTTP. WebSocket: bidirectional, persistent connection, real-time. REST for CRUD; WebSocket for chat, live updates, gaming."

**Q20: How do you handle API pagination with cursors?**
A: "Encode cursor as base64 of last seen ID. Client sends cursor in next request. Server uses WHERE id > cursor. More efficient than offset for large datasets."

**Q21: What is BFF (Backend for Frontend)?**
A: "Separate backend for each frontend (mobile, web, desktop). Each BFF optimized for its frontend's needs. Reduces over-fetching. API gateway pattern variant."

**Q22: How do you handle API caching?**
A: "HTTP caching headers (Cache-Control, ETag, Last-Modified). CDN caching. Application-level caching (Redis). Cache invalidation strategies (TTL, manual, event-based)."

**Q23: What is API chaining?**
A: "Client makes sequential requests, each using data from previous. Problem: multiple round trips. Solution: GraphQL (single request), BFF, API composition."

**Q24: How do you handle API monitoring?**
A: "Log all requests (method, path, status, duration). Track error rates. Monitor response times. Set up alerts for anomalies. Tools: Datadog, New Relic, Prometheus."

**Q25: What is API contract testing?**
A: "Verify API matches documentation. Tools: Pact, Dredd. Consumer-driven contracts. Ensures backward compatibility. Run in CI/CD pipeline."

---

### Complete Error Handling System

```javascript
// Custom Error Classes
class AppError extends Error {
    constructor(message, statusCode, code, details = null) {
        super(message);
        this.statusCode = statusCode;
        this.code = code;
        this.details = details;
        this.isOperational = true;
        Error.captureStackTrace(this, this.constructor);
    }
}

class ValidationError extends AppError {
    constructor(details) {
        super('Validation failed', 400, 'VALIDATION_ERROR', details);
    }
}

class NotFoundError extends AppError {
    constructor(resource, id) {
        super(`${resource} with ID ${id} not found`, 404, 'NOT_FOUND');
    }
}

class ConflictError extends AppError {
    constructor(message = 'Resource already exists') {
        super(message, 409, 'CONFLICT');
    }
}

class UnauthorizedError extends AppError {
    constructor(message = 'Authentication required') {
        super(message, 401, 'UNAUTHORIZED');
    }
}

class ForbiddenError extends AppError {
    constructor(message = 'Insufficient permissions') {
        super(message, 403, 'FORBIDDEN');
    }
}

class RateLimitError extends AppError {
    constructor(retryAfter) {
        super('Too many requests', 429, 'RATE_LIMITED', { retryAfter });
    }
}

// Global Error Handler
app.use((err, req, res, next) => {
    // Log error
    console.error({
        message: err.message,
        stack: err.stack,
        statusCode: err.statusCode,
        path: req.path,
        method: req.method,
        ip: req.ip,
        userId: req.user?.id
    });
    
    // Handle operational errors
    if (err.isOperational) {
        return res.status(err.statusCode).json({
            error: {
                code: err.code,
                message: err.message,
                ...(err.details && { details: err.details })
            }
        });
    }
    
    // Handle validation errors (Joi)
    if (err.isJoi) {
        const details = err.details.map(d => ({
            field: d.path.join('.'),
            message: d.message
        }));
        return res.status(400).json({
            error: {
                code: 'VALIDATION_ERROR',
                message: 'Validation failed',
                details
            }
        });
    }
    
    // Handle MongoDB errors
    if (err.code === 11000) {
        return res.status(409).json({
            error: {
                code: 'CONFLICT',
                message: 'Duplicate key error'
            }
        });
    }
    
    // Handle JWT errors
    if (err.name === 'JsonWebTokenError') {
        return res.status(401).json({
            error: {
                code: 'INVALID_TOKEN',
                message: 'Invalid token'
            }
        });
    }
    
    if (err.name === 'TokenExpiredError') {
        return res.status(401).json({
            error: {
                code: 'TOKEN_EXPIRED',
                message: 'Token expired'
            }
        });
    }
    
    // Default: 500 Internal Server Error
    res.status(500).json({
        error: {
            code: 'INTERNAL_ERROR',
            message: process.env.NODE_ENV === 'production' 
                ? 'An unexpected error occurred' 
                : err.message
        }
    });
});
```

---

### Complete Rate Limiting System

```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const Redis = require('ioredis');

const redisClient = new Redis({
    host: process.env.REDIS_HOST,
    port: process.env.REDIS_PORT,
    password: process.env.REDIS_PASSWORD
});

// Basic rate limiter
const basicLimiter = rateLimit({
    store: new RedisStore({ sendCommand: (...args) => redisClient.call(...args) }),
    windowMs: 15 * 60 * 1000,  // 15 minutes
    max: 100,                    // 100 requests per window
    standardHeaders: true,
    legacyHeaders: false,
    message: {
        error: {
            code: 'RATE_LIMITED',
            message: 'Too many requests, please try again later',
            retryAfter: 900
        }
    },
    keyGenerator: (req) => req.ip || req.headers['x-forwarded-for']
});

// Strict limiter for auth endpoints
const authLimiter = rateLimit({
    store: new RedisStore({ sendCommand: (...args) => redisClient.call(...args) }),
    windowMs: 15 * 60 * 1000,
    max: 5,
    skipSuccessfulRequests: true,
    message: {
        error: {
            code: 'RATE_LIMITED',
            message: 'Too many login attempts, please try again later'
        }
    }
});

// Per-user limiter
const userLimiter = rateLimit({
    store: new RedisStore({ sendCommand: (...args) => redisClient.call(...args) }),
    windowMs: 60 * 1000,  // 1 minute
    max: 60,               // 60 requests per minute per user
    keyGenerator: (req) => req.user?.id || req.ip
});

// Apply limiters
app.use('/api/', basicLimiter);
app.use('/api/auth/login', authLimiter);
app.use('/api/auth/register', authLimiter);
app.use('/api/', userLimiter);
```

---

### Additional Interview Questions (15+)

**Q26: How do you handle API pagination with multiple filters?**
A: "Combine all filters in WHERE clause. Use database indexes on filtered columns. Cursor-based pagination with composite cursor (encoded last seen values). Cache total count separately."

**Q27: What is API response envelope pattern?**
A: "Wrap response in consistent structure: { data: ..., meta: ..., errors: ... }. Provides consistent shape regardless of content. Include pagination metadata, request ID, timestamp."

**Q28: How do you implement API search?**
A: "Full-text search with Elasticsearch or database full-text search. Autocomplete with prefix matching. Faceted search for filtering. Pagination of results. Relevance scoring."

**Q29: What is API webhook pattern?**
A: "Server pushes events to client URL. Client registers webhook URL. Server sends HTTP POST on events. Retry on failure. Verify signatures for security."

**Q30: How do you handle API file downloads?**
A: "Stream large files. Set Content-Disposition header. Support range requests for resumption. Use cloud storage with signed URLs. Set appropriate content type."

**Q31: What is API idempotency key?**
A: "Unique key sent with request to prevent duplicate processing. Client generates UUID. Server checks if key already processed. Important for payment processing."

**Q32: How do you implement API pagination with GraphQL?**
A: "Cursor-based pagination with Relay specification. Connection type with edges and nodes. PageInfo with hasNextPage, hasPreviousPage. Use after/before arguments."

**Q33: What is API response compression?**
A: "Reduce response size with gzip/brotli. Use compression middleware. Reduces bandwidth, improves latency. Enable for text-based responses. CDN compression."

**Q34: How do you handle API timeouts?**
A: "Set request timeout on server. Set connection timeout on client. Handle timeout errors gracefully. Retry with exponential backoff. Circuit breaker for persistent timeouts."

**Q35: What is API request deduplication?**
A: "Prevent processing same request multiple times. Use request ID or idempotency key. Cache in Redis. Return cached response for duplicate requests."

**Q36: How do you implement API filtering?**
A: "Query parameters for filters (?status=active&role=admin). Support multiple values (?status=active,pending). Support range filters (?age_min=18&age_max=30). Database indexes on filtered fields."

**Q37: What is API sorting best practice?**
A: "Query parameter for sort (?sort=name:asc,-createdAt). Support multiple sort fields. Default sort (usually by createdAt desc). Database indexes on sortable fields."

**Q38: How do you handle API field selection?**
A: "GraphQL: client specifies fields. REST: ?fields=id,name,email sparse fieldsets. Reduces response size. Database: SELECT only needed columns."

**Q39: What is API request logging?**
A: "Log method, path, status, duration, user ID, IP. Structured logging (JSON). Don't log sensitive data (passwords, tokens). Use for debugging and analytics."

**Q40: How do you implement API health checks?**
A: "/health endpoint. Check database, Redis, external services. Return status codes (200 healthy, 503 unhealthy). Include uptime, version. Use for load balancer health checks."

---

*Next: [03 — Authentication & Authorization](03-Auth.md)*
