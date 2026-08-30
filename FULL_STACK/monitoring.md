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

### Additional Interview Questions (25+)

**Q4: What is distributed tracing?**
A: "Track requests across multiple services. Propagate trace ID. Tools: Jaeger, Zipkin, AWS X-Ray. Visualize request flow. Identify bottlenecks."

**Q5: How do you implement logging best practices?**
A: "Structured logging (JSON). Log levels (error, warn, info, debug). Include context (request ID, user ID). Don't log sensitive data. Centralized logging."

**Q6: What is alerting and how do you set it up?**
A: "Notify on anomalies. Tools: PagerDuty, OpsGenie, Grafana alerts. Set thresholds. Escalation policies. On-call rotation."

**Q7: How do you monitor database performance?**
A: "Query performance. Connection pool usage. Slow query log. Tools: pg_stat_statements, Prometheus exporters."

**Q8: What is APM (Application Performance Monitoring)?**
A: "Tools to monitor application performance. Track response times, errors, database queries. Examples: New Relic, Datadog, Sentry."

**Q9: How do you handle error tracking?**
A: "Sentry for exception monitoring. Capture stack traces. Group similar errors. Alert on new errors. Track error trends."

**Q10: What is SLA/SLO/SLI?**
A: "SLI: Service Level Indicator (metric). SLO: Service Level Objective (target). SLA: Service Level Agreement (contract). Example: SLI=99.9% uptime, SLO=99.95%, SLA=99.9%."

**Q11: How do you monitor frontend performance?**
A: "Core Web Vitals (LCP, FID, CLS). Lighthouse scores. Real User Monitoring (RUM). Tools: Sentry, LogRocket, Datadog RUM."

**Q12: What is log aggregation?**
A: "Collect logs from multiple sources. Centralize in one place. Search and analyze. Tools: ELK Stack, Splunk, Loki."

**Q13: How do you handle high-cardinality metrics?**
A: "Use labels carefully. Avoid high-cardinality labels (user IDs). Aggregate before storing. Use sampling for traces."

**Q14: What is observability vs monitoring?**
A: "Monitoring: track known metrics. Observability: understand unknown issues. Monitoring tells you what; observability tells you why."

**Q15: How do you implement health checks?**
A: "/health endpoint. Check database, Redis, external services. Return status codes. Include uptime, version. Use for load balancer health checks."

**Q16: What is incident management?**
A: "Process for handling production issues. Detection, response, resolution, post-mortem. Tools: PagerDuty, OpsGenie."

**Q17: How do you monitor costs?**
A: "Track cloud spending. Set budgets. Alert on anomalies. Tools: AWS Cost Explorer, GCP Billing. Optimize resources."

**Q18: What is chaos engineering?**
A: "Intentionally introduce failures to test resilience. Tools: Chaos Monkey, Gremlin. Build confidence in system reliability."

**Q19: How do you handle log rotation?**
A: "Rotate logs to prevent disk full. Compress old logs. Set retention policy. Tools: logrotate, Docker log drivers."

**Q20: What is synthetic monitoring?**
A: "Simulate user interactions. Test critical paths regularly. Alert on failures. Tools: Pingdom, Checkly, Datadog Synthetics."

**Q21: How do you monitor microservices?**
A: "Distributed tracing. Centralized logging. Metrics per service. Health checks. Service mesh observability."

**Q22: What is dashboard best practices?**
A: "Key metrics at a glance. Drill-down capability. Historical trends. Alert status. Tools: Grafana, Datadog, Kibana."

**Q23: How do you handle alert fatigue?**
A: "Set meaningful thresholds. Reduce noise. Group related alerts. Escalation policies. Regular review and tuning."

**Q24: What is observability pipeline?**
A: "Collect, process, store, analyze telemetry data. Tools: OpenTelemetry, Fluentd, Vector. Route data to multiple backends."

**Q25: How do you implement distributed tracing?**
A: "Propagate trace ID in headers. Use Jaeger, Zipkin, or AWS X-Ray. Add spans for each service call. Visualize request flow."

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
