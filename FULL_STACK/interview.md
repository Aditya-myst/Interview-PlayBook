# 12 — Full Stack Interview Questions & Answers

## 50+ Real Questions with Perfect Answers

---

### Architecture Questions

**Q1: Walk me through what happens when a user types a URL.**
A: "1) DNS resolution. 2) TCP handshake. 3) TLS handshake. 4) HTTP request. 5) Server processes (routing, middleware, handler, database). 6) HTTP response. 7) Browser renders (DOM, CSSOM, render tree, paint)."

**Q2: What's the difference between frontend and backend?**
A: "Frontend: browser, UI/UX, HTML/CSS/JS/React. Backend: server, business logic, database, authentication. Frontend sends requests; backend processes and returns responses."

**Q3: What is client-server architecture?**
A: "Client requests resources from server. Server processes and returns responses. Stateless (HTTP). Separation of concerns."

**Q4: What is REST?**
A: "Representational State Transfer. Resources (nouns), HTTP methods (verbs), stateless, cacheable, uniform interface."

**Q5: What is the difference between SQL and NoSQL?**
A: "SQL: relational, fixed schema, ACID, complex queries. NoSQL: document/key-value, flexible schema, horizontal scaling."

---

### API Questions

**Q6: What's the difference between REST and GraphQL?**
A: "REST: multiple endpoints, fixed response. GraphQL: single endpoint, client specifies fields. REST simpler; GraphQL eliminates over/under-fetching."

**Q7: What does idempotent mean?**
A: "Same request multiple times = same result. GET, PUT, DELETE idempotent. POST not."

**Q8: What's the difference between 401 and 403?**
A: "401: not authenticated. 403: not authorized."

**Q9: How do you handle API errors?**
A: "Consistent format: { error: { code, message, details } }. Appropriate HTTP status codes. Global error handler."

**Q10: What is rate limiting?**
A: "Limit requests per client per time window. Redis for distributed. Return 429 when exceeded."

---

### Authentication Questions

**Q11: How does JWT work?**
A: "Header (algorithm), payload (claims), signature. Server creates, client sends in Authorization header. Stateless."

**Q12: JWT vs Sessions?**
A: "JWT: stateless, scalable, can't invalidate. Sessions: server-side, can invalidate, simpler security."

**Q13: How does OAuth 2.0 work?**
A: "Delegated authorization. User redirected to provider, approves, gets auth code, exchanges for access token."

**Q14: What's the difference between authentication and authorization?**
A: "Authentication: 'Who are you?' Authorization: 'What can you do?'"

**Q15: How do you implement RBAC?**
A: "Roles with permissions. Check role on each request. authorize('admin', 'moderator') middleware."

---

### Database Questions

**Q16: What is indexing?**
A: "Data structure for fast lookups. B-tree for range queries, hash for equality. Speeds up SELECT, slows down writes."

**Q17: How do you optimize slow queries?**
A: "EXPLAIN, add indexes, avoid SELECT *, avoid functions on indexed columns, use LIMIT."

**Q18: What is connection pooling?**
A: "Reuse database connections. Pool maintains ready connections. Configure max based on concurrent requests."

**Q19: What is sharding?**
A: "Split data across servers. Strategies: range, hash. Benefits: write scalability."

**Q20: What is ACID?**
A: "Atomicity: all or nothing. Consistency: valid state. Isolation: concurrent transactions don't interfere. Durability: committed data survives crashes."

---

### Caching Questions

**Q21: Cache-aside vs write-through?**
A: "Cache-aside: app manages, loads on miss. Write-through: cache + DB simultaneously."

**Q22: What's a cache stampede?**
A: "Many requests hit DB when cache expires. Prevention: locking, early refresh."

**Q23: When use Redis vs Memcached?**
A: "Redis: rich data types, persistence. Memcached: simple key-value, multi-threaded."

---

### Message Queue Questions

**Q24: When use message queues?**
A: "Async processing, service decoupling, load leveling."

**Q25: Kafka vs RabbitMQ?**
A: "Kafka: high-throughput streaming, replay. RabbitMQ: task queues, lower latency."

---

### Microservices Questions

**Q26: When use microservices over monolith?**
A: "Multiple teams, different scaling needs, technology diversity. Monolith for small team/simple domain."

**Q27: What is circuit breaker?**
A: "Prevent cascading failures. States: closed, open, half-open."

**Q28: What is saga pattern?**
A: "Distributed transaction with compensating actions."

---

### System Design Questions

**Q29: Design a URL shortener.**
A: "POST → generate short code → store in DB → return short URL. GET → lookup → redirect. Redis for caching."

**Q30: Design a rate limiter.**
A: "Track requests in Redis. Sliding window or token bucket. Return 429."

**Q31: How ensure high availability?**
A: "Multiple servers, load balancing, database replication, health checks, circuit breakers."

---

### Security Questions

**Q32: How prevent SQL injection?**
A: "Parameterized queries or ORM. Never concatenate user input."

**Q33: What is CORS?**
A: "Cross-Origin Resource Sharing. Browsers block cross-domain requests."

**Q34: What is XSS?**
A: "Cross-Site Scripting. Inject malicious scripts. Prevention: input validation, output encoding, CSP."

---

### Performance Questions

**Q35: How optimize slow API?**
A: "Profile, caching, optimize DB queries, connection pooling, compression, pagination."

**Q36: What is connection pooling?**
A: "Reuse database connections. Pool maintains ready connections."

---

### Node.js Questions

**Q37: How does event loop work?**
A: "Single-threaded. Processes: call stack (sync), microtasks (Promises), macrotasks (setTimeout)."

**Q38: When use clustering?**
A: "Utilize multiple CPU cores. PM2: pm2 start app.js -i max."

---

### Testing Questions

**Q39: What's testing pyramid?**
A: "Many unit tests (fast), some integration (test interactions), few E2E (comprehensive)."

**Q40: What is TDD?**
A: "Write failing test, write code to pass, refactor."

---

### Docker Questions

**Q41: What is Docker?**
A: "Containerization platform. Packages app with dependencies. Consistent across environments."

**Q42: Image vs container?**
A: "Image: read-only template. Container: running instance."

**Q43: What is Docker Compose?**
A: "Multi-container applications. YAML configuration."

---

### CI/CD Questions

**Q44: What is CI/CD?**
A: "CI: auto test on push. CD: auto deploy when tests pass."

**Q45: What should CI pipeline include?**
A: "Install, lint, test, build, coverage. Optional: security scan, Docker build."

---

### Monitoring Questions

**Q46: What is observability?**
A: "Understanding system behavior. Three pillars: logs, metrics, traces."

**Q47: What should you monitor?**
A: "Health checks, response time, error rate, CPU, memory, disk."

---

### Design Pattern Questions

**Q48: What patterns do you use?**
A: "Singleton, Factory, Strategy, Observer, Repository, Middleware."

**Q49: What is Singleton?**
A: "Ensure only one instance. Database connections, configuration."

**Q50: What is Strategy pattern?**
A: "Define family of algorithms, make them interchangeable."

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

*Good luck with your full stack interviews!*
