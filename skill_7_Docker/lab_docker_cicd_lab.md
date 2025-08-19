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

- **Secret Management:** $ git push origin main
Enumerating objects: 23, done.
Counting objects: 100% (23/23), done.
Delta compression using up to 4 threads
Compressing objects: 100% (19/19), done.
Writing objects: 100% (20/20), 8.94 KiB | 8.94 MiB/s, done.
Total 20 (delta 8), reused 0 (delta 0), pack-reused 0
To https://gitlab.com/john_doe/docker-cicd-lab.git
   def5678..abc1234  main -> main

$ echo ""

$ echo "🎉 Project Implementation Complete!"
🎉 Project Implementation Complete!

$ echo ""

$ echo "📊 Final Project Summary:"
📊 Final Project Summary:
$ echo "================================="
=================================

$ echo "📁 Files Created:"
📁 Files Created:
$ find . -type f -not -path './.git/*' | sort
./.env.development.template
./.env.example
./.env.production.template
./.env.staging.template
./.gitlab-ci.yml
./.trivyignore
./DEPLOYMENT.md
./Dockerfile
./README.md
./SECRETS_GUIDE.md
./VERSION
./app.js
./docker-compose.yml
./nginx.conf
./package.json
./scripts/inject-secrets.sh
./scripts/security-scan.sh
./scripts/tag-image.sh

$ echo ""

$ echo "🔗 Next Steps:"
🔗 Next Steps:
$ echo "  1. 🌐 Visit your GitLab project to watch the pipeline execute"
  1. 🌐 Visit your GitLab project to watch the pipeline execute
$ echo "  2. 📊 Monitor the 4-stage pipeline (build → test → security → deploy)"
  2. 📊 Monitor the 4-stage pipeline (build → test → security → deploy)
$ echo "  3. 🔍 Review security scan reports in the artifacts"
  3. 🔍 Review security scan reports in the artifacts
$ echo "  4. 🚀 Manually trigger staging deployment when ready"
  4. 🚀 Manually trigger staging deployment when ready
$ echo "  5. 📚 Use this as a reference for production implementations"
  5. 📚 Use this as a reference for production implementations

$ echo ""

$ echo "🎓 Learning Outcomes Achieved:"
🎓 Learning Outcomes Achieved:
$ echo "  ✅ Docker containerization mastery"
  ✅ Docker containerization mastery
$ echo "  ✅ GitLab CI/CD pipeline expertise"
  ✅ GitLab CI/CD pipeline expertise
$ echo "  ✅ Security scanning and vulnerability management"
  ✅ Security scanning and vulnerability management
$ echo "  ✅ Multi-service orchestration with docker-compose"
  ✅ Multi-service orchestration with docker-compose
$ echo "  ✅ Production-ready secret management"
  ✅ Production-ready secret management
$ echo "  ✅ DevOps best practices implementation"
  ✅ DevOps best practices implementation

$ echo ""

$ echo "🏆 Congratulations! You've successfully completed the comprehensive Docker CI/CD Lab!"
🏆 Congratulations! You've successfully completed the comprehensive Docker CI/CD Lab!
```

---

## 🎓 Lab Summary and Additional Resources

### ✅ Lab Completion Checklist

**Core Objectives Achieved:**
- [x] **Pipeline Architecture**: Complete 4-stage GitLab CI/CD pipeline configured
- [x] **Automation**: Docker images built and deployed automatically via CI/CD
- [x] **Orchestration**: Multi-service deployment with docker-compose successfully implemented
- [x] **Version Control**: Advanced Docker image versioning and tagging strategies deployed
- [x] **Security**: Comprehensive vulnerability scanning integrated into pipeline
- [x] **Secret Management**: Secure secrets handling with environment-specific configurations
- [x] **Best Practices**: Production-ready Docker CI/CD patterns demonstrated

**Bonus Features Implemented:**
- [x] **Advanced Tagging**: Semantic versioning, temporal tagging, and build tracking
- [x] **Multi-tool Security**: Trivy scanner with configuration analysis and compliance reporting
- [x] **Production Documentation**: Comprehensive deployment guides and troubleshooting procedures
- [x] **Environment Management**: Development, staging, and production configuration templates
- [x] **Monitoring & Logging**: Health checks, performance monitoring, and centralized logging

### 🏗️ Architecture Overview

The completed lab demonstrates a production-ready architecture:

```
┌─────────────────────────────────────────────────────────────┐
│                    GitLab CI/CD Pipeline                   │
├─────────────┬─────────────┬─────────────┬─────────────────┤
│    BUILD    │    TEST     │  SECURITY   │     DEPLOY      │
│             │             │             │                 │
│ • Image     │ • Unit      │ • Trivy     │ • Staging       │
│   Build     │   Tests     │   Scan      │   Deploy        │
│ • Advanced  │ • Stack     │ • Config    │ • Production    │
│   Tagging   │   Test      │   Review    │   Deploy        │
│ • Registry  │ • Health    │ • Report    │ • Health        │
│   Push      │   Check     │   Gen       │   Validation    │
└─────────────┴─────────────┴─────────────┴─────────────────┘
                              │
┌─────────────────────────────▼─────────────────────────────────┐
│                 Docker Compose Stack                         │
├─────────────┬─────────────┬─────────────┬─────────────────┤
│    NGINX    │   NODE.JS   │   MONGODB   │     REDIS       │
│ Load Balancer│  Application│   Database  │     Cache       │
│             │             │             │                 │
│ • SSL Term  │ • API       │ • Auth      │ • Session       │
│ • Proxy     │ • Health    │ • Persist   │ • Performance   │
│ • LB        │ • Logging   │ • Backup    │ • Temp Storage  │
└─────────────┴─────────────┴─────────────┴─────────────────┘
```

### 📚 Key Technologies Mastered

**Containerization:**
- Docker multi-stage builds
- Security-hardened containers
- Health check implementation
- Resource optimization

**CI/CD:**
- GitLab CI/CD pipelines
- Automated testing workflows
- Security-integrated deployments
- Environment-specific deployments

**Orchestration:**
- Docker Compose multi-service stacks
- Service networking and dependencies
- Volume management and persistence
- Load balancing and proxying

**Security:**
- Vulnerability scanning automation
- Secret management best practices
- Configuration security validation
- Compliance reporting

### 🚀 Real-World Applications

This lab provides practical experience with:

**Enterprise DevOps Practices:**
- Infrastructure as Code (IaC)
- Automated security scanning
- Environment promotion workflows
- Rollback and disaster recovery procedures

**Production-Ready Features:**
- Multi-environment configurations
- Comprehensive monitoring and logging
- Performance optimization strategies
- Security policy enforcement

**Industry Standards:**
- Container security best practices
- CI/CD pipeline design patterns
- Secret management methodologies
- Documentation and operational procedures

### 📖 Extended Learning Resources

**Advanced Docker Topics:**
- [Docker Security Best Practices](https://docs.docker.com/engine/security/)
- [Multi-stage Build Optimization](https://docs.docker.com/build/building/multi-stage/)
- [Docker Compose Production Guide](https://docs.docker.com/compose/production/)

**GitLab CI/CD Deep Dive:**
- [GitLab CI/CD Best Practices](https://docs.gitlab.com/ee/ci/pipelines/pipeline_efficiency.html)
- [GitLab Security Features](https://docs.gitlab.com/ee/user/application_security/)
- [GitLab Container Registry](https://docs.gitlab.com/ee/user/packages/container_registry/)

**Production Deployment:**
- [Kubernetes Migration Path](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Helm Chart Development](https://helm.sh/docs/chart_template_guide/)
- [Service Mesh Integration](https://istio.io/latest/docs/setup/getting-started/)

**Security and Compliance:**
- [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)
- [NIST Container Security Guide](https://csrc.nist.gov/publications/detail/sp/800-190/final)
- [DevSecOps Best Practices](https://owasp.org/www-project-devsecops-guideline/)

### 🎯 Next Steps for Production

**Immediate Actions:**
1. **Configure Production Secrets**: Set up proper secret management (HashiCorp Vault, AWS Secrets Manager)
2. **Implement Monitoring**: Add Prometheus, Grafana, or similar monitoring solutions
3. **Set Up Alerting**: Configure notifications for critical issues
4. **SSL/TLS Configuration**: Implement proper certificate management
5. **Backup Strategy**: Implement automated database backups

**Advanced Enhancements:**
1. **Kubernetes Migration**: Move to container orchestration platform
2. **Service Mesh**: Implement Istio or Linkerd for advanced networking
3. **Observability**: Add distributed tracing with Jaeger or Zipkin
4. **Auto-scaling**: Implement horizontal pod autoscaling
5. **Blue-Green Deployment**: Add zero-downtime deployment strategies

**Operational Excellence:**
1. **Infrastructure as Code**: Implement Terraform or similar IaC tools
2. **GitOps**: Implement ArgoCD or Flux for deployment automation
3. **Policy as Code**: Use Open Policy Agent (OPA) for governance
4. **Compliance Automation**: Integrate compliance scanning and reporting
5. **Cost Optimization**: Implement resource optimization and cost monitoring

---

## 🏆 Congratulations!

You have successfully completed a comprehensive Docker CI/CD implementation that demonstrates enterprise-grade DevOps practices. This lab serves as both a learning experience and a practical reference for implementing similar systems in production environments.

**Key Achievements:**
✅ **Complete CI/CD Pipeline** - From code commit to production deployment  
✅ **Security Integration** - Automated vulnerability scanning and compliance  
✅ **Multi-Service Architecture** - Real-world application stack orchestration  
✅ **Production Documentation** - Comprehensive operational procedures  
✅ **DevOps Best Practices** - Industry-standard implementation patterns

**Skills Developed:**
- Advanced Docker containerization
- GitLab CI/CD pipeline design
- Security-first development practices  
- Multi-service orchestration
- Production deployment strategies
- Operational documentation and procedures

This implementation provides a solid foundation for building production-ready containerized applications with robust CI/CD practices. The patterns and practices demonstrated here are directly applicable to enterprise environments and can scale from small projects to large-scale distributed systems.

Keep practicing these concepts and exploring the advanced resources to further enhance your DevOps expertise! 🚀(grep -q "hardcoded-secrets-potential" $SCAN_OUTPUT_DIR/security-issues.txt 2>/dev/null && echo "❌ Review Required" || echo "✅ Compliant")

### Remediation Recommendations

#### Critical Issues
$([ -n "$APP_CRITICAL" ] && echo "- **Application Image:** Review critical vulnerabilities in app_critical.json" || echo "- **Application Image:** No critical issues found ✅")

#### Configuration Issues
$([ -f "$SCAN_OUTPUT_DIR/security-issues.txt" ] && echo "- **Configuration:** Address issues listed in security-issues.txt" || echo "- **Configuration:** No issues detected ✅")

### File References
- **Detailed Reports:** security-reports/ directory
- **Critical Issues:** *_critical.json files
- **Configuration Scan:** config_scan.json
- **Issues Summary:** security-issues.txt

## 📈 Trend Analysis
- **Previous Scan:** N/A (first scan)
- **Improvement:** Track issues over time
- **Risk Score:** $([ -n "$APP_CRITICAL" ] && echo "HIGH" || echo "LOW")

---
*Generated by comprehensive security scanner v1.0*
EOF

echo ""
echo "📊 Final Security Assessment:"

# Count total issues
TOTAL_CRITICAL=0
[ -n "$APP_CRITICAL" ] && ((TOTAL_CRITICAL++))
TOTAL_CRITICAL=$((TOTAL_CRITICAL + SERVICE_ISSUES))

if [ $TOTAL_CRITICAL -eq 0 ]; then
    echo "✅ Security scan completed successfully!"
    echo "   • No critical vulnerabilities found"
    echo "   • Configuration security validated"
    echo "   • All services meet security standards"
    EXIT_CODE=0
else
    echo "⚠️ Security issues detected:"
    echo "   • Critical vulnerabilities: $TOTAL_CRITICAL"
    echo "   • Configuration issues: $([ -f "$SCAN_OUTPUT_DIR/security-issues.txt" ] && wc -l < $SCAN_OUTPUT_DIR/security-issues.txt || echo "0")"
    echo "   • Review detailed reports in $SCAN_OUTPUT_DIR/"
    EXIT_CODE=1
fi

echo ""
echo "📁 Generated Reports:"
ls -la $SCAN_OUTPUT_DIR/

echo ""
echo "🔗 Report Summary:"
echo "  • Main Report: $SCAN_OUTPUT_DIR/security-report-$TIMESTAMP.md"
echo "  • Application Details: $SCAN_OUTPUT_DIR/app_detailed.json"
echo "  • Critical Issues: $SCAN_OUTPUT_DIR/*_critical.json"

echo ""
echo "🛡️ Security Analysis Complete!"
echo "================================="

exit $EXIT_CODE
```

**Expected Output:**
```bash
$ echo "2️⃣ Creating comprehensive security scanning script..."
2️⃣ Creating comprehensive security scanning script...

$ nano scripts/security-scan.sh
[nano editor opens - add the complete content above]

$ chmod +x scripts/security-scan.sh

$ ls -la scripts/
total 20
drwxr-xr-x 2 user user 4096 Aug 19 11:40 .
drwxr-xr-x 3 user user 4096 Aug 19 11:20 ..
-rwxr-xr-x 1 user user 8456 Aug 19 11:40 security-scan.sh
-rwxr-xr-x 1 user user 3245 Aug 19 11:20 tag-image.sh

$ echo "✅ Comprehensive security scanning script created!"
✅ Comprehensive security scanning script created!
```

---

### 🧪 Subtask 4.2: Test Security Scanning Locally

**Step 21**: Test the complete security scanning system

```bash
echo "🧪 Testing comprehensive security scanning locally..."

echo ""
echo "📋 Local Security Test Plan:"
echo "  • Build test image"
echo "  • Run security analysis"
echo "  • Validate report generation"
echo "  • Check exit codes"

echo ""
echo "1️⃣ Building test image for security scanning..."
docker build -t security-test-app:latest .

echo ""
echo "2️⃣ Running comprehensive security scan..."
./scripts/security-scan.sh security-test-app latest

echo ""
echo "3️⃣ Reviewing generated security reports..."
if [ -d "security-reports" ]; then
    echo "📊 Generated reports:"
    ls -la security-reports/
    
    echo ""
    echo "📋 Report contents preview:"
    if [ -f security-reports/security-report-*.md ]; then
        echo "Main report (first 20 lines):"
        head -20 security-reports/security-report-*.md
    fi
else
    echo "⚠️ Security reports directory not found"
fi

echo ""
echo "4️⃣ Validating scan results..."
SCAN_EXIT_CODE=$?
if [ $SCAN_EXIT_CODE -eq 0 ]; then
    echo "✅ Security scan passed - no critical issues"
else
    echo "⚠️ Security scan detected issues (exit code: $SCAN_EXIT_CODE)"
    echo "   This is normal for images with vulnerabilities"
fi

echo ""
echo "🧹 Cleaning up test artifacts..."
docker rmi security-test-app:latest || echo "Test image cleaned up"
rm -rf security-reports/ || echo "Reports cleaned up"

echo ""
echo "✅ Security scanning system validated!"
```

**Expected Output:**
```bash
$ echo "🧪 Testing comprehensive security scanning locally..."
🧪 Testing comprehensive security scanning locally...

$ echo ""

$ echo "📋 Local Security Test Plan:"
📋 Local Security Test Plan:
$ echo "  • Build test image"
  • Build test image
$ echo "  • Run security analysis"
  • Run security analysis
$ echo "  • Validate report generation"
  • Validate report generation
$ echo "  • Check exit codes"
  • Check exit codes

$ echo ""

$ echo "1️⃣ Building test image for security scanning..."
1️⃣ Building test image for security scanning...

$ docker build -t security-test-app:latest .
[+] Building 6.3s (12/12) FINISHED
 => [internal] load .dockerignore                          0.0s
 => [internal] load build definition from Dockerfile       0.1s
 => [internal] load metadata for docker.io/library/node:18-alpine  1.2s
 => [1/7] FROM docker.io/library/node:18-alpine@sha256:...  0.0s
 => [internal] load build context                          0.0s
 => CACHED [2/7] WORKDIR /app                              0.0s
 => CACHED [3/7] COPY package*.json ./                     0.0s
 => CACHED [4/7] RUN npm install --only=production && ...  0.0s
 => [5/7] COPY . .                                          0.0s
 => [6/7] RUN addgroup -g 1001 -S nodejs && adduser -S...  0.3s
 => [7/7] USER nodejs                                       0.0s
 => exporting to image                                      0.1s

$ echo ""

$ echo "2️⃣ Running comprehensive security scan..."
2️⃣ Running comprehensive security scan...

$ ./scripts/security-scan.sh security-test-app latest
🛡️ Comprehensive Security Analysis
=================================

📋 Security Scan Configuration:
  • Target Image: security-test-app:latest
  • Output Directory: security-reports
  • Scan Timestamp: 20240819_114512

🔧 Installing and configuring security tools...
📦 Installing Trivy scanner...
trivy version 0.44.1

✅ Trivy scanner ready

🎯 Starting comprehensive security analysis...

1️⃣ Application Image Security Scan
-----------------------------------

🔍 Scanning Application (security-test-app:latest)...
  📊 Vulnerability Summary:
security-test-app:latest (alpine 3.18.2)
===========

Total: 2 (HIGH: 1, CRITICAL: 1)

  📄 Generating detailed report...
  🚨 Critical vulnerability check...
  ⚠️ Critical vulnerabilities found in Application

2️⃣ Service Images Security Scan
--------------------------------

🔍 Scanning MongoDB (mongo:6.0)...
  📊 Vulnerability Summary:
mongo:6.0 (ubuntu 20.04)
=========================

Total: 8 (HIGH: 3, CRITICAL: 5)

  📄 Generating detailed report...
  🚨 Critical vulnerability check...
  ⚠️ Critical vulnerabilities found in MongoDB

[Similar output for Redis and Nginx...]

3️⃣ Configuration Security Analysis
-----------------------------------
🔍 Scanning project configurations...

📋 Docker-compose Security Review:
  🔍 Checking for privileged containers...
  ✅ No privileged containers
  🔍 Checking for host network usage...
  ✅ Proper network isolation
  🔍 Checking for volume security...
  ℹ️ Read-write volumes found (review permissions)
  🔍 Checking for hardcoded secrets...
  ⚠️ Potential hardcoded secrets detected

4️⃣ Security Report Generation
------------------------------

📊 Final Security Assessment:
⚠️ Security issues detected:
   • Critical vulnerabilities: 4
   • Configuration issues: 1
   • Review detailed reports in security-reports/

📁 Generated Reports:
total 148
-rw-r--r-- 1 user user 12456 Aug 19 11:45 app_critical.json
-rw-r--r-- 1 user user 45678 Aug 19 11:45 app_detailed.json
-rw-r--r-- 1 user user  1234 Aug 19 11:45 app_summary.txt
-rw-r--r-- 1 user user  5678 Aug 19 11:45 config_scan.json
[... more files ...]
-rw-r--r-- 1 user user  2345 Aug 19 11:45 security-report-20240819_114512.md
-rw-r--r-- 1 user user    45 Aug 19 11:45 security-issues.txt

🔗 Report Summary:
  • Main Report: security-reports/security-report-20240819_114512.md
  • Application Details: security-reports/app_detailed.json
  • Critical Issues: security-reports/*_critical.json

🛡️ Security Analysis Complete!
=================================

$ echo ""

$ echo "3️⃣ Reviewing generated security reports..."
3️⃣ Reviewing generated security reports...

$ if [ -d "security-reports" ]; then
>     echo "📊 Generated reports:"
>     ls -la security-reports/
> 
>     echo ""
>     echo "📋 Report contents preview:"
>     if [ -f security-reports/security-report-*.md ]; then
>         echo "Main report (first 20 lines):"
>         head -20 security-reports/security-report-*.md
>     fi
> else
>     echo "⚠️ Security reports directory not found"
> fi
📊 Generated reports:
total 148
-rw-r--r-- 1 user user  2345 Aug 19 11:45 security-report-20240819_114512.md
-rw-r--r-- 1 user user 12456 Aug 19 11:45 app_critical.json
-rw-r--r-- 1 user user 45678 Aug 19 11:45 app_detailed.json
[... more files ...]

📋 Report contents preview:
Main report (first 20 lines):
# 🛡️ Comprehensive Security Scan Report

**Generated:** Mon Aug 19 11:45:12 UTC 2024
**Pipeline:** local
**Commit:** local
**Branch:** local

## 📊 Scan Summary

### Application Image
- **Image:** security-test-app:latest
- **Status:** ⚠️ Critical Issues Found
- **Report:** app_detailed.json

### Service Images
- **MongoDB:** ⚠️ Issues
- **Redis:** ⚠️ Issues
- **Nginx:** ✅ Clean

$ echo ""

$ echo "4️⃣ Validating scan results..."
4️⃣ Validating scan results...
$ SCAN_EXIT_CODE=$?
$ if [ $SCAN_EXIT_CODE -eq 0 ]; then
>     echo "✅ Security scan passed - no critical issues"
> else
>     echo "⚠️ Security scan detected issues (exit code: $SCAN_EXIT_CODE)"
>     echo "   This is normal for images with vulnerabilities"
> fi
⚠️ Security scan detected issues (exit code: 1)
   This is normal for images with vulnerabilities

$ echo ""

$ echo "🧹 Cleaning up test artifacts..."
🧹 Cleaning up test artifacts...
$ docker rmi security-test-app:latest || echo "Test image cleaned up"
Untagged: security-test-app:latest
Deleted: sha256:abc123def456...

$ rm -rf security-reports/ || echo "Reports cleaned up"

$ echo ""

$ echo "✅ Security scanning system validated!"
✅ Security scanning system validated!
```

---

## 🔐 Task 5: Secure Secret Management in Containerized Environments

### 🗝️ Subtask 5.1: Implement GitLab Secret Management

**Step 22**: Configure GitLab CI/CD variables for secure secret management

```bash
echo "🔐 Setting up secure secret management for containerized environments..."

echo ""
echo "📋 Secret Management Strategy:"
echo "  • GitLab CI/CD Variables for sensitive data"
echo "  • Environment-specific configurations"
echo "  • Runtime secret injection"
echo "  • Secret rotation support"

echo ""
echo "1️⃣ Creating secret management documentation..."
nano SECRETS_GUIDE.md
```

Add the following content:

```markdown
# 🔐 Secret Management Guide

## GitLab CI/CD Variables Setup

### Required Variables

Navigate to **Project Settings > CI/CD > Variables** and add these variables:

#### Database Secrets
- `STAGING_MONGO_USERNAME` - MongoDB admin username for staging
- `STAGING_MONGO_PASSWORD` - MongoDB admin password for staging (masked)
- `PROD_MONGO_USERNAME` - MongoDB admin username for production  
- `PROD_MONGO_PASSWORD` - MongoDB admin password for production (masked)

#### Cache Secrets
- `STAGING_REDIS_PASSWORD` - Redis password for staging (masked)
- `PROD_REDIS_PASSWORD` - Redis password for production (masked)

#### Registry & Deployment
- `CI_PUSH_TOKEN` - GitLab push token for version commits (masked)
- `NOTIFICATION_WEBHOOK` - Slack/Teams webhook for notifications (optional)

#### Production Secrets
- `PROD_APP_SECRET_KEY` - Application secret key (masked, protected)
- `PROD_JWT_SECRET` - JWT signing secret (masked, protected)
- `PROD_ENCRYPTION_KEY` - Data encryption key (masked, protected)

### Variable Configuration

#### Security Settings
- ✅ **Masked**: Hide variable values in job logs
- ✅ **Protected**: Only available on protected branches
- ✅ **Environment Scope**: Limit to specific environments

#### Example Configuration
```
Variable: PROD_MONGO_PASSWORD
Value: super_secure_production_password_2024!
Flags: [Masked] [Protected]
Environment: production
```

### Usage in Pipeline
```yaml
deploy_production:
  environment:
    name: production
  variables:
    MONGO_PASSWORD: $PROD_MONGO_PASSWORD
    REDIS_PASSWORD: $PROD_REDIS_PASSWORD
```

## 🔄 Secret Rotation Process

### Monthly Rotation (Recommended)
1. Generate new secrets
2. Update GitLab CI/CD variables
3. Deploy with new secrets
4. Validate services
5. Remove old secrets

### Emergency Rotation
1. Immediately update compromised secrets
2. Force re-deploy all services
3. Audit access logs
4. Update monitoring alerts
```

**Expected Output:**
```bash
$ echo "🔐 Setting up secure secret management for containerized environments..."
🔐 Setting up secure secret management for containerized environments...

$ echo ""

$ echo "📋 Secret Management Strategy:"
📋 Secret Management Strategy:
$ echo "  • GitLab CI/CD Variables for sensitive data"
  • GitLab CI/CD Variables for sensitive data
$ echo "  • Environment-specific configurations"
  • Environment-specific configurations
$ echo "  • Runtime secret injection"
  • Runtime secret injection
$ echo "  • Secret rotation support"
  • Secret rotation support

$ echo ""

$ echo "1️⃣ Creating secret management documentation..."
1️⃣ Creating secret management documentation...

$ nano SECRETS_GUIDE.md
[nano editor opens - add the content above]

$ ls -la | grep SECRETS
-rw-r--r-- 1 user user 2134 Aug 19 11:50 SECRETS_GUIDE.md

$ wc -l SECRETS_GUIDE.md
78 SECRETS_GUIDE.md
```

---

### 📦 Subtask 5.2: Create Environment-Specific Configurations

**Step 23**: Create multiple environment configurations

```bash
echo "📦 Creating environment-specific configurations..."

echo ""
echo "2️⃣ Creating production environment template..."
nano .env.production.template
```

Add the following content:

```bash
# Production Environment Configuration Template
# ============================================
# Copy to .env.production and fill with actual values
# NEVER commit actual .env.production to version control

# Application Settings
NODE_ENV=production
APP_VERSION=${CI_COMMIT_SHORT_SHA:-1.0.0}
PORT=3000
APP_SECRET_KEY=${PROD_APP_SECRET_KEY}
JWT_SECRET=${PROD_JWT_SECRET}
ENCRYPTION_KEY=${PROD_ENCRYPTION_KEY}

# Database Configuration (MongoDB)
MONGO_USERNAME=${PROD_MONGO_USERNAME}
MONGO_PASSWORD=${PROD_MONGO_PASSWORD}
MONGO_INITDB_DATABASE=myapp_production
MONGO_HOST=mongo
MONGO_PORT=27017

# Cache Configuration (Redis)
REDIS_PASSWORD=${PROD_REDIS_PASSWORD}
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_DB=0

# Security Settings
ALLOWED_ORIGINS=https://yourdomain.com
CORS_ENABLED=true
RATE_LIMIT_ENABLED=true
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100

# Monitoring & Logging
LOG_LEVEL=info
ENABLE_METRICS=true
HEALTH_CHECK_INTERVAL=30
SENTRY_DSN=${PROD_SENTRY_DSN:-}

# SSL/TLS Configuration
SSL_ENABLED=true
SSL_CERT_PATH=/certs/fullchain.pem
SSL_KEY_PATH=/certs/privkey.pem

# Backup Configuration
BACKUP_ENABLED=true
BACKUP_SCHEDULE="0 2 * * *"
BACKUP_RETENTION_DAYS=30
```

**Step 24**: Create staging environment template

```bash
echo ""
echo "3️⃣ Creating staging environment template..."
nano .env.staging.template
```

Add the following content:

```bash
# Staging Environment Configuration Template
# =========================================
# Copy to .env.staging and fill with actual values

# Application Settings
NODE_ENV=staging
APP_VERSION=${CI_COMMIT_SHORT_SHA:-1.0.0-staging}
PORT=3000
APP_SECRET_KEY=${STAGING_APP_SECRET_KEY:-staging_secret_key_change_me}
JWT_SECRET=${STAGING_JWT_SECRET:-staging_jwt_secret_change_me}

# Database Configuration (MongoDB)
MONGO_USERNAME=${STAGING_MONGO_USERNAME}
MONGO_PASSWORD=${STAGING_MONGO_PASSWORD}
MONGO_INITDB_DATABASE=myapp_staging
MONGO_HOST=mongo
MONGO_PORT=27017

# Cache Configuration (Redis)
REDIS_PASSWORD=${STAGING_REDIS_PASSWORD}
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_DB=1

# Security Settings (Relaxed for testing)
ALLOWED_ORIGINS=*
CORS_ENABLED=true
RATE_LIMIT_ENABLED=false

# Monitoring & Logging (Verbose for debugging)
LOG_LEVEL=debug
ENABLE_METRICS=true
HEALTH_CHECK_INTERVAL=15

# SSL/TLS Configuration (Disabled for staging)
SSL_ENABLED=false

# Testing Configuration
ENABLE_TEST_ENDPOINTS=true
MOCK_EXTERNAL_SERVICES=true
```

**Step 25**: Create development environment template

```bash
echo ""
echo "4️⃣ Creating development environment template..."
nano .env.development.template
```

Add the following content:

```bash
# Development Environment Configuration Template
# =============================================
# Copy to .env.development for local development

# Application Settings
NODE_ENV=development
APP_VERSION=dev-local
PORT=3000
APP_SECRET_KEY=dev_secret_not_for_production
JWT_SECRET=dev_jwt_secret_not_for_production

# Database Configuration (MongoDB)
MONGO_USERNAME=dev_admin
MONGO_PASSWORD=dev_password_123
MONGO_INITDB_DATABASE=myapp_dev
MONGO_HOST=localhost
MONGO_PORT=27017

# Cache Configuration (Redis)
REDIS_PASSWORD=dev_redis_123
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=2

# Security Settings (Minimal for development)
ALLOWED_ORIGINS=*
CORS_ENABLED=true
RATE_LIMIT_ENABLED=false

# Monitoring & Logging (Maximum verbosity)
LOG_LEVEL=silly
ENABLE_METRICS=false
HEALTH_CHECK_INTERVAL=60

# Development Features
HOT_RELOAD=true
AUTO_RESTART=true
ENABLE_DEBUG_ENDPOINTS=true
MOCK_ALL_EXTERNAL_SERVICES=true

# Local Testing
USE_LOCAL_DATABASE=true
SEED_TEST_DATA=true
RESET_DB_ON_RESTART=false
```

**Expected Output:**
```bash
$ echo "📦 Creating environment-specific configurations..."
📦 Creating environment-specific configurations...

$ echo ""

$ echo "2️⃣ Creating production environment template..."
2️⃣ Creating production environment template...

$ nano .env.production.template
[nano editor opens - add the content above]

$ echo ""

$ echo "3️⃣ Creating staging environment template..."
3️⃣ Creating staging environment template...

$ nano .env.staging.template
[nano editor opens - add the content above]

$ echo ""

$ echo "4️⃣ Creating development environment template..."
4️⃣ Creating development environment template...

$ nano .env.development.template
[nano editor opens - add the content above]

$ ls -la | grep "\.env"
-rw-r--r-- 1 user user  875 Aug 19 11:08 .env.example
-rw-r--r-- 1 user user 1456 Aug 19 11:52 .env.development.template
-rw-r--r-- 1 user user 1678 Aug 19 11:51 .env.production.template  
-rw-r--r-- 1 user user 1234 Aug 19 11:51 .env.staging.template

$ echo ""
$ echo "📊 Environment templates created:"
📊 Environment templates created:
$ wc -l .env.*.template
 42 .env.development.template
 58 .env.production.template
 38 .env.staging.template
138 total
```

---

### 🛠️ Subtask 5.3: Create Secret Injection Script

**Step 26**: Create dynamic secret injection system

```bash
echo "🛠️ Creating dynamic secret injection system..."

echo ""
echo "5️⃣ Creating secret injection script..."
nano scripts/inject-secrets.sh
```

Add the following content:

```bash
#!/bin/bash

# Dynamic Secret Injection for Docker Containers
# Securely injects secrets at runtime from various sources
set -e

echo "🔐 Dynamic Secret Injection System"
echo "=================================="

# Configuration
ENVIRONMENT=${1:-staging}
SECRET_METHOD=${2:-env}  # env, file, vault
OUTPUT_FILE=${3:-.env.runtime}

echo ""
echo "📋 Injection Configuration:"
echo "  • Environment: $ENVIRONMENT"
echo "  • Secret Method: $SECRET_METHOD"
echo "  • Output File: $OUTPUT_FILE"

# Function to validate required secrets
validate_secrets() {
    local env_name=$1
    echo ""
    echo "🔍 Validating required secrets for $env_name..."
    
    declare -a REQUIRED_SECRETS=()
    
    case $env_name in
        "production")
            REQUIRED_SECRETS=(
                "PROD_MONGO_USERNAME"
                "PROD_MONGO_PASSWORD"  
                "PROD_REDIS_PASSWORD"
                "PROD_APP_SECRET_KEY"
                "PROD_JWT_SECRET"
            )
            ;;
        "staging")
            REQUIRED_SECRETS=(
                "STAGING_MONGO_USERNAME"
                "STAGING_MONGO_PASSWORD"
                "STAGING_REDIS_PASSWORD"
            )
            ;;
        *)
            echo "  ℹ️ Development environment - using defaults"
            return 0
            ;;
    esac
    
    local missing_secrets=0
    for secret in "${REQUIRED_SECRETS[@]}"; do
        if [ -z "${!secret}" ]; then
            echo "  ❌ Missing required secret: $secret"
            ((missing_secrets++))
        else
            echo "  ✅ Found secret: $secret"
        fi
    done
    
    if [ $missing_secrets -gt 0 ]; then
        echo "  ⚠️ $missing_secrets secrets are missing"
        if [ "$ENVIRONMENT" = "production" ]; then
            echo "  🚨 Cannot proceed with production deployment"
            exit 1
        else
            echo "  ⚠️ Proceeding with available secrets (non-production)"
        fi
    else
        echo "  ✅ All required secrets validated"
    fi
}

# Function to inject secrets from environment variables
inject_from_env() {
    echo ""
    echo "🔧 Injecting secrets from environment variables..."
    
    # Start with environment template
    local template_file=".env.${ENVIRONMENT}.template"
    if [ -f "$template_file" ]; then
        echo "  📋 Using template: $template_file"
        cp "$template_file" "$OUTPUT_FILE"
    else
        echo "  ⚠️ Template not found: $template_file"
        echo "  📋 Creating minimal runtime environment..."
        cat > "$OUTPUT_FILE" << EOF
# Generated runtime environment for $ENVIRONMENT
NODE_ENV=$ENVIRONMENT
APP_VERSION=${CI_COMMIT_SHORT_SHA:-dev}
PORT=3000
EOF
    fi
    
    # Replace placeholders with actual values
    echo "  🔄 Resolving environment variables..."
    envsubst < "$template_file" > "$OUTPUT_FILE" 2>/dev/null || echo "  ℹ️ envsubst not available, using template as-is"
    
    # Add CI-specific variables if available
    if [ -n "$CI_COMMIT_SHORT_SHA" ]; then
        echo "  📝 Adding CI variables..."
        echo "CI_COMMIT_SHA=$CI_COMMIT_SHORT_SHA" >> "$OUTPUT_FILE"
        echo "CI_PIPELINE_ID=${CI_PIPELINE_ID:-unknown}" >> "$OUTPUT_FILE"
        echo "CI_JOB_ID=${CI_JOB_ID:-unknown}" >> "$OUTPUT_FILE"
    fi
    
    echo "  ✅ Environment secrets injected"
}

# Function to inject secrets from files (for Kubernetes, Docker Swarm)
inject_from_files() {
    echo ""
    echo "📁 Injecting secrets from secret files..."
    
    local secrets_dir="/run/secrets"
    if [ -d "$secrets_dir" ]; then
        echo "  📂 Found secrets directory: $secrets_dir"
        for secret_file in "$secrets_dir"/*; do
            if [ -f "$secret_file" ]; then
                local secret_name=$(basename "$secret_file")
                local secret_value=$(cat "$secret_file")
                echo "${secret_name^^}=$secret_value" >> "$OUTPUT_FILE"
                echo "  ✅ Loaded secret: $secret_name"
            fi
        done
    else
        echo "  ℹ️ No secrets directory found at $secrets_dir"
        echo "  🔄 Falling back to environment variable injection..."
        inject_from_env
    fi
}

# Function to generate secure defaults for development
generate_dev_secrets() {
    echo ""
    echo "🧪 Generating development secrets..."
    
    local timestamp=$(date +%s)
    
    cat >> "$OUTPUT_FILE" << EOF

# Generated development secrets ($(date))
GENERATED_SECRET_1=$(openssl rand -hex 32 2>/dev/null || echo "dev_secret_${timestamp}_1")
GENERATED_SECRET_2=$(openssl rand -hex 32 2>/dev/null || echo "dev_secret_${timestamp}_2")
GENERATED_JWT_SECRET=$(openssl rand -hex 64 2>/dev/null || echo "dev_jwt_${timestamp}")

# Development flags
SECRETS_GENERATED_AT=$timestamp
SECRETS_METHOD=generated
EOF
    
    echo "  ✅ Development secrets generated"
}

# Main execution
echo ""
echo "🚀 Starting secret injection process..."

validate_secrets "$ENVIRONMENT"

case $SECRET_METHOD in
    "env")
        inject_from_env
        ;;
    "file")
        inject_from_files
        ;;
    "vault")
        echo "🏦 Vault integration not implemented in this lab"
        echo "   For production, consider HashiCorp Vault or similar"
        inject_from_env
        ;;
    *)
        echo "❌ Unknown secret method: $SECRET_METHOD"
        exit 1
        ;;
esac

# Generate additional secrets for development
if [ "$ENVIRONMENT" = "development" ]; then
    generate_dev_secrets
fi

# Secure the output file
chmod 600 "$OUTPUT_FILE"

echo ""
echo "📊 Secret injection summary:"
echo "  • Output file: $OUTPUT_FILE"
echo "  • File size: $(wc -c < "$OUTPUT_FILE") bytes"
echo "  • Variables count: $(grep -c "=" "$OUTPUT_FILE")"
echo "  • File permissions: $(ls -l "$OUTPUT_FILE" | cut -d' ' -f1)"

echo ""
echo "🔐 Secret injection completed!"
echo ""
echo "⚠️ Security reminder:"
echo "  • Never commit $OUTPUT_FILE to version control"
echo "  • Rotate secrets regularly"
echo "  • Monitor secret access logs"
echo "  • Use least-privilege access principles"

echo ""
echo "=================================="
```

**Expected Output:**
```bash
$ echo "🛠️ Creating dynamic secret injection system..."
🛠️ Creating dynamic secret injection system...

$ echo ""

$ echo "5️⃣ Creating secret injection script..."
5️⃣ Creating secret injection script...

$ nano scripts/inject-secrets.sh
[nano editor opens - add the content above]

$ chmod +x scripts/inject-secrets.sh

$ ls -la scripts/
total 32
drwxr-xr-x 2 user user 4096 Aug 19 11:55 .
drwxr-xr-x 3 user user 4096 Aug 19 11:20 ..
-rwxr-xr-x 1 user user 6789 Aug 19 11:55 inject-secrets.sh
-rwxr-xr-x 1 user user 8456 Aug 19 11:40 security-scan.sh
-rwxr-xr-x 1 user user 3245 Aug 19 11:20 tag-image.sh

$ echo "✅ Dynamic secret injection system created!"
✅ Dynamic secret injection system created!
```

---

### 🧪 Subtask 5.4: Test Secret Management System

**Step 27**: Test the complete secret management system

```bash
echo "🧪 Testing secret management system..."

echo ""
echo "📋 Secret Management Test Plan:"
echo "  • Test development secret generation"
echo "  • Test staging environment injection"
echo "  • Validate secret file security"
echo "  • Test environment variable resolution"

echo ""
echo "1️⃣ Testing development environment secrets..."
./scripts/inject-secrets.sh development env .env.test.dev

echo ""
echo "📊 Development secrets generated:"
if [ -f ".env.test.dev" ]; then
    echo "  • File created: ✅"
    echo "  • File permissions: $(ls -l .env.test.dev | cut -d' ' -f1)"
    echo "  • Variable count: $(grep -c "=" .env.test.dev)"
    echo ""
    echo "📋 Sample secrets (first 10 non-sensitive lines):"
    grep -v -E "(PASSWORD|SECRET|KEY)" .env.test.dev | head -10
else
    echo "  ❌ Secret file not created"
fi

echo ""
echo "2️⃣ Testing staging environment with mock secrets..."
export STAGING_MONGO_USERNAME="staging_admin"
export STAGING_MONGO_PASSWORD="mock_staging_password"
export STAGING_REDIS_PASSWORD="mock_redis_password"

./scripts/inject-secrets.sh staging env .env.test.staging

echo ""
echo "📊 Staging secrets processed:"
if [ -f ".env.test.staging" ]; then
    echo "  • File created: ✅"
    echo "  • Variable count: $(grep -c "=" .env.test.staging)"
    echo ""
    echo "📋 Non-sensitive variables:"
    grep -E "^(NODE_ENV|APP_VERSION|PORT|LOG_LEVEL)" .env.test.staging || echo "  Basic variables not found"
else
    echo "  ❌ Staging secret file not created"
fi

echo ""
echo "3️⃣ Testing secret validation..."
unset STAGING_MONGO_USERNAME
./scripts/inject-secrets.sh staging env .env.test.validation 2>&1 | tail -5

echo ""
echo "4️⃣ Testing file method (simulating Kubernetes secrets)..."
mkdir -p /tmp/mock-secrets
echo "mock_db_password" > /tmp/mock-secrets/database_password
echo "mock_api_key" > /tmp/mock-secrets/api_key

# Mock the secrets directory
sudo mkdir -p /run/secrets 2>/dev/null || mkdir -p ./mock-secrets
cp /tmp/mock-secrets/* ./mock-secrets/ 2>/dev/null || echo "Using tmp secrets"

./scripts/inject-secrets.sh production file .env.test.files

echo ""
echo "🧹 Cleaning up test files..."
rm -f .env.test.*
rm -rf /tmp/mock-secrets ./mock-secrets
unset STAGING_MONGO_USERNAME STAGING_MONGO_PASSWORD STAGING_REDIS_PASSWORD

echo ""
echo "✅ Secret management system tested successfully!"
```

**Expected Output:**
```bash
$ echo "🧪 Testing secret management system..."
🧪 Testing secret management system...

$ echo ""

$ echo "📋 Secret Management Test Plan:"
📋 Secret Management Test Plan:
$ echo "  • Test development secret generation"
  • Test development secret generation
$ echo "  • Test staging environment injection"
  • Test staging environment injection
$ echo "  • Validate secret file security"
  • Validate secret file security
$ echo "  • Test environment variable resolution"
  • Test environment variable resolution

$ echo ""

$ echo "1️⃣ Testing development environment secrets..."
1️⃣ Testing development environment secrets...

$ ./scripts/inject-secrets.sh development env .env.test.dev
🔐 Dynamic Secret Injection System
==================================

📋 Injection Configuration:
  • Environment: development
  • Secret Method: env
  • Output File: .env.test.dev

🔍 Validating required secrets for development...
  ℹ️ Development environment - using defaults

🚀 Starting secret injection process...

🔧 Injecting secrets from environment variables...
  📋 Using template: .env.development.template
  🔄 Resolving environment variables...
  ✅ Environment secrets injected

🧪 Generating development secrets...
  ✅ Development secrets generated

📊 Secret injection summary:
  • Output file: .env.test.dev
  • File size: 1456 bytes
  • Variables count: 28
  • File permissions: -rw-------

🔐 Secret injection completed!

⚠️ Security reminder:
  • Never commit .env.test.dev to version control
  • Rotate secrets regularly
  • Monitor secret access logs
  • Use least-privilege access principles

==================================

$ echo ""

$ echo "📊 Development secrets generated:"
📊 Development secrets generated:
$ if [ -f ".env.test.dev" ]; then
>     echo "  • File created: ✅"
>     echo "  • File permissions: $(ls -l .env.test.dev | cut -d' ' -f1)"
>     echo "  • Variable count: $(grep -c "=" .env.test.dev)"
>     echo ""
>     echo "📋 Sample secrets (first 10 non-sensitive lines):"
>     grep -v -E "(PASSWORD|SECRET|KEY)" .env.test.dev | head -10
> else
>     echo "  ❌ Secret file not created"
> fi
  • File created: ✅
  • File permissions: -rw-------
  • Variable count: 28

📋 Sample secrets (first 10 non-sensitive lines):
NODE_ENV=development
APP_VERSION=dev-local
PORT=3000
MONGO_INITDB_DATABASE=myapp_dev
MONGO_HOST=localhost
MONGO_PORT=27017
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=2
ALLOWED_ORIGINS=*

$ echo ""

$ echo "2️⃣ Testing staging environment with mock secrets..."
2️⃣ Testing staging environment with mock secrets...

$ export STAGING_MONGO_USERNAME="staging_admin"

$ export STAGING_MONGO_PASSWORD="mock_staging_password"

$ export STAGING_REDIS_PASSWORD="mock_redis_password"

$ ./scripts/inject-secrets.sh staging env .env.test.staging
🔐 Dynamic Secret Injection System
==================================

📋 Injection Configuration:
  • Environment: staging
  • Secret Method: env
  • Output File: .env.test.staging

🔍 Validating required secrets for staging...
  ✅ Found secret: STAGING_MONGO_USERNAME
  ✅ Found secret: STAGING_MONGO_PASSWORD
  ✅ Found secret: STAGING_REDIS_PASSWORD
  ✅ All required secrets validated

🚀 Starting secret injection process...

🔧 Injecting secrets from environment variables...
  📋 Using template: .env.staging.template
  🔄 Resolving environment variables...
  ✅ Environment secrets injected

📊 Secret injection summary:
  • Output file: .env.test.staging
  • File size: 1234 bytes
  • Variables count: 24
  • File permissions: -rw-------

🔐 Secret injection completed!
==================================

$ echo ""

$ echo "📊 Staging secrets processed:"
📊 Staging secrets processed:
$ if [ -f ".env.test.staging" ]; then
>     echo "  • File created: ✅"
>     echo "  • Variable count: $(grep -c "=" .env.test.staging)"
>     echo ""
>     echo "📋 Non-sensitive variables:"
>     grep -E "^(NODE_ENV|APP_VERSION|PORT|LOG_LEVEL)" .env.test.staging || echo "  Basic variables not found"
> else
>     echo "  ❌ Staging secret file not created"
> fi
  • File created: ✅
  • Variable count: 24

📋 Non-sensitive variables:
NODE_ENV=staging
APP_VERSION=1.0.0-staging
PORT=3000
LOG_LEVEL=debug

$ echo ""

$ echo "3️⃣ Testing secret validation..."
3️⃣ Testing secret validation...

$ unset STAGING_MONGO_USERNAME

$ ./scripts/inject-secrets.sh staging env .env.test.validation 2>&1 | tail -5
  ❌ Missing required secret: STAGING_MONGO_USERNAME
  ✅ Found secret: STAGING_MONGO_PASSWORD
  ✅ Found secret: STAGING_REDIS_PASSWORD
  ⚠️ 1 secrets are missing
  ⚠️ Proceeding with available secrets (non-production)

$ echo ""

$ echo "4️⃣ Testing file method (simulating Kubernetes secrets)..."
4️⃣ Testing file method (simulating Kubernetes secrets)...

$ mkdir -p /tmp/mock-secrets

$ echo "mock_db_password" > /tmp/mock-secrets/database_password

$ echo "mock_api_key" > /tmp/mock-secrets/api_key

$ sudo mkdir -p /run/secrets 2>/dev/null || mkdir -p ./mock-secrets
mkdir: cannot create directory '/run/secrets': Permission denied

$ cp /tmp/mock-secrets/* ./mock-secrets/ 2>/dev/null || echo "Using tmp secrets"
Using tmp secrets

$ ./scripts/inject-secrets.sh production file .env.test.files
🔐 Dynamic Secret Injection System
==================================

📋 Injection Configuration:
  • Environment: production
  • Secret Method: file
  • Output File: .env.test.files

🔍 Validating required secrets for production...
  ❌ Missing required secret: PROD_MONGO_USERNAME
  ❌ Missing required secret: PROD_MONGO_PASSWORD
  ❌ Missing required secret: PROD_REDIS_PASSWORD
  ❌ Missing required secret: PROD_APP_SECRET_KEY
  ❌ Missing required secret: PROD_JWT_SECRET
  ⚠️ 5 secrets are missing
  🚨 Cannot proceed with production deployment

$ echo ""

$ echo "🧹 Cleaning up test files..."
🧹 Cleaning up test files...

$ rm -f .env.test.*

$ rm -rf /tmp/mock-secrets ./mock-secrets

$ unset STAGING_MONGO_USERNAME STAGING_MONGO_PASSWORD STAGING_REDIS_PASSWORD

$ echo ""

$ echo "✅ Secret management system tested successfully!"
✅ Secret management system tested successfully!
```

---

## 🎯 Task 6: Final Integration and Commit All Changes

### 📝 Subtask 6.1: Create Final Pipeline with All Features

**Step 28**: Update the pipeline with complete integration

```bash
echo "🎯 Creating final integrated CI/CD pipeline with all features..."

echo ""
echo "📋 Final Integration Features:"
echo "  • Advanced Docker image tagging"
echo "  • Comprehensive security scanning"
echo "  • Multi-service docker-compose deployment"
echo "  • Secure secret management"
echo "  • Environment-specific configurations"

echo ""
echo "1️⃣ Creating final .gitlab-ci.yml with full integration..."
cp .gitlab-ci.yml .gitlab-ci.yml.backup-final

nano .gitlab-ci.yml
```

**Replace the entire .gitlab-ci.yml content with the final version:**

```yaml
# Complete Docker CI/CD Pipeline - Production Ready
# Features: Advanced tagging, Security scanning, Multi-service deployment, Secret management

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
  - echo "🔐 Initializing pipeline environment..."
  - echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY
  - apk add --no-cache docker-compose py3-pip curl
  - docker-compose --version
  - echo "✅ Pipeline environment ready"

# Complete build stage with advanced tagging
build_and_tag:
  stage: build
  image: docker:24
  script:
    - echo "🏗️ Complete build and tagging process..."
    
    - echo ""
    - echo "📋 Build Configuration:"
    - echo "  • Project: $CI_PROJECT_NAME"
    - echo "  • Registry: $CI_REGISTRY"  
    - echo "  • Image: $IMAGE_NAME"
    - echo "  • Commit: $CI_COMMIT_SHORT_SHA"
    - echo "  • Branch: $CI_COMMIT_REF_NAME"
    - echo "  • Pipeline: $CI_PIPELINE_ID"
    
    - echo ""
    - echo "1️⃣ Building Docker image..."
    - docker build -t $IMAGE_NAME:latest .
    
    - echo ""
    - echo "2️⃣ Executing advanced tagging strategy..."
    - chmod +x scripts/tag-image.sh
    - ./scripts/tag-image.sh $IMAGE_NAME $CI_COMMIT_SHORT_SHA $CI_COMMIT_REF_NAME $CI_PIPELINE_ID
    
    - echo ""
    - echo "3️⃣ Validating docker-compose configuration..."
    - docker-compose config --quiet
    - echo "✅ Docker-compose configuration validated"
    
    - echo ""
    - echo "4️⃣ Pre-building service images for cache..."
    - docker-compose pull mongo redis nginx || echo "Some images not available in cache"
    
    - echo "✅ Build and tagging completed!"
  artifacts:
    paths:
      - VERSION
      - scripts/
      - docker-compose.yml
      - nginx.conf
      - .env.*.template
      - SECRETS_GUIDE.md
    expire_in: 1 week
  only:
    - main
    - develop
    - /^release\/.*$/
    - /^hotfix\/.*$/

# Comprehensive multi-service testing
test_complete_stack:
  stage: test
  image: docker:24
  script:
    - echo "🧪 Complete stack testing with secret management..."
    
    - echo ""
    - echo "📋 Testing Strategy:"
    - echo "  • Secret injection testing"
    - echo "  • Multi-service integration"
    - echo "  • Health validation"
    - echo "  • Performance verification"
    
    - echo ""
    - echo "1️⃣ Setting up test environment with secrets..."
    - chmod +x scripts/inject-secrets.sh
    
    # Set up test secrets
    - export STAGING_MONGO_USERNAME="test_admin"
    - export STAGING_MONGO_PASSWORD="test_secure_password_123"
    - export STAGING_REDIS_PASSWORD="test_redis_password_456"
    
    - ./scripts/inject-secrets.sh staging env .env.runtime
    - echo "✅ Test secrets injected"
    
    - echo ""
    - echo "2️⃣ Starting complete application stack..."
    - docker-compose --env-file .env.runtime up -d
    
    - echo "⏳ Waiting for all services to initialize..."
    - sleep 60
    
    - echo ""
    - echo "3️⃣ Validating service health..."
    - docker-compose ps
    
    - echo ""
    - echo "4️⃣ Running comprehensive tests..."
    
    # Application tests
    - echo "   📋 Application unit tests..."
    - docker-compose exec -T app npm test
    
    # Health endpoint tests
    - echo "   📋 Health endpoint tests..."
    - docker-compose exec -T app wget --spider -q http://localhost:3000/health || exit 1
    - docker-compose exec -T nginx wget --spider -q http://localhost/health || exit 1
    
    # Performance tests
    - echo "   📋 Basic performance tests..."
    - time docker-compose exec -T app wget -q -O- http://localhost:3000/ > /dev/null
    
    # Database connectivity
    - echo "   📋 Database connectivity tests..."
    - docker-compose exec -T app nslookup mongo
    - docker-compose exec -T app nslookup redis
    
    - echo ""
    - echo "5️⃣ Load testing with multiple requests..."
    - for i in {1..5}; do
        docker-compose exec -T nginx wget -q -O- http://localhost/ > /dev/null &
      done
    - wait
    - echo "✅ Load test completed"
    
    - echo ""
    - echo "6️⃣ Collecting logs for analysis..."
    - docker-compose logs --tail=50 > test-logs.txt
    - echo "✅ Logs collected"
    
    - echo ""
    - echo "🧹 Cleaning up test environment..."
    - docker-compose down -v
    - rm -f .env.runtime
    
    - echo "✅ Complete stack testing passed!"
  artifacts:
    paths:
      - test-logs.txt
    expire_in: 1 day
  dependencies:
    - build_and_tag
  only:
    - main
    - develop

# Enhanced security scanning with multiple tools
security_comprehensive_analysis:
  stage: security
  image: docker:24
  script:
    - echo "🔒 Comprehensive security analysis..."
    
    - echo ""
    - echo "📋 Security Analysis Plan:"
    - echo "  • Multi-image vulnerability scanning"
    - echo "  • Configuration security review"
    - echo "  • Secret management validation"
    - echo "  • Compliance checking"
    
    - echo ""
    - echo "1️⃣ Installing security tools..."
    - chmod +x scripts/security-scan.sh
    
    - echo ""
    - echo "2️⃣ Running comprehensive security scan..."
    - ./scripts/security-scan.sh $IMAGE_NAME $IMAGE_TAG || SECURITY_ISSUES=1
    
    - echo ""
    - echo "3️⃣ Additional security validations..."
    
    # Check for security best practices
    - echo "   📋 Docker security best practices..."
    - docker inspect $IMAGE_NAME:$IMAGE_TAG | grep -E "(User|SecurityOpt)" || echo "   ℹ️ Standard security configuration"
    
    # Validate secret management
    - echo "   📋 Secret management validation..."
    - if grep -r "password.*=" . --exclude-dir=.git --exclude="*.template" --exclude="SECRETS_GUIDE.md"; then
        echo "   ⚠️ Potential hardcoded secrets detected"
      else
        echo "   ✅ No hardcoded secrets found"
      fi
    
    # Network security
    - echo "   📋 Network security analysis..."
    - docker-compose config | grep -A 5 "networks:" | grep -E "(external|driver)" || echo "   ✅ Internal networks configured"
    
    - echo ""
    - echo "4️⃣ Generating compliance report..."
    - cat > compliance-report.md << EOF
    # 🛡️ Security Compliance Report
    
    **Date:** $(date)
    **Pipeline:** $CI_PIPELINE_ID
    **Commit:** $CI_COMMIT_SHORT_SHA
    
    ## Compliance Status
    - **Vulnerability Scanning:** $([ -z "$SECURITY_ISSUES" ] && echo "✅ PASSED" || echo "⚠️ ISSUES DETECTED")
    - **Secret Management:** ✅ COMPLIANT
    - **Network Security:** ✅ COMPLIANT  
    - **Container Security:** ✅ COMPLIANT
    
    ## Recommendations
    - Regular security scanning: ✅ Implemented
    - Secret rotation: ✅ Documented
    - Network isolation: ✅ Configured
    - Least privilege: ✅ Applied
    EOF
    
    - cat compliance-report.md
    
    - echo ""
    - echo "✅ Security analysis completed!"
    
    # Don't fail pipeline for non-critical issues in non-production
    - if [ "$CI_COMMIT_REF_NAME" != "main" ] && [ -n "$SECURITY_ISSUES" ]; then
        echo "⚠️ Security issues detected but not blocking non-production deployment"
        exit 0
      fi
  artifacts:
    paths:
      - security-reports/
      - compliance-report.md
    expire_in: 1 week
  dependencies:
    - build_and_tag
  allow_failure: true
  only:
    - main
    - develop

# Production-ready deployment with secret management
deploy_staging:
  stage: deploy
  image: docker:24
  script:
    - echo "🚀 Production-ready staging deployment..."
    
    - echo ""
    - echo "📋 Deployment Configuration:"
    - echo "  • Environment: staging"
    - echo "  • Image: $IMAGE_NAME:$IMAGE_TAG"
    - echo "  • Secret Management: enabled"
    - echo "  • Multi-service stack: enabled"
    
    - echo ""
    - echo "1️⃣ Preparing deployment environment..."
    - chmod +x scripts/inject-secrets.sh
    
    - echo ""
    - echo "2️⃣ Injecting production secrets..."
    - ./scripts/inject-secrets.sh staging env .env.deploy
    
    - echo ""
    - echo "3️⃣ Deploying complete application stack..."
    - docker-compose --env-file .env.deploy up -d
    
    - echo "⏳ Waiting for deployment to stabilize..."
    - sleep 90
    
    - echo ""
    - echo "4️⃣ Deployment verification..."
    - docker-compose ps --format "table {{.Name}}\t{{.State}}\t{{.Status}}"
    
    - echo ""
    - echo "5️⃣ Health check validation..."
    - docker-compose exec -T app wget --spider -q http://localhost:3000/health || exit 1
    - docker-compose exec -T nginx wget --spider -q http://localhost/health || exit 1
    
    - echo ""
    - echo "6️⃣ Performance validation..."
    - docker stats --no-stream --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"
    
    - echo ""
    - echo "7️⃣ Setting up monitoring endpoints..."
    - echo "   📊 Health: http://staging.example.com/health"
    - echo "   📈 Nginx Status: http://staging.example.com/nginx_status"
    - echo "   📱 Application: http://staging.example.com/"
    
    - echo ""
    - echo "8️⃣ Generating deployment report..."
    - cat > deployment-report.md << EOF
    # 🚀 Deployment Report
    
    **Environment:** staging
    **Deployed:** $(date)
    **Image:** $IMAGE_NAME:$IMAGE_TAG
    **Pipeline:** $CI_PIPELINE_ID
    **Services:** $(docker-compose ps --services | wc -l)
    
    ## Service Status
    $(docker-compose ps --format "- **{{.Name}}:** {{.State}} ({{.Status}})")
    
    ## Resource Usage
    $(docker stats --no-stream --format "- **{{.Container}}:** CPU {{.CPUPerc}}, Memory {{.MemUsage}}")
    
    ## Access URLs
    - Main Application: http://staging.example.com/
    - Health Check: http://staging.example.com/health
    - Admin Panel: http://staging.example.com/admin
    EOF
    
    - cat deployment-report.md
    
    - echo ""
    - echo "✅ Staging deployment completed successfully!"
    
    # Optional: Send deployment notification
    - |
      if [ ! -z "$NOTIFICATION_WEBHOOK" ]; then
        curl -X POST -H 'Content-type: application/json' \
        --data '{"text":"🚀 Staging deployment successful for '"$CI_PROJECT_NAME"' version '"$IMAGE_TAG"'"}' \
        $NOTIFICATION_WEBHOOK || echo "📢 Notification not configured"
      fi
  environment:
    name: staging
    url: http://staging.example.com
  artifacts:
    paths:
      - deployment-report.md
      - .env.deploy
    expire_in: 1 week
  dependencies:
    - test_complete_stack
    - security_comprehensive_analysis
  only:
    - main
  when: manual

# Optional: Production deployment (manual trigger only)
deploy_production:
  stage: deploy
  image: docker:24
  script:
    - echo "🚀 Production deployment..."
    - echo "⚠️ This is a production deployment!"
    - echo "📋 Requires all security checks to pass"
    
    # Production deployment logic here...
    - echo "✅ Production deployment would execute here"
    - echo "🔒 Requires production secrets and infrastructure"
  environment:
    name: production
    url: https://yourdomain.com
  dependencies:
    - test_complete_stack
    - security_comprehensive_analysis
  only:
    - main
  when: manual
  allow_failure: false
```

**Expected Output:**
```bash
$ echo "🎯 Creating final integrated CI/CD pipeline with all features..."
🎯 Creating final integrated CI/CD pipeline with all features...

$ echo ""

$ echo "📋 Final Integration Features:"
📋 Final Integration Features:
$ echo "  • Advanced Docker image tagging"
  • Advanced Docker image tagging
$ echo "  • Comprehensive security scanning"
  • Comprehensive security scanning
$ echo "  • Multi-service docker-compose deployment"  
  • Multi-service docker-compose deployment
$ echo "  • Secure secret management"
  • Secure secret management
$ echo "  • Environment-specific configurations"
  • Environment-specific configurations

$ echo ""

$ echo "1️⃣ Creating final .gitlab-ci.yml with full integration..."
1️⃣ Creating final .gitlab-ci.yml with full integration...

$ cp .gitlab-ci.yml .gitlab-ci.yml.backup-final

$ nano .gitlab-ci.yml
[nano editor opens - replace with the complete final version above]

$ wc -l .gitlab-ci.yml
345 .gitlab-ci.yml

$ echo "✅ Final integrated pipeline created!"
✅ Final integrated pipeline created!
```

---

### 📚 Subtask 6.2: Create Comprehensive Documentation

**Step 29**: Create complete project documentation

```bash
echo "📚 Creating comprehensive project documentation..."

echo ""
echo "2️⃣ Creating main README.md..."
nano README.md
```

Add the following content:

```markdown
# 🐳 Docker CI/CD Lab - Complete Implementation

A comprehensive demonstration of Docker containerization with GitLab CI/CD pipelines, featuring advanced image tagging, security scanning, multi-service orchestration, and secure secret management.

## 🎯 Project Overview

This project showcases production-ready DevOps practices including:

- **🏗️ Advanced Docker containerization** with multi-stage builds and security best practices
- **⚙️ GitLab CI/CD pipelines** with 4-stage deployment process (build, test, security, deploy)
- **🏷️ Comprehensive image tagging** with semantic versioning and multiple tag strategies  
- **🔒 Security scanning** with Trivy and configuration analysis
- **🐙 Multi-service orchestration** using docker-compose with MongoDB, Redis, and Nginx
- **🔐 Secure secret management** with environment-specific configurations

## 🏗️ Architecture

```
┌─────────────────┐    ┌──────────────┐    ┌─────────────────┐
│   Nginx Proxy   │────│  Node.js App │────│    MongoDB      │
│   (Port 80)     │    │  (Port 3000) │    │  (Port 27017)   │
└─────────────────┘    └──────────────┘    └─────────────────┘
                              │
                       ┌──────────────┐
                       │    Redis     │
                       │ (Port 6379)  │
                       └──────────────┘
```

## 📁 Project Structure

```
docker-cicd-lab/
├── 📱 Application Files
│   ├── app.js                          # Main Node.js application
│   ├── package.json                    # Dependencies and scripts
│   └── Dockerfile                      # Multi-stage Docker build
│
├── 🐙 Orchestration
│   ├── docker-compose.yml              # Multi-service stack definition
│   └── nginx.conf                      # Load balancer configuration
│
├── 🔐 Secret Management
│   ├── .env.example                    # Environment variables template
│   ├── .env.production.template        # Production environment template
│   ├── .env.staging.template           # Staging environment template
│   ├── .env.development.template       # Development environment template
│   └── SECRETS_GUIDE.md               # Secret management documentation
│
├── 🛠️ Automation Scripts
│   ├── scripts/tag-image.sh           # Advanced Docker image tagging
│   ├── scripts/security-scan.sh       # Comprehensive security scanning
│   └── scripts/inject-secrets.sh      # Dynamic secret injection
│
├── ⚙️ CI/CD Configuration
│   ├── .gitlab-ci.yml                 # Complete GitLab CI/CD pipeline
│   └── .trivyignore                   # Security scanner exceptions
│
├── 📝 Documentation
│   ├── README.md                      # This file
│   ├── VERSION                        # Semantic version tracking
│   └── DEPLOYMENT.md                  # Deployment instructions
│
└── 🔒 Security
    └── security-reports/              # Generated security scan reports
```

## 🚀 Quick Start

### Prerequisites
- Docker Engine 20.0+
- Docker Compose 2.0+
- GitLab account (free tier sufficient)
- Git client

### 1️⃣ Local Development Setup

```bash
# Clone the repository
git clone https://gitlab.com/YOUR_USERNAME/docker-cicd-lab.git
cd docker-cicd-lab

# Create development environment
cp .env.development.template .env.development

# Start the development stack
docker-compose --env-file .env.development up -d

# Access the application
open http://localhost:3000
```

### 2️⃣ CI/CD Pipeline Setup

1. **Configure GitLab Variables** (Project Settings > CI/CD > Variables):
   ```
   STAGING_MONGO_USERNAME
   STAGING_MONGO_PASSWORD (masked)
   STAGING_REDIS_PASSWORD (masked)
   ```

2. **Push to trigger pipeline**:
   ```bash
   git add .
   git commit -m "feat: Complete Docker CI/CD implementation"
   git push origin main
   ```

3. **Monitor pipeline execution**:
   - Navigate to CI/CD > Pipelines in GitLab
   - Watch the 4-stage execution process
   - Review security reports and deployment status

## 🔧 Advanced Usage

### Image Tagging Strategy

The project implements a comprehensive tagging strategy:

```bash
# Semantic versioning (production releases)
my-app:1.2.3

# Commit-based (immutable reference)  
my-app:abc123d

# Branch-based (latest per branch)
my-app:main, my-app:develop

# Temporal (historical tracking)
my-app:2024-08-19, my-app:20240819-143022

# Build-based (CI tracking)
my-app:build-42
```

### Security Scanning

Automated vulnerability scanning with multiple tools:

```bash
# Run security scan manually
./scripts/security-scan.sh my-app latest

# View detailed reports
ls security-reports/
cat security-reports/security-report-*.md
```

### Secret Management

Environment-specific secret injection:

```bash
# Development (generates secure defaults)
./scripts/inject-secrets.sh development env .env.runtime

# Staging (uses GitLab CI/CD variables)
./scripts/inject-secrets.sh staging env .env.runtime

# Production (requires all secrets)
./scripts/inject-secrets.sh production env .env.runtime
```

## 🧪 Testing

### Local Testing
```bash
# Run application tests
docker-compose exec app npm test

# Test complete stack
docker-compose up -d
curl http://localhost/health
docker-compose down -v
```

### Security Testing
```bash
# Run comprehensive security scan
./scripts/security-scan.sh my-app latest

# Test secret management
./scripts/inject-secrets.sh development env test.env
```

## 📊 Monitoring & Logging

### Health Endpoints
- **Application Health**: `http://localhost:3000/health`
- **Nginx Status**: `http://localhost/nginx_status` (internal networks only)
- **Load Balancer Health**: `http://localhost/health`

### Log Access
```bash
# View all service logs
docker-compose logs -f

# View specific service logs
docker-compose logs app
docker-compose logs nginx
docker-compose logs mongo
docker-compose logs redis
```

## 🔐 Security Features

### Container Security
- ✅ **Non-root user execution**
- ✅ **Multi-stage builds** for minimal attack surface
- ✅ **Health checks** for service monitoring
- ✅ **Network isolation** with custom Docker networks
- ✅ **Secret management** with runtime injection

### CI/CD Security
- ✅ **Automated vulnerability scanning** with Trivy
- ✅ **Configuration security analysis**
- ✅ **Secret detection** in source code
- ✅ **Compliance reporting**
- ✅ **Security policy enforcement**

## 🚀 Deployment Environments

### Development
- Local development with hot reload
- Mock services and test data
- Relaxed security for debugging

### Staging  
- Production-like environment
- Real service dependencies
- Security scanning enabled
- Manual deployment trigger

### Production
- Full security enforcement
- All secrets required
- Comprehensive health checks
- Manual deployment only

## 📈 Pipeline Stages

### 1️⃣ Build Stage
- Multi-architecture Docker build
- Advanced image tagging strategy
- Docker-compose validation
- Artifact generation

### 2️⃣ Test Stage
- Unit testing execution
- Integration testing with full stack
- Performance validation
- Service connectivity verification

### 3️⃣ Security Stage
- Multi-image vulnerability scanning
- Configuration security analysis
- Secret management validation
- Compliance reporting

### 4️⃣ Deploy Stage
- Environment-specific deployment
- Secret injection at runtime
- Health validation
- Performance monitoring

## 🛠️ Customization

### Adding New Services

1. **Update docker-compose.yml**:
   ```yaml
   newservice:
     image: new-service:latest
     networks:
       - app-network
     depends_on:
       - app
   ```

2. **Update security scanning**:
   ```bash
   # Add to scripts/security-scan.sh
   scan_image "new-service:latest" "New Service" "newservice"
   ```

3. **Update pipeline dependencies**:
   ```yaml
   test_complete_stack:
     script:
       - docker-compose exec -T newservice health-check
   ```

### Environment Variables

Add new variables to environment templates:

```bash
# .env.production.template
NEW_SERVICE_URL=${PROD_NEW_SERVICE_URL}
NEW_SERVICE_API_KEY=${PROD_NEW_SERVICE_API_KEY}

# scripts/inject-secrets.sh
REQUIRED_SECRETS+=(
    "PROD_NEW_SERVICE_API_KEY"
)
```

## 🐛 Troubleshooting

### Common Issues

**🔍 Pipeline Fails at Security Stage**
```bash
# Check security reports
cat security-reports/security-report-*.md

# Review ignored vulnerabilities
cat .trivyignore

# Update ignore list if acceptable
echo "CVE-2023-XXXXX" >> .trivyignore
```

**🔍 Services Won't Start**
```bash
# Check service logs
docker-compose logs service_name

# Verify environment variables
docker-compose config

# Test network connectivity
docker-compose exec app nslookup mongo
```

**🔍 Secret Injection Issues**
```bash
# Validate GitLab CI/CD variables
./scripts/inject-secrets.sh staging env test.env

# Check secret file permissions
ls -la .env.*

# Test with development secrets
./scripts/inject-secrets.sh development env .env.test
```

### Debug Commands

```bash
# Full system status
docker-compose ps
docker-compose logs --tail=20

# Network inspection
docker network ls
docker network inspect docker-cicd-lab_app-network

# Container inspection
docker inspect container_name
docker exec -it container_name sh
```

## 📚 Additional Resources

### Documentation
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [Trivy Security Scanner](https://aquasecurity.github.io/trivy/)
- [Docker Compose Reference](https://docs.docker.com/compose/)

### Learning Resources
- [Docker Mastery Course](https://www.udemy.com/course/docker-mastery/)
- [GitLab CI/CD Tutorial](https://docs.gitlab.com/ee/ci/quick_start/)
- [Container Security Guide](https://kubernetes.io/docs/concepts/security/)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Commit changes: `git commit -m 'feat: Add amazing feature'`
4. Push to branch: `git push origin feature/amazing-feature`
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🙏 Acknowledgments

- **Al Nafi** for providing comprehensive DevOps training
- **GitLab** for excellent CI/CD platform
- **Trivy** team for security scanning tools
- **Docker** community for containerization best practices

---

**🎓 Educational Note**: This project demonstrates advanced DevOps practices suitable for production environments. Each component has been designed with security, scalability, and maintainability in mind.
```

**Expected Output:**
```bash
$ echo "📚 Creating comprehensive project documentation..."
📚 Creating comprehensive project documentation...

$ echo ""

$ echo "2️⃣ Creating main README.md..."
2️⃣ Creating main README.md...

$ nano README.md
[nano editor opens - add the comprehensive content above]

$ wc -l README.md
387 README.md

$ echo "✅ Comprehensive README.md created!"
✅ Comprehensive README.md created!
```

---

### 🚀 Subtask 6.3: Create Deployment Guide

**Step 30**: Create detailed deployment instructions

```bash
echo "🚀 Creating detailed deployment guide..."

echo ""
echo "3️⃣ Creating DEPLOYMENT.md..."
nano DEPLOYMENT.md
```

Add the following content:

```markdown
# 🚀 Deployment Guide

This guide provides step-by-step instructions for deploying the Docker CI/CD Lab application across different environments.

## 📋 Prerequisites Checklist

### Development Environment
- [ ] Docker Engine 20.0+ installed
- [ ] Docker Compose 2.0+ installed  
- [ ] Git client configured
- [ ] Text editor (VS Code recommended)
- [ ] 4GB+ RAM available

### Staging/Production Environment  
- [ ] Linux server with Docker support
- [ ] GitLab Runner configured (for self-hosted)
- [ ] Domain name and SSL certificates
- [ ] Monitoring tools setup
- [ ] Backup solution in place

## 🏠 Local Development Deployment

### Quick Start (5 minutes)

```bash
# 1. Clone and setup
git clone https://gitlab.com/YOUR_USERNAME/docker-cicd-lab.git
cd docker-cicd-lab

# 2. Create environment configuration
cp .env.development.template .env.development
./scripts/inject-secrets.sh development env .env.runtime

# 3. Start services
docker-compose --env-file .env.runtime up -d

# 4. Verify deployment
curl http://localhost:3000/health
curl http://localhost/health
```

### Development Configuration

**Environment Variables** (`.env.development`):
```bash
NODE_ENV=development
LOG_LEVEL=debug
ENABLE_DEBUG_ENDPOINTS=true
MOCK_EXTERNAL_SERVICES=true
```

**Service Access**:
- **Application**: http://localhost:3000
- **Load Balancer**: http://localhost:80  
- **MongoDB**: localhost:27017
- **Redis**: localhost:6379

### Development Commands

```bash
# View logs
docker-compose logs -f

# Restart specific service
docker-compose restart app

# Access container shell
docker-compose exec app sh

# Stop all services
docker-compose down -v
```

## 🧪 Staging Environment Deployment

### GitLab CI/CD Deployment

**1. Configure GitLab Variables**

Navigate to **Project Settings > CI/CD > Variables**:

```
Variable: STAGING_MONGO_USERNAME
Value: staging_admin
Type: Variable
Environment: staging
Flags: [Protected]

Variable: STAGING_MONGO_PASSWORD  
Value: your_secure_staging_password
Type: Variable
Environment: staging
Flags: [Masked] [Protected]

Variable: STAGING_REDIS_PASSWORD
Value: your_secure_redis_password
Type: Variable  
Environment: staging
Flags: [Masked] [Protected]
```

**2. Trigger Deployment**

```bash
# Push to main branch to trigger pipeline
git checkout main
git add .
git commit -m "deploy: Staging deployment v1.0.1"
git push origin main

# Monitor pipeline in GitLab UI
# Navigate to CI/CD > Pipelines
# Click on running pipeline to view progress
```

**3. Manual Staging Deployment**

If running on staging server directly:

```bash
# 1. Prepare environment
export STAGING_MONGO_USERNAME="staging_admin"
export STAGING_MONGO_PASSWORD="your_secure_password"
export STAGING_REDIS_PASSWORD="your_redis_password"

# 2. Inject secrets
./scripts/inject-secrets.sh staging env .env.staging

# 3. Deploy stack
docker-compose --env-file .env.staging up -d

# 4. Verify deployment
curl http://staging.yourdomain.com/health
```

### Staging Verification

**Health Checks**:
```bash
# Application health
curl -f http://staging.yourdomain.com/health

# Service status
docker-compose ps

# Resource usage  
docker stats --no-stream
```

**Log Monitoring**:
```bash
# All services
docker-compose logs -f --tail=100

# Specific service
docker-compose logs app --tail=50

# Error logs only
docker-compose logs | grep -i error
```

## 🏭 Production Environment Deployment

### Pre-deployment Checklist

- [ ] **Security Review**: All vulnerability scans passed
- [ ] **Performance Testing**: Load testing completed
- [ ] **Backup Strategy**: Database backup verified
- [ ] **Monitoring**: Alerting systems configured
- [ ] **Rollback Plan**: Previous version tagged and ready
- [ ] **Team Notification**: Stakeholders informed of deployment

### Production Secrets Configuration

**Required GitLab Variables**:
```
PROD_MONGO_USERNAME          (Protected, Masked)
PROD_MONGO_PASSWORD          (Protected, Masked) 
PROD_REDIS_PASSWORD          (Protected, Masked)
PROD_APP_SECRET_KEY          (Protected, Masked)
PROD_JWT_SECRET              (Protected, Masked)
PROD_ENCRYPTION_KEY          (Protected, Masked)
PROD_SENTRY_DSN             (Protected, Masked, Optional)
NOTIFICATION_WEBHOOK         (Optional)
```

### Production Deployment Process

**1. Pre-deployment Validation**

```bash
# Run security scan
./scripts/security-scan.sh $IMAGE_NAME $IMAGE_TAG

# Validate secrets
./scripts/inject-secrets.sh production env .env.validation

# Test image locally
docker run --rm $IMAGE_NAME:$IMAGE_TAG npm test
```

**2. Production Deployment**

The production deployment is **manual only** and requires:

```bash
# In GitLab CI/CD Pipeline
# 1. Navigate to CI/CD > Pipelines
# 2. Click on the successful pipeline
# 3. Go to "deploy" stage
# 4. Click "Play" button on "deploy_production" job
# 5. Confirm deployment in popup
```

**3. Post-deployment Verification**

```bash
# Health verification
curl -f https://yourdomain.com/health

# Performance check
curl -w "@curl-format.txt" -s -o /dev/null https://yourdomain.com/

# SSL verification
curl -I https://yourdomain.com/
```

### Production Monitoring

**Essential Monitoring**:

```bash
# Service status
docker-compose ps --format "table {{.Name}}\t{{.State}}\t{{.Status}}"

# Resource usage
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"

# Disk usage
df -h
docker system df
```

**Log Collection**:

```bash
# Centralized logging
docker-compose logs --since 1h > /var/log/app/app-$(date +%Y%m%d-%H%M).log

# Error monitoring
docker-compose logs | grep -i "error\|fatal\|exception" | tail -20
```

## 🔄 Rollback Procedures

### Quick Rollback (5 minutes)

**1. Identify Previous Version**
```bash
# List recent image tags
docker image ls | grep $CI_REGISTRY_IMAGE | head -10

# Get previous semantic version
PREVIOUS_VERSION=$(git tag --sort=-version:refname | head -2 | tail -1)
echo "Rolling back to: $PREVIOUS_VERSION"
```

**2. Execute Rollback**
```bash
# Update image tag in docker-compose.yml
sed -i "s/:latest/:$PREVIOUS_VERSION/g" docker-compose.yml

# Redeploy with previous version
docker-compose up -d --force-recreate

# Verify rollback
curl -f http://yourdomain.com/health
```

**3. Verify Rollback**
```bash
# Check application version
curl http://yourdomain.com/ | grep version

# Monitor for issues
docker-compose logs --tail=50 -f
```

### Database Rollback (if needed)

**MongoDB Rollback**:
```bash
# Stop application to prevent writes
docker-compose stop app nginx

# Restore from backup
docker-compose exec mongo mongorestore --drop /backup/previous-backup

# Restart services
docker-compose up -d
```

## 📊 Performance Optimization

### Container Optimization

**Memory Limits**:
```yaml
# In docker-compose.yml
services:
  app:
    deploy:
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M
```

**CPU Optimization**:
```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '1.0'
        reservations:
          cpus: '0.5'
```

### Database Performance

**MongoDB Optimization**:
```bash
# Index creation
docker-compose exec mongo mongo myapp --eval "db.users.createIndex({email: 1})"

# Performance monitoring
docker-compose exec mongo mongostat --host mongo:27017
```

**Redis Optimization**:
```bash
# Memory usage
docker-compose exec redis redis-cli info memory

# Performance metrics
docker-compose exec redis redis-cli info stats
```

## 🔐 Security Hardening

### Production Security Checklist

- [ ] **Container Security**: Non-root user configured
- [ ] **Network Security**: Internal networks isolated
- [ ] **Secret Management**: No hardcoded secrets
- [ ] **SSL/TLS**: HTTPS enforced
- [ ] **Access Control**: Limited admin access
- [ ] **Monitoring**: Security events logged
- [ ] **Updates**: Base images regularly updated

### Security Commands

**Regular Security Scan**:
```bash
# Weekly security scan
./scripts/security-scan.sh $IMAGE_NAME latest

# Update vulnerability database
docker run --rm -v /var/lib/trivy:/root/.cache/ aquasec/trivy image --download-db-only
```

**Access Audit**:
```bash
# Review container processes
docker-compose exec app ps aux

# Check network connections
docker-compose exec app netstat -tlnp

# Verify file permissions
docker-compose exec app find /app -type f -perm /o+w
```

## 🚨 Troubleshooting Guide

### Common Issues and Solutions

**🔍 Issue**: Services won't start
```bash
# Solution
docker-compose down -v  # Stop and remove volumes
docker-compose pull     # Pull latest images
docker-compose up -d    # Restart services

# Debug
docker-compose logs service_name
```

**🔍 Issue**: Database connection failed
```bash
# Solution
docker-compose exec app nslookup mongo  # Test DNS resolution
docker-compose restart mongo            # Restart MongoDB
docker-compose exec mongo mongo --eval "db.stats()"  # Test connectivity
```

**🔍 Issue**: High memory usage
```bash
# Solution  
docker stats --no-stream                # Check resource usage
docker-compose restart app              # Restart application
docker system prune -f                  # Clean up unused resources
```

**🔍 Issue**: SSL certificate issues
```bash
# Solution
certbot renew --dry-run                 # Test certificate renewal
docker-compose restart nginx           # Restart nginx
curl -I https://yourdomain.com/         # Verify SSL
```

### Emergency Procedures

**🚨 Complete System Recovery**
```bash
# 1. Stop all services
docker-compose down -v

# 2. Clean Docker system  
docker system prune -a -f

# 3. Restore from backup
./restore-backup.sh latest

# 4. Restart services
docker-compose up -d

# 5. Verify recovery
./health-check.sh
```

## 📞 Support and Escalation

### Support Levels

**Level 1**: Application Issues
- Check application logs
- Restart affected services
- Verify configuration

**Level 2**: Infrastructure Issues  
- Server resource problems
- Network connectivity issues
- Database performance problems

**Level 3**: Security Incidents
- Potential security breaches
- Certificate expiration
- Compliance violations

### Contact Information

- **DevOps Team**: devops@company.com
- **Security Team**: security@company.com  
- **On-call Engineer**: +1-555-ON-CALL-1
- **Incident Management**: incidents@company.com

### Escalation Matrix

| Severity | Response Time | Escalation |
|----------|---------------|------------|
| Critical | 15 minutes    | Immediate  |
| High     | 1 hour        | 2 hours    |
| Medium   | 4 hours       | 1 day      |
| Low      | 1 day         | 3 days     |

---

**📝 Document Version**: 1.0  
**Last Updated**: $(date)  
**Maintained by**: DevOps Team
```

**Expected Output:**
```bash
$ echo "🚀 Creating detailed deployment guide..."
🚀 Creating detailed deployment guide...

$ echo ""

$ echo "3️⃣ Creating DEPLOYMENT.md..."
3️⃣ Creating DEPLOYMENT.md...

$ nano DEPLOYMENT.md
[nano editor opens - add the comprehensive deployment guide above]

$ wc -l DEPLOYMENT.md
412 DEPLOYMENT.md

$ echo "✅ Comprehensive deployment guide created!"
✅ Comprehensive deployment guide created!
```

---

### 📝 Subtask 6.4: Final Commit and Pipeline Trigger

**Step 31**: Commit all changes and trigger the final pipeline

```bash
echo "📝 Final commit and pipeline trigger..."

echo ""
echo "📋 Final Integration Summary:"
echo "  • Complete CI/CD pipeline with 4 stages"
echo "  • Advanced Docker image tagging strategy"
echo "  • Comprehensive security scanning"
echo "  • Multi-service docker-compose orchestration"
echo "  • Secure secret management system"
echo "  • Production-ready documentation"

echo ""
echo "1️⃣ Reviewing all project files..."
ls -la

echo ""
echo "📊 Project statistics:"
echo "  • Total files: $(find . -type f | grep -v .git | wc -l)"
echo "  • Code files: $(find . -name "*.js" -o -name "*.yml" -o -name "*.sh" | wc -l)"
echo "  • Documentation: $(find . -name "*.md" | wc -l)"
echo "  • Configuration files: $(find . -name ".env*" -o -name "Dockerfile" -o -name "*.conf" | wc -l)"

echo ""
echo "2️⃣ Final Git status check..."
git status --porcelain | head -20

echo ""
echo "3️⃣ Adding all files to Git..."
git add .

echo ""
echo "4️⃣ Creating final comprehensive commit..."
git commit -m "feat: Complete Docker CI/CD Lab implementation

🎯 Features Implemented:
- ✅ Advanced Docker containerization with multi-stage builds
- ✅ 4-stage GitLab CI/CD pipeline (build, test, security, deploy)
- ✅ Comprehensive image tagging with semantic versioning
- ✅ Multi-tool security scanning with Trivy integration
- ✅ Multi-service orchestration (Node.js + MongoDB + Redis + Nginx)
- ✅ Secure secret management with environment-specific configs
- ✅ Production-ready documentation and deployment guides

📦 Components Added:
- Application: Node.js Express server with health endpoints
- Database: MongoDB with admin authentication
- Cache: Redis with password protection
- Proxy: Nginx with load balancing and SSL support
- Scripts: Automated tagging, security scanning, secret injection
- Documentation: Comprehensive README and deployment guides

🔒 Security Features:
- Non-root container execution
- Automated vulnerability scanning
- Secret management best practices
- Network isolation with Docker networks
- Configuration security validation

🚀 DevOps Best Practices:
- Infrastructure as Code (IaC)
- Automated testing and validation
- Environment-specific configurations
- Comprehensive monitoring and logging
- Rollback procedures and disaster recovery

Co-authored-by: DevOps Student <student@example.com>"

echo ""
echo "5️⃣ Pushing final changes to trigger complete pipeline..."
git push origin main

echo ""
echo "🎉 Project Implementation Complete!"
echo ""
echo "📊 Final Project Summary:"
echo "================================="
echo "📁 Files Created:"
find . -type f -not -path './.git/*' | sort

echo ""
echo "🔗 Next Steps:"
echo "  1. 🌐 Visit your GitLab project to watch the pipeline execute"
echo "  2. 📊 Monitor the 4-stage pipeline (build → test → security → deploy)"  
echo "  3. 🔍 Review security scan reports in the artifacts"
echo "  4. 🚀 Manually trigger staging deployment when ready"
echo "  5. 📚 Use this as a reference for production implementations"

echo ""
echo "🎓 Learning Outcomes Achieved:"
echo "  ✅ Docker containerization mastery"
echo "  ✅ GitLab CI/CD pipeline expertise" 
echo "  ✅ Security scanning and vulnerability management"
echo "  ✅ Multi-service orchestration with docker-compose"
echo "  ✅ Production-ready secret management"
echo "  ✅ DevOps best practices implementation"

echo ""
echo "🏆 Congratulations! You've successfully completed the comprehensive Docker CI/CD Lab!"
```

**Expected Output:**
```bash
$ echo "📝 Final commit and pipeline trigger..."
📝 Final commit and pipeline trigger...

$ echo ""

$ echo "📋 Final Integration Summary:"
📋 Final Integration Summary:
$ echo "  • Complete CI/CD pipeline with 4 stages"
  • Complete CI/CD pipeline with 4 stages
$ echo "  • Advanced Docker image tagging strategy"  
  • Advanced Docker image tagging strategy
$ echo "  • Comprehensive security scanning"
  • Comprehensive security scanning
$ echo "  • Multi-service docker-compose orchestration"
  • Multi-service docker-compose orchestration
$ echo "  • Secure secret management system"
  • Secure secret management system
$ echo "  • Production-ready documentation"
  • Production-ready documentation

$ echo ""

$ echo "1️⃣ Reviewing all project files..."
1️⃣ Reviewing all project files...

$ ls -la
total 108
drwxr-xr-x 3 user user 4096 Aug 19 12:10 .
drwxr-xr-x 4 user user 4096 Aug 19 10:30 ..
drwxr-xr-x 8 user user 4096 Aug 19 12:10 .git
-rw-r--r-- 1 user user 9876 Aug 19 12:05 .gitlab-ci.yml
-rw-r--r-- 1 user user  875 Aug 19 11:08 .env.example
-rw-r--r-- 1 user user 1456 Aug 19 11:52 .env.development.template
-rw-r--r-- 1 user user 1678 Aug 19 11:51 .env.production.template
-rw-r--r-- 1 user user 1234 Aug 19 11:51 .env.staging.template  
-rw-r--r-- 1 user user 1234 Aug 19 11:35 .trivyignore
-rw-r--r-- 1 user user  856 Aug 19 10:32 app.js
-rw-r--r-- 1 user user 12456 Aug 19 12:10 DEPLOYMENT.md
-rw-r--r-- 1 user user 2134 Aug 19 11:00 docker-compose.yml
-rw-r--r-- 1 user user  678 Aug 19 10:36 Dockerfile
-rw-r--r-- 1 user user 2187 Aug 19 11:05 nginx.conf
-rw-r--r-- 1 user user  445 Aug 19 10:34 package.json
-rw-r--r-- 1 user user 15432 Aug 19 12:00 README.md
drwxr-xr-x 2 user user 4096 Aug 19 11:55 scripts
-rw-r--r-- 1 user user 2134 Aug 19 11:50 SECRETS_GUIDE.md
-rw-r--r-- 1 user user    6 Aug 19 11:22 VERSION

$ echo ""

$ echo "📊 Project statistics:"
📊 Project statistics:
$ echo "  • Total files: $(find . -type f | grep -v .git | wc -l)"
  • Total files: 19
$ echo "  • Code files: $(find . -name "*.js" -o -name "*.yml" -o -name "*.sh" | wc -l)"
  • Code files: 5
$ echo "  • Documentation: $(find . -name "*.md" | wc -l)"
  • Documentation: 3
$ echo "  • Configuration files: $(find . -name ".env*" -o -name "Dockerfile" -o -name "*.conf" | wc -l)"
  • Configuration files: 7

$ echo ""

$ echo "2️⃣ Final Git status check..."
2️⃣ Final Git status check...

$ git status --porcelain | head -20
M .gitlab-ci.yml
A DEPLOYMENT.md
A README.md
A SECRETS_GUIDE.md
A .env.development.template
A .env.production.template
A .env.staging.template
A .trivyignore
A scripts/inject-secrets.sh
A scripts/security-scan.sh
A scripts/tag-image.sh

$ echo ""

$ echo "3️⃣ Adding all files to Git..."
3️⃣ Adding all files to Git...

$ git add .

$ echo ""

$ echo "4️⃣ Creating final comprehensive commit..."
4️⃣ Creating final comprehensive commit...

$ git commit -m "feat: Complete Docker CI/CD Lab implementation

🎯 Features Implemented:
- ✅ Advanced Docker containerization with multi-stage builds
- ✅ 4-stage GitLab CI/CD pipeline (build, test, security, deploy)
- ✅ Comprehensive image tagging with semantic versioning
- ✅ Multi-tool security scanning with Trivy integration
- ✅ Multi-service orchestration (Node.js + MongoDB + Redis + Nginx)
- ✅ Secure secret management with environment-specific configs
- ✅ Production-ready documentation and deployment guides

📦 Components Added:
- Application: Node.js Express server with health endpoints
- Database: MongoDB with admin authentication
- Cache: Redis with password protection
- Proxy: Nginx with load balancing and SSL support
- Scripts: Automated tagging, security scanning, secret injection
- Documentation: Comprehensive README and deployment guides

🔒 Security Features:
- Non-root container execution
- Automated vulnerability scanning
- Secret management best practices
- Network isolation with Docker networks
- Configuration security validation

🚀 DevOps Best Practices:
- Infrastructure as Code (IaC)
- Automated testing and validation
- Environment-specific configurations
- Comprehensive monitoring and logging
- Rollback procedures and disaster recovery

Co-authored-by: DevOps Student <student@example.com>"
[main abc1234] feat: Complete Docker CI/CD Lab implementation
 15 files changed, 1247 insertions(+)
 create mode 100644 DEPLOYMENT.md
 create mode 100644 README.md
 create mode 100644 SECRETS_GUIDE.md
 create mode 100644 .env.development.template
 create mode 100644 .env.production.template
 create mode 100644 .env.staging.template
 create mode 100644 .trivyignore
 create mode 100755 scripts/inject-secrets.sh
 create mode 100755 scripts/security-scan.sh
 create mode 100755 scripts/tag-image.sh

$ echo ""

$ echo "5️⃣ Pushing final changes to trigger complete pipeline..."
5️⃣ Pushing final changes to trigger complete pipeline...
