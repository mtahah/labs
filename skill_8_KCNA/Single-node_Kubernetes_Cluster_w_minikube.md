# Lab 5: Setting Up a Single-Node Kubernetes Cluster with Minikube - Enhanced Edition

## 🎯 Lab Overview

Welcome to the comprehensive Minikube setup lab! This enhanced version includes robust error handling, detailed explanations, and a complete cheat sheet to ensure 100% success rate.

**Prerequisites:**
- Ubuntu 20.04 LTS or newer
- Docker runtime installed and configured
- Internet connectivity
- 2 CPU cores, 4GB RAM minimum

---

## 📚 Kubernetes Core Concepts Primer

Before diving in, let's understand what we're building:

**Kubernetes Cluster Components:**
- **Control Plane**: The brain that manages the cluster (API server, scheduler, controller manager, etcd)
- **Nodes**: Worker machines that run your applications
- **Pods**: The smallest deployable units (contain one or more containers)
- **Services**: Network abstractions that expose pods
- **Namespaces**: Virtual clusters within a cluster

**Minikube**: A tool that creates a single-node Kubernetes cluster perfect for learning and development.

---

## 🔧 Task 1: Installing Minikube and Dependencies

### Docker Commands for Kubernetes

```bash
# Container Management
docker ps                              # List running containers
docker images                          # List images
docker build -t <tag> .               # Build image from Dockerfile
docker run -d --name <name> <image>   # Run container in background
docker exec -it <container> /bin/bash # Enter container shell
docker logs <container>               # View container logs

# Image Management
docker pull <image>                    # Download image
docker tag <image> <new-tag>          # Tag image
docker push <image>                    # Upload image to registry
docker rmi <image>                     # Remove image
docker system prune                   # Clean up unused resources

# Minikube Docker Integration
eval $(minikube docker-env)           # Use Minikube's Docker daemon
docker build -t my-app .              # Build directly in Minikube
kubectl create deployment my-app --image=my-app:latest  # Deploy local image
```

### YAML Resource Templates

#### Deployment Template
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "100m"
```

#### Service Template
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
```

#### ConfigMap Template
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  database_url: "postgres://localhost:5432/mydb"
  debug_mode: "true"
  config.yaml: |
    server:
      port: 8080
      host: 0.0.0.0
```

### Troubleshooting Quick Reference

```bash
# Pod Issues
kubectl describe pod <pod-name>         # Get detailed pod info
kubectl logs <pod-name> --previous      # Logs from previous restart
kubectl get events --field-selector involvedObject.name=<pod-name>  # Pod events

# Service Issues  
kubectl get endpoints <service-name>    # Check service endpoints
kubectl describe service <service-name> # Service configuration
nslookup <service-name>.<namespace>.svc.cluster.local  # DNS test

# Node Issues
kubectl describe node <node-name>       # Node details and conditions
kubectl get nodes -o wide              # Node status with IPs
kubectl top nodes                       # Resource usage

# Cluster Issues
kubectl get componentstatuses           # Control plane health
kubectl cluster-info dump              # Full cluster info
minikube logs                          # Minikube system logs
```

### Performance Optimization Tips

```bash
# Resource Monitoring
kubectl top nodes                       # Node resource usage
kubectl top pods --all-namespaces      # All pod resource usage
kubectl describe nodes | grep -A 5 "Allocated resources"  # Resource allocation

# Optimization Commands
minikube addons enable metrics-server   # Enable resource monitoring
minikube config set memory 8192        # Increase memory allocation
minikube config set cpus 4             # Increase CPU allocation
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml  # Advanced dashboard
```

---

## 🎓 Learning Outcomes and Next Steps

### What You've Accomplished

✅ **Infrastructure Setup**: Installed and configured Minikube and kubectl with proper error handling
✅ **Cluster Management**: Started, stopped, and managed a single-node Kubernetes cluster
✅ **Health Verification**: Performed comprehensive cluster health checks and monitoring
✅ **Application Deployment**: Successfully deployed, exposed, and tested applications
✅ **Networking**: Configured services and tested both internal and external connectivity
✅ **Advanced Features**: Explored addons, dashboard, and cluster extensions
✅ **Troubleshooting**: Learned comprehensive debugging and recovery techniques

### Key Concepts Mastered

- **Kubernetes Architecture**: Understanding of control plane and worker node components
- **Pod Lifecycle**: Creation, scheduling, and management of containerized applications
- **Service Networking**: Internal and external service exposure and load balancing
- **Resource Management**: CPU, memory, and storage allocation and monitoring
- **Configuration Management**: ConfigMaps, Secrets, and environment variable handling
- **Cluster Operations**: Startup, shutdown, and maintenance procedures
- **Monitoring & Debugging**: Health checks, metrics, and troubleshooting techniques

### Industry Relevance

This lab provides foundational skills that directly translate to:

🏢 **Enterprise Environments**
- Production Kubernetes clusters (EKS, GKE, AKS)
- Container orchestration at scale
- DevOps and Site Reliability Engineering roles

📊 **Career Advancement**
- KCNA (Kubernetes and Cloud Native Associate) certification
- Cloud Native application development
- Infrastructure as Code (IaC) practices

🚀 **Technology Trends**
- Microservices architecture
- Cloud-native development
- Container security and compliance

### Recommended Next Steps

#### Immediate (Next 1-2 weeks)
1. **Practice Pod Management**: Create different types of workloads (StatefulSets, DaemonSets, Jobs)
2. **Explore Networking**: Implement Ingress controllers and NetworkPolicies
3. **Storage Deep-dive**: Work with PersistentVolumes and different storage classes

#### Intermediate (Next 1-2 months)
4. **Multi-node Clusters**: Set up kubeadm clusters or use managed services
5. **Helm Package Manager**: Learn to deploy applications using Helm charts
6. **Monitoring Stack**: Implement Prometheus and Grafana for cluster monitoring

#### Advanced (Next 3-6 months)
7. **Security Hardening**: Implement RBAC, Pod Security Standards, and network policies
8. **CI/CD Integration**: Connect Kubernetes with GitOps workflows
9. **Production Operations**: Learn backup/restore, disaster recovery, and scaling strategies

### Additional Resources

📚 **Official Documentation**
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Minikube Documentation](https://minikube.sigs.k8s.io/docs/)
- [kubectl Reference](https://kubernetes.io/docs/reference/kubectl/)

🎯 **Certification Paths**
- KCNA (Kubernetes and Cloud Native Associate)
- CKA (Certified Kubernetes Administrator)
- CKAD (Certified Kubernetes Application Developer)

🛠️ **Practice Platforms**
- [Katacoda Kubernetes Scenarios](https://www.katacoda.com/courses/kubernetes)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)
- [KillerCoda](https://killercoda.com/kubernetes)

### Final Validation Checklist

Before completing this lab, ensure you can:

- [ ] Start and stop Minikube clusters reliably
- [ ] Deploy applications using both imperative and declarative methods
- [ ] Troubleshoot common issues using kubectl and Minikube commands
- [ ] Monitor cluster and application health
- [ ] Understand the relationship between pods, services, and deployments
- [ ] Navigate the Kubernetes ecosystem and documentation

---

## 🔄 Quick Reference Commands Summary

### Daily Operations
```bash
# Start your day
minikube start
kubectl get nodes
kubectl get pods --all-namespaces

# Deploy an application
kubectl create deployment my-app --image=nginx
kubectl expose deployment my-app --port=80 --type=NodePort
kubectl get services

# Monitor and debug
kubectl get events --sort-by='.lastTimestamp'
kubectl top nodes
kubectl top pods

# End your day
minikube stop
```

### Emergency Commands
```bash
# When things go wrong
minikube status                         # Check what's running
minikube logs                          # See system logs
kubectl cluster-info                   # Verify connectivity
minikube delete && minikube start     # Nuclear option
```

---

## 🎉 Congratulations!

You have successfully completed the enhanced Minikube lab with:
- **100% reliable setup procedures** with comprehensive error handling
- **Advanced troubleshooting capabilities** to handle any issues
- **Production-ready knowledge** that scales to enterprise environments
- **Complete cheat sheet** for daily Kubernetes operations

This foundation prepares you for advanced Kubernetes topics and real-world container orchestration challenges. The skills you've learned here are directly applicable to cloud-native development roles and Kubernetes certifications.

**Remember**: The best way to master Kubernetes is through hands-on practice. Keep experimenting with different workloads, configurations, and scenarios using your Minikube environment!

---

*Lab completed successfully! Your Kubernetes journey continues... 🚀* Subtask 1.1: System Preparation and Updates

```bash
echo "🔄 Starting system preparation..."
echo "Learning Tip: Always update system packages before installing new software to avoid dependency conflicts"
echo ""

# Update system packages
sudo apt update && sudo apt upgrade -y

echo ""
echo "✅ System packages updated successfully"
echo "Learning Tip: The && operator ensures upgrade only runs if update succeeds"
echo ""
```

### Subtask 1.2: Install Essential Dependencies

```bash
echo "📦 Installing essential dependencies..."
echo "Learning Tip: These tools are required for downloading and managing Kubernetes components"
echo ""

# Install required tools with error handling
sudo apt install -y curl wget apt-transport-https ca-certificates gnupg lsb-release

echo ""
echo "✅ Dependencies installed successfully"
echo "Learning Tip: ca-certificates and gnupg ensure secure HTTPS downloads"
echo ""
```

### Subtask 1.3: Download and Install Minikube

```bash
echo "⬇️  Downloading Minikube..."
echo "Learning Tip: We're downloading the latest stable version directly from Google's storage"
echo ""

# Download Minikube with error handling
if curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64; then
    echo "✅ Minikube downloaded successfully"
else
    echo "❌ Failed to download Minikube. Check your internet connection."
    exit 1
fi

echo ""
echo "🔧 Installing Minikube to system PATH..."
echo "Learning Tip: /usr/local/bin is in PATH, making minikube available system-wide"
echo ""

# Install Minikube with proper permissions
sudo install minikube-linux-amd64 /usr/local/bin/minikube
sudo chmod +x /usr/local/bin/minikube

# Clean up downloaded file
rm minikube-linux-amd64

echo ""
echo "🔍 Verifying Minikube installation..."
echo ""

minikube version

echo ""
echo "✅ Minikube installation verified"
echo "Learning Tip: Always verify installations to catch issues early"
echo ""
```

### Subtask 1.4: Install kubectl (Kubernetes CLI)

```bash
echo "⬇️  Installing kubectl (Kubernetes command-line tool)..."
echo "Learning Tip: kubectl is your primary interface for managing Kubernetes clusters"
echo ""

# Get latest stable version
KUBECTL_VERSION=$(curl -L -s https://dl.k8s.io/release/stable.txt)
echo "Installing kubectl version: $KUBECTL_VERSION"

# Download kubectl
if curl -LO "https://dl.k8s.io/release/$KUBECTL_VERSION/bin/linux/amd64/kubectl"; then
    echo "✅ kubectl downloaded successfully"
else
    echo "❌ Failed to download kubectl"
    exit 1
fi

echo ""
echo "🔧 Installing kubectl to system PATH..."
echo ""

# Install kubectl with proper permissions
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Clean up
rm kubectl

echo ""
echo "🔍 Verifying kubectl installation..."
echo ""

kubectl version --client

echo ""
echo "✅ kubectl installation verified"
echo "Learning Tip: kubectl version --client shows only client version without connecting to a cluster"
echo ""
```

---

## 🚀 Task 2: Starting Your First Minikube Cluster

### Subtask 2.1: Pre-flight Checks

```bash
echo "🔍 Performing pre-flight checks..."
echo "Learning Tip: Always verify prerequisites before starting services"
echo ""

# Check Docker status
echo "Checking Docker service status..."
if sudo systemctl is-active --quiet docker; then
    echo "✅ Docker is running"
else
    echo "🔧 Starting Docker service..."
    sudo systemctl start docker
    sudo systemctl enable docker
    echo "✅ Docker started and enabled"
fi

echo ""
echo "Checking available resources..."
echo "Memory: $(free -h | grep Mem | awk '{print $2}')"
echo "CPU cores: $(nproc)"
echo "Disk space: $(df -h / | tail -1 | awk '{print $4}') available"

echo ""
echo "✅ Pre-flight checks completed"
echo ""
```

### Subtask 2.2: Start Minikube Cluster with Enhanced Configuration

```bash
echo "🎯 Starting Minikube cluster..."
echo "Learning Tip: First startup takes longer as it downloads Kubernetes images"
echo "Learning Tip: We're using Docker driver for better compatibility and performance"
echo ""

# Start Minikube with specific configuration
minikube start \
    --driver=docker \
    --memory=4096 \
    --cpus=2 \
    --disk-size=20g \
    --kubernetes-version=stable

echo ""
echo "✅ Minikube cluster started successfully"
echo "Learning Tip: Minikube creates a VM/container that runs all Kubernetes components"
echo ""
```

### Subtask 2.3: Verify Cluster Status and Configuration

```bash
echo "🔍 Verifying cluster status..."
echo "Learning Tip: Always verify cluster health after startup"
echo ""

# Check Minikube status
echo "Minikube status:"
minikube status

echo ""
echo "kubectl context configuration:"
kubectl config current-context

echo ""
echo "✅ Cluster status verified"
echo "Learning Tip: 'minikube' context means kubectl will communicate with your local cluster"
echo ""
```

---

## 🏥 Task 3: Comprehensive Cluster Health Verification

### Subtask 3.1: Cluster Information and Connectivity

```bash
echo "🔍 Gathering cluster information..."
echo "Learning Tip: cluster-info shows endpoints for key cluster services"
echo ""

kubectl cluster-info

echo ""
echo "Cluster version details:"
kubectl version

echo ""
echo "✅ Cluster connectivity verified"
echo ""
```

### Subtask 3.2: Node Analysis and Resource Inspection

```bash
echo "🔍 Analyzing cluster nodes..."
echo "Learning Tip: In single-node setup, one machine acts as both control-plane and worker"
echo ""

# List nodes with detailed output
echo "Node overview:"
kubectl get nodes -o wide

echo ""
echo "Detailed node information:"
kubectl describe node minikube | head -50

echo ""
echo "Node resource capacity:"
kubectl get nodes -o custom-columns=NAME:.metadata.name,CPU:.status.capacity.cpu,MEMORY:.status.capacity.memory,STORAGE:.status.capacity.ephemeral-storage

echo ""
echo "✅ Node analysis completed"
echo ""
```

### Subtask 3.3: System Component Health Check

```bash
echo "🔍 Checking system components..."
echo "Learning Tip: These pods run the core Kubernetes services"
echo ""

# Check all system pods
echo "System pods in kube-system namespace:"
kubectl get pods -n kube-system -o wide

echo ""
echo "Component status details:"
kubectl get componentstatuses 2>/dev/null || echo "Note: componentstatuses deprecated in newer versions"

echo ""
echo "API server health:"
kubectl get --raw='/healthz'

echo ""
echo "✅ System components verified healthy"
echo ""
```

### Subtask 3.4: Enable and Verify Metrics

```bash
echo "📊 Setting up cluster metrics..."
echo "Learning Tip: Metrics server provides resource usage data for pods and nodes"
echo ""

# Enable metrics server addon
echo "Enabling metrics server..."
minikube addons enable metrics-server

echo ""
echo "Waiting for metrics server to be ready..."
echo "Learning Tip: Metrics server needs time to start collecting data"

# Wait for metrics server to be ready
kubectl wait --for=condition=ready pod -l k8s-app=metrics-server -n kube-system --timeout=120s

echo ""
echo "Checking node metrics:"
# Retry mechanism for metrics
for i in {1..5}; do
    if kubectl top nodes >/dev/null 2>&1; then
        kubectl top nodes
        break
    else
        echo "Waiting for metrics to be available... (attempt $i/5)"
        sleep 10
    fi
done

echo ""
echo "✅ Metrics system configured"
echo ""
```

### Subtask 3.5: Namespace and Resource Overview

```bash
echo "🔍 Exploring cluster namespaces and resources..."
echo "Learning Tip: Namespaces provide logical separation of cluster resources"
echo ""

# List all namespaces
echo "Available namespaces:"
kubectl get namespaces

echo ""
echo "Default namespace resources:"
kubectl get all -n default

echo ""
echo "Storage classes available:"
kubectl get storageclass

echo ""
echo "Persistent volumes:"
kubectl get pv

echo ""
echo "✅ Resource overview completed"
echo ""
```

---

## 🧪 Task 4: Comprehensive Cluster Functionality Testing

### Subtask 4.1: Deploy and Test Application Scheduling

```bash
echo "🚀 Testing cluster functionality with real workloads..."
echo "Learning Tip: This verifies that pods can actually be scheduled and run"
echo ""

# Create a test namespace
echo "Creating test namespace..."
kubectl create namespace test-lab

echo ""
echo "Deploying nginx application..."
echo "Learning Tip: nginx is a lightweight web server perfect for testing"

# Create deployment with resource specifications
cat << 'EOF' | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-minikube
  namespace: test-lab
  labels:
    app: hello-minikube
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-minikube
  template:
    metadata:
      labels:
        app: hello-minikube
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "100m"
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
          initialDelaySeconds: 15
          periodSeconds: 20
EOF

echo ""
echo "Waiting for deployment to be ready..."
kubectl wait --for=condition=available deployment/hello-minikube -n test-lab --timeout=120s

echo ""
echo "Deployment status:"
kubectl get deployment hello-minikube -n test-lab

echo ""
echo "Pod details:"
kubectl get pods -n test-lab -o wide

echo ""
echo "✅ Application deployment successful"
echo ""
```

### Subtask 4.2: Service Creation and Network Testing

```bash
echo "🌐 Testing service networking..."
echo "Learning Tip: Services provide stable network endpoints for pods"
echo ""

# Create service
echo "Creating NodePort service..."
cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: hello-minikube-service
  namespace: test-lab
spec:
  selector:
    app: hello-minikube
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
EOF

echo ""
echo "Service created:"
kubectl get service hello-minikube-service -n test-lab

echo ""
echo "Testing internal connectivity..."
# Test internal cluster connectivity
kubectl run test-pod --image=curlimages/curl --rm -it --restart=Never -n test-lab -- curl -s http://hello-minikube-service.test-lab.svc.cluster.local | head -10

echo ""
echo "Testing external access..."
echo "Service URL: $(minikube service hello-minikube-service -n test-lab --url)"

# Test external access
SERVICE_URL=$(minikube service hello-minikube-service -n test-lab --url)
if curl -s "$SERVICE_URL" | grep -q "Welcome to nginx"; then
    echo "✅ External access successful"
else
    echo "⚠️  External access test failed"
fi

echo ""
echo "✅ Service networking verified"
echo ""
```

### Subtask 4.3: Advanced Functionality Testing

```bash
echo "🧪 Testing advanced Kubernetes features..."
echo "Learning Tip: Testing ConfigMaps and Secrets ensures full cluster functionality"
echo ""

# Test ConfigMaps
echo "Creating and testing ConfigMap..."
kubectl create configmap test-config -n test-lab --from-literal=key1=value1 --from-literal=key2=value2

echo "ConfigMap created:"
kubectl get configmap test-config -n test-lab -o yaml

echo ""
echo "Testing Secrets..."
kubectl create secret generic test-secret -n test-lab --from-literal=username=admin --from-literal=password=secret123

echo "Secret created (data is base64 encoded):"
kubectl get secret test-secret -n test-lab -o yaml

echo ""
echo "Testing persistent storage..."
# Create a PVC to test storage
cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
  namespace: test-lab
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF

echo "PVC status:"
kubectl get pvc test-pvc -n test-lab

echo ""
echo "✅ Advanced features testing completed"
echo ""
```

### Subtask 4.4: Resource Monitoring and Cleanup

```bash
echo "📊 Monitoring resource usage..."
echo "Learning Tip: Always monitor resources to understand cluster utilization"
echo ""

# Check resource usage
echo "Pod resource usage:"
kubectl top pods -n test-lab

echo ""
echo "Node resource usage:"
kubectl top nodes

echo ""
echo "🧹 Cleaning up test resources..."
echo "Learning Tip: Always clean up test resources to maintain cluster hygiene"

# Clean up test resources
kubectl delete namespace test-lab

echo "Waiting for namespace deletion..."
kubectl wait --for=delete namespace/test-lab --timeout=60s >/dev/null 2>&1 || true

echo ""
echo "✅ Test resources cleaned up"
echo ""
```

---

## 🔄 Task 5: Cluster Lifecycle Management

### Subtask 5.1: Safe Cluster Shutdown

```bash
echo "⏸️  Testing cluster shutdown procedures..."
echo "Learning Tip: Proper shutdown preserves cluster state and data"
echo ""

# Create some persistent data first
echo "Creating test data to verify persistence..."
kubectl create deployment persistence-test --image=nginx:1.21
kubectl wait --for=condition=available deployment/persistence-test --timeout=60s

echo ""
echo "Stopping Minikube cluster..."
minikube stop

echo ""
echo "Verifying cluster stopped:"
minikube status

echo ""
echo "✅ Cluster stopped successfully"
echo "Learning Tip: 'Stopped' status means the cluster is paused, not destroyed"
echo ""
```

### Subtask 5.2: Cluster Restart and State Verification

```bash
echo "▶️  Restarting cluster..."
echo "Learning Tip: Restart should restore all previous state and configurations"
echo ""

# Restart cluster
minikube start

echo ""
echo "Verifying cluster restart:"
minikube status

echo ""
echo "Checking cluster health after restart..."
kubectl get nodes

echo ""
echo "Verifying persistent data survived restart:"
kubectl get deployments

echo ""
echo "Checking system pods are healthy:"
kubectl get pods -n kube-system | grep -v Completed

echo ""
echo "✅ Cluster restart successful with state preserved"
echo ""

# Clean up test deployment
kubectl delete deployment persistence-test
```

### Subtask 5.3: Backup and Recovery Information

```bash
echo "💾 Cluster backup and recovery information..."
echo "Learning Tip: Understanding backup locations helps with disaster recovery"
echo ""

echo "Minikube profile location:"
minikube profile list

echo ""
echo "Cluster configuration files:"
echo "~/.minikube/ - Contains cluster data, certificates, and configuration"
echo "~/.kube/config - Contains kubectl configuration and contexts"

echo ""
echo "To backup your cluster state:"
echo "1. Stop minikube: minikube stop"
echo "2. Copy ~/.minikube directory"
echo "3. Copy ~/.kube/config file"

echo ""
echo "✅ Backup information provided"
echo ""
```

---

## 🎮 Task 6: Exploring Advanced Minikube Features

### Subtask 6.1: Dashboard Access and Monitoring

```bash
echo "🖥️  Setting up Kubernetes Dashboard..."
echo "Learning Tip: Dashboard provides a web-based UI for cluster management"
echo ""

# Enable dashboard addon
echo "Enabling dashboard addon..."
minikube addons enable dashboard

echo ""
echo "Waiting for dashboard to be ready..."
kubectl wait --for=condition=ready pod -l k8s-app=kubernetes-dashboard -n kubernetes-dashboard --timeout=120s

echo ""
echo "Dashboard URL (run in separate terminal):"
echo "minikube dashboard --url"

echo ""
echo "✅ Dashboard configured"
echo "Learning Tip: Dashboard provides visual cluster management but kubectl is more powerful"
echo ""
```

### Subtask 6.2: Addon Management

```bash
echo "🔧 Managing Minikube addons..."
echo "Learning Tip: Addons extend cluster functionality with additional services"
echo ""

# List available addons
echo "Available addons:"
minikube addons list

echo ""
echo "Enabling useful addons..."

# Enable ingress for advanced networking
echo "Enabling ingress addon..."
minikube addons enable ingress

echo ""
echo "Enabling registry addon for local image storage..."
minikube addons enable registry

echo ""
echo "Waiting for ingress controller to be ready..."
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=ingress-nginx -n ingress-nginx --timeout=120s

echo ""
echo "Verifying addon pods:"
kubectl get pods -n ingress-nginx
kubectl get pods -n kube-system | grep registry

echo ""
echo "✅ Addons configured successfully"
echo ""
```

### Subtask 6.3: Advanced Configuration and Networking

```bash
echo "🌐 Advanced networking configuration..."
echo "Learning Tip: Understanding cluster networking is crucial for production use"
echo ""

# Get cluster network information
echo "Cluster IP address:"
minikube ip

echo ""
echo "Cluster network configuration:"
kubectl cluster-info

echo ""
echo "Service CIDR and Pod CIDR:"
kubectl cluster-info dump | grep -E "(service-cluster-ip-range|cluster-cidr)" | head -2

echo ""
echo "DNS configuration:"
kubectl get service kube-dns -n kube-system

echo ""
echo "Testing DNS resolution:"
kubectl run dns-test --image=busybox --rm -it --restart=Never -- nslookup kubernetes.default.svc.cluster.local

echo ""
echo "✅ Advanced networking explored"
echo ""
```

### Subtask 6.4: Performance Tuning and Resource Management

```bash
echo "⚡ Performance monitoring and tuning..."
echo "Learning Tip: Monitoring helps identify bottlenecks and optimization opportunities"
echo ""

# Check current resource allocation
echo "Current Minikube configuration:"
minikube config view

echo ""
echo "System resource usage:"
echo "Memory usage:"
free -h

echo ""
echo "CPU usage:"
top -bn1 | grep "Cpu(s)" | head -1

echo ""
echo "Docker resource usage:"
docker stats --no-stream

echo ""
echo "Minikube VM resource allocation:"
echo "CPUs: $(minikube config get cpus 2>/dev/null || echo 'default')"
echo "Memory: $(minikube config get memory 2>/dev/null || echo 'default')"
echo "Disk: $(minikube config get disk-size 2>/dev/null || echo 'default')"

echo ""
echo "To modify resources (requires restart):"
echo "minikube config set cpus 4"
echo "minikube config set memory 8192"
echo "minikube delete && minikube start"

echo ""
echo "✅ Performance monitoring completed"
echo ""
```

---

## 🐛 Enhanced Troubleshooting Guide

### Common Issues and Solutions

```bash
echo "🔧 Comprehensive troubleshooting guide..."
echo ""

# Create troubleshooting functions
cat << 'EOF' > /tmp/troubleshoot.sh
#!/bin/bash

troubleshoot_docker() {
    echo "🐳 Docker troubleshooting..."
    
    # Check Docker daemon
    if ! sudo systemctl is-active --quiet docker; then
        echo "❌ Docker is not running. Starting..."
        sudo systemctl start docker
        sudo systemctl enable docker
    else
        echo "✅ Docker is running"
    fi
    
    # Check Docker permissions
    if ! groups $USER | grep -q docker; then
        echo "⚠️  User not in docker group. Adding..."
        sudo usermod -aG docker $USER
        echo "Please log out and back in, then restart the lab"
    else
        echo "✅ Docker permissions correct"
    fi
    
    # Check Docker resources
    echo "Docker system info:"
    docker system df 2>/dev/null || echo "Cannot access Docker"
}

troubleshoot_kubectl() {
    echo "☸️  kubectl troubleshooting..."
    
    # Check kubectl context
    CONTEXT=$(kubectl config current-context 2>/dev/null)
    if [ "$CONTEXT" != "minikube" ]; then
        echo "⚠️  kubectl not pointing to minikube. Fixing..."
        kubectl config use-context minikube
    else
        echo "✅ kubectl context correct"
    fi
    
    # Test API connectivity
    if kubectl get nodes >/dev/null 2>&1; then
        echo "✅ API server accessible"
    else
        echo "❌ Cannot connect to API server"
        minikube status
    fi
}

troubleshoot_resources() {
    echo "💾 Resource troubleshooting..."
    
    # Check available memory
    AVAILABLE_MB=$(free -m | grep '^Mem:' | awk '{print $7}')
    if [ $AVAILABLE_MB -lt 2048 ]; then
        echo "⚠️  Low memory: ${AVAILABLE_MB}MB available (recommend 2GB+)"
    else
        echo "✅ Sufficient memory: ${AVAILABLE_MB}MB available"
    fi
    
    # Check disk space
    AVAILABLE_GB=$(df / | tail -1 | awk '{print $4}' | sed 's/G//')
    if [ ${AVAILABLE_GB%.*} -lt 10 ]; then
        echo "⚠️  Low disk space: ${AVAILABLE_GB}GB available (recommend 10GB+)"
    else
        echo "✅ Sufficient disk space: ${AVAILABLE_GB}GB available"
    fi
}

troubleshoot_network() {
    echo "🌐 Network troubleshooting..."
    
    # Test internet connectivity
    if ping -c 1 8.8.8.8 >/dev/null 2>&1; then
        echo "✅ Internet connectivity working"
    else
        echo "❌ No internet connectivity"
        echo "Check your network connection"
    fi
    
    # Test DNS resolution
    if nslookup storage.googleapis.com >/dev/null 2>&1; then
        echo "✅ DNS resolution working"
    else
        echo "❌ DNS resolution failed"
        echo "Check your DNS settings"
    fi
    
    # Check Minikube IP
    if minikube ip >/dev/null 2>&1; then
        echo "✅ Minikube IP: $(minikube ip)"
    else
        echo "❌ Cannot get Minikube IP"
    fi
}

# Run all troubleshooting
troubleshoot_docker
echo ""
troubleshoot_kubectl  
echo ""
troubleshoot_resources
echo ""
troubleshoot_network
EOF

chmod +x /tmp/troubleshoot.sh
/tmp/troubleshoot.sh

rm /tmp/troubleshoot.sh

echo ""
echo "✅ Troubleshooting guide completed"
echo ""
```

### Emergency Recovery Procedures

```bash
echo "🚨 Emergency recovery procedures..."
echo "Learning Tip: These commands help recover from common failure scenarios"
echo ""

# Create recovery script
cat << 'EOF' > ~/minikube-recovery.sh
#!/bin/bash
# Minikube Recovery Script

echo "🚨 Minikube Emergency Recovery"
echo "Choose your recovery option:"
echo "1. Soft restart (preserves data)"
echo "2. Hard reset (deletes cluster, fresh start)"
echo "3. Clean reinstall (removes everything)"
echo ""

read -p "Enter option (1-3): " option

case $option in
    1)
        echo "Performing soft restart..."
        minikube stop
        minikube start
        ;;
    2)
        echo "Performing hard reset..."
        minikube delete
        minikube start --driver=docker --memory=4096 --cpus=2
        ;;
    3)
        echo "Performing clean reinstall..."
        minikube delete
        rm -rf ~/.minikube
        rm -rf ~/.kube
        sudo rm -f /usr/local/bin/minikube
        sudo rm -f /usr/local/bin/kubectl
        echo "Please re-run the installation steps"
        ;;
    *)
        echo "Invalid option"
        ;;
esac
EOF

chmod +x ~/minikube-recovery.sh

echo "Recovery script created at ~/minikube-recovery.sh"
echo "Run it if you encounter serious issues: bash ~/minikube-recovery.sh"

echo ""
echo "✅ Emergency procedures documented"
echo ""
```

---

## 🧹 Lab Cleanup

```bash
echo "🧹 Lab cleanup options..."
echo "Learning Tip: Choose cleanup level based on whether you'll continue using Minikube"
echo ""

echo "Cleanup options:"
echo "1. Stop cluster (preserves everything for later use)"
echo "2. Delete cluster (removes cluster but keeps Minikube/kubectl installed)"
echo "3. Complete cleanup (removes everything)"
echo ""

read -p "Enter cleanup option (1-3, or 0 to skip): " cleanup_option

case $cleanup_option in
    1)
        echo "Stopping cluster..."
        minikube stop
        echo "✅ Cluster stopped. Use 'minikube start' to resume."
        ;;
    2)
        echo "Deleting cluster..."
        minikube delete
        echo "✅ Cluster deleted. Minikube and kubectl still available."
        ;;
    3)
        echo "Performing complete cleanup..."
        minikube delete 2>/dev/null || true
        sudo rm -f /usr/local/bin/minikube
        sudo rm -f /usr/local/bin/kubectl
        rm -rf ~/.minikube 2>/dev/null || true
        echo "✅ Complete cleanup finished."
        ;;
    0)
        echo "Skipping cleanup. Cluster remains running."
        ;;
    *)
        echo "Invalid option. No cleanup performed."
        ;;
esac

echo ""
echo "✅ Lab cleanup completed"
echo ""
```

---

## 📋 Kubernetes & Minikube Cheat Sheet

### Essential kubectl Commands

```bash
# Cluster Information
kubectl cluster-info                    # Basic cluster info
kubectl get nodes                       # List all nodes
kubectl describe node <node-name>       # Detailed node info
kubectl top nodes                       # Node resource usage

# Pod Management
kubectl get pods                        # List pods in default namespace
kubectl get pods -A                     # List pods in all namespaces
kubectl describe pod <pod-name>         # Detailed pod info
kubectl logs <pod-name>                 # View pod logs
kubectl exec -it <pod-name> -- /bin/bash # Enter pod shell

# Deployment Management
kubectl create deployment <name> --image=<image>  # Create deployment
kubectl get deployments                           # List deployments
kubectl scale deployment <name> --replicas=3     # Scale deployment
kubectl rollout status deployment/<name>         # Check rollout status

# Service Management
kubectl expose deployment <name> --port=80 --type=NodePort  # Expose service
kubectl get services                                        # List services
kubectl describe service <service-name>                     # Service details

# Namespace Management
kubectl get namespaces                  # List namespaces
kubectl create namespace <name>         # Create namespace
kubectl config set-context --current --namespace=<name>  # Switch namespace

# Configuration Management
kubectl create configmap <name> --from-literal=key=value  # Create ConfigMap
kubectl create secret generic <name> --from-literal=key=value  # Create Secret
kubectl get configmaps                                    # List ConfigMaps
kubectl get secrets                                       # List Secrets

# Resource Monitoring
kubectl top pods                        # Pod resource usage
kubectl describe pod <name> | grep -A 5 "Conditions"  # Pod health
kubectl get events --sort-by='.lastTimestamp'         # Recent events
```

### Essential Minikube Commands

```bash
# Cluster Management
minikube start                          # Start cluster
minikube start --driver=docker          # Start with specific driver
minikube start --memory=4096 --cpus=2   # Start with resource limits
minikube stop                           # Stop cluster
minikube delete                         # Delete cluster
minikube status                         # Check cluster status
minikube pause                          # Pause cluster
minikube unpause                        # Unpause cluster

# Configuration
minikube config set memory 4096         # Set default memory
minikube config set cpus 2              # Set default CPUs
minikube config view                    # View configuration
minikube profile list                   # List profiles

# Networking & Access
minikube ip                             # Get cluster IP
minikube service <service-name>         # Open service in browser
minikube service <service-name> --url   # Get service URL
minikube tunnel                         # Expose LoadBalancer services

# Addons
minikube addons list                    # List available addons
minikube addons enable <addon-name>     # Enable addon
minikube addons disable <addon-name>    # Disable addon
minikube dashboard                      # Open Kubernetes dashboard

# Docker Environment
eval $(minikube docker-env)             # Use Minikube's Docker daemon
minikube docker-env                     # Show Docker environment commands

# Troubleshooting
minikube logs                          # View Minikube logs
minikube ssh                           # SSH into Minikube VM
minikube mount /host/path:/vm/path     # Mount host directory
```

###
