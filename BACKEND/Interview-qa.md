# 08 — Backend Interview Questions & Answers

## Real Questions, Real Answers

---

### API Questions

#### Q: What's the difference between REST and GraphQL?

**A:** "REST: multiple endpoints, fixed response structure, client gets all data or makes multiple requests. GraphQL: single endpoint, client specifies exact fields needed, eliminates over-fetching and under-fetching. REST is simpler and cacheable; GraphQL is more flexible for complex data needs."

---

#### Q: What does idempotent mean in APIs?

**A:** "An operation is idempotent if performing it multiple times has the same effect as once. GET is idempotent (reading doesn't change state). PUT is idempotent (setting to same value). POST is not idempotent (creates new resource each time). DELETE is idempotent (deleting already-deleted resource is same)."

---

#### Q: How do you handle API versioning?

**A:** "URL path is most common (/v1/users)—explicit and easy to route. Alternatives: query parameter, header, content negotiation. I'd use URL versioning for public APIs. When deprecating, maintain old version for 6-12 months with sunset headers."

---

#### Q: What's the difference between 401 and 403?

**A:** "401 Unauthorized: not authenticated—you don't know who I am. 403 Forbidden: not authenticated—I know who you are, but you don't have permission. 401 means 'please log in'; 403 means 'you can't do this.'"

---

### Authentication Questions

#### Q: How does JWT work?

**A:** "JWT has three parts: header (algorithm), payload (claims), signature. Server creates JWT with user info, signs with secret. Client sends JWT in Authorization header. Server verifies signature and extracts user info. Stateless—no server-side session storage needed."

---

#### Q: How would you implement token refresh?

**A:** "Use two tokens: short-lived access token (15 min) and long-lived refresh token (7 days). Access token expires, client uses refresh token to get new access token. Refresh token stored securely (httpOnly cookie). If refresh token is compromised, user must re-authenticate."

---

#### Q: What's OAuth 2.0 and when would you use it?

**A:** "Delegated authorization—let users sign in with Google/GitHub without sharing credentials. Authorization Code flow for web apps, PKCE for mobile/SPA. Use when you want social login or need to access third-party APIs on behalf of users."

---

### Caching Questions

#### Q: What's the difference between cache-aside and write-through?

**A:** "Cache-aside: app manages cache, loads on miss, invalidates on write. Write-through: write to cache and database simultaneously. Cache-aside is simpler, caches only requested data. Write-through ensures consistency but adds write latency."

---

#### Q: How do you handle cache invalidation?

**A:** "TTL for data that changes infrequently. Manual invalidation on write for critical data. Event-based for microservices. The key: stale cache is acceptable for many use cases. Choose consistency vs performance based on requirements."

---

#### Q: When would you use Redis vs Memcached?

**A:** "Redis: rich data types, persistence, pub/sub, Lua scripting. Memcached: simple key-value, multi-threaded, slightly faster for basic caching. Redis for most cases; Memcached for simple, high-throughput caching."

---

### Message Queue Questions

#### Q: When would you use a message queue?

**A:** "When you need async processing, service decoupling, or load leveling. Examples: sending emails after order, processing uploads, distributing work across workers. Don't use when you need immediate response or when operations must be synchronous."

---

#### Q: Kafka vs RabbitMQ—which would you choose?

**A:** "Kafka for high-throughput event streaming, log aggregation, replay capability. RabbitMQ for task queues, request/reply, lower latency. Kafka processes millions of events per second; RabbitMQ is simpler for traditional message brokering."

---

#### Q: How do you handle failed messages?

**A:** "Retry with exponential backoff (1s, 2s, 4s). After max retries, send to Dead Letter Queue (DLQ). Monitor DLQ for patterns. Implement idempotent handlers to safely retry. Alert on DLQ growth."

---

### Database Questions

#### Q: SQL vs NoSQL—when would you use each?

**A:** "SQL for structured data with relationships, ACID requirements, complex queries. NoSQL for flexible schemas, horizontal scaling, high throughput. Most applications start with SQL; add NoSQL when specific needs arise."

---

#### Q: What's connection pooling?

**A:** "A pool of reusable database connections. Creating connections is expensive (TCP handshake, authentication). Pool maintains ready connections. Configure max size based on concurrent requests and database limits. Use libraries like pg-pool or HikariCP."

---

#### Q: How do you optimize slow queries?

**A:** "1) Run EXPLAIN to see execution plan. 2) Add appropriate indexes. 3) Avoid SELECT *. 4) Avoid functions on indexed columns. 5) Use LIMIT. 6) Optimize JOINs. 7) Consider denormalization for read-heavy workloads."

---

### System Design Questions

#### Q: Design a URL shortener.

**A:** "POST /urls with long URL → generate short code (base62 of hash or auto-increment) → store in database → return short URL. GET /:code → look up code → redirect to long URL. Use Redis to cache popular URLs. Handle collisions with retry or unique constraint."

---

#### Q: Design a rate limiter.

**A:** "Track requests per user/IP in Redis. Use sliding window or token bucket algorithm. Store count with TTL. Return 429 when limit exceeded. Different limits for different endpoints. Use Redis for distributed rate limiting."

---

#### Q: Design a notification system.

**A:** "API to create notifications → store in database → push to message queue → workers process by channel (email, SMS, push). Use WebSocket for real-time delivery. Queue ensures reliability. Retry failed deliveries."

---

### The "I Don't Know" Strategy

If stumped:

1. **Start with what you know.** "I haven't worked with that specifically, but based on what I know..."
2. **Think out loud.** "Let me reason through this..."
3. **Give an analogy.** "This is similar to..."
4. **Ask for clarification.** "Can you clarify what aspect you're asking about?"

---

### Quick Reference

| Topic | Key Points |
|-------|-----------|
| REST | Resources, HTTP methods, stateless |
| GraphQL | Single endpoint, client specifies fields |
| gRPC | HTTP/2, binary, fast, streaming |
| JWT | Stateless tokens, access + refresh |
| OAuth | Delegated auth, social login |
| Cache-Aside | App manages cache, load on miss |
| Write-Through | Write to cache + DB simultaneously |
| Kafka | Event streaming, high throughput |
| RabbitMQ | Task queues, lower latency |
| Redis | In-memory, rich data types |
| PostgreSQL | Full ACID, JSON support |
| MongoDB | Document store, flexible schema |

---

*Good luck with your backend interviews!*
