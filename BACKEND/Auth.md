# 03 — Authentication & Authorization

## JWT, OAuth, SSO — Security Fundamentals

---

### Authentication vs Authorization

| | Authentication | Authorization |
|---|---------------|---------------|
| **Question** | "Who are you?" | "What can you do?" |
| **Purpose** | Verify identity | Check permissions |
| **Example** | Login with email/password | Admin can delete users |
| **When** | Before authorization | After authentication |

---

### Authentication Methods

#### 1. Session-Based Authentication

```
Client                    Server
  │                          │
  │ POST /login              │
  │ {email, password}        │
  │─────────────────────────>│
  │                          │ Verify credentials
  │                          │ Create session
  │                          │ Store in session store (Redis)
  │ 200 OK                   │
  │ Set-Cookie: session=abc  │
  │<─────────────────────────│
  │                          │
  │ GET /dashboard           │
  │ Cookie: session=abc      │
  │─────────────────────────>│
  │                          │ Look up session
  │ 200 OK                   │
  │<─────────────────────────│
```

```javascript
// Express.js session
const session = require('express-session');
const RedisStore = require('connect-redis').default;
const redis = require('redis');

const redisClient = redis.createClient();
app.use(session({
    store: new RedisStore({ client: redisClient }),
    secret: 'your-secret-key',
    resave: false,
    saveUninitialized: false,
    cookie: { 
        secure: true,      // HTTPS only
        httpOnly: true,     // No JavaScript access
        maxAge: 24 * 60 * 60 * 1000  // 24 hours
    }
}));

app.post('/login', async (req, res) => {
    const user = await verifyCredentials(req.body.email, req.body.password);
    if (!user) return res.status(401).json({ error: 'Invalid credentials' });
    
    req.session.userId = user.id;
    res.json({ message: 'Logged in' });
});

app.get('/dashboard', (req, res) => {
    if (!req.session.userId) return res.status(401).json({ error: 'Not authenticated' });
    // User is authenticated
});
```

**Pros:** Server can invalidate sessions, simple to implement
**Cons:** Requires server-side storage, doesn't scale well horizontally

---

#### 2. JWT (JSON Web Tokens)

Stateless authentication—no server-side session storage.

```
Client                    Server
  │                          │
  │ POST /login              │
  │ {email, password}        │
  │─────────────────────────>│
  │                          │ Verify credentials
  │                          │ Generate JWT
  │ 200 OK                   │
  │ { token: "eyJhb..." }   │
  │<─────────────────────────│
  │                          │
  │ GET /dashboard           │
  │ Authorization: Bearer eyJhb...
  │─────────────────────────>│
  │                          │ Verify JWT signature
  │                          │ Extract user info from token
  │ 200 OK                   │
  │<─────────────────────────│
```

#### JWT Structure

```
Header.Payload.Signature

Header:
{
    "alg": "HS256",  // Algorithm
    "typ": "JWT"
}

Payload:
{
    "sub": "1234567890",  // Subject (user ID)
    "name": "John Doe",
    "email": "john@example.com",
    "role": "admin",
    "iat": 1516239022,    // Issued at
    "exp": 1516242622     // Expiration
}

Signature:
HMACSHA256(base64(header) + "." + base64(payload), secret)
```

#### JWT Implementation

```javascript
const jwt = require('jsonwebtoken');

const SECRET_KEY = process.env.JWT_SECRET;
const ACCESS_TOKEN_EXPIRY = '15m';
const REFRESH_TOKEN_EXPIRY = '7d';

// Generate tokens
function generateTokens(user) {
    const accessToken = jwt.sign(
        { userId: user.id, email: user.email, role: user.role },
        SECRET_KEY,
        { expiresIn: ACCESS_TOKEN_EXPIRY }
    );
    
    const refreshToken = jwt.sign(
        { userId: user.id },
        SECRET_KEY,
        { expiresIn: REFRESH_TOKEN_EXPIRY }
    );
    
    return { accessToken, refreshToken };
}

// Verify token middleware
function authenticate(req, res, next) {
    const authHeader = req.headers.authorization;
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
        return res.status(401).json({ error: 'No token provided' });
    }
    
    const token = authHeader.split(' ')[1];
    try {
        const decoded = jwt.verify(token, SECRET_KEY);
        req.user = decoded;
        next();
    } catch (err) {
        if (err.name === 'TokenExpiredError') {
            return res.status(401).json({ error: 'Token expired' });
        }
        return res.status(401).json({ error: 'Invalid token' });
    }
}

// Login endpoint
app.post('/login', async (req, res) => {
    const user = await verifyCredentials(req.body.email, req.body.password);
    if (!user) return res.status(401).json({ error: 'Invalid credentials' });
    
    const tokens = generateTokens(user);
    res.json(tokens);
});

// Refresh token endpoint
app.post('/refresh', async (req, res) => {
    const { refreshToken } = req.body;
    try {
        const decoded = jwt.verify(refreshToken, SECRET_KEY);
        const user = await getUserById(decoded.userId);
        const tokens = generateTokens(user);
        res.json(tokens);
    } catch (err) {
        res.status(401).json({ error: 'Invalid refresh token' });
    }
});

// Protected route
app.get('/dashboard', authenticate, (req, res) => {
    res.json({ user: req.user });
});
```

#### JWT vs Sessions

| Aspect | JWT | Sessions |
|--------|-----|----------|
| Storage | Client-side | Server-side |
| Scalability | Easy (stateless) | Harder (need shared store) |
| Invalidation | Hard (until expiry) | Easy (delete session) |
| Size | Larger (contains data) | Smaller (just ID) |
| Security | XSS risk (stored in client) | CSRF risk (cookies) |
| Use case | APIs, microservices | Traditional web apps |

#### JWT Security Best Practices

```javascript
// 1. Use HTTPS only
cookie: { secure: true }

// 2. Short expiry for access tokens
{ expiresIn: '15m' }

// 3. Use refresh tokens
// Access token: 15 min
// Refresh token: 7 days

// 4. Don't store sensitive data in JWT
// BAD: { password: "..." }
// GOOD: { userId: 123, role: "admin" }

// 5. Use strong secret
const SECRET = crypto.randomBytes(64).toString('hex');

// 6. Implement token blacklist for logout
const tokenBlacklist = new Set();
function logout(req, res) {
    tokenBlacklist.add(req.token);
}
```

---

#### 3. OAuth 2.0

Delegated authorization—let users sign in with Google, GitHub, etc.

```
User                    Your App              Google
  │                        │                     │
  │ Click "Login with Google"                    │
  │───────────────────────>│                     │
  │                        │ Redirect to Google  │
  │<───────────────────────│                     │
  │                        │                     │
  │ Login on Google                              │
  │─────────────────────────────────────────────>│
  │                        │                     │
  │                        │    Authorization Code│
  │                        │<────────────────────│
  │                        │                     │
  │                        │ Exchange code for token
  │                        │────────────────────>│
  │                        │                     │
  │                        │    Access Token      │
  │                        │<────────────────────│
  │                        │                     │
  │                        │ Fetch user info     │
  │                        │────────────────────>│
  │                        │                     │
  │                        │    User info         │
  │                        │<────────────────────│
  │                        │                     │
  │   Login successful     │                     │
  │<───────────────────────│                     │
```

```javascript
// Passport.js with Google OAuth
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;

passport.use(new GoogleStrategy({
    clientID: process.env.GOOGLE_CLIENT_ID,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    callbackURL: '/auth/google/callback'
}, async (accessToken, refreshToken, profile, done) => {
    // Find or create user
    let user = await User.findOne({ googleId: profile.id });
    if (!user) {
        user = await User.create({
            googleId: profile.id,
            email: profile.emails[0].value,
            name: profile.displayName
        });
    }
    return done(null, user);
}));

// Routes
app.get('/auth/google', passport.authenticate('google', { scope: ['profile', 'email'] }));
app.get('/auth/google/callback', passport.authenticate('google'), (req, res) => {
    const tokens = generateTokens(req.user);
    res.redirect(`/dashboard?token=${tokens.accessToken}`);
});
```

#### OAuth 2.0 Grant Types

| Grant Type | Use Case |
|------------|----------|
| **Authorization Code** | Web apps (most secure) |
| **Authorization Code + PKCE** | Mobile/SPA apps |
| **Client Credentials** | Machine-to-machine |
| **Device Code** | Smart TV, CLI tools |

---

#### 4. API Keys

Simple authentication for machine-to-machine communication.

```javascript
// Middleware
function apiKeyAuth(req, res, next) {
    const apiKey = req.headers['x-api-key'];
    if (!apiKey) {
        return res.status(401).json({ error: 'API key required' });
    }
    
    const client = await db.query('SELECT * FROM api_keys WHERE key = ?', [apiKey]);
    if (!client || !client.active) {
        return res.status(401).json({ error: 'Invalid API key' });
    }
    
    req.client = client;
    next();
}

app.use('/api/', apiKeyAuth);
```

**Best practices:**
- Generate cryptographically random keys
- Hash keys before storing in database
- Set expiration dates
- Implement rate limiting per key
- Allow key rotation

---

#### 5. SSO (Single Sign-On)

One login for multiple applications.

```
User → App A → IdP (Identity Provider) → Logged in
User → App B → IdP → Already logged in → No re-login needed
```

**Protocols:** SAML, OpenID Connect (OIDC), OAuth 2.0

---

### Authorization

#### Role-Based Access Control (RBAC)

```javascript
const roles = {
    user: ['read:own_profile', 'update:own_profile'],
    admin: ['read:all_users', 'create:user', 'delete:user'],
    superadmin: ['*']
};

function authorize(...requiredPermissions) {
    return (req, res, next) => {
        const userRole = req.user.role;
        const userPermissions = roles[userRole] || [];
        
        const hasPermission = requiredPermissions.every(perm => 
            userPermissions.includes(perm) || userPermissions.includes('*')
        );
        
        if (!hasPermission) {
            return res.status(403).json({ error: 'Insufficient permissions' });
        }
        next();
    };
}

// Usage
app.delete('/users/:id', authenticate, authorize('delete:user'), deleteUser);
```

#### Attribute-Based Access Control (ABAC)

```javascript
function authorizeAccess(req, res, next) {
    const { user } = req;
    const resource = req.params;
    
    // Users can only access their own data (unless admin)
    if (user.role !== 'admin' && user.id !== resource.userId) {
        return res.status(403).json({ error: 'Access denied' });
    }
    next();
}
```

---

### Password Hashing

**Never store plain-text passwords!**

```javascript
const bcrypt = require('bcrypt');

// Hash password before storing
async function hashPassword(password) {
    const saltRounds = 12;
    return bcrypt.hash(password, saltRounds);
}

// Verify password during login
async function verifyPassword(password, hash) {
    return bcrypt.compare(password, hash);
}

// Registration
app.post('/register', async (req, res) => {
    const { email, password } = req.body;
    const hashedPassword = await hashPassword(password);
    await db.query('INSERT INTO users (email, password) VALUES (?, ?)', [email, hashedPassword]);
    res.status(201).json({ message: 'User created' });
});
```

---

### Interview Questions

**Q: What's the difference between authentication and authorization?**

A: "Authentication verifies identity ('who are you?')—login with email/password, JWT, OAuth. Authorization checks permissions ('what can you do?')—RBAC, ABAC. Authentication happens first, then authorization."

**Q: JWT vs Sessions—which would you use?**

A: "JWT for APIs and microservices—stateless, scalable, no server storage needed. Sessions for traditional web apps—server can invalidate sessions, simpler security model. JWT trade-off: can't easily revoke tokens until expiry."

**Q: How does OAuth 2.0 work?**

A: "Delegated authorization. User clicks 'Login with Google' → redirected to Google → approves access → Google sends authorization code to your app → your app exchanges code for access token → uses token to get user info. The user never shares their Google password with your app."

**Q: How do you securely store passwords?**

A: "Hash with bcrypt (or Argon2) using a unique salt per password. Never store plain text. Never use MD5 or SHA (too fast, vulnerable to brute force). Bcrypt is slow by design—makes brute force impractical."

**Q: What's the difference between RBAC and ABAC?**

A: "RBAC: users assigned roles (admin, user), roles have permissions. Simple but coarse. ABAC: access based on attributes (user's department, resource's owner, time of day). More flexible but complex. Use RBAC for simple apps, ABAC for complex access control."

---

*Next: [04 — Caching Strategies](04-Caching.md)*
