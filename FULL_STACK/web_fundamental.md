# 09 — Web Fundamentals

## HTTP, Browser, Networking — 30+ Interview Questions

---

### HTTP Protocol

```
Request:
GET /api/users HTTP/1.1
Host: example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
Accept: application/json

Response:
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=3600
ETag: "abc123"

{"users": [...]}
```

---

### HTTP Status Codes

```javascript
// 1xx Informational
100 Continue
101 Switching Protocols

// 2xx Success
200 OK                    // GET, PUT, PATCH successful
201 Created               // POST successful
202 Accepted              // Async processing started
204 No Content            // DELETE successful

// 3xx Redirection
301 Moved Permanently     // URL changed permanently
302 Found                 // Temporary redirect
304 Not Modified          // Cache is valid

// 4xx Client Error
400 Bad Request           // Invalid input
401 Unauthorized          // Not authenticated
403 Forbidden             // Not authorized
404 Not Found             // Resource doesn't exist
405 Method Not Allowed    // HTTP method not supported
409 Conflict              // Duplicate resource
422 Unprocessable Entity  // Validation failed
429 Too Many Requests     // Rate limited

// 5xx Server Error
500 Internal Server Error // Generic server error
502 Bad Gateway           // Upstream service failed
503 Service Unavailable   // Server overloaded
504 Gateway Timeout       // Upstream timeout
```

---

### Browser Rendering Pipeline

```
1. Parse HTML → DOM Tree
2. Parse CSS → CSSOM Tree
3. Combine DOM + CSSOM → Render Tree
4. Layout (Calculate positions and sizes)
5. Paint (Draw pixels to screen)
6. Composite (Layer management)

Critical Rendering Path:
HTML → DOM → CSSOM → Render Tree → Layout → Paint → Composite
```

---

### DNS Resolution

```
Browser Cache → OS Cache → Router → ISP DNS → Root Server → TLD Server → Authoritative Server

Types:
- A Record: domain → IPv4
- AAAA Record: domain → IPv6
- CNAME: domain → another domain
- MX: domain → mail server
- TXT: domain → text (verification)
```

---

### TCP/IP Model

```
Application Layer: HTTP, HTTPS, FTP, SMTP, DNS
Transport Layer: TCP, UDP
Network Layer: IP, ICMP
Data Link Layer: Ethernet, Wi-Fi

TCP Three-Way Handshake:
Client → Server: SYN
Server → Client: SYN-ACK
Client → Server: ACK
```

---

### CORS (Cross-Origin Resource Sharing)

```javascript
// Server configuration
app.use(cors({
    origin: ['https://myapp.com', 'https://www.myapp.com'],
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization'],
    credentials: true,
    maxAge: 86400
}));

// Preflight request (OPTIONS)
// Browser sends before actual request for:
// - Custom headers
// - Methods other than GET/POST
// - Content-Type other than form-data
```

---

### Caching Headers

```javascript
// Cache-Control
res.set('Cache-Control', 'public, max-age=3600');  // Cache for 1 hour
res.set('Cache-Control', 'no-cache');  // Revalidate before use
res.set('Cache-Control', 'no-store');  // Don't cache

// ETag
res.set('ETag', '"abc123"');
// Client sends: If-None-Match: "abc123"
// Server returns 304 if unchanged

// Last-Modified
res.set('Last-Modified', new Date().toUTCString());
// Client sends: If-Modified-Since: <date>
// Server returns 304 if unchanged
```

---

### Interview Questions (30+)

**Q1: What is HTTP and how does it work?**
A: "HyperText Transfer Protocol. Client sends request, server sends response. Stateless (no memory between requests). Methods: GET, POST, PUT, DELETE. Uses TCP for reliable delivery."

**Q2: What's the difference between HTTP and HTTPS?**
A: "HTTP: unencrypted, data in plain text. HTTPS: encrypted with TLS/SSL. HTTPS prevents eavesdropping, man-in-the-middle attacks. Use HTTPS for all production."

**Q3: What are HTTP methods and when would you use each?**
A: "GET: retrieve (idempotent, safe). POST: create (not idempotent). PUT: replace (idempotent). PATCH: partial update. DELETE: remove (idempotent)."

**Q4: What is the difference between HTTP/1.1 and HTTP/2?**
A: "HTTP/1.1: one request per connection. HTTP/2: multiplexing (multiple requests over single connection), header compression, server push. HTTP/2 is faster."

**Q5: What is DNS and how does it work?**
A: "Domain Name System: translates domain names to IP addresses. Browser cache → OS cache → recursive resolver → root server → TLD server → authoritative server."

**Q6: What is TCP three-way handshake?**
A: "SYN → SYN-ACK → ACK. Establishes reliable connection. Ensures both sides agree on sequence numbers. Happens before HTTP request."

**Q7: What is the difference between TCP and UDP?**
A: "TCP: reliable, ordered, connection-oriented. UDP: unreliable, unordered, connectionless. TCP for web, email. UDP for gaming, video, DNS."

**Q8: What is CORS and why is it needed?**
A: "Cross-Origin Resource Sharing. Browsers block requests to different domains. CORS allows servers to specify allowed origins. Preflight request for complex requests."

**Q9: What is a cookie and how does it work?**
A: "Small piece of data stored in browser. Sent with every request to same domain. Can be httpOnly (no JavaScript access), secure (HTTPS only), SameSite (CSRF protection)."

**Q10: What is the difference between cookies, localStorage, and sessionStorage?**
A: "Cookies: sent with requests, 4KB, can be httpOnly. localStorage: persists, 5-10MB, same origin. sessionStorage: cleared on tab close, 5-10MB."

**Q11: What is WebSocket?**
A: "Full-duplex communication over single TCP connection. Persistent connection. Server can push to client. Use for real-time: chat, live updates, gaming."

**Q12: What is Server-Sent Events (SSE)?**
A: "Server pushes updates to client over HTTP. Unidirectional (server to client). Simpler than WebSocket for one-way communication."

**Q13: What is a reverse proxy?**
A: "Server between clients and backend. Handles load balancing, SSL termination, caching, compression. Examples: Nginx, HAProxy."

**Q14: What is load balancing?**
A: "Distribute requests across servers. Algorithms: round robin, least connections, IP hash. Health checks. Improves availability."

**Q15: What is CDN?**
A: "Content Delivery Network. Caches content at edge locations. Reduces latency. Serves static assets. Examples: CloudFront, Cloudflare."

**Q16: What is browser caching?**
A: "Store responses locally. Cache-Control headers. ETag for validation. Last-Modified for conditional requests. Reduces server load."

**Q17: What is the critical rendering path?**
A: "HTML → DOM → CSSOM → Render Tree → Layout → Paint → Composite. Optimize by minimizing CSS, deferring JavaScript, inlining critical CSS."

**Q18: What is TLS handshake?**
A: "Establish secure connection. Server sends certificate. Client verifies. Agree on encryption keys. Happens after TCP handshake."

**Q19: What is HTTP/2 server push?**
A: "Server sends resources before client requests them. Push CSS, JavaScript with HTML. Reduces round trips. Use for critical resources."

**Q20: What is REST?**
A: "Representational State Transfer. Architectural style for APIs. Resources (nouns), HTTP methods (verbs), stateless, cacheable, uniform interface."

**Q21: What is GraphQL?**
A: "Query language for APIs. Single endpoint, client specifies fields. No over/under-fetching. Schema-based. Subscriptions for real-time."

**Q22: What is gRPC?**
A: "High-performance RPC framework. HTTP/2, Protocol Buffers. Binary format. Bidirectional streaming. Use for microservice communication."

**Q23: What is API versioning?**
A: "URL path (/v1/users), query parameter, header. Deprecation strategy. Backward compatibility. Maintain old versions."

**Q24: What is rate limiting?**
A: "Limit requests per client per time window. Prevents abuse. Implement with Redis. Return 429 when exceeded."

**Q25: What is idempotency?**
A: "Same request multiple times = same result. GET, PUT, DELETE idempotent. POST not idempotent. Important for retries."

**Q26: What is the difference between authentication and authorization?**
A: "Authentication: 'Who are you?' Authorization: 'What can you do?' Authentication first, then authorization."

**Q27: What is JWT?**
A: "JSON Web Token. Stateless authentication. Header, payload, signature. Server creates, client sends in Authorization header."

**Q28: What is OAuth 2.0?**
A: "Delegated authorization. User redirected to provider, approves, gets auth code, exchanges for access token."

**Q29: What is session-based authentication?**
A: "Server stores session data. Client has session ID in cookie. Server looks up session on each request. Can be invalidated server-side."

**Q30: What is CSRF?**
A: "Cross-Site Request Forgery. Trick user into making unwanted requests. Prevention: CSRF tokens, SameSite cookies, check Origin header."

---

*Next: [10 — Performance Optimization](10-Performance.md)*
