# 11 — Performance Optimization

## Profiling, Optimization, Monitoring — 25+ Interview Questions

---

### Performance Metrics

| Metric | Description | Target |
|--------|-------------|--------|
| **Response Time** | Time to complete request | < 200ms |
| **Throughput** | Requests per second | Depends on load |
| **Error Rate** | % of failed requests | < 0.1% |
| **CPU Usage** | Server CPU utilization | < 70% |
| **Memory Usage** | Server memory utilization | < 80% |

---

### Node.js Optimization

```javascript
// 1. Clustering
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
    for (let i = 0; i < numCPUs; i++) cluster.fork();
} else {
    app.listen(3000);
}

// 2. Compression
const compression = require('compression');
app.use(compression());

// 3. Response caching
const apicache = require('apicache');
app.get('/api/products', apicache.middleware('5 minutes'), getProducts);

// 4. Database optimization
User.find().select('name email').limit(10).lean();

// 5. Parallel execution
const [users, posts] = await Promise.all([User.find(), Post.find()]);
```

---

### Interview Questions (25+)

**Q1: How do you optimize a slow API?**
A: "1) Profile to find bottleneck. 2) Add caching (Redis). 3) Optimize database queries (indexes, avoid N+1). 4) Connection pooling. 5) Enable compression. 6) Implement pagination. 7) Async/parallel operations."

**Q2: What is connection pooling?**
A: "Reuse database connections. Pool maintains ready connections. Configure max based on concurrent requests. Handle connection errors."

**Q3: How do you profile Node.js applications?**
A: "Clinic.js for visualization. Node.js built-in profiler. Flame graphs for CPU analysis. Heap snapshots for memory leaks."

**Q4: What is memory leak and how do you find it?**
A: "Memory not being freed. Symptoms: increasing memory usage. Tools: heap snapshots, memory profiling. Common causes: event listeners, closures, global variables."

**Q5: How do you optimize database queries?**
A: "EXPLAIN to see execution plan. Add appropriate indexes. Avoid SELECT *. Avoid functions on indexed columns. Use LIMIT. Optimize JOINs."

**Q6: What is response compression?**
A: "Reduce response size with gzip/brotli. Use compression middleware. Reduces bandwidth, improves latency. Enable for text-based responses."

**Q7: How do you handle high traffic?**
A: "Horizontal scaling. Load balancing. Caching. CDN for static assets. Database read replicas. Async processing with queues."

**Q8: What is caching strategy?**
A: "Cache-aside: app manages. Write-through: cache + DB. Write-behind: cache first. Read-through: cache loads from DB. Choose based on consistency needs."

**Q9: How do you optimize static file serving?**
A: "CDN for delivery. Compression (gzip). Caching headers (Cache-Control, ETag). Minification. Image optimization."

**Q10: What is lazy loading?**
A: "Load resources only when needed. Reduces initial load time. Use for images, components, data. Implement with intersection observer."

**Q11: How do you handle database connection issues?**
A: "Connection pooling. Retry logic. Circuit breaker. Health checks. Connection timeout configuration."

**Q12: What is query optimization?**
A: "Analyze execution plan. Add indexes. Avoid N+1 queries. Use EXPLAIN. Optimize JOINs. Denormalize for read-heavy."

**Q13: How do you monitor application performance?**
A: "APM tools (New Relic, Datadog). Response time metrics. Error rates. Database query times. Custom metrics."

**Q14: What is CDN and how does it help?**
A: "Content Delivery Network. Caches content at edge locations. Reduces latency. Serves static assets. Examples: CloudFront, Cloudflare."

**Q15: How do you optimize API responses?**
A: "Pagination. Field selection. Compression. Caching. Avoid over-fetching. Use GraphQL for flexible queries."

**Q16: What is database indexing?**
A: "Data structure for fast lookups. Speeds up SELECT, slows down INSERT/UPDATE. Choose indexes based on query patterns."

**Q17: How do you handle slow database queries?**
A: "EXPLAIN to see plan. Add indexes. Optimize JOINs. Use connection pooling. Consider caching. Denormalize if needed."

**Q18: What is async programming?**
A: "Non-blocking I/O. Node.js event loop. Promises, async/await. Parallel execution with Promise.all. Avoid blocking operations."

**Q19: How do you optimize image loading?**
A: "Responsive images (srcset). Lazy loading. WebP format. CDN delivery. Image compression. Proper sizing."

**Q20: What is code splitting?**
A: "Split code into smaller bundles. Load on demand. Reduces initial load time. Use dynamic imports. Frameworks support this."

**Q21: How do you handle database scaling?**
A: "Read replicas for read scaling. Sharding for write scaling. Connection pooling. Query optimization. Caching."

**Q22: What is APM (Application Performance Monitoring)?**
A: "Tools to monitor application performance. Track response times, errors, database queries. Examples: New Relic, Datadog, Sentry."

**Q23: How do you optimize for mobile?**
A: "Reduce payload size. Enable compression. Optimize images. Implement caching. Use CDN. Minimize round trips."

**Q24: What is database query caching?**
A: "Cache query results. Redis for application cache. Database query cache (MySQL). Materialized views for complex queries."

**Q25: How do you handle performance in production?**
A: "Monitor metrics. Set up alerts. Profile regularly. Load test before releases. Have rollback plan. Document performance budget."

---

### Complete Performance Optimization

```javascript
const cluster = require('cluster');
const compression = require('compression');
const apicache = require('apicache');
const Redis = require('ioredis');

// Clustering
if (cluster.isMaster) {
    const numCPUs = require('os').cpus().length;
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
    cluster.on('exit', (worker) => {
        console.log(`Worker ${worker.process.pid} died`);
        cluster.fork();
    });
} else {
    const app = require('express')();
    
    // Compression
    app.use(compression({
        level: 6,
        threshold: 1024,
        filter: (req, res) => {
            if (req.headers['x-no-compression']) return false;
            return compression.filter(req, res);
        }
    }));
    
    // Response caching
    const cache = apicache.middleware;
    app.get('/api/products', cache('5 minutes'), getProducts);
    app.get('/api/users', cache('1 minute'), getUsers);
    
    // Database optimization
    const { Pool } = require('pg');
    const pool = new Pool({
        max: 20,
        idleTimeoutMillis: 30000,
        connectionTimeoutMillis: 2000
    });
    
    // Query optimization
    async function getUsersOptimized(page = 1, limit = 10) {
        const offset = (page - 1) * limit;
        const query = `
            SELECT id, name, email 
            FROM users 
            WHERE status = 'active'
            ORDER BY created_at DESC 
            LIMIT $1 OFFSET $2
        `;
        return pool.query(query, [limit, offset]);
    }
    
    // Parallel operations
    async function getDashboardData() {
        const [users, posts, comments] = await Promise.all([
            User.countDocuments(),
            Post.countDocuments(),
            Comment.countDocuments()
        ]);
        return { users, posts, comments };
    }
    
    // Memory optimization
    async function processLargeDataset() {
        const cursor = User.find().cursor();
        for await (const user of cursor) {
            await processUser(user);
        }
    }
    
    app.listen(3000);
}
```

---

### Additional Interview Questions (20+)

**Q26: How do you implement caching in Node.js?**
A: "Redis for distributed cache. In-memory (node-cache) for local. HTTP caching headers. Response caching middleware. Cache invalidation strategies."

**Q27: What is connection pooling best practices?**
A: "Set max connections based on CPU cores. Configure idle timeout. Handle connection errors. Monitor pool usage. Close connections in finally blocks."

**Q28: How do you optimize database queries?**
A: "EXPLAIN to see plan. Add indexes. Avoid SELECT *. Avoid functions on indexed columns. Use LIMIT. Optimize JOINs."

**Q29: What is lazy loading?**
A: "Load resources only when needed. Reduces initial load time. Use for images, components, data. Implement with intersection observer."

**Q30: How do you handle memory leaks?**
A: "Heap snapshots. Monitor memory usage. Common causes: event listeners, closures, global variables. Tools: clinic.js, heapdump."

**Q31: What is response compression?**
A: "Reduce response size with gzip/brotli. Use compression middleware. Reduces bandwidth, improves latency. Enable for text-based responses."

**Q32: How do you optimize static file serving?**
A: "CDN for delivery. Compression (gzip). Caching headers (Cache-Control, ETag). Minification. Image optimization."

**Q33: What is database query caching?**
A: "Cache query results. Redis for application cache. Database query cache (MySQL). Materialized views for complex queries."

**Q34: How do you handle high traffic?**
A: "Horizontal scaling. Load balancing. Caching. CDN for static assets. Database read replicas. Async processing with queues."

**Q35: What is APM (Application Performance Monitoring)?**
A: "Tools to monitor application performance. Track response times, errors, database queries. Examples: New Relic, Datadog, Sentry."

**Q36: How do you optimize for mobile?**
A: "Reduce payload size. Enable compression. Optimize images. Implement caching. Use CDN. Minimize round trips."

**Q37: What is code splitting?**
A: "Split code into smaller bundles. Load on demand. Reduces initial load time. Use dynamic imports. Frameworks support this."

**Q38: How do you handle database scaling?**
A: "Read replicas for read scaling. Sharding for write scaling. Connection pooling. Query optimization. Caching."

**Q39: What is async programming?**
A: "Non-blocking I/O. Node.js event loop. Promises, async/await. Parallel execution with Promise.all. Avoid blocking operations."

**Q40: How do you optimize image loading?**
A: "Responsive images (srcset). Lazy loading. WebP format. CDN delivery. Image compression. Proper sizing."

**Q41: What is database indexing?**
A: "Data structure for fast lookups. Speeds up SELECT, slows down INSERT/UPDATE. Choose indexes based on query patterns."

**Q42: How do you handle slow database queries?**
A: "EXPLAIN to see plan. Add indexes. Optimize JOINs. Use connection pooling. Consider caching. Denormalize if needed."

**Q43: How do you monitor application performance?**
A: "APM tools (New Relic, Datadog). Response time metrics. Error rates. Database query times. Custom metrics."

**Q44: What is CDN and how does it help?**
A: "Content Delivery Network. Caches content at edge locations. Reduces latency. Serves static assets. Examples: CloudFront, Cloudflare."

**Q45: How do you optimize API responses?**
A: "Pagination. Field selection. Compression. Caching. Avoid over-fetching. Use GraphQL for flexible queries."

---

*Next: [12 — Testing Strategies](12-Testing.md)*
