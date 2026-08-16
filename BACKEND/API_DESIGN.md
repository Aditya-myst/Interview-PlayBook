# 02 — API Design Best Practices

## Versioning, Pagination, Error Handling, Rate Limiting

---

### API Versioning Strategies

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| **URL Path** | `/v1/users`, `/v2/users` | Explicit, easy to route | URL changes |
| **Query Parameter** | `/users?version=1` | Clean URLs | Less visible |
| **Header** | `Accept: application/vnd.api.v1+json` | Clean URLs | Hidden |
| **Content Negotiation** | `Accept: application/json;version=1` | RESTful | Complex |

**Best practice: URL path versioning for public APIs**

```javascript
app.use('/v1/users', usersV1Router);
app.use('/v2/users', usersV2Router);

// Deprecation header
app.use('/v1/*', (req, res, next) => {
    res.set('Sunset', 'Sat, 01 Jan 2027 00:00:00 GMT');
    res.set('Deprecation', 'true');
    res.set('Link', '</v2/users>; rel="successor-version"');
    next();
});
```

---

### Pagination

#### Offset-Based

```javascript
GET /api/users?limit=10&offset=20

// Response
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

```sql
-- SQL
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;
-- Problem: Slow for large offsets (DB must scan and skip rows)
```

#### Cursor-Based

```javascript
GET /api/users?cursor=eyJpZCI6MTAwfQ==&limit=10

// Response
{
    "data": [...],
    "pagination": {
        "nextCursor": "eyJpZCI6MTEwfQ==",
        "hasMore": true
    }
}
```

```sql
-- SQL (much faster)
SELECT * FROM users WHERE id > 100 ORDER BY id LIMIT 10;
-- Uses index efficiently
```

| Aspect | Offset-Based | Cursor-Based |
|--------|--------------|--------------|
| Jump to page | ✓ | ✗ |
| Consistency | ✗ (shifts on insert/delete) | ✓ |
| Performance | Degrades with offset | Constant |
| Use case | Admin dashboards | Infinite scroll, feeds |

---

### Error Handling

```javascript
// Consistent error response
class AppError extends Error {
    constructor(message, statusCode, code, details = null) {
        super(message);
        this.statusCode = statusCode;
        this.code = code;
        this.details = details;
    }
}

// Error classes
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
    constructor(message) {
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

// Global error handler
app.use((err, req, res, next) => {
    const status = err.statusCode || 500;
    const response = {
        error: {
            code: err.code || 'INTERNAL_ERROR',
            message: err.message || 'An unexpected error occurred',
            ...(err.details && { details: err.details }),
            ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
        }
    };
    res.status(status).json(response);
});
```

---

### Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const Redis = require('ioredis');

const redisClient = new Redis();

// Basic rate limiter
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000,  // 15 minutes
    max: 100,                    // 100 requests per window
    standardHeaders: true,
    legacyHeaders: false,
    message: {
        error: {
            code: 'RATE_LIMITED',
            message: 'Too many requests, please try again later'
        }
    }
});

// Redis-backed (for distributed systems)
const redisLimiter = rateLimit({
    store: new RedisStore({
        sendCommand: (...args) => redisClient.call(...args),
    }),
    windowMs: 15 * 60 * 1000,
    max: 100,
});

// Different limits for different endpoints
const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 5,  // Stricter for auth
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

app.post('/api/users', validate(userSchema), createUser);
```

---

### Filtering, Sorting, Searching

```javascript
GET /api/users?status=active&role=admin           // Filter
GET /api/users?sort=name:asc,createdAt:desc       // Sort
GET /api/users?search=john                         // Search
GET /api/users?fields=id,name,email                // Field selection
GET /api/users?include=posts,profile               // Related resources

// Implementation
function buildQuery(params) {
    let query = {};
    
    // Filtering
    if (params.status) query.status = params.status;
    if (params.role) query.role = params.role;
    
    return query;
}

function buildSort(sortParam) {
    if (!sortParam) return { createdAt: -1 };
    
    return sortParam.split(',').reduce((acc, field) => {
        const [key, order] = field.split(':');
        acc[key] = order === 'desc' ? -1 : 1;
        return acc;
    }, {});
}
```

---

### Interview Questions

**Q: How do you version your APIs?**

A: "URL path versioning (/v1/users) is most common—explicit and easy to route. Include deprecation headers (Sunset, Deprecation) when retiring old versions. Maintain old versions for 6-12 months."

**Q: What's the difference between offset and cursor pagination?**

A: "Offset: skip N records, simple but inconsistent with concurrent changes and slow for large offsets. Cursor: use unique identifier, consistent and efficient. Use offset for admin dashboards, cursor for infinite scroll."

**Q: How do you handle API errors consistently?**

A: "Standard error format: { error: { code, message, details } }. Use appropriate HTTP status codes. Create custom error classes (ValidationError, NotFoundError). Global error handler middleware. Never expose stack traces in production."

---

*Next: [03 — Authentication & Authorization](03-Auth.md)*
