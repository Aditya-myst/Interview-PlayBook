# 11 — Debugging & Troubleshooting

## Finding and Fixing Issues — 20+ Interview Questions

---

### Frontend Debugging

```javascript
// 1. Console methods
console.log('Basic log');
console.warn('Warning');
console.error('Error');
console.table([{ name: 'Alice', age: 30 }]);
console.time('operation');
// ... code ...
console.timeEnd('operation');

// 2. React DevTools
// - Inspect component tree
// - View props and state
// - Profile re-renders
// - Identify performance issues

// 3. Network tab
// - View API requests
// - Check response status
// - Inspect headers
// - Measure load times

// 4. Performance tab
// - Record performance
// - Identify bottlenecks
// - View flame charts
// - Analyze paint times

// 5. Memory tab
// - Take heap snapshots
// - Identify memory leaks
// - Compare snapshots
```

---

### Backend Debugging

```javascript
// 1. Logging
const winston = require('winston');
const logger = winston.createLogger({
    level: 'debug',
    format: winston.format.json(),
    transports: [
        new winston.transports.File({ filename: 'error.log', level: 'error' }),
        new winston.transports.Console()
    ]
});

// 2. Error handling
app.use((err, req, res, next) => {
    logger.error({
        message: err.message,
        stack: err.stack,
        path: req.path,
        method: req.method
    });
    res.status(500).json({ error: 'Internal server error' });
});

// 3. Debug with Node.js
// node --inspect app.js
// chrome://inspect

// 4. Database debugging
// EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
// Slow query log
// pg_stat_statements

// 5. Memory profiling
// node --prof app.js
// node --prof-process isolate-*.log
```

---

### Common Issues and Solutions

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **Memory Leak** | Increasing memory usage | Heap snapshots, fix event listeners |
| **N+1 Query** | Slow database queries | Eager loading, DataLoader |
| **Race Condition** | Inconsistent data | Mutex, locks, atomic operations |
| **Deadlock** | System hangs | Lock ordering, timeouts |
| **CORS Error** | Blocked requests | Configure CORS middleware |
| **Auth Error** | 401/403 responses | Check token, refresh logic |
| **Timeout** | Request timeout | Increase timeout, optimize query |
| **Connection Pool Exhaustion** | Can't connect to DB | Increase pool size, close connections |

---

### Interview Questions (20+)

**Q1: How do you debug a slow API?**
A: "1) Profile to find bottleneck. 2) Check database queries (EXPLAIN). 3) Check for N+1 queries. 4) Monitor memory usage. 5) Check external service calls. 6) Use APM tools."

**Q2: How do you find memory leaks?**
A: "Heap snapshots. Compare snapshots over time. Look for growing objects. Common causes: event listeners, closures, global variables. Tools: clinic.js, Chrome DevTools."

**Q3: How do you debug production issues?**
A: "Check logs. Monitor metrics. Reproduce in staging. Use feature flags to disable features. Rollback if needed."

**Q4: What is distributed tracing?**
A: "Track requests across services. Propagate trace ID. Tools: Jaeger, Zipkin. Visualize request flow. Identify bottlenecks."

**Q5: How do you handle errors in production?**
A: "Log errors with context. Alert on critical errors. Track error rates. Use Sentry for exception monitoring. Post-mortem analysis."

**Q6: How do you debug database issues?**
A: "EXPLAIN for query plans. Slow query log. Connection pool monitoring. Index usage. pg_stat_statements."

**Q7: What is log analysis?**
A: "Search logs for patterns. Correlate errors with requests. Tools: ELK Stack, Splunk, Loki. Structured logging helps."

**Q8: How do you debug CORS issues?**
A: "Check browser console. Verify allowed origins. Check preflight requests. Configure CORS middleware properly."

**Q9: How do you debug authentication issues?**
A: "Check token validity. Verify token in request. Check token expiration. Test with fresh token. Check CORS for auth endpoints."

**Q10: What is APM?**
A: "Application Performance Monitoring. Track response times, errors, database queries. Examples: New Relic, Datadog, Sentry."

**Q11: How do you debug performance issues?**
A: "Profile CPU and memory. Identify bottlenecks. Check database queries. Monitor network requests. Use APM tools."

**Q12: What is chaos engineering?**
A: "Intentionally introduce failures to test resilience. Tools: Chaos Monkey, Gremlin. Build confidence in reliability."

**Q13: How do you handle production incidents?**
A: "Detect, respond, resolve, post-mortem. Alert on anomalies. Escalation policies. Communication. Learn from incidents."

**Q14: What is observability?**
A: "Understanding system behavior from external outputs. Three pillars: logs, metrics, traces. Tools: Prometheus, Grafana, Datadog."

**Q15: How do you debug WebSocket issues?**
A: "Check connection status. Monitor messages. Test with wscat. Check authentication. Handle reconnection."

**Q16: What is root cause analysis?**
A: "Find the underlying cause of an issue. Ask 'why' five times. Document findings. Implement fixes to prevent recurrence."

**Q17: How do you debug Docker containers?**
A: "docker logs for output. docker exec for shell access. docker inspect for configuration. Debug images with tools."

**Q18: What is log aggregation?**
A: "Collect logs from multiple sources. Centralize in one place. Search and analyze. Tools: ELK Stack, Splunk, Loki."

**Q19: How do you debug CI/CD pipelines?**
A: "Check pipeline logs. Run locally. Test each step. Check environment variables. Verify dependencies."

**Q20: What is debugging best practices?**
A: "Reproduce the issue. Isolate the problem. Check logs and metrics. Use debugging tools. Document findings."

---

*Next: [12 — Full Stack Interview Q&A](12-Interview-QA.md)*
