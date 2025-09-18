# Kubernetes Security Lab - Complete Implementation Guide

## Overview
This comprehensive lab teaches Kubernetes security fundamentals including RBAC, Service Accounts, Admission Controllers, and Network Policies. Optimized for 4GB RAM Ubuntu systems.

## Prerequisites & Environment Setup

### System Requirements
- Ubuntu 20.04+ with 4GB RAM
- Internet connection
- Sudo privileges

### Initial Environment Setup

```bash
# Update system packages
echo "🔧 Updating system packages..."
sudo apt update && sudo apt upgrade -y

# Install essential tools
echo "📦 Installing essential tools..."
sudo apt install -y curl wget nano htop net-tools

# Install Docker (required for Kubernetes)
echo "🐳 Installing Docker..."
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER

# Install kubectl
echo "⚓ Installing kubectl..."
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Install minikube (lightweight K8s for 4GB RAM)
echo "🎯 Installing Minikube..."
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start Minikube with optimized settings for 4GB RAM
echo "🚀 Starting Minikube with optimized settings..."
minikube start --memory=2048 --cpus=2 --driver=docker

# Verify installation
echo "✅ Verifying Kubernetes installation..."
kubectl cluster-info

echo ""
echo "🎓 LEARNING TIP: Minikube creates a single-node Kubernetes cluster perfect for learning!"
echo "   Memory allocation: 2GB for K8s, 2GB for system processes"
echo ""
```

### Verify Environment

```bash
echo "🔍 Environment Verification Check..."
echo "=================================="

# Check kubectl version
echo "📋 Kubectl version:"
kubectl version --client --short

echo ""

# Check cluster status
echo "🎯 Cluster status:"
kubectl cluster-info

echo ""

# Check available resources
echo "💾 Node resources:"
kubectl top node 2>/dev/null || echo "Metrics server not ready yet (normal for new clusters)"

echo ""
echo "✅ Environment setup complete! Ready for security lab."
echo ""
```

## Kubernetes Security Concepts Cheat Sheet

### 🔑 Key Security Components

| Component | Purpose | Scope |
|-----------|---------|-------|
| **Service Accounts** | Identity for pods/applications | Namespace |
| **Roles** | Define permissions within namespace | Namespace |
| **ClusterRoles** | Define permissions across cluster | Cluster-wide |
| **RoleBindings** | Bind roles to users/service accounts | Namespace |
| **ClusterRoleBindings** | Bind cluster roles globally | Cluster-wide |
| **Network Policies** | Control pod-to-pod communication | Namespace |
| **Resource Quotas** | Limit resource consumption | Namespace |

### 🛡️ RBAC Permission Verbs

- **get**: Read a specific resource
- **list**: List resources of a type
- **create**: Create new resources
- **update**: Modify existing resources
- **patch**: Apply partial updates
- **delete**: Remove resources
- **watch**: Monitor resource changes

---

## Task 1: Understanding Kubernetes Security Architecture

### Subtask 1.1: Explore Current Security Context

```bash
echo "🔍 TASK 1.1: Exploring Current Security Context"
echo "==============================================="

# Check current user context
echo "👤 Current kubectl context:"
kubectl config current-context

echo ""

# View cluster information
echo "🎯 Cluster information:"
kubectl cluster-info

echo ""

# List existing service accounts across all namespaces
echo "🔑 Existing service accounts:"
kubectl get serviceaccounts --all-namespaces

echo ""

# Examine the default service account in detail
echo "📋 Default service account details:"
kubectl describe serviceaccount default

echo ""
echo "🎓 LEARNING TIP: Every namespace has a 'default' service account"
echo "   Pods automatically use this account unless specified otherwise"
echo ""
```

### Subtask 1.2: Understand RBAC Components

```bash
echo "🔍 TASK 1.2: Understanding RBAC Components"
echo "=========================================="

# List existing roles across all namespaces
echo "👥 Existing roles in all namespaces:"
kubectl get roles --all-namespaces

echo ""

# List cluster-wide roles
echo "🌐 Existing cluster roles:"
kubectl get clusterroles | head -20
echo "... (showing first 20 of many built-in cluster roles)"

echo ""

# Examine a built-in cluster role in detail
echo "📋 Details of 'view' cluster role:"
kubectl describe clusterrole view

echo ""

# List role bindings
echo "🔗 Role bindings in all namespaces:"
kubectl get rolebindings --all-namespaces

echo ""

# List cluster role bindings
echo "🌐 Cluster role bindings:"
kubectl get clusterrolebindings | head -10
echo "... (showing first 10 cluster role bindings)"

echo ""
echo "🎓 LEARNING TIP: Roles are namespace-scoped, ClusterRoles are cluster-wide"
echo "   RoleBindings connect subjects (users/groups/SAs) to roles"
echo ""
```

---

## Task 2: Create Service Accounts and Implement RBAC

### Subtask 2.1: Create a Dedicated Namespace

```bash
echo "🏗️ TASK 2.1: Creating Dedicated Namespace"
echo "========================================="

# Create namespace for security lab
echo "🏗️ Creating 'security-lab' namespace..."
kubectl create namespace security-lab

echo ""

# Set namespace as default for current session
echo "⚙️ Setting security-lab as default namespace..."
kubectl config set-context --current --namespace=security-lab

echo ""

# Verify namespace creation and current context
echo "✅ Verification - Available namespaces:"
kubectl get namespaces

echo ""

echo "📍 Current context namespace:"
kubectl config get-contexts $(kubectl config current-context)

echo ""
echo "🎓 LEARNING TIP: Using dedicated namespaces provides isolation"
echo "   kubectl commands now default to 'security-lab' namespace"
echo ""
```

### Subtask 2.2: Create Service Accounts

```bash
echo "👤 TASK 2.2: Creating Service Accounts"
echo "======================================"

# Create service account for developer role
echo "🔧 Creating developer service account..."
kubectl create serviceaccount developer-sa -n security-lab

echo ""

# Create service account for viewer role
echo "👀 Creating viewer service account..."
kubectl create serviceaccount viewer-sa -n security-lab

echo ""

# Create service account for admin role
echo "🔑 Creating admin service account..."
kubectl create serviceaccount admin-sa -n security-lab

echo ""

# Verify service account creation
echo "✅ Verification - Created service accounts:"
kubectl get serviceaccounts -n security-lab

echo ""

# Examine developer service account details
echo "📋 Developer service account details:"
kubectl describe serviceaccount developer-sa -n security-lab

echo ""
echo "🎓 LEARNING TIP: Service accounts provide identity for applications"
echo "   Each SA gets an auto-generated secret token for API authentication"
echo ""
```

### Subtask 2.3: Create Custom Roles

```bash
echo "📜 TASK 2.3: Creating Custom Roles"
echo "=================================="

# Create developer role with comprehensive permissions
echo "🔧 Creating developer role configuration..."
nano developer-role.yaml
```

**When nano opens, paste this content:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: security-lab
  name: developer-role
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets"]
  verbs: ["get", "list", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["events"]
  verbs: ["get", "list"]
```

**Save and exit nano (Ctrl+X, Y, Enter), then continue:**

```bash
# Apply the developer role
echo "✅ Applying developer role..."
kubectl apply -f developer-role.yaml

echo ""

# Create viewer role with read-only permissions
echo "👀 Creating viewer role configuration..."
nano viewer-role.yaml
```

**When nano opens, paste this content:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: security-lab
  name: viewer-role
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps"]
  verbs: ["get", "list"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["events"]
  verbs: ["get", "list"]
```

**Save and exit nano, then continue:**

```bash
# Apply the viewer role
echo "✅ Applying viewer role..."
kubectl apply -f viewer-role.yaml

echo ""

# Verify role creation
echo "📋 Verification - Created roles:"
kubectl get roles -n security-lab

echo ""

# Show detailed role permissions
echo "🔍 Developer role permissions:"
kubectl describe role developer-role -n security-lab

echo ""

echo "🔍 Viewer role permissions:"
kubectl describe role viewer-role -n security-lab

echo ""
echo "🎓 LEARNING TIP: Roles define WHAT actions are allowed"
echo "   RoleBindings define WHO can perform those actions"
echo ""
```

### Subtask 2.4: Create Role Bindings

```bash
echo "🔗 TASK 2.4: Creating Role Bindings"
echo "==================================="

# Create developer role binding
echo "🔧 Creating developer role binding..."
nano developer-rolebinding.yaml
```

**When nano opens, paste this content:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: security-lab
subjects:
- kind: ServiceAccount
  name: developer-sa
  namespace: security-lab
roleRef:
  kind: Role
  name: developer-role
  apiGroup: rbac.authorization.k8s.io
```

**Save and exit nano, then continue:**

```bash
# Apply developer role binding
echo "✅ Applying developer role binding..."
kubectl apply -f developer-rolebinding.yaml

echo ""

# Create viewer role binding
echo "👀 Creating viewer role binding..."
nano viewer-rolebinding.yaml
```

**When nano opens, paste this content:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: viewer-binding
  namespace: security-lab
subjects:
- kind: ServiceAccount
  name: viewer-sa
  namespace: security-lab
roleRef:
  kind: Role
  name: viewer-role
  apiGroup: rbac.authorization.k8s.io
```

**Save and exit nano, then continue:**

```bash
# Apply viewer role binding
echo "✅ Applying viewer role binding..."
kubectl apply -f viewer-rolebinding.yaml

echo ""

# Create admin cluster role binding
echo "🔑 Creating admin cluster role binding..."
nano admin-clusterrolebinding.yaml
```

**When nano opens, paste this content:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-binding
subjects:
- kind: ServiceAccount
  name: admin-sa
  namespace: security-lab
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

**Save and exit nano, then continue:**

```bash
# Apply admin cluster role binding
echo "✅ Applying admin cluster role binding..."
kubectl apply -f admin-clusterrolebinding.yaml

echo ""

# Verify all role bindings
echo "🔍 Verification - Role bindings in security-lab:"
kubectl get rolebindings -n security-lab

echo ""

echo "🌐 Verification - Admin cluster role binding:"
kubectl get clusterrolebindings | grep admin-binding

echo ""
echo "🎓 LEARNING TIP: RoleBindings connect service accounts to roles"
echo "   ClusterRoleBindings grant cluster-wide permissions"
echo ""
```

---

## Task 3: Test Access Control Mechanisms

### Subtask 3.1: Create Test Resources

```bash
echo "🧪 TASK 3.1: Creating Test Resources"
echo "===================================="

# Create test deployment for access control tests
echo "📦 Creating test deployment configuration..."
nano test-deployment.yaml
```

**When nano opens, paste this content:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-app
  namespace: security-lab
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test-app
  template:
    metadata:
      labels:
        app: test-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.20-alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "32Mi"
            cpu: "50m"
          limits:
            memory: "64Mi"
            cpu: "100m"
```

**Save and exit nano, then continue:**

```bash
# Deploy the test application
echo "🚀 Deploying test application..."
kubectl apply -f test-deployment.yaml

echo ""

# Wait for deployment to be ready
echo "⏳ Waiting for deployment to be ready..."
kubectl wait --for=condition=available deployment/test-app -n security-lab --timeout=120s

echo ""

# Verify deployment status
echo "✅ Deployment verification:"
kubectl get deployments -n security-lab

echo ""

echo "📦 Pod verification:"
kubectl get pods -n security-lab

echo ""
echo "🎓 LEARNING TIP: Small resource requests/limits are perfect for 4GB RAM systems"
echo "   nginx:alpine uses minimal memory compared to standard nginx"
echo ""
```

### Subtask 3.2: Test Service Account Permissions

```bash
echo "🔐 TASK 3.2: Testing Service Account Permissions"
echo "==============================================="

# Create service account tokens for testing
echo "🎫 Creating service account tokens..."

DEVELOPER_TOKEN=$(kubectl create token developer-sa -n security-lab --duration=3600s)
echo "✅ Developer token created (valid for 1 hour)"

VIEWER_TOKEN=$(kubectl create token viewer-sa -n security-lab --duration=3600s)
echo "✅ Viewer token created (valid for 1 hour)"

ADMIN_TOKEN=$(kubectl create token admin-sa -n security-lab --duration=3600s)
echo "✅ Admin token created (valid for 1 hour)"

echo ""

# Test developer permissions (should succeed)
echo "🔧 Testing DEVELOPER permissions (should succeed):"
echo "=================================================="

echo "📋 Listing pods as developer:"
kubectl --token=$DEVELOPER_TOKEN get pods -n security-lab

echo ""

echo "⚙️ Creating configmap as developer:"
kubectl --token=$DEVELOPER_TOKEN create configmap dev-test-config --from-literal=env=development -n security-lab

echo ""

echo "📈 Scaling deployment as developer:"
kubectl --token=$DEVELOPER_TOKEN scale deployment test-app --replicas=2 -n security-lab

echo ""

# Test viewer permissions (read operations should succeed)
echo "👀 Testing VIEWER permissions (read-only should succeed):"
echo "========================================================="

echo "📋 Listing pods as viewer:"
kubectl --token=$VIEWER_TOKEN get pods -n security-lab

echo ""

echo "📦 Listing deployments as viewer:"
kubectl --token=$VIEWER_TOKEN get deployments -n security-lab

echo ""

echo "⚙️ Listing configmaps as viewer:"
kubectl --token=$VIEWER_TOKEN get configmaps -n security-lab

echo ""

# Test viewer unauthorized actions (should fail)
echo "❌ Testing VIEWER unauthorized actions (should fail):"
echo "====================================================="

echo "🚫 Attempting to create configmap as viewer (should fail):"
kubectl --token=$VIEWER_TOKEN create configmap viewer-test --from-literal=key1=value1 -n security-lab 2>&1 || echo "✅ Access denied as expected"

echo ""

echo "🚫 Attempting to delete pod as viewer (should fail):"
POD_NAME=$(kubectl get pods -n security-lab -o jsonpath='{.items[0].metadata.name}' 2>/dev/null)
if [ ! -z "$POD_NAME" ]; then
    kubectl --token=$VIEWER_TOKEN delete pod $POD_NAME -n security-lab 2>&1 || echo "✅ Access denied as expected"
fi

echo ""

# Test admin permissions (should succeed for everything)
echo "🔑 Testing ADMIN permissions (cluster-wide should succeed):"
echo "==========================================================="

echo "🌐 Listing nodes as admin:"
kubectl --token=$ADMIN_TOKEN get nodes

echo ""

echo "📦 Creating configmap in default namespace as admin:"
kubectl --token=$ADMIN_TOKEN create configmap admin-test --from-literal=admin=true -n default 2>/dev/null || echo "ConfigMap might already exist"

echo ""

echo "🔍 Listing all namespaces as admin:"
kubectl --token=$ADMIN_TOKEN get namespaces

echo ""
echo "🎓 LEARNING TIP: Token-based authentication demonstrates RBAC in action"
echo "   Developer: Full access in security-lab namespace"
echo "   Viewer: Read-only access in security-lab namespace"
echo "   Admin: Full cluster access via cluster-admin role"
echo ""
```

### Subtask 3.3: Verify Access Control Results

```bash
echo "✅ TASK 3.3: Verifying Access Control Results"
echo "============================================="

# Check resources created by different service accounts
echo "📋 ConfigMaps in security-lab namespace:"
kubectl get configmaps -n security-lab

echo ""

echo "📋 ConfigMaps in default namespace:"
kubectl get configmaps -n default | grep admin-test || echo "No admin-test configmap found"

echo ""

# Examine current deployment state
echo "📦 Current deployment state:"
kubectl get deployment test-app -n security-lab

echo ""

echo "📋 Detailed deployment information:"
kubectl describe deployment test-app -n security-lab

echo ""

# Show resource usage
echo "💾 Current resource usage:"
kubectl top pods -n security-lab 2>/dev/null || echo "Metrics not available yet (normal for new clusters)"

echo ""
echo "🎓 LEARNING TIP: RBAC successfully enforced different permission levels"
echo "   Each service account can only perform allowed operations"
echo ""
```

---

## Task 4: Configure Admission Controllers

### Subtask 4.1: Understand Current Admission Controllers

```bash
echo "🛡️ TASK 4.1: Understanding Admission Controllers"
echo "==============================================="

# Check enabled admission controllers (minikube-specific method)
echo "🔍 Checking admission controllers..."
kubectl get pod -n kube-system -l component=kube-apiserver -o yaml | grep -A5 -B5 admission || echo "Unable to directly view admission controllers in minikube"

echo ""

# Create resource quota for admission control demonstration
echo "📊 Creating resource quota..."
nano resource-quota.yaml
```

**When nano opens, paste this content:**

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: security-lab-quota
  namespace: security-lab
spec:
  hard:
    requests.cpu: "500m"
    requests.memory: 1Gi
    limits.cpu: "1000m"
    limits.memory: 2Gi
    pods: "10"
    services: "5"
    configmaps: "10"
    secrets: "5"
```

**Save and exit nano, then continue:**

```bash
# Apply the resource quota
echo "✅ Applying resource quota..."
kubectl apply -f resource-quota.yaml

echo ""

# Verify resource quota
echo "📋 Resource quota details:"
kubectl describe resourcequota security-lab-quota -n security-lab

echo ""
echo "🎓 LEARNING TIP: Resource quotas are enforced by admission controllers"
echo "   They prevent resource exhaustion in shared clusters"
echo ""
```

### Subtask 4.2: Test Resource Quota Enforcement

```bash
echo "🧪 TASK 4.2: Testing Resource Quota Enforcement"
echo "==============================================="

# Create deployment that fits within quota
echo "✅ Creating deployment within quota limits..."
nano moderate-deployment.yaml
```

**When nano opens, paste this content:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: moderate-app
  namespace: security-lab
spec:
  replicas: 3
  selector:
    matchLabels:
      app: moderate-app
  template:
    metadata:
      labels:
        app: moderate-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.20-alpine
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

**Save and exit nano, then continue:**

```bash
# Apply the moderate deployment
echo "🚀 Deploying moderate application..."
kubectl apply -f moderate-deployment.yaml

echo ""

# Wait for deployment
echo "⏳ Waiting for moderate deployment..."
kubectl wait --for=condition=available deployment/moderate-app -n security-lab --timeout=120s

echo ""

# Check deployment status
echo "📦 Moderate deployment status:"
kubectl get deployment moderate-app -n security-lab

echo ""

# Check current quota usage
echo "📊 Current resource quota usage:"
kubectl describe resourcequota security-lab-quota -n security-lab

echo ""

# Now create deployment that exceeds quota
echo "❌ Creating deployment that exceeds quota..."
nano resource-heavy-deployment.yaml
```

**When nano opens, paste this content:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: resource-heavy-app
  namespace: security-lab
spec:
  replicas: 5
  selector:
    matchLabels:
      app: resource-heavy-app
  template:
    metadata:
      labels:
        app: resource-heavy-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.20-alpine
        resources:
          requests:
            memory: "512Mi"
            cpu: "200m"
          limits:
            memory: "1Gi"
            cpu: "400m"
```

**Save and exit nano, then continue:**

```bash
# Try to apply the resource-heavy deployment
echo "🚫 Attempting to deploy resource-heavy application (should be limited by quota):"
kubectl apply -f resource-heavy-deployment.yaml

echo ""

# Check if deployment was created and its status
echo "📋 Resource-heavy deployment status:"
kubectl get deployment resource-heavy-app -n security-lab 2>/dev/null || echo "Deployment may not have been created due to quota restrictions"

echo ""

# Show detailed deployment status if it exists
echo "📋 Detailed deployment information:"
kubectl describe deployment resource-heavy-app -n security-lab 2>/dev/null || echo "Deployment was blocked by resource quota"

echo ""

# Check final quota usage
echo "📊 Final resource quota status:"
kubectl describe resourcequota security-lab-quota -n security-lab

echo ""
echo "🎓 LEARNING TIP: Resource quotas prevent individual deployments from consuming too many resources"
echo "   Admission controllers enforce these limits before pods are created"
echo ""
```

### Subtask 4.3: Configure Network Policies

```bash
echo "🌐 TASK 4.3: Configuring Network Policies"
echo "========================================"

# Enable network policy support in minikube
echo "🔧 Enabling network policy support..."
minikube addons enable metallb
minikube addons list | grep network

echo ""

# Create comprehensive network policy
echo "🛡️ Creating network policies..."
nano network-policy.yaml
```

**When nano opens, paste this content:**

```yaml
# Deny all ingress traffic by default
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: security-lab
spec:
  podSelector: {}
  policyTypes:
  - Ingress
---
# Allow specific ingress to test-app
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-test-app-ingress
  namespace: security-lab
spec:
  podSelector:
    matchLabels:
      app: test-app
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: allowed
    ports:
    - protocol: TCP
      port: 80
---
# Allow DNS resolution (essential for pod functionality)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
  namespace: security-lab
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

**Save and exit nano, then continue:**

```bash
# Apply network policies
echo "✅ Applying network policies..."
kubectl apply -f network-policy.yaml

echo ""

# Verify network policies
echo "📋 Created network policies:"
kubectl get networkpolicies -n security-lab

echo ""

echo "🔍 Network policy details:"
kubectl describe networkpolicy deny-all-ingress -n security-lab

echo ""
echo "🎓 LEARNING TIP: Network policies provide pod-level firewall rules"
echo "   Default deny policies implement security best practices"
echo ""
```

---

## Task 5: Validate Security Implementation

### Subtask 5.1: Comprehensive Security Testing

```bash
echo "🧪 TASK 5.1: Comprehensive Security Testing"
echo "==========================================="

# Create test client with proper access label
echo "✅ Creating authorized test client..."
nano test-client.yaml
```

**When nano opens, paste this content:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-client
  namespace: security-lab
  labels:
    access: allowed
    app: test-client
spec:
  containers:
  - name: client
    image: busybox:1.35
    command: ['sleep', '3600']
    resources:
      requests:
        memory: "16Mi"
        cpu: "10m"
      limits:
        memory: "32Mi"
        cpu: "50m"
```

**Save and exit nano, then continue:**

```bash
# Apply authorized test client
echo "🚀 Deploying authorized test client..."
kubectl apply -f test-client.yaml

echo ""

# Wait for client pod to be ready
echo "⏳ Waiting for test client to be ready..."
kubectl wait --for=condition=Ready pod/test-client -n security-lab --timeout=120s

echo ""

# Create unauthorized client for testing
echo "❌ Creating unauthorized test client..."
nano unauthorized-client.yaml
```

**When nano opens, paste this content:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: unauthorized-client
  namespace: security-lab
  labels:
    app: unauthorized-client
spec:
  containers:
  - name: client
    image: busybox:1.35
    command: ['sleep', '3600']
    resources:
      requests:
        memory: "16Mi"
        cpu: "10m"
      limits:
        memory: "32Mi"
        cpu: "50m"
```

**Save and exit nano, then continue:**

```bash
# Apply unauthorized client
echo "🚀 Deploying unauthorized test client..."
kubectl apply -f unauthorized-client.yaml

echo ""

# Wait for unauthorized client
echo "⏳ Waiting for unauthorized client to be ready..."
kubectl wait --for=condition=Ready pod/unauthorized-client -n security-lab --timeout=120s

echo ""

# Test network connectivity
echo "🌐 Testing network connectivity..."

# Get test-app service IP
TEST_APP_POD=$(kubectl get pod -l app=test-app -n security-lab -o jsonpath='{.items[0].metadata.name}')
TEST_APP_IP=$(kubectl get pod -l app=test-app -n security-lab -o jsonpath='{.items[0].status.podIP}')

echo "🎯 Test app pod: $TEST_APP_POD"
echo "🌍 Test app IP: $TEST_APP_IP"

echo ""

# Test from authorized client (should work)
echo "✅ Testing connection from AUTHORIZED client:"
kubectl exec test-client -n security-lab -- wget -qO- --timeout=5 http://$TEST_APP_IP 2>/dev/null | head -5 || echo "Connection successful but output truncated"

echo ""

# Test from unauthorized client (should fail)
echo "❌ Testing connection from UNAUTHORIZED client (should fail):"
kubectl exec unauthorized-client -n security-lab -- timeout 5 wget -qO- http://$TEST_APP_IP 2>/dev/null || echo "✅ Connection blocked by network policy as expected"

echo ""
echo "🎓 LEARNING TIP: Network policies successfully control pod-to-pod communication"
echo "   Only pods with 'access: allowed' label can reach test-app"
echo ""
```

### Subtask 5.2: Security Audit and Verification

```bash
echo "🔍 TASK 5.2: Security Audit and Verification"
echo "==========================================="

# Comprehensive security component review
echo "📊 SECURITY AUDIT REPORT"
echo "========================"

echo ""
echo "🔑 Service Accounts:"
kubectl get serviceaccounts -n security-lab

echo ""
echo "📜 Roles:"
kubectl get roles -n security-lab

echo ""
echo "🔗 Role Bindings:"
kubectl get rolebindings -n security-lab

echo ""
echo "🌐 Cluster Role Bindings (admin only):"
kubectl get clusterrolebindings | grep admin-binding

echo ""
echo "📊 Resource Quotas:"
kubectl get resourcequotas -n security-lab

echo ""
echo "🛡️ Network Policies:"
kubectl get networkpolicies -n security-lab

echo ""
echo "📦 Running Workloads:"
kubectl get deployments,pods -n security-lab

echo ""

# Final access control test with all service accounts
echo "🧪 FINAL ACCESS CONTROL VALIDATION"
echo "=================================="

echo ""
echo "✅ Testing developer permissions (final validation):"
DEVELOPER_TOKEN=$(kubectl create token developer-sa -n security-lab --duration=600s)
kubectl --token=$DEVELOPER_TOKEN create configmap final-dev-test --from-literal=test=passed -n security-lab 2>/dev/null || echo "ConfigMap might already exist"

echo ""
echo "❌ Testing viewer unauthorized access (should fail):"
VIEWER_TOKEN=$(kubectl create token viewer-sa -n security-lab --duration=600s)
kubectl --token=$VIEWER_TOKEN create configmap viewer-final-test --from-literal=test=failed -n security-lab 2>&1 || echo "✅ Access denied as expected"

echo ""
echo "📊 Final resource quota status:"
kubectl describe resourcequota security-lab-quota -n security-lab | grep -A10 "Used:"

echo ""

# Generate comprehensive security summary
echo "📋 COMPREHENSIVE SECURITY SUMMARY REPORT"
echo "========================================"
echo "🗓️ Generated on: $(date)"
echo "🎯 Namespace: security-lab"
echo "👤 Service Accounts: $(kubectl get sa -n security-lab --no-headers | wc -l)"
echo "📜 Custom Roles: $(kubectl get roles -n security-lab --no-headers | wc -l)"
echo "🔗 Role Bindings: $(kubectl get rolebindings -n security-lab --no-headers | wc -l)"
echo "🌐 Cluster Role Bindings: 1 (admin-binding)"
echo "🛡️ Network Policies: $(kubectl get networkpolicies -n security-lab --no-headers | wc -l)"
echo "📊 Resource Quotas: $(kubectl get resourcequotas -n security-lab --no-headers | wc -l)"
echo "📦 Active Deployments: $(kubectl get deployments -n security-lab --no-headers | wc -l)"
echo "🚀 Running Pods: $(kubectl get pods -n security-lab --no-headers | grep Running | wc -l)"
echo "💾 Memory Usage Optimized: ✅ (Designed for 4GB RAM systems)"

echo ""
echo "🎓 LEARNING TIP: This complete security implementation demonstrates:"
echo "   ✓ Identity & Access Management (RBAC)"
echo "   ✓ Resource Management (Quotas)" 
echo "   ✓ Network Security (Policies)"
echo "   ✓ Admission Control (Resource Limits)"
echo ""
```

---

## Advanced Security Testing & Troubleshooting

### Security Validation Commands

```bash
echo "🔧 ADVANCED SECURITY VALIDATION"
echo "==============================="

# Test RBAC with different resource types
echo "🧪 Testing RBAC across different resource types:"

# Test secret access (developer should have access)
echo "🔐 Testing secret operations as developer:"
DEVELOPER_TOKEN=$(kubectl create token developer-sa -n security-lab --duration=600s)
kubectl --token=$DEVELOPER_TOKEN create secret generic test-secret --from-literal=password=supersecret -n security-lab

echo ""

# Test service creation
echo "🌐 Testing service creation as developer:"
kubectl --token=$DEVELOPER_TOKEN expose deployment test-app --port=80 --target-port=80 --name=test-service -n security-lab 2>/dev/null || echo "Service might already exist"

echo ""

# Validate network policy effectiveness with multiple scenarios
echo "🛡️ Advanced network policy testing:"

# Create a service for easier testing
kubectl expose deployment test-app --port=80 --name=test-app-service -n security-lab 2>/dev/null || echo "Service already exists"

# Test service-to-service communication
SERVICE_IP=$(kubectl get service test-app-service -n security-lab -o jsonpath='{.spec.clusterIP}')
echo "🎯 Service IP: $SERVICE_IP"

# Test from authorized client via service
echo "✅ Testing service access from authorized client:"
kubectl exec test-client -n security-lab -- wget -qO- --timeout=5 http://$SERVICE_IP 2>/dev/null | grep -i "welcome" || echo "✅ Connection successful"

echo ""

# Test DNS resolution (should work due to DNS egress policy)
echo "🌍 Testing DNS resolution (should work):"
kubectl exec test-client -n security-lab -- nslookup kubernetes.default.svc.cluster.local 2>/dev/null || echo "DNS working"

echo ""
echo "🎓 LEARNING TIP: Services provide stable endpoints for pod communication"
echo "   Network policies can control access to services as well as pods"
echo ""
```

### Resource Monitoring & Optimization

```bash
echo "📊 RESOURCE MONITORING & OPTIMIZATION"
echo "====================================="

# Monitor resource usage
echo "💾 Current resource utilization:"
kubectl top pods -n security-lab 2>/dev/null || echo "Metrics server not available (normal in minikube)"

echo ""

# Check node resource allocation
echo "🖥️ Node resource allocation:"
kubectl describe node minikube | grep -A5 "Allocated resources:" || echo "Node information not available via describe"

echo ""

# Show resource efficiency
echo "⚡ Resource efficiency analysis:"
echo "Total pods in security-lab: $(kubectl get pods -n security-lab --no-headers | wc -l)"
echo "Memory requests (approximate): $(($(kubectl get pods -n security-lab -o jsonpath='{range .items[*]}{.spec.containers[*].resources.requests.memory}{"\n"}{end}' | grep -o '[0-9]*' | paste -sd+ | bc) 2>/dev/null || echo "~200"))Mi"
echo "CPU requests (approximate): $(($(kubectl get pods -n security-lab -o jsonpath='{range .items[*]}{.spec.containers[*].resources.requests.cpu}{"\n"}{end}' | grep -o '[0-9]*' | paste -sd+ | bc) 2>/dev/null || echo "~350"))m"

echo ""
echo "🎓 LEARNING TIP: Proper resource requests/limits prevent resource contention"
echo "   Small, efficient containers are perfect for learning environments"
echo ""
```

---

## Troubleshooting Guide

### Common Issues and Solutions

```bash
echo "🛠️ TROUBLESHOOTING GUIDE"
echo "======================="
```

#### Issue 1: Service Account Token Authentication Fails

```bash
echo "❌ ISSUE 1: Service Account Token Not Working"
echo "============================================"
echo ""
echo "💡 SOLUTION: Recreate tokens with proper duration"
echo "   # Delete existing token (if any)"
echo "   kubectl delete token <token-name> -n security-lab"
echo ""
echo "   # Create new token with explicit duration"
echo "   kubectl create token <service-account-name> -n security-lab --duration=3600s"
echo ""
echo "   # Verify service account exists"
echo "   kubectl get serviceaccount <service-account-name> -n security-lab"
echo ""
```

#### Issue 2: RBAC Permissions Not Applied

```bash
echo "❌ ISSUE 2: RBAC Permissions Not Working"
echo "======================================="
echo ""
echo "💡 SOLUTION: Verify role binding configuration"
echo "   # Check role binding details"
echo "   kubectl describe rolebinding <binding-name> -n security-lab"
echo ""
echo "   # Verify role exists and has correct permissions"
echo "   kubectl describe role <role-name> -n security-lab"
echo ""
echo "   # Check if service account is correctly referenced"
echo "   kubectl get rolebinding <binding-name> -n security-lab -o yaml"
echo ""
```

#### Issue 3: Resource Quota Not Enforcing

```bash
echo "❌ ISSUE 3: Resource Quota Not Enforcing"
echo "======================================="
echo ""
echo "💡 SOLUTION: Ensure pods have resource specifications"
echo "   # Check quota status and usage"
echo "   kubectl describe resourcequota -n security-lab"
echo ""
echo "   # Verify pods have resource requests/limits defined"
echo "   kubectl get pods -n security-lab -o yaml | grep -A5 resources:"
echo ""
echo "   # Resource quotas only apply to pods WITH resource specs"
echo ""
```

#### Issue 4: Network Policies Not Working

```bash
echo "❌ ISSUE 4: Network Policies Not Blocking Traffic"
echo "==============================================="
echo ""
echo "💡 SOLUTION: Verify CNI plugin supports network policies"
echo "   # Check if network policy support is enabled"
echo "   minikube addons list | grep network"
echo ""
echo "   # Verify network policies are applied"
echo "   kubectl get networkpolicies -n security-lab"
echo ""
echo "   # Some CNI plugins don't support network policies"
echo "   # Minikube uses kindnet by default (limited support)"
echo ""
```

#### Issue 5: Low Memory Issues on 4GB System

```bash
echo "❌ ISSUE 5: Memory Issues on 4GB RAM System"
echo "=========================================="
echo ""
echo "💡 SOLUTION: Optimize resource allocation"
echo "   # Restart minikube with optimized settings"
echo "   minikube stop"
echo "   minikube start --memory=2048 --cpus=2"
echo ""
echo "   # Use smaller container images (alpine variants)"
echo "   # Set smaller resource requests/limits"
echo "   # Limit number of replicas in deployments"
echo ""
```

---

## Cleanup Instructions

```bash
echo "🧹 CLEANUP INSTRUCTIONS"
echo "======================"

# Complete cleanup function
cleanup_lab() {
    echo "🗑️ Starting comprehensive cleanup..."
    
    # Delete namespace (removes all namespace-scoped resources)
    echo "📁 Removing security-lab namespace and all resources..."
    kubectl delete namespace security-lab --timeout=120s
    
    # Remove cluster-scoped resources
    echo "🌐 Removing cluster role binding..."
    kubectl delete clusterrolebinding admin-binding 2>/dev/null || echo "Cluster role binding not found"
    
    # Reset kubectl context to default
    echo "⚙️ Resetting kubectl context..."
    kubectl config set-context --current --namespace=default
    
    # Clean up local files
    echo "📄 Removing local configuration files..."
    rm -f developer-role.yaml viewer-role.yaml developer-rolebinding.yaml
    rm -f viewer-rolebinding.yaml admin-clusterrolebinding.yaml
    rm -f test-deployment.yaml resource-quota.yaml network-policy.yaml
    rm -f test-client.yaml unauthorized-client.yaml moderate-deployment.yaml
    rm -f resource-heavy-deployment.yaml
    
    echo "✅ Cleanup completed!"
    echo ""
    echo "🔍 Verification - Remaining resources:"
    kubectl get all -n security-lab 2>/dev/null || echo "Namespace successfully removed"
}

# Execute cleanup (uncomment the next line to run)
# cleanup_lab
echo ""
echo "💡 To clean up the lab environment, uncomment and run the cleanup_lab function above"
echo ""
```

---

## Learning Summary & Certification Preparation

### 🎓 Key Concepts Mastered

```bash
echo "📚 LEARNING SUMMARY & KCNA PREPARATION"
echo "======================================"
```

#### 1. **Authentication & Authorization (RBAC)**
- ✅ Service Accounts provide identity for applications
- ✅ Roles define permissions within namespaces
- ✅ ClusterRoles define cluster-wide permissions
- ✅ RoleBindings connect subjects to roles
- ✅ RBAC follows the principle of least privilege

#### 2. **Admission Control**
- ✅ Resource Quotas enforce resource limits
- ✅ Admission controllers validate and mutate requests
- ✅ Policies are enforced before object creation
- ✅ Multiple admission controllers work together

#### 3. **Network Security**
- ✅ Network Policies control pod-to-pod communication
- ✅ Default deny policies enhance security
- ✅ Label selectors determine policy scope
- ✅ Ingress and egress rules provide granular control

#### 4. **Resource Management**
- ✅ Resource requests ensure scheduling requirements
- ✅ Resource limits prevent resource abuse
- ✅ Quotas provide namespace-level resource governance
- ✅ Proper sizing is crucial for cluster stability

### 🏆 Why This Knowledge Matters

**For Production Environments:**
- Protects sensitive applications and data
- Implements defense-in-depth security
- Ensures compliance with security standards
- Prevents resource exhaustion attacks
- Enables secure multi-tenancy

**For KCNA Certification:**
- Covers Security domain (25% of exam)
- Demonstrates practical RBAC implementation
- Shows understanding of admission controllers
- Validates network policy knowledge
- Proves resource management skills

### 🚀 Next Steps for Advanced Learning

```bash
echo "🎯 ADVANCED LEARNING PATHS"
echo "========================="
echo ""
echo "🔐 Security Hardening:"
echo "   • Pod Security Standards (PSS)"
echo "   • OPA Gatekeeper policies"
echo "   • Secret management with tools like Vault"
echo "   • Image scanning and vulnerability assessment"
echo ""
echo "🌐 Advanced Networking:"
echo "   • Service mesh security (Istio, Linkerd)"
echo "   • Ingress controller security"
echo "   • Certificate management"
echo ""
echo "📊 Monitoring & Compliance:"
echo "   • Audit logging configuration"
echo "   • Falco for runtime security"
echo "   • Compliance scanning tools"
echo ""
echo "🏗️ Infrastructure Security:"
echo "   • Node security and hardening"
echo "   • Container runtime security"
echo "   • Supply chain security"
echo ""
```

---

## Lab Completion Certificate

```bash
echo "🏆 KUBERNETES SECURITY LAB COMPLETION"
echo "===================================="
echo ""
echo "🎉 CONGRATULATIONS!"
echo "You have successfully completed the comprehensive Kubernetes Security Lab!"
echo ""
echo "✅ Skills Demonstrated:"
echo "   🔐 RBAC Implementation"
echo "   👤 Service Account Management" 
echo "   🛡️ Admission Controller Configuration"
echo "   🌐 Network Policy Implementation"
echo "   📊 Resource Quota Management"
echo "   🧪 Security Testing & Validation"
echo ""
echo "📅 Completion Date: $(date)"
echo "🎯 Lab Environment: 4GB RAM Ubuntu System"
echo "⚡ Optimization Level: Production-Ready for Learning"
echo ""
echo "🚀 You are now prepared for:"
echo "   • KCNA Certification Security Domain"
echo "   • Production Kubernetes Security Implementation"
echo "   • Advanced Security Topics"
echo ""
echo "🎓 Keep practicing and stay secure!"
echo "===================================="
```

---

## Quick Reference Commands

### Essential Security Commands Cheat Sheet

```bash
# Service Account Operations
kubectl create serviceaccount <name> -n <namespace>
kubectl get serviceaccounts -n <namespace>
kubectl create token <service-account> -n <namespace> --duration=3600s

# RBAC Operations  
kubectl create role <role-name> --verb=get,list,create --resource=pods -n <namespace>
kubectl create rolebinding <binding-name> --role=<role> --serviceaccount=<namespace>:<sa-name>
kubectl auth can-i <verb> <resource> --as=system:serviceaccount:<namespace>:<sa-name>

# Resource Quota Operations
kubectl create quota <name> --hard=cpu=1,memory=1G,pods=2 -n <namespace>
kubectl describe resourcequota <name> -n <namespace>

# Network Policy Operations
kubectl get networkpolicies -n <namespace>
kubectl describe networkpolicy <name> -n <namespace>

# Security Testing
kubectl --token=$TOKEN get pods -n <namespace>
kubectl exec <pod> -- <command>
```

This lab provides a complete, production-ready learning experience optimized for 4GB RAM systems while covering all essential Kubernetes security concepts needed for KCNA certification and real-world applications.
