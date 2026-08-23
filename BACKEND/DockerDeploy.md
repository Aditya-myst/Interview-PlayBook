# 14 — Docker & Deployment

## Containers, CI/CD, Cloud — 25+ Interview Questions

---

### Dockerfile

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "src/index.js"]
```

---

### Multi-Stage Build

```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
RUN npm ci --only=production
CMD ["node", "dist/index.js"]
```

---

### Docker Compose

```yaml
version: '3.8'
services:
  api:
    build: .
    ports: ["3000:3000"]
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/mydb
    depends_on: [postgres, redis]
  
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: password
    volumes: [postgres_data:/var/lib/postgresql/data]
  
  redis:
    image: redis:7-alpine

volumes:
  postgres_data:
```

---

### CI/CD (GitHub Actions)

```yaml
name: CI/CD
on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm test
  
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: docker build -t myapp .
      - run: docker push myapp
```

---

### Interview Questions (25+)

**Q1: What is Docker?**
A: "Platform for containerizing applications. Packages app with dependencies into container. Consistent across environments. Lightweight compared to VMs."

**Q2: What's the difference between Docker image and container?**
A: "Image: read-only template with app and dependencies. Container: running instance of image. One image can run multiple containers."

**Q3: What is Docker Compose?**
A: "Tool for multi-container applications. Uses YAML to configure services, networks, volumes. One command starts entire stack."

**Q4: What is multi-stage Docker build?**
A: "Multiple FROM statements. First stage builds (large), second stage runs (small). Reduces final image size significantly."

**Q5: How do you optimize Docker images?**
A: "Multi-stage builds. Alpine base images. Layer caching (COPY package.json first). .dockerignore file. Minimize layers."

**Q6: What is CI/CD?**
A: "Continuous Integration: auto test on push. Continuous Deployment: auto deploy when tests pass. Benefits: catch bugs early, faster releases."

**Q7: What should a CI pipeline include?**
A: "Install dependencies. Lint code. Run tests. Build application. Generate coverage. Optional: security scan, Docker build."

**Q8: How do you deploy to production?**
A: "Vercel/Netlify for frontend. Railway/Render for backend. AWS/GCP for enterprise. Docker for consistency. PM2 for Node.js."

**Q9: What is blue-green deployment?**
A: "Two identical environments. Switch traffic from blue to green. Instant rollback. Zero downtime."

**Q10: What is canary deployment?**
A: "Route small percentage to new version. Monitor for issues. Gradually increase traffic. Rollback if problems."

**Q11: How do you manage environment variables?**
A: ".env files for local (never commit). CI/CD secrets for pipelines. Cloud secret managers for production. Validate on startup."

**Q12: What is container orchestration?**
A: "Manage multiple containers. Kubernetes, Docker Swarm, ECS. Handles scaling, load balancing, health checks, rolling updates."

**Q13: How do you handle database migrations in deployment?**
A: "Run migrations before deployment. Rollback capability. Blue-green with migration compatibility. Zero-downtime migrations."

**Q14: What is infrastructure as code?**
A: "Define infrastructure in code. Tools: Terraform, CloudFormation, Pulumi. Version control. Reproducible environments."

**Q15: How do you monitor production?**
A: "APM tools (Datadog, New Relic). Log aggregation (ELK). Metrics (Prometheus, Grafana). Alerting on anomalies."

**Q16: What is load balancing?**
A: "Distribute requests across servers. Algorithms: round robin, least connections. Health checks. Tools: Nginx, HAProxy, AWS ALB."

**Q17: How do you handle secrets in Docker?**
A: "Docker secrets. Environment variables (not in image). Secret managers (Vault, AWS Secrets Manager). Never in Dockerfile."

**Q18: What is Kubernetes?**
A: "Container orchestration platform. Manages containers at scale. Handles scaling, networking, storage. Complex but powerful."

**Q19: How do you rollback a deployment?**
A: "Keep previous version. Switch traffic back. Database rollback if needed. CI/CD rollback commands. Blue-green makes it instant."

**Q20: What is horizontal pod autoscaling?**
A: "Automatically scale pods based on metrics (CPU, memory). Kubernetes feature. Configure thresholds. Handle scaling events."

**Q21: How do you handle logging in containers?**
A: "Log to stdout/stderr. Collect with log driver. Centralize with ELK, Fluentd. Structured logging (JSON)."

**Q22: What is health check in Docker?**
A: "HEALTHCHECK instruction in Dockerfile. Docker monitors container health. Orchestrator restarts unhealthy containers."

**Q23: How do you test Docker images?**
A: "Build in CI. Run tests in container. Security scanning (Trivy). Integration tests with docker-compose."

**Q24: What is service discovery?**
A: "How services find each other. DNS-based, environment variables, service registry. Tools: Consul, Kubernetes DNS."

**Q25: How do you handle stateful applications?**
A: "Persistent volumes. External databases. StatefulSets in Kubernetes. Avoid storing state in containers."

---

### Complete Docker Implementation

```dockerfile
# Production Dockerfile
FROM node:18-alpine AS base
WORKDIR /app
COPY package*.json ./

# Development
FROM base AS development
RUN npm ci
COPY . .
CMD ["npm", "run", "dev"]

# Build
FROM base AS builder
RUN npm ci
COPY . .
RUN npm run build

# Production
FROM node:18-alpine AS production
WORKDIR /app
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
RUN npm ci --only=production && npm cache clean --force
USER nodejs
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1
CMD ["node", "dist/index.js"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build:
      context: .
      target: production
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:pass@postgres:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydb"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./certs:/etc/nginx/certs
    depends_on:
      - api
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

---

### Additional Interview Questions (20+)

**Q26: How do you implement Docker health checks?**
A: "HEALTHCHECK instruction in Dockerfile. Check endpoint, port, or command. Configure interval, timeout, retries. Orchestrator restarts unhealthy containers."

**Q27: What is Docker layer caching?**
A: "Docker caches layers. Copy package.json before code. Only rebuild when dependencies change. Reduces build time significantly."

**Q28: How do you handle Docker secrets?**
A: "Docker secrets for sensitive data. Environment variables for non-sensitive. Secret managers (Vault, AWS Secrets Manager). Never in Dockerfile."

**Q29: What is Kubernetes deployment strategies?**
A: "Rolling update: gradual replacement. Blue-green: instant switch. Canary: small percentage first. Recreate: stop all, start new."

**Q30: How do you monitor containers?**
A: "Container metrics (CPU, memory, network). Log aggregation (ELK, Fluentd). APM (Datadog, New Relic). Prometheus, Grafana."

**Q31: What is Docker networking?**
A: "Bridge: default, container-to-container. Host: share host network. None: isolated. Overlay: multi-host. Custom networks."

**Q32: How do you handle database migrations in Docker?**
A: "Run migrations in entrypoint script. Use init scripts. Separate migration service. Run before application starts."

**Q33: What is container security best practices?**
A: "Use minimal base images. Run as non-root user. Scan for vulnerabilities. Don't store secrets in images. Use read-only filesystem."

**Q34: How do you implement CI/CD with Docker?**
A: "Build image in CI. Run tests in container. Push to registry. Deploy to staging. Run E2E tests. Deploy to production."

**Q35: What is Docker volume types?**
A: "Named volumes: managed by Docker. Bind mounts: host directory. tmpfs: memory only. Use named volumes for persistence."

**Q36: How do you optimize Docker builds?**
A: "Multi-stage builds. Alpine base images. Layer caching. .dockerignore file. Minimize layers. Use Docker BuildKit."

**Q37: What is Kubernetes pod?**
A: "Smallest deployable unit. Contains one or more containers. Shared network and volumes. Managed by controllers."

**Q38: How do you handle logging in Docker?**
A: "Log to stdout/stderr. Docker log drivers. Centralized logging (ELK, Fluentd). Structured logging (JSON)."

**Q39: What is Docker Compose vs Kubernetes?**
A: "Docker Compose: local development, simple orchestration. Kubernetes: production, complex orchestration, scaling, self-healing."

**Q40: How do you implement zero-downtime deployment?**
A: "Blue-green deployment. Rolling updates. Health checks. Load balancer. Database migrations compatible with both versions."

**Q41: What is container registry?**
A: "Store Docker images. Docker Hub, ECR, GCR, ACR. Public or private. Version tags. Security scanning."

**Q42: How do you handle configuration in Docker?**
A: "Environment variables. Config files mounted as volumes. Secret managers. ConfigMaps in Kubernetes."

**Q43: What is Docker BuildKit?**
A: "Improved build system. Parallel builds. Better caching. Secret mounting. Build secrets. Enable with DOCKER_BUILDKIT=1."

**Q44: How do you test Docker images?**
A: "Build in CI. Run tests in container. Security scanning (Trivy). Integration tests with docker-compose. Smoke tests."

**Q45: What is horizontal pod autoscaling?**
A: "Automatically scale pods based on metrics (CPU, memory). Kubernetes feature. Configure thresholds. Handle scaling events."

---

*Next: [15 — Design Patterns](https://github.com/Aditya-myst/Interview-PlayBook/blob/main/BACKEND/Design-Pattern.md)*
