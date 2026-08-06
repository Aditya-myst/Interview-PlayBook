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

*Next: [04 — Environment Variables](04-Environment-Variables.md)*
