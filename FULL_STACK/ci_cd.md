# 07 —  CI/CD

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

*Next: [08 — Monitoring & Observability](08-Monitoring.md)*
