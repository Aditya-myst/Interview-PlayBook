# 04 — Environment Variables

## Security and Configuration

---

### Why Environment Variables?

Never hardcode sensitive data (API keys, database passwords, JWT secrets) in your code. Use environment variables.

```javascript
// BAD: Hardcoded secrets
const dbPassword = 'mysecretpassword123';
const jwtSecret = 'super-secret-key';

// GOOD: Environment variables
const dbPassword = process.env.DB_PASSWORD;
const jwtSecret = process.env.JWT_SECRET;
```

---

### .env Files

```bash
# .env (local development - NEVER commit to git)
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
JWT_SECRET=my-super-secret-jwt-key
JWT_REFRESH_SECRET=my-refresh-secret-key
REDIS_URL=redis://localhost:6379
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
STRIPE_SECRET_KEY=sk_test_4eC39HqLyjWDarjtT1zdp7dc
PORT=3000
NODE_ENV=development
```

```bash
# .env.example (commit this - shows required variables)
DATABASE_URL=
JWT_SECRET=
JWT_REFRESH_SECRET=
REDIS_URL=
PORT=3000
NODE_ENV=development
```

```bash
# .gitignore
.env
.env.local
.env.production
```

---

### Loading Environment Variables

#### Node.js (dotenv)

```javascript
// Load at application entry point
require('dotenv').config();

// Or with specific path
require('dotenv').config({ path: '.env.production' });
```

#### Next.js

```javascript
// next.config.js
module.exports = {
    env: {
        DATABASE_URL: process.env.DATABASE_URL,
    },
    // Public variables (accessible in browser)
    publicRuntimeConfig: {
        NEXT_PUBLIC_API_URL: process.env.NEXT_PUBLIC_API_URL,
    }
};

// .env.local (Next.js auto-loads this)
NEXT_PUBLIC_API_URL=https://api.example.com
DATABASE_URL=postgresql://...
```

**Next.js rules:**
- `NEXT_PUBLIC_*` → accessible in browser
- Without prefix → server-side only

#### React (Create React App)

```javascript
// .env
REACT_APP_API_URL=https://api.example.com
REACT_APP_STRIPE_KEY=pk_test_...

// Access in code
const apiUrl = process.env.REACT_APP_API_URL;
```

---

### Environment-Specific Configuration

```bash
# .env.development
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb_dev
NODE_ENV=development
LOG_LEVEL=debug

# .env.staging
DATABASE_URL=postgresql://user:pass@staging-db:5432/mydb_staging
NODE_ENV=staging
LOG_LEVEL=info

# .env.production
DATABASE_URL=postgresql://user:pass@prod-db:5432/mydb_prod
NODE_ENV=production
LOG_LEVEL=error
```

```javascript
// config/index.js
const config = {
    port: process.env.PORT || 3000,
    database: {
        url: process.env.DATABASE_URL,
    },
    jwt: {
        secret: process.env.JWT_SECRET,
        refreshSecret: process.env.JWT_REFRESH_SECRET,
        accessExpiry: '15m',
        refreshExpiry: '7d',
    },
    redis: {
        url: process.env.REDIS_URL,
    },
    aws: {
        accessKeyId: process.env.AWS_ACCESS_KEY_ID,
        secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
        region: process.env.AWS_REGION || 'us-east-1',
    },
};

// Validate required variables
const required = ['DATABASE_URL', 'JWT_SECRET'];
for (const key of required) {
    if (!process.env[key]) {
        throw new Error(`Missing required environment variable: ${key}`);
    }
}

module.exports = config;
```

---

### Secrets Management

| Method | Use Case |
|--------|----------|
| .env files | Local development |
| CI/CD secrets | GitHub Actions, GitLab CI |
| Cloud secrets | AWS Secrets Manager, Google Secret Manager |
| Vault | HashiCorp Vault for enterprise |

```yaml
# GitHub Actions secrets
# Settings → Secrets → New repository secret
name: Deploy
on: push
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          JWT_SECRET: ${{ secrets.JWT_SECRET }}
        run: npm run deploy
```

---

### Interview Questions

**Q: Why use environment variables?**

A: "Security: don't hardcode secrets in code. Flexibility: different configs for dev/staging/prod. 12-factor app methodology. Secrets in code get committed to git, exposed in builds, visible to anyone with code access."

**Q: What's the difference between .env and .env.example?**

A: ".env: contains actual secrets, never committed. .env.example: lists required variables with empty values, committed to git. Helps other developers know what variables are needed."

**Q: How do you handle secrets in production?**

A: "CI/CD secrets (GitHub Actions, GitLab CI), cloud secret managers (AWS Secrets Manager, Google Secret Manager), or HashiCorp Vault. Never in code, never in git, never in Docker images."

---

### Additional Interview Questions (20+)

**Q4: What is the 12-factor app methodology?**
A: "Methodology for building modern applications. Factor 3: store config in environment variables. Benefits: portability, scalability, maintainability. Other factors: codebase, dependencies, backing services."

**Q5: How do you validate environment variables?**
A: "Validate on startup. Check required variables exist. Validate format (URLs, numbers). Use libraries like envalid, joi. Fail fast if missing."

**Q6: What is the difference between process.env and config files?**
A: "process.env: environment variables, 12-factor compliant, secure. Config files: JSON/YAML, easier to read, version control. Use env vars for secrets, config files for non-sensitive settings."

**Q7: How do you handle environment variables in Docker?**
A: "Pass with -e flag. Use .env file with --env-file. Use Docker secrets for sensitive data. Use Docker Compose environment section."

**Q8: What is the difference between .env.local and .env.production?**
A: ".env.local: local overrides, never committed. .env.production: production values, committed or managed by CI/CD. .env.development: development defaults."

**Q9: How do you handle environment variables in serverless?**
A: "AWS Lambda: environment variables in function config. Use AWS Secrets Manager for sensitive data. Vercel: environment variables in dashboard."

**Q10: What is secret rotation?**
A: "Regularly change secrets (passwords, API keys). Automate with secret managers. Update all services using the secret. Zero-downtime rotation."

**Q11: How do you prevent secrets from leaking?**
A: "Never commit .env files. Use .gitignore. Scan commits for secrets (git-secrets, truffleHog). Use secret managers. Don't log secrets."

**Q12: What is environment variable precedence?**
A: "Order of precedence: shell env > .env.local > .env.[NODE_ENV] > .env. Later files override earlier. Shell variables always win."

**Q13: How do you share environment variables across teams?**
A: ".env.example for documentation. Secret managers for actual values. CI/CD secrets for pipelines. Never share via Slack/email."

**Q14: What is the difference between config and secrets?**
A: "Config: non-sensitive settings (ports, URLs, feature flags). Secrets: sensitive data (passwords, API keys). Store config in files, secrets in secret managers."

**Q15: How do you handle environment variables in monorepos?**
A: "Root .env for shared variables. Package-specific .env for package variables. Use dotenv-expand for variable interpolation."

**Q16: What is runtime vs build-time configuration?**
A: "Runtime: available when app runs (process.env). Build-time: baked into build (NEXT_PUBLIC_*). Use runtime for secrets, build-time for public config."

**Q17: How do you test code that uses environment variables?**
A: "Mock process.env in tests. Use dotenv for test-specific .env files. Set variables in test setup. Clean up after tests."

**Q18: What is environment variable injection?**
A: "Inject variables at runtime. Docker: -e flag. Kubernetes: ConfigMaps, Secrets. CI/CD: pipeline variables."

**Q19: How do you handle feature flags?**
A: "Environment variables for simple flags. Feature flag services (LaunchDarkly, Unleash) for complex. A/B testing integration."

**Q20: What is the difference between NODE_ENV=development and NODE_ENV=production?**
A: "development: verbose logging, detailed errors, no optimization. production: minimal logging, generic errors, optimizations enabled. Affects Express, React, webpack behavior."

**Q21: How do you handle environment variables in CI/CD?**
A: "Store in CI/CD secrets (GitHub Actions, GitLab CI). Inject during build/test/deploy. Never hardcode in pipeline files."

**Q22: What is the difference between AWS Secrets Manager and Parameter Store?**
A: "Secrets Manager: automatic rotation, higher cost, for secrets. Parameter Store: no rotation, lower cost, for config. Use Secrets Manager for passwords, Parameter Store for URLs."

---

*Next: [05 — Docker Basics](05-Docker.md)*
