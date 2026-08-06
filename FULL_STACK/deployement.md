# 06 — Deployment

## Getting Code to Production

---

### Deployment Options

| Platform | Type | Best For |
|----------|------|----------|
| **Vercel** | PaaS | Next.js, static sites |
| **Netlify** | PaaS | Static sites, JAMstack |
| **Railway** | PaaS | Full stack apps |
| **Render** | PaaS | Web services, databases |
| **AWS** | IaaS | Enterprise, full control |
| **Google Cloud** | IaaS | Enterprise, ML |
| **Azure** | IaaS | Enterprise, .NET |
| **DigitalOcean** | IaaS | Simple VPS |
| **Heroku** | PaaS | Prototypes, small apps |

---

### Deploying to Vercel (Next.js)

```bash
# Install Vercel CLI
npm i -g vercel

# Login
vercel login

# Deploy
vercel

# Deploy to production
vercel --prod
```

```json
// vercel.json
{
    "builds": [
        { "src": "package.json", "use": "@vercel/next" }
    ],
    "env": {
        "DATABASE_URL": "@database-url",
        "JWT_SECRET": "@jwt-secret"
    }
}
```

---

### Deploying to Railway

```bash
# Install Railway CLI
npm i -g @railway/cli

# Login
railway login

# Init project
railway init

# Deploy
railway up
```

---

### Deploying to AWS (EC2)

```bash
# 1. SSH into EC2 instance
ssh -i key.pem ec2-user@your-ec2-ip

# 2. Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# 3. Clone repository
git clone https://github.com/your/repo.git
cd repo

# 4. Install dependencies
npm install

# 5. Set environment variables
export DATABASE_URL=postgresql://...
export JWT_SECRET=...

# 6. Start with PM2
npm install -g pm2
pm2 start src/index.js --name myapp
pm2 startup  # Auto-start on reboot
pm2 save

# 7. Setup Nginx reverse proxy
sudo apt install nginx
```

```nginx
# /etc/nginx/sites-available/myapp
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

### Deploying with Docker

```bash
# Build and push to Docker Hub
docker build -t yourusername/myapp:latest .
docker push yourusername/myapp:latest

# On server
docker pull yourusername/myapp:latest
docker run -d -p 3000:3000 --env-file .env yourusername/myapp:latest
```

---

### Process Managers (PM2)

```bash
# Start application
pm2 start src/index.js --name myapp

# Start with cluster mode (multiple instances)
pm2 start src/index.js -i max --name myapp

# View running processes
pm2 list

# View logs
pm2 logs myapp

# Restart
pm2 restart myapp

# Stop
pm2 stop myapp

# Monitor
pm2 monit
```

```javascript
// ecosystem.config.js
module.exports = {
    apps: [{
        name: 'myapp',
        script: 'src/index.js',
        instances: 'max',
        exec_mode: 'cluster',
        env: {
            NODE_ENV: 'development',
        },
        env_production: {
            NODE_ENV: 'production',
        },
    }],
};
```

---

### SSL/HTTPS with Let's Encrypt

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Get certificate
sudo certbot --nginx -d yourdomain.com

# Auto-renewal (cron job)
sudo crontab -e
# Add: 0 0 1 * * certbot renew --quiet
```

---

### Interview Questions

**Q: How would you deploy a full stack application?**

A: "Frontend: deploy to Vercel/Netlify (CDN, automatic SSL). Backend: deploy to Railway/Render or EC2 with PM2. Database: managed service (AWS RDS, Railway PostgreSQL). Use Docker for consistency. Set up CI/CD for automatic deployment."

**Q: What's the difference between PaaS and IaaS?**

A: "PaaS (Platform as a Service): managed platform, deploy code only (Vercel, Heroku, Railway). IaaS (Infrastructure as a Service): manage servers yourself (AWS EC2, DigitalOcean). PaaS is easier; IaaS gives more control."

**Q: What is PM2?**

A: "Process manager for Node.js. Keeps app running (auto-restart on crash), cluster mode (multiple instances), log management, monitoring. Essential for production Node.js deployments."

---

*Next: [07 — CI/CD](07-CICD.md)*
