# Lab 7: Mastering Kubernetes Building Blocks - The Complete Guide

## 🎯 Learning Objectives

By completing this comprehensive lab, you will:
- **Master Pod deployment** using YAML manifests with deep understanding of each field
- **Create and manage Deployments** with advanced scaling strategies and rollout controls
- **Understand architectural differences** between Pods, ReplicaSets, and Deployments
- **Implement ConfigMaps and Secrets** for production-grade configuration management
- **Master kubectl commands** with advanced querying and troubleshooting techniques
- **Apply production best practices** including health checks, resource management, and security
- **Troubleshoot common issues** with systematic debugging approaches

## 📚 Core Concepts Explained

### What is a Pod?
A **Pod** is the smallest deployable unit in Kubernetes, representing a group of one or more containers that:
- Share the same network namespace (IP address and port space)
- Share storage volumes
- Are scheduled together on the same node
- Live and die together (shared lifecycle)

**Think of it as**: A Pod is like an apartment where roommates (containers) share the same address, utilities, and storage space.

### What is a Deployment?
A **Deployment** is a higher-level abstraction that manages:
- ReplicaSets (which manage Pods)
- Rolling updates and rollbacks
- Scaling operations
- Self-healing capabilities

**Think of it as**: A Deployment is like a property manager who ensures the right number of apartments (Pods) are always available and handles renovations (updates) smoothly.

### What is a ConfigMap?
A **ConfigMap** is a Kubernetes object used to store non-confidential configuration data in key-value pairs, allowing you to:
- Separate configuration from application code
- Update configuration without rebuilding images
- Share configuration across multiple Pods

**Think of it as**: A ConfigMap is like a bulletin board where you post configuration notices that all apartments (Pods) can read.

## 🔧 Prerequisites Verification

- Docker basics and containerization concepts
- YAML syntax understanding
- Linux command line proficiency
- Basic networking concepts (ports, IPs)
- Environment variables knowledge

## 🚀 Enhanced Lab Environment Setup

### Complete Environment Initialization

```bash
# Create lab directory structure
mkdir -p ~/k8s-lab7/{manifests,configs,scripts,logs}
cd ~/k8s-lab7

echo "================================================"
echo "📁 Lab directory structure created successfully!"
echo "================================================"
echo

# Set convenient aliases for the lab
echo "alias k='kubectl'" >> ~/.bashrc
echo "alias kgp='kubectl get pods'" >> ~/.bashrc
echo "alias kgs='kubectl get svc'" >> ~/.bashrc
echo "alias kgd='kubectl get deployments'" >> ~/.bashrc
echo "alias kdp='kubectl describe pod'" >> ~/.bashrc
echo "alias kdd='kubectl describe deployment'" >> ~/.bashrc
echo "alias kaf='kubectl apply -f'" >> ~/.bashrc
echo "alias kdf='kubectl delete -f'" >> ~/.bashrc
echo "alias klo='kubectl logs'" >> ~/.bashrc
echo "alias kex='kubectl exec -it'" >> ~/.bashrc
source ~/.bashrc

echo "✅ Kubectl aliases configured for faster operations!"
echo
echo "💡 Learning Tip: Using aliases speeds up your workflow significantly."
echo "   Type 'alias' to see all configured shortcuts!"
echo

# Install required tools (ensures everything is available)
echo "================================================"
echo "🔧 Installing/Verifying Required Tools..."
echo "================================================"
echo

# Update package lists
sudo apt-get update -qq

# Install essential tools
sudo apt-get install -y curl wget git nano vim jq yamllint tree htop > /dev/null 2>&1

echo "✅ Essential tools installed/verified!"
echo

# Install kubectl if not present
if ! command -v kubectl &> /dev/null; then
    echo "📦 Installing kubectl..."
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    chmod +x kubectl
    sudo mv kubectl /usr/local/bin/
    echo "✅ kubectl installed successfully!"
else
    echo "✅ kubectl is already installed: $(kubectl version --client --short 2>/dev/null)"
fi
echo

# Install minikube if not present
if ! command -v minikube &> /dev/null; then
    echo "📦 Installing minikube..."
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    rm minikube-linux-amd64
    echo "✅ minikube installed successfully!"
else
    echo "✅ minikube is already installed: $(minikube version --short)"
fi
echo

# Start minikube if not running
if ! minikube status &> /dev/null; then
    echo "🚀 Starting minikube cluster..."
    minikube start --driver=docker --cpus=2 --memory=4096
    echo "✅ Minikube cluster started!"
else
    echo "✅ Minikube is already running!"
fi
echo

# Enable useful addons
echo "🔌 Enabling minikube addons..."
minikube addons enable metrics-server
minikube addons enable dashboard
echo "✅ Addons enabled!"
echo

# Verify cluster health
echo "================================================"
echo "🏥 Verifying Cluster Health..."
echo "================================================"
echo

kubectl cluster-info

echo
kubectl get nodes -o wide

echo
echo "💡 Learning Tip: The '-o wide' flag shows additional information like"
echo "   internal IPs, kernel version, and container runtime!"
echo

# Check system resources
echo "================================================"
echo "📊 Checking Available Resources..."
echo "================================================"
echo

kubectl top nodes 2>/dev/null || echo "⚠️  Metrics server is still initializing..."

echo
kubectl describe nodes | grep -A 5 "Allocated resources:"

echo
echo "💡 Learning Tip: Understanding node capacity helps prevent resource"
echo "   exhaustion and Pod scheduling failures!"
echo
```

## 📝 Task 1: Advanced Pod Deployment with YAML Manifests

### Understanding Pod Architecture

```bash
echo "================================================"
echo "📚 Task 1: Mastering Pod Deployment"
echo "================================================"
echo
echo "🧠 Concept: A Pod is like a logical host for containers."
echo "   Multiple containers in a Pod share:"
echo "   - Network (same IP)"
echo "   - Storage volumes"
echo "   - IPC namespace"
echo "   - Process namespace (optionally)"
echo "================================================"
echo
```

### Subtask 1.1: Create a Basic Pod Manifest

```bash
# Navigate to manifests directory
cd ~/k8s-lab7/manifests

echo "📝 Creating basic Pod manifest..."
echo

# Create the Pod YAML using nano
nano simple-pod.yaml
```

**Add the following content to simple-pod.yaml:**

```yaml
# This is a Pod manifest - the fundamental unit of deployment in Kubernetes
apiVersion: v1                # API version for Pod objects
kind: Pod                     # The type of Kubernetes object
metadata:                     # Information about the Pod
  name: nginx-pod            # Unique name within the namespace
  namespace: default         # Namespace (isolated environment)
  labels:                    # Key-value pairs for organization
    app: nginx              # Application identifier
    environment: lab        # Environment designation
    version: "1.21"         # Version tracking
    tier: frontend          # Application tier
  annotations:              # Non-identifying metadata
    description: "Lab Pod for learning Kubernetes basics"
    owner: "student"
    created-by: "lab-exercise"
spec:                       # Desired state specification
  containers:              # List of containers in the Pod
  - name: nginx-container  # Container name (unique within Pod)
    image: nginx:1.21      # Docker image with tag
    imagePullPolicy: IfNotPresent  # When to pull the image
    ports:
    - containerPort: 80    # Port exposed by container
      protocol: TCP        # Network protocol
      name: http          # Port name for service discovery
    resources:            # Resource requirements and limits
      requests:           # Minimum guaranteed resources
        memory: "64Mi"    # 64 Megabytes of RAM
        cpu: "250m"       # 250 millicores (0.25 CPU)
      limits:             # Maximum allowed resources
        memory: "128Mi"   # Container killed if exceeds
        cpu: "500m"       # Container throttled if exceeds
    env:                  # Environment variables
    - name: POD_ENV
      value: "development"
    - name: LOG_LEVEL
      value: "info"
    readinessProbe:       # Check if container is ready
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:        # Check if container is alive
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 15
      periodSeconds: 20
    lifecycle:            # Lifecycle hooks
      preStop:
        exec:
          command: ["/bin/sh", "-c", "nginx -s quit && sleep 15"]
  restartPolicy: Always   # What to do when container exits
  dnsPolicy: ClusterFirst # DNS resolution policy
  hostname: nginx-pod     # Hostname within Pod network
```

**Save and exit nano** (Ctrl+X, Y, Enter)

```bash
echo "✅ Pod manifest created!"
echo
echo "💡 Learning Tip: Every field in the manifest serves a specific purpose:"
echo "   - metadata: Identifies and organizes resources"
echo "   - spec: Defines desired state"
echo "   - resources: Prevents resource starvation"
echo "   - probes: Ensures availability and health"
echo
```

### Subtask 1.2: Validate and Deploy the Pod

```bash
echo "================================================"
echo "🔍 Validating YAML Syntax..."
echo "================================================"
echo

# Validate YAML syntax
yamllint simple-pod.yaml || echo "⚠️  Minor style warnings (can be ignored)"

echo
echo "📋 Dry-run to validate manifest..."
kubectl apply -f simple-pod.yaml --dry-run=client -o yaml | head -20

echo
echo "💡 Learning Tip: --dry-run=client validates manifests without"
echo "   actually creating resources. Perfect for testing!"
echo

echo "================================================"
echo "🚀 Deploying the Pod..."
echo "================================================"
echo

# Apply the manifest
kubectl apply -f simple-pod.yaml

echo
echo "⏳ Waiting for Pod to be ready..."
kubectl wait --for=condition=Ready pod/nginx-pod --timeout=60s

echo
echo "💡 Learning Tip: The 'wait' command blocks until conditions are met."
echo "   Great for scripting and automation!"
echo
```

### Subtask 1.3: Comprehensive Pod Inspection

```bash
echo "================================================"
echo "🔍 Comprehensive Pod Inspection"
echo "================================================"
echo

# Basic Pod information
echo "📊 Pod Status:"
kubectl get pod nginx-pod -o wide

echo
echo "📝 Pod Details (Condensed):"
kubectl describe pod nginx-pod | grep -E "^Name:|^Status:|^IP:|^Node:|Events:" -A 5

echo
echo "📦 Pod in JSON format (partial):"
kubectl get pod nginx-pod -o json | jq '{name: .metadata.name, status: .status.phase, podIP: .status.podIP, conditions: .status.conditions}'

echo
echo "📈 Resource usage:"
kubectl top pod nginx-pod 2>/dev/null || echo "⚠️  Metrics still initializing..."

echo
echo "📜 Pod logs:"
kubectl logs nginx-pod --tail=10

echo
echo "🏷️  Pod labels:"
kubectl get pod nginx-pod --show-labels

echo
echo "💡 Learning Tip: Understanding different output formats helps in:"
echo "   - Debugging (describe)"
echo "   - Automation (json/yaml)"
echo "   - Quick checks (get with -o wide)"
echo "   - Monitoring (top)"
echo
```

### Subtask 1.4: Advanced Pod Interaction

```bash
echo "================================================"
echo "🔧 Advanced Pod Interactions"
echo "================================================"
echo

# Test Pod connectivity
echo "🌐 Testing Pod connectivity..."
POD_IP=$(kubectl get pod nginx-pod -o jsonpath='{.status.podIP}')
echo "Pod IP: $POD_IP"
echo

# Create a test Pod for network testing
echo "Creating network test Pod..."
nano test-pod.yaml
```

**Add the following to test-pod.yaml:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  labels:
    purpose: network-testing
spec:
  containers:
  - name: busybox
    image: busybox:1.35
    command: ['sh', '-c', 'sleep 3600']
```

**Save and exit nano**

```bash
kubectl apply -f test-pod.yaml

echo "⏳ Waiting for test Pod..."
kubectl wait --for=condition=Ready pod/test-pod --timeout=30s

echo
echo "🔌 Testing connectivity from test-pod to nginx-pod..."
kubectl exec test-pod -- wget -q -O- http://$POD_IP | head -5

echo
echo "💡 Learning Tip: Pods can communicate directly using their IP addresses"
echo "   within the cluster network. This is the foundation of service discovery!"
echo

# Execute commands in the Pod
echo "================================================"
echo "🖥️  Executing Commands Inside the Pod"
echo "================================================"
echo

echo "📁 Listing nginx configuration files:"
kubectl exec nginx-pod -- ls -la /etc/nginx/

echo
echo "📊 Checking process list:"
kubectl exec nginx-pod -- ps aux

echo
echo "🔍 Checking environment variables we set:"
kubectl exec nginx-pod -- env | grep -E "POD_ENV|LOG_LEVEL"

echo
echo "💡 Learning Tip: 'kubectl exec' is your gateway into containers."
echo "   Use -it flags for interactive sessions!"
echo
```

## 📝 Task 2: Mastering Deployments for Production-Grade Applications

### Understanding Deployments vs Pods

```bash
echo "================================================"
echo "📚 Task 2: Mastering Deployments"
echo "================================================"
echo
echo "🧠 Concept: Deployment Architecture"
echo "   Deployment -> ReplicaSet -> Pods"
echo ""
echo "   Benefits over standalone Pods:"
echo "   ✅ Self-healing (replaces failed Pods)"
echo "   ✅ Scaling (horizontal pod autoscaling)"
echo "   ✅ Rolling updates (zero-downtime deployments)"
echo "   ✅ Rollback capability (version history)"
echo "   ✅ Declarative updates (GitOps friendly)"
echo "================================================"
echo
```

### Subtask 2.1: Create an Advanced Deployment

```bash
cd ~/k8s-lab7/manifests

echo "📝 Creating advanced Deployment manifest..."
nano nginx-deployment.yaml
```

**Add the following to nginx-deployment.yaml:**

```yaml
# Advanced Deployment manifest with production-ready configurations
apiVersion: apps/v1           # API version for Deployment
kind: Deployment              # Object type
metadata:
  name: nginx-deployment      # Deployment name
  namespace: default
  labels:
    app: nginx
    tier: frontend
    version: "1.21"
  annotations:
    description: "Production-grade nginx deployment"
    kubernetes.io/change-cause: "Initial deployment of nginx 1.21"
spec:
  replicas: 3                 # Number of Pod replicas
  revisionHistoryLimit: 10    # Number of old ReplicaSets to keep
  progressDeadlineSeconds: 600 # Max time for deployment to make progress
  strategy:                   # Deployment strategy
    type: RollingUpdate       # RollingUpdate or Recreate
    rollingUpdate:
      maxSurge: 1            # Max pods above desired replicas during update
      maxUnavailable: 1      # Max pods that can be unavailable during update
  selector:                   # How Deployment finds Pods to manage
    matchLabels:
      app: nginx
      tier: frontend
  template:                   # Pod template (same as Pod spec)
    metadata:
      labels:                # Labels must match selector
        app: nginx
        tier: frontend
        version: "1.21"
      annotations:
        prometheus.io/scrape: "true"  # Example annotation for monitoring
        prometheus.io/port: "80"
    spec:
      affinity:              # Pod affinity rules
        podAntiAffinity:     # Spread pods across nodes
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - nginx
              topologyKey: kubernetes.io/hostname
      containers:
      - name: nginx
        image: nginx:1.21
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        env:
        - name: DEPLOYMENT_ENV
          value: "production"
        - name: REPLICA_ID
          valueFrom:
            fieldRef:
              fieldPath: metadata.name  # Pod name
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName   # Node where Pod is running
        readinessProbe:
          httpGet:
            path: /
            port: 80
            scheme: HTTP
          initialDelaySeconds: 5
          periodSeconds: 10
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 3
        livenessProbe:
          httpGet:
            path: /
            port: 80
            scheme: HTTP
          initialDelaySeconds: 15
          periodSeconds: 20
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 3
        startupProbe:          # Gives time for slow-starting containers
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 0
          periodSeconds: 10
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 30
        volumeMounts:          # Mount volumes into container
        - name: cache-volume
          mountPath: /cache
        - name: config-volume
          mountPath: /etc/config
      volumes:                 # Volume definitions
      - name: cache-volume
        emptyDir:             # Temporary directory
          sizeLimit: 1Gi
      - name: config-volume
        emptyDir: {}
      terminationGracePeriodSeconds: 30  # Time to gracefully shutdown
      dnsPolicy: ClusterFirst
      restartPolicy: Always
```

**Save and exit nano**

```bash
echo "✅ Advanced Deployment manifest created!"
echo
echo "💡 Learning Tip: This production-grade manifest includes:"
echo "   - Anti-affinity: Spreads pods across nodes for HA"
echo "   - Multiple probes: Ensures proper health checking"
echo "   - Resource limits: Prevents resource exhaustion"
echo "   - Volume mounts: For caching and configuration"
echo "   - Rolling update strategy: Zero-downtime deployments"
echo
```

### Subtask 2.2: Deploy and Monitor the Deployment

```bash
echo "================================================"
echo "🚀 Deploying the Application"
echo "================================================"
echo

# Deploy the application
kubectl apply -f nginx-deployment.yaml

echo
echo "📊 Monitoring deployment rollout..."
kubectl rollout status deployment/nginx-deployment

echo
echo "🔍 Deployment details:"
kubectl get deployment nginx-deployment -o wide

echo
echo "📦 ReplicaSet created by deployment:"
kubectl get replicaset -l app=nginx

echo
echo "🎯 Pods created by deployment:"
kubectl get pods -l app=nginx -o wide

echo
echo "📈 Deployment events:"
kubectl get events --field-selector involvedObject.name=nginx-deployment --sort-by='.lastTimestamp' | tail -5

echo
echo "💡 Learning Tip: A Deployment creates a ReplicaSet, which creates Pods."
echo "   This hierarchy enables rolling updates and rollbacks!"
echo
```

### Subtask 2.3: Advanced Scaling Operations

```bash
echo "================================================"
echo "⚡ Advanced Scaling Operations"
echo "================================================"
echo

# Manual scaling
echo "📈 Scaling to 5 replicas..."
kubectl scale deployment nginx-deployment --replicas=5

echo
echo "⏳ Waiting for scale operation..."
kubectl wait --for=condition=Progressing deployment/nginx-deployment --timeout=60s

echo
echo "📊 Current replica status:"
kubectl get deployment nginx-deployment -o jsonpath='{.status}' | jq '.'

echo
echo "🎯 Pod distribution across nodes:"
kubectl get pods -l app=nginx -o wide | awk '{print $7}' | sort | uniq -c

echo
echo "💡 Learning Tip: Pods are distributed across nodes based on"
echo "   resource availability and affinity rules!"
echo

# Create Horizontal Pod Autoscaler
echo "================================================"
echo "🤖 Setting up Horizontal Pod Autoscaler (HPA)"
echo "================================================"
echo

nano hpa.yaml
```

**Add the following to hpa.yaml:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 60
```

**Save and exit nano**

```bash
kubectl apply -f hpa.yaml

echo
echo "📊 HPA Status:"
kubectl get hpa nginx-hpa

echo
echo "💡 Learning Tip: HPA automatically scales based on CPU/memory usage."
echo "   It's essential for handling variable workloads!"
echo
```

### Subtask 2.4: Rolling Updates and Rollbacks

```bash
echo "================================================"
echo "🔄 Performing Rolling Updates"
echo "================================================"
echo

# Record the update for rollback capability
echo "📦 Updating nginx from 1.21 to 1.22..."
kubectl set image deployment/nginx-deployment nginx=nginx:1.22 --record

echo
echo "📊 Watching rollout status..."
kubectl rollout status deployment/nginx-deployment

echo
echo "📜 Rollout history:"
kubectl rollout history deployment/nginx-deployment

echo
echo "🔍 Checking updated pods:"
kubectl get pods -l app=nginx -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[0].image}{"\n"}{end}'

echo
echo "💡 Learning Tip: Rolling updates replace pods gradually,"
echo "   ensuring zero downtime during updates!"
echo

# Simulate a problematic update
echo "================================================"
echo "⚠️  Simulating Problematic Update"
echo "================================================"
echo

echo "📦 Updating to a non-existent image..."
kubectl set image deployment/nginx-deployment nginx=nginx:9.99-does-not-exist --record

echo
echo "⏳ Waiting 20 seconds to observe failure..."
sleep 20

echo
echo "❌ Rollout status (should show issues):"
kubectl rollout status deployment/nginx-deployment --timeout=10s || true

echo
echo "🔍 Checking pod status:"
kubectl get pods -l app=nginx | grep -E "ImagePull|ErrImagePull|Pending" || echo "Some pods may still be running old version"

echo
echo "🔄 Rolling back to previous version..."
kubectl rollout undo deployment/nginx-deployment

echo
echo "✅ Rollback status:"
kubectl rollout status deployment/nginx-deployment

echo
echo "📜 Updated rollout history:"
kubectl rollout history deployment/nginx-deployment

echo
echo "💡 Learning Tip: Kubernetes keeps rollout history, allowing"
echo "   quick rollbacks when updates fail!"
echo
```

## 📝 Task 3: Advanced Configuration Management

### Understanding ConfigMaps and Secrets

```bash
echo "================================================"
echo "📚 Task 3: Configuration Management Excellence"
echo "================================================"
echo
echo "🧠 Concept: Configuration Management"
echo "   ConfigMaps: Non-sensitive configuration data"
echo "   Secrets: Sensitive data (base64 encoded, not encrypted!)"
echo ""
echo "   Mounting methods:"
echo "   1. Environment variables"
echo "   2. Volume mounts (files)"
echo "   3. Command-line arguments"
echo "================================================"
echo
```

### Subtask 3.1: Create Multiple ConfigMaps

```bash
cd ~/k8s-lab7/configs

echo "📝 Creating application ConfigMap..."
nano app-config.yaml
```

**Add the following to app-config.yaml:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
  labels:
    app: myapp
    environment: production
data:
  # Database configuration
  database.host: "mysql.production.svc.cluster.local"
  database.port: "3306"
  database.name: "production_db"
  database.pool_size: "20"
  
  # Application settings
  app.name: "MyAwesomeApp"
  app.version: "2.1.0"
  app.environment: "production"
  app.debug: "false"
  app.timezone: "UTC"
  
  # Logging configuration
  log.level: "INFO"
  log.format: "json"
  log.output: "stdout"
  
  # Cache settings
  cache.enabled: "true"
  cache.ttl: "3600"
  cache.max_size: "1000"
  
  # Feature flags
  feature.new_ui: "true"
  feature.beta_features: "false"
  feature.maintenance_mode: "false"
  
  # API settings
  api.rate_limit: "1000"
  api.timeout: "30"
  api.retry_count: "3"
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: default
  labels:
    app: nginx
binaryData:
  # Binary data is base64 encoded automatically
  favicon.ico: |
    AAABAAEAEBAAAAEAIABoBAAAFgAAACgAAAAQAAAAIAAAAAEAIAAAAAAAAAAAAAAAAAAAAAAA
data:
  # Nginx configuration file
  nginx.conf: |
    server {
        listen 80;
        server_name example.com;
        
        location / {
            root /usr/share/nginx/html;
            index index.html index.htm;
            try_files $uri $uri/ =404;
        }
        
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
        
        location /metrics {
            stub_status on;
            access_log off;
        }
        
        # Gzip compression
        gzip on;
        gzip_vary on;
        gzip_min_length 1024;
        gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
        
        # Security headers
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-XSS-Protection "1; mode=block" always;
    }
  
  # HTML content
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>Kubernetes Lab</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 40px; }
            .container { max-width: 800px; margin: 0 auto; }
            h1 { color: #326CE5; }
            .info { background: #f0f0f0; padding: 20px; border-radius: 5px; }
            code { background: #e0e0e0; padding: 2px 5px; border-radius: 3px; }
        </style>
    </head>
    <body>
        <div class="container">
            <h1>🎉 Welcome to Kubernetes Lab 7!</h1>
            <div class="info">
                <h2>Configuration Status</h2>
                <p>This page is served from a ConfigMap-mounted volume!</p>
                <ul>
                    <li>Pod Name: <code>POD_NAME_PLACEHOLDER</code></li>
                    <li>Node Name: <code>NODE_NAME_PLACEHOLDER</code></li>
                    <li>Namespace: <code>NAMESPACE_PLACEHOLDER</code></li>
                </ul>
            </div>
        </div>
    </body>
    </html>
```

**Save and exit nano**

```bash
# Create a Secret for sensitive data
echo "📝 Creating Secret manifest..."
nano app-secret.yaml
```

**Add the following to app-secret.yaml:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  namespace: default
type: Opaque
stringData:  # Use stringData for plain text (gets encoded automatically)
  database.username: "admin"
  database.password: "SuperSecretPass123!"
  api.key: "sk-1234567890abcdef"
  jwt.secret: "your-256-bit-secret-key-here-keep-it-secret"
  encryption.key: "AES256-encryption-key"
data:  # Already base64 encoded values
  tls.cert: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCi4uLgotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0t
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCi4uLgotLS0tLUVORCBQUklWQVRFIEtFWS0tLS0t
```

**Save and exit nano**

```bash
echo "================================================"
echo "🚀 Creating Configuration Resources"
echo "================================================"
echo

# Apply configurations
kubectl apply -f app-config.yaml
kubectl apply -f app-secret.yaml

echo
echo "📊 Listing ConfigMaps:"
kubectl get configmaps

echo
echo "🔐 Listing Secrets:"
kubectl get secrets

echo
echo "💡 Learning Tip: Secrets are base64 encoded, NOT encrypted!"
echo "   For true encryption, use tools like Sealed Secrets or Vault."
echo
```

### Subtask 3.2: Create Advanced Pod with Multiple Mount Types

```bash
cd ~/k8s-lab7/manifests

echo "📝 Creating Pod with comprehensive configuration..."
nano advanced-config-pod.yaml
```

**Add the following to advanced-config-pod.yaml:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-demo-pod
  labels:
    app: config-demo
    purpose: configuration-testing
spec:
  containers:
  - name: demo-container
    image: busybox:1.35
    command: ['sh', '-c']
    args:
    - |
      echo "================================================"
      echo "🚀 Container Starting with Configuration Demo"
      echo "================================================"
      echo
      echo "📊 Environment Variables from ConfigMap:"
      echo "Database Host: $DATABASE_HOST"
      echo "Database Port: $DATABASE_PORT"
      echo "App Version: $APP_VERSION"
      echo "Log Level: $LOG_LEVEL"
      echo
      echo "🔐 Environment Variables from Secret:"
      echo "Database User: $DB_USERNAME"
      echo "API Key (first 10 chars): ${API_KEY:0:10}..."
      echo
      echo "📁 Files from ConfigMap Volume:"
      ls -la /config/
      echo
      echo "📄 Content of nginx.conf:"
      head -n 10 /config/nginx.conf
      echo
      echo "📁 Files from Secret Volume:"
      ls -la /secrets/
      echo
      echo "🔄 Watching for configuration changes..."
      echo "Initial config hash:"
      md5sum /config/* | head -3
      echo
      while true; do
        echo "[$(date '+%Y-%m-%d %H:%M:%S')] Container is running... Config hash:"
        md5sum /config/* 2>/dev/null | head -1
        sleep 30
      done
    env:
    # Individual environment variables from ConfigMap
    - name: DATABASE_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database.host
    - name: DATABASE_PORT
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database.port
    - name: APP_VERSION
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: app.version
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: log.level
    # Environment variables from Secret
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: database.username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: database.password
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: api.key
    # All ConfigMap entries as environment variables with prefix
    envFrom:
    - configMapRef:
        name: app-config
      prefix: CONFIG_
    # All Secret entries as environment variables with prefix
    - secretRef:
        name: app-secret
      prefix: SECRET_
    volumeMounts:
    # Mount entire ConfigMap as directory
    - name: config-volume
      mountPath: /config
      readOnly: true
    # Mount specific ConfigMap keys as files
    - name: nginx-config-volume
      mountPath: /etc/nginx
      readOnly: true
    # Mount Secret as directory
    - name: secret-volume
      mountPath: /secrets
      readOnly: true
    # Mount specific Secret key as file
    - name: api-key-volume
      mountPath: /var/run/secrets/api
      readOnly: true
    resources:
      requests:
        memory: "32Mi"
        cpu: "100m"
      limits:
        memory: "64Mi"
        cpu: "200m"
  volumes:
  # Entire ConfigMap as volume
  - name: config-volume
    configMap:
      name: app-config
      defaultMode: 0644
  # Specific items from ConfigMap
  - name: nginx-config-volume
    configMap:
      name: nginx-config
      items:
      - key: nginx.conf
        path: nginx.conf
        mode: 0644
      - key: index.html
        path: html/index.html
        mode: 0644
  # Entire Secret as volume
  - name: secret-volume
    secret:
      secretName: app-secret
      defaultMode: 0400  # More restrictive for secrets
  # Specific Secret key as file
  - name: api-key-volume
    secret:
      secretName: app-secret
      items:
      - key: api.key
        path: key.txt
        mode: 0400
```

**Save and exit nano**

```bash
echo "================================================"
echo "🚀 Deploying Configuration Demo Pod"
echo "================================================"
echo

kubectl apply -f advanced-config-pod.yaml

echo
echo "⏳ Waiting for Pod to start..."
kubectl wait --for=condition=Ready pod/config-demo-pod --timeout=30s

echo
echo "📜 Checking Pod logs to see configuration:"
kubectl logs config-demo-pod | head -40

echo
echo "💡 Learning Tip: Configuration can be mounted as:"
echo "   1. Individual env vars (specific keys)"
echo "   2. All env vars with prefix (entire ConfigMap/Secret)"
echo "   3. Files in a directory (volume mount)"
echo "   4. Specific files (selected keys only)"
echo
```

### Subtask 3.3: Dynamic Configuration Updates

```bash
echo "================================================"
echo "🔄 Testing Dynamic Configuration Updates"
echo "================================================"
echo

echo "📝 Current ConfigMap data:"
kubectl get configmap app-config -o jsonpath='{.data.log\.level}'
echo

echo
echo "🔧 Updating ConfigMap..."
kubectl patch configmap app-config --patch '{"data":{"log.level":"DEBUG","app.version":"2.2.0","feature.beta_features":"true"}}'

echo
echo "✅ ConfigMap updated!"
echo

echo "📊 New ConfigMap values:"
kubectl get configmap app-config -o jsonpath='{.data}' | jq 'with_entries(select(.key | startswith("log") or startswith("app.version") or startswith("feature.beta")))'

echo
echo "⚠️  Note: Environment variables don't update automatically!"
echo "    Only volume-mounted configs update in running Pods."
echo

echo
echo "🔄 Creating a new Pod to demonstrate env var updates..."
kubectl delete pod config-demo-pod --force --grace-period=0
kubectl apply -f advanced-config-pod.yaml

echo
echo "⏳ Waiting for new Pod..."
sleep 5

echo
echo "📜 Checking new Pod's environment (should have DEBUG):"
kubectl exec config-demo-pod -- env | grep -E "LOG_LEVEL|APP_VERSION"

echo
echo "💡 Learning Tip: To update env vars, you must recreate the Pod."
echo "   Volume mounts update automatically (may need app reload)."
echo
```

## 📝 Task 4: Production Deployment with Full Stack

### Create a Complete Application Stack

```bash
echo "================================================"
echo "🏗️  Task 4: Production-Grade Full Stack Deployment"
echo "================================================"
echo

cd ~/k8s-lab7/manifests

echo "📝 Creating production namespace..."
nano production-namespace.yaml
```

**Add the following to production-namespace.yaml:**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    environment: production
    managed-by: kubectl
  annotations:
    description: "Production environment for applications"
```

**Save and exit nano**

```bash
kubectl apply -f production-namespace.yaml

echo
echo "📝 Creating complete application stack..."
nano production-app.yaml
```

**Add the following to production-app.yaml:**

```yaml
# Complete production application stack
---
# ConfigMap for application
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config
  namespace: production
data:
  APP_NAME: "Production Web App"
  APP_PORT: "8080"
  LOG_LEVEL: "INFO"
  CACHE_ENABLED: "true"
  DATABASE_HOST: "postgres.production.svc.cluster.local"
  DATABASE_PORT: "5432"
---
# Secret for credentials
apiVersion: v1
kind: Secret
metadata:
  name: webapp-secret
  namespace: production
type: Opaque
stringData:
  DATABASE_USER: "webapp_user"
  DATABASE_PASSWORD: "ProductionPass123!"
  JWT_SECRET: "super-secret-jwt-key-for-production"
---
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: production
  labels:
    app: webapp
    tier: frontend
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0  # Zero downtime
  selector:
    matchLabels:
      app: webapp
      tier: frontend
  template:
    metadata:
      labels:
        app: webapp
        tier: frontend
        version: "1.0.0"
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - webapp
            topologyKey: kubernetes.io/hostname
      containers:
      - name: webapp
        image: nginx:alpine
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
        envFrom:
        - configMapRef:
            name: webapp-config
        - secretRef:
            name: webapp-secret
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15"]
---
# Service
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
  namespace: production
  labels:
    app: webapp
spec:
  type: ClusterIP
  selector:
    app: webapp
    tier: frontend
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
---
# HorizontalPodAutoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
---
# NetworkPolicy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: webapp-netpol
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: webapp
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
    - namespaceSelector:
        matchLabels:
          name: production
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector: {}
    ports:
    - protocol: TCP
      port: 5432  # PostgreSQL
    - protocol: TCP
      port: 443   # HTTPS
    - protocol: UDP
      port: 53    # DNS
```

**Save and exit nano**

```bash
echo "================================================"
echo "🚀 Deploying Production Stack"
echo "================================================"
echo

kubectl apply -f production-app.yaml

echo
echo "⏳ Waiting for deployment to complete..."
kubectl rollout status deployment/webapp -n production

echo
echo "📊 Production deployment status:"
kubectl get all -n production

echo
echo "🔍 Checking pod distribution:"
kubectl get pods -n production -o wide

echo
echo "📈 HPA status:"
kubectl get hpa -n production

echo
echo "💡 Learning Tip: This production setup includes:"
echo "   - Namespace isolation"
echo "   - Anti-affinity for high availability"
echo "   - Zero-downtime rolling updates"
echo "   - Autoscaling based on resources"
echo "   - Network policies for security"
echo "   - Session affinity for stateful apps"
echo
```

## 📝 Task 5: Advanced Troubleshooting and Debugging

```bash
echo "================================================"
echo "🔍 Task 5: Advanced Troubleshooting Techniques"
echo "================================================"
echo

# Create a problematic pod for debugging
echo "📝 Creating problematic pods for debugging practice..."
nano debug-pods.yaml
```

**Add the following to debug-pods.yaml:**

```yaml
---
# Pod with image pull error
apiVersion: v1
kind: Pod
metadata:
  name: error-pod-image
  labels:
    debug: "true"
    issue: "imagepull"
spec:
  containers:
  - name: bad-image
    image: this-image-definitely-does-not-exist:latest
    imagePullPolicy: Always
---
# Pod with insufficient resources
apiVersion: v1
kind: Pod
metadata:
  name: error-pod-resources
  labels:
    debug: "true"
    issue: "resources"
spec:
  containers:
  - name: resource-hungry
    image: nginx:alpine
    resources:
      requests:
        memory: "100Gi"  # Requesting way too much memory
        cpu: "50"        # Requesting way too many CPUs
---
# Pod with crashloop
apiVersion: v1
kind: Pod
metadata:
  name: error-pod-crash
  labels:
    debug: "true"
    issue: "crashloop"
spec:
  containers:
  - name: crasher
    image: busybox:1.35
    command: ['sh', '-c', 'echo "Starting..."; sleep 5; exit 1']
    restartPolicy: Always
---
# Pod with probe failures
apiVersion: v1
kind: Pod
metadata:
  name: error-pod-probe
  labels:
    debug: "true"
    issue: "probe"
spec:
  containers:
  - name: probe-fail
    image: nginx:alpine
    readinessProbe:
      httpGet:
        path: /this-path-does-not-exist
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
    livenessProbe:
      httpGet:
        path: /another-bad-path
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 5
```

**Save and exit nano**

```bash
kubectl apply -f debug-pods.yaml

echo
echo "⏳ Waiting for pods to show their issues..."
sleep 15

echo
echo "================================================"
echo "🔍 Debugging Practice"
echo "================================================"
echo

# Debug each type of issue
echo "1️⃣ Debugging Image Pull Errors:"
echo "--------------------------------"
kubectl get pod error-pod-image
kubectl describe pod error-pod-image | grep -A 10 Events:
echo
echo "💡 Solution: Check image name, registry access, and pull secrets"
echo

echo "2️⃣ Debugging Resource Issues:"
echo "------------------------------"
kubectl get pod error-pod-resources
kubectl describe pod error-pod-resources | grep -A 5 "Warning"
echo
echo "💡 Solution: Reduce resource requests or add more nodes"
echo

echo "3️⃣ Debugging CrashLoopBackOff:"
echo "--------------------------------"
kubectl get pod error-pod-crash
kubectl logs error-pod-crash --previous 2>/dev/null || kubectl logs error-pod-crash
echo
echo "💡 Solution: Fix application code or startup command"
echo

echo "4️⃣ Debugging Probe Failures:"
echo "-----------------------------"
kubectl get pod error-pod-probe
kubectl describe pod error-pod-probe | grep -E "Liveness|Readiness" -A 3
echo
echo "💡 Solution: Fix probe endpoints or adjust timing parameters"
echo

# Advanced debugging commands
echo "================================================"
echo "🛠️  Advanced Debugging Commands"
echo "================================================"
echo

echo "📊 Resource usage across all namespaces:"
kubectl top pods --all-namespaces | head -10

echo
echo "🔍 Finding pods with issues:"
kubectl get pods --all-namespaces --field-selector=status.phase!=Running,status.phase!=Succeeded

echo
echo "📜 Recent events across cluster:"
kubectl get events --all-namespaces --sort-by='.lastTimestamp' | tail -10

echo
echo "🏷️  Finding resources by label:"
kubectl get all -l debug=true

echo
echo "📝 Getting pod YAML for editing:"
kubectl get pod error-pod-probe -o yaml > pod-backup.yaml
echo "✅ Pod YAML saved to pod-backup.yaml"
echo

echo "💡 Advanced Debugging Tips:"
echo "   - Use 'kubectl debug' to create debugging containers"
echo "   - Use 'kubectl port-forward' to access pod services locally"
echo "   - Use 'kubectl cp' to copy files to/from pods"
echo "   - Use 'kubectl top' to monitor resource usage"
echo "   - Use 'kubectl explain' to understand resource fields"
echo
```

## 📝 Comprehensive Cleanup

```bash
echo "================================================"
echo "🧹 Comprehensive Cleanup"
echo "================================================"
echo

echo "❓ Do you want to clean up all lab resources? (y/n)"
read -r cleanup_answer

if [ "$cleanup_answer" = "y" ]; then
    echo
    echo "🗑️  Removing all lab resources..."
    
    # Delete pods
    kubectl delete pod nginx-pod --ignore-not-found=true
    kubectl delete pod test-pod --ignore-not-found=true
    kubectl delete pod app-pod --ignore-not-found=true
    kubectl delete pod config-demo-pod --ignore-not-found=true
    kubectl delete -f debug-pods.yaml --ignore-not-found=true
    
    # Delete deployments and related resources
    kubectl delete deployment nginx-deployment --ignore-not-found=true
    kubectl delete hpa nginx-hpa --ignore-not-found=true
    
    # Delete configmaps and secrets
    kubectl delete configmap app-config nginx-config --ignore-not-found=true
    kubectl delete secret app-secret --ignore-not-found=true
    
    # Delete production namespace (deletes all resources within)
    kubectl delete namespace production --ignore-not-found=true
    
    echo
    echo "✅ All lab resources cleaned up!"
    
    # Show remaining resources
    echo
    echo "📊 Remaining resources in default namespace:"
    kubectl get all
    
else
    echo "⏭️  Skipping cleanup. Resources remain deployed."
    echo "   To clean up later, run: kubectl delete -f ~/k8s-lab7/manifests/"
fi

echo
echo "================================================"
echo "🎓 Lab Summary and Next Steps"
echo "================================================"
echo
echo "✅ Completed Learning Objectives:"
echo "   1. Deployed Pods with comprehensive YAML manifests"
echo "   2. Created and scaled Deployments with rolling updates"
echo "   3. Implemented ConfigMaps and Secrets for configuration"
echo "   4. Built a production-grade application stack"
echo "   5. Practiced advanced troubleshooting techniques"
echo
echo "📚 Key Concepts Mastered:"
echo "   - Pod lifecycle and health checking"
echo "   - Deployment strategies and rollbacks"
echo "   - Configuration management best practices"
echo "   - Resource management and autoscaling"
echo "   - Network policies and security"
echo "   - Debugging and troubleshooting"
echo
echo "🚀 Recommended Next Steps:"
echo "   1. Explore StatefulSets for stateful applications"
echo "   2. Learn about Ingress for external access"
echo "   3. Implement persistent storage with PVCs"
echo "   4. Study RBAC for security"
echo "   5. Practice with Helm for package management"
echo
echo "💡 Pro Tips:"
echo "   - Always use resource limits in production"
echo "   - Implement proper health checks for all containers"
echo "   - Use namespaces for environment isolation"
echo "   - Keep configurations external (ConfigMaps/Secrets)"
echo "   - Monitor and log everything"
echo
echo "📖 Further Learning Resources:"
echo "   - Official Kubernetes Documentation: https://kubernetes.io/docs/"
echo "   - KCNA Certification: https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/"
echo "   - Kubernetes The Hard Way: https://github.com/kelseyhightower/kubernetes-the-hard-way"
echo
echo "================================================"
echo "🎉 Congratulations on completing Lab 7!"
echo "================================================"
```

## 📌 Quick Reference Cheat Sheet

### Essential kubectl Commands

```bash
# === POD OPERATIONS ===
kubectl get pods                          # List all pods
kubectl get pods -o wide                  # Detailed pod list
kubectl get pods --all-namespaces         # Pods across namespaces
kubectl describe pod <pod-name>           # Detailed pod info
kubectl logs <pod-name>                   # View pod logs
kubectl logs <pod-name> -c <container>    # Specific container logs
kubectl logs <pod-name> --previous        # Previous instance logs
kubectl exec -it <pod-name> -- /bin/sh    # Shell into pod
kubectl port-forward <pod-name> 8080:80   # Port forwarding
kubectl cp <pod-name>:/path/file ./file   # Copy from pod
kubectl top pod <pod-name>                # Resource usage
kubectl debug <pod-name>                  # Create debug container

# === DEPLOYMENT OPERATIONS ===
kubectl get deployments                   # List deployments
kubectl describe deployment <name>        # Deployment details
kubectl scale deployment <name> --replicas=5  # Scale deployment
kubectl rollout status deployment <name>  # Check rollout status
kubectl rollout history deployment <name> # Rollout history
kubectl rollout undo deployment <name>    # Rollback deployment
kubectl rollout restart deployment <name> # Restart all pods
kubectl set image deployment/<name> <container>=<image>  # Update image

# === CONFIGURATION ===
kubectl get configmap                     # List ConfigMaps
kubectl describe configmap <name>         # ConfigMap details
kubectl create configmap <name> --from-file=<path>  # Create from file
kubectl create secret generic <name> --from-literal=key=value  # Create secret
kubectl get secret <name> -o jsonpath='{.data}'  # View secret data

# === TROUBLESHOOTING ===
kubectl get events --sort-by='.lastTimestamp'  # Recent events
kubectl get pods --field-selector=status.phase!=Running  # Non-running pods
kubectl describe node                     # Node details
kubectl api-resources                     # Available resources
kubectl explain <resource>                # Resource documentation
kubectl get pod <name> -o yaml           # Export YAML

# === USEFUL ALIASES ===
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deployments'
alias kdp='kubectl describe pod'
alias kaf='kubectl apply -f'
alias kdf='kubectl delete -f'
alias klo='kubectl logs'
alias kex='kubectl exec -it'
```

### YAML Structure Quick Reference

```yaml
# Pod Template
apiVersion: v1
kind: Pod
metadata:
  name: <pod-name>
  labels:
    key: value
spec:
  containers:
  - name: <container-name>
    image: <image>:<tag>
    ports:
    - containerPort: <port>
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"

# Deployment Template
apiVersion: apps/v1
kind: Deployment
metadata:
  name: <deployment-name>
spec:
  replicas: 3
  selector:
    matchLabels:
      app: <app-name>
  template:
    <pod-spec>

# ConfigMap Template
apiVersion: v1
kind: ConfigMap
metadata:
  name: <configmap-name>
data:
  key1: value1
  key2: value2

# Secret Template
apiVersion: v1
kind: Secret
metadata:
  name: <secret-name>
type: Opaque
stringData:
  key1: value1
data:
  key2: <base64-encoded-value>
```

### Resource Specifications

```yaml
# CPU: 1 = 1 CPU core, 100m = 0.1 CPU
# Memory: 1Gi = 1 Gibibyte, 1G = 1 Gigabyte, 128Mi = 128 Mebibytes

resources:
  requests:    # Guaranteed resources
    memory: "64Mi"
    cpu: "250m"
  limits:      # Maximum resources
    memory: "128Mi"
    cpu: "500m"
```

### Common Issues and Solutions

| Issue | Symptoms | Solution |
|-------|----------|----------|
| ImagePullBackOff | Pod can't pull image | Check image name, registry access |
| CrashLoopBackOff | Container keeps restarting | Check logs, fix application |
| Pending | Pod not scheduled | Check resources, node capacity |
| OOMKilled | Out of memory | Increase memory limits |
| Probe Failures | Container not ready/healthy | Fix endpoints, adjust timing |

---

**🎯 Lab Completion: You are now equipped with production-ready Kubernetes knowledge!**
