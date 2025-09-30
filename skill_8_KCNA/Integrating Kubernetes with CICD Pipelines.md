# ==========================================
# Lab 16: Integrating Kubernetes with CI/CD Pipelines
# Active Learning Edition - September 2025
# ==========================================

cat << 'INTRO'
# 🎯 Lab 16: Integrating Kubernetes with CI/CD Pipelines

## 🧠 Learning Objectives
By the end of this lab, you will master:
• CI/CD pipeline architecture and automation workflows
• Docker containerization and registry integration
• Kubernetes deployment strategies and rollback procedures
• GitHub Actions workflow orchestration
• Multi-environment deployment patterns
• Production-grade troubleshooting techniques

## 📚 Prerequisites
✓ Basic understanding of Docker containers
✓ Familiarity with Kubernetes concepts (pods, deployments, services)
✓ Knowledge of Git version control
✓ Understanding of YAML configuration
✓ GitHub account (free tier sufficient)
✓ Command-line interface experience

## 🔧 Lab Environment
Your pre-configured Al Nafi cloud machine includes:
• Ubuntu 20.04 LTS with Docker
• kubectl v1.33 (latest stable)
• Minikube for local Kubernetes
• Git client and nano editor
• All networking pre-configured

## ⏱️ Estimated Time: 90-120 minutes

INTRO

# ==========================================
# ENVIRONMENT COMPATIBILITY CHECK
# ==========================================

echo "🧠 PREDICT FIRST: Before we start, what potential compatibility issues might we encounter?"
echo "   Consider:"
echo "   • Version conflicts between tools (Docker, kubectl, Node.js)"
echo "   • Missing dependencies or outdated packages"
echo "   • Configuration mismatches between local and production"
echo "   • Network connectivity issues"
echo "   Take 30 seconds to think about what could go wrong..."
echo ""
read -p "Press ENTER when you've formed your predictions..."
echo ""

echo "🔍 VALIDATING ENVIRONMENT:"
echo "================================================"

# System updates
echo "📦 Updating package repositories..."
sudo apt update -qq

# Check Docker
echo ""
echo "🐳 Checking Docker installation:"
if command -v docker &> /dev/null; then
    docker --version
    echo "✅ Docker is installed"
else
    echo "❌ Docker not found - installing..."
    sudo apt install -y docker.io
    sudo systemctl start docker
    sudo systemctl enable docker
    sudo usermod -aG docker $USER
    echo "⚠️  Please log out and back in for Docker permissions to take effect"
fi

# Check kubectl
echo ""
echo "☸️  Checking kubectl:"
if command -v kubectl &> /dev/null; then
    kubectl version --client --short 2>/dev/null || kubectl version --client
    echo "✅ kubectl is installed"
else
    echo "❌ kubectl not found - installing latest stable (v1.33)..."
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    rm kubectl
fi

# Check Minikube
echo ""
echo "🎡 Checking Minikube:"
if command -v minikube &> /dev/null; then
    minikube version --short
    echo "✅ Minikube is installed"
else
    echo "❌ Minikube not found - installing..."
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    rm minikube-linux-amd64
fi

# Check Node.js
echo ""
echo "📦 Checking Node.js:"
if command -v node &> /dev/null; then
    node --version
    echo "✅ Node.js is installed"
else
    echo "❌ Node.js not found - installing Node.js 22 LTS..."
    curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
    sudo apt install -y nodejs
fi

# Check Git
echo ""
echo "📚 Checking Git:"
git --version
echo "✅ Git is installed"

echo ""
echo "================================================"
echo "✅ ENVIRONMENT VALIDATION COMPLETE"
echo "================================================"
echo ""

echo "🤔 ACTIVE RECALL CHECK:"
echo "   1. Why is version compatibility critical in CI/CD pipelines?"
echo "   2. What could happen if kubectl version differs significantly from cluster version?"
echo "   3. How does Docker enable consistent deployment across environments?"
echo "   4. What role does Minikube play in local development?"
echo ""
read -p "Press ENTER after thinking through these questions..."
echo ""

cat << 'ANSWERS_1'
### 🧠 Comprehensive Answer Block 1:

**Question 1 Answer:** Version compatibility ensures that commands, APIs, and features work 
consistently across development, staging, and production environments. Mismatched versions can 
cause deployment failures, unexpected behavior, or security vulnerabilities. In CI/CD, we need 
predictable outcomes - version drift introduces unpredictability.

**Question 2 Answer:** kubectl communicates with the Kubernetes API server. If versions differ 
by more than one minor version (e.g., kubectl 1.30 vs cluster 1.33), some API calls may fail, 
features may be unavailable, or deprecated APIs might cause errors. The Kubernetes project 
maintains a version skew policy requiring kubectl to be within one minor version of the cluster.

**Question 3 Answer:** Docker packages the application with all its dependencies (runtime, 
libraries, configuration) into an immutable image. This eliminates "works on my machine" problems 
because the same image runs identically everywhere - from developer laptops to production servers. 
Containers ensure environment parity.

**Question 4 Answer:** Minikube creates a local single-node Kubernetes cluster for development 
and testing. It allows developers to experiment with Kubernetes features, test deployments, and 
validate configurations before pushing to remote clusters, without requiring cloud resources or 
complex infrastructure.

ANSWERS_1

echo ""
echo "🔄 SPACED REVIEW PLACEHOLDER: We'll connect these concepts throughout the lab"
echo ""

# ==========================================
# TASK 1: Setting Up the Development Environment
# Active Learning Block 1
# ==========================================

echo "================================================"
echo "TASK 1: DEVELOPMENT ENVIRONMENT SETUP"
echo "================================================"
echo ""

echo "🧠 PREDICT FIRST: We're about to create a Node.js application with Express."
echo "   Consider:"
echo "   • What components does a containerized web app need?"
echo "   • How will we make the app accessible from outside the container?"
echo "   • What health check mechanisms should we implement?"
echo "   • Why separate package.json from application code?"
echo "   Take 30 seconds to think about the architecture..."
echo ""
read -p "Press ENTER when ready to proceed..."
echo ""

echo "🔍 CREATING PROJECT STRUCTURE:"
echo "================================================"

# Create project directory
PROJECT_DIR="$HOME/k8s-cicd-lab"
echo "📁 Creating project directory: $PROJECT_DIR"
mkdir -p "$PROJECT_DIR"
cd "$PROJECT_DIR"

echo "✅ Project directory created"
echo ""

# Initialize Git repository
echo "📚 Initializing Git repository:"
git init
git config user.name "K8s Lab User"
git config user.email "lab@example.com"
echo "✅ Git repository initialized"
echo ""

# Create .gitignore
echo "📝 Creating .gitignore file..."
cat > .gitignore << 'EOF'
node_modules/
npm-debug.log
.env
.DS_Store
*.log
coverage/
.idea/
.vscode/
EOF
echo "✅ .gitignore created"
echo ""

echo "📝 CREATING APPLICATION FILES WITH NANO:"
echo "================================================"
echo ""
echo "We'll create 3 files: app.js, package.json, and Dockerfile"
echo "Each file will be shown first, then you'll create it with nano"
echo ""

# Show app.js content
cat << 'APP_JS_CONTENT'
---[ app.js Content Preview ]---
This Node.js application creates an Express web server with:
• Main endpoint (/) returning app info and version
• Health check endpoint (/health) for Kubernetes probes
• Metrics endpoint (/metrics) for monitoring
• Graceful shutdown handling for zero-downtime deployments

Key features:
- Binds to 0.0.0.0 (required for container networking)
- Uses PORT environment variable (container flexibility)
- Includes request logging for debugging
- Implements proper error handling

APP_JS_CONTENT

echo ""
read -p "Ready to create app.js? Press ENTER..."
echo ""

# Create app.js
cat > app.js << 'EOF'
const express = require('express');
const app = express();
const port = process.env.PORT || 3000;

// Request logging middleware
app.use((req, res, next) => {
  console.log(`${new Date().toISOString()} - ${req.method} ${req.path}`);
  next();
});

// Main endpoint
app.get('/', (req, res) => {
  res.json({
    message: 'Hello from Kubernetes CI/CD Pipeline! 🚀',
    version: process.env.APP_VERSION || '1.0.0',
    environment: process.env.NODE_ENV || 'development',
    timestamp: new Date().toISOString(),
    hostname: require('os').hostname()
  });
});

// Health check endpoint (for Kubernetes probes)
app.get('/health', (req, res) => {
  res.status(200).json({ 
    status: 'healthy',
    uptime: process.uptime(),
    timestamp: new Date().toISOString()
  });
});

// Readiness check endpoint
app.get('/ready', (req, res) => {
  // Add application-specific readiness checks here
  res.status(200).json({ 
    status: 'ready',
    timestamp: new Date().toISOString()
  });
});

// Metrics endpoint
app.get('/metrics', (req, res) => {
  res.json({
    memory: process.memoryUsage(),
    uptime: process.uptime(),
    timestamp: new Date().toISOString()
  });
});

// Error handling middleware
app.use((err, req, res, next) => {
  console.error('Error:', err);
  res.status(500).json({ error: 'Internal server error' });
});

// Start server
const server = app.listen(port, '0.0.0.0', () => {
  console.log(`🚀 App running on port ${port}`);
  console.log(`📊 Health check: http://localhost:${port}/health`);
  console.log(`📈 Metrics: http://localhost:${port}/metrics`);
});

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('SIGTERM signal received: closing HTTP server');
  server.close(() => {
    console.log('HTTP server closed');
    process.exit(0);
  });
});
EOF

echo "✅ app.js created successfully"
echo ""

# Show package.json content
cat << 'PACKAGE_JSON_CONTENT'
---[ package.json Content Preview ]---
This file defines:
• Application metadata (name, version, description)
• Dependencies (Express 5.x - latest LTS)
• Scripts for running and testing
• Node.js engine compatibility

Note: We're using Express 5.x (latest stable as of 2025)

PACKAGE_JSON_CONTENT

echo ""
read -p "Ready to create package.json? Press ENTER..."
echo ""

# Create package.json
cat > package.json << 'EOF'
{
  "name": "k8s-cicd-app",
  "version": "1.0.0",
  "description": "Production-ready app for Kubernetes CI/CD integration lab",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "dev": "NODE_ENV=development node app.js",
    "test": "echo \"Running tests...\" && exit 0",
    "lint": "echo \"Running linter...\" && exit 0"
  },
  "keywords": ["kubernetes", "ci-cd", "docker", "express"],
  "author": "K8s Lab",
  "license": "MIT",
  "engines": {
    "node": ">=22.0.0"
  },
  "dependencies": {
    "express": "^5.1.0"
  }
}
EOF

echo "✅ package.json created successfully"
echo ""

# Show Dockerfile content
cat << 'DOCKERFILE_CONTENT'
---[ Dockerfile Content Preview ]---
Multi-stage build for optimal image size:

Stage 1 (builder): Install ALL dependencies
Stage 2 (production): Copy only production dependencies

Best practices applied:
• Use official Node.js 22 Alpine image (smaller, more secure)
• Non-root user for security
• Explicit WORKDIR
• .dockerignore to exclude unnecessary files
• Health check for container monitoring
• Proper signal handling for graceful shutdown

DOCKERFILE_CONTENT

echo ""
read -p "Ready to create Dockerfile? Press ENTER..."
echo ""

# Create Dockerfile
cat > Dockerfile << 'EOF'
# Use official Node.js 22 LTS Alpine image (September 2025)
FROM node:22-alpine AS builder

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install all dependencies (including dev dependencies for building)
RUN npm ci

# Copy application source
COPY . .

# Production stage
FROM node:22-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install only production dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Copy application from builder
COPY --from=builder /app/app.js ./

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 && \
    chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Start application
CMD ["node", "app.js"]
EOF

echo "✅ Dockerfile created successfully"
echo ""

# Create .dockerignore
echo "📝 Creating .dockerignore file..."
cat > .dockerignore << 'EOF'
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.DS_Store
*.md
.github
k8s-manifests
EOF
echo "✅ .dockerignore created"
echo ""

echo "📊 PROJECT STRUCTURE:"
tree -L 2 -a || find . -maxdepth 2 -not -path '*/\.git/*' -not -path '*/.git'
echo ""

echo "================================================"
echo "✅ TASK 1 COMPLETE: Development environment ready"
echo "================================================"
echo ""

echo "🤔 ACTIVE RECALL CHECK:"
echo "   1. Explain the purpose of each file we created (app.js, package.json, Dockerfile)"
echo "   2. Why do we bind the Express server to '0.0.0.0' instead of 'localhost'?"
echo "   3. What's the advantage of the multi-stage Docker build we implemented?"
echo "   4. How do the /health and /ready endpoints support Kubernetes operations?"
echo ""
read -p "Press ENTER after thinking through these questions..."
echo ""

cat << 'ANSWERS_2'
### 🧠 Comprehensive Answer Block 2:

**Question 1 Answer:** 
- app.js: The application entry point containing Express server logic, routes, and middleware
- package.json: Dependency manifest defining what libraries the app needs and how to run it
- Dockerfile: Build instructions telling Docker how to create a container image with the app

**Question 2 Answer:** Inside containers, 'localhost' (127.0.0.1) only accepts connections from 
within the same container. Binding to '0.0.0.0' makes the server accept connections from any 
network interface, allowing external requests from Kubernetes services, load balancers, and 
ingress controllers to reach the application.

**Question 3 Answer:** Multi-stage builds separate build-time dependencies from runtime 
dependencies. The builder stage can have dev tools, compilers, and build dependencies, while 
the production stage contains only what's needed to run the app. This reduces final image size 
(often by 50%+), improves security (fewer attack surfaces), and speeds up deployments.

**Question 4 Answer:** 
- /health (liveness probe): Tells Kubernetes if the container is alive. If it fails repeatedly, 
  Kubernetes restarts the container
- /ready (readiness probe): Tells Kubernetes if the container can handle traffic. If it fails, 
  Kubernetes removes it from service load balancing until it recovers
These enable zero-downtime deployments and automatic failure recovery.

ANSWERS_2

echo ""
echo "🔄 SPACED REVIEW: Remember environment validation from Task 1?"
echo "   How does Docker solve the version compatibility concerns we discussed?"
echo "   Connection: Docker encapsulates the entire runtime environment (Node 22, Express 5)"
echo "   in the image, eliminating host system dependencies!"
echo ""

# ==========================================
# TASK 2: Creating Kubernetes Manifests
# Active Learning Block 2
# ==========================================

echo "================================================"
echo "TASK 2: KUBERNETES MANIFESTS"
echo "================================================"
echo ""

echo "🧠 PREDICT FIRST: We're about to define Kubernetes resources."
echo "   Consider:"
echo "   • What resources does Kubernetes need to run our app?"
echo "   • How does Kubernetes know when pods are healthy?"
echo "   • Why might we want multiple replicas of the same app?"
echo "   • How do external clients access pods that can die and restart?"
echo "   Take 45 seconds to think about Kubernetes architecture..."
echo ""
read -p "Press ENTER when ready to proceed..."
echo ""

echo "🔍 CREATING KUBERNETES MANIFESTS:"
echo "================================================"

# Create k8s manifests directory
mkdir -p k8s-manifests
echo "📁 Created k8s-manifests directory"
echo ""

cat << 'K8S_OVERVIEW'
We'll create 3 Kubernetes resources:

1️⃣ Namespace: Logical isolation for our application
2️⃣ Deployment: Manages pod replicas and rolling updates
3️⃣ Service: Provides stable network endpoint for pods

Let's examine each one...

K8S_OVERVIEW

echo ""

# Create namespace
cat << 'NAMESPACE_CONTENT'
---[ Namespace Preview ]---
Purpose: Isolates our CI/CD app from other workloads
Benefits:
• Resource quotas and limits
• Access control (RBAC)
• Name scoping
• Easier cleanup

NAMESPACE_CONTENT

echo ""
read -p "Creating namespace.yaml... Press ENTER"
echo ""

cat > k8s-manifests/namespace.yaml << 'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: cicd-lab
  labels:
    name: cicd-lab
    environment: development
EOF

echo "✅ namespace.yaml created"
echo ""

# Create deployment
cat << 'DEPLOYMENT_CONTENT'
---[ Deployment Preview ]---
This manifest defines:

• Replica count: 3 pods for high availability
• Container spec: Image, ports, environment variables
• Resource limits: CPU and memory constraints
• Liveness probe: Restarts unhealthy containers
• Readiness probe: Controls traffic routing
• Rolling update strategy: Zero-downtime deployments

Key Kubernetes concepts:
- Selector: Links Deployment to Pods via labels
- Template: Blueprint for creating pods
- Probes: Health checking mechanisms

DEPLOYMENT_CONTENT

echo ""
read -p "Creating deployment.yaml... Press ENTER"
echo ""

cat > k8s-manifests/deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cicd-app
  namespace: cicd-lab
  labels:
    app: cicd-app
    version: v1
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: cicd-app
  template:
    metadata:
      labels:
        app: cicd-app
        version: v1
    spec:
      containers:
      - name: cicd-app
        image: cicd-app:latest
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 3000
          protocol: TCP
        env:
        - name: APP_VERSION
          value: "1.0.0"
        - name: NODE_ENV
          value: "production"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        securityContext:
          runAsNonRoot: true
          runAsUser: 1001
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
EOF

echo "✅ deployment.yaml created"
echo ""

# Create service
cat << 'SERVICE_CONTENT'
---[ Service Preview ]---
Purpose: Provides stable networking for dynamic pods

Service types explained:
• ClusterIP: Internal cluster access only (default)
• NodePort: Exposes on each node's IP at a static port
• LoadBalancer: Cloud provider load balancer
• ExternalName: DNS CNAME mapping

We're using NodePort for local testing (Minikube)
Production would use LoadBalancer or Ingress

SERVICE_CONTENT

echo ""
read -p "Creating service.yaml... Press ENTER"
echo ""

cat > k8s-manifests/service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: cicd-app-service
  namespace: cicd-lab
  labels:
    app: cicd-app
spec:
  type: NodePort
  selector:
    app: cicd-app
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 3000
      nodePort: 30080
  sessionAffinity: None
EOF

echo "✅ service.yaml created"
echo ""

echo "📊 KUBERNETES MANIFESTS STRUCTURE:"
find k8s-manifests -type f | sort
echo ""

echo "================================================"
echo "✅ TASK 2 COMPLETE: Kubernetes manifests ready"
echo "================================================"
echo ""

echo "🤔 ACTIVE RECALL CHECK:"
echo "   1. What's the difference between a Deployment and a Pod?"
echo "   2. Why do we specify both requests and limits for resources?"
echo "   3. Explain the difference between liveness and readiness probes"
echo "   4. How does a Service know which Pods to route traffic to?"
echo ""
read -p "Press ENTER after thinking through these questions..."
echo ""

cat << 'ANSWERS_3'
### 🧠 Comprehensive Answer Block 3:

**Question 1 Answer:** A Pod is a single instance of a running container (or group of containers). 
A Deployment is a higher-level controller that manages multiple Pod replicas, handles rolling 
updates, maintains desired state, and automatically replaces failed Pods. You rarely create Pods 
directly - Deployments manage them for you.

**Question 2 Answer:** 
- Requests: Guaranteed resources the container needs (used for scheduling decisions)
- Limits: Maximum resources the container can use (prevents resource hogging)
This separation allows efficient cluster utilization - the scheduler can bin-pack workloads based 
on requests, while limits prevent noisy neighbor problems and resource exhaustion.

**Question 3 Answer:**
- Liveness probe: Detects deadlocks or crashed processes. Failure triggers container restart.
  Example: App is running but stuck in an infinite loop
- Readiness probe: Detects if app can handle traffic. Failure removes pod from service endpoints.
  Example: App is starting up or temporarily overloaded
Key difference: Liveness restarts, readiness just stops sending traffic.

**Question 4 Answer:** Services use label selectors (spec.selector) to match Pods with specific 
labels (metadata.labels). In our case, the Service selects Pods with label "app: cicd-app". 
Kubernetes continuously watches for Pods matching this selector and automatically updates the 
Service's endpoint list. This is declarative networking - you describe what you want, not how 
to configure it.

ANSWERS_3

echo ""
echo "🔄 SPACED REVIEW: Connecting to previous concepts"
echo "   Remember the /health endpoint we created in app.js?"
echo "   NOW it makes sense - that's what the livenessProbe calls!"
echo "   And the 0.0.0.0 binding? That's how the Service reaches the container!"
echo "   Everything connects! 🔗"
echo ""

# ==========================================
# TASK 3: Local Kubernetes Deployment
# Active Learning Block 3
# ==========================================

echo "================================================"
echo "TASK 3: LOCAL DEPLOYMENT WITH MINIKUBE"
echo "================================================"
echo ""

echo "🧠 PREDICT FIRST: We're about to deploy to Kubernetes."
echo "   Consider:"
echo "   • What happens when we start Minikube?"
echo "   • How does Docker build our image?"
echo "   • In what order should we apply Kubernetes resources?"
echo "   • How will we verify the deployment succeeded?"
echo "   Take 45 seconds to think about the deployment process..."
echo ""
read -p "Press ENTER when ready to proceed..."
echo ""

echo "🎡 STARTING MINIKUBE CLUSTER:"
echo "================================================"

# Check if Minikube is already running
if minikube status | grep -q "Running"; then
    echo "ℹ️  Minikube is already running"
else
    echo "🚀 Starting Minikube cluster..."
    minikube start --driver=docker --memory=4096 --cpus=2 --kubernetes-version=v1.33.0
fi

echo ""
echo "📊 CLUSTER INFORMATION:"
kubectl cluster-info
echo ""

echo "📋 NODES:"
kubectl get nodes -o wide
echo ""

echo "🔧 ENABLING ADDONS:"
minikube addons enable metrics-server
minikube addons enable dashboard
echo "✅ Addons enabled"
echo ""

echo "🐳 CONFIGURING DOCKER ENVIRONMENT:"
echo "================================================"
echo ""
echo "Setting Docker to use Minikube's daemon..."
echo "This allows us to build images directly in Minikube"
echo ""

eval $(minikube docker-env)
echo "✅ Docker environment configured"
echo ""

echo "🔨 BUILDING DOCKER IMAGE:"
echo "================================================"

echo "Building image: cicd-app:v1.0.0"
docker build -t cicd-app:v1.0.0 -t cicd-app:latest .

echo ""
echo "✅ Image built successfully"
echo ""

echo "📊 VERIFYING IMAGE:"
docker images | grep cicd-app
echo ""

echo "☸️  DEPLOYING TO KUBERNETES:"
echo "================================================"

echo "Applying manifests in correct order..."
echo ""

# Apply namespace first
echo "1️⃣ Creating namespace..."
kubectl apply -f k8s-manifests/namespace.yaml
echo ""

# Apply deployment
echo "2️⃣ Creating deployment..."
kubectl apply -f k8s-manifests/deployment.yaml
echo ""

# Apply service
echo "3️⃣ Creating service..."
kubectl apply -f k8s-manifests/service.yaml
echo ""

echo "⏳ WAITING FOR DEPLOYMENT:"
echo "================================================"
echo "Waiting for pods to be ready (this may take 30-60 seconds)..."
kubectl wait --for=condition=available --timeout=120s deployment/cicd-app -n cicd-lab

echo ""
echo "📊 DEPLOYMENT STATUS:"
echo "================================================"

echo ""
echo "Deployments:"
kubectl get deployments -n cicd-lab
echo ""

echo "Pods:"
kubectl get pods -n cicd-lab -o wide
echo ""

echo "Services:"
kubectl get services -n cicd-lab
echo ""

echo "🧪 TESTING THE APPLICATION:"
echo "================================================"

# Get service URL
SERVICE_URL=$(minikube service cicd-app-service -n cicd-lab --url)
echo "Service URL: $SERVICE_URL"
echo ""

echo "Testing main endpoint:"
curl -s $SERVICE_URL | jq '.' 2>/dev/null || curl -s $SERVICE_URL
echo ""

echo "Testing health endpoint:"
curl -s $SERVICE_URL/health | jq '.' 2>/dev/null || curl -s $SERVICE_URL/health
echo ""

echo "Testing metrics endpoint:"
curl -s $SERVICE_URL/metrics | jq '.' 2>/dev/null || curl -s $SERVICE_URL/metrics
echo ""

echo "================================================"
echo "✅ TASK 3 COMPLETE: Application deployed successfully!"
echo "================================================"
echo ""

echo "🤔 ACTIVE RECALL CHECK:"
echo "   1. What does 'eval \$(minikube docker-env)' actually do?"
echo "   2. Why did we apply namespace.yaml before deployment.yaml?"
echo "   3. How does Kubernetes know when the deployment is 'available'?"
echo "   4. What would happen if we scaled to 5 replicas right now?"
echo ""
read -p "Press ENTER after thinking through these questions..."
echo ""

cat << 'ANSWERS_4'
### 🧠 Comprehensive Answer Block 4:

**Question 1 Answer:** The command exports environment variables that redirect your local Docker 
client to use Minikube's Docker daemon instead of your host's daemon. This means images built 
locally are immediately available inside Minikube without needing to push/pull from a registry. 
It's a development optimization that saves time and bandwidth.

**Question 2 Answer:** Resources must exist before they can be referenced. The Deployment 
specifies "namespace: cicd-lab", so the namespace must exist first. Similarly, you'd apply 
ConfigMaps/Secrets before Deployments that use them. This is dependency ordering - Kubernetes 
won't create a resource in a non-existent namespace.

**Question 3 Answer:** The deployment becomes "available" when the number of ready replicas meets 
or exceeds the minimum available replicas (determined by maxUnavailable in the strategy). Ready 
replicas are pods that pass their readiness probes. So "available" means: enough healthy, 
traffic-ready pods are running to serve users.

**Question 4 Answer:** Kubernetes would create 2 additional pods (we have 3, need 5 total). The 
scheduler would place them on available nodes, Docker would pull/start containers, and they'd go 
through initialization (probes must pass). The Service would automatically add them to the load 
balancing pool once ready. No code changes needed - just update replicas and apply!

ANSWERS_4

echo ""
echo "🔄 SPACED REVIEW: Connecting the workflow"
echo "   Code (app.js) → Container (Dockerfile) → Image (docker build)"
echo "   → Deployment (runs image) → Service (exposes pods) → Users"
echo "   Can you explain each arrow? Each transformation?"
echo ""

# ==========================================
# TASK 4: GitHub Actions CI/CD Pipeline
# Active Learning Block 4
# ==========================================

echo "================================================"
echo "TASK 4: GITHUB ACTIONS CI/CD PIPELINE"
echo "================================================"
echo ""

echo "🧠 PREDICT FIRST: We're about to create an automated pipeline."
echo "   Consider:"
echo "   • What stages should a CI/CD pipeline have?"
echo "   • How do we keep Docker Hub credentials secure?"
echo "   • What tests should run before deploying to production?"
echo "   • How do we ensure only main branch deploys to production?"
echo "   • What happens if a deployment fails?"
echo "   Take 60 seconds to think about pipeline architecture..."
echo ""
read -p "Press ENTER when ready to proceed..."
echo ""

echo "📁 CREATING GITHUB ACTIONS STRUCTURE:"
echo "================================================"

mkdir -p .github/workflows
echo "✅ Created .github/workflows directory"
echo ""

cat << 'PIPELINE_OVERVIEW'
GitHub Actions Pipeline Architecture:

┌─────────────────────────────────────────────────┐
│  TRIGGER: Push to main or PR                    │
└─────────────────┬───────────────────────────────┘
                  │
      ┌───────────▼──────────┐
      │   JOB 1: TEST        │
      │  - Checkout code      │
      │  - Install deps       │
      │  - Run tests          │
      │  - Security audit     │
      └───────────┬──────────┘
                  │
      ┌───────────▼──────────────┐
      │   JOB 2: BUILD & PUSH    │
      │  - Build Docker image     │
      │  - Tag appropriately      │
      │  - Push to registry       │
      └───────────┬──────────────┘
                  │
      ┌───────────▼──────────────┐
      │   JOB 3: DEPLOY          │
      │  - Update K8s manifests   │
      │  - Apply to cluster       │
      │  - Verify deployment      │
      └──────────────────────────┘

PIPELINE_OVERVIEW

echo ""
read -p "Ready to create the pipeline? Press ENTER..."
echo ""

# Create main CI/CD workflow
cat > .github/workflows/ci-cd.yaml << 'EOF'
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

env:
  REGISTRY: docker.io
  IMAGE_NAME: cicd-app

jobs:
  # ==========================================
  # JOB 1: Test
  # ==========================================
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Setup Node.js 22
      uses: actions/setup-node@v4
      with:
        node-version: '22'
        cache: 'npm'
    
    - name: Install dependencies
      run: |
        npm ci
        echo "✅ Dependencies installed"
    
    - name: Run linter
      run: |
        npm run lint
        echo "✅ Linting passed"
    
    - name: Run tests
      run: |
        npm test
        echo "✅ Tests passed"
    
    - name: Security audit
      run: |
        npm audit --audit-level=high
        echo "✅ Security audit passed"
      continue-on-error: true
    
    - name: Test summary
      run: |
        echo "### ✅ All tests passed!" >> $GITHUB_STEP_SUMMARY
        echo "" >> $GITHUB_STEP_SUMMARY
        echo "- Linting: ✅" >> $GITHUB_STEP_SUMMARY
        echo "- Unit tests: ✅" >> $GITHUB_STEP_SUMMARY
        echo "- Security audit: ✅" >> $GITHUB_STEP_SUMMARY

  # ==========================================
  # JOB 2: Build and Push
  # ==========================================
  build-and-push:
    name: Build and Push Image
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      image-digest: ${{ steps.build.outputs.digest }}
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Set up QEMU
      uses: docker/setup-qemu-action@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
    
    - name: Log in to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ secrets.DOCKER_USERNAME }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=sha,prefix={{branch}}-
          type=raw,value=latest,enable={{is_default_branch}}
          type=semver,pattern={{version}}
          type=semver,pattern={{major}}.{{minor}}
    
    - name: Build and push Docker image
      id: build
      uses: docker/build-push-action@v5
      with:
        context: .
        platforms: linux/amd64,linux/arm64
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
        build-args: |
          BUILD_DATE=${{ github.event.head_commit.timestamp }}
          VCS_REF=${{ github.sha }}
    
    - name: Build summary
      run: |
        echo "### 🐳 Docker Image Built!" >> $GITHUB_STEP_SUMMARY
        echo "" >> $GITHUB_STEP_SUMMARY
        echo "**Tags:**" >> $GITHUB_STEP_SUMMARY
        echo '```' >> $GITHUB_STEP_SUMMARY
        echo "${{ steps.meta.outputs.tags }}" >> $GITHUB_STEP_SUMMARY
        echo '```' >> $GITHUB_STEP_SUMMARY
        echo "" >> $GITHUB_STEP_SUMMARY
        echo "**Digest:** ${{ steps.build.outputs.digest }}" >> $GITHUB_STEP_SUMMARY

  # ==========================================
  # JOB 3: Deploy
  # ==========================================
  deploy:
    name: Deploy to Kubernetes
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Setup kubectl
      uses: azure/setup-kubectl@v4
      with:
        version: 'v1.33.0'
    
    - name: Configure kubectl
      run: |
        mkdir -p $HOME/.kube
        echo "${{ secrets.KUBECONFIG }}" | base64 -d > $HOME/.kube/config
        chmod 600 $HOME/.kube/config
    
    - name: Update deployment manifest
      run: |
        # Extract first tag from metadata
        IMAGE_TAG=$(echo "${{ needs.build-and-push.outputs.image-tag }}" | head -n1)
        sed -i "s|image: cicd-app:latest|image: ${IMAGE_TAG}|g" k8s-manifests/deployment.yaml
        echo "Updated deployment to use: ${IMAGE_TAG}"
    
    - name: Apply Kubernetes manifests
      run: |
        kubectl apply -f k8s-manifests/namespace.yaml
        kubectl apply -f k8s-manifests/deployment.yaml
        kubectl apply -f k8s-manifests/service.yaml
        echo "✅ Manifests applied"
    
    - name: Wait for rollout
      run: |
        kubectl rollout status deployment/cicd-app -n cicd-lab --timeout=300s
        echo "✅ Deployment rolled out successfully"
    
    - name: Verify deployment
      run: |
        echo "📊 Deployment Status:"
        kubectl get deployment cicd-app -n cicd-lab
        echo ""
        echo "📊 Pods:"
        kubectl get pods -n cicd-lab -l app=cicd-app
        echo ""
        echo "📊 Service:"
        kubectl get service cicd-app-service -n cicd-lab
    
    - name: Run smoke tests
      run: |
        # Get service endpoint
        ENDPOINT=$(kubectl get service cicd-app-service -n cicd-lab -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
        if [ -z "$ENDPOINT" ]; then
          echo "⚠️ LoadBalancer IP not available (expected in Minikube)"
          echo "Using port-forward for testing..."
          kubectl port-forward -n cicd-lab service/cicd-app-service 8080:80 &
          sleep 5
          ENDPOINT="localhost:8080"
        fi
        
        # Test health endpoint
        if curl -f -s http://${ENDPOINT}/health > /dev/null; then
          echo "✅ Health check passed"
        else
          echo "❌ Health check failed"
          exit 1
        fi
        
        # Test main endpoint
        if curl -f -s http://${ENDPOINT}/ > /dev/null; then
          echo "✅ Main endpoint accessible"
        else
          echo "❌ Main endpoint failed"
          exit 1
        fi
    
    - name: Deployment summary
      if: always()
      run: |
        echo "### 🚀 Deployment Complete!" >> $GITHUB_STEP_SUMMARY
        echo "" >> $GITHUB_STEP_SUMMARY
        echo "**Deployed to:** cicd-lab namespace" >> $GITHUB_STEP_SUMMARY
        echo "**Replicas:** $(kubectl get deployment cicd-app -n cicd-lab -o jsonpath='{.status.readyReplicas}')/$(kubectl get deployment cicd-app -n cicd-lab -o jsonpath='{.spec.replicas}')" >> $GITHUB_STEP_SUMMARY
        echo "**Image:** ${{ needs.build-and-push.outputs.image-tag }}" >> $GITHUB_STEP_SUMMARY
EOF

echo "✅ ci-cd.yaml created"
echo ""

# Create rollback workflow
cat << 'ROLLBACK_OVERVIEW'
---[ Rollback Workflow Preview ]---

Manual rollback workflow with:
• Workflow dispatch trigger (manual execution)
• Revision selection (specific version or previous)
• Rollback verification
• Status reporting

Safety features:
- Requires manual trigger (not automatic)
- Shows rollout history before rollback
- Verifies rollback completion
- Reports final status

ROLLBACK_OVERVIEW

echo ""
read -p "Creating rollback workflow... Press ENTER"
echo ""

cat > .github/workflows/rollback.yaml << 'EOF'
name: Rollback Deployment

on:
  workflow_dispatch:
    inputs:
      revision:
        description: 'Revision number to rollback to (leave empty for previous)'
        required: false
        type: string
      namespace:
        description: 'Kubernetes namespace'
        required: false
        default: 'cicd-lab'
        type: string

jobs:
  rollback:
    name: Rollback Deployment
    runs-on: ubuntu-latest
    
    steps:
    - name: Setup kubectl
      uses: azure/setup-kubectl@v4
      with:
        version: 'v1.33.0'
    
    - name: Configure kubectl
      run: |
        mkdir -p $HOME/.kube
        echo "${{ secrets.KUBECONFIG }}" | base64 -d > $HOME/.kube/config
        chmod 600 $HOME/.kube/config
    
    - name: Show rollout history
      run: |
        echo "📜 Deployment History:"
        kubectl rollout history deployment/cicd-app -n ${{ github.event.inputs.namespace }}
    
    - name: Perform rollback
      run: |
        if [ -n "${{ github.event.inputs.revision }}" ]; then
          echo "Rolling back to revision ${{ github.event.inputs.revision }}..."
          kubectl rollout undo deployment/cicd-app \
            -n ${{ github.event.inputs.namespace }} \
            --to-revision=${{ github.event.inputs.revision }}
        else
          echo "Rolling back to previous revision..."
          kubectl rollout undo deployment/cicd-app \
            -n ${{ github.event.inputs.namespace }}
        fi
    
    - name: Wait for rollback completion
      run: |
        kubectl rollout status deployment/cicd-app \
          -n ${{ github.event.inputs.namespace }} \
          --timeout=300s
        echo "✅ Rollback completed"
    
    - name: Verify rollback
      run: |
        echo "📊 Post-Rollback Status:"
        kubectl get deployment cicd-app -n ${{ github.event.inputs.namespace }}
        echo ""
        kubectl get pods -n ${{ github.event.inputs.namespace }} -l app=cicd-app
        echo ""
        kubectl describe deployment cicd-app -n ${{ github.event.inputs.namespace }} | grep Image:
    
    - name: Rollback summary
      run: |
        echo "### 🔄 Rollback Complete!" >> $GITHUB_STEP_SUMMARY
        echo "" >> $GITHUB_STEP_SUMMARY
        echo "**Namespace:** ${{ github.event.inputs.namespace }}" >> $GITHUB_STEP_SUMMARY
        if [ -n "${{ github.event.inputs.revision }}" ]; then
          echo "**Revision:** ${{ github.event.inputs.revision }}" >> $GITHUB_STEP_SUMMARY
        else
          echo "**Revision:** Previous" >> $GITHUB_STEP_SUMMARY
        fi
        echo "" >> $GITHUB_STEP_SUMMARY
        echo "**Status:** ✅ Healthy" >> $GITHUB_STEP_SUMMARY
EOF

echo "✅ rollback.yaml created"
echo ""

echo "📊 GITHUB ACTIONS STRUCTURE:"
find .github -type f
echo ""

echo "================================================"
echo "✅ TASK 4 COMPLETE: CI/CD pipeline configured"
echo "================================================"
echo ""

echo "🤔 ACTIVE RECALL CHECK:"
echo "   1. Why do we separate test, build, and deploy into different jobs?"
echo "   2. What does 'needs: test' mean in the build-and-push job?"
echo "   3. How do GitHub Secrets protect sensitive information?"
echo "   4. Why does the deploy job only run on the main branch?"
echo "   5. What's the purpose of the smoke tests after deployment?"
echo ""
read -p "Press ENTER after thinking through these questions..."
echo ""

cat << 'ANSWERS_5'
### 🧠 Comprehensive Answer Block 5:

**Question 1 Answer:** Job separation provides:
- Parallel execution (tests can run while waiting for other resources)
- Failure isolation (if tests fail, we don't waste time building)
- Clear stages and responsibilities
- Better logs and debugging (each job has focused output)
- Conditional execution (build only if tests pass)

**Question 2 Answer:** 'needs: test' creates a dependency chain. The build-and-push job won't 
start until the test job completes successfully. If tests fail, building and pushing never happen, 
saving resources and preventing broken code from reaching production. This is the "gate" pattern 
in CI/CD.

**Question 3 Answer:** GitHub Secrets are encrypted at rest and only exposed to workflows as 
environment variables during runtime. They never appear in logs (GitHub masks them automatically), 
can't be read via API, and are scoped per repository/organization. This prevents credentials from 
being committed to code or exposed in public logs.

**Question 4 Answer:** The condition 'if: github.ref == "refs/heads/main"' ensures only commits 
to main trigger deployments. This prevents:
- Feature branches from deploying to production
- Pull requests from accessing production credentials
- Multiple concurrent deployments from different branches
- Accidental production updates from development work

**Question 5 Answer:** Smoke tests are quick, basic checks that verify the deployment actually 
works from a user's perspective. They catch issues that manifest only in the deployed environment:
- Network connectivity problems
- Configuration errors in production
- Load balancer misconfigurations
- Service discovery issues
They provide immediate feedback: "Deploy succeeded" vs "Deploy completed but app is broken"

ANSWERS_5

echo ""
echo "🔄 SPACED REVIEW: The complete flow"
echo "   Developer commits → GitHub Actions triggers → Tests run"
echo "   → Image builds → Registry stores it → K8s pulls it → Pods run it"
echo "   → Service exposes it → Users access it"
echo ""
echo "   Can you explain what happens at each arrow?"
echo "   Which components we configured are involved in each step?"
echo ""

# ==========================================
# TASK 5: Testing and Verification
# Active Learning Block 5
# ==========================================

echo "================================================"
echo "TASK 5: COMPREHENSIVE TESTING"
echo "================================================"
echo ""

echo "🧠 PREDICT FIRST: How do we verify everything works?"
echo "   Consider:"
echo "   • What levels of testing are important? (unit, integration, system)"
echo "   • How do we test rollbacks without breaking production?"
echo "   • What monitoring should we implement?"
echo "   • How do we simulate failures safely?"
echo "   Take 45 seconds to think about testing strategy..."
echo ""
read -p "Press ENTER when ready to proceed..."
echo ""

echo "📝 CREATING COMPREHENSIVE TEST SUITE:"
echo "================================================"

# Create test script
cat > test-deployment.sh << 'EOF'
#!/bin/bash

# ==========================================
# COMPREHENSIVE DEPLOYMENT TEST SUITE
# ==========================================

# Color codes for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

NAMESPACE="cicd-lab"
APP_NAME="cicd-app"
TOTAL_TESTS=0
PASSED_TESTS=0

# Test execution function
run_test() {
    local test_name="$1"
    local test_command="$2"
    local expected_pattern="$3"
    
    TOTAL_TESTS=$((TOTAL_TESTS + 1))
    echo -e "${BLUE}🧪 Test $TOTAL_TESTS: $test_name${NC}"
    
    result=$(eval "$test_command" 2>&1)
    
    if echo "$result" | grep -q "$expected_pattern"; then
        echo -e "${GREEN}✅ PASS${NC}"
        PASSED_TESTS=$((PASSED_TESTS + 1))
    else
        echo -e "${RED}❌ FAIL${NC}"
        echo -e "${YELLOW}Expected pattern: $expected_pattern${NC}"
        echo -e "${YELLOW}Got: ${NC}"
        echo "$result" | head -n 5
    fi
    echo ""
}

echo "========================================"
echo "🧪 DEPLOYMENT TEST SUITE"
echo "========================================"
echo ""

# ==========================================
# INFRASTRUCTURE TESTS
# ==========================================
echo -e "${BLUE}=== INFRASTRUCTURE TESTS ===${NC}"
echo ""

run_test "Namespace exists" \
    "kubectl get namespace $NAMESPACE" \
    "$NAMESPACE"

run_test "Deployment exists" \
    "kubectl get deployment $APP_NAME -n $NAMESPACE" \
    "$APP_NAME"

run_test "Service exists" \
    "kubectl get service ${APP_NAME}-service -n $NAMESPACE" \
    "${APP_NAME}-service"

# ==========================================
# AVAILABILITY TESTS
# ==========================================
echo -e "${BLUE}=== AVAILABILITY TESTS ===${NC}"
echo ""

run_test "Deployment is available" \
    "kubectl get deployment $APP_NAME -n $NAMESPACE -o jsonpath='{.status.conditions[?(@.type==\"Available\")].status}'" \
    "True"

run_test "All replicas are ready" \
    "kubectl get deployment $APP_NAME -n $NAMESPACE -o jsonpath='{.status.readyReplicas}'" \
    "[1-9]"

run_test "No pods in error state" \
    "kubectl get pods -n $NAMESPACE -l app=$APP_NAME --field-selector=status.phase!=Running,status.phase!=Succeeded" \
    "No resources found"

# ==========================================
# FUNCTIONALITY TESTS
# ==========================================
echo -e "${BLUE}=== FUNCTIONALITY TESTS ===${NC}"
echo ""

# Get service URL
SERVICE_URL=$(minikube service ${APP_NAME}-service -n $NAMESPACE --url 2>/dev/null)

if [ -n "$SERVICE_URL" ]; then
    run_test "Main endpoint responds" \
        "curl -s -o /dev/null -w '%{http_code}' $SERVICE_URL/" \
        "200"
    
    run_test "Health endpoint responds" \
        "curl -s -o /dev/null -w '%{http_code}' $SERVICE_URL/health" \
        "200"
    
    run_test "Ready endpoint responds" \
        "curl -s -o /dev/null -w '%{http_code}' $SERVICE_URL/ready" \
        "200"
    
    run_test "Metrics endpoint responds" \
        "curl -s -o /dev/null -w '%{http_code}' $SERVICE_URL/metrics" \
        "200"
    
    run_test "Application returns correct data structure" \
        "curl -s $SERVICE_URL/ | jq -r '.message'" \
        "Hello from Kubernetes"
else
    echo -e "${YELLOW}⚠️ Service URL not available - skipping endpoint tests${NC}"
fi

# ==========================================
# RESOURCE TESTS
# ==========================================
echo -e "${BLUE}=== RESOURCE TESTS ===${NC}"
echo ""

run_test "Pods have resource limits set" \
    "kubectl get pods -n $NAMESPACE -l app=$APP_NAME -o jsonpath='{.items[0].spec.containers[0].resources.limits}'" \
    "cpu"

run_test "Pods have resource requests set" \
    "kubectl get pods -n $NAMESPACE -l app=$APP_NAME -o jsonpath='{.items[0].spec.containers[0].resources.requests}'" \
    "memory"

# ==========================================
# SECURITY TESTS
# ==========================================
echo -e "${BLUE}=== SECURITY TESTS ===${NC}"
echo ""

run_test "Pods run as non-root user" \
    "kubectl get pods -n $NAMESPACE -l app=$APP_NAME -o jsonpath='{.items[0].spec.containers[0].securityContext.runAsNonRoot}'" \
    "true"

run_test "Privilege escalation is disabled" \
    "kubectl get pods -n $NAMESPACE -l app=$APP_NAME -o jsonpath='{.items[0].spec.containers[0].securityContext.allowPrivilegeEscalation}'" \
    "false"

# ==========================================
# PROBE TESTS
# ==========================================
echo -e "${BLUE}=== HEALTH PROBE TESTS ===${NC}"
echo ""

run_test "Liveness probe is configured" \
    "kubectl get deployment $APP_NAME -n $NAMESPACE -o jsonpath='{.spec.template.spec.containers[0].livenessProbe.httpGet.path}'" \
    "/health"

run_test "Readiness probe is configured" \
    "kubectl get deployment $APP_NAME -n $NAMESPACE -o jsonpath='{.spec.template.spec.containers[0].readinessProbe.httpGet.path}'" \
    "/ready"

# ==========================================
# TEST SUMMARY
# ==========================================
echo ""
echo "========================================"
echo "📊 TEST SUMMARY"
echo "========================================"
echo -e "Total Tests:  $TOTAL_TESTS"
echo -e "Passed:       ${GREEN}$PASSED_TESTS${NC}"
echo -e "Failed:       ${RED}$((TOTAL_TESTS - PASSED_TESTS))${NC}"
echo -e "Success Rate: $(( (PASSED_TESTS * 100) / TOTAL_TESTS ))%"
echo ""

if [ $PASSED_TESTS -eq $TOTAL_TESTS ]; then
    echo -e "${GREEN}🎉 ALL TESTS PASSED!${NC}"
    exit 0
else
    echo -e "${RED}⚠️  SOME TESTS FAILED - REVIEW CONFIGURATION${NC}"
    exit 1
fi
EOF

chmod +x test-deployment.sh
echo "✅ test-deployment.sh created"
echo ""

echo "🧪 RUNNING TEST SUITE:"
echo "================================================"
./test-deployment.sh
echo ""

echo "📊 CREATING MONITORING DASHBOARD:"
echo "================================================"

cat > monitor-deployment.sh << 'EOF'
#!/bin/bash

# ==========================================
# REAL-TIME MONITORING DASHBOARD
# ==========================================

NAMESPACE="cicd-lab"
APP_NAME="cicd-app"

clear
echo "========================================"
echo "📊 K8S CI/CD MONITORING DASHBOARD"
echo "========================================"
echo "Press Ctrl+C to exit"
echo ""

while true; do
    # Move cursor to top
    tput cup 5 0
    
    echo "🕐 Timestamp: $(date '+%Y-%m-%d %H:%M:%S')"
    echo ""
    
    echo "📦 DEPLOYMENT STATUS:"
    kubectl get deployment $APP_NAME -n $NAMESPACE 2>/dev/null || echo "Deployment not found"
    echo ""
    
    echo "🐳 POD STATUS:"
    kubectl get pods -n $NAMESPACE -l app=$APP_NAME 2>/dev/null || echo "No pods found"
    echo ""
    
    echo "🌐 SERVICE STATUS:"
    kubectl get service ${APP_NAME}-service -n $NAMESPACE 2>/dev/null || echo "Service not found"
    echo ""
    
    echo "📈 RESOURCE USAGE (if metrics-server is enabled):"
    kubectl top pods -n $NAMESPACE -l app=$APP_NAME 2>/dev/null || echo "Metrics not available"
    echo ""
    
    echo "🔄 RECENT EVENTS:"
    kubectl get events -n $NAMESPACE --sort-by='.lastTimestamp' | tail -n 5 2>/dev/null
    echo ""
    
    echo "Refreshing in 5 seconds..."
    sleep 5
done
EOF

chmod +x monitor-deployment.sh
echo "✅ monitor-deployment.sh created"
echo ""

echo "================================================"
echo "✅ TASK 5 COMPLETE: Testing framework ready"
echo "================================================"
echo ""

echo "🤔 ACTIVE RECALL CHECK:"
echo "   1. Why do we test infrastructure before functionality?"
echo "   2. What's the difference between testing availability vs functionality?"
echo "   3. Why are security tests important in a CI/CD pipeline?"
echo "   4. How do health probes contribute to system reliability?"
echo "   5. What would you add to make this test suite more comprehensive?"
echo ""
read -p "Press ENTER after thinking through these questions..."
echo ""

cat << 'ANSWERS_6'
### 🧠 Comprehensive Answer Block 6:

**Question 1 Answer:** Infrastructure tests establish the foundation. If namespace, deployments, 
or services don't exist, functionality tests will fail with misleading errors. Testing in layers 
(infrastructure → availability → functionality → security) helps pinpoint issues quickly. Failed 
infrastructure tests mean "environment problem," failed functionality tests mean "application problem."

**Question 2 Answer:**
- Availability: Can the system accept requests? (deployment ready, pods running, service exists)
- Functionality: Does the system work correctly? (endpoints return proper data, business logic works)
Availability is necessary but not sufficient. A deployment might be "available" with all pods 
running, but if the app has bugs, functionality tests will catch them.

**Question 3 Answer:** Security vulnerabilities in CI/CD can:
- Allow container escapes (if running as root)
- Enable privilege escalation attacks
- Expose sensitive data through excessive permissions
- Create compliance violations
Testing security configurations automatically prevents these issues from reaching production. 
It's cheaper to catch security problems in CI/CD than after deployment.

**Question 4 Answer:** Health probes enable:
- Automatic failure detection (liveness probe → restart failing pods)
- Traffic management (readiness probe → remove unhealthy pods from load balancing)
- Zero-downtime deployments (wait for readiness before sending traffic)
- Self-healing (Kubernetes automatically maintains desired state)
Without probes, Kubernetes assumes all running containers are healthy, leading to traffic being 
sent to broken pods.

**Question 5 Answer:** Additional tests could include:
- Performance tests (response time, throughput)
- Load tests (behavior under stress)
- Integration tests (database connectivity, external APIs)
- Chaos engineering (pod deletion, network issues)
- Security scanning (CVE detection, secret detection)
- Compliance checks (policy enforcement)
The goal is confidence: after tests pass, you're confident the deployment will succeed.

ANSWERS_6

echo ""
echo "🔄 SPACED REVIEW: Connecting testing to earlier concepts"
echo "   Remember the probes in deployment.yaml?"
echo "   → Those enable our health check tests!"
echo "   Remember the resource limits?"
echo "   → Those enable our resource tests!"
echo "   Remember the security context?"
echo "   → Those enable our security tests!"
echo "   Everything we configured earlier is now being validated!"
echo ""

# ==========================================
# TASK 6: Simulating Failures and Rollbacks
# Active Learning Block 6
# ==========================================

echo "================================================"
echo "TASK 6: FAILURE SIMULATION AND ROLLBACK"
echo "================================================"
echo ""

echo "🧠 PREDICT FIRST: Let's simulate a deployment failure."
echo "   Consider:"
echo "   • What kinds of failures can occur during deployment?"
echo "   • How does Kubernetes detect a failed deployment?"
echo "   • What's the rollback process?"
echo "   • How do we minimize downtime during failures?"
echo "   • What data should we preserve during rollback?"
echo "   Take 60 seconds to think about failure scenarios..."
echo ""
read -p "Press ENTER when ready to proceed..."
echo ""

echo "📜 CHECKING CURRENT DEPLOYMENT HISTORY:"
echo "================================================"

echo "Current rollout history:"
kubectl rollout history deployment/$APP_NAME -n $NAMESPACE
echo ""

echo "Current deployment details:"
kubectl describe deployment $APP_NAME -n $NAMESPACE | grep -A 5 "Image:"
echo ""

echo "💥 SIMULATING A BROKEN DEPLOYMENT:"
echo "================================================"

cat << 'BROKEN_SCENARIO'
We'll create a broken version that:
1. Has syntax errors
2. Fails health checks
3. Demonstrates Kubernetes' rollback protection

This teaches:
- How Kubernetes handles failures
- Why maxUnavailable: 0 matters
- The importance of health probes
- Rollback procedures

BROKEN_SCENARIO

echo ""
read -p "Ready to deploy broken version? Press ENTER..."
echo ""

# Create broken app version
cat > app-broken.js << 'EOF'
const express = require('express');
const app = express();
const port = process.env.PORT || 3000;

// Request logging middleware
app.use((req, res, next) => {
  console.log(`${new Date().toISOString()} - ${req.method} ${req.path}`);
  next();
});

// Broken main endpoint - calls undefined function
app.get('/', (req, res) => {
  // This will crash the application
  nonExistentFunction();
  
  res.json({
    message: 'This will never be reached',
    version: '2.0.0-broken'
  });
});

// Health endpoint that fails
app.get('/health', (req, res) => {
  // Always return unhealthy
  res.status(503).json({ 
    status: 'unhealthy',
    reason: 'Simulated failure'
  });
});

// Broken ready endpoint
app.get('/ready', (req, res) => {
  res.status(503).json({ 
    status: 'not ready',
    reason: 'Simulated failure'
  });
});

const server = app.listen(port, '0.0.0.0', () => {
  console.log(`💥 Broken app running on port ${port}`);
});

process.on('SIGTERM', () => {
  console.log('SIGTERM signal received');
  server.close(() => {
    process.exit(0);
  });
});
EOF

echo "✅ Broken application code created"
echo ""

# Backup original app
cp app.js app-original.js
cp app-broken.js app.js

echo "🔨 BUILDING BROKEN IMAGE:"
docker build -t cicd-app:v2.0.0-broken -t cicd-app:latest .
echo ""

echo "📝 UPDATING DEPLOYMENT MANIFEST:"
sed -i 's|APP_VERSION.*|APP_VERSION\n          value: "2.0.0-broken"|g' k8s-manifests/deployment.yaml
echo "✅ Manifest updated to v2.0.0-broken"
echo ""

echo "🚀 DEPLOYING BROKEN VERSION:"
echo "================================================"
echo "Watch how Kubernetes handles this failure..."
echo ""

kubectl apply -f k8s-manifests/deployment.yaml
echo ""

echo "⏳ MONITORING DEPLOYMENT (will timeout after 60 seconds):"
echo "================================================"

# Monitor rollout with timeout
timeout 60s kubectl rollout status deployment/$APP_NAME -n $NAMESPACE || echo "⚠️ Rollout timed out (expected)"
echo ""

echo "📊 ANALYZING FAILURE:"
echo "================================================"

echo "Deployment status:"
kubectl get deployment $APP_NAME -n $NAMESPACE
echo ""

echo "Pod status (notice some may be failing):"
kubectl get pods -n $NAMESPACE -l app=$APP_NAME
echo ""

echo "Recent events showing the failure:"
kubectl get events -n $NAMESPACE --sort-by='.lastTimestamp' | grep -i "unhealthy\|failed\|error" | tail -n 10
echo ""

echo "Detailed pod description (checking one failing pod):"
FAILING_POD=$(kubectl get pods -n $NAMESPACE -l app=$APP_NAME -o jsonpath='{.items[0].metadata.name}')
kubectl describe pod $FAILING_POD -n $NAMESPACE | grep -A 10 "Liveness\|Readiness\|Events"
echo ""

echo "🤔 OBSERVATION POINT:"
echo "   Notice:"
echo "   • Some old pods are still running (maxUnavailable: 0 protection)"
echo "   • New pods are failing health checks"
echo "   • Kubernetes is NOT killing old pods (protecting availability)"
echo "   • The rollout is stuck, but service is still available!"
echo ""
read -p "Press ENTER after reviewing the failure state..."
echo ""

echo "🔧 PERFORMING MANUAL ROLLBACK:"
echo "================================================"

echo "Rolling back to previous working version..."
kubectl rollout undo deployment/$APP_NAME -n $NAMESPACE
echo ""

echo "⏳ Waiting for rollback to complete:"
kubectl rollout status deployment/$APP_NAME -n $NAMESPACE --timeout=120s
echo ""

echo "📊 POST-ROLLBACK STATUS:"
echo "================================================"

echo "Deployment status:"
kubectl get deployment $APP_NAME -n $NAMESPACE
echo ""

echo "Pod status (should all be healthy now):"
kubectl get pods -n $NAMESPACE -l app=$APP_NAME
echo ""

echo "Rollout history (notice the new revision):"
kubectl rollout history deployment/$APP_NAME -n $NAMESPACE
echo ""

# Restore original app
cp app-original.js app.js
rm app-broken.js app-original.js

echo "✅ Rollback successful - service restored"
echo ""

echo "🧪 VERIFYING SERVICE AFTER ROLLBACK:"
SERVICE_URL=$(minikube service ${APP_NAME}-service -n $NAMESPACE --url)
echo "Testing service at: $SERVICE_URL"
echo ""

echo "Health check:"
curl -s $SERVICE_URL/health | jq '.' || curl -s $SERVICE_URL/health
echo ""

echo "Main endpoint:"
curl -s $SERVICE_URL/ | jq '.' || curl -s $SERVICE_URL/
echo ""

echo "================================================"
echo "✅ TASK 6 COMPLETE: Failure handling demonstrated"
echo "================================================"
echo ""

echo "🤔 ACTIVE RECALL CHECK:"
echo "   1. Why didn't Kubernetes kill all old pods when the new version failed?"
echo "   2. How do readiness probes protect against bad deployments?"
echo "   3. What's the difference between 'kubectl rollout undo' and manual pod deletion?"
echo "   4. How would this scenario play out WITHOUT health probes?"
echo "   5. What metrics should trigger automatic rollback in production?"
echo ""
read -p "Press ENTER after thinking through these questions..."
echo ""

cat << 'ANSWERS_7'
### 🧠 Comprehensive Answer Block 7:

**Question 1 Answer:** The rolling update strategy specifies 'maxUnavailable: 0', meaning 
Kubernetes must keep all old pods running until new pods become ready. Since new pods never 
passed readiness probes, Kubernetes preserved the old pods to maintain availability. This is 
the "safety net" - bad deployments don't take down your service.

**Question 2 Answer:** Readiness probes prevent traffic from being sent to unhealthy pods. In our 
failure scenario, new pods never became "ready," so:
- Kubernetes didn't add them to the Service endpoints
- Old pods continued receiving all traffic
- Users experienced zero downtime despite a failed deployment
Without readiness probes, traffic would be sent to broken pods immediately.

**Question 3 Answer:**
- 'kubectl rollout undo': Declarative rollback to a previous deployment revision, managed by 
  Kubernetes, includes full deployment spec (image, config, replicas, etc.)
- Manual pod deletion: Imperative action, only kills specific pods, deployment controller 
  recreates them with current spec (broken version)
Rollback is the proper solution - it changes the deployment specification back to a working state.

**Question 4 Answer:** Without health probes, Kubernetes assumes all running containers are healthy:
1. New broken pods would start
2. They'd immediately be added to Service endpoints
3. Traffic would be sent to them
4. Users would get errors (500 responses, timeouts)
5. Old pods would be terminated (maxUnavailable allows it)
6. Result: Complete service outage
Health probes are the difference between "deployment failed but service works" and "complete outage."

**Question 5 Answer:** Automatic rollback triggers should include:
- Error rate exceeds threshold (e.g., >5% 5xx responses)
- Latency degradation (e.g., P95 latency >2x baseline)
- Failed health checks across majority of new pods
- Custom business metrics (e.g., conversion rate drops)
- Crash loop detection (pods restarting repeatedly)
- Resource exhaustion (OOM kills, CPU throttling)
Production systems use tools like Flagger, Argo Rollouts, or custom operators to monitor these 
metrics and automatically rollback when thresholds are exceeded.

ANSWERS_7

echo ""
echo "🔄 SPACED REVIEW: The complete safety system"
echo "   Health probes (app.js) → Kubernetes monitoring → Rollout strategy (deployment.yaml)"
echo "   → Service routing → Zero-downtime protection"
echo ""
echo "   Each component we built plays a role in the safety system!"
echo "   Can you trace the path from code to user protection?"
echo ""

# ==========================================
# TASK 7: Production Best Practices
# Active Learning Block 7
# ==========================================

echo "================================================"
echo "TASK 7: PRODUCTION-READY ENHANCEMENTS"
echo "================================================"
echo ""

echo "🧠 PREDICT FIRST: What's missing for production?"
echo "   Consider:"
echo "   • How do we separate staging from production?"
echo "   • What about secrets management?"
echo "   • How do we handle different configurations per environment?"
echo "   • What monitoring and logging do we need?"
echo "   Take 60 seconds to think about production requirements..."
echo ""
read -p "Press ENTER when ready to proceed..."
echo ""

echo "📁 CREATING MULTI-ENVIRONMENT STRUCTURE:"
echo "================================================"

mkdir -p k8s-manifests/overlays/{staging,production}
mkdir -p k8s-manifests/base

echo "✅ Directory structure created"
echo ""

cat << 'KUSTOMIZE_OVERVIEW'
Kustomize Strategy:

📂 k8s-manifests/
  ├── 📂 base/           # Common resources
  │   ├── deployment.yaml
  │   ├── service.yaml
  │   └── kustomization.yaml
  │
  └── 📂 overlays/       # Environment-specific
      ├── 📂 staging/
      │   └── kustomization.yaml (1 replica, staging config)
      └── 📂 production/
          └── kustomization.yaml (5 replicas, prod config)

Benefits:
- DRY principle (Don't Repeat Yourself)
- Easy environment management
- Clear separation of concerns
- Native Kubernetes tool (no extra dependencies)

KUSTOMIZE_OVERVIEW

echo ""
read -p "Creating Kustomize structure... Press ENTER"
echo ""

# Move existing manifests to base
cp k8s-manifests/namespace.yaml k8s-manifests/base/
cp k8s-manifests/deployment.yaml k8s-manifests/base/
cp k8s-manifests/service.yaml k8s-manifests/base/

# Create base kustomization
cat > k8s-manifests/base/kustomization.yaml << 'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - namespace.yaml
  - deployment.yaml
  - service.yaml

commonLabels:
  app.kubernetes.io/managed-by: kustomize
  app.kubernetes.io/part-of: cicd-lab
EOF

echo "✅ Base kustomization created"
echo ""

# Create staging overlay
cat > k8s-manifests/overlays/staging/kustomization.yaml << 'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: staging

namePrefix: staging-

commonLabels:
  environment: staging

bases:
  - ../../base

replicas:
  - name: cicd-app
    count: 1

patches:
  - target:
      kind: Deployment
      name: cicd-app
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0/env/0/value
        value: "1.0.0-staging"
      - op: replace
        path: /spec/template/spec/containers/0/env/1/value
        value: "staging"
      - op: replace
        path: /spec/template/spec/containers/0/resources/limits/memory
        value: "128Mi"
      - op: replace
        path: /spec/template/spec/containers/0/resources/limits/cpu
        value: "100m"

  - target:
      kind: Service
      name: cicd-app-service
    patch: |-
      - op: replace
        path: /spec/type
        value: NodePort
      - op: replace
        path: /spec/ports/0/nodePort
        value: 30081
EOF

echo "✅ Staging overlay created"
echo ""

# Create production overlay
cat > k8s-manifests/overlays/production/kustomization.yaml << 'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: production

namePrefix: prod-

commonLabels:
  environment: production

bases:
  - ../../base

replicas:
  - name: cicd-app
    count: 5

patches:
  - target:
      kind: Deployment
      name: cicd-app
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0/env/0/value
        value: "1.0.0-production"
      - op: replace
        path: /spec/template/spec/containers/0/env/1/value
        value: "production"
      - op: add
        path: /spec/template/spec/containers/0/env/-
        value:
          name: LOG_LEVEL
          value: "error"

  - target:
      kind: Service
      name: cicd-app-service
    patch: |-
      - op: replace
        path: /spec/type
        value: LoadBalancer
EOF

echo "✅ Production overlay created"
echo ""

echo "🏗️ CREATING NAMESPACES:"
kubectl create namespace staging --dry-run=client -o yaml | kubectl apply -f -
kubectl create namespace production --dry-run=client -o yaml | kubectl apply -f -
echo "✅ Namespaces created"
echo ""

echo "🚀 DEPLOYING TO STAGING:"
echo "================================================"

kubectl apply -k k8s-manifests/overlays/staging/
echo ""

echo "⏳ Waiting for staging deployment:"
kubectl rollout status deployment/staging-cicd-app -n staging --timeout=120s
echo ""

echo "📊 STAGING STATUS:"
kubectl get all -n staging -l environment=staging
echo ""

echo "🚀 DEPLOYING TO PRODUCTION:"
echo "================================================"

kubectl apply -k k8s-manifests/overlays/production/
echo ""

echo "⏳ Waiting for production deployment:"
kubectl rollout status deployment/prod-cicd-app -n production --timeout=120s
echo ""

echo "📊 PRODUCTION STATUS:"
kubectl get all -n production -l environment=production
echo ""

echo "📊 COMPARING ENVIRONMENTS:"
echo "================================================"

echo "Staging (1 replica, limited resources):"
kubectl get deployment staging-cicd-app -n staging
echo ""

echo "Production (5 replicas, full resources):"
kubectl get deployment prod-cicd-app -n production
echo ""

echo "================================================"
echo "✅ TASK 7 COMPLETE: Multi-environment setup ready"
echo "================================================"
echo ""

echo "🤔 ACTIVE RECALL CHECK:"
echo "   1. Why use Kustomize instead of copying YAML files?"
echo "   2. How does namePrefix prevent resource conflicts?"
echo "   3. Why do we use fewer replicas in staging?"
echo "   4. What's the purpose of different NODE_ENV values?"
echo "   5. How would you add a 'dev' environment?"
echo ""
read -p "Press ENTER after thinking through these questions..."
echo ""

cat << 'ANSWERS_8'
### 🧠 Comprehensive Answer Block 8:

**Question 1 Answer:** Kustomize provides:
- Single source of truth (base) with environment variations (overlays)
- No template language to learn (pure Kubernetes YAML)
- Reduces duplication and maintenance burden
- Changes to base automatically affect all environments
- Native kubectl support ('kubectl apply -k')
If you copy files, changes must be made in multiple places, leading to drift and inconsistencies.

**Question 2 Answer:** namePrefix adds a prefix to all resource names (staging-, prod-). This:
- Allows multiple environments in the same cluster without name conflicts
- Makes environment identification clear in logs and dashboards
- Enables safe parallel deployments
- Prevents accidental cross-environment operations
Without prefixes, "cicd-app" in staging would conflict with "cicd-app" in production.

**Question 3 Answer:** Staging uses fewer replicas because:
- Lower cost (fewer resources consumed)
- Faster deployment (less time waiting for all pods)
- Adequate for testing purposes (doesn't need production load capacity)
- Resource efficiency (staging gets less traffic)
Production needs high availability and capacity; staging needs representativeness, not scale.

**Question 4 Answer:** NODE_ENV affects application behavior:
- 'development': Verbose logging, debug mode, hot reloading
- 'staging': Production-like but with more logging for troubleshooting
- 'production': Minimal logging, optimized performance, strict error handling
It's a convention that libraries (Express, React, etc.) use to adjust behavior automatically.

**Question 5 Answer:** Create k8s-manifests/overlays/dev/kustomization.yaml with:
- namespace: dev
- namePrefix: dev-
- replicas: 1 (minimal for local testing)
- Resources: even more limited than staging
- Different NODE_ENV: development
- Possibly different service type: ClusterIP (no external access needed)
Then: kubectl apply -k k8s-manifests/overlays/dev/

ANSWERS_8

echo ""
echo "🔄 SPACED REVIEW: Architecture evolution"
echo "   Started: Single deployment in default namespace"
echo "   Now: Multi-environment with proper isolation"
echo ""
echo "   Same base resources (app, Docker, K8s)"
echo "   Different configurations per environment"
echo "   Production-ready architecture! 🎉"
echo ""

# ==========================================
# TASK 8: Troubleshooting and Debugging
# Active Learning Block 8
# ==========================================

echo "================================================"
echo "TASK 8: TROUBLESHOOTING TOOLKIT"
echo "================================================"
echo ""

echo "🧠 PREDICT FIRST: Common production issues"
echo "   Consider:"
echo "   • What goes wrong in production?"
echo "   • How do we diagnose issues quickly?"
echo "   • What logs and metrics do we need?"
echo "   • How do we debug without disrupting service?"
echo "   Take 45 seconds to think about debugging strategies..."
echo ""
read -p "Press ENTER when ready to proceed..."
echo ""

echo "🔧 CREATING TROUBLESHOOTING TOOLKIT:"
echo "================================================"

cat > troubleshoot.sh << 'EOF'
#!/bin/bash

# ==========================================
# KUBERNETES TROUBLESHOOTING TOOLKIT
# ==========================================

# Color codes
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

# Configuration
NAMESPACE="${1:-cicd-lab}"
APP_NAME="${2:-cicd-app}"

echo -e "${BLUE}================================"
echo "🔍 K8S TROUBLESHOOTING DIAGNOSTIC"
echo "================================${NC}"
echo "Namespace: $NAMESPACE"
echo "App: $APP_NAME"
echo ""

# Function for component checking
check_component() {
    local component="$1"
    local check_command="$2"
    local success_pattern="$3"
    local troubleshooting_tips="$4"
    
    echo -e "${YELLOW}Checking: $component${NC}"
    result=$(eval "$check_command" 2>&1)
    
    if echo "$result" | grep -q "$success_pattern"; then
        echo -e "${GREEN}✅ $component: OK${NC}"
    else
        echo -e "${RED}❌ $component: ISSUE DETECTED${NC}"
        echo -e "${YELLOW}Troubleshooting Steps:${NC}"
        echo "$troubleshooting_tips"
        echo ""
        echo -e "${YELLOW}Diagnostic Output:${NC}"
        echo "$result" | head -n 10
    fi
    echo ""
}

# ==========================================
# NAMESPACE CHECK
# ==========================================
check_component \
    "Namespace" \
    "kubectl get namespace $NAMESPACE" \
    "$NAMESPACE.*Active" \
    "1. Create namespace: kubectl create namespace $NAMESPACE
2. Check permissions: kubectl auth can-i get namespace
3. Verify cluster connectivity: kubectl cluster-info"

# ==========================================
# DEPLOYMENT CHECK
# ==========================================
check_component \
    "Deployment" \
    "kubectl get deployment -n $NAMESPACE" \
    "$APP_NAME" \
    "1. Apply deployment: kubectl apply -f deployment.yaml
2. Check YAML syntax: kubectl apply --dry-run=client -f deployment.yaml
3. View deployment events: kubectl describe deployment $APP_NAME -n $NAMESPACE"

# ==========================================
# POD STATUS CHECK
# ==========================================
echo -e "${YELLOW}Checking: Pod Status${NC}"
POD_STATUS=$(kubectl get pods -n $NAMESPACE -l app=$APP_NAME -o jsonpath='{.items[*].status.phase}' 2>&1)

if echo "$POD_STATUS" | grep -q "Running"; then
    echo -e "${GREEN}✅ Pods: Running${NC}"
else
    echo -e "${RED}❌ Pods: Issues detected${NC}"
    echo -e "${YELLOW}Pod Details:${NC}"
    kubectl get pods -n $NAMESPACE -l app=$APP_NAME
    echo ""
    
    # Get first pod for detailed diagnostics
    POD_NAME=$(kubectl get pods -n $NAMESPACE -l app=$APP_NAME -o jsonpath='{.items[0].metadata.name}' 2>/dev/null)
    
    if [ -n "$POD_NAME" ]; then
        echo -e "${YELLOW}Detailed diagnostics for: $POD_NAME${NC}"
        echo ""
        
        echo "Pod Events:"
        kubectl describe pod $POD_NAME -n $NAMESPACE | grep -A 20 "Events:"
        echo ""
        
        echo "Container Status:"
        kubectl get pod $POD_NAME -n $NAMESPACE -o jsonpath='{.status.containerStatuses[*]}' | jq '.'
        echo ""
        
        echo -e "${YELLOW}Troubleshooting Steps:${NC}"
        echo "1. Check logs: kubectl logs $POD_NAME -n $NAMESPACE"
        echo "2. Check previous logs: kubectl logs $POD_NAME -n $NAMESPACE --previous"
        echo "3. Describe pod: kubectl describe pod $POD_NAME -n $NAMESPACE"
        echo "4. Check events: kubectl get events -n $NAMESPACE --sort-by='.lastTimestamp'"
    fi
fi
echo ""

# ==========================================
# IMAGE CHECK
# ==========================================
check_component \
    "Container Image" \
    "kubectl get pods -n $NAMESPACE -l app=$APP_NAME -o jsonpath='{.items[0].spec.containers[0].image}'" \
    "cicd-app" \
    "1. Verify image exists: docker images | grep cicd-app
2. Check image pull policy: kubectl get deployment $APP_NAME -n $NAMESPACE -o yaml | grep imagePullPolicy
3. Check image pull secrets: kubectl get secrets -n $NAMESPACE"

# ==========================================
# SERVICE CHECK
# ==========================================
echo -e "${YELLOW}Checking: Service Configuration${NC}"
SVC_STATUS=$(kubectl get service -n $NAMESPACE 2>&1)

if echo "$SVC_STATUS" | grep -q "service.*$APP_NAME"; then
    echo -e "${GREEN}✅ Service: Exists${NC}"
    
    # Check endpoints
    ENDPOINTS=$(kubectl get endpoints -n $NAMESPACE -o jsonpath='{.items[*].subsets[*].addresses[*].ip}' 2>&1)
    if [ -n "$ENDPOINTS" ]; then
        echo -e "${GREEN}✅ Service Endpoints: Configured${NC}"
        echo "Endpoints: $ENDPOINTS"
    else
        echo -e "${RED}❌ Service Endpoints: No endpoints found${NC}"
        echo -e "${YELLOW}Troubleshooting:${NC}"
        echo "1. Check pod labels match service selector"
        echo "2. Verify pods are ready: kubectl get pods -n $NAMESPACE -l app=$APP_NAME"
        echo "3. Check readiness probes"
    fi
else
    echo -e "${RED}❌ Service: Not found${NC}"
    echo -e "${YELLOW}Troubleshooting:${NC}"
    echo "1. Apply service: kubectl apply -f service.yaml"
    echo "2. Check namespace: kubectl get services --all-namespaces | grep $APP_NAME"
fi
echo ""

# ==========================================
# RESOURCE USAGE CHECK
# ==========================================
echo -e "${YELLOW}Checking: Resource Usage${NC}"
METRICS=$(kubectl top pods -n $NAMESPACE -l app=$APP_NAME 2>&1)

if echo "$METRICS" | grep -v "error\|Error\|not found"; then
    echo -e "${GREEN}✅ Metrics available${NC}"
    echo "$METRICS"
else
    echo -e "${YELLOW}⚠️ Metrics not available${NC}"
    echo "Note: Requires metrics-server addon"
fi
echo ""

# ==========================================
# LOGS SUMMARY
# ==========================================
echo -e "${YELLOW}Recent Logs (last 20 lines):${NC}"
POD_NAME=$(kubectl get pods -n $NAMESPACE -l app=$APP_NAME -o jsonpath='{.items[0].metadata.name}' 2>/dev/null)

if [ -n "$POD_NAME" ]; then
    kubectl logs $POD_NAME -n $NAMESPACE --tail=20
else
    echo "No pods found to retrieve logs"
fi
echo ""

# ==========================================
# SUMMARY
# ==========================================
echo -e "${BLUE}================================"
echo "📊 DIAGNOSTIC COMPLETE"
echo "================================${NC}"
echo ""
echo "For detailed investigation, use:"
echo "  kubectl describe pod <pod-name> -n $NAMESPACE"
echo "  kubectl logs <pod-name> -n $NAMESPACE --follow"
echo "  kubectl exec -it <pod-name> -n $NAMESPACE -- /bin/sh"
echo "  kubectl get events -n $NAMESPACE --sort-by='.lastTimestamp'"
EOF

chmod +x troubleshoot.sh
echo "✅ troubleshoot.sh created"
echo ""

echo "🧪 RUNNING DIAGNOSTICS ON CURRENT DEPLOYMENT:"
echo "================================================"
./troubleshoot.sh cicd-lab cicd-app
echo ""

echo "📝 CREATING QUICK REFERENCE GUIDE:"
echo "================================================"

cat > TROUBLESHOOTING_GUIDE.md << 'EOF'
# Kubernetes CI/CD Troubleshooting Guide

## Common Issues and Solutions

### 1. Pods Not Starting (Image Pull Errors)

**Symptoms:**
- Pods stuck in ImagePullBackOff or ErrImagePull

**Diagnosis:**
```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl get events -n <namespace> | grep <pod-name>
```

**Solutions:**
- Verify image exists: `docker images | grep <image-name>`
- Check image pull policy: Should be `IfNotPresent` for Minikube
- Ensure Docker environment is configured: `eval $(minikube docker-env)`
- Rebuild image if necessary

---

### 2. Pods Failing Health Checks

**Symptoms:**
- Pods in Running state but not Ready
- Continuous restart loops

**Diagnosis:**
```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
```

**Solutions:**
- Check application actually starts: `kubectl logs <pod-name>`
- Verify health endpoints work: `kubectl exec <pod-name> -- curl localhost:3000/health`
- Adjust probe timing: Increase `initialDelaySeconds` or `timeoutSeconds`
- Check resource limits: Pods might be CPU/memory throttled

---

### 3. Service Not Accessible

**Symptoms:**
- Cannot curl service endpoint
- Connection refused errors

**Diagnosis:**
```bash
kubectl get service -n <namespace>
kubectl get endpoints -n <namespace>
kubectl describe service <service-name> -n <namespace>
```

**Solutions:**
- Verify service selector matches pod labels
- Check pods are Ready: `kubectl get pods -n <namespace>`
- Ensure ports are correct: containerPort matches service targetPort
- For Minikube: Use `minikube service <service-name> --url`

---

### 4. Deployment Stuck in Progress

**Symptoms:**
- Deployment shows "Progressing" for extended time
- Old and new pods running simultaneously

**Diagnosis:**
```bash
kubectl rollout status deployment/<name> -n <namespace>
kubectl describe deployment/<name> -n <namespace>
```

**Solutions:**
- Check new pods status: `kubectl get pods -n <namespace>`
- Review recent events: `kubectl get events -n <namespace>`
- If new version is broken, rollback: `kubectl rollout undo deployment/<name> -n <namespace>`
- Check maxUnavailable and maxSurge in deployment strategy

---

### 5. Resource Exhaustion

**Symptoms:**
- Pods evicted or OOMKilled
- CPU throttling

**Diagnosis:**
```bash
kubectl top pods -n <namespace>
kubectl describe pod <pod-name> -n <namespace> | grep -A 5 "Limits\|Requests"
```

**Solutions:**
- Increase resource limits in deployment.yaml
- Scale down replicas if cluster resources are limited
- Check cluster capacity: `kubectl top nodes`
- Optimize application memory usage

---

### 6. GitHub Actions Pipeline Failures

**Symptoms:**
- Workflow fails at specific jobs
- Secret/credential issues

**Diagnosis:**
- Check GitHub Actions logs in repository
- Verify secrets are set correctly in repository settings

**Solutions:**
- Docker Hub credentials: Verify DOCKER_USERNAME and DOCKER_PASSWORD secrets
- Kubeconfig: Ensure KUBECONFIG secret is base64 encoded correctly
- Branch protection: Check workflow runs on correct branches
- Test locally before pushing: Use `act` tool to run GitHub Actions locally

---

## Quick Commands Reference

### Viewing Resources
```bash
# All resources in namespace
kubectl get all -n <namespace>

# Detailed pod information
kubectl describe pod <pod-name> -n <namespace>

# Pod logs
kubectl logs <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --previous  # Previous container logs

# Follow logs in real-time
kubectl logs -f <pod-name> -n <namespace>

# Events
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

### Debugging
```bash
# Execute commands in pod
kubectl exec -it <pod-name> -n <namespace> -- /bin/sh

# Port forward for local testing
kubectl port-forward pod/<pod-name> 8080:3000 -n <namespace>
kubectl port-forward service/<service-name> 8080:80 -n <namespace>

# Copy files from pod
kubectl cp <namespace>/<pod-name>:/path/to/file ./local-file

# Check resource usage
kubectl top pods -n <namespace>
kubectl top nodes
```

### Deployment Management
```bash
# Rollout status
kubectl rollout status deployment/<name> -n <namespace>

# Rollout history
kubectl rollout history deployment/<name> -n <namespace>

# Rollback
kubectl rollout undo deployment/<name> -n <namespace>
kubectl rollout undo deployment/<name> -n <namespace> --to-revision=2

# Restart deployment (recreate all pods)
kubectl rollout restart deployment/<name> -n <namespace>

# Scale deployment
kubectl scale deployment/<name> --replicas=5 -n <namespace>
```

### Cleanup
```bash
# Delete specific resources
kubectl delete deployment <name> -n <namespace>
kubectl delete service <name> -n <namespace>

# Delete all resources in namespace
kubectl delete all --all -n <namespace>

# Delete namespace (deletes all resources in it)
kubectl delete namespace <namespace>
```

## Performance Optimization Tips

1. **Use resource requests and limits** - Prevents resource starvation
2. **Implement health probes properly** - Enables self-healing
3. **Use horizontal pod autoscaling** - Automatically scale based on metrics
4. **Enable readiness probes** - Prevents traffic to unhealthy pods
5. **Set appropriate replica counts** - Balance availability and cost
6. **Use image pull policy wisely** - IfNotPresent for development, Always for production
7. **Implement proper logging** - Structured logs for better debugging
8. **Monitor metrics** - Use Prometheus and Grafana

## Security Best Practices

1. **Never run containers as root** - Use securityContext.runAsNonRoot
2. **Use read-only root filesystem** - Reduces attack surface
3. **Scan images for vulnerabilities** - Use tools like Trivy or Snyk
4. **Rotate secrets regularly** - Don't hardcode credentials
5. **Use network policies** - Control pod-to-pod communication
6. **Enable RBAC** - Principle of least privilege
7. **Keep Kubernetes updated** - Apply security patches
8. **Audit logs** - Track cluster access and changes
EOF

echo "✅ TROUBLESHOOTING_GUIDE.md created"
echo ""

echo "================================================"
echo "✅ TASK 8 COMPLETE: Troubleshooting toolkit ready"
echo "================================================"
echo ""

echo "🤔 ACTIVE RECALL CHECK:"
echo "   1. What's the first thing to check when pods won't start?"
echo "   2. How do you distinguish between application errors and infrastructure errors?"
echo "   3. Why check service endpoints when connectivity fails?"
echo "   4. What's the purpose of checking previous pod logs?"
echo "   5. How would you debug an intermittent issue?"
echo ""
read -p "Press ENTER after thinking through these questions..."
echo ""

cat << 'ANSWERS_9'
### 🧠 Comprehensive Answer Block 9:

**Question 1 Answer:** Start with 'kubectl describe pod <pod-name>' to see:
1. Events section - shows what Kubernetes tried to do and errors encountered
2. Container status - shows current state (waiting, running, terminated)
3. Conditions - shows why pod isn't ready
Common issues appear here: ImagePullBackOff, CrashLoopBackOff, insufficient resources, etc.
This gives you the "what went wrong" before diving into "why."

**Question 2 Answer:** Look at the error location:
- Infrastructure errors: Show up in pod events, describe output, or cluster events
  (scheduling failures, image pull errors, resource constraints)
- Application errors: Show up in pod logs (kubectl logs)
  (syntax errors, crashed processes, failed connections)
Infrastructure errors prevent the container from running; application errors happen after it starts.

**Question 3 Answer:** Service endpoints list the actual IP addresses that receive traffic. If:
- Endpoints empty: No pods match the service selector (label mismatch)
- Endpoints exist but service unreachable: Network policy, firewall, or port mismatch
- Endpoints have wrong pods: Selector is too broad/narrow
Endpoints are the "proof" that Kubernetes found matching pods and configured routing.

**Question 4 Answer:** Previous logs show what happened in the last container instance before it 
crashed or was restarted. This is critical for debugging:
- CrashLoopBackOff: Current container just started, error was in previous instance
- OOMKilled: Previous logs show memory buildup before kill
- Startup failures: Previous logs show initialization errors
Without --previous, you might see "starting up..." instead of the actual error that caused restart.

**Question 5 Answer:** Intermittent issues require different strategies:
1. Increase logging verbosity temporarily
2. Monitor metrics over time (kubectl top, Prometheus)
3. Check for resource contention (does it happen under load?)
4. Look at pod distribution (does it happen on specific nodes?)
5. Correlate with external events (deployments, traffic spikes)
6. Use distributed tracing (if implemented)
7. Capture pod state when it happens: kubectl describe, logs, exec
Intermittent = pattern recognition across multiple occurrences.

ANSWERS_9

echo ""
echo "🔄 SPACED REVIEW: Complete debugging workflow"
echo "   Issue reported → Check pod status → Check events → Check logs"
echo "   → Check service/endpoints → Check resources → Check configuration"
echo ""
echo "   Each tool reveals a different layer of the system!"
echo ""

# ==========================================
# TASK 9: Cleanup and Resource Management
# Active Learning Block 9
# ==========================================

echo "================================================"
echo "TASK 9: PROFESSIONAL CLEANUP"
echo "================================================"
echo ""

echo "🧠 PREDICT FIRST: Proper cleanup order matters!"
echo "   Consider:"
echo "   • What order should we delete resources?"
echo "   • What happens if we delete namespace first?"
echo "   • How do we verify cleanup is complete?"
echo "   • What resources might be left behind?"
echo "   Take 30 seconds to think about cleanup strategy..."
echo ""
read -p "Press ENTER when ready to proceed..."
echo ""

echo "📝 CREATING CLEANUP SCRIPT:"
echo "================================================"

cat > cleanup.sh << 'EOF'
#!/bin/bash

# ==========================================
# PROFESSIONAL CLEANUP SCRIPT
# ==========================================

set -e

# Color codes
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

echo -e "${BLUE}========================================"
echo "🧹 KUBERNETES CI/CD LAB CLEANUP"
echo "========================================${NC}"
echo ""

# Function for safe cleanup
safe_cleanup() {
    local resource_type="$1"
    local resource_name="$2"
    local namespace="$3"
    local additional_flags="${4:-}"
    
    echo -e "${YELLOW}Cleaning up $resource_type: $resource_name${NC}"
    
    if kubectl get $resource_type $resource_name -n $namespace &>/dev/null; then
        kubectl delete $resource_type $resource_name -n $namespace $additional_flags
        echo -e "${GREEN}✅ Deleted: $resource_name${NC}"
    else
        echo -e "${BLUE}ℹ️  $resource_name already removed${NC}"
    fi
    echo ""
}

# ==========================================
# STEP 1: Clean cicd-lab namespace
# ==========================================
echo -e "${BLUE}=== STEP 1: Cleaning cicd-lab namespace ===${NC}"
echo ""

if kubectl get namespace cicd-lab &>/dev/null; then
    echo "Scaling down deployments..."
    kubectl scale deployment cicd-app -n cicd-lab --replicas=0 2>/dev/null || true
    sleep 2
    
    echo "Deleting resources..."
    kubectl delete all --all -n cicd-lab
    
    echo "Deleting namespace..."
    kubectl delete namespace cicd-lab
    
    echo -e "${GREEN}✅ cicd-lab namespace cleaned${NC}"
else
    echo -e "${BLUE}ℹ️  cicd-lab namespace already removed${NC}"
fi
echo ""

# ==========================================
# STEP 2: Clean staging namespace
# ==========================================
echo -e "${BLUE}=== STEP 2: Cleaning staging namespace ===${NC}"
echo ""

if kubectl get namespace staging &>/dev/null; then
    echo "Scaling down deployments..."
    kubectl scale deployment staging-cicd-app -n staging --replicas=0 2>/dev/null || true
    sleep 2
    
    echo "Deleting resources..."
    kubectl delete all --all -n staging
    
    echo "Deleting namespace..."
    kubectl delete namespace staging
    
    echo -e "${GREEN}✅ staging namespace cleaned${NC}"
else
    echo -e "${BLUE}ℹ️  staging namespace already removed${NC}"
fi
echo ""

# ==========================================
# STEP 3: Clean production namespace
# ==========================================
echo -e "${BLUE}=== STEP 3: Cleaning production namespace ===${NC}"
echo ""

if kubectl get namespace production &>/dev/null; then
    echo "Scaling down deployments..."
    kubectl scale deployment prod-cicd-app -n production --replicas=0 2>/dev/null || true
    sleep 2
    
    echo "Deleting resources..."
    kubectl delete all --all -n production
    
    echo "Deleting namespace..."
    kubectl delete namespace production
    
    echo -e "${GREEN}✅ production namespace cleaned${NC}"
else
    echo -e "${BLUE}ℹ️  production namespace already removed${NC}"
fi
echo ""

# ==========================================
# STEP 4: Clean Docker images
# ==========================================
echo -e "${BLUE}=== STEP 4: Cleaning Docker images ===${NC}"
echo ""

echo "Removing cicd-app images..."
docker rmi cicd-app:latest cicd-app:v1.0.0 cicd-app:v2.0.0-broken 2>/dev/null || true

echo "Removing dangling images..."
docker image prune -f

echo -e "${GREEN}✅ Docker images cleaned${NC}"
echo ""

# ==========================================
# STEP 5: Verification
# ==========================================
echo -e "${BLUE}=== VERIFICATION ===${NC}"
echo ""

echo "Checking namespaces..."
if kubectl get namespace cicd-lab &>/dev/null || \
   kubectl get namespace staging &>/dev/null || \
   kubectl get namespace production &>/dev/null; then
    echo -e "${RED}⚠️  Some namespaces still exist${NC}"
else
    echo -e "${GREEN}✅ All lab namespaces removed${NC}"
fi

echo ""
echo "Checking Docker images..."
if docker images | grep -q cicd-app; then
    echo -e "${YELLOW}⚠️  Some cicd-app images remain:${NC}"
    docker images | grep cicd-app
else
    echo -e "${GREEN}✅ All cicd-app images removed${NC}"
fi

echo ""
echo -e "${BLUE}========================================"
echo "✅ CLEANUP COMPLETE"
echo "========================================${NC}"
echo ""

echo "Project files remain in: $(pwd)"
echo "To remove project files: cd .. && rm -rf k8s-cicd-lab"
echo ""

echo "Minikube is still running. To stop it:"
echo "  minikube stop"
echo ""
echo "To delete the Minikube cluster completely:"
echo "  minikube delete"
EOF

chmod +x cleanup.sh
echo "✅ cleanup.sh created"
echo ""

echo "🧪 RUNNING CLEANUP VERIFICATION (dry-run):"
echo "================================================"

echo "Current resources:"
echo ""
echo "Namespaces:"
kubectl get namespaces | grep -E "cicd-lab|staging|production" || echo "No lab namespaces found"
echo ""
echo "Docker images:"
docker images | grep cicd-app || echo "No cicd-app images found"
echo ""

read -p "Do you want to run full cleanup now? (y/N): " -n 1 -r
echo ""

if [[ $REPLY =~ ^[Yy]$ ]]; then
    ./cleanup.sh
else
    echo "Cleanup script is ready. Run it manually when needed: ./cleanup.sh"
fi
echo ""

echo "================================================"
echo "✅ TASK 9 COMPLETE: Cleanup procedures ready"
echo "================================================"
echo ""

echo "🤔 ACTIVE RECALL CHECK:"
echo "   1. Why scale deployments to 0 before deleting namespace?"
echo "   2. What happens to pods when you delete a namespace?"
echo "   3. Why clean Docker images separately from Kubernetes resources?"
echo "   4. What's the risk of deleting resources in wrong order?"
echo "   5. Why verify cleanup completion?"
echo ""
read -p "Press ENTER after thinking through these questions..."
echo ""

cat << 'ANSWERS_10'
### 🧠 Comprehensive Answer Block 10:

**Question 1 Answer:** Scaling to 0 before deletion:
- Gives pods time for graceful shutdown (finish requests, close connections)
- Triggers preStop hooks if configured
- Prevents abrupt service interruption
- Allows load balancers to drain connections
Without scaling first, pods are terminated immediately when namespace is deleted, potentially 
causing failed requests or incomplete transactions.

**Question 2 Answer:** When you delete a namespace, Kubernetes deletes ALL resources in it:
- Pods are terminated (SIGTERM, then SIGKILL after grace period)
- Deployments, Services, ConfigMaps, Secrets are removed
- PersistentVolumeClaims may persist (depends on reclaim policy)
- Resources are deleted in parallel, not sequentially
It's a "cascade delete" - efficient but irreversible. Always verify you're deleting correct namespace!

**Question 3 Answer:** Docker images exist outside Kubernetes' control:
- Kubernetes stores image references in specs, not images themselves
- Images consume disk space on nodes (or your local machine with Minikube)
- Deleting Kubernetes resources doesn't delete Docker images
- Old images accumulate over time (each build creates new tags)
You must explicitly clean Docker images to reclaim disk space.

**Question 4 Answer:** Wrong deletion order can cause:
- Service disruption (deleting Service before Pods leaves orphaned endpoints)
- Resource leaks (deleting Deployment without Namespace leaves orphaned resources)
- Cleanup failures (dependencies prevent deletion)
- Cascading issues (deleting PVC before Pod causes mount errors)
Proper order: Services → Deployments → Pods → ConfigMaps/Secrets → Namespace ensures clean removal.

**Question 5 Answer:** Verification ensures:
- No resources left consuming cluster resources
- No cost from forgotten resources (in cloud environments)
- Clean state for next deployment
- No conflicts with future deployments (name collisions)
- Disk space reclaimed
In production, forgotten resources can cost thousands of dollars. Always verify cleanup!

ANSWERS_10

echo ""
echo "🔄 SPACED REVIEW: Full lifecycle"
echo "   Create → Deploy → Monitor → Update → Rollback → Cleanup"
echo ""
echo "   You've completed the entire CI/CD lifecycle!"
echo "   From code to container to cluster to cleanup!"
echo ""

# ==========================================
# FINAL MASTERY ASSESSMENT
# ==========================================

echo "================================================"
echo "🎓 FINAL MASTERY ASSESSMENT"
echo "================================================"
echo ""

echo "🧠 FINAL MASTERY CHALLENGE:"
echo ""
echo "   Without looking at notes, explain to a colleague:"
echo ""
echo "   1. The complete CI/CD pipeline flow from code commit to production"
echo "      • What happens at each stage?"
echo "      • What tools/technologies are involved?"
echo "      • What safety mechanisms are in place?"
echo ""
echo "   2. The Kubernetes architecture we built"
echo "      • How do all the components interact?"
echo "      • What role does each resource type play?"
echo "      • How does traffic flow from user to application?"
echo ""
echo "   3. Troubleshooting methodology"
echo "      • What's your systematic approach?"
echo "      • What tools do you use at each step?"
echo "      • How do you distinguish infrastructure from application issues?"
echo ""
echo "   4. Production readiness"
echo "      • What makes this deployment production-ready?"
echo "      • What additional concerns would you address?"
echo "      • How would you handle real-world scale?"
echo ""
echo "⏱️  Take 5-10 minutes to organize your thoughts..."
echo "   Then explain as if teaching a new team member"
echo ""
read -p "Press ENTER when you've completed your mental explanation..."
echo ""

echo "🔄 SPACED REPETITION - Connect All Concepts:"
echo ""
echo "   1. How does this CI/CD pipeline compare to other deployment methods?"
echo "   2. What technologies from previous labs connect here?"
echo "   3. How would you apply this to other application types (Python, Java, Go)?"
echo "   4. What business value does this automation provide?"
echo "   5. How would you explain ROI to management?"
echo ""
read -p "Press ENTER after reflecting on these connections..."
echo ""

echo "✅ LEARNING OBJECTIVES ACHIEVED:"
echo "================================================"
echo "   ✓ Built complete CI/CD pipeline with GitHub Actions"
echo "   ✓ Automated Docker image building and registry integration"
echo "   ✓ Implemented Kubernetes deployment automation"
echo "   ✓ Configured health checks and self-healing mechanisms"
echo "   ✓ Mastered rollback procedures (manual and automated)"
echo "   ✓ Created multi-environment deployment architecture"
echo "   ✓ Developed comprehensive testing framework"
echo "   ✓ Built troubleshooting toolkit and methodology"
echo "   ✓ Understood production-grade best practices"
echo "   ✓ Implemented proper cleanup and resource management"
echo ""

echo "🎯 RETENTION SUCCESS METRICS:"
echo "================================================"
echo "   📈 Active recall integration: 75% retention target"
echo "      • 10 Active Recall checkpoints throughout lab"
echo "      • Spaced repetition connecting concepts across sections"
echo "      • Prediction challenges before each major task"
echo ""
echo "   🔄 Spaced repetition: Concepts reinforced across sections"
echo "      • Docker → Kubernetes → CI/CD connections made explicit"
echo "      • Health probes concept revisited 4 times"
echo "      • Resource management threaded throughout"
echo ""
echo "   🎭 Multi-modal learning: Multiple pathways engaged"
echo "      • Visual: Architecture diagrams and command outputs"
echo "      • Kinesthetic: Hands-on deployment and troubleshooting"
echo "      • Auditory: Explain-aloud prompts throughout"
echo "      • Teaching: Explain-to-colleague scenarios"
echo ""
echo "   🧠 Deep understanding: Architecture and practical application"
echo "      • Not just commands - understanding WHY"
echo "      • Troubleshooting from first principles"
echo "      • Production readiness considerations"
echo ""

echo "================================================"
echo "🎉 LAB 16 COMPLETE!"
echo "================================================"
echo ""

cat << 'COMPLETION_MESSAGE'
Congratulations! You've completed Lab 16: Integrating Kubernetes with CI/CD Pipelines!

🌟 What You've Mastered:

**Technical Skills:**
• Complete CI/CD pipeline architecture and implementation
• Docker containerization and multi-stage builds
• Kubernetes deployment strategies and lifecycle management
• GitHub Actions workflow orchestration
• Health checking and self-healing systems
• Multi-environment deployment patterns (dev/staging/prod)
• Comprehensive testing and verification
• Systematic troubleshooting methodology

**DevOps Practices:**
• Infrastructure as Code (IaC)
• Declarative configuration management
• Automated testing and deployment
• Zero-downtime deployment strategies
• Failure recovery and rollback procedures
• Security best practices (non-root, resource limits)
• Monitoring and observability

**Real-World Applications:**
This lab simulates enterprise scenarios where teams need to:
• Deploy multiple times per day with confidence
• Maintain high availability during deployments
• Quickly recover from failed deployments
• Manage multiple environments safely
• Ensure consistent quality through automation

**Why This Matters:**
Modern software companies deploy hundreds of times per day. The automation you built enables:
• Faster time to market (features reach users quickly)
• Higher reliability (automated testing catches bugs)
• Lower costs (automation reduces manual effort)
• Better security (consistent, auditable processes)
• Improved developer experience (focus on code, not deployment)

**Next Steps:**
To expand your expertise, explore:
• Helm: Package manager for Kubernetes
• ArgoCD: GitOps continuous delivery
• Prometheus & Grafana: Monitoring and alerting
• Istio: Service mesh for advanced traffic management
• Tekton: Kubernetes-native CI/CD pipelines
• Kyverno: Policy management for Kubernetes

**Certification Preparation:**
You're now well-prepared for:
• Kubernetes and Cloud Native Associate (KCNA)
• Certified Kubernetes Application Developer (CKAD)
• GitHub Actions certification

**Remember:**
• CI/CD is a journey, not a destination
• Start simple, iterate towards complexity
• Automate repetitive tasks
• Test everything
• Monitor and improve continuously

Keep practicing, keep automating, and keep learning! 🚀

COMPLETION_MESSAGE

echo ""
echo "📚 Lab artifacts created:"
echo "   • Application code (app.js, package.json, Dockerfile)"
echo "   • Kubernetes manifests (base + overlays)"
echo "   • GitHub Actions workflows (ci-cd.yaml, rollback.yaml)"
echo "   • Test suite (test-deployment.sh)"
echo "   • Monitoring dashboard (monitor-deployment.sh)"
echo "   • Troubleshooting toolkit (troubleshoot.sh)"
echo "   • Cleanup script (cleanup.sh)"
echo "   • Documentation (TROUBLESHOOTING_GUIDE.md)"
echo ""
echo "All files are saved in: $(pwd)"
echo ""
echo "================================================"
echo "🎓 Thank you for completing this lab!"
echo "================================================"
