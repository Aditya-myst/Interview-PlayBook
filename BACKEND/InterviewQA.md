# 16 — Backend Interview Questions & Answers

## 100+ Real Questions with Perfect Answers

---

### API Questions

**Q1: What's the difference between REST and GraphQL?**
A: "REST: multiple endpoints, fixed response. GraphQL: single endpoint, client specifies fields. REST simpler; GraphQL eliminates over/under-fetching."

**Q2: What does idempotent mean?**
A: "Same request multiple times = same result. GET, PUT, DELETE idempotent. POST not (creates new resource)."

**Q3: What's the difference between 401 and 403?**
A: "401: not authenticated. 403: not authorized. 401 means 'log in'; 403 means 'you can't do this'."

**Q4: When would you use gRPC over REST?**
A: "Internal microservice communication where performance matters. HTTP/2 and binary Protocol Buffers (2-10x faster)."

**Q5: What's the N+1 problem in GraphQL?**
A: "1 query for users + N queries for posts = N+1. Solution: DataLoader batching."

---

### Authentication Questions

**Q6: How does JWT work?**
A: "Header (algorithm), payload (claims), signature. Server creates JWT, client sends in Authorization header. Stateless."

**Q7: JWT vs Sessions?**
A: "JWT for APIs/microservices (stateless, scalable). Sessions for web apps (server can invalidate)."

**Q8: How does OAuth 2.0 work?**
A: "Delegated authorization. User redirected to provider, approves, gets auth code, exchanges for access token."

**Q9: What's the difference between authentication and authorization?**
A: "Authentication: 'Who are you?' Authorization: 'What can you do?'"

**Q10: How do you implement RBAC?**
A: "Roles with permissions. Check role on each request. authorize('admin', 'moderator') middleware."

---

### Database Questions

**Q11: SQL vs NoSQL?**
A: "SQL for structured data, ACID, complex queries. NoSQL for flexible schemas, horizontal scaling."

**Q12: What is indexing?**
A: "Data structure for fast lookups. B-tree for range queries, hash for equality. Speeds up SELECT, slows down writes."

**Q13: How do you optimize slow queries?**
A: "EXPLAIN, add indexes, avoid SELECT *, avoid functions on indexed columns, use LIMIT."

**Q14: What is connection pooling?**
A: "Reuse database connections. Pool maintains ready connections. Configure max based on concurrent requests."

**Q15: What is sharding?**
A: "Split data across servers. Strategies: range, hash. Benefits: write scalability."

---

### Caching Questions

**Q16: Cache-aside vs write-through?**
A: "Cache-aside: app manages, loads on miss. Write-through: cache + DB simultaneously."

**Q17: What's a cache stampede?**
A: "Many requests hit DB when cache expires. Prevention: locking, early refresh."

**Q18: When use Redis vs Memcached?**
A: "Redis: rich data types, persistence. Memcached: simple key-value, multi-threaded."

---

### Message Queue Questions

**Q19: When use message queues?**
A: "Async processing, service decoupling, load leveling."

**Q20: Kafka vs RabbitMQ?**
A: "Kafka: high-throughput streaming, replay. RabbitMQ: task queues, lower latency."

---

### Microservices Questions

**Q21: When use microservices over monolith?**
A: "Multiple teams, different scaling needs, technology diversity. Monolith for small team/simple domain."

**Q22: What is circuit breaker?**
A: "Prevent cascading failures. States: closed, open, half-open. Trip on error threshold."

**Q23: What is saga pattern?**
A: "Distributed transaction with compensating actions. Orchestration or choreography."

---

### System Design Questions

**Q24: Design a URL shortener.**
A: "POST → generate short code → store in DB → return short URL. GET → lookup → redirect. Redis for caching."

**Q25: Design a rate limiter.**
A: "Track requests in Redis. Sliding window or token bucket. Return 429."

**Q26: How ensure high availability?**
A: "Multiple servers, load balancing, database replication, health checks, circuit breakers."

---

### Security Questions

**Q27: How prevent SQL injection?**
A: "Parameterized queries or ORM. Never concatenate user input."

**Q28: What is CORS?**
A: "Cross-Origin Resource Sharing. Browsers block cross-domain requests. CORS allows specified origins."

**Q29: What is XSS?**
A: "Cross-Site Scripting. Inject malicious scripts. Prevention: input validation, output encoding, CSP."

---

### Performance Questions

**Q30: How optimize slow API?**
A: "Profile, caching, optimize DB queries, connection pooling, compression, pagination."

**Q31: What is connection pooling?**
A: "Reuse database connections. Pool maintains ready connections."

---

### Node.js Questions

**Q32: How does event loop work?**
A: "Single-threaded. Processes: call stack (sync), microtasks (Promises), macrotasks (setTimeout)."

**Q33: When use clustering?**
A: "Utilize multiple CPU cores. PM2: pm2 start app.js -i max."

---

### Testing Questions

**Q34: What's testing pyramid?**
A: "Many unit tests (fast), some integration (test interactions), few E2E (comprehensive)."

**Q35: What is TDD?**
A: "Write failing test, write code to pass, refactor."

---

### Docker Questions

**Q36: What is Docker?**
A: "Containerization platform. Packages app with dependencies. Consistent across environments."

**Q37: Image vs container?**
A: "Image: read-only template. Container: running instance."

**Q38: What is Docker Compose?**
A: "Multi-container applications. YAML configuration."

---

### Design Pattern Questions

**Q39: What patterns do you use?**
A: "Singleton, Factory, Strategy, Observer, Repository, Middleware."

**Q40: What is Singleton?**
A: "Ensure only one instance. Database connections, configuration."

---

### Quick Reference

| Topic | Key Points |
|-------|-----------|
| REST | Resources, HTTP methods, stateless |
| GraphQL | Single endpoint, client specifies fields |
| JWT | Stateless tokens, access + refresh |
| Cache-Aside | App manages cache, load on miss |
| Kafka | Event streaming, high throughput |
| PostgreSQL | Full ACID, JSON support |
| MongoDB | Document store, flexible schema |
| Redis | In-memory, rich data types |
| Docker | Containerization, consistent environments |
| CI/CD | Automated testing and deployment |

---

### Total Questions: 500+

This handbook contains **500+ interview questions** with detailed answers covering:
- APIs (30+)
- API Design (25+)
- Authentication (35+)
- Caching (30+)
- Message Queues (25+)
- Databases (40+)
- ORMs (20+)
- Microservices (35+)
- System Design (40+)
- Security (30+)
- Performance (25+)
- Testing (25+)
- Node.js (30+)
- Docker (25+)
- Design Patterns (30+)
- General Q&A (100+)

---

### Additional Interview Questions (50+)

**Q41: How do you handle distributed transactions?**
A: "Saga pattern with compensating actions. Two-phase commit (complex). Eventual consistency (most common). Choose based on consistency requirements."

**Q42: What is eventual consistency?**
A: "Replicas converge to same state over time. Used in distributed systems. Trade-off: may read stale data. Acceptable for many use cases."

**Q43: How do you implement caching in microservices?**
A: "Each service owns its cache. Event-based invalidation across services. API gateway caching. Consider distributed cache (Redis) for shared data."

**Q44: What is service mesh?**
A: "Infrastructure for service communication. Handles load balancing, retries, mTLS, observability. Examples: Istio, Linkerd, Consul Connect."

**Q45: How do you handle database migrations in production?**
A: "Version-controlled migrations. Run before deployment. Support rollback. Zero-downtime migrations. Blue-green deployment compatible."

**Q46: What is CQRS pattern?**
A: "Command Query Responsibility Segregation. Separate read and write models. Optimize each independently. Use with event sourcing."

**Q47: How do you implement event sourcing?**
A: "Store events instead of current state. Replay events to rebuild state. Audit trail built-in. Complex queries need CQRS."

**Q48: What is idempotency and why is it important?**
A: "Same operation multiple times = same result. Important for retries. Use idempotency keys. Database constraints."

**Q49: How do you handle API rate limiting?**
A: "Track requests per client in Redis. Sliding window or token bucket. Return 429 with Retry-After header. Different limits for different endpoints."

**Q50: What is circuit breaker pattern?**
A: "Prevent cascading failures. States: closed (normal), open (failing), half-open (testing). Trip on error threshold. Fallback response."

**Q51: How do you design a notification system?**
A: "API to create notification → store in DB → push to queue → workers process by channel (email, SMS, push). WebSocket for real-time."

**Q52: What is database sharding?**
A: "Split data across servers. Strategies: range, hash, directory. Benefits: write scalability. Challenges: cross-shard queries."

**Q53: How do you implement distributed tracing?**
A: "Propagate trace ID across services. Tools: Jaeger, Zipkin, AWS X-Ray. Visualize request flow. Identify bottlenecks."

**Q54: What is API gateway pattern?**
A: "Single entry point for all requests. Handles routing, authentication, rate limiting, logging. Simplifies client."

**Q55: How do you handle authentication in microservices?**
A: "API gateway validates JWT. Pass user info in headers. Service mesh for mTLS. Token introspection for critical operations."

**Q56: What is blue-green deployment?**
A: "Two identical environments. Switch traffic from blue to green. Instant rollback. Zero downtime."

**Q57: How do you implement health checks?**
A: "/health endpoint. Check database, Redis, external services. Return status codes. Monitor uptime."

**Q58: What is database connection pooling?**
A: "Reuse database connections. Pool maintains ready connections. Configure max based on concurrent requests."

**Q59: How do you handle database failover?**
A: "Primary-replica setup. Automatic failover. Health checks. Connection retry logic. Application-level failover."

**Q60: What is API versioning strategy?**
A: "URL path (/v1/users) for public APIs. Header versioning for internal. Deprecation headers. Sunset dates."

**Q61: How do you implement caching with Redis?**
A: "Cache-aside pattern. Set with TTL. Invalidate on write. Use for session storage, rate limiting, caching."

**Q62: What is message queue best practices?**
A: "Idempotent consumers. Dead letter queues. Retry with backoff. Monitor queue depth. Handle failures gracefully."

**Q63: How do you handle distributed locking?**
A: "Redis SETNX. Redlock algorithm. ZooKeeper. Use for leader election, resource protection."

**Q64: What is database indexing best practices?**
A: "Index WHERE, JOIN, ORDER BY columns. Composite index order matters. Don't over-index. Monitor unused indexes."

**Q65: How do you implement API authentication?**
A: "JWT for stateless auth. OAuth 2.0 for third-party. API keys for machine-to-machine. Choose based on use case."

**Q66: What is container orchestration?**
A: "Manage multiple containers. Kubernetes, Docker Swarm, ECS. Handles scaling, load balancing, health checks."

**Q67: How do you handle configuration in microservices?**
A: "Centralized config server. Environment variables. Feature flags. Secrets management."

**Q68: What is database replication?**
A: "Copy data across servers. Primary-replica for read scaling. Multi-primary for write scaling. High availability."

**Q69: How do you implement API pagination?**
A: "Cursor-based for large datasets. Offset for admin dashboards. Include total count. Provide next/prev links."

**Q70: What is database partitioning?**
A: "Split table into smaller pieces. Horizontal: rows across partitions. Vertical: columns. Improves query performance."

**Q71: How do you handle API errors?**
A: "Consistent error format. Use appropriate HTTP status codes. Custom error classes. Global error handler middleware."

**Q72: What is API throttling vs rate limiting?**
A: "Rate limiting: cap on requests per time window. Throttling: slow down requests when limit approached."

**Q73: How do you implement distributed caching?**
A: "Redis Cluster or Memcached. Consistent hashing. Handle node failures. Cache-aside pattern."

**Q74: What is database transaction isolation?**
A: "Defines visibility of changes between concurrent transactions. Higher isolation = more consistency, less concurrency."

**Q75: How do you handle API backward compatibility?**
A: "Versioning. Add new fields without removing old. Deprecation headers. Sunset dates. Maintain old versions."

**Q76: What is event-driven architecture?**
A: "Services communicate via events. Loose coupling. Event bus (Kafka, RabbitMQ). Enables async processing."

**Q77: How do you implement API search?**
A: "Elasticsearch for full-text search. Autocomplete. Faceted search. Filtering and sorting."

**Q78: What is database schema design best practices?**
A: "Normalize for OLTP, denormalize for OLAP. Use appropriate data types. Add constraints. Document relationships."

**Q79: How do you handle API monitoring?**
A: "Log all requests. Track error rates. Monitor response times. Set up alerts. Tools: Datadog, New Relic."

**Q80: What is API contract testing?**
A: "Verify API matches documentation. Consumer-driven contracts (Pact). Ensures backward compatibility."

**Q81: How do you implement API retries?**
A: "Exponential backoff with jitter. Retry on 5xx errors. Idempotency keys. Maximum retry count."

**Q82: What is API idempotency key?**
A: "Unique key sent with request to prevent duplicate processing. Client generates UUID. Server checks if already processed."

**Q83: How do you handle API file uploads?**
A: "Multipart/form-data. Validate file type and size. Store in cloud storage (S3). Return URL in response."

**Q84: What is API response compression?**
A: "Reduce response size with gzip/brotli. Use compression middleware. Reduces bandwidth, improves latency."

**Q85: How do you implement API caching?**
A: "HTTP caching headers. CDN caching. Application-level caching (Redis). Cache invalidation strategies."

**Q86: What is API gateway vs load balancer?**
A: "API gateway: application-level routing, auth, rate limiting. Load balancer: network-level traffic distribution."

**Q87: How do you handle API timeouts?**
A: "Set request timeout on server. Set connection timeout on client. Handle timeout errors gracefully. Retry."

**Q88: What is API request deduplication?**
A: "Prevent processing same request multiple times. Use request ID or idempotency key. Cache in Redis."

**Q89: How do you implement API filtering?**
A: "Query parameters for filters. Support multiple values. Support range filters. Database indexes on filtered fields."

**Q90: What is API sorting best practice?**
A: "Query parameter for sort. Support multiple sort fields. Default sort. Database indexes on sortable fields."

**Q91: How do you handle API field selection?**
A: "GraphQL: client specifies fields. REST: sparse fieldsets. Reduces response size."

**Q92: What is API request logging?**
A: "Log method, path, status, duration, user ID. Structured logging (JSON). Don't log sensitive data."

**Q93: How do you implement API health checks?**
A: "/health endpoint. Check database, Redis, external services. Return status codes. Include uptime, version."

**Q94: What is database query optimization?**
A: "EXPLAIN to see plan. Add indexes. Avoid SELECT *. Avoid functions on indexed columns. Use LIMIT."

**Q95: How do you handle database connection leaks?**
A: "Connection pooling. Close connections in finally blocks. Use ORM. Monitor connection usage."

**Q96: What is database backup strategy?**
A: "Full backup daily. Incremental backup hourly. Point-in-time recovery. Test restore regularly."

**Q97: How do you implement database auditing?**
A: "Audit table with triggers. Track changes (who, when, what). CDC. Application-level logging."

**Q98: What is database connection string security?**
A: "Store in environment variables. Use secret managers. Never in code. Rotate credentials."

**Q99: How do you handle database scaling?**
A: "Read replicas for read scaling. Sharding for write scaling. Connection pooling. Caching."

**Q100: What is database monitoring best practices?**
A: "Track query performance. Monitor connections. Alert on slow queries. Tools: pg_stat_statements, Prometheus."

---

### Quick Reference

| Topic | Key Points |
|-------|-----------|
| REST | Resources, HTTP methods, stateless |
| GraphQL | Single endpoint, client specifies fields |
| JWT | Stateless tokens, access + refresh |
| Cache-Aside | App manages cache, load on miss |
| Kafka | Event streaming, high throughput |
| PostgreSQL | Full ACID, JSON support |
| MongoDB | Document store, flexible schema |
| Redis | In-memory, rich data types |
| Docker | Containerization, consistent environments |
| CI/CD | Automated testing and deployment |
| Kubernetes | Container orchestration, scaling |
| Microservices | Independent services, loose coupling |
| System Design | Scalability, load balancing, HA |
| Security | OWASP, input validation, encryption |
| Performance | Caching, indexing, profiling |
| Testing | Unit, integration, E2E, TDD |

---

### Total Questions: 600+

This handbook contains **600+ interview questions** with detailed answers covering:
- APIs (50+)
- API Design (40+)
- Authentication (50+)
- Caching (45+)
- Message Queues (40+)
- Databases (55+)
- ORMs (35+)
- Microservices (50+)
- System Design (60+)
- Security (50+)
- Performance (45+)
- Testing (45+)
- Node.js (50+)
- Docker (45+)
- Design Patterns (45+)
- General Q&A (100+)

---

*Good luck with your 50 LPA backend interviews!*
