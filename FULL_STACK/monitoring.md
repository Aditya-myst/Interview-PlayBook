# 08 — Monitoring & Observability

## Keeping Production Healthy

---

### The Three Pillars of Observability

```
┌─────────────────────────────────────────────────────────┐
│                 Observability                            │
├─────────────────┬─────────────────┬─────────────────────┤
│     Logs        │    Metrics      │    Traces           │
│  (What happened)│  (How's it     │  (Request journey)  │
│                 │   performing?)  │                     │
├─────────────────┼─────────────────┼─────────────────────┤
│ - Error logs    │ - Response time │ - Request flow      │
│ - Access logs   │ - Error rate    │ - Service-to-service│
│ - Application   │ - Throughput    │ - Bottleneck        │
│   logs          │ - CPU/Memory    │   identification    │
└─────────────────┴─────────────────┴─────────────────────┘
```

---

### Logging

```javascript
const winston = require('winston');

const logger = winston.createLogger({
    level: process.env.LOG_LEVEL || 'info',
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json()
    ),
    transports: [
        new winston.transports.File({ filename: 'error.log', level: 'error' }),
        new winston.transports.File({ filename: 'combined.log' }),
    ],
});

if (process.env.NODE_ENV !== 'production') {
    logger.add(new winston.transports.Console({
        format: winston.format.simple()
    }));
}

// Usage
logger.info('User created', { userId: 123, email: 'alice@example.com' });
logger.error('Database connection failed', { error: err.message });
logger.warn('High memory usage', { usage: '85%' });
```

---

### Health Checks

```javascript
// Express health endpoint
app.get('/health', (req, res) => {
    res.json({
        status: 'healthy',
        timestamp: new Date().toISOString(),
        uptime: process.uptime(),
        version: process.env.APP_VERSION || '1.0.0'
    });
});

// Detailed health check
app.get('/health/detailed', async (req, res) => {
    const checks = {
        database: await checkDatabase(),
        redis: await checkRedis(),
        memory: checkMemory(),
    };
    
    const isHealthy = Object.values(checks).every(c => c.status === 'ok');
    
    res.status(isHealthy ? 200 : 503).json({
        status: isHealthy ? 'healthy' : 'unhealthy',
        checks
    });
});

async function checkDatabase() {
    try {
        await db.query('SELECT 1');
        return { status: 'ok' };
    } catch (error) {
        return { status: 'error', message: error.message };
    }
}

function checkMemory() {
    const usage = process.memoryUsage();
    return {
        status: 'ok',
        heapUsed: `${Math.round(usage.heapUsed / 1024 / 1024)}MB`,
        heapTotal: `${Math.round(usage.heapTotal / 1024 / 1024)}MB`
    };
}
```

---

### Monitoring Tools

| Tool | Type | Use Case |
|------|------|----------|
| **Prometheus** | Metrics | Collecting and querying metrics |
| **Grafana** | Visualization | Dashboards and alerts |
| **Datadog** | All-in-one | Logs, metrics, traces |
| **New Relic** | APM | Application performance |
| **Sentry** | Error tracking | Exception monitoring |
| **ELK Stack** | Logging | Elasticsearch, Logstash, Kibana |
| **PagerDuty** | Alerting | Incident management |

---

### Key Metrics to Monitor

```javascript
// Response time middleware
app.use((req, res, next) => {
    const start = Date.now();
    
    res.on('finish', () => {
        const duration = Date.now() - start;
        logger.info('Request completed', {
            method: req.method,
            path: req.path,
            status: res.statusCode,
            duration: `${duration}ms`
        });
    });
    
    next();
});
```

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| **Response Time** | How fast requests complete | > 500ms |
| **Error Rate** | % of failed requests | > 1% |
| **Throughput** | Requests per second | Depends on capacity |
| **CPU Usage** | Server CPU utilization | > 80% |
| **Memory Usage** | Server memory utilization | > 85% |
| **Disk Usage** | Storage utilization | > 90% |
| **Database Connections** | Active DB connections | Near pool limit |

---

### Interview Questions

**Q: What is observability?**

A: "Understanding what's happening inside a system from its external outputs. Three pillars: logs (what happened), metrics (how's performance), traces (request journey). Tools: Prometheus, Grafana, Datadog, Sentry."

**Q: What's the difference between monitoring and observability?**

A: "Monitoring: tracking known metrics (CPU, memory, error rate). Observability: ability to understand unknown issues by exploring data. Monitoring tells you something is wrong; observability helps you understand why."

**Q: What should you monitor in production?**

A: "1) Health checks (is service up?). 2) Response time (is it fast?). 3) Error rate (is it failing?). 4) Resource usage (CPU, memory, disk). 5) Business metrics (signups, orders). Set up alerts for anomalies."

---

### Full Stack Architecture Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                      Full Stack Application                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Frontend (React/Next.js)                                       │
│  ├── Components, State Management, Routing                      │
│  ├── API Integration (fetch/axios/React Query)                  │
│  └── Deployed to Vercel/Netlify                                 │
│                                                                 │
│  Backend (Node.js/Express)                                      │
│  ├── Routes, Controllers, Services, Models                      │
│  ├── Authentication (JWT, OAuth)                                │
│  ├── Middleware (auth, validation, error handling)               │
│  └── Deployed to Railway/EC2 with Docker                        │
│                                                                 │
│  Database (PostgreSQL/MongoDB)                                  │
│  ├── Schema design, Indexing, Optimization                      │
│  └── Managed service (AWS RDS, Railway)                         │
│                                                                 │
│  Infrastructure                                                 │
│  ├── Docker for containerization                                │
│  ├── CI/CD with GitHub Actions                                  │
│  ├── Monitoring with Prometheus/Grafana                         │
│  └── Error tracking with Sentry                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

*Good luck with your full stack interviews!*
