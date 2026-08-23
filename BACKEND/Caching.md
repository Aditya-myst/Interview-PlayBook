# 04 — Caching Strategies

## Redis, Cache Patterns, Invalidation — 30+ Interview Questions

---

### Redis Deep Dive

```redis
# Strings
SET user:1:name "Alice"
GET user:1:name
MSET user:1:name "Alice" user:1:email "alice@example.com"
INCR page:views:homepage
SET session:abc123 "user_data" EX 3600

# Hashes
HSET user:1 name "Alice" email "alice@example.com" age 30
HGET user:1 name
HGETALL user:1

# Lists (queues)
LPUSH queue:emails "email1" "email2"
RPOP queue:emails

# Sets
SADD user:1:friends "user2" "user3"
SISMEMBER user:1:friends "user2"

# Sorted Sets
ZADD leaderboard 100 "Alice" 85 "Bob"
ZREVRANGE leaderboard 0 2 WITHSCORES
```

---

### Caching Patterns

#### Cache-Aside (Lazy Loading)

```javascript
async function getUser(id) {
    const cacheKey = `user:${id}`;
    const cached = await redis.get(cacheKey);
    if (cached) return JSON.parse(cached);
    
    const user = await User.findById(id);
    await redis.set(cacheKey, JSON.stringify(user), 'EX', 3600);
    return user;
}

async function updateUser(id, data) {
    const user = await User.findByIdAndUpdate(id, data, { new: true });
    await redis.del(`user:${id}`);
    return user;
}
```

#### Write-Through

```javascript
async function createUser(userData) {
    const user = await User.create(userData);
    await redis.set(`user:${user.id}`, JSON.stringify(user), 'EX', 3600);
    return user;
}
```

#### Write-Behind

```javascript
async function updateProfile(userId, data) {
    await redis.set(`user:${userId}`, JSON.stringify(data), 'EX', 3600);
    await messageQueue.publish('user.update', { userId, data });
    return data;
}
```

---

### Cache Invalidation

```javascript
// TTL-based
await redis.set('user:1', data, 'EX', 3600);

// Manual invalidation
async function updateUser(id, data) {
    await User.findByIdAndUpdate(id, data);
    await redis.del(`user:${id}`);
}

// Stampede prevention
async function getUserWithLock(id) {
    const cacheKey = `user:${id}`;
    const lockKey = `lock:${cacheKey}`;
    
    let cached = await redis.get(cacheKey);
    if (cached) return JSON.parse(cached);
    
    const acquired = await redis.set(lockKey, '1', 'NX', 'EX', 5);
    if (acquired) {
        const user = await User.findById(id);
        await redis.set(cacheKey, JSON.stringify(user), 'EX', 3600);
        await redis.del(lockKey);
        return user;
    } else {
        await new Promise(r => setTimeout(r, 100));
        return getUserWithLock(id);
    }
}
```

---

### Interview Questions (30+)

**Q1: What is caching and why use it?**
A: "Storing frequently accessed data in faster storage (memory). Reduces database load, improves response times. Cache hit rates of 80-95% are common for well-designed systems."

**Q2: What's the difference between cache-aside and write-through?**
A: "Cache-aside: app manages cache, loads on miss, invalidates on write. Write-through: write to cache and database simultaneously. Cache-aside is simpler; write-through ensures consistency."

**Q3: What is Redis?**
A: "In-memory data store supporting strings, hashes, lists, sets, sorted sets. Used for caching, session storage, real-time analytics, pub/sub. Supports persistence (RDB, AOF)."

**Q4: How do you handle cache invalidation?**
A: "TTL for infrequently changing data. Manual invalidation on write for critical data. Event-based for microservices. The key: stale cache is acceptable for many use cases."

**Q5: What's a cache stampede?**
A: "Many requests hit database when cache expires. Prevention: locking (only one request updates cache), early refresh (update before expiry), probabilistic expiration."

**Q6: What is cache penetration?**
A: "Requests for non-existent data bypass cache. Prevention: cache null values with short TTL, bloom filter to check existence first."

**Q7: What is cache breakdown?**
A: "Hot key expires causing many requests. Prevention: set different TTLs for different keys, use distributed locks, never expire critical keys."

**Q8: What is cache avalanche?**
A: "Many keys expire at same time. Prevention: randomize TTLs, use different cache layers, implement circuit breakers."

**Q9: When would you use Redis vs Memcached?**
A: "Redis: rich data types, persistence, pub/sub, Lua scripting. Memcached: simple key-value, multi-threaded, slightly faster for basic caching. Redis for most cases."

**Q10: How do you implement distributed caching?**
A: "Use Redis Cluster or Memcached. Consistent hashing for key distribution. Handle node failures gracefully. Consider cache-aside pattern for consistency."

**Q11: What is write-behind caching?**
A: "Write to cache immediately, database asynchronously. Fast writes but risk of data loss. Use message queue for reliable async updates. Good for high-write workloads."

**Q12: How do you handle cache consistency?**
A: "Cache-aside with invalidation. Write-through for strong consistency. Event-based invalidation for microservices. Choose based on consistency requirements."

**Q13: What is Redis pub/sub?**
A: "Publish/subscribe messaging. Publisher sends to channel, all subscribers receive. Use for real-time notifications, chat, event broadcasting. Not for reliable message delivery."

**Q14: How do you monitor cache performance?**
A: "Track hit rate, miss rate, latency. Monitor memory usage, evictions. Tools: Redis INFO command, Prometheus, Grafana. Alert on low hit rates."

**Q15: What is Redis persistence?**
A: "RDB: periodic snapshots (fast recovery, data loss risk). AOF: log every write (slower, more durable). Can use both. Configure based on durability requirements."

**Q16: How do you handle cache in microservices?**
A: "Each service owns its cache. Event-based invalidation across services. API gateway caching. Consider distributed cache (Redis) for shared data."

**Q17: What is cache warming?**
A: "Pre-populate cache before it's needed. Load frequently accessed data on startup. Reduces cold start latency. Use background jobs to refresh."

**Q18: How do you implement session storage with Redis?**
A: "Store session data with session ID as key. Set appropriate TTL. Use connect-redis for Express. Handle session serialization/deserialization."

**Q19: What is Redis Cluster?**
A: "Distributed Redis across multiple nodes. Automatic sharding with hash slots. High availability with replicas. Handles node failures gracefully."

**Q20: How do you handle large objects in cache?**
A: "Compress before storing. Split into smaller chunks. Use appropriate serialization (MessagePack vs JSON). Consider cache size limits."

**Q21: What is Redis Sentinel?**
A: "High availability for Redis. Monitors master/slave instances. Automatic failover. Client discovery of new master. Use for production deployments."

**Q22: How do you implement rate limiting with Redis?**
A: "Track requests per client in Redis. Use sliding window algorithm. INCR with EXPIRE for fixed window. Sorted sets for sliding window."

**Q23: What is cache aside vs read through?**
A: "Cache-aside: app manages cache explicitly. Read-through: cache itself loads from database on miss. Cache-aside gives more control; read-through is simpler."

**Q24: How do you handle cache in serverless?**
A: "Use Redis (ElastiCache) or DAX (DynamoDB). Lambda functions connect to cache. Consider cold start impact. Use provisioned concurrency for critical paths."

**Q25: What is multi-level caching?**
A: "Multiple cache layers: in-process (fastest), distributed (Redis), CDN. Each level has different TTL and size. Reduces load on database significantly."

**Q26: How do you test caching logic?**
A: "Mock Redis in tests. Test cache hit/miss scenarios. Test invalidation logic. Integration tests with real Redis. Load test cache performance."

**Q27: What is cache eviction policy?**
A: "LRU (Least Recently Used), LFU (Least Frequently Used), FIFO, Random. Configure based on access patterns. Redis uses LRU by default."

**Q28: How do you handle cache in distributed systems?**
A: "Consistent hashing for key distribution. Cache-aside with event-based invalidation. Handle network partitions gracefully. Consider eventual consistency."

**Q29: What is Redis transactions?**
A: "MULTI/EXEC for atomic operations. WATCH for optimistic locking. Pipeline for batch operations. Not true ACID transactions."

**Q30: How do you implement caching in GraphQL?**
A: "DataLoader for per-request caching. Response caching at gateway level. Client-side caching (Apollo Client). Persisted queries for CDN caching."

---

### Complete Redis Implementation

```javascript
const Redis = require('ioredis');

const redis = new Redis({
    host: process.env.REDIS_HOST,
    port: process.env.REDIS_PORT,
    password: process.env.REDIS_PASSWORD,
    retryStrategy: (times) => Math.min(times * 50, 2000)
});

// Cache Manager Class
class CacheManager {
    constructor(redis, defaultTTL = 3600) {
        this.redis = redis;
        this.defaultTTL = defaultTTL;
    }
    
    async get(key) {
        const cached = await this.redis.get(key);
        return cached ? JSON.parse(cached) : null;
    }
    
    async set(key, value, ttl = this.defaultTTL) {
        await this.redis.set(key, JSON.stringify(value), 'EX', ttl);
    }
    
    async delete(key) {
        await this.redis.del(key);
    }
    
    async getOrSet(key, fetchFn, ttl = this.defaultTTL) {
        const cached = await this.get(key);
        if (cached) return cached;
        
        const value = await fetchFn();
        await this.set(key, value, ttl);
        return value;
    }
    
    async invalidatePattern(pattern) {
        const keys = await this.redis.keys(pattern);
        if (keys.length > 0) {
            await this.redis.del(...keys);
        }
    }
}

// Rate Limiter
class RateLimiter {
    constructor(redis) {
        this.redis = redis;
    }
    
    async isAllowed(key, limit, window) {
        const current = await this.redis.incr(key);
        if (current === 1) {
            await this.redis.expire(key, window);
        }
        return current <= limit;
    }
    
    async slidingWindow(key, limit, window) {
        const now = Date.now();
        const windowStart = now - window * 1000;
        
        await this.redis.zremrangebyscore(key, 0, windowStart);
        const count = await this.redis.zcard(key);
        
        if (count >= limit) {
            return false;
        }
        
        await this.redis.zadd(key, now, `${now}-${Math.random()}`);
        await this.redis.expire(key, window);
        return true;
    }
}

// Session Store
class SessionStore {
    constructor(redis) {
        this.redis = redis;
        this.prefix = 'session:';
    }
    
    async create(userId, data, ttl = 86400) {
        const sessionId = require('crypto').randomBytes(32).toString('hex');
        await this.redis.set(
            `${this.prefix}${sessionId}`,
            JSON.stringify({ userId, ...data }),
            'EX',
            ttl
        );
        return sessionId;
    }
    
    async get(sessionId) {
        const data = await this.redis.get(`${this.prefix}${sessionId}`);
        return data ? JSON.parse(data) : null;
    }
    
    async destroy(sessionId) {
        await this.redis.del(`${this.prefix}${sessionId}`);
    }
}
```

---

### Additional Interview Questions (15+)

**Q31: How do you implement cache-aside pattern with Redis?**
A: "Check cache first. On miss, fetch from DB, store in cache with TTL. On write, update DB, invalidate cache. Use getOrSet helper for convenience."

**Q32: What is Redis pipelining?**
A: "Send multiple commands without waiting for responses. Reduces network round trips. 10x performance improvement for batch operations. Use pipeline() method."

**Q33: How do you handle Redis connection failures?**
A: "Retry strategy with exponential backoff. Connection pooling. Health checks. Circuit breaker for Redis. Fallback to database on failure."

**Q34: What is Redis Lua scripting?**
A: "Execute Lua scripts atomically on Redis. Use for complex operations (compare-and-set, rate limiting). EVAL command. Scripts cached for reuse."

**Q35: How do you implement distributed locks with Redis?**
A: "SET with NX and EX options. Use Redlock algorithm for distributed systems. Release lock with Lua script (check owner). Libraries: redlock, ioredis-lock."

**Q36: What is Redis Streams?**
A: "Log data structure for event sourcing. Append-only. Consumer groups for parallel processing. Message acknowledgment. Use for event streaming."

**Q37: How do you implement caching with TTL jitter?**
A: "Add random time to TTL to prevent cache avalanche. TTL = base_ttl + random(0, jitter). Prevents many keys expiring at same time."

**Q38: What is Redis memory optimization?**
A: "Use appropriate data structures. Compress large values. Set maxmemory-policy. Use hash encoding for small objects. Monitor with INFO memory."

**Q39: How do you implement cache warming in production?**
A: "Background job loads frequently accessed data on startup. Use batch operations for efficiency. Monitor cache hit rate after warming."

**Q40: What is Redis Cluster vs Sentinel?**
A: "Cluster: horizontal scaling, automatic sharding. Sentinel: high availability, automatic failover. Use Cluster for scaling, Sentinel for HA."

**Q41: How do you handle cache in multi-tenant applications?**
A: "Prefix cache keys with tenant ID. Separate Redis databases per tenant. Or use namespacing. Configure per-tenant TTLs."

**Q42: What is cache aside with write-through hybrid?**
A: "Read: cache-aside pattern. Write: write-through to cache + async write to DB. Balances consistency and performance."

**Q43: How do you implement caching for GraphQL resolvers?**
A: "DataLoader for per-request batching. Response caching at gateway. Field-level caching. Cache invalidation on mutation."

**Q44: What is Redis geo commands?**
A: "GEOADD, GEODIST, GEORADIUS for location-based queries. Use for nearby search, geofencing. Backed by sorted sets."

**Q45: How do you monitor Redis in production?**
A: "INFO command for metrics. Prometheus exporter. Grafana dashboards. Alert on memory, connections, latency. Tools: RedisInsight, redis-cli."

---

*Next: [05 — Message Queues & Event-Driven](https://github.com/Aditya-myst/Interview-PlayBook/blob/main/BACKEND/Messages_queue.md)*
