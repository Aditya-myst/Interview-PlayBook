# 13 — Node.js Deep Dive

## Event Loop, Streams, Clustering — 30+ Interview Questions

---

### Event Loop

```javascript
console.log('1');              // Call stack

setTimeout(() => {
    console.log('2');          // Macrotask
}, 0);

Promise.resolve().then(() => {
    console.log('3');          // Microtask
});

process.nextTick(() => {
    console.log('4');          // Microtask (highest priority)
});

console.log('5');              // Call stack

// Output: 1, 5, 4, 3, 2
```

---

### Streams

```javascript
const fs = require('fs');

// Readable stream
const readable = fs.createReadStream('large-file.txt');
readable.on('data', (chunk) => console.log(`Received ${chunk.length} bytes`));

// Writable stream
const writable = fs.createWriteStream('output.txt');
writable.write('Hello');
writable.end();

// Piping
readable.pipe(transform).pipe(writable);
```

---

### Clustering

```javascript
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
    for (let i = 0; i < numCPUs; i++) cluster.fork();
    cluster.on('exit', (worker) => {
        console.log(`Worker ${worker.process.pid} died`);
        cluster.fork();
    });
} else {
    require('./app').listen(3000);
}
```

---

### Interview Questions (30+)

**Q1: How does the Node.js event loop work?**
A: "Single-threaded event loop handles async operations. Processes: call stack (sync), microtask queue (Promises, nextTick), macrotask queue (setTimeout, I/O). Microtasks have priority."

**Q2: What is the difference between process.nextTick and Promise?**
A: "process.nextTick: executes before any other microtask. Promise: executes after nextTick but before setTimeout. nextTick can starve I/O if overused."

**Q3: When would you use clustering?**
A: "To utilize multiple CPU cores. Each cluster runs in separate process with own memory. Use PM2 for production: pm2 start app.js -i max."

**Q4: What are streams in Node.js?**
A: "Handle data in chunks. Types: readable, writable, transform, duplex. Use for large files, network operations. Pipe for composition."

**Q5: What is the difference between spawn, exec, and fork?**
A: "spawn: new process with streams. exec: buffer output, shell. fork: new Node.js process with IPC. Use spawn for large output."

**Q6: How do you handle errors in Node.js?**
A: "Try/catch for sync. .catch() for promises. Error event for streams. Unhandled rejection handler. Process exit handler."

**Q7: What is the module system in Node.js?**
A: "CommonJS (require/module.exports) and ES Modules (import/export). CommonJS synchronous, ES Modules async. Node supports both."

**Q8: How do you handle memory leaks?**
A: "Heap snapshots. Monitor memory usage. Common causes: event listeners, closures, global variables. Tools: clinic.js, heapdump."

**Q9: What is middleware in Express?**
A: "Functions that execute during request-response cycle. Access req/res/next. Used for auth, logging, error handling. Order matters."

**Q10: How do you handle async operations?**
A: "Callbacks (old). Promises (better). Async/await (best). Handle errors with try/catch. Use Promise.all for parallel."

**Q11: What is the buffer in Node.js?**
A: "Handle binary data. Used for I/O operations, network protocols. Buffer.from(), Buffer.alloc(). Not resizable."

**Q12: How do you implement caching in Node.js?**
A: "Redis for distributed cache. In-memory (node-cache) for local. HTTP caching headers. Response caching middleware."

**Q13: What is worker_threads?**
A: "Run JavaScript in parallel threads. Share memory with SharedArrayBuffer. Use for CPU-intensive tasks. Different from clustering."

**Q14: How do you handle file uploads?**
A: "Multer middleware. Stream to disk or cloud. Validate file type and size. Handle large files with streams."

**Q15: What is the difference between Node.js and browser JavaScript?**
A: "Node.js: server-side, file system access, no DOM, CommonJS/ESM. Browser: client-side, DOM, Web APIs, ES Modules."

**Q16: How do you secure a Node.js application?**
A: "Helmet for security headers. CORS configuration. Input validation. Rate limiting. Dependency scanning. Environment variables."

**Q17: What is garbage collection in Node.js?**
A: "V8 engine automatically frees memory. Generational GC (young/old). Mark-and-sweep. Avoid memory leaks."

**Q18: How do you handle database connections?**
A: "Connection pooling (pg-pool, mongoose). Connection timeout. Retry logic. Health checks. Graceful shutdown."

**Q19: What is middleware chaining?**
A: "Multiple middleware functions for same route. Execute in order. Call next() to continue. Error middleware with 4 params."

**Q20: How do you implement logging?**
A: "Winston or Pino for structured logging. Log levels (error, warn, info, debug). Request logging middleware. Centralized logging."

**Q21: What is graceful shutdown?**
A: "Handle SIGTERM/SIGINT. Close server, database connections. Finish pending requests. Exit cleanly."

**Q22: How do you handle configuration?**
A: "Environment variables. Config files per environment. dotenv for development. Validate config on startup."

**Q23: What is the difference between spawn and fork?**
A: "spawn: new process with streams. fork: new Node.js process with IPC channel. Use fork for Node.js child processes."

**Q24: How do you implement rate limiting?**
A: "express-rate-limit middleware. Redis store for distributed. Different limits per endpoint. Return 429 with Retry-After."

**Q25: What is clustering vs load balancing?**
A: "Clustering: multiple Node.js processes on same machine. Load balancing: distribute across machines. Use both for production."

**Q26: How do you handle WebSocket in Node.js?**
A: "ws or socket.io library. Handle connections, messages, disconnections. Scale with Redis adapter. Authentication middleware."

**Q27: What is the stream pipeline?**
A: "Connect streams with pipe(). Handle errors with pipeline(). Transform data through chain. Memory efficient."

**Q28: How do you optimize Node.js performance?**
A: "Clustering. Caching. Compression. Connection pooling. Async operations. Avoid blocking. Profile with clinic.js."

**Q29: What is dependency injection in Node.js?**
A: "Provide dependencies from outside. Improves testability. Libraries: awilix, tsyringe. Common in NestJS."

**Q30: How do you implement health checks?**
A: "/health endpoint. Check database, Redis, external services. Return status codes. Monitor uptime. Tools: PM2, Kubernetes."

---

### Complete Node.js Implementation

```javascript
const express = require('express');
const compression = require('compression');
const helmet = require('helmet');
const cors = require('cors');
const { Pool } = require('pg');
const Redis = require('ioredis');
const winston = require('winston');

// Logger setup
const logger = winston.createLogger({
    level: 'info',
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json()
    ),
    transports: [
        new winston.transports.File({ filename: 'error.log', level: 'error' }),
        new winston.transports.File({ filename: 'combined.log' })
    ]
});

if (process.env.NODE_ENV !== 'production') {
    logger.add(new winston.transports.Console({ format: winston.format.simple() }));
}

// Database connection pool
const pool = new Pool({
    host: process.env.DB_HOST,
    port: process.env.DB_PORT,
    database: process.env.DB_NAME,
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    max: 20,
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 2000
});

// Redis connection
const redis = new Redis({
    host: process.env.REDIS_HOST,
    port: process.env.REDIS_PORT,
    retryStrategy: (times) => Math.min(times * 50, 2000)
});

// Express app
const app = express();

// Middleware
app.use(helmet());
app.use(cors());
app.use(compression());
app.use(express.json({ limit: '10mb' }));

// Request logging
app.use((req, res, next) => {
    const start = Date.now();
    res.on('finish', () => {
        logger.info({
            method: req.method,
            path: req.path,
            status: res.statusCode,
            duration: Date.now() - start,
            ip: req.ip
        });
    });
    next();
});

// Health check
app.get('/health', async (req, res) => {
    try {
        await pool.query('SELECT 1');
        await redis.ping();
        res.json({ status: 'healthy', timestamp: new Date().toISOString() });
    } catch (error) {
        res.status(503).json({ status: 'unhealthy', error: error.message });
    }
});

// Graceful shutdown
process.on('SIGTERM', async () => {
    logger.info('SIGTERM received, shutting down gracefully');
    await pool.end();
    await redis.quit();
    process.exit(0);
});

process.on('unhandledRejection', (reason, promise) => {
    logger.error('Unhandled Rejection:', { reason, promise });
});

app.listen(3000, () => {
    logger.info('Server running on port 3000');
});
```

---

### Additional Interview Questions (20+)

**Q31: How do you implement graceful shutdown?**
A: "Handle SIGTERM/SIGINT. Close server, database connections, Redis. Finish pending requests. Exit cleanly. Use process.on('SIGTERM')."

**Q32: What is Node.js event loop phases?**
A: "Timers (setTimeout, setInterval), I/O callbacks, idle/prepare, poll (I/O), check (setImmediate), close callbacks. Microtasks between phases."

**Q33: How do you handle unhandled rejections?**
A: "process.on('unhandledRejection'). Log error. Exit process in production. Use --unhandled-rejections=throw in Node 15+."

**Q34: What is Node.js clustering best practices?**
A: "Use PM2 for production. Fork based on CPU cores. Handle worker exit. Share state with Redis. Don't share in-memory state."

**Q35: How do you implement streaming responses?**
A: "res.write() for chunked responses. Use for large datasets. Transform streams for data processing. Pipe from file to response."

**Q36: What is Node.js worker threads?**
A: "Run JavaScript in parallel threads. Share memory with SharedArrayBuffer. Use for CPU-intensive tasks. Different from clustering."

**Q37: How do you handle database transactions?**
A: "BEGIN, COMMIT, ROLLBACK with pg client. Use try/catch. Release client in finally. Use connection pool."

**Q38: What is Express middleware order?**
A: "Execute in order defined. Global middleware first. Route-specific middleware next. Error middleware last (4 params)."

**Q39: How do you implement WebSocket authentication?**
A: "Verify JWT on connection. Store user info in socket. Middleware for WebSocket. Handle disconnection."

**Q40: What is Node.js memory management?**
A: "V8 garbage collection. Generational GC (young/old). Avoid memory leaks. Monitor with process.memoryUsage()."

**Q41: How do you optimize Express performance?**
A: "Compression. Caching. Connection pooling. Async operations. Avoid blocking. Use cluster module."

**Q42: What is Node.js streams best practices?**
A: "Use for large data. Handle backpressure. Pipe for composition. Error handling with pipeline(). Use transform streams."

**Q43: How do you implement request validation?**
A: "express-validator, Joi, Yup. Validate at middleware level. Return detailed errors. Validate body, params, query."

**Q44: What is Node.js security best practices?**
A: "Helmet for headers. CORS configuration. Input validation. Rate limiting. Dependency scanning. Environment variables."

**Q45: How do you handle configuration management?**
A: "Environment variables. dotenv for development. Validate on startup. Different configs per environment. Secret managers."

---

*Next: [14 — Docker & Deployment](14-Docker-Deploy.md)*
