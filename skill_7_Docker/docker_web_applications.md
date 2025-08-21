# Docker for Web Applications - Enhanced Lab Guide
## Deploying Production-Ready Web Applications with Docker

### Lab Objectives
By the end of this lab, students will be able to:

- Create production-ready Dockerfiles for Python Flask web applications
- Build optimized Docker images and manage container lifecycles
- Configure advanced port mapping and networking for web services
- Implement multi-container architectures with database integration
- Master Docker Compose for complex application orchestration
- Deploy scalable web applications in containerized environments
- Apply production best practices for containerized deployments
- Troubleshoot common container deployment issues

### Prerequisites
**System Requirements:**
- Ubuntu 20.04 LTS or newer (4GB RAM, 2 CPU cores, 20GB disk space)
- Docker Engine 20.10+ and Docker Compose V2
- Basic understanding of HTTP protocols and networking concepts
- Python 3.8+ familiarity (helpful but not required)
- Command-line interface proficiency

**Environment Verification:**
```bash
# Verify system specifications
free -h
# Output: Shows available memory (need 4GB+)

lscpu | grep "CPU(s)"
# Output: Shows CPU count (need 2+)

df -h /
# Output: Shows disk space (need 20GB+ available)

# Verify Docker installation
docker --version
# Output: Docker version 20.10.x or newer

docker-compose --version
# Output: Docker Compose version v2.x.x or newer
```

### Lab Environment Setup
**Ready-to-Use Cloud Environment:** Al Nafi provides pre-configured Ubuntu machines with Docker pre-installed. Click "Start Lab" to access your environment.

**Verification Commands:**
```bash
# Test Docker daemon
docker info | grep "Server Version"
# Output: Server Version: 20.10.x (confirms Docker is running)

# Test Docker Compose
docker-compose version
# Output: Shows compose version and build info

# Check available space for images
docker system df
# Output: Shows Docker disk usage summary
```

---

## Task 1: Create Enterprise Flask Application (15 minutes)

### Subtask 1.1: Project Structure Setup
**Why:** Proper project structure enables maintainable, scalable applications and follows industry best practices.

```bash
# Create organized project directory
mkdir -p flask-docker-app/{app,database,config,scripts}
cd flask-docker-app

# Verify structure
tree . || ls -la
# Output: Shows directory hierarchy with app/, database/, config/, scripts/
```

### Subtask 1.2: Advanced Flask Application Development
Create a production-ready Flask application with comprehensive features:

```bash
nano app/app.py
```

**Application Code:**
```python
from flask import Flask, render_template, request, jsonify, session
import os
import sqlite3
import logging
from datetime import datetime
import hashlib

app = Flask(__name__)
app.secret_key = os.environ.get('SECRET_KEY', 'dev-key-change-in-production')

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Database configuration
DATABASE = '/app/data/visitors.db'

def init_db():
    """Initialize database with comprehensive schema"""
    os.makedirs('/app/data', exist_ok=True)
    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()
    
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS visitors (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            ip_address TEXT,
            visit_time TIMESTAMP,
            user_agent TEXT,
            session_id TEXT,
            page_path TEXT,
            response_time REAL
        )
    ''')
    
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS api_metrics (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            endpoint TEXT,
            method TEXT,
            status_code INTEGER,
            response_time REAL,
            timestamp TIMESTAMP
        )
    ''')
    
    conn.commit()
    conn.close()
    logger.info("Database initialized successfully")

def log_visitor(ip_address, user_agent, session_id, page_path):
    """Enhanced visitor logging with metrics"""
    try:
        conn = sqlite3.connect(DATABASE)
        cursor = conn.cursor()
        cursor.execute('''
            INSERT INTO visitors (ip_address, visit_time, user_agent, session_id, page_path)
            VALUES (?, ?, ?, ?, ?)
        ''', (ip_address, datetime.now(), user_agent, session_id, page_path))
        conn.commit()
        conn.close()
        logger.info(f"Visitor logged: {ip_address} -> {page_path}")
    except Exception as e:
        logger.error(f"Database error: {e}")

def get_analytics_data():
    """Retrieve comprehensive analytics"""
    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()
    
    # Total visitors
    cursor.execute('SELECT COUNT(DISTINCT session_id) FROM visitors')
    unique_visitors = cursor.fetchone()[0]
    
    # Total page views
    cursor.execute('SELECT COUNT(*) FROM visitors')
    total_views = cursor.fetchone()[0]
    
    # Popular pages
    cursor.execute('''
        SELECT page_path, COUNT(*) as views 
        FROM visitors 
        GROUP BY page_path 
        ORDER BY views DESC 
        LIMIT 5
    ''')
    popular_pages = cursor.fetchall()
    
    conn.close()
    
    return {
        'unique_visitors': unique_visitors,
        'total_views': total_views,
        'popular_pages': popular_pages
    }

@app.before_request
def before_request():
    """Request middleware for session management"""
    if 'session_id' not in session:
        session['session_id'] = hashlib.md5(
            f"{request.remote_addr}{datetime.now()}".encode()
        ).hexdigest()

@app.route('/')
def home():
    """Enhanced home page with analytics"""
    start_time = datetime.now()
    
    # Log visitor
    ip_address = request.environ.get('HTTP_X_FORWARDED_FOR', request.remote_addr)
    user_agent = request.environ.get('HTTP_USER_AGENT', 'Unknown')
    log_visitor(ip_address, user_agent, session['session_id'], '/')
    
    # Get analytics
    analytics = get_analytics_data()
    
    response_time = (datetime.now() - start_time).total_seconds()
    
    return f'''
    <!DOCTYPE html>
    <html>
    <head>
        <title>Enterprise Flask Docker App</title>
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <style>
            body {{ font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; margin: 0; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); min-height: 100vh; }}
            .container {{ max-width: 1200px; margin: 0 auto; padding: 40px 20px; }}
            .card {{ background: rgba(255,255,255,0.95); padding: 30px; border-radius: 15px; box-shadow: 0 8px 32px rgba(31, 38, 135, 0.37); margin-bottom: 20px; }}
            .header {{ text-align: center; color: #333; margin-bottom: 30px; }}
            .metrics {{ display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 20px; margin: 30px 0; }}
            .metric-card {{ background: #f8f9fa; padding: 20px; border-radius: 10px; text-align: center; border-left: 4px solid #667eea; }}
            .metric-value {{ font-size: 2em; font-weight: bold; color: #667eea; }}
            .metric-label {{ color: #666; margin-top: 5px; }}
            .features {{ list-style: none; padding: 0; }}
            .features li {{ padding: 10px; margin: 5px 0; background: #e9ecef; border-radius: 5px; }}
        </style>
    </head>
    <body>
        <div class="container">
            <div class="card">
                <div class="header">
                    <h1>🚀 Enterprise Flask Docker Application</h1>
                    <p>Production-ready containerized web application</p>
                </div>
                
                <div class="metrics">
                    <div class="metric-card">
                        <div class="metric-value">{analytics['unique_visitors']}</div>
                        <div class="metric-label">Unique Visitors</div>
                    </div>
                    <div class="metric-card">
                        <div class="metric-value">{analytics['total_views']}</div>
                        <div class="metric-label">Total Page Views</div>
                    </div>
                    <div class="metric-card">
                        <div class="metric-value">{response_time:.3f}s</div>
                        <div class="metric-label">Response Time</div>
                    </div>
                    <div class="metric-card">
                        <div class="metric-value">{session['session_id'][:8]}</div>
                        <div class="metric-label">Session ID</div>
                    </div>
                </div>
                
                <h3>🏗️ Architecture Features:</h3>
                <ul class="features">
                    <li>📊 Real-time analytics and visitor tracking</li>
                    <li>🔒 Session management and security</li>
                    <li>📝 Comprehensive logging and monitoring</li>
                    <li>🗄️ SQLite database integration</li>
                    <li>🌐 Production-ready containerization</li>
                    <li>⚖️ Load balancer and reverse proxy ready</li>
                </ul>
                
                <div style="margin-top: 30px; text-align: center;">
                    <a href="/analytics" style="margin: 0 10px; padding: 10px 20px; background: #667eea; color: white; text-decoration: none; border-radius: 5px;">📈 Analytics</a>
                    <a href="/health" style="margin: 0 10px; padding: 10px 20px; background: #28a745; color: white; text-decoration: none; border-radius: 5px;">🔍 Health Check</a>
                    <a href="/api/metrics" style="margin: 0 10px; padding: 10px 20px; background: #ffc107; color: black; text-decoration: none; border-radius: 5px;">📊 API Metrics</a>
                </div>
            </div>
        </div>
    </body>
    </html>
    '''

@app.route('/health')
def health_check():
    """Comprehensive health endpoint"""
    try:
        # Database connectivity test
        conn = sqlite3.connect(DATABASE)
        cursor = conn.cursor()
        cursor.execute('SELECT COUNT(*) FROM visitors')
        visitor_count = cursor.fetchone()[0]
        conn.close()
        
        return jsonify({
            'status': 'healthy',
            'timestamp': datetime.now().isoformat(),
            'database': 'connected',
            'visitors': visitor_count,
            'uptime': 'running',
            'version': '2.0'
        }), 200
    except Exception as e:
        return jsonify({
            'status': 'unhealthy',
            'error': str(e),
            'timestamp': datetime.now().isoformat()
        }), 503

@app.route('/analytics')
def analytics_dashboard():
    """Analytics dashboard"""
    analytics = get_analytics_data()
    
    pages_html = '<br>'.join([
        f"<li>{page}: {views} views</li>" 
        for page, views in analytics['popular_pages']
    ])
    
    return f'''
    <html>
    <head><title>Analytics Dashboard</title></head>
    <body style="font-family: Arial, sans-serif; margin: 40px;">
        <h1>📊 Analytics Dashboard</h1>
        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px;">
            <div style="background: #f0f8ff; padding: 20px; border-radius: 10px;">
                <h3>Visitor Statistics</h3>
                <p><strong>Unique Visitors:</strong> {analytics['unique_visitors']}</p>
                <p><strong>Total Page Views:</strong> {analytics['total_views']}</p>
            </div>
            <div style="background: #f0fff0; padding: 20px; border-radius: 10px;">
                <h3>Popular Pages</h3>
                <ul>{pages_html}</ul>
            </div>
        </div>
        <br><a href="/">← Back to Home</a>
    </body>
    </html>
    '''

@app.route('/api/metrics')
def api_metrics():
    """API metrics endpoint"""
    analytics = get_analytics_data()
    
    return jsonify({
        'metrics': analytics,
        'system': {
            'timestamp': datetime.now().isoformat(),
            'database_status': 'operational'
        }
    })

if __name__ == '__main__':
    init_db()
    app.run(host='0.0.0.0', port=5000, debug=False)
```

### Subtask 1.3: Production Dependencies
Create comprehensive requirements file:

```bash
nano app/requirements.txt
```

**Requirements Content:**
```
Flask==2.3.3
Werkzeug==2.3.7
redis==5.0.1
gunicorn==21.2.0
psutil==5.9.5
```

**Verification:**
```bash
# Check Python version compatibility
python3 --version
# Output: Python 3.8.x or newer

# Validate requirements syntax
cat app/requirements.txt | grep -E '^[a-zA-Z]'
# Output: Lists all package names (validates format)
```

---

## Task 2: Production Dockerfile Creation (20 minutes)

### Subtask 2.1: Multi-Stage Production Dockerfile
**Why:** Multi-stage builds reduce image size and improve security by excluding build tools from final images.

```bash
nano Dockerfile
```

**Production Dockerfile:**
```dockerfile
# Multi-stage build for production optimization
FROM python:3.9-slim as builder

# Set build arguments
ARG BUILD_DATE
ARG VERSION=1.0.0

# Install build dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Create virtual environment
WORKDIR /app
COPY app/requirements.txt .
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
RUN pip install --no-cache-dir --upgrade pip \
    && pip install --no-cache-dir -r requirements.txt

# Production stage
FROM python:3.9-slim

# Add labels for metadata
LABEL maintainer="devops@company.com"
LABEL version="${VERSION}"
LABEL build-date="${BUILD_DATE}"
LABEL description="Enterprise Flask Web Application"

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PATH="/opt/venv/bin:$PATH" \
    FLASK_ENV=production \
    FLASK_DEBUG=False

# Create app user for security
RUN groupadd -r appgroup && useradd -r -g appgroup appuser

# Copy virtual environment from builder
COPY --from=builder /opt/venv /opt/venv

# Set working directory
WORKDIR /app

# Copy application code
COPY --chown=appuser:appgroup app/ .

# Create directories with proper permissions
RUN mkdir -p /app/data /app/logs && \
    chown -R appuser:appgroup /app

# Install curl for health checks
RUN apt-get update && apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*

# Expose port
EXPOSE 5000

# Health check configuration
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:5000/health || exit 1

# Switch to non-root user
USER appuser

# Production command with Gunicorn
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "--timeout", "120", "app:app"]
```

### Subtask 2.2: Docker Ignore Configuration
```bash
nano .dockerignore
```

**Optimized .dockerignore:**
```
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/

# Virtual environments
venv/
ENV/
env/
.venv/

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Database
*.db
*.sqlite3

# Docker
.dockerignore
Dockerfile*

# Git
.git/
.gitignore

# Testing
.coverage
.pytest_cache/
.tox/
htmlcov/

# Documentation
README.md
docs/
```

**Why This Matters:** Proper .dockerignore reduces build context size, improving build speed and security.

```bash
# Verify ignore patterns work
docker build --no-cache -t flask-test . 2>&1 | grep "Sending build context"
# Output: Shows reduced context size (should be < 50KB)
```

---

## Task 3: Advanced Container Operations (25 minutes)

### Subtask 3.1: Optimized Image Building
```bash
# Build with build arguments and labels
docker build \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  --build-arg VERSION=2.0.0 \
  -t flask-enterprise:v2.0 \
  -t flask-enterprise:latest \
  .

# Verify build success and image details
docker images flask-enterprise
# Output: Shows image size, created time, tags

# Inspect image metadata
docker inspect flask-enterprise:latest | grep -A 10 "Labels"
# Output: Shows build metadata and labels
```

**Expected Build Output Analysis:**
```bash
# Analyze image layers
docker history flask-enterprise:latest
# Output: Shows layer sizes (multi-stage should show reduced final size)

# Check image size optimization
docker images | head -5
# Output: Compare flask-enterprise size to base python image
```

### Subtask 3.2: Production Container Deployment
```bash
# Create persistent volume for data
docker volume create flask-app-data

# Verify volume creation
docker volume ls | grep flask-app-data
# Output: Shows volume name and driver information

# Deploy production container with comprehensive configuration
docker run -d \
  --name flask-prod-app \
  --restart unless-stopped \
  -p 8080:5000 \
  -v flask-app-data:/app/data \
  -e SECRET_KEY="$(openssl rand -hex 32)" \
  -e FLASK_ENV=production \
  --memory=512m \
  --cpus=1.0 \
  --health-cmd="curl -f http://localhost:5000/health || exit 1" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  flask-enterprise:latest

# Verify container deployment
docker ps | grep flask-prod-app
# Output: Shows running status, ports, health status
```

### Subtask 3.3: Container Health and Performance Monitoring
```bash
# Monitor container health status
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
# Output: Shows container name, health status, port mappings

# Real-time resource monitoring
docker stats flask-prod-app --no-stream
# Output: CPU %, Memory usage/limit, Network I/O, Block I/O

# Check application logs
docker logs --tail 20 flask-prod-app
# Output: Application startup logs and access patterns

# Test application endpoints
curl -s http://localhost:8080/health | python3 -m json.tool
# Output: JSON health check response with database status
```

**Performance Testing:**
```bash
# Simple load test
for i in {1..10}; do
  echo "Request $i: $(curl -s -w '%{http_code} - %{time_total}s\n' -o /dev/null http://localhost:8080/)"
done
# Output: HTTP status codes and response times for each request
```

---

## Task 4: Multi-Container Architecture with Docker Compose (30 minutes)

### Subtask 4.1: Advanced Docker Compose Configuration
**Why:** Production environments require orchestration of multiple services including databases, caching, and load balancing.

```bash
nano docker-compose.yml
```

**Production Docker Compose:**
```yaml
version: '3.8'

services:
  web:
    build:
      context: .
      args:
        BUILD_DATE: ${BUILD_DATE:-}
        VERSION: ${VERSION:-2.0.0}
    image: flask-enterprise:compose
    container_name: flask-web-app
    restart: unless-stopped
    ports:
      - "8080:5000"
    volumes:
      - app-data:/app/data
      - app-logs:/app/logs
    environment:
      - FLASK_ENV=production
      - SECRET_KEY=${SECRET_KEY:-default-change-in-production}
      - REDIS_URL=redis://redis:6379/0
      - DATABASE_PATH=/app/data/visitors.db
    depends_on:
      redis:
        condition: service_healthy
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  redis:
    image: redis:7-alpine
    container_name: flask-redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - app-network
    command: redis-server --appendonly yes --maxmemory 256mb --maxmemory-policy allkeys-lru
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 256M

  nginx:
    image: nginx:alpine
    container_name: flask-nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./config/nginx.conf:/etc/nginx/nginx.conf:ro
      - nginx-logs:/var/log/nginx
    depends_on:
      web:
        condition: service_healthy
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:80/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 128M

  prometheus:
    image: prom/prometheus:latest
    container_name: flask-prometheus
    restart: unless-stopped
    ports:
      - "9090:9090"
    volumes:
      - ./config/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
    networks:
      - app-network

volumes:
  app-data:
    driver: local
  app-logs:
    driver: local
  redis-data:
    driver: local
  nginx-logs:
    driver: local
  prometheus-data:
    driver: local

networks:
  app-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

### Subtask 4.2: Nginx Load Balancer Configuration
```bash
# Create configuration directory
mkdir -p config

nano config/nginx.conf
```

**Production Nginx Configuration:**
```nginx
events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Logging format
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                   '$status $body_bytes_sent "$http_referer" '
                   '"$http_user_agent" "$http_x_forwarded_for" '
                   'rt=$request_time uct="$upstream_connect_time" '
                   'uht="$upstream_header_time" urt="$upstream_response_time"';

    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log warn;

    # Performance optimizations
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 10240;
    gzip_proxied expired no-cache no-store private must-revalidate auth;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=web:10m rate=10r/s;

    # Upstream configuration
    upstream flask_app {
        server web:5000 max_fails=3 fail_timeout=30s;
        keepalive 32;
    }

    server {
        listen 80;
        server_name localhost;

        # Rate limiting
        limit_req zone=web burst=20 nodelay;

        # Security headers
        add_header X-Frame-Options DENY;
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";

        location / {
            proxy_pass http://flask_app;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_cache_bypass $http_upgrade;
            
            # Timeouts
            proxy_connect_timeout 30s;
            proxy_send_timeout 30s;
            proxy_read_timeout 30s;
        }

        location /health {
            proxy_pass http://flask_app/health;
            access_log off;
        }

        location /nginx-health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
    }
}
```

### Subtask 4.3: Monitoring Configuration
```bash
nano config/prometheus.yml
```

**Prometheus Configuration:**
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  # - "first_rules.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'flask-app'
    static_configs:
      - targets: ['web:5000']
    metrics_path: '/health'
    scrape_interval: 30s

  - job_name: 'nginx'
    static_configs:
      - targets: ['nginx:80']
    metrics_path: '/nginx-health'
    scrape_interval: 30s

  - job_name: 'redis'
    static_configs:
      - targets: ['redis:6379']
    scrape_interval: 30s
```

### Subtask 4.4: Environment Configuration
```bash
nano .env
```

**Environment Variables:**
```bash
# Application Configuration
FLASK_ENV=production
SECRET_KEY=your-super-secret-key-change-in-production
VERSION=2.0.0
BUILD_DATE=2024-01-01T00:00:00Z

# Database Configuration
DATABASE_PATH=/app/data/visitors.db
REDIS_URL=redis://redis:6379/0

# Monitoring
PROMETHEUS_URL=http://prometheus:9090
```

---

## Task 5: Production Deployment and Scaling (25 minutes)

### Subtask 5.1: Clean Previous Deployments
```bash
# Stop single container if running
docker stop flask-prod-app 2>/dev/null || true
docker rm flask-prod-app 2>/dev/null || true

# Verify cleanup
docker ps -a | grep flask
# Output: Should show no running flask containers
```

### Subtask 5.2: Multi-Service Deployment
```bash
# Deploy complete application stack
docker-compose up -d --build

# Verify all services are running
docker-compose ps
# Output: Shows all services (web, redis, nginx, prometheus) as Up

# Monitor startup logs
docker-compose logs -f --tail 10
# Output: Shows startup logs from all services
```

**Wait for Health Checks:**
```bash
# Check service health status
watch -n 2 'docker-compose ps'
# Output: Monitor until all services show healthy status (Ctrl+C to exit)

# Alternative: Check individual service health
docker inspect flask-web-app | grep '"Health"' -A 10
# Output: Shows detailed health check status and history
```

### Subtask 5.3: Application Testing and Validation
```bash
# Test through Nginx load balancer (port 80)
curl -s http://localhost/ | grep -o '<title>.*</title>'
# Output: <title>Enterprise Flask Docker App</title>

# Test direct Flask application (port 8080)
curl -s http://localhost:8080/health | python3 -m json.tool
# Output: JSON health status with database connectivity info

# Test Redis connectivity
docker-compose exec redis redis-cli ping
# Output: PONG (confirms Redis is responding)

# Test analytics endpoint
curl -s http://localhost/api/metrics | python3 -m json.tool
# Output: JSON metrics data with visitor statistics
```

### Subtask 5.4: Performance and Load Testing
```bash
# Create comprehensive load test
nano scripts/load_test.sh
```

**Load Test Script:**
```bash
#!/bin/bash

echo "🚀 Starting comprehensive load test..."

# Test main application
echo "Testing main application endpoint..."
for i in {1..20}; do
    response=$(curl -s -w "HTTP: %{http_code}, Time: %{time_total}s" -o /dev/null http://localhost/)
    echo "Request $i: $response"
    sleep 0.5
done

# Test health endpoints
echo -e "\n🔍 Testing health endpoints..."
curl -s http://localhost/health | python3 -m json.tool
curl -s http://localhost:8080/health | python3 -m json.tool

# Test analytics
echo -e "\n📊 Testing analytics endpoint..."
curl -s http://localhost/api/metrics | python3 -m json.tool

# Test Redis performance
echo -e "\n⚡ Redis performance test..."
docker-compose exec -T redis redis-cli --latency-history -i 1 | head -5

echo -e "\n✅ Load test completed!"
```

```bash
# Make executable and run
chmod +x scripts/load_test.sh
./scripts/load_test.sh
# Output: Shows response times and status codes for all endpoints
```

### Subtask 5.5: Horizontal Scaling
**Why:** Production applications need to handle varying loads by scaling services dynamically.

```bash
# Scale web application to 3 instances
docker-compose up -d --scale web=3

# Verify scaling
docker-compose ps | grep web
# Output: Shows 3 web service instances running

# Test load distribution
echo "Testing load balancing across scaled instances..."
for i in {1..10}; do
    curl -s http://localhost/ | grep -o "Session ID: [a-f0-9]*" | head -1
    sleep 1
done
# Output: Shows different session IDs indicating load distribution
```

**Monitor Scaled Services:**
```bash
# Real-time resource monitoring for all web containers
docker stats $(docker-compose ps -q web) --no-stream
# Output: CPU and memory usage for each web container instance

# Check Nginx load balancing logs
docker-compose logs nginx | tail -20
# Output: Shows requests being distributed across backend servers
```

---

## Task 6: Advanced Monitoring and Management (20 minutes)

### Subtask 6.1: Container Resource Monitoring
```bash
# Comprehensive system monitoring
docker system df
# Output: Shows Docker disk usage breakdown

# Monitor all compose services
docker-compose top
# Output: Shows running processes in each container

# Network inspection
docker network ls | grep flask
docker network inspect flask-docker-app_app-network | grep -A 5 "Containers"
# Output: Shows network configuration and connected containers
```

### Subtask 6.2: Database Operations and Persistence
```bash
# Access web container for database operations
docker-compose exec web /bin/bash

# Inside container: Verify database operations
python3 -c "
import sqlite3
conn = sqlite3.connect('/app/data/visitors.db')
cursor = conn.cursor()

# Show table structure
cursor.execute('PRAGMA table_info(visitors)')
columns = cursor.fetchall()
print('Database Schema:')
for col in columns:
    print(f'  {col[1]} ({col[2]})')

# Show recent data
cursor.execute('SELECT COUNT(*) FROM visitors')
total = cursor.fetchone()[0]
print(f'\nTotal visitors: {total}')

cursor.execute('SELECT ip_address, visit_time, page_path FROM visitors ORDER BY visit_time DESC LIMIT 5')
recent = cursor.fetchall()
print('\nRecent visitors:')
for visitor in recent:
    print(f'  {visitor[0]} -> {visitor[2]} at {visitor[1]}')

conn.close()
"
# Output: Database schema and recent visitor data

# Exit container
exit
```

### Subtask 6.3: Production Backup and Recovery
```bash
# Create backup directory
mkdir -p backups

# Create comprehensive backup script
nano scripts/backup.sh
```

**Backup Script:**
```bash
#!/bin/bash

BACKUP_DIR="./backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
COMPOSE_PROJECT="flask-docker-app"

echo "🔄 Starting backup process at $TIMESTAMP"

# Create timestamped backup directory
mkdir -p "$BACKUP_DIR/$TIMESTAMP"

# Backup application data volume
echo "📦 Backing up application data..."
docker run --rm \
  -v ${COMPOSE_PROJECT}_app-data:/data \
  -v $(pwd)/$BACKUP_DIR/$TIMESTAMP:/backup \
  alpine tar czf /backup/app-data.tar.gz -C /data .

# Backup Redis data volume
echo "📦 Backing up Redis data..."
docker run --rm \
  -v ${COMPOSE_PROJECT}_redis-data:/data \
  -v $(pwd)/$BACKUP_DIR/$TIMESTAMP:/backup \
  alpine tar czf /backup/redis-data.tar.gz -C /data .

# Backup configuration files
echo "📦 Backing up configuration..."
tar czf "$BACKUP_DIR/$TIMESTAMP/config-backup.tar.gz" \
  docker-compose.yml \
  Dockerfile \
  .env \
  config/ \
  app/

# Create backup manifest
echo "📋 Creating backup manifest..."
cat > "$BACKUP_DIR/$TIMESTAMP/manifest.txt" << EOF
Backup Created: $TIMESTAMP
Components:
- app-data.tar.gz: Application database and files
- redis-data.tar.gz: Redis persistent data
- config-backup.tar.gz: Configuration files and code

Docker Images:
$(docker images --format "{{.Repository}}:{{.Tag}} {{.Size}}" | grep flask)

Container Status at Backup:
$(docker-compose ps --format "table {{.Name}}\\t{{.State}}\\t{{.Status}}")
EOF

echo "✅ Backup completed: $BACKUP_DIR/$TIMESTAMP"
ls -la "$BACKUP_DIR/$TIMESTAMP/"
```

```bash
# Make executable and run backup
chmod +x scripts/backup.sh
./scripts/backup.sh
# Output: Shows backup progress and final file listing
```

### Subtask 6.4: Disaster Recovery Testing
**Intentional Failure Scenario:** Let's simulate a database corruption and recovery.

```bash
# Simulate disaster: Stop services and corrupt data
docker-compose down
docker volume rm flask-docker-app_app-data
# Output: Removes application data volume (simulates data loss)

# Attempt to restart - this should create empty database
docker-compose up -d

# Verify data loss
curl -s http://localhost/api/metrics | python3 -m json.tool
# Output: Should show zero visitors (data lost)

# Recovery process: Stop and restore from backup
docker-compose down

# Find latest backup
LATEST_BACKUP=$(ls -1t backups/ | head -1)
echo "Restoring from backup: $LATEST_BACKUP"

# Restore application data
docker volume create flask-docker-app_app-data
docker run --rm \
  -v flask-docker-app_app-data:/data \
  -v $(pwd)/backups/$LATEST_BACKUP:/backup \
  alpine tar xzf /backup/app-data.tar.gz -C /data

# Restart services
docker-compose up -d

# Wait for health checks
sleep 30

# Verify recovery
curl -s http://localhost/api/metrics | python3 -m json.tool
# Output: Should show restored visitor data
```

---

## Task 7: Production Optimization and Security (15 minutes)

### Subtask 7.1: Security Hardening
```bash
# Security scan of images
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  -v $(pwd):/tmp aquasec/trivy:latest image flask-enterprise:latest
# Output: Security vulnerability report

# Check container security best practices
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  -v $(pwd):/tmp \
  goodwithtech/dockle:latest flask-enterprise:latest
# Output: Security and best practice recommendations
```

### Subtask 7.2: Performance Optimization
```bash
# Create optimized production Dockerfile
nano Dockerfile.production
```

**Optimized Production Dockerfile:**
```dockerfile
# Production-optimized multi-stage build
FROM python:3.9-slim as base

# Build stage
FROM base as builder
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY app/requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /app/wheels -r requirements.txt

# Production stage
FROM base as production

# Security: Create non-root user
RUN groupadd -r appgroup && useradd -r -g appgroup --home-dir /app appuser

# Install runtime dependencies only
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get purge -y --auto-remove

# Copy wheels and install packages
COPY --from=builder /app/wheels /wheels
RUN pip install --no-cache /wheels/*

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    FLASK_ENV=production \
    PATH="/home/appuser/.local/bin:$PATH"

# Set working directory and copy app
WORKDIR /app
COPY --chown=appuser:appgroup app/ .

# Create directories with proper permissions
RUN mkdir -p /app/data /app/logs && \
    chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

EXPOSE 5000

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:5000/health || exit 1

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "--worker-class", "gevent", "--worker-connections", "1000", "--timeout", "120", "--keep-alive", "5", "--max-requests", "1000", "--max-requests-jitter", "100", "app:app"]
```

### Subtask 7.3: Container Registry Operations
```bash
# Build optimized image
docker build -f Dockerfile.production -t flask-enterprise:optimized .

# Compare image sizes
docker images | grep flask-enterprise | sort -k7 -h
# Output: Shows size comparison between versions

# Tag for production deployment
docker tag flask-enterprise:optimized flask-enterprise:prod-$(date +%Y%m%d)

# Verify final images
docker images flask-enterprise
# Output: Shows all tagged versions with sizes and creation dates
```

---

## Task 8: Troubleshooting and Maintenance (10 minutes)

### Common Issues and Solutions

#### Issue 1: Container Health Check Failures
**Problem:** Containers show unhealthy status

```bash
# Diagnose health check issues
docker-compose ps
# Output: Shows service status and health

# Check detailed health status
docker inspect flask-web-app | grep -A 20 '"Health"'
# Output: Shows health check command and recent results

# Manual health check test
docker-compose exec web curl -f http://localhost:5000/health
# Output: Should return HTTP 200 with JSON status
```

**Solution:**
```bash
# If health check fails, check application logs
docker-compose logs web | tail -20
# Look for Python errors or database issues

# Restart specific service
docker-compose restart web
# Output: Restarts web service only
```

#### Issue 2: Database Connection Problems
**Problem:** Application cannot connect to database

```bash
# Check database file permissions
docker-compose exec web ls -la /app/data/
# Output: Should show visitors.db with appuser ownership

# Test database accessibility
docker-compose exec web python3 -c "
import sqlite3
try:
    conn = sqlite3.connect('/app/data/visitors.db')
    cursor = conn.cursor()
    cursor.execute('SELECT COUNT(*) FROM visitors')
    print('Database accessible:', cursor.fetchone()[0], 'records')
    conn.close()
except Exception as e:
    print('Database error:', e)
"
# Output: Either success message or specific error
```

#### Issue 3: Memory and Resource Issues
**Problem:** Containers consuming excessive resources

```bash
# Monitor resource usage
docker stats --no-stream
# Output: Shows CPU and memory usage for all containers

# Check for memory leaks in application
docker-compose exec web python3 -c "
import psutil
import os
process = psutil.Process(os.getpid())
memory_info = process.memory_info()
print(f'Memory usage: {memory_info.rss / 1024 / 1024:.2f} MB')
print(f'CPU usage: {process.cpu_percent()}%')
"
# Output: Shows current process resource usage
```

### Maintenance Commands
```bash
# Clean up unused resources
docker system prune -f
# Output: Shows space reclaimed

# Update and rebuild services
docker-compose pull
docker-compose up -d --build
# Output: Updates base images and rebuilds services

# Rotate logs to prevent disk space issues
docker-compose logs --no-color > logs/app-$(date +%Y%m%d).log 2>&1
# Creates dated log file
```

---

## Final Verification and Assessment (10 minutes)

### Comprehensive System Check
```bash
# Final verification script
nano scripts/final_check.sh
```

**Final Check Script:**
```bash
#!/bin/bash

echo "🔍 Final System Verification"
echo "=========================="

echo -e "\n1. Service Status:"
docker-compose ps --format "table {{.Name}}\t{{.State}}\t{{.Status}}"

echo -e "\n2. Health Checks:"
for service in web redis nginx; do
    health=$(docker inspect flask-${service} 2>/dev/null | grep '"Status"' | tail -1 | cut -d'"' -f4)
    echo "  $service: ${health:-N/A}"
done

echo -e "\n3. Endpoint Tests:"
endpoints=("/" "/health" "/analytics" "/api/metrics")
for endpoint in "${endpoints[@]}"; do
    status=$(curl -s -o /dev/null -w "%{http_code}" http://localhost$endpoint)
    echo "  $endpoint: HTTP $status"
done

echo -e "\n4. Database Verification:"
visitor_count=$(curl -s http://localhost/api/metrics | python3 -c "import sys, json; data=json.load(sys.stdin); print(data['metrics']['unique_visitors'])" 2>/dev/null || echo "Error")
echo "  Total visitors: $visitor_count"

echo -e "\n5. Resource Usage:"
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

echo -e "\n6. Volume Status:"
docker volume ls | grep flask-docker-app

echo -e "\n✅ Verification Complete!"
```

```bash
# Run final verification
chmod +x scripts/final_check.sh
./scripts/final_check.sh
# Output: Comprehensive system status report
```

### Lab Completion Checklist
**Verify each item is working:**

- [ ] **Multi-stage Docker build** creates optimized images
- [ ] **Flask application** serves dynamic content with database
- [ ] **Health checks** report healthy status for all services
- [ ] **Load balancer** distributes requests via Nginx
- [ ] **Redis integration** provides caching capabilities
- [ ] **Persistent storage** maintains data across restarts
- [ ] **Horizontal scaling** enables multiple web instances
- [ ] **Monitoring** tracks application metrics
- [ ] **Backup/Recovery** protects against data loss
- [ ] **Security measures** use non-root containers

### Performance Benchmarks
**Expected Results:**
- **Response time:** < 200ms for main page
- **Memory usage:** < 512MB per web container
- **CPU usage:** < 50% under normal load
- **Database queries:** < 50ms average
- **Container startup:** < 30 seconds to healthy

---

## Conclusion and Next Steps

### Key Achievements
You have successfully built and deployed a **production-ready containerized web application** demonstrating:

**Technical Mastery:**
- **Advanced Dockerization** with multi-stage builds and security hardening
- **Microservices Architecture** using Docker Compose orchestration
- **Load Balancing** and reverse proxy configuration with Nginx
- **Data Persistence** and backup/recovery procedures
- **Horizontal Scaling** for high availability deployments
- **Monitoring and Observability** with health checks and metrics

**Production Readiness:**
- **Security Best Practices** including non-root containers and resource limits
- **Performance Optimization** with efficient image builds and caching strategies
- **Disaster Recovery** procedures with automated backup and restore
- **Comprehensive Monitoring** for application health and performance metrics

### Real-World Applications
This lab demonstrates skills directly applicable to:

- **Enterprise Application Deployment** in cloud environments (AWS, Azure, GCP)
- **DevOps Pipeline Integration** with CI/CD systems
- **Microservices Architecture** implementation and management
- **Container Orchestration** preparation for Kubernetes adoption
- **Production Operations** including monitoring, scaling, and incident response

### Advanced Learning Paths

**Kubernetes Migration:**
- Convert Docker Compose to Kubernetes manifests
- Implement Helm charts for application packaging
- Configure ingress controllers and service meshes

**CI/CD Integration:**
- Automate builds with Jenkins/GitLab CI/GitHub Actions
- Implement blue-green deployment strategies
- Add automated testing and security scanning

**Cloud-Native Patterns:**
- Implement distributed tracing with Jaeger
- Add centralized logging with ELK stack
- Integrate with service discovery systems

**Security Enhancement:**
- Implement OAuth/JWT authentication
- Add SSL/TLS termination and certificate management
- Integrate with secret management systems (HashiCorp Vault)

### Certification Preparation
This comprehensive lab directly prepares you for:

- **Docker Certified Associate (DCA)** - Container lifecycle and orchestration
- **Kubernetes Administrator (CKA)** - Foundation for container orchestration
- **AWS/Azure/GCP Cloud Certifications** - Cloud-native application deployment

The hands-on experience with real-world scenarios, production patterns, and troubleshooting procedures provides the practical knowledge essential for professional container operations and modern application deployment strategies.

Your mastery of these containerization concepts positions you at the forefront of modern software deployment practices, ready to tackle enterprise-scale challenges in cloud-native environments.
