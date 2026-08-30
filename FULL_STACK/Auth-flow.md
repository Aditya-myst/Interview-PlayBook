# 03 — Authentication Flow

## End-to-End Auth Implementation

---

### Complete Login Flow

```
┌─────────────┐                    ┌─────────────┐                ┌─────────────┐
│   Client    │                    │   Server    │                │  Database   │
│  (React)    │                    │  (Express)  │                │ (PostgreSQL)│
└──────┬──────┘                    └──────┬──────┘                └──────┬──────┘
       │                                  │                              │
       │  POST /api/auth/login            │                              │
       │  { email, password }             │                              │
       │─────────────────────────────────>│                              │
       │                                  │  Find user by email          │
       │                                  │─────────────────────────────>│
       │                                  │                              │
       │                                  │  Return user + hash          │
       │                                  │<─────────────────────────────│
       │                                  │                              │
       │                                  │  Compare password with hash  │
       │                                  │  (bcrypt.compare)            │
       │                                  │                              │
       │                                  │  Generate JWT tokens         │
       │                                  │  (access + refresh)          │
       │                                  │                              │
       │  Return { accessToken,           │                              │
       │    refreshToken }                │                              │
       │<─────────────────────────────────│                              │
       │                                  │                              │
       │  Store tokens                    │                              │
       │  (httpOnly cookie or memory)     │                              │
       │                                  │                              │
```

---

### Backend Implementation

```javascript
// routes/auth.js
const express = require('express');
const router = express.Router();
const authController = require('../controllers/authController');

router.post('/register', authController.register);
router.post('/login', authController.login);
router.post('/refresh', authController.refreshToken);
router.post('/logout', authController.logout);
router.get('/me', authenticate, authController.getMe);

module.exports = router;
```

```javascript
// controllers/authController.js
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const User = require('../models/User');

const ACCESS_TOKEN_EXPIRY = '15m';
const REFRESH_TOKEN_EXPIRY = '7d';

function generateTokens(user) {
    const accessToken = jwt.sign(
        { userId: user.id, email: user.email, role: user.role },
        process.env.JWT_SECRET,
        { expiresIn: ACCESS_TOKEN_EXPIRY }
    );
    
    const refreshToken = jwt.sign(
        { userId: user.id },
        process.env.JWT_REFRESH_SECRET,
        { expiresIn: REFRESH_TOKEN_EXPIRY }
    );
    
    return { accessToken, refreshToken };
}

exports.register = async (req, res, next) => {
    try {
        const { name, email, password } = req.body;
        
        // Check if user exists
        const existingUser = await User.findOne({ email });
        if (existingUser) {
            return res.status(409).json({ error: 'Email already registered' });
        }
        
        // Hash password
        const hashedPassword = await bcrypt.hash(password, 12);
        
        // Create user
        const user = await User.create({ name, email, password: hashedPassword });
        
        // Generate tokens
        const tokens = generateTokens(user);
        
        // Set refresh token in httpOnly cookie
        res.cookie('refreshToken', tokens.refreshToken, {
            httpOnly: true,
            secure: process.env.NODE_ENV === 'production',
            sameSite: 'strict',
            maxAge: 7 * 24 * 60 * 60 * 1000  // 7 days
        });
        
        res.status(201).json({
            accessToken: tokens.accessToken,
            user: { id: user.id, name: user.name, email: user.email }
        });
    } catch (error) {
        next(error);
    }
};

exports.login = async (req, res, next) => {
    try {
        const { email, password } = req.body;
        
        // Find user
        const user = await User.findOne({ email });
        if (!user) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        // Verify password
        const isValid = await bcrypt.compare(password, user.password);
        if (!isValid) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        // Generate tokens
        const tokens = generateTokens(user);
        
        // Set refresh token in httpOnly cookie
        res.cookie('refreshToken', tokens.refreshToken, {
            httpOnly: true,
            secure: process.env.NODE_ENV === 'production',
            sameSite: 'strict',
            maxAge: 7 * 24 * 60 * 60 * 1000
        });
        
        res.json({
            accessToken: tokens.accessToken,
            user: { id: user.id, name: user.name, email: user.email }
        });
    } catch (error) {
        next(error);
    }
};

exports.refreshToken = async (req, res, next) => {
    try {
        const { refreshToken } = req.cookies;
        
        if (!refreshToken) {
            return res.status(401).json({ error: 'No refresh token' });
        }
        
        // Verify refresh token
        const decoded = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET);
        
        // Get user
        const user = await User.findById(decoded.userId);
        if (!user) {
            return res.status(401).json({ error: 'User not found' });
        }
        
        // Generate new tokens
        const tokens = generateTokens(user);
        
        res.cookie('refreshToken', tokens.refreshToken, {
            httpOnly: true,
            secure: process.env.NODE_ENV === 'production',
            sameSite: 'strict',
            maxAge: 7 * 24 * 60 * 60 * 1000
        });
        
        res.json({ accessToken: tokens.accessToken });
    } catch (error) {
        res.status(401).json({ error: 'Invalid refresh token' });
    }
};
```

```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');

exports.authenticate = (req, res, next) => {
    const authHeader = req.headers.authorization;
    
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
        return res.status(401).json({ error: 'No token provided' });
    }
    
    const token = authHeader.split(' ')[1];
    
    try {
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        req.user = decoded;
        next();
    } catch (error) {
        if (error.name === 'TokenExpiredError') {
            return res.status(401).json({ error: 'Token expired' });
        }
        return res.status(401).json({ error: 'Invalid token' });
    }
};

exports.authorize = (...roles) => {
    return (req, res, next) => {
        if (!roles.includes(req.user.role)) {
            return res.status(403).json({ error: 'Insufficient permissions' });
        }
        next();
    };
};
```

---

### Frontend Implementation

```javascript
// context/AuthContext.jsx
import { createContext, useContext, useState, useEffect } from 'react';

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        // Check if user is logged in on mount
        const token = localStorage.getItem('accessToken');
        if (token) {
            fetchUser(token);
        } else {
            setLoading(false);
        }
    }, []);

    const fetchUser = async (token) => {
        try {
            const response = await fetch('/api/auth/me', {
                headers: { Authorization: `Bearer ${token}` }
            });
            if (response.ok) {
                const data = await response.json();
                setUser(data.user);
            } else {
                localStorage.removeItem('accessToken');
            }
        } catch (error) {
            localStorage.removeItem('accessToken');
        } finally {
            setLoading(false);
        }
    };

    const login = async (email, password) => {
        const response = await fetch('/api/auth/login', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email, password }),
            credentials: 'include'  // Include cookies
        });
        
        if (!response.ok) {
            const error = await response.json();
            throw new Error(error.message);
        }
        
        const data = await response.json();
        localStorage.setItem('accessToken', data.accessToken);
        setUser(data.user);
        return data;
    };

    const logout = async () => {
        await fetch('/api/auth/logout', { 
            method: 'POST',
            credentials: 'include'
        });
        localStorage.removeItem('accessToken');
        setUser(null);
    };

    return (
        <AuthContext.Provider value={{ user, login, logout, loading }}>
            {children}
        </AuthContext.Provider>
    );
}

export const useAuth = () => useContext(AuthContext);
```

```jsx
// Protected Route component
function ProtectedRoute({ children }) {
    const { user, loading } = useAuth();
    
    if (loading) return <Spinner />;
    if (!user) return <Navigate to="/login" />;
    
    return children;
}

// Usage in router
<Routes>
    <Route path="/login" element={<LoginPage />} />
    <Route path="/dashboard" element={
        <ProtectedRoute>
            <DashboardPage />
        </ProtectedRoute>
    } />
</Routes>
```

---

### Interview Questions

**Q: Walk me through the login flow.**

A: "1) User submits email/password. 2) Frontend sends POST to /api/auth/login. 3) Backend finds user by email, compares password with bcrypt hash. 4) If valid, generates access token (15min) and refresh token (7 days). 5) Access token sent in response, refresh token in httpOnly cookie. 6) Frontend stores access token, includes in Authorization header for subsequent requests."

**Q: Why use two tokens (access + refresh)?**

A: "Access token: short-lived (15min), used for API requests, stored in memory/localStorage. Refresh token: long-lived (7 days), used to get new access tokens, stored in httpOnly cookie. If access token is stolen, it expires quickly. Refresh token is more secure (httpOnly, not accessible via JavaScript)."

**Q: Where should you store tokens?**

A: "Access token: memory or localStorage (memory is more secure). Refresh token: httpOnly cookie (not accessible via JavaScript, prevents XSS). Avoid storing sensitive tokens in localStorage for production apps."

---

### Additional Interview Questions (35+)

**Q4: What is the difference between authentication and authorization?**
A: "Authentication: 'Who are you?'—verify identity (JWT, OAuth). Authorization: 'What can you do?'—check permissions (RBAC, ABAC). Authentication happens first, then authorization."

**Q5: How does OAuth 2.0 work?**
A: "Delegated authorization. User clicks 'Login with Google' → redirected to Google → approves access → Google sends auth code → your app exchanges code for access token → uses token to get user info."

**Q6: What is RBAC vs ABAC?**
A: "RBAC: users have roles (admin, user), roles have permissions. Simple but coarse. ABAC: access based on attributes (user's department, resource's owner, time). More flexible but complex."

**Q7: How do you implement password hashing?**
A: "Use bcrypt with salt rounds (12). Never store plain text passwords. bcrypt automatically handles salting. Use bcrypt.compare() for verification."

**Q8: What is MFA (Multi-Factor Authentication)?**
A: "Multiple verification methods: password + SMS code, password + authenticator app, password + biometric. Increases security. Implement with TOTP (Time-based One-Time Password)."

**Q9: How do you handle token refresh in frontend?**
A: "When access token expires (401 response), use refresh token to get new access token. Implement in axios interceptor or fetch wrapper. Handle refresh token expiration (redirect to login)."

**Q10: What is token blacklisting?**
A: "List of revoked tokens. Check blacklist on each request. Use Redis for fast lookups. Implement on logout or security events. Trade-off: adds state to stateless JWT."

**Q11: How do you prevent JWT theft?**
A: "Short expiry for access tokens. httpOnly cookies for refresh tokens. HTTPS only. Token binding. Implement token rotation. Monitor for suspicious activity."

**Q12: What is OpenID Connect?**
A: "Authentication layer on top of OAuth 2.0. OAuth handles authorization; OIDC handles authentication. Returns ID token with user info. Used for single sign-on (SSO)."

**Q13: How do you implement SSO?**
A: "Single Sign-On: one login for multiple applications. Use OpenID Connect or SAML. Identity Provider (IdP) manages authentication. Service Providers (SPs) trust IdP."

**Q14: What is API key authentication?**
A: "Simple authentication for machine-to-machine. Generate unique key per client. Include in header (X-API-Key) or query param. Hash keys before storage. Implement rate limiting per key."

**Q15: How do you handle session management?**
A: "Store session data server-side (Redis). Use session ID in cookie. Set appropriate expiry. Implement session invalidation on logout. Handle concurrent sessions."

**Q16: What is CSRF and how do you prevent it?**
A: "Cross-Site Request Forgery: trick user into making unwanted requests. Prevention: CSRF tokens, SameSite cookies, check Origin header, require custom header."

**Q17: What is XSS and how do you prevent it?**
A: "Cross-Site Scripting: inject malicious scripts. Prevention: input validation, output encoding, Content Security Policy (CSP), httpOnly cookies."

**Q18: How do you implement OAuth 2.0 PKCE?**
A: "Proof Key for Code Exchange: for public clients (mobile, SPA). Generate code verifier and code challenge. Send challenge in auth request. Send verifier in token request."

**Q19: What is the difference between OAuth 2.0 grant types?**
A: "Authorization Code: web apps (most secure). Client Credentials: machine-to-machine. Implicit: deprecated (security issues). Device Code: smart TVs, CLI tools."

**Q20: How do you handle OAuth scopes?**
A: "Define granular permissions (read:users, write:posts). Request specific scopes during authorization. Validate scopes on each request. Limit access based on granted scopes."

**Q21: What is JWT audience and issuer?**
A: "Audience (aud): intended recipient of token. Issuer (iss): who created token. Validate both to prevent token misuse. Set in JWT claims and verify on receipt."

**Q22: How do you implement password reset?**
A: "Generate reset token (random, time-limited). Send email with reset link. Verify token on submission. Hash new password. Invalidate token after use."

**Q23: What is account lockout policy?**
A: "Lock account after N failed login attempts. Prevents brute force attacks. Implement with exponential backoff. Notify user of lockout. Reset on successful login."

**Q24: How do you handle concurrent sessions?**
A: "Limit active sessions per user. Store sessions in Redis. Invalidate oldest session on new login. Allow user to view and terminate sessions."

**Q25: What is token introspection?**
A: "Validate token with authorization server. Used when resource server can't validate locally. RFC 7662 standard. Adds network call but ensures freshness."

**Q26: How do you implement audit logging for auth?**
A: "Log all auth events: login, logout, password change, permission changes. Include timestamp, user, IP, action. Store in separate audit log. Monitor for suspicious patterns."

**Q27: What is mutual TLS authentication?**
A: "Both client and server present certificates. Stronger than API keys. Used for service-to-service communication. Complex to implement but very secure."

**Q28: How do you handle JWT in microservices?**
A: "API gateway validates JWT. Pass user info in headers to services. Or use service mesh for mTLS. Consider token introspection for critical operations."

**Q29: What is OAuth 2.0 device flow?**
A: "For devices without browsers (smart TVs, CLI). Device shows code, user enters on another device. Polls for authorization. Used by streaming services."

**Q30: How do you implement role hierarchy?**
A: "Admin inherits Moderator permissions. Implement with role hierarchy in RBAC. Or use permission-based system with role grouping."

**Q31: What is token binding?**
A: "Bind token to specific client (fingerprint). Prevents token theft and replay attacks. Include client fingerprint in token claims."

**Q32: How do you handle auth in serverless?**
A: "JWT validation in Lambda authorizer. Cache authorizer results. Use API Gateway built-in auth. Implement with custom authorizer functions."

**Q33: What is zero-trust authentication?**
A: "Never trust, always verify. Authenticate every request regardless of network location. Use short-lived tokens. Implement least privilege access. Monitor all access."

**Q34: How do you implement social login?**
A: "Use Passport.js with multiple strategies (Google, GitHub). Link accounts by email. Store provider IDs in user document. Handle account merging."

**Q35: What is JWT token rotation?**
A: "Issue new refresh token on each use. Invalidate old refresh token. Prevents token reuse if stolen. Store token version to invalidate old tokens."

**Q36: How do you handle auth in GraphQL?**
A: "Context-based auth. Directive-based auth (@auth). Field-level permissions. Middleware for queries/mutations. DataLoader for user loading."

**Q37: How do you implement SSO with SAML?**
A: "SAML 2.0: XML-based. Identity Provider (IdP) and Service Provider (SP). SAML assertion contains user attributes. Complex but enterprise-standard."

**Q38: What is OAuth 2.0 token exchange?**
A: "RFC 8693: exchange one token type for another. Use for impersonation, delegation. Cross-domain token exchange."

---

*Next: [04 — Environment Variables](04-Environment-Variables.md)*
