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

*Next: [06 — Deployment](06-Deployment.md)*
