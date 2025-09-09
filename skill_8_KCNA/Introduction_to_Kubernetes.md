# Lab 2: Introduction to Kubernetes - Enhanced Version

## Learning Objectives
By the end of this lab, students will be able to:
• Install and configure Minikube for local Kubernetes development
• Deploy and manage applications using both imperative commands and declarative YAML manifests
• Understand core Kubernetes concepts including pods, deployments, services, and namespaces
• Demonstrate Kubernetes scaling and self-healing capabilities
• Implement advanced features like health checks, rolling updates, and resource management
• Compare Kubernetes orchestration capabilities with Docker Swarm
• Navigate the Kubernetes ecosystem and troubleshoot common issues

## Prerequisites
Before starting this lab, students should have:
• Basic understanding of containerization concepts and Docker
• Familiarity with Linux command-line operations
• Knowledge of YAML file structure and syntax
• Understanding of basic networking concepts (ports, IP addresses)
• Experience with text editors (nano, vim, or similar)

## Lab Environment Requirements
• Ubuntu 20.04 LTS or later operating system
• Minimum 4GB RAM and 2 CPU cores available
• Administrative privileges for system configuration
• Stable internet connectivity for downloading packages

---

## Task 1: Environment Setup and Kubernetes Installation

### Subtask 1.1: System Preparation and Dependencies

First, let's ensure our system is properly prepared and all dependencies are installed.

```bash
# Display current system information
echo "=== System Information ==="
lsb_release -a
echo ""

# Update package repository and system
echo "=== Updating System Packages ==="
sudo apt update && sudo apt upgrade -y
echo ""

# Install essential dependencies
echo "=== Installing Essential Dependencies ==="
sudo apt install -y curl wget apt-transport-https ca-certificates gnupg lsb-release
echo ""

# Install additional tools for better CLI experience
echo "=== Installing Additional CLI Tools ==="
sudo apt install -y tree htop jq
echo ""

# Install VirtualBox and related packages
echo "=== Installing VirtualBox ==="
sudo apt install -y virtualbox virtualbox-ext-pack
echo ""

echo "💡 Learning Tip: We install VirtualBox as it provides a reliable virtualization driver for Minikube"
echo "   Alternative drivers include Docker, KVM, and others depending on your system"
echo ""
```

### Subtask 1.2: Install kubectl (Kubernetes CLI)

```bash
# Download and install kubectl
echo "=== Installing kubectl ==="
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
echo ""

# Make kubectl executable and move to system PATH
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
echo ""

# Verify kubectl installation
echo "=== Verifying kubectl Installation ==="
kubectl version --client --output=yaml
echo ""

# Enable kubectl autocompletion for better CLI experience
echo "=== Setting up kubectl Autocompletion ==="
echo 'source <(kubectl completion bash)' >> ~/.bashrc
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null
echo ""

echo "💡 Learning Tip: kubectl is the primary tool for interacting with Kubernetes clusters"
echo "   It communicates with the Kubernetes API server to manage cluster resources"
echo ""
```

### Subtask 1.3: Install Minikube

```bash
# Download and install Minikube
echo "=== Installing Minikube ==="
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64
echo ""

# Verify Minikube installation
echo "=== Verifying Minikube Installation ==="
minikube version
echo ""

echo "💡 Learning Tip: Minikube creates a single-node Kubernetes cluster locally"
echo "   It's perfect for development, testing, and learning Kubernetes concepts"
echo ""
```

### Subtask 1.4: Start and Configure Minikube Cluster

```bash
# Start Minikube with specific configuration
echo "=== Starting Minikube Cluster ==="
minikube start --driver=virtualbox --memory=3072 --cpus=2 --disk-size=20g
echo ""

# Verify cluster status
echo "=== Verifying Cluster Status ==="
minikube status
echo ""

# Get cluster information
echo "=== Cluster Information ==="
kubectl cluster-info
echo ""

# View cluster nodes
echo "=== Cluster Nodes ==="
kubectl get nodes -o wide
echo ""

# Check cluster components
echo "=== Cluster Components ==="
kubectl get pods -n kube-system
echo ""

# Enable useful Minikube addons
echo "=== Enabling Minikube Addons ==="
minikube addons enable dashboard
minikube addons enable metrics-server
minikube addons enable ingress
echo ""

# List enabled addons
echo "=== Enabled Addons ==="
minikube addons list | grep enabled
echo ""

echo "💡 Learning Tip: The kube-system namespace contains essential cluster components"
echo "   Addons extend cluster functionality - dashboard provides a web UI, metrics-server enables resource monitoring"
echo ""
```

---

## Task 2: Basic Application Deployment

### Subtask 2.1: Create Working Directory and Explore kubectl

```bash
# Create organized workspace
echo "=== Setting up Workspace ==="
mkdir -p ~/k8s-lab/{manifests,configs,scripts}
cd ~/k8s-lab
echo ""

# Display current context
echo "=== Current Kubernetes Context ==="
kubectl config current-context
kubectl config get-contexts
echo ""

# Explore kubectl commands
echo "=== Exploring kubectl Commands ==="
kubectl api-resources --output=wide | head -20
echo ""

echo "💡 Learning Tip: kubectl uses contexts to manage connections to different clusters"
echo "   API resources show all the types of objects you can create and manage in Kubernetes"
echo ""
```

### Subtask 2.2: Imperative Application Deployment

```bash
# Create a simple deployment using imperative commands
echo "=== Creating Deployment (Imperative) ==="
kubectl create deployment nginx-imperative --image=nginx:1.21 --replicas=2
echo ""

# View deployment details
echo "=== Deployment Status ==="
kubectl get deployments
echo ""

kubectl get pods -l app=nginx-imperative
echo ""

# Get detailed deployment information
echo "=== Detailed Deployment Information ==="
kubectl describe deployment nginx-imperative
echo ""

# View replica sets (managed by deployment)
echo "=== Replica Sets ==="
kubectl get replicasets -l app=nginx-imperative
echo ""

echo "💡 Learning Tip: Deployments manage ReplicaSets, which in turn manage Pods"
echo "   This creates a hierarchy: Deployment -> ReplicaSet -> Pods"
echo ""
```

### Subtask 2.3: Declarative Application Deployment

```bash
# Create declarative deployment manifest
echo "=== Creating Declarative Deployment Manifest ==="
cat > manifests/nginx-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-declarative
  labels:
    app: nginx-declarative
    environment: lab
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: nginx-declarative
  template:
    metadata:
      labels:
        app: nginx-declarative
        version: v1
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
          name: http
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
EOF
echo ""

# Apply the deployment
echo "=== Applying Declarative Deployment ==="
kubectl apply -f manifests/nginx-deployment.yaml
echo ""

# Monitor deployment rollout
echo "=== Monitoring Deployment Rollout ==="
kubectl rollout status deployment/nginx-declarative
echo ""

# View all deployments
echo "=== All Deployments ==="
kubectl get deployments -o wide
echo ""

echo "💡 Learning Tip: Declarative configurations are preferred for production"
echo "   They're version-controllable, repeatable, and support GitOps workflows"
echo ""
```

### Subtask 2.4: Service Creation and Networking

```bash
# Create service manifest
echo "=== Creating Service Manifest ==="
cat > manifests/nginx-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx-declarative
spec:
  type: NodePort
  selector:
    app: nginx-declarative
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
  labels:
    app: nginx-declarative
spec:
  type: ClusterIP
  selector:
    app: nginx-declarative
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
EOF
echo ""

# Apply service configuration
echo "=== Applying Service Configuration ==="
kubectl apply -f manifests/nginx-service.yaml
echo ""

# View services
echo "=== Service Information ==="
kubectl get services -o wide
echo ""

# View endpoints
echo "=== Service Endpoints ==="
kubectl get endpoints
echo ""

# Describe service details
echo "=== Service Details ==="
kubectl describe service nginx-service
echo ""

echo "💡 Learning Tip: Services provide stable network endpoints for dynamic pod sets"
echo "   NodePort exposes services externally, ClusterIP is internal-only"
echo ""
```

### Subtask 2.5: Application Access and Testing

```bash
# Get Minikube IP and service URL
echo "=== Getting Service Access Information ==="
MINIKUBE_IP=$(minikube ip)
echo "Minikube IP: $MINIKUBE_IP"
echo ""

# Get NodePort service URL
SERVICE_URL=$(minikube service nginx-service --url)
echo "Service URL: $SERVICE_URL"
echo ""

# Test application connectivity
echo "=== Testing Application Connectivity ==="
curl -s $SERVICE_URL | head -10
echo ""

# Test from within cluster
echo "=== Testing Internal Connectivity ==="
kubectl run test-pod --image=curlimages/curl --rm -it --restart=Never -- curl -s nginx-clusterip
echo ""

# Port forward for local access
echo "=== Setting up Port Forward (Background) ==="
kubectl port-forward service/nginx-clusterip 8080:80 &
PF_PID=$!
sleep 2
echo ""

# Test port forward
echo "=== Testing Port Forward ==="
curl -s http://localhost:8080 | head -5
echo ""

# Clean up port forward
kill $PF_PID 2>/dev/null
echo ""

echo "💡 Learning Tip: Port forwarding is useful for debugging and local development"
echo "   It creates a secure tunnel from your local machine to the cluster"
echo ""
```

---

## Task 3: Kubernetes Core Features Exploration

### Subtask 3.1: Scaling Operations

```bash
# Check current deployment scale
echo "=== Current Deployment Status ==="
kubectl get deployment nginx-declarative
kubectl get pods -l app=nginx-declarative
echo ""

# Scale up using kubectl
echo "=== Scaling Up to 5 Replicas ==="
kubectl scale deployment nginx-declarative --replicas=5
echo ""

# Watch scaling in real-time
echo "=== Watching Scaling Operation ==="
kubectl get pods -l app=nginx-declarative -w &
WATCH_PID=$!
sleep 10
kill $WATCH_PID 2>/dev/null
echo ""

# Verify scale operation
echo "=== Verifying Scale Operation ==="
kubectl get deployment nginx-declarative
kubectl get pods -l app=nginx-declarative -o wide
echo ""

# Scale down
echo "=== Scaling Down to 2 Replicas ==="
kubectl scale deployment nginx-declarative --replicas=2
sleep 5
echo ""

kubectl get pods -l app=nginx-declarative
echo ""

echo "💡 Learning Tip: Horizontal scaling changes the number of pod replicas"
echo "   Kubernetes automatically distributes pods across available nodes"
echo ""
```

### Subtask 3.2: Self-Healing Demonstration

```bash
# List current pods with details
echo "=== Current Pod Status ==="
kubectl get pods -l app=nginx-declarative -o wide
echo ""

# Simulate pod failure by deleting a pod
echo "=== Simulating Pod Failure ==="
POD_TO_DELETE=$(kubectl get pods -l app=nginx-declarative -o jsonpath='{.items[0].metadata.name}')
echo "Deleting pod: $POD_TO_DELETE"
kubectl delete pod $POD_TO_DELETE
echo ""

# Immediately check status
echo "=== Immediate Status Check ==="
kubectl get pods -l app=nginx-declarative
echo ""

# Watch self-healing process
echo "=== Watching Self-Healing Process ==="
sleep 3
kubectl get pods -l app=nginx-declarative
echo ""

# Check deployment events
echo "=== Deployment Events ==="
kubectl describe deployment nginx-declarative | tail -10
echo ""

echo "💡 Learning Tip: Kubernetes continuously monitors desired vs actual state"
echo "   When pods fail, the ReplicaSet controller creates replacements automatically"
echo ""
```

### Subtask 3.3: Resource Monitoring and Management

```bash
# Check node resource usage
echo "=== Node Resource Usage ==="
kubectl top nodes
echo ""

# Check pod resource usage
echo "=== Pod Resource Usage ==="
kubectl top pods -l app=nginx-declarative
echo ""

# Get detailed node information
echo "=== Detailed Node Information ==="
kubectl describe node minikube | grep -A 10 "Allocated resources"
echo ""

# View resource quotas and limits
echo "=== Resource Configuration ==="
kubectl describe deployment nginx-declarative | grep -A 10 "Limits\|Requests"
echo ""

# Create resource quota example
echo "=== Creating Resource Quota Example ==="
cat > manifests/resource-quota.yaml << 'EOF'
apiVersion: v1
kind: ResourceQuota
metadata:
  name: lab-quota
  namespace: default
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 2Gi
    limits.cpu: "4"
    limits.memory: 4Gi
    pods: "10"
EOF

kubectl apply -f manifests/resource-quota.yaml
echo ""

kubectl describe resourcequota lab-quota
echo ""

echo "💡 Learning Tip: Resource quotas prevent any single namespace from consuming all cluster resources"
echo "   Requests are guaranteed resources, limits are maximum allowed usage"
echo ""
```

### Subtask 3.4: Namespace Management

```bash
# Create a new namespace
echo "=== Creating New Namespace ==="
kubectl create namespace lab-environment
echo ""

# List all namespaces
echo "=== All Namespaces ==="
kubectl get namespaces
echo ""

# Deploy application in new namespace
echo "=== Deploying to New Namespace ==="
kubectl apply -f manifests/nginx-deployment.yaml -n lab-environment
kubectl apply -f manifests/nginx-service.yaml -n lab-environment
echo ""

# Compare resources across namespaces
echo "=== Resources in Default Namespace ==="
kubectl get pods -n default
echo ""

echo "=== Resources in Lab Namespace ==="
kubectl get pods -n lab-environment
echo ""

# Switch default namespace context
echo "=== Switching Default Namespace ==="
kubectl config set-context --current --namespace=lab-environment
kubectl get pods
echo ""

# Switch back to default
kubectl config set-context --current --namespace=default
echo ""

echo "💡 Learning Tip: Namespaces provide logical separation of resources within a cluster"
echo "   They're useful for organizing applications, teams, or environments"
echo ""
```

---

## Task 4: Advanced Kubernetes Operations

### Subtask 4.1: ConfigMaps and Secrets

```bash
# Create ConfigMap from literal values
echo "=== Creating ConfigMaps ==="
kubectl create configmap app-config \
  --from-literal=environment=lab \
  --from-literal=log_level=info \
  --from-literal=max_connections=100
echo ""

# Create ConfigMap from file
echo "Creating nginx configuration file..."
cat > configs/nginx.conf << 'EOF'
server {
    listen 80;
    server_name localhost;
    
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
    }
    
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
EOF

kubectl create configmap nginx-config --from-file=configs/nginx.conf
echo ""

# Create Secret
echo "=== Creating Secrets ==="
kubectl create secret generic app-secrets \
  --from-literal=database_password=super-secret-password \
  --from-literal=api_key=abcd1234567890
echo ""

# View created resources
echo "=== Created ConfigMaps and Secrets ==="
kubectl get configmaps
kubectl get secrets
echo ""

# Describe ConfigMap
echo "=== ConfigMap Details ==="
kubectl describe configmap app-config
echo ""

# View Secret (values are base64 encoded)
echo "=== Secret Details ==="
kubectl get secret app-secrets -o yaml
echo ""

echo "💡 Learning Tip: ConfigMaps store non-sensitive configuration data"
echo "   Secrets store sensitive data like passwords and are base64 encoded"
echo ""
```

### Subtask 4.2: Enhanced Deployment with ConfigMaps and Secrets

```bash
# Create enhanced deployment with ConfigMaps and Secrets
echo "=== Creating Enhanced Deployment ==="
cat > manifests/nginx-enhanced.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-enhanced
  labels:
    app: nginx-enhanced
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-enhanced
  template:
    metadata:
      labels:
        app: nginx-enhanced
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        env:
        - name: ENVIRONMENT
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: environment
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: log_level
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: database_password
        volumeMounts:
        - name: nginx-config-volume
          mountPath: /etc/nginx/conf.d
        - name: secret-volume
          mountPath: /etc/secrets
          readOnly: true
      volumes:
      - name: nginx-config-volume
        configMap:
          name: nginx-config
      - name: secret-volume
        secret:
          secretName: app-secrets
EOF

kubectl apply -f manifests/nginx-enhanced.yaml
echo ""

# Verify deployment
echo "=== Verifying Enhanced Deployment ==="
kubectl get pods -l app=nginx-enhanced
echo ""

# Check environment variables in pod
echo "=== Checking Environment Variables ==="
POD_NAME=$(kubectl get pods -l app=nginx-enhanced -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD_NAME -- env | grep -E "ENVIRONMENT|LOG_LEVEL|DATABASE"
echo ""

# Check mounted volumes
echo "=== Checking Mounted Volumes ==="
kubectl exec $POD_NAME -- ls -la /etc/nginx/conf.d/
kubectl exec $POD_NAME -- ls -la /etc/secrets/
echo ""

echo "💡 Learning Tip: ConfigMaps and Secrets can be mounted as volumes or environment variables"
echo "   Volume mounts are useful for configuration files, env vars for simple key-value pairs"
echo ""
```

### Subtask 4.3: Rolling Updates and Rollbacks

```bash
# Check current deployment image
echo "=== Current Deployment Image ==="
kubectl describe deployment nginx-enhanced | grep Image
echo ""

# Perform rolling update
echo "=== Performing Rolling Update ==="
kubectl set image deployment/nginx-enhanced nginx=nginx:1.22
echo ""

# Watch rolling update progress
echo "=== Watching Rolling Update ==="
kubectl rollout status deployment/nginx-enhanced
echo ""

# Check rollout history
echo "=== Rollout History ==="
kubectl rollout history deployment/nginx-enhanced
echo ""

# Verify new image version
echo "=== Verifying New Image Version ==="
kubectl describe deployment nginx-enhanced | grep Image
echo ""

# Demonstrate rollback
echo "=== Demonstrating Rollback ==="
kubectl rollout undo deployment/nginx-enhanced
echo ""

# Watch rollback progress
echo "=== Watching Rollback Progress ==="
kubectl rollout status deployment/nginx-enhanced
echo ""

# Verify rollback
echo "=== Verifying Rollback ==="
kubectl describe deployment nginx-enhanced | grep Image
kubectl rollout history deployment/nginx-enhanced
echo ""

echo "💡 Learning Tip: Rolling updates ensure zero-downtime deployments"
echo "   Kubernetes maintains old pods until new ones are ready and healthy"
echo ""
```

### Subtask 4.4: Persistent Storage

```bash
# Create PersistentVolume
echo "=== Creating Persistent Volume ==="
cat > manifests/persistent-volume.yaml << 'EOF'
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nginx-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /tmp/nginx-data
  storageClassName: manual
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual
EOF

kubectl apply -f manifests/persistent-volume.yaml
echo ""

# Check PV and PVC status
echo "=== Persistent Volume Status ==="
kubectl get pv
kubectl get pvc
echo ""

# Create deployment with persistent storage
echo "=== Creating Deployment with Persistent Storage ==="
cat > manifests/nginx-persistent.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-persistent
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-persistent
  template:
    metadata:
      labels:
        app: nginx-persistent
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-storage
          mountPath: /usr/share/nginx/html
      volumes:
      - name: nginx-storage
        persistentVolumeClaim:
          claimName: nginx-pvc
EOF

kubectl apply -f manifests/nginx-persistent.yaml
echo ""

# Write data to persistent volume
echo "=== Writing Data to Persistent Volume ==="
kubectl exec -it deployment/nginx-persistent -- bash -c "echo '<h1>Persistent Data Test</h1>' > /usr/share/nginx/html/index.html"
echo ""

# Test persistence by deleting and recreating pod
echo "=== Testing Persistence ==="
kubectl delete pod -l app=nginx-persistent
sleep 5
kubectl exec -it deployment/nginx-persistent -- cat /usr/share/nginx/html/index.html
echo ""

echo "💡 Learning Tip: PersistentVolumes survive pod restarts and rescheduling"
echo "   They're essential for stateful applications like databases"
echo ""
```

---

## Task 5: Kubernetes vs Docker Swarm Comparison

### Subtask 5.1: Architecture Comparison

```bash
# Create comprehensive comparison document
echo "=== Creating Architecture Comparison ==="
cat > orchestration-comparison.md << 'EOF'
# Kubernetes vs Docker Swarm: Comprehensive Comparison

## Architecture Overview

### Kubernetes Architecture
- **Master Node Components:**
  - API Server: Central management entity
  - etcd: Distributed key-value store
  - Controller Manager: Maintains desired state
  - Scheduler: Assigns pods to nodes
  
- **Worker Node Components:**
  - kubelet: Node agent
  - kube-proxy: Network proxy
  - Container Runtime: Docker/containerd/CRI-O

### Docker Swarm Architecture
- **Manager Nodes:**
  - Raft consensus algorithm
  - Built-in load balancer
  - Service discovery
  
- **Worker Nodes:**
  - Docker Engine
  - Swarm agent

## Deployment Complexity

### Kubernetes
```bash
# Multi-step deployment process
kubectl create deployment app --image=nginx
kubectl expose deployment app --port=80 --type=NodePort
kubectl scale deployment app --replicas=3
```

### Docker Swarm  
```bash
# Single command deployment
docker service create --name app --replicas 3 --publish 80:80 nginx
```

## Feature Comparison Matrix

| Feature | Kubernetes | Docker Swarm | Notes |
|---------|------------|--------------|-------|
| Setup Complexity | High | Low | K8s requires more initial configuration |
| Learning Curve | Steep | Gentle | Docker Swarm uses familiar Docker commands |
| Scalability | Excellent | Good | K8s handles larger clusters better |
| Auto-scaling | Built-in HPA/VPA | Manual | K8s has sophisticated autoscaling |
| Service Discovery | DNS-based | Built-in | Both provide service discovery |
| Load Balancing | Ingress + Services | Built-in | K8s more flexible, Swarm simpler |
| Rolling Updates | Advanced | Basic | K8s offers more update strategies |
| Health Checks | Comprehensive | Basic | K8s has liveness/readiness probes |
| Configuration Management | ConfigMaps/Secrets | Docker Configs/Secrets | Similar capabilities |
| Storage | Persistent Volumes | Docker Volumes | K8s more sophisticated |
| Networking | CNI plugins | Overlay networks | K8s more flexible |
| Monitoring | Prometheus ecosystem | Limited | K8s has rich monitoring ecosystem |
| Security | RBAC, Network Policies | Basic | K8s more comprehensive security |

## Use Case Recommendations

### Choose Kubernetes For:
- Large-scale production deployments
- Complex microservices architectures  
- Multi-cloud or hybrid cloud scenarios
- Teams requiring advanced features
- Organizations with DevOps maturity

### Choose Docker Swarm For:
- Simple containerized applications
- Small to medium deployments
- Teams already using Docker extensively
- Quick prototypes and development
- Resource-constrained environments

## Real-World Examples

### Kubernetes Command Patterns
```bash
# Declarative approach
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml

# Imperative management
kubectl create deployment webapp --image=myapp:v1
kubectl expose deployment webapp --port=80
kubectl autoscale deployment webapp --cpu-percent=50 --min=1 --max=10
```

### Docker Swarm Equivalent
```bash
# Service-centric approach  
docker service create --name webapp --replicas 3 myapp:v1
docker service update --publish-add 80:80 webapp
# Note: No built-in autoscaling equivalent
```
EOF

cat orchestration-comparison.md
echo ""

echo "💡 Learning Tip: The choice between K8s and Swarm depends on complexity needs"
echo "   Kubernetes offers more features but requires more operational overhead"
echo ""
```

### Subtask 5.2: Practical Demonstration

```bash
# Show current Kubernetes deployment complexity
echo "=== Current Kubernetes Resources ==="
kubectl get all
echo ""

# Demonstrate Kubernetes advanced features
echo "=== Kubernetes Advanced Features Demo ==="
kubectl get hpa 2>/dev/null || echo "No HPA configured (would require metrics-server)"
kubectl get networkpolicies 2>/dev/null || echo "No Network Policies configured"
kubectl get ingress 2>/dev/null || echo "No Ingress configured"
echo ""

# Create simple autoscaling example
echo "=== Creating Horizontal Pod Autoscaler ==="
kubectl autoscale deployment nginx-enhanced --cpu-percent=50 --min=1 --max=5
kubectl get hpa
echo ""

# Simulate Docker Swarm equivalent commands
echo "=== Docker Swarm Equivalent Commands ==="
cat > scripts/swarm-equivalent.sh << 'EOF'
#!/bin/bash
echo "Docker Swarm equivalent for current Kubernetes setup:"
echo ""
echo "# Initialize swarm (one-time setup)"  
echo "docker swarm init"
echo ""
echo "# Create service equivalent to our nginx deployment"
echo "docker service create --name nginx-service --replicas 3 --publish 80:80 nginx:1.21"
echo ""
echo "# Scale service"
echo "docker service scale nginx-service=5"
echo ""
echo "# Update service"  
echo "docker service update --image nginx:1.22 nginx-service"
echo ""
echo "# View service status"
echo "docker service ls"
echo "docker service ps nginx-service"
EOF

chmod +x scripts/swarm-equivalent.sh
./scripts/swarm-equivalent.sh
echo ""

echo "💡 Learning Tip: Notice how Kubernetes requires multiple resources (Deployment, Service, HPA)"
echo "   while Docker Swarm combines these concepts into a single 'service'"
echo ""
```

---

## Task 6: Troubleshooting and Best Practices

### Common Issue Diagnosis

```bash
# Create troubleshooting toolkit
echo "=== Creating Troubleshooting Toolkit ==="
cat > scripts/k8s-troubleshoot.sh << 'EOF'
#!/bin/bash

echo "=== Kubernetes Cluster Troubleshooting ==="
echo ""

echo "1. Cluster Status:"
kubectl cluster-info
echo ""

echo "2. Node Status:"  
kubectl get nodes -o wide
echo ""

echo "3. System Pods Status:"
kubectl get pods -n kube-system
echo ""

echo "4. Recent Events:"
kubectl get events --sort-by='.metadata.creationTimestamp' | tail -10
echo ""

echo "5. Resource Usage:"
kubectl top nodes 2>/dev/null || echo "Metrics server not available"
echo ""

echo "6. Failed Pods:"
kubectl get pods --all-namespaces --field-selector=status.phase=Failed
echo ""

echo "7. Pending Pods:"
kubectl get pods --all-namespaces --field-selector=status.phase=Pending
echo ""

echo "8. Disk Usage on Minikube:"
minikube ssh 'df -h'
echo ""

echo "9. Docker Status in Minikube:"
minikube ssh 'sudo systemctl status docker'
echo ""
EOF

chmod +x scripts/k8s-troubleshoot.sh
./scripts/k8s-troubleshoot.sh
echo ""

echo "💡 Learning Tip: Systematic troubleshooting starts with cluster health"
echo "   Events often provide the most valuable debugging information"
echo ""
```
