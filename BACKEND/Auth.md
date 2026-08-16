# 03 — Authentication & Authorization

## JWT, OAuth 2.0, RBAC, Security — 35+ Interview Questions

---

### JWT (JSON Web Tokens) Deep Dive

#### JWT Structure
```
Header.Payload.Signature

Header: { "alg": "HS256", "typ": "JWT" }
Payload: { "sub": "123", "name": "Alice", "role": "admin", "exp": 1234567890 }
Signature: HMACSHA256(base64(header) + "." + base64(payload), secret)
```

#### Complete Implementation

```javascript
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

const ACCESS_TOKEN_SECRET = process.env.JWT_SECRET;
const REFRESH_TOKEN_SECRET = process.env.JWT_REFRESH_SECRET;
const ACCESS_TOKEN_EXPIRY = '15m';
const REFRESH_TOKEN_EXPIRY = '7d';

// Generate token pair
function generateTokens(user) {
    const payload = { sub: user.id, email: user.email, role: user.role };
    const accessToken = jwt.sign(payload, ACCESS_TOKEN_SECRET, { expiresIn: ACCESS_TOKEN_EXPIRY });
    const refreshToken = jwt.sign({ sub: user.id, tokenVersion: user.tokenVersion }, REFRESH_TOKEN_SECRET, { expiresIn: REFRESH_TOKEN_EXPIRY });
    return { accessToken, refreshToken };
}

// Verify middleware
function authenticate(req, res, next) {
    const authHeader = req.headers.authorization;
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
        return res.status(401).json({ error: 'No token provided' });
    }
    const token = authHeader.split(' ')[1];
    try {
        req.user = jwt.verify(token, ACCESS_TOKEN_SECRET);
        next();
    } catch (error) {
        if (error.name === 'TokenExpiredError') {
            return res.status(401).json({ error: 'Token expired' });
        }
        return res.status(401).json({ error: 'Invalid token' });
    }
}

// Role-based authorization
function authorize(...roles) {
    return (req, res, next) => {
        if (!roles.includes(req.user.role)) {
            return res.status(403).json({ error: 'Insufficient permissions' });
        }
        next();
    };
}

// Login endpoint
router.post('/login', async (req, res) => {
    const { email, password } = req.body;
    const user = await User.findOne({ email });
    if (!user || !await bcrypt.compare(password, user.password)) {
        return res.status(401).json({ error: 'Invalid credentials' });
    }
    const tokens = generateTokens(user);
    res.cookie('refreshToken', tokens.refreshToken, { httpOnly: true, secure: true, sameSite: 'strict', maxAge: 7 * 24 * 60 * 60 * 1000 });
    res.json({ accessToken: tokens.accessToken });
});

// Refresh endpoint
router.post('/refresh', async (req, res) => {
    const { refreshToken } = req.cookies;
    if (!refreshToken) return res.status(401).json({ error: 'No refresh token' });
    try {
        const decoded = jwt.verify(refreshToken, REFRESH_TOKEN_SECRET);
        const user = await User.findById(decoded.sub);
        if (!user || user.tokenVersion !== decoded.tokenVersion) {
            return res.status(401).json({ error: 'Invalid refresh token' });
        }
        const tokens = generateTokens(user);
        res.cookie('refreshToken', tokens.refreshToken, { httpOnly: true, secure: true, sameSite: 'strict', maxAge: 7 * 24 * 60 * 60 * 1000 });
        res.json({ accessToken: tokens.accessToken });
    } catch (error) {
        res.status(401).json({ error: 'Invalid refresh token' });
    }
});
```

---

### OAuth 2.0

```javascript
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;

passport.use(new GoogleStrategy({
    clientID: process.env.GOOGLE_CLIENT_ID,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    callbackURL: '/auth/google/callback'
}, async (accessToken, refreshToken, profile, done) => {
    let user = await User.findOne({ googleId: profile.id });
    if (!user) {
        user = await User.create({ googleId: profile.id, email: profile.emails[0].value, name: profile.displayName });
    }
    return done(null, user);
}));

router.get('/auth/google', passport.authenticate('google', { scope: ['profile', 'email'] }));
router.get('/auth/google/callback', passport.authenticate('google', { session: false }), (req, res) => {
    const tokens = generateTokens(req.user);
    res.redirect(`/dashboard?token=${tokens.accessToken}`);
});
```

---

### RBAC (Role-Based Access Control)

```javascript
const roles = {
    user: ['read:own_profile', 'update:own_profile'],
    moderator: ['read:own_profile', 'update:own_profile', 'read:all_posts', 'delete:posts'],
    admin: ['*']
};

function checkPermission(requiredPermission) {
    return (req, res, next) => {
        const userPermissions = roles[req.user.role] || [];
        if (userPermissions.includes('*') || userPermissions.includes(requiredPermission)) {
            return next();
        }
        return res.status(403).json({ error: 'Insufficient permissions' });
    };
}
```

---

### Password Hashing

```javascript
const bcrypt = require('bcrypt');
const SALT_ROUNDS = 12;

userSchema.pre('save', async function(next) {
    if (!this.isModified('password')) return next();
    this.password = await bcrypt.hash(this.password, SALT_ROUNDS);
    next();
});

userSchema.methods.comparePassword = async function(candidatePassword) {
    return bcrypt.compare(candidatePassword, this.password);
};
```

---

### Interview Questions (35+)

**Q1: What is JWT and how does it work?**
A: "JWT has three parts: header (algorithm), payload (claims), signature. Server creates JWT with user info, signs with secret. Client sends in Authorization header. Server verifies signature and extracts user info. Stateless—no server-side session storage needed."

**Q2: What's the difference between access and refresh tokens?**
A: "Access token: short-lived (15min), used for API requests, stored in memory/localStorage. Refresh token: long-lived (7 days), used to get new access tokens, stored in httpOnly cookie. If access token is stolen, it expires quickly."

**Q3: How does OAuth 2.0 work?**
A: "Delegated authorization. User clicks 'Login with Google' → redirected to Google → approves access → Google sends auth code → your app exchanges code for access token → uses token to get user info. User never shares Google password with your app."

**Q4: What's the difference between authentication and authorization?**
A: "Authentication: 'Who are you?'—verify identity (JWT, OAuth). Authorization: 'What can you do?'—check permissions (RBAC, ABAC). Authentication happens first, then authorization."

**Q5: JWT vs Sessions—which would you use?**
A: "JWT for APIs and microservices—stateless, scalable, no server storage. Sessions for traditional web apps—server can invalidate, simpler security. JWT trade-off: can't easily revoke until expiry."

**Q6: How do you handle token refresh?**
A: "Use two tokens: short-lived access token (15min) and long-lived refresh token (7days). Access token expires, client uses refresh token to get new access token. Refresh token stored in httpOnly cookie."

**Q7: Where should you store tokens?**
A: "Access token: memory or localStorage (memory is more secure). Refresh token: httpOnly cookie (not accessible via JavaScript, prevents XSS). Avoid storing sensitive tokens in localStorage for production."

**Q8: What is RBAC vs ABAC?**
A: "RBAC: users have roles (admin, user), roles have permissions. Simple but coarse. ABAC: access based on attributes (user's department, resource's owner, time). More flexible but complex."

**Q9: How do you implement password hashing?**
A: "Use bcrypt with salt rounds (12). Never store plain text passwords. bcrypt automatically handles salting. Use bcrypt.compare() for verification."

**Q10: What is MFA (Multi-Factor Authentication)?**
A: "Multiple verification methods: password + SMS code, password + authenticator app, password + biometric. Increases security. Implement with TOTP (Time-based One-Time Password)."

**Q11: How do you handle OAuth token expiration?**
A: "Use refresh tokens to get new access tokens. Implement token rotation (new refresh token on each use). Store refresh tokens securely (httpOnly cookies). Handle expired tokens gracefully in frontend."

**Q12: What is token blacklisting?**
A: "List of revoked tokens. Check blacklist on each request. Use Redis for fast lookups. Implement on logout or security events. Trade-off: adds state to stateless JWT."

**Q13: How do you prevent JWT theft?**
A: "Short expiry for access tokens. httpOnly cookies for refresh tokens. HTTPS only. Token binding. Implement token rotation. Monitor for suspicious activity."

**Q14: What is OpenID Connect?**
A: "Authentication layer on top of OAuth 2.0. OAuth handles authorization; OIDC handles authentication. Returns ID token with user info. Used for single sign-on (SSO)."

**Q15: How do you implement SSO?**
A: "Single Sign-On: one login for multiple applications. Use OpenID Connect or SAML. Identity Provider (IdP) manages authentication. Service Providers (SPs) trust IdP."

**Q16: What is API key authentication?**
A: "Simple authentication for machine-to-machine. Generate unique key per client. Include in header (X-API-Key) or query param. Hash keys before storage. Implement rate limiting per key."

**Q17: How do you handle session management?**
A: "Store session data server-side (Redis). Use session ID in cookie. Set appropriate expiry. Implement session invalidation on logout. Handle concurrent sessions."

**Q18: What is CSRF and how do you prevent it?**
A: "Cross-Site Request Forgery: trick user into making unwanted requests. Prevention: CSRF tokens, SameSite cookies, check Origin header, require custom header."

**Q19: What is XSS and how do you prevent it?**
A: "Cross-Site Scripting: inject malicious scripts. Prevention: input validation, output encoding, Content Security Policy (CSP), httpOnly cookies."

**Q20: How do you implement OAuth 2.0 PKCE?**
A: "Proof Key for Code Exchange: for public clients (mobile, SPA). Generate code verifier and code challenge. Send challenge in auth request. Send verifier in token request. Prevents auth code interception."

**Q21: What is the difference between OAuth 2.0 grant types?**
A: "Authorization Code: web apps (most secure). Client Credentials: machine-to-machine. Implicit: deprecated (security issues). Device Code: smart TVs, CLI tools."

**Q22: How do you handle OAuth scopes?**
A: "Define granular permissions (read:users, write:posts). Request specific scopes during authorization. Validate scopes on each request. Limit access based on granted scopes."

**Q23: What is JWT audience and issuer?**
A: "Audience (aud): intended recipient of token. Issuer (iss): who created token. Validate both to prevent token misuse. Set in JWT claims and verify on receipt."

**Q24: How do you implement password reset?**
A: "Generate reset token (random, time-limited). Send email with reset link. Verify token on submission. Hash new password. Invalidate token after use."

**Q25: What is account lockout policy?**
A: "Lock account after N failed login attempts. Prevents brute force attacks. Implement with exponential backoff. Notify user of lockout. Reset on successful login."

**Q26: How do you handle concurrent sessions?**
A: "Limit active sessions per user. Store sessions in Redis. Invalidate oldest session on new login. Allow user to view and terminate sessions."

**Q27: What is token introspection?**
A: "Validate token with authorization server. Used when resource server can't validate locally. RFC 7662 standard. Adds network call but ensures freshness."

**Q28: How do you implement audit logging for auth?**
A: "Log all auth events: login, logout, password change, permission changes. Include timestamp, user, IP, action. Store in separate audit log. Monitor for suspicious patterns."

**Q29: What is mutual TLS authentication?**
A: "Both client and server present certificates. Stronger than API keys. Used for service-to-service communication. Complex to implement but very secure."

**Q30: How do you handle JWT in microservices?**
A: "API gateway validates JWT. Pass user info in headers to services. Or use service mesh for mTLS. Consider token introspection for critical operations."

**Q31: What is OAuth 2.0 device flow?**
A: "For devices without browsers (smart TVs, CLI). Device shows code, user enters on another device. Polls for authorization. Used by streaming services."

**Q32: How do you implement role hierarchy?**
A: "Admin inherits Moderator permissions. Implement with role hierarchy in RBAC. Or use permission-based system with role grouping."

**Q33: What is token binding?**
A: "Bind token to specific client (fingerprint). Prevents token theft and replay attacks. Include client fingerprint in token claims."

**Q34: How do you handle auth in serverless?**
A: "JWT validation in Lambda authorizer. Cache authorizer results. Use API Gateway built-in auth. Implement with custom authorizer functions."

**Q35: What is zero-trust authentication?**
A: "Never trust, always verify. Authenticate every request regardless of network location. Use short-lived tokens. Implement least privilege access. Monitor all access."

---

### Complete OAuth 2.0 Implementation

```javascript
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;
const GitHubStrategy = require('passport-github2').Strategy;

// Google OAuth
passport.use(new GoogleStrategy({
    clientID: process.env.GOOGLE_CLIENT_ID,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    callbackURL: '/auth/google/callback',
    scope: ['profile', 'email']
}, async (accessToken, refreshToken, profile, done) => {
    try {
        let user = await User.findOne({ $or: [{ googleId: profile.id }, { email: profile.emails[0].value }] });
        
        if (!user) {
            user = await User.create({
                googleId: profile.id,
                email: profile.emails[0].value,
                name: profile.displayName,
                avatar: profile.photos[0]?.value,
                emailVerified: true
            });
        } else if (!user.googleId) {
            user.googleId = profile.id;
            await user.save();
        }
        
        return done(null, user);
    } catch (error) {
        return done(error, null);
    }
}));

// GitHub OAuth
passport.use(new GitHubStrategy({
    clientID: process.env.GITHUB_CLIENT_ID,
    clientSecret: process.env.GITHUB_CLIENT_SECRET,
    callbackURL: '/auth/github/callback',
    scope: ['user:email']
}, async (accessToken, refreshToken, profile, done) => {
    try {
        let user = await User.findOne({ $or: [{ githubId: profile.id }, { email: profile.emails[0].value }] });
        
        if (!user) {
            user = await User.create({
                githubId: profile.id,
                email: profile.emails[0].value,
                name: profile.displayName || profile.username,
                avatar: profile.photos[0]?.value,
                emailVerified: true
            });
        }
        
        return done(null, user);
    } catch (error) {
        return done(error, null);
    }
}));

// Routes
router.get('/auth/google', passport.authenticate('google', { scope: ['profile', 'email'] }));
router.get('/auth/google/callback', passport.authenticate('google', { session: false }), (req, res) => {
    const tokens = generateTokens(req.user);
    res.cookie('refreshToken', tokens.refreshToken, { httpOnly: true, secure: true, sameSite: 'strict' });
    res.redirect(`${process.env.FRONTEND_URL}/auth/callback?token=${tokens.accessToken}`);
});

router.get('/auth/github', passport.authenticate('github', { scope: ['user:email'] }));
router.get('/auth/github/callback', passport.authenticate('github', { session: false }), (req, res) => {
    const tokens = generateTokens(req.user);
    res.cookie('refreshToken', tokens.refreshToken, { httpOnly: true, secure: true, sameSite: 'strict' });
    res.redirect(`${process.env.FRONTEND_URL}/auth/callback?token=${tokens.accessToken}`);
});
```

---

### Complete RBAC System

```javascript
// Permission-based RBAC
const permissions = {
    // User permissions
    'users:read': 'Read user profiles',
    'users:write': 'Create/update users',
    'users:delete': 'Delete users',
    
    // Post permissions
    'posts:read': 'Read posts',
    'posts:write': 'Create/update posts',
    'posts:delete': 'Delete posts',
    'posts:publish': 'Publish posts',
    
    // Admin permissions
    'admin:users': 'Manage all users',
    'admin:settings': 'Manage system settings',
    'admin:analytics': 'View analytics'
};

const roles = {
    user: {
        permissions: ['users:read', 'posts:read', 'posts:write'],
        description: 'Regular user'
    },
    moderator: {
        permissions: ['users:read', 'posts:read', 'posts:write', 'posts:delete', 'posts:publish'],
        description: 'Content moderator'
    },
    admin: {
        permissions: ['*'],
        description: 'Administrator'
    }
};

// Role middleware
function requireRole(...allowedRoles) {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({ error: { code: 'UNAUTHORIZED', message: 'Authentication required' } });
        }
        
        if (!allowedRoles.includes(req.user.role)) {
            return res.status(403).json({ error: { code: 'FORBIDDEN', message: 'Insufficient role' } });
        }
        
        next();
    };
}

// Permission middleware
function requirePermission(...requiredPermissions) {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({ error: { code: 'UNAUTHORIZED', message: 'Authentication required' } });
        }
        
        const userRole = roles[req.user.role];
        if (!userRole) {
            return res.status(403).json({ error: { code: 'FORBIDDEN', message: 'Invalid role' } });
        }
        
        const hasPermission = requiredPermissions.every(perm => 
            userRole.permissions.includes('*') || userRole.permissions.includes(perm)
        );
        
        if (!hasPermission) {
            return res.status(403).json({ error: { code: 'FORBIDDEN', message: 'Insufficient permissions' } });
        }
        
        next();
    };
}

// Resource ownership middleware
function requireOwnershipOrAdmin(getResourceOwnerId) {
    return async (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({ error: { code: 'UNAUTHORIZED', message: 'Authentication required' } });
        }
        
        if (req.user.role === 'admin') {
            return next();
        }
        
        const ownerId = await getResourceOwnerId(req);
        if (req.user.id !== ownerId) {
            return res.status(403).json({ error: { code: 'FORBIDDEN', message: 'Not resource owner' } });
        }
        
        next();
    };
}

// Usage
router.get('/api/users', requirePermission('users:read'), getUsers);
router.put('/api/users/:id', requireOwnershipOrAdmin(async (req) => {
    const user = await User.findById(req.params.id);
    return user?._id.toString();
}), updateUser);
router.delete('/api/users/:id', requireRole('admin'), deleteUser);
```

---

### Complete MFA Implementation

```javascript
const speakeasy = require('speakeasy');
const QRCode = require('qrcode');

// Generate MFA secret
router.post('/auth/mfa/setup', authenticate, async (req, res) => {
    const secret = speakeasy.generateSecret({
        name: `MyApp:${req.user.email}`,
        issuer: 'MyApp'
    });
    
    // Store secret temporarily
    await redis.set(`mfa:${req.user.id}`, secret.base32, 'EX', 300);
    
    // Generate QR code
    const qrCodeUrl = await QRCode.toDataURL(secret.otpauth_url);
    
    res.json({
        secret: secret.base32,
        qrCode: qrCodeUrl
    });
});

// Verify MFA and enable
router.post('/auth/mfa/verify', authenticate, async (req, res) => {
    const { code } = req.body;
    const secret = await redis.get(`mfa:${req.user.id}`);
    
    if (!secret) {
        return res.status(400).json({ error: { code: 'INVALID', message: 'MFA setup expired' } });
    }
    
    const verified = speakeasy.totp.verify({
        secret,
        encoding: 'base32',
        token: code,
        window: 1
    });
    
    if (!verified) {
        return res.status(400).json({ error: { code: 'INVALID_CODE', message: 'Invalid MFA code' } });
    }
    
    // Enable MFA for user
    await User.findByIdAndUpdate(req.user.id, {
        mfaEnabled: true,
        mfaSecret: secret
    });
    
    // Generate backup codes
    const backupCodes = generateBackupCodes();
    await User.findByIdAndUpdate(req.user.id, { mfaBackupCodes: backupCodes });
    
    await redis.del(`mfa:${req.user.id}`);
    
    res.json({ backupCodes });
});

// Login with MFA
router.post('/auth/login', async (req, res) => {
    const { email, password, mfaCode } = req.body;
    
    const user = await User.findOne({ email });
    if (!user || !await user.comparePassword(password)) {
        return res.status(401).json({ error: { code: 'INVALID_CREDENTIALS', message: 'Invalid credentials' } });
    }
    
    // Check if MFA is enabled
    if (user.mfaEnabled) {
        if (!mfaCode) {
            return res.status(200).json({ requiresMfa: true, tempToken: generateTempToken(user.id) });
        }
        
        const verified = speakeasy.totp.verify({
            secret: user.mfaSecret,
            encoding: 'base32',
            token: mfaCode,
            window: 1
        });
        
        if (!verified) {
            // Check backup codes
            if (!user.mfaBackupCodes.includes(mfaCode)) {
                return res.status(401).json({ error: { code: 'INVALID_MFA', message: 'Invalid MFA code' } });
            }
            // Remove used backup code
            user.mfaBackupCodes = user.mfaBackupCodes.filter(c => c !== mfaCode);
            await user.save();
        }
    }
    
    const tokens = generateTokens(user);
    res.cookie('refreshToken', tokens.refreshToken, { httpOnly: true, secure: true, sameSite: 'strict' });
    res.json({ accessToken: tokens.accessToken, user: user.toPublic() });
});
```

---

### Additional Interview Questions (15+)

**Q36: How do you implement social login with multiple providers?**
A: "Use Passport.js with multiple strategies (Google, GitHub, Facebook). Link accounts by email. Store provider IDs in user document. Handle account merging when same email exists."

**Q37: What is JWT token rotation?**
A: "Issue new refresh token on each use. Invalidate old refresh token. Prevents token reuse if stolen. Store token version to invalidate old tokens."

**Q38: How do you handle auth in microservices?**
A: "API gateway validates JWT. Pass user info in headers. Service mesh for mTLS. Token introspection for critical operations. Shared auth service."

**Q39: What is ABAC (Attribute-Based Access Control)?**
A: "Access based on attributes: user attributes (role, department), resource attributes (owner, type), environment (time, location). More flexible than RBAC."

**Q40: How do you implement password policies?**
A: "Minimum length, complexity requirements (uppercase, lowercase, number, special char). Check against common passwords. Password history. Expiration policy."

**Q41: What is session fixation attack?**
A: "Attacker sets user's session ID before login. Prevention: regenerate session ID on login. Use secure session configuration."

**Q42: How do you handle auth in serverless?**
A: "JWT validation in Lambda authorizer. Cache authorizer results. API Gateway built-in auth. Custom authorizer functions."

**Q43: What is OAuth 2.0 token exchange?**
A: "RFC 8693: exchange one token type for another. Use for impersonation, delegation. Cross-domain token exchange."

**Q44: How do you implement account recovery?**
A: "Security questions. Email recovery. Phone recovery. Backup codes. Multi-step verification. Time-limited recovery tokens."

**Q45: What is JWT token claims validation?**
A: "Validate issuer (iss), audience (aud), expiration (exp), not before (nbf). Prevent token misuse across services. Configure in JWT verification."

**Q46: How do you handle auth in GraphQL?**
A: "Context-based auth. Directive-based auth (@auth). Field-level permissions. Middleware for queries/mutations. DataLoader for user loading."

**Q47: What is token binding?**
A: "Bind token to client fingerprint. Include fingerprint in token claims. Validate on each request. Prevents token theft."

**Q48: How do you implement SSO with SAML?**
A: "SAML 2.0: XML-based. Identity Provider (IdP) and Service Provider (SP). SAML assertion contains user attributes. Complex but enterprise-standard."

**Q49: What is OAuth 2.0 device authorization grant?**
A: "For devices without browsers. Device displays code. User enters code on another device. Device polls for authorization. Used by smart TVs, CLI tools."

**Q50: How do you handle auth rate limiting?**
A: "Stricter rate limits for auth endpoints. Limit by IP and email. Account lockout after N failures. Exponential backoff. CAPTCHA for repeated failures."

---

*Next: [04 — Caching Strategies](04-Caching.md)*
