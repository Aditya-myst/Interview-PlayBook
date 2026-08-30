# 05 — Docker Basics

## Containerization Fundamentals

---

### What is Docker?

Docker packages applications into **containers**—lightweight, portable, self-contained units that run consistently across environments.

```
┌─────────────────────────────────────────┐
│            Virtual Machine              │
│  ┌──────────────────────────────────┐  │
│  │  Guest OS (Full OS)              │  │
│  │  ┌──────────────────────────┐   │  │
│  │  │  App + Dependencies      │   │  │
│  │  └──────────────────────────┘   │  │
│  └──────────────────────────────────┘  │
│  Host OS                               │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│            Container                    │
│  ┌──────────────────────────────────┐  │
│  │  App + Dependencies              │  │
│  │  (shares host OS kernel)         │  │
│  └──────────────────────────────────┘  │
│  Host OS + Docker Engine               │
└─────────────────────────────────────────┘
```

---

### Dockerfile

```dockerfile
# Node.js application Dockerfile
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files first (for caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Set environment variable
ENV NODE_ENV=production

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl -f http://localhost:3000/health || exit 1

# Run application
CMD ["node", "src/index.js"]
```

---

### Multi-Stage Build

```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
RUN npm ci --only=production
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

---

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Backend API
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://user:password@postgres:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
    volumes:
      - .:/app
      - /app/node_modules
    command: npm run dev

  # Frontend
  frontend:
    build: ./frontend
    ports:
      - "3001:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:3000

  # PostgreSQL
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  # Redis
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

---

### Docker Commands

```bash
# Build image
docker build -t myapp:latest .

# Run container
docker run -p 3000:3000 -d myapp:latest

# List running containers
docker ps

# Stop container
docker stop <container_id>

# View logs
docker logs <container_id>

# Execute command in container
docker exec -it <container_id> /bin/sh

# Docker Compose
docker-compose up -d          # Start all services
docker-compose down           # Stop all services
docker-compose logs -f api    # View API logs
docker-compose build          # Rebuild images
docker-compose restart api    # Restart API service
```

---

### .dockerignore

```
node_modules
npm-debug.log
.git
.gitignore
.env
.env.local
dist
build
coverage
.DS_Store
```

---

### Interview Questions

**Q: What is Docker and why use it?**

A: "Docker packages apps into containers—lightweight, portable units. Benefits: consistent environments (works on my machine = works everywhere), easy deployment, isolation, scalability. Containers share host OS kernel, unlike VMs which need full OS."

**Q: What's the difference between a Docker image and container?**

A: "Image: read-only template with app code and dependencies (like a class). Container: running instance of an image (like an object). One image can run multiple containers."

**Q: What's a multi-stage Docker build?**

A: "Using multiple FROM statements in Dockerfile. First stage builds the app (large, includes dev dependencies). Second stage copies only production artifacts (smaller final image). Reduces image size significantly."

**Q: What is Docker Compose?**

A: "Tool for defining and running multi-container applications. Uses YAML to configure services, networks, volumes. One command starts entire stack (API, database, cache). Great for development and testing."

---

### Additional Interview Questions (30+)

**Q5: How do you optimize Docker images?**
A: "Multi-stage builds. Alpine base images. Layer caching (COPY package.json first). .dockerignore file. Minimize layers. Use Docker BuildKit."

**Q6: What is Docker layer caching?**
A: "Docker caches layers. Copy package.json before code. Only rebuild when dependencies change. Reduces build time significantly."

**Q7: How do you handle Docker secrets?**
A: "Docker secrets for sensitive data. Environment variables for non-sensitive. Secret managers (Vault, AWS Secrets Manager). Never in Dockerfile."

**Q8: What is Docker networking?**
A: "Bridge: default, container-to-container. Host: share host network. None: isolated. Overlay: multi-host. Custom networks."

**Q9: How do you handle database migrations in Docker?**
A: "Run migrations in entrypoint script. Use init scripts. Separate migration service. Run before application starts."

**Q10: What is container security best practices?**
A: "Use minimal base images. Run as non-root user. Scan for vulnerabilities. Don't store secrets in images. Use read-only filesystem."

**Q11: How do you implement CI/CD with Docker?**
A: "Build image in CI. Run tests in container. Push to registry. Deploy to staging. Run E2E tests. Deploy to production."

**Q12: What is Docker volume types?**
A: "Named volumes: managed by Docker. Bind mounts: host directory. tmpfs: memory only. Use named volumes for persistence."

**Q13: How do you handle logging in Docker?**
A: "Log to stdout/stderr. Docker log drivers. Centralized logging (ELK, Fluentd). Structured logging (JSON)."

**Q14: What is Docker Compose vs Kubernetes?**
A: "Docker Compose: local development, simple orchestration. Kubernetes: production, complex orchestration, scaling, self-healing."

**Q15: How do you implement zero-downtime deployment?**
A: "Blue-green deployment. Rolling updates. Health checks. Load balancer. Database migrations compatible with both versions."

**Q16: What is container registry?**
A: "Store Docker images. Docker Hub, ECR, GCR, ACR. Public or private. Version tags. Security scanning."

**Q17: How do you handle configuration in Docker?**
A: "Environment variables. Config files mounted as volumes. Secret managers. ConfigMaps in Kubernetes."

**Q18: What is Docker BuildKit?**
A: "Improved build system. Parallel builds. Better caching. Secret mounting. Build secrets. Enable with DOCKER_BUILDKIT=1."

**Q19: How do you test Docker images?**
A: "Build in CI. Run tests in container. Security scanning (Trivy). Integration tests with docker-compose. Smoke tests."

**Q20: What is horizontal pod autoscaling?**
A: "Automatically scale pods based on metrics (CPU, memory). Kubernetes feature. Configure thresholds. Handle scaling events."

**Q21: How do you handle Docker in production?**
A: "Use orchestration (Kubernetes). Health checks. Logging. Monitoring. Resource limits. Restart policies."

**Q22: What is Docker health check?**
A: "HEALTHCHECK instruction in Dockerfile. Docker monitors container health. Orchestrator restarts unhealthy containers."

**Q23: How do you handle Docker networking?**
A: "Custom networks for service isolation. DNS for service discovery. Port mapping for external access. Network policies for security."

**Q24: What is Docker swarm?**
A: "Docker's native orchestration. Simpler than Kubernetes. Good for small deployments. Built into Docker."

**Q25: How do you handle Docker storage?**
A: "Volumes for persistent data. Bind mounts for development. tmpfs for temporary data. Don't store data in containers."

**Q26: What is Docker image tagging?**
A: "Version tags (v1.0.0). Latest tag (avoid in production). SHA tags for immutability. Semantic versioning."

**Q27: How do you handle Docker dependencies?**
A: "Multi-stage builds. Separate build and runtime dependencies. Use package managers. Minimize installed packages."

**Q28: What is Docker container orchestration?**
A: "Manage multiple containers. Kubernetes, Docker Swarm, ECS. Handles scaling, load balancing, health checks."

**Q29: How do you debug Docker containers?**
A: "docker logs for output. docker exec for shell access. docker inspect for configuration. Debug images with tools."

**Q30: What is Docker security scanning?**
A: "Scan images for vulnerabilities. Tools: Trivy, Snyk, Docker Scout. Run in CI/CD. Fix critical vulnerabilities."

---

*Next: [06 — Deployment](06-Deployment.md)*
