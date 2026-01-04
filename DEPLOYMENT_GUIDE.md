# Complete Deployment Guide

This guide covers all deployment options from development to production, including Docker, systemd, Kubernetes, and cloud platforms.

## Table of Contents
1. [Local Development](#local-development)
2. [Docker Deployment](#docker-deployment)
3. [Linux System Service](#linux-system-service)
4. [Kubernetes Deployment](#kubernetes-deployment)
5. [Cloud Deployments](#cloud-deployments)
6. [Production Checklist](#production-checklist)
7. [Monitoring & Maintenance](#monitoring--maintenance)

## Local Development

### Quick Start
```bash
# 1. Setup environment
python3 -m venv venv
source venv/bin/activate
pip install -r requirements_api.txt

# 2. Configure environment
cp .env.example .env
# Edit .env with your API key

# 3. Run the service
python api_service.py
```

### Development with hot reload
```bash
uvicorn api_service:app --reload --host 0.0.0.0 --port 8000
```

## Docker Deployment

### Single Container
```bash
# Build image
docker build -t webcategorizer:latest .

# Run container
docker run -d \
  --name webcategorizer \
  -p 8000:8000 \
  -e OPENROUTER_API_KEY="your_key" \
  -e API_KEYS="key1,key2" \
  --restart unless-stopped \
  webcategorizer:latest
```

### Docker Compose (Recommended)
```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f webcategorizer

# Stop services
docker-compose down
```

### Production Docker Compose
Create `docker-compose.prod.yml`:
```yaml
version: '3.8'

services:
  webcategorizer:
    image: ghcr.io/yourusername/webcategorizer:latest
    container_name: webcategorizer
    ports:
      - "127.0.0.1:8000:8000"
    environment:
      - OPENROUTER_API_KEY=${OPENROUTER_API_KEY}
      - API_KEYS=${API_KEYS}
      - WORKERS=8
      - LOG_LEVEL=warning
    volumes:
      - ./logs:/app/logs
      - ./data:/app/data
    restart: always
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '1.0'
          memory: 1G
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/"]
      interval: 30s
      timeout: 3s
      retries: 3

  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - ./ssl:/etc/ssl/certs:ro
      - ./logs/nginx:/var/log/nginx
    depends_on:
      - webcategorizer
    restart: always

  redis:
    image: redis:7-alpine
    container_name: redis
    command: redis-server --maxmemory 512mb --maxmemory-policy allkeys-lru
    volumes:
      - redis-data:/data
    restart: always

volumes:
  redis-data:
```

## Linux System Service

### 1. Install as systemd service
```bash
# Copy service file
sudo cp webcategorizer.service /etc/systemd/system/

# Create user
sudo useradd --system --no-create-home webcategorizer

# Setup directory
sudo mkdir -p /opt/website-categorizer
sudo cp -r * /opt/website-categorizer/
sudo chown -R webcategorizer:webcategorizer /opt/website-categorizer

# Setup virtual environment
cd /opt/website-categorizer
sudo -u webcategorizer python3 -m venv venv
sudo -u webcategorizer ./venv/bin/pip install -r requirements_api.txt

# Configure environment
sudo cp .env.example /opt/website-categorizer/.env
sudo nano /opt/website-categorizer/.env  # Add your API key

# Start service
sudo systemctl daemon-reload
sudo systemctl enable webcategorizer
sudo systemctl start webcategorizer

# Check status
sudo systemctl status webcategorizer
```

### 2. Service Management
```bash
# View logs
sudo journalctl -u webcategorizer -f

# Restart service
sudo systemctl restart webcategorizer

# Stop service
sudo systemctl stop webcategorizer
```

## Kubernetes Deployment

### 1. Create ConfigMap
```yaml
# config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: webcategorizer-config
data:
  DEFAULT_MODEL: "anthropic/claude-3.5-sonnet"
  CACHE_TTL: "3600"
  WORKERS: "4"
  LOG_LEVEL: "info"
```

### 2. Create Secret
```bash
kubectl create secret generic webcategorizer-secrets \
  --from-literal=OPENROUTER_API_KEY=your_key \
  --from-literal=API_KEYS=key1,key2
```

### 3. Create Deployment
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webcategorizer
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webcategorizer
  template:
    metadata:
      labels:
        app: webcategorizer
    spec:
      containers:
      - name: webcategorizer
        image: ghcr.io/yourusername/webcategorizer:latest
        ports:
        - containerPort: 8000
        envFrom:
        - configMapRef:
            name: webcategorizer-config
        - secretRef:
            name: webcategorizer-secrets
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
```

### 4. Create Service
```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: webcategorizer-service
spec:
  selector:
    app: webcategorizer
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: LoadBalancer
```

### 5. Deploy to Kubernetes
```bash
kubectl apply -f config.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Check deployment
kubectl get pods
kubectl get services
kubectl logs -l app=webcategorizer
```

## Cloud Deployments

### AWS EC2
```bash
# 1. Launch EC2 instance (Ubuntu 22.04)
# 2. SSH into instance
ssh -i your-key.pem ubuntu@your-instance-ip

# 3. Install dependencies
sudo apt update
sudo apt install -y python3-pip python3-venv nginx

# 4. Clone repository
git clone https://github.com/yourusername/website-categorizer.git
cd website-categorizer

# 5. Run setup
sudo bash setup.sh

# 6. Configure Nginx
sudo cp nginx.conf /etc/nginx/sites-available/webcategorizer
sudo ln -s /etc/nginx/sites-available/webcategorizer /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

### Google Cloud Run
```bash
# 1. Build and push image
gcloud builds submit --tag gcr.io/PROJECT_ID/webcategorizer

# 2. Deploy to Cloud Run
gcloud run deploy webcategorizer \
  --image gcr.io/PROJECT_ID/webcategorizer \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars OPENROUTER_API_KEY=your_key \
  --memory 1Gi \
  --cpu 2 \
  --min-instances 1 \
  --max-instances 10
```

### Azure Container Instances
```bash
# 1. Create resource group
az group create --name webcategorizer-rg --location eastus

# 2. Create container
az container create \
  --resource-group webcategorizer-rg \
  --name webcategorizer \
  --image ghcr.io/yourusername/webcategorizer:latest \
  --dns-name-label webcategorizer \
  --ports 8000 \
  --environment-variables \
    OPENROUTER_API_KEY=your_key \
    API_KEYS=key1,key2 \
  --cpu 2 \
  --memory 2
```

### Heroku
```bash
# 1. Create Procfile
echo "web: uvicorn api_service:app --host 0.0.0.0 --port $PORT" > Procfile

# 2. Create app
heroku create your-app-name

# 3. Set config
heroku config:set OPENROUTER_API_KEY=your_key

# 4. Deploy
git push heroku main

# 5. Scale
heroku ps:scale web=1
```

## Production Checklist

### Security
- [ ] API keys stored securely (environment variables or secrets management)
- [ ] HTTPS/TLS configured
- [ ] Rate limiting enabled
- [ ] Authentication required for API endpoints
- [ ] Firewall rules configured
- [ ] Regular security updates applied
- [ ] Input validation and sanitization
- [ ] CORS configured appropriately

### Performance
- [ ] Caching enabled (Redis/Memcached)
- [ ] Database indexes optimized (if using DB)
- [ ] Load balancing configured
- [ ] Auto-scaling setup
- [ ] CDN for static assets
- [ ] Compression enabled
- [ ] Connection pooling configured

### Monitoring
- [ ] Health checks configured
- [ ] Logging aggregation setup (ELK/CloudWatch)
- [ ] Metrics collection (Prometheus/Grafana)
- [ ] Error tracking (Sentry)
- [ ] Uptime monitoring (UptimeRobot/Pingdom)
- [ ] Performance monitoring (New Relic/DataDog)
- [ ] Alerting configured

### Backup & Recovery
- [ ] Regular backups scheduled
- [ ] Backup restoration tested
- [ ] Disaster recovery plan documented
- [ ] Failover mechanism in place
- [ ] Data retention policy defined

### Documentation
- [ ] API documentation up to date
- [ ] Deployment procedures documented
- [ ] Runbook for common issues
- [ ] Contact information for escalation
- [ ] Change log maintained

## Monitoring & Maintenance

### Start Monitoring
```bash
# Run monitoring script
python monitor.py

# Or as a service
sudo cp monitor.service /etc/systemd/system/
sudo systemctl enable monitor
sudo systemctl start monitor
```

### View Metrics
```bash
# API statistics
curl http://localhost:8000/stats

# System metrics
docker stats webcategorizer

# Logs
tail -f logs/app.log
tail -f logs/error.log
```

### Regular Maintenance
```bash
# Weekly: Clear old logs
find logs/ -name "*.log" -mtime +30 -delete

# Monthly: Update dependencies
pip install --upgrade -r requirements.txt

# Quarterly: Security audit
pip install safety
safety check

# As needed: Clear cache
curl -X DELETE http://localhost:8000/cache
```

### Troubleshooting

#### Service won't start
```bash
# Check logs
sudo journalctl -u webcategorizer -n 100
# Check port
sudo lsof -i :8000
# Test manually
cd /opt/website-categorizer
source venv/bin/activate
python api_service.py
```

#### High memory usage
```bash
# Restart service
sudo systemctl restart webcategorizer
# Adjust workers
export WORKERS=2
# Clear cache
redis-cli FLUSHALL
```

#### Slow responses
```bash
# Check CPU
top -p $(pgrep -f webcategorizer)
# Check network
ping api.openrouter.ai
# Use faster model
export DEFAULT_MODEL=anthropic/claude-3-haiku
```

## Support

For production support:
1. Check logs first
2. Review this guide
3. Search existing issues
4. Create detailed bug report
5. Include system information

---

**Remember**: Always test in staging before deploying to production!
