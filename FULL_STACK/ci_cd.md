# 07 — CI/CD

## Automated Testing and Deployment

---

### What is CI/CD?

**CI (Continuous Integration):** Automatically test code when pushed to repository.
**CD (Continuous Deployment):** Automatically deploy code when tests pass.

```
Developer pushes code
        │
        ▼
┌─────────────────┐
│   CI Pipeline   │
│  - Lint code    │
│  - Run tests    │
│  - Build        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   CD Pipeline   │
│  - Deploy to    │
│    staging      │
│  - Run E2E tests│
│  - Deploy to    │
│    production   │
└─────────────────┘
```

---

### GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
```

---

### CI/CD with Docker

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm test

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          push: true
          tags: yourusername/myapp:latest

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to server
        uses: appleboy/ssh-action@v0.1.5
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /app
            docker-compose pull
            docker-compose up -d
```

---

### Testing in CI

```javascript
// Jest configuration
// jest.config.js
module.exports = {
    testEnvironment: 'node',
    coverageDirectory: 'coverage',
    collectCoverageFrom: ['src/**/*.js'],
    coverageThreshold: {
        global: {
            branches: 80,
            functions: 80,
            lines: 80,
        },
    },
};

// Example test
// src/__tests__/user.test.js
const request = require('supertest');
const app = require('../app');

describe('User API', () => {
    it('should create a new user', async () => {
        const res = await request(app)
            .post('/api/users')
            .send({ name: 'Alice', email: 'alice@example.com' });
        
        expect(res.status).toBe(201);
        expect(res.body.data.name).toBe('Alice');
    });

    it('should return 400 for invalid email', async () => {
        const res = await request(app)
            .post('/api/users')
            .send({ name: 'Alice', email: 'invalid' });
        
        expect(res.status).toBe(400);
    });
});
```

---

### Interview Questions

**Q: What is CI/CD?**

A: "CI: automatically test code on every push (lint, unit tests, build). CD: automatically deploy when tests pass. Benefits: catch bugs early, faster releases, consistent deployments. Tools: GitHub Actions, GitLab CI, Jenkins."

**Q: What should a CI pipeline include?**

A: "1) Install dependencies. 2) Lint code. 3) Run unit tests. 4) Run integration tests. 5) Build application. 6) Generate coverage report. Optional: security scanning, Docker build, deploy to staging."

**Q: What's the difference between CI and CD?**

A: "CI (Continuous Integration): merge code frequently, run automated tests. CD (Continuous Delivery): always ready to deploy, manual trigger. CD (Continuous Deployment): automatically deploy every change that passes tests."

---

### Additional Interview Questions (25+)

**Q4: What is GitHub Actions?**
A: "CI/CD platform by GitHub. YAML-based workflow files. Runs on GitHub infrastructure. Free for public repos. Supports matrix builds, caching, secrets."

**Q5: How do you handle secrets in CI/CD?**
A: "Store in CI/CD secrets (GitHub Actions, GitLab CI). Inject during build/test/deploy. Never hardcode in pipeline files. Rotate regularly."

**Q6: What is pipeline caching?**
A: "Cache dependencies between runs. Reduces build time. GitHub Actions: actions/cache. Cache node_modules, Docker layers."

**Q7: How do you run tests in CI/CD?**
A: "Unit tests on every commit. Integration tests on PR. E2E tests before deployment. Performance tests periodically."

**Q8: What is matrix builds?**
A: "Run tests across multiple environments. Different Node.js versions, OS, browsers. GitHub Actions: matrix strategy."

**Q9: How do you handle database migrations in CI/CD?**
A: "Run migrations before deployment. Rollback capability. Test migrations in staging. Zero-downtime migrations."

**Q10: What is deployment automation?**
A: "Automated deployment pipeline. No manual steps. Infrastructure as code. Reduces human error. Faster releases."

**Q11: How do you handle rollback in CI/CD?**
A: "Keep previous version. Switch traffic back. Database rollback. CI/CD rollback commands. Test rollback process."

**Q12: What is continuous delivery vs continuous deployment?**
A: "Continuous Delivery: always ready to deploy, manual trigger. Continuous Deployment: automatically deploy every change. CD is more automated."

**Q13: How do you handle environment-specific configs?**
A: "Environment variables per environment. CI/CD secrets. Config files. Infrastructure as code."

**Q14: What is CI/CD best practices?**
A: "Fast feedback. Small, frequent commits. Automated testing. Infrastructure as code. Monitoring. Rollback capability."

**Q15: How do you handle code quality in CI/CD?**
A: "Linting (ESLint). Formatting (Prettier). Type checking (TypeScript). Code coverage. Security scanning."

**Q16: What is artifact management?**
A: "Store build artifacts. Docker images, npm packages. Version control. Security scanning. Tools: Docker Hub, npm, GitHub Packages."

**Q17: How do you handle parallel jobs in CI/CD?**
A: "Run independent jobs in parallel. Matrix builds. Faster feedback. GitHub Actions: jobs.runs-on."

**Q18: What is CI/CD monitoring?**
A: "Track pipeline success/failure. Monitor build times. Alert on failures. Deployment dashboards."

**Q19: How do you handle flaky tests?**
A: "Identify flaky tests. Quarantine them. Fix root cause. Don't ignore flaky tests. Retry logic for transient failures."

**Q20: What is infrastructure as code?**
A: "Define infrastructure in code. Tools: Terraform, CloudFormation, Pulumi. Version control. Reproducible environments."

**Q21: How do you handle secrets rotation?**
A: "Automate rotation. Update CI/CD secrets. Update application config. Zero-downtime rotation."

**Q22: What is CI/CD security?**
A: "Scan dependencies. SAST, DAST. Container scanning. Secrets scanning. Run in pipeline. Block on critical findings."

**Q23: How do you handle multi-environment deployment?**
A: "Separate pipelines per environment. Promotion from dev → staging → production. Environment-specific configs."

**Q24: What is deployment strategy?**
A: "Blue-green: instant switch. Canary: gradual rollout. Rolling: incremental update. Choose based on risk tolerance."

**Q25: How do you handle CI/CD for monorepos?**
A: "Selective builds based on changed files. Shared pipelines. Independent deployments. Tools: Nx, Turborepo."

---

*Next: [08 — Monitoring & Observability](08-Monitoring.md)*
