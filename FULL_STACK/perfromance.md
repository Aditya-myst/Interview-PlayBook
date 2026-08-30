# 10 — Performance Optimization

## Frontend & Backend Performance — 25+ Interview Questions

---

### Frontend Performance

```javascript
// 1. Code Splitting
const Dashboard = React.lazy(() => import('./Dashboard'));

function App() {
    return (
        <Suspense fallback={<Loading />}>
            <Dashboard />
        </Suspense>
    );
}

// 2. Lazy Loading Images
<img src="photo.jpg" loading="lazy" alt="Photo" />

// 3. Memoization
const MemoizedComponent = React.memo(function Component({ data }) {
    return <div>{data.name}</div>;
});

// 4. useMemo for expensive computations
const sortedData = useMemo(() => {
    return [...data].sort((a, b) => a.name.localeCompare(b.name));
}, [data]);

// 5. useCallback for stable references
const handleClick = useCallback((id) => {
    console.log(id);
}, []);

// 6. Virtual Lists for large data
import { FixedSizeList } from 'react-window';

function VirtualList({ items }) {
    return (
        <FixedSizeList height={400} itemCount={items.length} itemSize={50}>
            {({ index, style }) => (
                <div style={style}>{items[index].name}</div>
            )}
        </FixedSizeList>
    );
}
```

---

### Backend Performance

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

// 3. Caching
const Redis = require('ioredis');
const redis = new Redis();

async function getUser(id) {
    const cached = await redis.get(`user:${id}`);
    if (cached) return JSON.parse(cached);
    
    const user = await User.findById(id);
    await redis.set(`user:${id}`, JSON.stringify(user), 'EX', 3600);
    return user;
}

// 4. Connection Pooling
const { Pool } = require('pg');
const pool = new Pool({ max: 20 });

// 5. Database Optimization
User.find().select('name email').limit(10).lean();

// 6. Parallel Operations
const [users, posts] = await Promise.all([User.find(), Post.find()]);

// 7. Pagination
const users = await User.find()
    .skip((page - 1) * limit)
    .limit(limit);
```

---

### Image Optimization

```html
<!-- Responsive images -->
<img 
    src="small.jpg" 
    srcset="small.jpg 480w, medium.jpg 768w, large.jpg 1200w"
    sizes="(max-width: 480px) 100vw, (max-width: 768px) 50vw, 33vw"
    alt="Responsive image"
    loading="lazy"
/>

<!-- WebP format -->
<picture>
    <source srcset="image.webp" type="image/webp">
    <img src="image.jpg" alt="Image">
</picture>
```

---

### Interview Questions (25+)

**Q1: How do you optimize a slow React application?**
A: "React.memo for expensive components. useMemo/useCallback for computations. Code splitting with React.lazy. Virtual lists for large data. Avoid unnecessary re-renders."

**Q2: What is code splitting?**
A: "Split code into smaller bundles. Load on demand with React.lazy. Reduces initial load time. Use for routes, heavy components."

**Q3: How do you optimize API responses?**
A: "Pagination. Field selection. Compression. Caching. Avoid over-fetching. Use GraphQL for flexible queries."

**Q4: What is lazy loading?**
A: "Load resources only when needed. Reduces initial load time. Use for images, components, data. Implement with intersection observer."

**Q5: How do you optimize database queries?**
A: "EXPLAIN to see plan. Add indexes. Avoid SELECT *. Avoid functions on indexed columns. Use LIMIT. Optimize JOINs."

**Q6: What is caching strategy?**
A: "Cache-aside: app manages. Write-through: cache + DB. Write-behind: cache first. Read-through: cache loads from DB."

**Q7: How do you handle high traffic?**
A: "Horizontal scaling. Load balancing. Caching. CDN for static assets. Database read replicas. Async processing."

**Q8: What is CDN?**
A: "Content Delivery Network. Caches content at edge locations. Reduces latency. Serves static assets."

**Q9: How do you optimize images?**
A: "Responsive images (srcset). Lazy loading. WebP format. CDN delivery. Image compression. Proper sizing."

**Q10: What is response compression?**
A: "Reduce response size with gzip/brotli. Use compression middleware. Reduces bandwidth, improves latency."

**Q11: How do you optimize Node.js performance?**
A: "Clustering. Caching. Compression. Connection pooling. Async operations. Avoid blocking. Profile with clinic.js."

**Q12: What is connection pooling?**
A: "Reuse database connections. Pool maintains ready connections. Configure max based on concurrent requests."

**Q13: How do you handle memory leaks?**
A: "Heap snapshots. Monitor memory usage. Common causes: event listeners, closures, global variables."

**Q14: What is database indexing?**
A: "Data structure for fast lookups. Speeds up SELECT, slows down INSERT/UPDATE. Choose indexes based on query patterns."

**Q15: How do you optimize static file serving?**
A: "CDN for delivery. Compression (gzip). Caching headers (Cache-Control, ETag). Minification."

**Q16: What is APM?**
A: "Application Performance Monitoring. Track response times, errors, database queries. Examples: New Relic, Datadog."

**Q17: How do you optimize for mobile?**
A: "Reduce payload size. Enable compression. Optimize images. Implement caching. Use CDN."

**Q18: What is tree shaking?**
A: "Remove unused code from bundle. Webpack, Rollup support. Reduces bundle size. Import specific functions."

**Q19: How do you handle large datasets?**
A: "Pagination. Virtual lists. Lazy loading. Database indexing. Caching. Streaming."

**Q20: What is database query optimization?**
A: "EXPLAIN to see plan. Add indexes. Avoid N+1 queries. Use LIMIT. Optimize JOINs."

**Q21: How do you optimize CSS?**
A: "Minification. Critical CSS inlining. Remove unused CSS. CSS modules for scoping."

**Q22: What is HTTP/2 optimization?**
A: "Multiplexing. Header compression. Server push. Single connection for multiple requests."

**Q23: How do you optimize JavaScript?**
A: "Minification. Tree shaking. Code splitting. Lazy loading. Avoid blocking operations."

**Q24: What is performance budget?**
A: "Set limits on metrics: bundle size, load time, Lighthouse score. Monitor and enforce. Tools: Webpack, Lighthouse CI."

**Q25: How do you measure performance?**
A: "Lighthouse for overall score. WebPageTest for detailed analysis. Chrome DevTools for profiling. Real User Monitoring."

---

*Next: [11 — Debugging & Troubleshooting](11-Debugging.md)*
