# 02 — API Design & Best Practices

## Versioning, Pagination, Error Handling

---

### API Versioning

When you change your API, you need versioning to avoid breaking existing clients.

#### Versioning Strategies

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| **URL Path** | `/v1/users`, `/v2/users` | Explicit, easy to route | URL changes |
| **Query Parameter** | `/users?version=1` | Clean URLs | Less visible |
| **Header** | `Accept: application/vnd.api.v1+json` | Clean URLs | Hidden |
| **Content Negotiation** | `Accept: application/json;version=1` | RESTful | Complex |

**Most common: URL path versioning**

```javascript
// Express.js versioning
app.use('/v1/users', usersV1Router);
app.use('/v2/users', usersV2Router);
```

---

### Pagination

Don't return all records at once—paginate large result sets.

#### Offset-Based Pagination

```sql
-- SQL
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;

-- API
GET /users?limit=10&offset=20
```

```javascript
// Response
{
    "data": [...],
    "pagination": {
        "total": 100,
        "limit": 10,
        "offset": 20,
        "hasMore": true
    }
}
```

**Problem:** Inconsistent with concurrent inserts/deletes (items shift).

#### Cursor-Based Pagination

```javascript
// API
GET /users?cursor=eyJpZCI6MTAwfQ==&limit=10

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
-- SQL (more efficient)
SELECT * FROM users WHERE id > 100 ORDER BY id LIMIT 10;
```

**Pros:** Consistent even with concurrent changes. Efficient with indexes.
**Cons:** Can't jump to arbitrary pages.

| Aspect | Offset-Based | Cursor-Based |
|--------|--------------|--------------|
| Jump to page | ✓ | ✗ |
| Consistency | ✗ (shifts) | ✓ |
| Performance | Degrades with offset | Constant |
| Use case | Admin dashboards | Infinite scroll, feeds |

---

### Error Handling

#### Consistent Error Response Format

```javascript
// Good error response
{
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "Invalid input data",
        "details": [
            { "field": "email", "message": "Invalid email format" },
            { "field": "age", "message": "Must be between 0 and 150" }
        ]
    }
}

// Different error levels
{
    "error": {
        "code": "NOT_FOUND",
        "message": "User with ID 123 not found"
    }
}
```

#### Error Handling Middleware

```javascript
// Express.js error handling
class AppError extends Error {
    constructor(message, statusCode, code) {
        super(message);
        this.statusCode = statusCode;
        this.code = code;
    }
}

// Usage
throw new AppError('User not found', 404, 'NOT_FOUND');
throw new AppError('Invalid email', 400, 'VALIDATION_ERROR');

// Global error handler
app.use((err, req, res, next) => {
    const status = err.statusCode || 500;
    const code = err.code || 'INTERNAL_ERROR';
    
    res.status(status).json({
        error: {
            code,
            message: err.message,
            ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
        }
    });
});
```

---

### Rate Limiting

Prevent abuse by limiting requests per client.

```javascript
const rateLimit = require('express-rate-limit');

// Basic rate limiting
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000,  // 15 minutes
    max: 100,                    // 100 requests per window
    message: { error: { code: 'RATE_LIMITED', message: 'Too many requests' } },
    standardHeaders: true,       // Return rate limit info in headers
    legacyHeaders: false,
});

app.use('/api/', limiter);

// Different limits for different endpoints
const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 5,  // Stricter for auth endpoints
});

app.use('/api/auth/login', authLimiter);
```

#### Rate Limiting Headers

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
```

#### Rate Limiting Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **Fixed Window** | Count requests per fixed time window | Simple, general use |
| **Sliding Window** | Rolling window of last N seconds | Smoother limiting |
| **Token Bucket** | Tokens refilled at fixed rate | Allow bursts |
| **Leaky Bucket** | Requests processed at fixed rate | Smooth output |

---

### Request Validation

```javascript
// Using Joi (Node.js)
const Joi = require('joi');

const userSchema = Joi.object({
    name: Joi.string().min(2).max(50).required(),
    email: Joi.string().email().required(),
    age: Joi.number().integer().min(0).max(150),
});

app.post('/users', (req, res) => {
    const { error, value } = userSchema.validate(req.body);
    if (error) {
        return res.status(400).json({
            error: {
                code: 'VALIDATION_ERROR',
                message: error.details[0].message
            }
        });
    }
    // Process valid data
});
```

---

### API Response Format

#### Consistent Envelope

```javascript
// Success
{
    "data": { "id": 1, "name": "Alice" },
    "meta": { "requestId": "abc-123" }
}

// List with pagination
{
    "data": [...],
    "pagination": {
        "total": 100,
        "page": 1,
        "perPage": 10,
        "totalPages": 10
    }
}

// Error
{
    "error": {
        "code": "NOT_FOUND",
        "message": "Resource not found"
    }
}
```

---

### Filtering, Sorting, Searching

```
GET /users?status=active&role=admin        # Filter
GET /users?sort=name:asc,createdAt:desc    # Sort
GET /users?search=john                     # Search
GET /users?fields=id,name,email            # Field selection
```

```javascript
// Implementation
app.get('/users', async (req, res) => {
    let query = 'SELECT * FROM users WHERE 1=1';
    const params = [];
    
    // Filtering
    if (req.query.status) {
        query += ' AND status = ?';
        params.push(req.query.status);
    }
    
    // Sorting
    if (req.query.sort) {
        const [field, order] = req.query.sort.split(':');
        query += ` ORDER BY ${field} ${order === 'desc' ? 'DESC' : 'ASC'}`;
    }
    
    // Pagination
    const limit = parseInt(req.query.limit) || 10;
    const offset = parseInt(req.query.offset) || 0;
    query += ' LIMIT ? OFFSET ?';
    params.push(limit, offset);
    
    const users = await db.query(query, params);
    res.json({ data: users });
});
```

---

### HATEOAS (Hyperlinks)

```javascript
// Response with hyperlinks
{
    "data": {
        "id": 123,
        "name": "Alice",
        "links": {
            "self": "/users/123",
            "posts": "/users/123/posts",
            "followers": "/users/123/followers"
        }
    }
}
```

---

### Interview Questions

**Q: How do you version your APIs?**

A: "URL path versioning is most common (/v1/users). It's explicit and easy to route. Alternatives: query parameter (?version=1), header (Accept: application/vnd.api.v1+json). I'd use URL versioning for public APIs and header versioning for internal APIs."

**Q: What's the difference between offset and cursor pagination?**

A: "Offset: skip N records—simple but inconsistent with concurrent changes and slow for large offsets. Cursor: use a unique identifier to fetch next page—consistent and efficient. Use offset for admin dashboards (need page numbers), cursor for infinite scroll and feeds."

**Q: How do you handle errors in APIs?**

A: "Consistent error format with code, message, and details. Use appropriate HTTP status codes (400 for validation, 401 for auth, 404 for not found, 500 for server error). Include request ID for debugging. Never expose internal errors or stack traces in production."

**Q: How do you implement rate limiting?**

A: "Track requests per client (by IP or API key) in a time window. Use Redis for distributed rate limiting. Return 429 Too Many Requests with Retry-After header. Strategies: fixed window, sliding window, token bucket. Apply different limits for different endpoints (stricter for auth)."

---

*Next: [03 — Authentication & Authorization](03-Auth.md)*
