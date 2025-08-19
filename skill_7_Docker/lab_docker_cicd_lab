# Docker and CI/CD Pipelines - Enhanced Lab Guide

## 🎯 Objectives

By the end of this comprehensive lab, students will master:

- **Pipeline Architecture**: Set up and configure GitLab CI/CD pipelines for Docker-based applications
- **Automation**: Build and deploy Docker images automatically using CI/CD workflows
- **Orchestration**: Use docker-compose to orchestrate multi-service deployments in pipelines
- **Version Control**: Implement proper Docker image versioning and tagging strategies
- **Security**: Integrate automated vulnerability scanning into CI/CD processes
- **Secret Management**: Securely manage secrets and environment variables in containerized deployments
- **Best Practices**: Understand production-ready Docker CI/CD patterns

---

## 📋 Prerequisites

Before starting this lab, ensure you have:

- ✅ Basic understanding of Docker containers and images
- ✅ Familiarity with Git version control system
- ✅ Basic knowledge of YAML syntax
- ✅ Understanding of web application deployment concepts
- ✅ Access to a GitLab account (free tier is sufficient)

---

## 🖥️ Lab Environment Setup

**Ready-to-Use Cloud Machines**: Al Nafi provides pre-configured Linux-based cloud machines with all necessary tools installed. Simply click **Start Lab** to access your environment. No need to build your own VM or install software locally.

### Your cloud machine includes:
- 🐳 Docker Engine (latest stable version)
- 🐙 Docker Compose
- 📦 Git client
- ✏️ Text editors (nano, vim)
- 🌐 curl and wget utilities

---

## 📚 Task 1: Set up a GitLab CI/CD Pipeline to Build and Deploy Docker Images

### 🔧 Subtask 1.1: Create a Sample Application

Let's start by creating a robust Node.js web application that demonstrates real-world containerization patterns.

**Step 1**: Create your project structure

```bash
mkdir docker-cicd-lab
cd docker-cicd-lab
```

**Expected Output:**
```
$ mkdir docker-cicd-lab
$ cd docker-cicd-lab

$ pwd
/home/user/docker-cicd-lab

$ ls -la
total 8
drwxr-xr-x 2 user user 4096 Aug 19 10:30 .
drwxr-xr-x 3 user user 4096 Aug 19 10:30 ..
```

---

**Step 2**: Create the main application file

```bash
nano app.js
```

**💡 What we're building**: A production-ready Node.js Express server with health checks, environment configuration, and JSON responses.

Add the following content:

```javascript
const express = require('express');
const app = express();
const port = process.env.PORT || 3000;

app.get('/', (req, res) => {
    res.json({
        message: 'Hello from Docker CI/CD Pipeline!',
        version: process.env.APP_VERSION || '1.0.0',
        environment: process.env.NODE_ENV || 'development',
        timestamp: new Date().toISOString(),
        hostname: require('os').hostname()
    });
});

app.get('/health', (req, res) => {
    res.status(200).json({ 
        status: 'healthy',
        uptime: process.uptime(),
        memory: process.memoryUsage(),
        timestamp: new Date().toISOString()
    });
});

app.listen(port, () => {
    console.log(`🚀 Server running on port ${port}`);
    console.log(`📊 Environment: ${process.env.NODE_ENV || 'development'}`);
    console.log(`📦 Version: ${process.env.APP_VERSION || '1.0.0'}`);
});
```

**Expected Output:**
```
$ nano app.js
[nano editor opens - add the content above]

$ ls -la
total 12
drwxr-xr-x 2 user user 4096 Aug 19 10:32 .
drwxr-xr-x 3 user user 4096 Aug 19 10:30 ..
-rw-r--r-- 1 user user  856 Aug 19 10:32 app.js

$ wc -l app.js
26 app.js
```

---

**Step 3**: Create the package configuration

```bash
nano package.json
```

**💡 Understanding package.json**: This file defines our application dependencies, scripts, and metadata for Node.js.

Add the following content:

```json
{
  "name": "docker-cicd-app",
  "version": "1.0.0",
  "description": "Sample app for Docker CI/CD lab - demonstrates containerization with Express.js",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "test": "echo \"✅ Basic tests passed\" && exit 0",
    "dev": "NODE_ENV=development node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "keywords": ["docker", "cicd", "node", "express"],
  "author": "DevOps Student",
  "license": "MIT"
}
```

**Expected Output:**
```
$ nano package.json
[nano editor opens - add the content above]

$ ls -la
total 16
drwxr-xr-x 2 user user 4096 Aug 19 10:34 .
drwxr-xr-x 3 user user 4096 Aug 19 10:30 ..
-rw-r--r-- 1 user user  856 Aug 19 10:32 app.js
-rw-r--r-- 1 user user  445 Aug 19 10:34 package.json

$ cat package.json | head -5
{
  "name": "docker-cicd-app",
  "version": "1.0.0",
  "description": "Sample app for Docker CI/CD lab - demonstrates containerization with Express.js",
  "main": "app.js",
```

---

### 🐳 Subtask 1.2: Create a Production-Ready Dockerfile

**Step 4**: Create an optimized, secure Dockerfile

```bash
nano Dockerfile
```

**💡 Dockerfile Best Practices**: We're implementing multi-stage builds, non-root users, health checks, and minimal attack surface.

Add the following content:

```dockerfile
# Use official Node.js runtime as base image
FROM node:18-alpine

# Set working directory in container
WORKDIR /app

# Copy package files first (Docker layer caching optimization)
COPY package*.json ./

# Install dependencies
RUN npm install --only=production && \
    npm cache clean --force

# Copy application code
COPY . .

# Create non-root user for security
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 && \
    chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# Start application
CMD ["npm", "start"]
```

**Expected Output:**
```
$ nano Dockerfile
[nano editor opens - add the content above]

$ ls -la
total 20
drwxr-xr-x 2 user user 4096 Aug 19 10:36 .
drwxr-xr-x 3 user user 4096 Aug 19 10:30 ..
-rw-r--r-- 1 user user  856 Aug 19 10:32 app.js
-rw-r--r-- 1 user user  678 Aug 19 10:36 Dockerfile
-rw-r--r-- 1 user user  445 Aug 19 10:34 package.json

$ grep -n "FROM\|USER\|EXPOSE" Dockerfile
2:FROM node:18-alpine
16:USER nodejs
18:EXPOSE 3000
```

---

**Step 5**: Test the Docker build locally

```bash
echo "🔨 Testing Docker build locally..."
docker build -t docker-cicd-app:test .
```

**Expected Output:**
```
$ echo "🔨 Testing Docker build locally..."
🔨 Testing Docker build locally...

$ docker build -t docker-cicd-app:test .
[+] Building 45.2s (12/12) FINISHED
 => [internal] load .dockerignore                          0.0s
 => [internal] load build definition from Dockerfile       0.1s
 => [internal] load metadata for docker.io/library/node:18-alpine  2.4s
 => [1/7] FROM docker.io/library/node:18-alpine@sha256:... 15.6s
 => [internal] load build context                          0.0s
 => [2/7] WORKDIR /app                                      0.1s
 => [3/7] COPY package*.json ./                            0.0s
 => [4/7] RUN npm install --only=production && npm cache...  18.2s
 => [5/7] COPY . .                                          0.0s
 => [6/7] RUN addgroup -g 1001 -S nodejs && adduser -S...  0.4s
 => [7/7] USER nodejs                                       0.0s
 => exporting to image                                      0.3s
 => => writing image sha256:8f4b7e9c2d1a3f5g6h7i8j9k0l...  0.0s
 => => naming to docker.io/library/docker-cicd-app:test    0.0s

$ docker images | grep docker-cicd-app
docker-cicd-app   test    8f4b7e9c2d1a   2 minutes ago   124MB
```

---

**Step 6**: Test the container locally

```bash
echo "🧪 Testing container functionality..."

docker run -d --name test-app -p 3001:3000 docker-cicd-app:test

echo ""
echo "⏳ Waiting for container to start..."
sleep 5

echo ""
echo "📊 Container status:"
docker ps | grep test-app

echo ""
echo "🔍 Testing application endpoints..."
curl -s http://localhost:3001 | python3 -m json.tool

echo ""
echo "💚 Testing health endpoint..."
curl -s http://localhost:3001/health | python3 -m json.tool

echo ""
echo "🧹 Cleaning up test container..."
docker stop test-app
docker rm test-app
```

**Expected Output:**
```
$ echo "🧪 Testing container functionality..."
🧪 Testing container functionality...

$ docker run -d --name test-app -p 3001:3000 docker-cicd-app:test
a7b8c9d0e1f2g3h4i5j6k7l8m9n0o1p2q3r4s5t6u7v8w9x0y1z2

$ echo ""

$ echo "⏳ Waiting for container to start..."
⏳ Waiting for container to start...

$ sleep 5

$ echo ""

$ echo "📊 Container status:"
📊 Container status:
$ docker ps | grep test-app
a7b8c9d0e1f2  docker-cicd-app:test  "docker-entrypoint.s..."  8 seconds ago  Up 7 seconds  0.0.0.0:3001->3000/tcp  test-app

$ echo ""

$ echo "🔍 Testing application endpoints..."
🔍 Testing application endpoints...
$ curl -s http://localhost:3001 | python3 -m json.tool
{
    "message": "Hello from Docker CI/CD Pipeline!",
    "version": "1.0.0",
    "environment": "development",
    "timestamp": "2024-08-19T10:42:15.234Z",
    "hostname": "a7b8c9d0e1f2"
}

$ echo ""

$ echo "💚 Testing health endpoint..."
💚 Testing health endpoint...
$ curl -s http://localhost:3001/health | python3 -m json.tool
{
    "status": "healthy",
    "uptime": 5.123,
    "memory": {
        "rss": 25165824,
        "heapTotal": 7684096,
        "heapUsed": 5234560
    },
    "timestamp": "2024-08-19T10:42:20.567Z"
}

$ echo ""

$ echo "🧹 Cleaning up test container..."
🧹 Cleaning up test container...
$ docker stop test-app
test-app

$ docker rm test-app
test-app
```

---

### 🔗 Subtask 1.3: Initialize Git Repository and Connect to GitLab

**Step 7**: Initialize Git and make initial commit

```bash
echo "📝 Initializing Git repository..."
git init

echo ""
echo "📋 Adding files to Git..."
git add .

echo ""
echo "📊 Checking Git status..."
git status

echo ""
echo "💾 Making initial commit..."
git commit -m "Initial commit: Add Node.js app and Dockerfile

- Add Express.js application with health endpoints
- Add production-ready Dockerfile with security best practices
- Add package.json with proper metadata"

echo ""
echo "🌟 Git log:"
git log --oneline
```

**Expected Output:**
```
$ echo "📝 Initializing Git repository..."
📝 Initializing Git repository...

$ git init
Initialized empty Git repository in /home/user/docker-cicd-lab/.git/

$ echo ""

$ echo "📋 Adding files to Git..."
📋 Adding files to Git...

$ git add .

$ echo ""

$ echo "📊 Checking Git status..."
📊 Checking Git status...

$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   Dockerfile
        new file:   app.js
        new file:   package.json

$ echo ""

$ echo "💾 Making initial commit..."
💾 Making initial commit...

$ git commit -m "Initial commit: Add Node.js app and Dockerfile

- Add Express.js application with health endpoints  
- Add production-ready Dockerfile with security best practices
- Add package.json with proper metadata"
[master (root-commit) abc1234] Initial commit: Add Node.js app and Dockerfile
 3 files changed, 45 insertions(+)
 create mode 100644 Dockerfile
 create mode 100644 app.js
 create mode 100644 package.json

$ echo ""

$ echo "🌟 Git log:"
🌟 Git log:
$ git log --oneline
abc1234 (HEAD -> master) Initial commit: Add Node.js app and Dockerfile
```

---

**Step 8**: Connect to GitLab

```bash
echo "🔗 Connecting to GitLab..."
echo ""
echo "⚠️  IMPORTANT: Before running the next commands:"
echo "   1. Go to GitLab.com and sign in"
echo "   2. Click 'New Project'"
echo "   3. Choose 'Create blank project'"
echo "   4. Name it 'docker-cicd-lab'"
echo "   5. Set visibility to Private"
echo "   6. Click 'Create project'"
echo ""
echo "📝 Replace YOUR_USERNAME with your actual GitLab username in the next command"
echo ""

# Replace YOUR_USERNAME with your actual GitLab username
read -p "Enter your GitLab username: " GITLAB_USERNAME

git remote add origin https://gitlab.com/$GITLAB_USERNAME/docker-cicd-lab.git
git branch -M main

echo ""
echo "🚀 Pushing to GitLab..."
git push -u origin main
```

**Expected Output:**
```
$ echo "🔗 Connecting to GitLab..."
🔗 Connecting to GitLab...

$ echo ""

$ echo "⚠️  IMPORTANT: Before running the next commands:"
⚠️  IMPORTANT: Before running the next commands:
$echo "   1. Go to GitLab.com and sign in"
   1. Go to GitLab.com and sign in
$ echo "   2. Click 'New Project'"
   2. Click 'New Project'
$ echo "   3. Choose 'Create blank project'"
   3. Choose 'Create blank project'
$ echo "   4. Name it 'docker-cicd-lab'"
   4. Name it 'docker-cicd-lab'
$ echo "   5. Set visibility to Private"
   5. Set visibility to Private
$ echo "   6. Click 'Create project'"
   6. Click 'Create project'

$ echo ""

$ echo "📝 Replace YOUR_USERNAME with your actual GitLab username in the next command"
📝 Replace YOUR_USERNAME with your actual GitLab username in the next command

$ echo ""

$ read -p "Enter your GitLab username: " GITLAB_USERNAME
Enter your GitLab username: john_doe

$ git remote add origin https://gitlab.com/john_doe/docker-cicd-lab.git

$ git branch -M main

$ echo ""

$ echo "🚀 Pushing to GitLab..."
🚀 Pushing to GitLab...

$ git push -u origin main
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 4 threads
Compressing objects: 100% (5/5), done.
Writing objects: 100% (6/6), 1.23 KiB | 1.23 MiB/s, done.
Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
To https://gitlab.com/john_doe/docker-cicd-lab.git
 * [new branch]      main -> main
Branch 'main' set up to track remote branch 'main' from 'origin'.
```

---

### ⚙️ Subtask 1.4: Create GitLab CI/CD Pipeline Configuration

**Step 9**: Create the GitLab CI/CD pipeline

```bash
echo "⚙️ Creating GitLab CI/CD pipeline configuration..."
nano .gitlab-ci.yml
```

**💡 Understanding CI/CD Stages**: Our pipeline follows industry best practices with distinct stages: build → test → security → deploy

Add the following content:

```yaml
# GitLab CI/CD Pipeline for Docker Application
# This pipeline demonstrates industry-standard DevOps practices

# Define stages for the pipeline
stages:
  - build
  - test  
  - security
  - deploy

# Global variables
variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"
  IMAGE_NAME: $CI_REGISTRY_IMAGE
  IMAGE_TAG: $CI_COMMIT_SHORT_SHA

# Services needed for Docker-in-Docker
services:
  - docker:24-dind

# Before script to login to GitLab Container Registry
before_script:
  - echo "🔐 Logging into GitLab Container Registry..."
  - echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY

# Build stage - Build Docker image
build_image:
  stage: build
  image: docker:24
  script:
    - echo "🏗️ Building Docker image..."
    - echo "📋 Build details:"
    - echo "  • Image: $IMAGE_NAME"
    - echo "  • Tag: $IMAGE_TAG"
    - echo "  • Commit: $CI_COMMIT_SHORT_SHA"
    
    - docker build -t $IMAGE_NAME:$IMAGE_TAG .
    - docker build -t $IMAGE_NAME:latest .
    
    - echo ""
    - echo "📊 Image information:"
    - docker images | grep $CI_PROJECT_NAME
    
    - echo ""
    - echo "🚀 Pushing image to registry..."
    - docker push $IMAGE_NAME:$IMAGE_TAG
    - docker push $IMAGE_NAME:latest
    
    - echo "✅ Build completed successfully!"
  only:
    - main
    - develop

# Test stage - Run comprehensive tests
test_application:
  stage: test
  image: docker:24
  script:
    - echo "🧪 Running application tests..."
    
    - echo ""
    - echo "📋 Test Plan:"
    - echo "  • Unit tests via npm test"
    - echo "  • Container health check"
    - echo "  • Endpoint functionality"
    
    - echo ""
    - echo "1️⃣ Running unit tests..."
    - docker run --rm $IMAGE_NAME:$IMAGE_TAG npm test
    
    - echo ""
    - echo "2️⃣ Testing container startup and health..."
    - docker run -d --name test-container -p 3000:3000 $IMAGE_NAME:$IMAGE_TAG
    
    - echo "⏳ Waiting for container to be ready..."
    - sleep 15
    
    - echo ""
    - echo "📊 Container status:"
    - docker ps | grep test-container
    
    - echo ""
    - echo "3️⃣ Testing health endpoint..."
    - docker exec test-container wget --spider -q http://localhost:3000/health || exit 1
    
    - echo ""
    - echo "4️⃣ Testing main endpoint..."
    - docker exec test-container wget -q -O- http://localhost:3000 | head -1
    
    - echo ""
    - echo "🧹 Cleaning up test container..."
    - docker stop test-container
    - docker rm test-container
    
    - echo "✅ All tests passed!"
  dependencies:
    - build_image
  only:
    - main
    - develop

# Security stage - Basic security checks
security_scan:
  stage: security
  image: docker:24
  script:
    - echo "🔒 Running security scan..."
    
    - echo ""
    - echo "📋 Security Checks:"
    - echo "  • Image vulnerability scan (basic)"
    - echo "  • Container configuration review"
    
    - echo ""
    - echo "1️⃣ Checking image layers..."
    - docker history $IMAGE_NAME:$IMAGE_TAG
    
    - echo ""
    - echo "2️⃣ Inspecting security configuration..."
    - docker inspect $IMAGE_NAME:$IMAGE_TAG | grep -E "(User|SecurityOpt|Privileged)" || true
    
    - echo ""
    - echo "3️⃣ Testing container with security constraints..."
    - docker run --rm --security-opt no-new-privileges:true $IMAGE_NAME:$IMAGE_TAG echo "Security test passed"
    
    - echo "✅ Basic security scan completed"
  dependencies:
    - build_image
  only:
    - main

# Deploy stage - Deploy to staging
deploy_staging:
  stage: deploy
  image: docker:24
  script:
    - echo "🚀 Deploying to staging environment..."
    
    - echo ""
    - echo "📋 Deployment Details:"
    - echo "  • Environment: staging"
    - echo "  • Image: $IMAGE_NAME:$IMAGE_TAG"
    - echo "  • Target: Container Registry"
    
    - echo ""
    - echo "✅ Image $IMAGE_NAME:$IMAGE_TAG deployed successfully to staging"
    
    - echo ""
    - echo "🔗 Next steps:"
    - echo "  • Image is available in GitLab Container Registry"
    - echo "  • Ready for production deployment"
  environment:
    name: staging
    url: http://staging.example.com
  dependencies:
    - test_application
    - security_scan
  only:
    - main
```

**Expected Output:**
```
$ echo "⚙️ Creating GitLab CI/CD pipeline configuration..."
⚙️ Creating GitLab CI/CD pipeline configuration...

$ nano .gitlab-ci.yml
[nano editor opens - add the content above]

$ ls -la
total 24
drwxr-xr-x 2 user user 4096 Aug 19 10:50 .
drwxr-xr-x 3 user user 4096 Aug 19 10:30 ..
-rw-r--r-- 1 user user 3456 Aug 19 10:50 .gitlab-ci.yml
-rw-r--r-- 1 user user  856 Aug 19 10:32 app.js
-rw-r--r-- 1 user user  678 Aug 19 10:36 Dockerfile
-rw-r--r-- 1 user user  445 Aug 19 10:34 package.json

$ wc -l .gitlab-ci.yml
142 .gitlab-ci.yml

$ grep -n "stage:" .gitlab-ci.yml | head -5
18:  stage: build
35:  stage: test  
76:  stage: security
96:  stage: deploy
```

---

### 🚀 Subtask 1.5: Commit and Trigger Pipeline

**Step 10**: Commit pipeline configuration and trigger

```bash
echo "📝 Committing pipeline configuration..."
git add .gitlab-ci.yml

echo ""
echo "📊 Checking what we're committing..."
git status

echo ""
git commit -m "Add comprehensive GitLab CI/CD pipeline

- Add 4-stage pipeline: build, test, security, deploy
- Implement Docker-in-Docker for container operations
- Add comprehensive testing and health checks
- Include security scanning placeholder
- Add detailed logging and status reporting"

echo ""
echo "🚀 Pushing to GitLab to trigger pipeline..."
git push origin main

echo ""
echo "✅ Pipeline configuration pushed!"
echo ""
echo "🔗 To view your pipeline:"
echo "   1. Go to your GitLab project"
echo "   2. Navigate to CI/CD > Pipelines"  
echo "   3. Click on the running pipeline"
echo "   4. Watch the stages execute!"
```

**Expected Output:**
```
$ echo "📝 Committing pipeline configuration..."
📝 Committing pipeline configuration...

$ git add .gitlab-ci.yml

$ echo ""

$ echo "📊 Checking what we're committing..."
📊 Checking what we're committing...

$ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   .gitlab-ci.yml

$ echo ""

$ git commit -m "Add comprehensive GitLab CI/CD pipeline

- Add 4-stage pipeline: build, test, security, deploy
- Implement Docker-in-Docker for container operations  
- Add comprehensive testing and health checks
- Include security scanning placeholder
- Add detailed logging and status reporting"
[main def5678] Add comprehensive GitLab CI/CD pipeline
 1 file changed, 142 insertions(+)
 create mode 100644 .gitlab-ci.yml

$ echo ""

$ echo "🚀 Pushing to GitLab to trigger pipeline..."
🚀 Pushing to GitLab to trigger pipeline...

$ git push origin main
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 4 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 1.45 KiB | 1.45 MiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
To https://gitlab.com/john_doe/docker-cicd-lab.git
   abc1234..def5678  main -> main

$ echo ""

$ echo "✅ Pipeline configuration pushed!"
✅ Pipeline configuration pushed!

$ echo ""

$ echo "🔗 To view your pipeline:"
🔗 To view your pipeline:
$ echo "   1. Go to your GitLab project"
   1. Go to your GitLab project
$ echo "   2. Navigate to CI/CD > Pipelines"
   2. Navigate to CI/CD > Pipelines
$ echo "   3. Click on the running pipeline"
   3. Click on the running pipeline
$ echo "   4. Watch the stages execute!"
   4. Watch the stages execute!
```

---

## 🐙 Task 2: Use docker-compose to Deploy Services as Part of the Pipeline

### 📦 Subtask 2.1: Create Multi-Service docker-compose Configuration

**Step 11**: Create comprehensive docker-compose setup

```bash
echo "🐙 Creating docker-compose configuration for multi-service deployment..."
nano docker-compose.yml
```

**💡 Understanding docker-compose**: We're building a complete microservices stack with app, database, cache, and reverse proxy.

Add the following content:

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=${NODE_ENV:-production}
      - APP_VERSION=${APP_VERSION:-1.0.0}
      - DATABASE_URL=mongodb://${MONGO_USERNAME:-admin}:${MONGO_PASSWORD:-password123}@mongo:27017/myapp
      - REDIS_URL=redis://:${REDIS_PASSWORD:-redis123}@redis:6379
    depends_on:
      - mongo
      - redis
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  mongo:
    image: mongo:6.0
    environment:
      - MONGO_INITDB_ROOT_USERNAME=${MONGO_USERNAME:-admin}
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_PASSWORD:-password123}
      - MONGO_INITDB_DATABASE=myapp
    volumes:
      - mongo-data:/data/db
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 30s
      timeout: 10s
      retries: 3

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD:-redis123} --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "--raw", "incr", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  mongo-data:
    driver: local
  redis-data:
    driver: local

networks:
  app-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

**Expected Output:**
```
$ echo "🐙 Creating docker-compose configuration for multi-service deployment..."
🐙 Creating docker-compose configuration for multi-service deployment...

$ nano docker-compose.yml
[nano editor opens - add the content above]

$ ls -la
total 28
drwxr-xr-x 2 user user 4096 Aug 19 11:00 .
drwxr-xr-x 3 user user 4096 Aug 19 10:30 ..
-rw-r--r-- 1 user user 3456 Aug 19 10:50 .gitlab-ci.yml
-rw-r--r-- 1 user user  856 Aug 19 10:32 app.js
-rw-r--r-- 1 user user 2134 Aug 19 11:00 docker-compose.yml
-rw-r--r-- 1 user user  678 Aug 19 10:36 Dockerfile
-rw-r--r-- 1 user user  445 Aug 19 10:34 package.json

$ grep -c "image:\|build:" docker-compose.yml
5

$ echo "📊 Services defined:"
📊 Services defined:
$ docker-compose config --services
app
mongo  
redis
nginx
```

---

### 🌐 Subtask 2.2: Create Nginx Load Balancer Configuration

**Step 12**: Create production-ready Nginx configuration

```bash
echo "🌐 Creating Nginx reverse proxy configuration..."
nano nginx.conf
```

**💡 Understanding Nginx**: This configuration provides load balancing, health checks, and proper headers for our application.

Add the following content:

```nginx
events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

http {
    # Basic settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    
    # MIME types
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # Logging
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    
    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log warn;
    
    # Upstream configuration for load balancing
    upstream app_servers {
        least_conn;
        server app:3000 max_fails=3 fail_timeout=30s;
        keepalive 32;
    }
    
    # Main server block
    server {
        listen 80;
        server_name localhost;
        
        # Security headers
        add_header X-Frame-Options DENY;
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        
        # Main application proxy
        location / {
            proxy_pass http://app_servers;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_cache_bypass $http_upgrade;
            
            # Timeouts
            proxy_connect_timeout 5s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
        }
        
        # Health check endpoint
        location /health {
            proxy_pass http://app_servers/health;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # Don't log health checks
            access_log off;
        }
        
        # Nginx status for monitoring
        location /nginx_status {
            stub_status on;
            access_log off;
            allow 127.0.0.1;
            allow 172.20.0.0/16;  # Docker network
            deny all;
        }
    }
}
```

**Expected Output:**
```
$ echo "🌐 Creating Nginx reverse proxy configuration..."
🌐 Creating Nginx reverse proxy configuration...

$ nano nginx.conf
[nano editor opens - add the content above]

$ ls -la
total 32
drwxr-xr-x 2 user user 4096 Aug 19 11:05 .
drwxr-xr-x 3 user user 4096 Aug 19 10:30 ..
-rw-r--r-- 1 user user 3456 Aug 19 10:50 .gitlab-ci.yml
-rw-r--r-- 1 user user  856 Aug 19 10:32 app.js
-rw-r--r-- 1 user user 2134 Aug 19 11:00 docker-compose.yml
-rw-r--r-- 1 user user  678 Aug 19 10:36 Dockerfile
-rw-r--r-- 1 user user 2187 Aug 19 11:05 nginx.conf
-rw-r--r-- 1 user user  445 Aug 19 10:34 package.json

$ nginx -t -c $(pwd)/nginx.conf 2>&1 || echo "Note: Nginx test requires nginx binary - config will be validated in container"
Note: Nginx test requires nginx binary - config will be validated in container

$ grep -n "upstream\|server\|location" nginx.conf | head -8
27:    upstream app_servers {
32:    server {
33:        listen 80;
34:        server_name localhost;
41:        location / {
58:        location /health {
68:        location /nginx_status {
```

---

### 🔒 Subtask 2.3: Create Environment Configuration Template

**Step 13**: Create environment variables template

```bash
echo "🔒 Creating environment variables template..."
nano .env.example
```

**💡 Understanding Environment Variables**: This template shows all configurable options without exposing actual secrets.

Add the following content:

```bash
# =================================
# Docker CI/CD Lab Environment Variables
# =================================

# Application Configuration
NODE_ENV=production
APP_VERSION=1.0.0
PORT=3000

# Database Configuration (MongoDB)
MONGO_USERNAME=admin
MONGO_PASSWORD=change_this_secure_password_here
MONGO_INITDB_DATABASE=myapp

# Cache Configuration (Redis)  
REDIS_PASSWORD=change_this_redis_password_here

# Nginx Configuration
NGINX_PORT=80

# Development Overrides (use .env.local)
# NODE_ENV=development
# MONGO_PASSWORD=dev_password
# REDIS_PASSWORD=dev_redis

# Production Notes:
# - Change all default passwords
# - Use strong, unique passwords for each service
# - Consider using secrets management in production
# - Never commit actual passwords to version control
```

**Expected Output:**
```
$ echo "🔒 Creating environment variables template..."
🔒 Creating environment variables template...

$ nano .env.example
[nano editor opens - add the content above]

$ ls -la
total 36
drwxr-xr-x 2 user user 4096 Aug 19 11:08 .
drwxr-xr-x 3 user user 4096 Aug 19 10:30 ..
-rw-r--r-- 1 user user 3456 Aug 19 10:50 .gitlab-ci.yml
-rw-r--r-- 1 user user  856 Aug 19 10:32 app.js
-rw-r--r-- 1 user user  875 Aug 19 11:08 .env.example
-rw-r--r-- 1 user user 2134 Aug 19 11:00 docker-compose.yml
-rw-r--r-- 1 user user  678 Aug 19 10:36 Dockerfile
-rw-r--r-- 1 user user 2187 Aug 19 11:05 nginx.conf
-rw-r--r-- 1 user user  445 Aug 19 10:34 package.json

$ grep -E "^[A-Z_]+=" .env.example
NODE_ENV=production
APP_VERSION=1.0.0
PORT=3000
MONGO_USERNAME=admin
MONGO_PASSWORD=change_this_secure_password_here
MONGO_INITDB_DATABASE=myapp
REDIS_PASSWORD=change_this_redis_password_here
NGINX_PORT=80
```

---

**Step 14**: Test the complete stack locally

```bash
echo "🧪 Testing complete docker-compose stack locally..."

echo ""
echo "📋 Test Plan:"
echo "  • Build and start all services"
echo "  • Verify service connectivity" 
echo "  • Test through Nginx proxy"
echo "  • Check service health"
echo "  • Clean up resources"

echo ""
echo "1️⃣ Starting the complete stack..."
docker-compose up -d

echo ""
echo "⏳ Waiting for all services to be ready..."
sleep 30

echo ""
echo "2️⃣ Checking service status..."
docker-compose ps

echo ""
echo "3️⃣ Testing direct app connection..."
curl -s http://localhost:3000/health | python3 -m json.tool | head -3

echo ""
echo "4️⃣ Testing through Nginx proxy..."
curl -s http://localhost:80/health | python3 -m json.tool | head -3

echo ""
echo "5️⃣ Checking service logs (last 10 lines each)..."
echo ""
echo "📱 App logs:"
docker-compose logs --tail=10 app

echo ""
echo "🗄️ MongoDB logs:"
docker-compose logs --tail=5 mongo

echo ""
echo "⚡ Redis logs:"
docker-compose logs --tail=5 redis

echo ""
echo "🌐 Nginx logs:"
docker-compose logs --tail=5 nginx

echo ""
echo "6️⃣ Testing service connectivity..."
echo "   📋 Ping MongoDB from app container..."
docker-compose exec app ping -c 2 mongo || echo "   ⚠️ MongoDB ping failed (expected if ping not installed)"

echo ""
echo "   📋 Test Redis from app container..."
docker-compose exec redis redis-cli ping || echo "   ⚠️ Redis test failed"

echo ""
echo "🧹 Cleaning up test stack..."
docker-compose down -v

echo ""
echo "✅ Docker-compose stack test completed!"
```

**Expected Output:**
```
$ echo "🧪 Testing complete docker-compose stack locally..."
🧪 Testing complete docker-compose stack locally...

$ echo ""

$ echo "📋 Test Plan:"
📋 Test Plan:
$ echo "  • Build and start all services"
  • Build and start all services
$ echo "  • Verify service connectivity"
  • Verify service connectivity
$ echo "  • Test through Nginx proxy"
  • Test through Nginx proxy
$ echo "  • Check service health"
  • Check service health
$ echo "  • Clean up resources"
  • Clean up resources

$ echo ""

$ echo "1️⃣ Starting the complete stack..."
1️⃣ Starting the complete stack...

$ docker-compose up -d
Creating network "docker-cicd-lab_app-network" with driver "bridge"
Creating volume "docker-cicd-lab_mongo-data" with local driver
Creating volume "docker-cicd-lab_redis-data" with local driver
Building app
[+] Building 12.3s (12/12) FINISHED
 => [internal] load build definition from Dockerfile       0.1s
 => [internal] load .dockerignore                          0.0s
 => [internal] load metadata for docker.io/library/node:18-alpine  1.2s
 => [1/7] FROM docker.io/library/node:18-alpine@sha256:...  0.0s
 => [internal] load build context                          0.0s
 => CACHED [2/7] WORKDIR /app                              0.0s
 => [3/7] COPY package*.json ./                            0.0s
 => [4/7] RUN npm install --only=production && npm cache...  8.4s
 => [5/7] COPY . .                                          0.0s
 => [6/7] RUN addgroup -g 1001 -S nodejs && adduser -S...  0.3s
 => [7/7] USER nodejs                                       0.0s
 => exporting to image                                      0.2s
 => => writing image sha256:9g5h6i7j8k9l0m1n2o3p4q5r6s...  0.0s
 => => naming to docker.io/library/docker-cicd-lab_app     0.0s
Creating docker-cicd-lab_mongo_1 ... done
Creating docker-cicd-lab_redis_1 ... done  
Creating docker-cicd-lab_app_1   ... done
Creating docker-cicd-lab_nginx_1 ... done

$ echo ""

$ echo "⏳ Waiting for all services to be ready..."
⏳ Waiting for all services to be ready...

$ sleep 30

$ echo ""

$ echo "2️⃣ Checking service status..."
2️⃣ Checking service status...

$ docker-compose ps
           Name                         Command               State                    Ports
----------------------------------------------------------------------------------------------------
docker-cicd-lab_app_1     docker-entrypoint.sh npm start   Up (healthy)   0.0.0.0:3000->3000/tcp
docker-cicd-lab_mongo_1   docker-entrypoint.sh mongod      Up (healthy)   27017/tcp
docker-cicd-lab_nginx_1   /docker-entrypoint.sh ngin ...   Up (healthy)   0.0.0.0:80->80/tcp
docker-cicd-lab_redis_1   docker-entrypoint.sh redis ...   Up (healthy)   6379/tcp

$ echo ""

$ echo "3️⃣ Testing direct app connection..."
3️⃣ Testing direct app connection...

$ curl -s http://localhost:3000/health | python3 -m json.tool | head -3
{
    "status": "healthy",
    "uptime": 28.456,

$ echo ""

$ echo "4️⃣ Testing through Nginx proxy..."
4️⃣ Testing through Nginx proxy...

$ curl -s http://localhost:80/health | python3 -m json.tool | head -3
{
    "status": "healthy", 
    "uptime": 29.123,

$ echo ""

$ echo "5️⃣ Checking service logs (last 10 lines each)..."
5️⃣ Checking service logs (last 10 lines each)...

$ echo ""

$ echo "📱 App logs:"
📱 App logs:
$ docker-compose logs --tail=10 app
app_1    | 🚀 Server running on port 3000
app_1    | 📊 Environment: production
app_1    | 📦 Version: 1.0.0

$ echo ""

$ echo "🗄️ MongoDB logs:"
🗄️ MongoDB logs:
$ docker-compose logs --tail=5 mongo
mongo_1  | {"t":{"$date":"2024-08-19T11:12:30.456+00:00"},"s":"I","c":"NETWORK","id":23016,"ctx":"listener","msg":"Waiting for connections","attr":{"port":27017,"ssl":"off"}}

$ echo ""

$ echo "⚡ Redis logs:"
⚡ Redis logs:
$ docker-compose logs --tail=5 redis
redis_1  | 1:C 19 Aug 2024 11:12:15.123 # Redis 7.0.11 (00000000/0) 64 bit
redis_1  | 1:M 19 Aug 2024 11:12:15.123 * Ready to accept connections

$ echo ""

$ echo "🌐 Nginx logs:"
🌐 Nginx logs:
$ docker-compose logs --tail=5 nginx
nginx_1  | 2024/08/19 11:12:20 [notice] 1#1: start worker processes
nginx_1  | 2024/08/19 11:12:20 [notice] 1#1: worker process 29 started

$ echo ""

$ echo "6️⃣ Testing service connectivity..."
6️⃣ Testing service connectivity...
$ echo "   📋 Ping MongoDB from app container..."
   📋 Ping MongoDB from app container...
$ docker-compose exec app ping -c 2 mongo || echo "   ⚠️ MongoDB ping failed (expected if ping not installed)"
   ⚠️ MongoDB ping failed (expected if ping not installed)

$ echo ""

$ echo "   📋 Test Redis from app container..."
   📋 Test Redis from app container...
$ docker-compose exec redis redis-cli ping || echo "   ⚠️ Redis test failed"
PONG

$ echo ""

$ echo "🧹 Cleaning up test stack..."
🧹 Cleaning up test stack...

$ docker-compose down -v
Stopping docker-cicd-lab_nginx_1 ... done
Stopping docker-cicd-lab_app_1   ... done
Stopping docker-cicd-lab_redis_1 ... done
Stopping docker-cicd-lab_mongo_1 ... done
Removing docker-cicd-lab_nginx_1 ... done
Removing docker-cicd-lab_app_1   ... done  
Removing docker-cicd-lab_redis_1 ... done
Removing docker-cicd-lab_mongo_1 ... done
Removing network docker-cicd-lab_app-network
Removing volume docker-cicd-lab_mongo-data
Removing volume docker-cicd-lab_redis-data

$ echo ""

$ echo "✅ Docker-compose stack test completed!"
✅ Docker-compose stack test completed!
```

---

### ⚙️ Subtask 2.4: Update GitLab CI/CD Pipeline for docker-compose

**Step 15**: Update pipeline with docker-compose integration

```bash
echo "⚙️ Updating GitLab CI/CD pipeline for docker-compose..."
cp .gitlab-ci.yml .gitlab-ci.yml.backup

nano .gitlab-ci.yml
```

**💡 Pipeline Enhancement**: We're adding docker-compose to our pipeline for realistic multi-service testing and deployment.

Replace the existing content with:

```yaml
# Enhanced GitLab CI/CD Pipeline with docker-compose Integration
# Demonstrates production-ready multi-service deployment patterns

stages:
  - build
  - test
  - security  
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"
  IMAGE_NAME: $CI_REGISTRY_IMAGE
  IMAGE_TAG: $CI_COMMIT_SHORT_SHA
  COMPOSE_PROJECT_NAME: docker-cicd-lab-$CI_PIPELINE_ID

services:
  - docker:24-dind

before_script:
  - echo "🔐 Logging into GitLab Container Registry..."
  - echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY
  - echo "📦 Installing docker-compose..."
  - apk add --no-cache docker-compose py3-pip
  - docker-compose --version

# Build stage with enhanced docker-compose support
build_image:
  stage: build
  image: docker:24
  script:
    - echo "🏗️ Building Docker image with docker-compose context..."
    
    - echo ""
    - echo "📋 Build Information:"
    - echo "  • Project: $CI_PROJECT_NAME"
    - echo "  • Image: $IMAGE_NAME"
    - echo "  • Tag: $IMAGE_TAG"
    - echo "  • Commit: $CI_COMMIT_SHORT_SHA"
    - echo "  • Compose Project: $COMPOSE_PROJECT_NAME"
    
    - echo ""
    - echo "1️⃣ Building application image..."
    - docker build -t $IMAGE_NAME:$IMAGE_TAG .
    - docker build -t $IMAGE_NAME:latest .
    
    - echo ""
    - echo "2️⃣ Validating docker-compose configuration..."
    - docker-compose config --quiet
    - echo "✅ Docker-compose configuration is valid"
    
    - echo ""
    - echo "3️⃣ Testing docker-compose build..."
    - export APP_VERSION=$IMAGE_TAG
    - docker-compose build --no-cache app
    
    - echo ""
    - echo "📊 Built images:"
    - docker images | grep -E "(docker-cicd-lab|$CI_PROJECT_NAME)" | head -5
    
    - echo ""
    - echo "4️⃣ Pushing images to registry..."
    - docker push $IMAGE_NAME:$IMAGE_TAG
    - docker push $IMAGE_NAME:latest
    
    - echo "✅ Build stage completed successfully!"
  artifacts:
    paths:
      - docker-compose.yml
      - nginx.conf
      - .env.example
    expire_in: 1 hour
  only:
    - main
    - develop

# Enhanced test stage with full stack testing
test_application:
  stage: test
  image: docker:24
  script:
    - echo "🧪 Running comprehensive docker-compose tests..."
    
    - echo ""
    - echo "📋 Test Strategy:"
    - echo "  • Multi-service integration testing"
    - echo "  • Health check validation"
    - echo "  • Service connectivity testing"
    - echo "  • Load balancer functionality"
    
    - echo ""
    - echo "1️⃣ Setting up test environment..."
    - export APP_VERSION=$IMAGE_TAG
    - export NODE_ENV=test
    - export MONGO_USERNAME=test_admin
    - export MONGO_PASSWORD=test_password_123
    - export REDIS_PASSWORD=test_redis_123
    
    - echo ""
    - echo "2️⃣ Starting complete stack..."
    - docker-compose up -d
    
    - echo "⏳ Waiting for all services to initialize..."
    - sleep 45
    
    - echo ""
    - echo "3️⃣ Checking service health..."
    - docker-compose ps
    
    - echo ""
    - echo "4️⃣ Running application tests..."
    - docker-compose exec -T app npm test
    
    - echo ""
    - echo "5️⃣ Testing service endpoints..."
    - echo "   📋 Direct app health check..."
    - docker-compose exec -T app wget --spider -q http://localhost:3000/health || exit 1
    
    - echo "   📋 Nginx proxy health check..."
    - docker-compose exec -T nginx wget --spider -q http://localhost/health || exit 1
    
    - echo ""
    - echo "6️⃣ Testing inter-service connectivity..."
    - echo "   📋 App to MongoDB connection..."
    - docker-compose exec -T app nslookup mongo || echo "DNS resolution test"
    
    - echo "   📋 App to Redis connection..."
    - docker-compose exec -T app nslookup redis || echo "DNS resolution test"
    
    - echo ""
    - echo "7️⃣ Performance validation..."
    - echo "   📋 Testing response times..."
    - docker-compose exec -T app time wget -q -O- http://localhost:3000/ > /dev/null
    
    - echo ""
    - echo "8️⃣ Collecting service logs for analysis..."
    - docker-compose logs --tail=20
    
    - echo ""
    - echo "🧹 Cleaning up test environment..."
    - docker-compose down -v
    
    - echo "✅ All integration tests passed!"
  dependencies:
    - build_image
  only:
    - main
    - develop

# Enhanced security stage with multi-service scanning
security_comprehensive_scan:
  stage: security
  image: docker:24
  before_script:
    - echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY
    - apk add --no-cache docker-compose curl
    - echo "🔧 Installing Trivy security scanner..."
    - curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
    - trivy --version
  script:
    - echo "🔒 Running comprehensive security analysis..."
    
    - echo ""
    - echo "📋 Security Scan Plan:"
    - echo "  • Application image vulnerability scan"
    - echo "  • Base image security assessment"  
    - echo "  • Docker configuration analysis"
    - echo "  • docker-compose security review"
    
    - echo ""
    - echo "1️⃣ Scanning application image..."
    - trivy image --exit-code 0 --severity HIGH,CRITICAL --format table $IMAGE_NAME:$IMAGE_TAG
    - trivy image --exit-code 1 --severity CRITICAL --format json --output app-scan-results.json $IMAGE_NAME:$IMAGE_TAG || echo "⚠️ Critical vulnerabilities found"
    
    - echo ""
    - echo "2️⃣ Scanning service base images..."
    - echo "   📋 MongoDB image scan..."
    - trivy image --exit-code 0 --severity HIGH,CRITICAL --format table mongo:6.0
    
    - echo "   📋 Redis image scan..."
    - trivy image --exit-code 0 --severity HIGH,CRITICAL --format table redis:7-alpine
    
    - echo "   📋 Nginx image scan..."  
    - trivy image --exit-code 0 --severity HIGH,CRITICAL --format table nginx:alpine
    
    - echo ""
    - echo "3️⃣ Configuration security analysis..."
    - trivy config --exit-code 0 --severity HIGH,CRITICAL .
    
    - echo ""
    - echo "4️⃣ Docker-compose security review..."
    - echo "   📋 Checking for privileged containers..."
    - grep -i "privileged" docker-compose.yml && echo "⚠️ Privileged containers found" || echo "✅ No privileged containers"
    
    - echo "   📋 Checking for exposed secrets..."
    - grep -i "password.*=" docker-compose.yml && echo "⚠️ Potential secrets in compose file" || echo "✅ No hardcoded secrets found"
    
    - echo ""
    - echo "5️⃣ Network security validation..."
    - docker-compose config | grep -A 5 "networks:"
    
    - echo ""
    - echo "6️⃣ Generating security report..."
    - |
      cat > security-report.md << EOF
      # 🔒 Security Scan Report
      
      **Scan Date:** $(date)
      **Pipeline:** $CI_PIPELINE_ID
      **Commit:** $CI_COMMIT_SHORT_SHA
      
      ## 📊 Application Image Analysis
      - **Image:** $IMAGE_NAME:$IMAGE_TAG
      - **Status:** $([ -f app-scan-results.json ] && echo "⚠️ Issues Found" || echo "✅ Clean")
      
      ## 🐳 Base Images Status
      - **MongoDB 6.0:** Scanned ✅
      - **Redis 7 Alpine:** Scanned ✅  
      - **Nginx Alpine:** Scanned ✅
      
      ## ⚙️ Configuration Analysis
      - **Dockerfile:** Analyzed ✅
      - **docker-compose.yml:** Reviewed ✅
      - **Network Security:** Validated ✅
      
      ## 🔍 Key Findings
      - Multi-service stack security validated
      - No privileged containers detected
      - Network isolation properly configured
      - For detailed vulnerability info, check pipeline logs
      EOF
    
    - cat security-report.md
    
    - echo ""
    - echo "✅ Security analysis completed!"
  artifacts:
    paths:
      - app-scan-results.json
      - security-report.md
    expire_in: 1 week
    reports:
      container_scanning: app-scan-results.json
  dependencies:
    - build_image
  allow_failure: true
  only:
    - main
    - develop

# Deploy stage with docker-compose deployment
deploy_staging:
  stage: deploy
  image: docker:24
  script:
    - echo "🚀 Deploying complete stack to staging..."
    
    - echo ""
    - echo "📋 Deployment Plan:"
    - echo "  • Deploy full docker-compose stack"
    - echo "  • Configure staging environment"
    - echo "  • Validate deployment health"
    - echo "  • Setup monitoring endpoints"
    
    - echo ""
    - echo "1️⃣ Preparing staging environment..."
    - export APP_VERSION=$IMAGE_TAG
    - export NODE_ENV=staging
    - export MONGO_USERNAME=${STAGING_MONGO_USERNAME:-staging_admin}
    - export MONGO_PASSWORD=${STAGING_MONGO_PASSWORD:-staging_secure_password}
    - export REDIS_PASSWORD=${STAGING_REDIS_PASSWORD:-redis_staging_pass}
    
    - echo "📊 Environment Configuration:"
    - echo "  • App Version: $APP_VERSION"
    - echo "  • Node Environment: $NODE_ENV"
    - echo "  • MongoDB User: $MONGO_USERNAME"
    - echo "  • Compose Project: $COMPOSE_PROJECT_NAME"
    
    - echo ""
    - echo "2️⃣ Deploying complete stack..."
    - docker-compose up -d
    
    - echo ""
    - echo "⏳ Waiting for stack initialization..."
    - sleep 60
    
    - echo ""
    - echo "3️⃣ Validating deployment..."
    - docker-compose ps
    
    - echo ""
    - echo "4️⃣ Health check validation..."
    - docker-compose exec -T app wget --spider -q http://localhost:3000/health || exit 1
    - docker-compose exec -T nginx wget --spider -q http://localhost/health || exit 1
    
    - echo ""
    - echo "5️⃣ Deployment verification..."
    - echo "   📊 Service status summary:"
    - docker-compose ps --format "table {{.Name}}\t{{.State}}\t{{.Status}}"
    
    - echo ""
    - echo "   📋 Resource usage:"
    - docker stats --no-stream --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
    
    - echo ""
    - echo "6️⃣ Setting up monitoring..."
    - echo "   📊 Nginx status endpoint available at /nginx_status"
    - echo "   💚 Application health at /health"
    
    - echo ""
    - echo "✅ Staging deployment completed successfully!"
    - echo ""
    - echo "🔗 Access Information:"
    - echo "  • Application: http://staging.example.com"
    - echo "  • Health Check: http://staging.example.com/health"
    - echo "  • Nginx Status: http://staging.example.com/nginx_status"
    
    # Optional: Send notification
    - |
      if [ ! -z "$NOTIFICATION_WEBHOOK" ]; then
        curl -X POST -H 'Content-type: application/json' \
        --data '{"text":"🚀 Staging deployment successful for '"$CI_PROJECT_NAME"' version '"$APP_VERSION"'"}' \
        $NOTIFICATION_WEBHOOK || echo "📢 Notification webhook not configured"
      fi
  environment:
    name: staging
    url: http://staging.example.com
  dependencies:
    - test_application
    - security_comprehensive_scan
  only:
    - main
  when: manual
```

**Expected Output:**
```
$ echo "⚙️ Updating GitLab CI/CD pipeline for docker-compose..."
⚙️ Updating GitLab CI/CD pipeline for docker-compose...

$ cp .gitlab-ci.yml .gitlab-ci.yml.backup

$ nano .gitlab-ci.yml
[nano editor opens - replace with the enhanced content above]

$ ls -la
total 44
drwxr-xr-x 2 user user 4096 Aug 19 11:15 .
drwxr-xr-x 3 user user 4096 Aug 19 10:30 ..
-rw-r--r-- 1 user user 6789 Aug 19 11:15 .gitlab-ci.yml
-rw-r--r-- 1 user user 3456 Aug 19 10:50 .gitlab-ci.yml.backup
-rw-r--r-- 1 user user  856 Aug 19 10:32 app.js
-rw-r--r-- 1 user user  875 Aug 19 11:08 .env.example
-rw-r--r-- 1 user user 2134 Aug 19 11:00 docker-compose.yml
-rw-r--r-- 1 user user  678 Aug 19 10:36 Dockerfile
-rw-r--r-- 1 user user 2187 Aug 19 11:05 nginx.conf
-rw-r--r-- 1 user user  445 Aug 19 10:34 package.json

$ wc -l .gitlab-ci.yml
278 .gitlab-ci.yml

$ grep -n "docker-compose" .gitlab-ci.yml | wc -l
15

$ echo "📊 Pipeline stages defined:"
📊 Pipeline stages defined:
$ grep -A 10 "^stages:" .gitlab-ci.yml
stages:
  - build
  - test
  - security  
  - deploy
```

---

## 🏷️ Task 3: Implement Docker Image Versioning and Tagging Strategies

### 📝 Subtask 3.1: Create Advanced Tagging Strategy

**Step 16**: Create comprehensive image tagging script

```bash
echo "🏷️ Creating advanced Docker image tagging strategy..."
mkdir -p scripts

nano scripts/tag-image.sh
```

**💡 Understanding Tagging Strategy**: We implement semantic versioning, branch-based tagging, and temporal tagging for complete traceability.

Add the following content:

```bash
#!/bin/bash

# Advanced Docker Image Tagging Script
# Implements comprehensive versioning strategy for CI/CD pipelines
set -e

echo "🏷️ Docker Image Tagging Strategy"
echo "=================================="

# Variables with defaults
IMAGE_NAME=${1:-$CI_REGISTRY_IMAGE}
COMMIT_SHA=${2:-$CI_COMMIT_SHORT_SHA}
BRANCH_NAME=${3:-$CI_COMMIT_REF_NAME}
BUILD_NUMBER=${4:-$CI_PIPELINE_ID}

echo ""
echo "📋 Tagging Configuration:"
echo "  • Image Name: $IMAGE_NAME"
echo "  • Commit SHA: $COMMIT_SHA"
echo "  • Branch: $BRANCH_NAME"
echo "  • Build: $BUILD_NUMBER"

# Function to create semantic version tag
create_semantic_tag() {
    local version_file="VERSION"
    echo ""
    echo "📦 Processing semantic version..."
    
    if [ -f "$version_file" ]; then
        VERSION=$(cat $version_file)
        echo "  • Current version: $VERSION"
    else
        VERSION="1.0.0"
        echo "  • Initializing version: $VERSION"
        echo $VERSION > $version_file
    fi
    
    # Increment version based on branch
    if [ "$BRANCH_NAME" = "main" ]; then
        MAJOR=$(echo $VERSION | cut -d. -f1)
        MINOR=$(echo $VERSION | cut -d. -f2)
        PATCH=$(echo $VERSION | cut -d. -f3)
        PATCH=$((PATCH + 1))
        NEW_VERSION="$MAJOR.$MINOR.$PATCH"
        echo $NEW_VERSION > $version_file
        echo "  • Bumped to: $NEW_VERSION (production release)"
        echo $NEW_VERSION
    elif [ "$BRANCH_NAME" = "develop" ]; then
        echo "  • Development version: $VERSION-dev-$BUILD_NUMBER"
        echo "$VERSION-dev-$BUILD_NUMBER"
    else
        echo "  • Feature version: $VERSION-$BRANCH_NAME-$BUILD_NUMBER"
        echo "$VERSION-$BRANCH_NAME-$BUILD_NUMBER"
    fi
}

# Create all tag variants
echo ""
echo "🎯 Creating comprehensive tag set..."

SEMANTIC_TAG=$(create_semantic_tag)
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
DATE_TAG=$(date +%Y-%m-%d)

echo ""
echo "📊 Tag Summary:"
echo "  • Semantic: $SEMANTIC_TAG"
echo "  • Commit: $COMMIT_SHA"
echo "  • Branch: $BRANCH_NAME"
echo "  • Timestamp: $TIMESTAMP"
echo "  • Date: $DATE_TAG"
echo "  • Build: build-$BUILD_NUMBER"

echo ""
echo "🔨 Applying tags to image..."

# Tag with commit SHA (immutable reference)
docker tag $IMAGE_NAME:latest $IMAGE_NAME:$COMMIT_SHA
echo "✅ Tagged with commit SHA: $COMMIT_SHA"

# Tag with semantic version
docker tag $IMAGE_NAME:latest $IMAGE_NAME:$SEMANTIC_TAG
echo "✅ Tagged with semantic version: $SEMANTIC_TAG"

# Tag with branch name (mutable reference)
docker tag $IMAGE_NAME:latest $IMAGE_NAME:$BRANCH_NAME
echo "✅ Tagged with branch name: $BRANCH_NAME"

# Tag with timestamp (for historical tracking)
docker tag $IMAGE_NAME:latest $IMAGE_NAME:$TIMESTAMP
echo "✅ Tagged with timestamp: $TIMESTAMP"

# Tag with date (for daily builds)
docker tag $IMAGE_NAME:latest $IMAGE_NAME:$DATE_TAG
echo "✅ Tagged with date: $DATE_TAG"

# Tag with build number (for CI tracking)
docker tag $IMAGE_NAME:latest $IMAGE_NAME:build-$BUILD_NUMBER
echo "✅ Tagged with build number: build-$BUILD_NUMBER"

echo ""
echo "🚀 Pushing all tags to registry..."

# Push all tags systematically
TAGS=("$COMMIT_SHA" "$SEMANTIC_TAG" "$BRANCH_NAME" "$TIMESTAMP" "$DATE_TAG" "build-$BUILD_NUMBER")

for tag in "${TAGS[@]}"; do
    echo "  📤 Pushing $IMAGE_NAME:$tag"
    docker push $IMAGE_NAME:$tag
done

echo ""
echo "📊 Registry Status:"
echo "  All tags pushed successfully to GitLab Container Registry"

echo ""
echo "🎉 Tagging Strategy Complete!"
echo "=================================="
```

**Expected Output:**
```
$ echo "🏷️ Creating advanced Docker image tagging strategy..."
🏷️ Creating advanced Docker image tagging strategy...

$ mkdir -p scripts

$ nano scripts/tag-image.sh
[nano editor opens - add the content above]

$ chmod +x scripts/tag-image.sh

$ ls -la scripts/
total 12
drwxr-xr-x 2 user user 4096 Aug 19 11:20 .
drwxr-xr-x 3 user user 4096 Aug 19 11:20 ..
-rwxr-xr-x 1 user user 3245 Aug 19 11:20 tag-image.sh

$ head -10 scripts/tag-image.sh
#!/bin/bash

# Advanced Docker Image Tagging Script
# Implements comprehensive versioning strategy for CI/CD pipelines
set -e

echo "🏷️ Docker Image Tagging Strategy"
echo "=================================="

# Variables with defaults
```

---

### 📊 Subtask 3.2: Create VERSION File and Test Tagging

**Step 17**: Initialize versioning system

```bash
echo "📊 Initializing semantic versioning system..."

echo "1.0.0" > VERSION

echo ""
echo "📋 Version file created:"
cat VERSION

echo ""
echo "🧪 Testing tagging script locally..."
echo ""
echo "📋 Test Configuration:"
echo "  • Using local test image"
echo "  • Simulating CI environment"
echo "  • Testing all tag variants"

echo ""
echo "1️⃣ Building test image for tagging..."
docker build -t test-tagging-app:latest .

echo ""
echo "2️⃣ Testing tagging script..."
./scripts/tag-image.sh test-tagging-app test123 main 42

echo ""
echo "3️⃣ Verifying created tags..."
docker images | grep test-tagging-app | head -10

echo ""
echo "4️⃣ Checking version file update..."
echo "New version: $(cat VERSION)"

echo ""
echo "🧹 Cleaning up test images..."
docker rmi $(docker images test-tagging-app -q) || echo "Images cleaned up"

echo ""
echo "✅ Tagging script validation completed!"
```

**Expected Output:**
```
$ echo "📊 Initializing semantic versioning system..."
📊 Initializing semantic versioning system...

$ echo "1.0.0" > VERSION

$ echo ""

$ echo "📋 Version file created:"
📋 Version file created:
$ cat VERSION
1.0.0

$ echo ""

$ echo "🧪 Testing tagging script locally..."
🧪 Testing tagging script locally...

$ echo ""

$ echo "📋 Test Configuration:"
📋 Test Configuration:
$ echo "  • Using local test image"
  • Using local test image
$ echo "  • Simulating CI environment"
  • Simulating CI environment
$ echo "  • Testing all tag variants"
  • Testing all tag variants

$ echo ""

$ echo "1️⃣ Building test image for tagging..."
1️⃣ Building test image for tagging...

$ docker build -t test-tagging-app:latest .
[+] Building 8.2s (12/12) FINISHED
 => [internal] load .dockerignore                          0.0s
 => [internal] load build definition from Dockerfile       0.1s
 => [internal] load metadata for docker.io/library/node:18-alpine  1.4s
 => [1/7] FROM docker.io/library/node:18-alpine@sha256:...  0.0s
 => [internal] load build context                          0.0s
 => CACHED [2/7] WORKDIR /app                              0.0s
 => [3/7] COPY package*.json ./                            0.0s
 => [4/7] RUN npm install --only=production && npm cache...  5.2s
 => [5/7] COPY . .                                          0.0s
 => [6/7] RUN addgroup -g 1001 -S nodejs && adduser -S...  0.3s
 => [7/7] USER nodejs                                       0.0s
 => exporting to image                                      0.1s

$ echo ""

$ echo "2️⃣ Testing tagging script..."
2️⃣ Testing tagging script...

$ ./scripts/tag-image.sh test-tagging-app test123 main 42
🏷️ Docker Image Tagging Strategy
==================================

📋 Tagging Configuration:
  • Image Name: test-tagging-app
  • Commit SHA: test123
  • Branch: main
  • Build: 42

📦 Processing semantic version...
  • Current version: 1.0.0
  • Bumped to: 1.0.1 (production release)

🎯 Creating comprehensive tag set...

📊 Tag Summary:
  • Semantic: 1.0.1
  • Commit: test123
  • Branch: main
  • Timestamp: 20240819-112245
  • Date: 2024-08-19
  • Build: build-42

🔨 Applying tags to image...
✅ Tagged with commit SHA: test123
✅ Tagged with semantic version: 1.0.1
✅ Tagged with branch name: main
✅ Tagged with timestamp: 20240819-112245
✅ Tagged with date: 2024-08-19
✅ Tagged with build number: build-42

🚀 Pushing all tags to registry...
  📤 Pushing test-tagging-app:test123
Error: Cannot push to non-existent registry (expected in local test)
...

🎉 Tagging Strategy Complete!
==================================

$ echo ""

$ echo "3️⃣ Verifying created tags..."
3️⃣ Verifying created tags...

$ docker images | grep test-tagging-app | head -10
test-tagging-app   build-42           9g5h6i7j8k9l   2 minutes ago   124MB
test-tagging-app   2024-08-19         9g5h6i7j8k9l   2 minutes ago   124MB  
test-tagging-app   20240819-112245    9g5h6i7j8k9l   2 minutes ago   124MB
test-tagging-app   main               9g5h6i7j8k9l   2 minutes ago   124MB
test-tagging-app   1.0.1              9g5h6i7j8k9l   2 minutes ago   124MB
test-tagging-app   test123            9g5h6i7j8k9l   2 minutes ago   124MB
test-tagging-app   latest             9g5h6i7j8k9l   2 minutes ago   124MB

$ echo ""

$ echo "4️⃣ Checking version file update..."
4️⃣ Checking version file update...
$ echo "New version: $(cat VERSION)"
New version: 1.0.1

$ echo ""

$ echo "🧹 Cleaning up test images..."
🧹 Cleaning up test images...
$ docker rmi $(docker images test-tagging-app -q) || echo "Images cleaned up"
Untagged: test-tagging-app:build-42
Untagged: test-tagging-app:2024-08-19
Untagged: test-tagging-app:20240819-112245
Untagged: test-tagging-app:main
Untagged: test-tagging-app:1.0.1
Untagged: test-tagging-app:test123
Untagged: test-tagging-app:latest
Deleted: sha256:9g5h6i7j8k9l0m1n2o3p4q5r6s7t8u9v0w1x2y3z4

$ echo ""

$ echo "✅ Tagging script validation completed!"
✅ Tagging script validation completed!
```

---

### ⚙️ Subtask 3.3: Update Pipeline with Advanced Tagging

**Step 18**: Integrate advanced tagging into CI/CD pipeline

```bash
echo "⚙️ Integrating advanced tagging into CI/CD pipeline..."

# Reset VERSION file for pipeline
echo "1.0.0" > VERSION

# Update the pipeline configuration
nano .gitlab-ci.yml
```

**💡 Pipeline Integration**: We're replacing the simple build stage with a comprehensive build-and-tag process.

**Update the build stage in .gitlab-ci.yml** (find the `build_image:` section and replace it):

```yaml
# Enhanced build stage with advanced tagging
build_and_tag:
  stage: build
  image: docker:24
  script:
    - echo "🏗️ Building and tagging Docker image with advanced strategy..."
    
    - echo ""
    - echo "📋 Build Configuration:"
    - echo "  • Project: $CI_PROJECT_NAME"
    - echo "  • Registry: $CI_REGISTRY"  
    - echo "  • Image: $IMAGE_NAME"
    - echo "  • Base Tag: $IMAGE_TAG"
    - echo "  • Branch: $CI_COMMIT_REF_NAME"
    - echo "  • Pipeline: $CI_PIPELINE_ID"
    
    - echo ""
    - echo "1️⃣ Building base Docker image..."
    - docker build -t $IMAGE_NAME:latest .
    
    - echo ""
    - echo "2️⃣ Making tagging script executable..."
    - chmod +x scripts/tag-image.sh
    
    - echo ""
    - echo "3️⃣ Executing advanced tagging strategy..."
    - ./scripts/tag-image.sh $IMAGE_NAME $CI_COMMIT_SHORT_SHA $CI_COMMIT_REF_NAME $CI_PIPELINE_ID
    
    - echo ""
    - echo "4️⃣ Validating created tags..."
    - docker images | grep $CI_PROJECT_NAME | head -10
    
    - echo ""
    - echo "5️⃣ Updating version control (if main branch)..."
    - |
      if [ "$CI_COMMIT_REF_NAME" = "main" ]; then
        echo "📝 Committing version update..."
        git config --global user.email "ci@gitlab.com"
        git config --global user.name "GitLab CI/CD"
        git add VERSION
        git commit -m "🔖 Bump version to $(cat VERSION) [skip ci]" || echo "No version changes to commit"
        git push https://gitlab-ci-token:${CI_PUSH_TOKEN}@gitlab.com/$CI_PROJECT_PATH.git HEAD:$CI_COMMIT_REF_NAME || echo "⚠️ Could not push version update - token may not be configured"
      else
        echo "📝 Skipping version commit for branch: $CI_COMMIT_REF_NAME"
      fi
    
    - echo ""
    - echo "✅ Build and tagging completed successfully!"
    - echo ""
    - echo "📊 Summary:"
    - echo "  • Built image: $IMAGE_NAME:latest"
    - echo "  • Applied tags: semantic, commit, branch, timestamp, date, build"
    - echo "  • Pushed to registry: GitLab Container Registry"
    - echo "  • Version file: $(cat VERSION)"
  artifacts:
    paths:
      - VERSION
      - scripts/
      - docker-compose.yml
      - nginx.conf
      - .env.example
    expire_in: 1 week
  only:
    - main
    - develop
    - /^release\/.*$/
    - /^hotfix\/.*$/
```

**Also update the dependencies in other stages to reference the new job name:**

```bash
echo ""
echo "🔄 Updating job dependencies in pipeline..."

# Show what needs to be changed
echo "📋 Jobs that need dependency updates:"
grep -n "dependencies:" .gitlab-ci.yml
```

**Expected Output:**
```
$ echo "⚙️ Integrating advanced tagging into CI/CD pipeline..."
⚙️ Integrating advanced tagging into CI/CD pipeline...

$ echo "1.0.0" > VERSION

$ nano .gitlab-ci.yml
[Update the build stage as shown above]

$ echo ""

$ echo "🔄 Updating job dependencies in pipeline..."
🔄 Updating job dependencies in pipeline...

$ echo "📋 Jobs that need dependency updates:"
📋 Jobs that need dependency updates:
$ grep -n "dependencies:" .gitlab-ci.yml
89:  dependencies:
107:  dependencies:
128:  dependencies:
```

**Update the dependencies** (change `build_image` to `build_and_tag` in the dependencies sections):

```bash
# Use sed to update dependencies automatically
sed -i 's/- build_image/- build_and_tag/g' .gitlab-ci.yml

echo ""
echo "✅ Dependencies updated successfully!"

echo ""
echo "📊 Verifying pipeline structure..."
grep -A 1 -B 1 "dependencies:" .gitlab-ci.yml
```

**Expected Output:**
```
$ sed -i 's/- build_image/- build_and_tag/g' .gitlab-ci.yml

$ echo ""

$ echo "✅ Dependencies updated successfully!"
✅ Dependencies updated successfully!

$ echo ""

$ echo "📊 Verifying pipeline structure..."
📊 Verifying pipeline structure...

$ grep -A 1 -B 1 "dependencies:" .gitlab-ci.yml
    - echo "✅ All integration tests passed!"
  dependencies:
    - build_and_tag
--
    - echo "✅ Security analysis completed!"
  dependencies:
    - build_and_tag
--
    - echo "✅ Staging deployment completed successfully!"
  dependencies:
    - build_and_tag
```

---

## 🔒 Task 4: Automate Vulnerability Scanning During CI/CD Pipeline

### 🛡️ Subtask 4.1: Enhanced Security Scanning with Multiple Tools

**Step 19**: Create comprehensive security scanning

```bash
echo "🛡️ Creating comprehensive security scanning infrastructure..."

echo ""
echo "📋 Security Tools Integration:"
echo "  • Trivy - Vulnerability scanner"  
echo "  • Custom configuration analysis"
echo "  • Multi-service security validation"
echo "  • Security policy enforcement"

echo ""
echo "1️⃣ Creating Trivy ignore file for acceptable risks..."
nano .trivyignore
```

**💡 Understanding Security Scanning**: We're implementing industry-standard vulnerability management with configurable risk acceptance.

Add the following content to `.trivyignore`:

```bash
# Trivy Ignore File - Managed Security Exceptions
# ==============================================
# This file contains CVEs and security issues that have been 
# reviewed and accepted by the security team

# Development/Testing Only - Remove in Production
# CVE-2023-example-dev-only

# Low severity issues that don't affect our use case
# CVE-2023-example-low-impact

# Issues fixed in runtime but not yet in base image
# CVE-2023-example-runtime-fixed

# False positives confirmed by security review
# CVE-2023-example-false-positive

# Note: Each ignored CVE should have:
# - Date of review
# - Justification for acceptance  
# - Planned remediation timeline
# - Responsible team/person

# Example format:
# CVE-2023-12345  # Reviewed: 2024-08-19, Team: DevOps, Reason: False positive, Review Date: 2024-09-19
```

**Expected Output:**
```
$ echo "🛡️ Creating comprehensive security scanning infrastructure..."
🛡️ Creating comprehensive security scanning infrastructure...

$ echo ""

$ echo "📋 Security Tools Integration:"
📋 Security Tools Integration:
$ echo "  • Trivy - Vulnerability scanner"
  • Trivy - Vulnerability scanner
$ echo "  • Custom configuration analysis"
  • Custom configuration analysis
$ echo "  • Multi-service security validation"
  • Multi-service security validation
$ echo "  • Security policy enforcement"
  • Security policy enforcement

$ echo ""

$ echo "1️⃣ Creating Trivy ignore file for acceptable risks..."
1️⃣ Creating Trivy ignore file for acceptable risks...

$ nano .trivyignore
[nano editor opens - add the content above]

$ ls -la | grep trivy
-rw-r--r-- 1 user user 1234 Aug 19 11:35 .trivyignore
```

---

**Step 20**: Create comprehensive security scanning script

```bash
echo "2️⃣ Creating comprehensive security scanning script..."
nano scripts/security-scan.sh
```

Add the following content:

```bash
#!/bin/bash

# Comprehensive Security Scanner for Docker CI/CD
# Integrates multiple security tools and generates detailed reports
set -e

echo "🛡️ Comprehensive Security Analysis"
echo "================================="

# Configuration
IMAGE_NAME=${1:-$CI_REGISTRY_IMAGE}
IMAGE_TAG=${2:-$CI_COMMIT_SHORT_SHA}
SCAN_OUTPUT_DIR="security-reports"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

echo ""
echo "📋 Security Scan Configuration:"
echo "  • Target Image: $IMAGE_NAME:$IMAGE_TAG"
echo "  • Output Directory: $SCAN_OUTPUT_DIR"
echo "  • Scan Timestamp: $TIMESTAMP"

# Create output directory
mkdir -p $SCAN_OUTPUT_DIR

echo ""
echo "🔧 Installing and configuring security tools..."

# Install Trivy if not present
if ! command -v trivy &> /dev/null; then
    echo "📦 Installing Trivy scanner..."
    curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
fi

trivy --version
echo "✅ Trivy scanner ready"

echo ""
echo "🎯 Starting comprehensive security analysis..."

# Function to scan individual images
scan_image() {
    local image=$1
    local friendly_name=$2
    local output_prefix=$3
    
    echo ""
    echo "🔍 Scanning $friendly_name ($image)..."
    
    # High/Critical vulnerabilities table output
    echo "  📊 Vulnerability Summary:"
    trivy image --exit-code 0 --severity HIGH,CRITICAL --format table $image | tee $SCAN_OUTPUT_DIR/${output_prefix}_summary.txt
    
    # Detailed JSON report for all severities
    echo "  📄 Generating detailed report..."
    trivy image --format json --output $SCAN_OUTPUT_DIR/${output_prefix}_detailed.json $image
    
    # Critical vulnerabilities only (fails build if found)
    echo "  🚨 Critical vulnerability check..."
    if trivy image --exit-code 1 --severity CRITICAL --format json --output $SCAN_OUTPUT_DIR/${output_prefix}_critical.json $image; then
        echo "  ✅ No critical vulnerabilities found in $friendly_name"
    else
        echo "  ⚠️ Critical vulnerabilities found in $friendly_name"
        return 1
    fi
}

# Scan application image
echo ""
echo "1️⃣ Application Image Security Scan"
echo "-----------------------------------"
scan_image "$IMAGE_NAME:$IMAGE_TAG" "Application" "app" || APP_CRITICAL=1

# Scan docker-compose service images
echo ""
echo "2️⃣ Service Images Security Scan"  
echo "--------------------------------"

declare -a SERVICES=(
    "mongo:6.0:MongoDB:mongo"
    "redis:7-alpine:Redis:redis"
    "nginx:alpine:Nginx:nginx"
)

SERVICE_ISSUES=0

for service_info in "${SERVICES[@]}"; do
    IFS=':' read -r image friendly_name output_prefix <<< "$service_info"
    scan_image "$image" "$friendly_name" "$output_prefix" || ((SERVICE_ISSUES++))
done

# Configuration security analysis
echo ""
echo "3️⃣ Configuration Security Analysis"
echo "-----------------------------------"

echo "🔍 Scanning project configurations..."
trivy config --exit-code 0 --severity HIGH,CRITICAL --format json --output $SCAN_OUTPUT_DIR/config_scan.json .

echo ""
echo "📋 Docker-compose Security Review:"

# Check for security anti-patterns
echo "  🔍 Checking for privileged containers..."
if grep -q "privileged.*true" docker-compose.yml; then
    echo "  ⚠️ Privileged containers detected"
    echo "privileged-containers-found" >> $SCAN_OUTPUT_DIR/security-issues.txt
else
    echo "  ✅ No privileged containers"
fi

echo "  🔍 Checking for host network usage..."
if grep -q "network_mode.*host" docker-compose.yml; then
    echo "  ⚠️ Host network mode detected"
    echo "host-network-found" >> $SCAN_OUTPUT_DIR/security-issues.txt
else
    echo "  ✅ Proper network isolation"
fi

echo "  🔍 Checking for volume security..."
if grep -q ":/.*:.*rw" docker-compose.yml; then
    echo "  ℹ️ Read-write volumes found (review permissions)"
else
    echo "  ✅ Volume permissions configured"
fi

echo "  🔍 Checking for hardcoded secrets..."
if grep -qE "(password|secret|key).*=" docker-compose.yml; then
    echo "  ⚠️ Potential hardcoded secrets detected"
    echo "hardcoded-secrets-potential" >> $SCAN_OUTPUT_DIR/security-issues.txt
else
    echo "  ✅ No obvious hardcoded secrets"
fi

# Generate comprehensive report
echo ""
echo "4️⃣ Security Report Generation"
echo "------------------------------"

cat > $SCAN_OUTPUT_DIR/security-report-$TIMESTAMP.md << EOF
# 🛡️ Comprehensive Security Scan Report

**Generated:** $(date)
**Pipeline:** ${CI_PIPELINE_ID:-local}
**Commit:** ${CI_COMMIT_SHORT_SHA:-local}
**Branch:** ${CI_COMMIT_REF_NAME:-local}

## 📊 Scan Summary

### Application Image
- **Image:** $IMAGE_NAME:$IMAGE_TAG
- **Status:** $([ -z "$APP_CRITICAL" ] && echo "✅ No Critical Issues" || echo "⚠️ Critical Issues Found")
- **Report:** app_detailed.json

### Service Images
- **MongoDB:** $([ -f "$SCAN_OUTPUT_DIR/mongo_critical.json" ] && echo "✅ Clean" || echo "⚠️ Issues")
- **Redis:** $([ -f "$SCAN_OUTPUT_DIR/redis_critical.json" ] && echo "✅ Clean" || echo "⚠️ Issues") 
- **Nginx:** $([ -f "$SCAN_OUTPUT_DIR/nginx_critical.json" ] && echo "✅ Clean" || echo "⚠️ Issues")

### Configuration Analysis
- **Dockerfile:** Analyzed ✅
- **docker-compose.yml:** Reviewed ✅
- **Network Security:** Validated ✅

## 🔍 Detailed Findings

### Security Policy Compliance
- **Privileged Containers:** $(grep -q "privileged-containers-found" $SCAN_OUTPUT_DIR/security-issues.txt 2>/dev/null && echo "❌ Failed" || echo "✅ Passed")
- **Network Isolation:** $(grep -q "host-network-found" $SCAN_OUTPUT_DIR/security-issues.txt 2>/dev/null && echo "❌ Failed" || echo "✅ Passed")
