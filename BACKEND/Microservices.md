# 08 — Microservices Architecture

## Patterns, Communication, Trade-offs — 35+ Interview Questions

---

### Monolith vs Microservices

| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| **Deployment** | Single unit | Independent |
| **Scaling** | Scale entire app | Scale individual services |
| **Technology** | Single stack | Polyglot |
| **Complexity** | Simple initially | Complex (distributed) |
| **Team** | Large team coordination | Small autonomous teams |

---

### Communication Patterns

```javascript
// Synchronous (HTTP)
const response = await axios.get('http://user-service/api/users/123');

// Asynchronous (Message Queue)
await kafka.publish('user.created', { userId: 123 });

// gRPC (faster)
const user = await userClient.getUser({ id: 123 });
```

---

### Circuit Breaker Pattern

```javascript
const CircuitBreaker = require('opossum');

const breaker = new CircuitBreaker(async function() {
    return await axios.get('http://external-service/api');
}, {
    timeout: 3000,
    errorThresholdPercentage: 50,
    resetTimeout: 30000
});

breaker.fallback(() => ({ data: 'cached or default' }));
breaker.on('open', () => console.log('Circuit opened'));
```

---

### Saga Pattern

```javascript
async function createOrder(orderData) {
    try {
        const order = await orderService.create(orderData);
        await paymentService.charge(order.payment);
        await inventoryService.reserve(order.items);
        await notificationService.send(order.user);
        return order;
    } catch (error) {
        await inventoryService.release(order.items);
        await paymentService.refund(order.payment);
        await orderService.cancel(order.id);
        throw error;
    }
}
```

---

### Interview Questions (35+)

**Q1: When would you use microservices over monolith?**
A: "Microservices: multiple teams, different scaling needs, technology diversity, independent deployment. Monolith: small team, simple domain, starting out. Most start monolith, extract as needed."

**Q2: How do services communicate in microservices?**
A: "Synchronous: HTTP/REST, gRPC. Asynchronous: message queues (Kafka, RabbitMQ). Sync for queries, async for events. Use API gateway for external clients."

**Q3: What is API gateway pattern?**
A: "Single entry point for all requests. Handles routing, authentication, rate limiting, logging. Examples: Kong, AWS API Gateway. Simplifies client, centralizes cross-cutting concerns."

**Q4: What is service discovery?**
A: "How services find each other. Client-side: client queries registry. Server-side: load balancer handles. Tools: Consul, Eureka, etcd."

**Q5: What is circuit breaker pattern?**
A: "Prevent cascading failures. States: closed (normal), open (failing, fast-fail), half-open (testing). Trip on error threshold. Fallback to cached/default response."

**Q6: What is saga pattern?**
A: "Distributed transaction using compensating actions. Orchestration: central coordinator. Choreography: events between services. Handle failures with rollback."

**Q7: What is event sourcing?**
A: "Store events instead of current state. Replay events to rebuild state. Audit trail built-in. Complex queries need CQRS."

**Q8: What is CQRS?**
A: "Command Query Responsibility Segregation. Separate write model from read model. Optimize each independently. Often combined with event sourcing."

**Q9: How do you handle distributed transactions?**
A: "Saga pattern with compensating actions. Two-phase commit (complex, slow). Eventual consistency (most common). Choose based on consistency requirements."

**Q10: What is database per service pattern?**
A: "Each microservice owns its database. No shared databases. Ensures loose coupling. Handle cross-service queries with API composition or events."

**Q11: How do you handle cross-cutting concerns?**
A: "API gateway for authentication, rate limiting. Service mesh for observability, security. Shared libraries for logging, configuration."

**Q12: What is service mesh?**
A: "Infrastructure layer for service-to-service communication. Handles load balancing, retries, mTLS, observability. Examples: Istio, Linkerd, Consul Connect."

**Q13: How do you test microservices?**
A: "Unit tests for business logic. Integration tests for service interactions. Contract tests (Pact) for API compatibility. End-to-end tests for critical flows."

**Q14: What is strangler fig pattern?**
A: "Gradually replace monolith with microservices. Route requests to new services incrementally. Old functionality 'strangled' over time. Low-risk migration."

**Q15: How do you handle configuration in microservices?**
A: "Centralized configuration server (Spring Cloud Config, Consul). Environment variables. Feature flags. Secrets management (Vault)."

**Q16: What is sidecar pattern?**
A: "Deploy helper process alongside main service. Handles cross-cutting concerns: logging, monitoring, proxying. Used in service mesh."

**Q17: How do you handle data consistency?**
A: "Eventual consistency (most common). Saga pattern for transactions. Event sourcing for audit trail. Choose based on business requirements."

**Q18: What is bulkhead pattern?**
A: "Isolate components to prevent cascading failures. Separate thread pools, connection pools. Failure in one doesn't affect others."

**Q19: How do you deploy microservices?**
A: "Containerization (Docker). Orchestration (Kubernetes). CI/CD pipeline per service. Blue-green or canary deployments."

**Q20: What is API composition pattern?**
A: "Aggregate data from multiple services. API gateway or dedicated composition service. Reduces client complexity."

**Q21: How do you handle service failures?**
A: "Circuit breaker, retry with backoff, fallback responses, bulkhead isolation. Monitor and alert on failures."

**Q22: What is domain-driven design?**
A: "Design services around business domains. Bounded contexts define service boundaries. Ubiquitous language within each context."

**Q23: How do you handle versioning in microservices?**
A: "API versioning (v1, v2). Backward compatibility. Deprecation strategy. Consumer-driven contracts."

**Q24: What is event-driven architecture?**
A: "Services communicate via events. Loose coupling. Event bus (Kafka, RabbitMQ). Enables async processing."

**Q25: How do you monitor microservices?**
A: "Distributed tracing (Jaeger, Zipkin). Centralized logging (ELK). Metrics (Prometheus, Grafana). Health checks."

**Q26: What is contract testing?**
A: "Verify API matches documentation. Consumer-driven contracts (Pact). Ensures backward compatibility. Run in CI/CD."

**Q27: How do you handle authentication in microservices?**
A: "API gateway validates JWT. Pass user info in headers. Or use service mesh for mTLS. Token introspection for critical operations."

**Q28: What is shared database anti-pattern?**
A: "Multiple services sharing one database. Creates tight coupling. Each service should own its data. Use API for cross-service data access."

**Q29: How do you handle logging in microservices?**
A: "Structured logging (JSON). Correlation IDs for request tracing. Centralized logging (ELK, Splunk). Log aggregation."

**Q30: What is blue-green deployment?**
A: "Two identical environments. Switch traffic from blue to green. Instant rollback capability. Zero downtime deployment."

**Q31: What is canary deployment?**
A: "Route small percentage of traffic to new version. Monitor for issues. Gradually increase traffic. Rollback if problems detected."

**Q32: How do you handle distributed tracing?**
A: "Propagate trace ID across services. Tools: Jaeger, Zipkin, AWS X-Ray. Visualize request flow. Identify bottlenecks."

**Q33: What is feature flag pattern?**
A: "Toggle features without deployment. Gradual rollout. A/B testing. Kill switch for problematic features."

**Q34: How do you handle service dependencies?**
A: "Define clear dependencies. Use circuit breaker for external calls. Implement graceful degradation. Monitor dependency health."

**Q35: What is choreography vs orchestration?**
A: "Orchestration: central coordinator directs saga. Choreography: services react to events. Orchestration easier to understand; choreography more decoupled."

---

### Complete Microservices Implementation

```javascript
// API Gateway (Express)
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');
const jwt = require('jsonwebtoken');

const app = express();

// Authentication middleware
const authenticate = (req, res, next) => {
    const token = req.headers.authorization?.split(' ')[1];
    if (!token) return res.status(401).json({ error: 'No token' });
    
    try {
        req.user = jwt.verify(token, process.env.JWT_SECRET);
        next();
    } catch (error) {
        res.status(401).json({ error: 'Invalid token' });
    }
};

// Rate limiting
const rateLimit = require('express-rate-limit');
const limiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 100 });
app.use(limiter);

// Service routes
app.use('/api/users', authenticate, createProxyMiddleware({
    target: 'http://user-service:3001',
    changeOrigin: true,
    pathRewrite: { '^/api/users': '/users' }
}));

app.use('/api/orders', authenticate, createProxyMiddleware({
    target: 'http://order-service:3002',
    changeOrigin: true,
    pathRewrite: { '^/api/orders': '/orders' }
}));

app.use('/api/products', createProxyMiddleware({
    target: 'http://product-service:3003',
    changeOrigin: true,
    pathRewrite: { '^/api/products': '/products' }
}));

// Circuit breaker
const CircuitBreaker = require('opossum');
const breaker = new CircuitBreaker(async (serviceUrl) => {
    const response = await axios.get(serviceUrl);
    return response.data;
}, {
    timeout: 3000,
    errorThresholdPercentage: 50,
    resetTimeout: 30000
});

breaker.fallback(() => ({ error: 'Service unavailable' }));
```

---

### Additional Interview Questions (15+)

**Q36: How do you implement service discovery?**
A: "Client-side: client queries registry (Consul, Eureka). Server-side: load balancer handles routing. Use DNS-based discovery in Kubernetes."

**Q37: What is API gateway vs service mesh?**
A: "API gateway: external traffic routing, auth, rate limiting. Service mesh: internal service communication, observability, mTLS. Both can coexist."

**Q38: How do you handle data consistency across services?**
A: "Saga pattern with compensating actions. Event sourcing for audit trail. Eventual consistency for most cases. Choose based on business requirements."

**Q39: What is database per service challenges?**
A: "Cross-service queries, data consistency, distributed transactions. Solutions: API composition, event sourcing, CQRS."

**Q40: How do you implement distributed tracing?**
A: "Propagate trace ID in headers. Use Jaeger, Zipkin, or AWS X-Ray. Add spans for each service call. Visualize request flow."

**Q41: What is service mesh benefits?**
A: "Automatic mTLS, load balancing, retries, circuit breaking, observability. Examples: Istio, Linkerd, Consul Connect."

**Q42: How do you handle service versioning?**
A: "URL path versioning (/v1/users). Header versioning. Backward compatibility. Deprecation strategy."

**Q43: What is event sourcing benefits?**
A: "Complete audit trail. Replay events to rebuild state. Temporal queries. Debugging. Complex queries need CQRS."

**Q44: How do you implement CQRS?**
A: "Separate write model (commands) and read model (queries). Write model: normalized DB. Read model: denormalized, optimized for queries."

**Q45: What is strangler fig migration?**
A: "Gradually replace monolith. Route requests to new services. Extract services incrementally. Low-risk migration."

**Q46: How do you handle configuration in microservices?**
A: "Centralized config server (Consul, Spring Cloud Config). Environment variables. Feature flags. Secrets management."

**Q47: What is bulkhead pattern?**
A: "Isolate components to prevent cascading failures. Separate thread pools, connection pools. Failure in one doesn't affect others."

**Q48: How do you test microservices?**
A: "Unit tests for business logic. Integration tests for service interactions. Contract tests (Pact). End-to-end tests."

**Q49: What is API composition pattern?**
A: "Aggregate data from multiple services. API gateway or dedicated composition service. Reduces client complexity."

**Q50: How do you handle service failures gracefully?**
A: "Circuit breaker, retry with backoff, fallback responses, bulkhead isolation. Monitor and alert on failures."

---

*Next: [09 — System Design Fundamentals](09-System-Design.md)*
