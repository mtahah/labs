# Lab 9: Mastering Kubernetes Services - Complete Deep Learning Guide

## 🎯 Enhanced Learning Objectives

By the end of this comprehensive lab, you will have mastered:
- **Service Architecture**: Deep understanding of all Kubernetes service types and their internal mechanisms
- **Network Flow**: Complete grasp of packet routing from client to pod through services
- **Service Discovery**: Advanced DNS resolution patterns and service mesh concepts
- **Production Deployment**: Real-world service configuration and troubleshooting expertise
- **Performance Optimization**: Load balancing strategies and connection management
- **Security Implementation**: Service-level security controls and best practices

## 📚 Prerequisites Validation

Let's ensure your environment is properly configured and verify your foundational knowledge.

### Environment Setup and Validation

```bash
# Environment setup and validation commands
echo "=== KUBERNETES SERVICES LAB SETUP ==="
echo "🚀 Initializing comprehensive Kubernetes services learning environment..."
echo

# Create organized lab directory structure
mkdir -p ~/k8s-services-lab/{deployments,services,configs,troubleshooting,manifests}
cd ~/k8s-services-lab

echo "📁 Created organized lab directory structure"
echo "Directory: $(pwd)"
ls -la
echo

# Validate Kubernetes cluster availability
echo "🔍 Validating Kubernetes cluster status..."
kubectl cluster-info
echo

# Check node readiness
echo "🖥️  Checking cluster nodes..."
kubectl get nodes -o wide
echo

# Verify essential system pods
echo "🔧 Verifying essential cluster components..."
kubectl get pods -n kube-system --field-selector=status.phase=Running
echo

# Install/verify required tools
echo "🛠️  Installing and verifying required tools..."
which curl || (echo "Installing curl..." && apt-get update && apt-get install -y curl)
which wget || (echo "Installing wget..." && apt-get update && apt-get install -y wget)
which jq || (echo "Installing jq..." && apt-get update && apt-get install -y jq)

echo "✅ Environment validation complete!"
echo "💡 Learning Tip: Always verify your cluster health before deploying services"
echo "🔧 Troubleshooting: If cluster components are not running, check system logs with 'kubectl logs -n kube-system'"
```

## 🎓 Kubernetes Services Deep Dive Theory

### Understanding Service Architecture

**What are Kubernetes Services?**
Services are an abstraction layer that provides a stable network endpoint for a dynamic set of pods. They solve the fundamental problem of pod IP volatility in Kubernetes clusters.

**Core Service Components:**
1. **Service Object**: Defines the service specification
2. **Endpoints**: Tracks the IP addresses of backing pods
3. **kube-proxy**: Implements service networking rules on each node
4. **iptables/IPVS**: Actual traffic routing mechanism

### Service Types Comparison

| Service Type | Use Case | Accessibility | External IP | Port Range |
|--------------|----------|---------------|-------------|------------|
| ClusterIP | Internal communication | Cluster-only | No | Any |
| NodePort | Development/Testing | Node IP + Port | No | 30000-32767 |
| LoadBalancer | Production external access | External LB | Yes | Any |
| ExternalName | External service mapping | DNS CNAME | No | N/A |

## 🏗️ Task 1: Advanced Application Deployment with Service Foundations

### Subtask 1.1: Deploy Multi-Tier Application Architecture

```bash
# Deploy comprehensive multi-tier application
echo "=== TASK 1: ADVANCED APPLICATION DEPLOYMENT ==="
echo "🏗️  Deploying multi-tier application architecture..."
echo

# Create namespace for better organization
echo "📦 Creating dedicated namespace..."
kubectl create namespace services-lab
kubectl config set-context --current --namespace=services-lab
echo "Active namespace: $(kubectl config view --minify -o jsonpath='{.contexts[0].context.namespace}')"
echo

# Create enhanced nginx deployment with resource limits and health checks
echo "🔧 Creating production-ready nginx deployment..."
nano deployments/nginx-deployment.yaml
```

**File Content for nginx-deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  namespace: services-lab
  labels:
    app: nginx
    tier: frontend
    version: v1
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
      app: nginx
      tier: frontend
  template:
    metadata:
      labels:
        app: nginx
        tier: frontend
        version: v1
    spec:
      containers:
      - name: nginx
        image: nginx:1.21-alpine
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
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
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        env:
        - name: NGINX_PORT
          value: "80"
        volumeMounts:
        - name: nginx-config
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: nginx-config
        configMap:
          name: nginx-config
          optional: true
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
```

```bash
# Create ConfigMap for nginx configuration
echo "⚙️  Creating nginx configuration..."
nano configs/nginx-configmap.yaml
```

**File Content for nginx-configmap.yaml:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: services-lab
data:
  default.conf: |
    server {
        listen 80;
        server_name localhost;
        
        # Add server identification for load balancing tests
        add_header X-Server-ID $hostname;
        
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
        
        # Health check endpoint
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
        
        # Metrics endpoint for monitoring
        location /metrics {
            access_log off;
            return 200 "nginx_up 1\n";
            add_header Content-Type text/plain;
        }
    }
```

```bash
# Deploy the application stack
echo "🚀 Deploying application components..."
kubectl apply -f configs/nginx-configmap.yaml
kubectl apply -f deployments/nginx-deployment.yaml
echo

# Verify deployment status with detailed monitoring
echo "🔍 Monitoring deployment progress..."
kubectl rollout status deployment/nginx-app --timeout=300s
echo

# Display comprehensive deployment information
echo "📊 Deployment Status Overview:"
kubectl get deployments -o wide
echo
kubectl get replicasets -l app=nginx -o wide
echo
kubectl get pods -l app=nginx -o wide --show-labels
echo

# Verify pod readiness and health
echo "🏥 Checking pod health status..."
kubectl get pods -l app=nginx -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.phase}{"\t"}{.status.conditions[?(@.type=="Ready")].status}{"\n"}{end}'
echo

echo "✅ Multi-tier application deployment complete!"
echo "💡 Learning Tip: Always use health checks in production deployments"
echo "📈 Notice how rolling updates ensure zero downtime during deployments"
echo "🔧 Troubleshooting: If pods aren't ready, check logs with 'kubectl logs -l app=nginx'"
```

### Subtask 1.2: Create and Analyze ClusterIP Service

```bash
echo "=== SUBTASK 1.2: CLUSTERIP SERVICE DEEP DIVE ==="
echo "🌐 Creating and analyzing ClusterIP service architecture..."
echo

# Create comprehensive ClusterIP service
echo "📝 Creating production-grade ClusterIP service..."
nano services/nginx-clusterip-service.yaml
```

**File Content for nginx-clusterip-service.yaml:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip-service
  namespace: services-lab
  labels:
    app: nginx
    service-type: clusterip
    environment: lab
  annotations:
    description: "Internal cluster service for nginx application"
    owner: "platform-team"
spec:
  type: ClusterIP
  selector:
    app: nginx
    tier: frontend
  ports:
  - name: http
    port: 80
    targetPort: http
    protocol: TCP
  - name: metrics
    port: 9090
    targetPort: 80
    protocol: TCP
  sessionAffinity: None
  ipFamilyPolicy: SingleStack
  ipFamilies:
  - IPv4
```

```bash
# Deploy and analyze the ClusterIP service
kubectl apply -f services/nginx-clusterip-service.yaml
echo

# Comprehensive service analysis
echo "🔬 Analyzing service configuration..."
kubectl get service nginx-clusterip-service -o wide
echo
kubectl describe service nginx-clusterip-service
echo

# Examine service endpoints in detail
echo "🎯 Examining service endpoints..."
kubectl get endpoints nginx-clusterip-service -o wide
kubectl describe endpoints nginx-clusterip-service
echo

# Analyze service networking details
echo "📡 Service networking analysis..."
SERVICE_IP=$(kubectl get service nginx-clusterip-service -o jsonpath='{.spec.clusterIP}')
echo "Service ClusterIP: $SERVICE_IP"
echo "Service DNS Names:"
echo "  - nginx-clusterip-service"
echo "  - nginx-clusterip-service.services-lab"
echo "  - nginx-clusterip-service.services-lab.svc"
echo "  - nginx-clusterip-service.services-lab.svc.cluster.local"
echo

echo "✅ ClusterIP service analysis complete!"
echo "💡 Learning Tip: ClusterIP provides stable internal networking for microservices"
echo "🔧 Troubleshooting: Check endpoints if service isn't working - empty endpoints indicate selector issues"
```

### Subtask 1.3: Advanced ClusterIP Service Testing

```bash
echo "=== SUBTASK 1.3: COMPREHENSIVE SERVICE CONNECTIVITY TESTING ==="
echo "🧪 Performing advanced service connectivity tests..."
echo

# Create dedicated test pod with network debugging tools
echo "🛠️  Creating network debugging test pod..."
nano manifests/network-debug-pod.yaml
```

**File Content for network-debug-pod.yaml:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: network-debug
  namespace: services-lab
  labels:
    role: debug
    type: network-tools
spec:
  containers:
  - name: debug-tools
    image: nicolaka/netshoot:latest
    command: ["/bin/bash"]
    args: ["-c", "sleep 3600"]
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  restartPolicy: Always
```

```bash
# Deploy and use the debug pod
kubectl apply -f manifests/network-debug-pod.yaml
kubectl wait --for=condition=ready pod/network-debug --timeout=60s
echo

# Comprehensive connectivity testing
echo "🔍 Running comprehensive connectivity tests..."

# Test 1: Basic service connectivity
echo "Test 1: Basic HTTP connectivity to service..."
kubectl exec -it network-debug -- curl -s -w "\n%{http_code}\n" http://nginx-clusterip-service/
echo

# Test 2: DNS resolution testing
echo "Test 2: DNS resolution verification..."
kubectl exec -it network-debug -- nslookup nginx-clusterip-service
echo
kubectl exec -it network-debug -- dig nginx-clusterip-service.services-lab.svc.cluster.local
echo

# Test 3: Service endpoint testing
echo "Test 3: Direct endpoint connectivity..."
ENDPOINT_IPS=$(kubectl get endpoints nginx-clusterip-service -o jsonpath='{.subsets[0].addresses[*].ip}')
for ip in $ENDPOINT_IPS; do
    echo "Testing direct connection to pod IP: $ip"
    kubectl exec -it network-debug -- curl -s -w "Response time: %{time_total}s\n" http://$ip/ | head -1
done
echo

# Test 4: Load balancing verification
echo "Test 4: Load balancing behavior analysis..."
echo "Making 10 requests to observe load distribution..."
for i in {1..10}; do
    RESPONSE=$(kubectl exec -it network-debug -- curl -s http://nginx-clusterip-service/ | grep -o 'nginx/[0-9.]*' || echo "Request $i")
    echo "Request $i: $RESPONSE"
done
echo

# Test 5: Service metrics endpoint
echo "Test 5: Testing custom metrics endpoint..."
kubectl exec -it network-debug -- curl -s http://nginx-clusterip-service:9090/metrics
echo

# Test 6: Performance testing
echo "Test 6: Basic performance testing..."
kubectl exec -it network-debug -- ab -n 100 -c 10 http://nginx-clusterip-service/ 2>/dev/null | grep -E "(Requests per second|Time per request)"
echo

echo "✅ Comprehensive connectivity testing complete!"
echo "💡 Learning Tip: Use network debug pods for troubleshooting service connectivity"
echo "⚡ Performance Tip: ClusterIP services add minimal latency compared to direct pod access"
echo "🔧 Troubleshooting: If DNS fails, check CoreDNS pods in kube-system namespace"
```

## 🌐 Task 2: NodePort Services - External Access Gateway

### Subtask 2.1: Advanced NodePort Configuration

```bash
echo "=== TASK 2: NODEPORT SERVICES - EXTERNAL ACCESS GATEWAY ==="
echo "🚪 Configuring advanced NodePort services for external access..."
echo

# Create comprehensive NodePort service
echo "⚙️  Creating advanced NodePort service configuration..."
nano services/nginx-nodeport-service.yaml
```

**File Content for nginx-nodeport-service.yaml:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport-service
  namespace: services-lab
  labels:
    app: nginx
    service-type: nodeport
    access-level: external
  annotations:
    description: "NodePort service for external access to nginx"
    external-access: "true"
    port-info: "Accessible on all nodes at port 30080"
spec:
  type: NodePort
  selector:
    app: nginx
    tier: frontend
  ports:
  - name: http
    port: 80
    targetPort: http
    nodePort: 30080
    protocol: TCP
  - name: metrics
    port: 9090
    targetPort: 80
    nodePort: 30090
    protocol: TCP
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 3600
  externalTrafficPolicy: Cluster
```

```bash
# Deploy and analyze NodePort service
kubectl apply -f services/nginx-nodeport-service.yaml
echo

# Comprehensive NodePort analysis
echo "🔬 Analyzing NodePort service configuration..."
kubectl get service nginx-nodeport-service -o wide
echo
kubectl describe service nginx-nodeport-service
echo

# Display node information for external access
echo "🖥️  Node information for external access..."
kubectl get nodes -o wide
echo

# Show NodePort range configuration
echo "📊 Cluster NodePort configuration..."
NODE_PORT_RANGE=$(kubectl cluster-info dump 2>/dev/null | grep -o 'service-node-port-range=[0-9-]*' | head -1)
echo "NodePort Range: ${NODE_PORT_RANGE:-30000-32767 (default)}"
echo

echo "✅ NodePort service configuration complete!"
echo "💡 Learning Tip: NodePort services are ideal for development and testing environments"
echo "🔒 Security Note: NodePort exposes services on all nodes - ensure proper firewall rules"
echo "🔧 Troubleshooting: Check node firewalls if external access fails"
```

### Subtask 2.2: Comprehensive External Access Testing

```bash
echo "=== SUBTASK 2.2: COMPREHENSIVE EXTERNAL ACCESS TESTING ==="
echo "🌍 Testing external access patterns and load balancing..."
echo

# Get node internal IPs for testing
NODE_IPS=$(kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}')
echo "🖥️  Available nodes for testing: $NODE_IPS"
echo

# Test internal node access
echo "🔍 Testing access from within cluster..."
for node_ip in $NODE_IPS; do
    echo "Testing node: $node_ip"
    timeout 5 curl -s http://$node_ip:30080 | head -1 && echo "✅ Success" || echo "❌ Failed"
done
echo

# Test using localhost (if running on single node)
echo "🏠 Testing localhost access..."
curl -s -w "HTTP Status: %{http_code}\nResponse Time: %{time_total}s\n" http://localhost:30080 | head -3
echo

# Advanced load balancing test
echo "⚖️  Testing NodePort load balancing..."
echo "Making 20 requests to analyze traffic distribution..."
declare -A response_count
for i in {1..20}; do
    RESPONSE=$(curl -s http://localhost:30080 | grep -o 'nginx/[0-9.]*' 2>/dev/null || echo "unknown")
    response_count[$RESPONSE]=$((${response_count[$RESPONSE]} + 1))
    echo -n "."
done
echo
echo "Response distribution:"
for response in "${!response_count[@]}"; do
    echo "  $response: ${response_count[$response]} requests"
done
echo

# Test session affinity
echo "🔗 Testing session affinity (ClientIP)..."
echo "Multiple requests from same client should go to same backend..."
CLIENT_IP=$(curl -s http://httpbin.org/ip | jq -r '.origin' 2>/dev/null || echo "unknown")
echo "Client IP: $CLIENT_IP"
for i in {1..5}; do
    BACKEND=$(curl -s http://localhost:30080 | grep -o 'Server: [^<]*' 2>/dev/null || echo "Backend $i")
    echo "Request $i: $BACKEND"
done
echo

# Test metrics endpoint on NodePort
echo "📊 Testing metrics endpoint via NodePort..."
curl -s http://localhost:30090/metrics
echo

echo "✅ External access testing complete!"
echo "💡 Learning Tip: NodePort services provide consistent access across all cluster nodes"
echo "⚡ Performance Note: Session affinity can improve performance but may cause uneven load distribution"
echo "🔧 Troubleshooting: Use 'kubectl get events' to check for service-related issues"
```

### Subtask 2.3: NodePort Advanced Features and Auto-Assignment

```bash
echo "=== SUBTASK 2.3: NODEPORT ADVANCED FEATURES ==="
echo "🎛️  Exploring NodePort auto-assignment and advanced configurations..."
echo

# Create NodePort with auto-assigned port
echo "🎯 Creating NodePort service with auto-assigned port..."
nano services/nginx-nodeport-auto.yaml
```

**File Content for nginx-nodeport-auto.yaml:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport-auto
  namespace: services-lab
  labels:
    app: nginx
    service-type: nodeport-auto
  annotations:
    description: "NodePort service with auto-assigned port"
spec:
  type: NodePort
  selector:
    app: nginx
    tier: frontend
  ports:
  - name: http-auto
    port: 80
    targetPort: http
    protocol: TCP
  externalTrafficPolicy: Local  # Only route to local node pods
```

```bash
# Deploy and analyze auto-assigned NodePort
kubectl apply -f services/nginx-nodeport-auto.yaml
echo

# Show the auto-assigned port
echo "🎲 Auto-assigned NodePort details..."
AUTO_PORT=$(kubectl get service nginx-nodeport-auto -o jsonpath='{.spec.ports[0].nodePort}')
echo "Auto-assigned NodePort: $AUTO_PORT"
kubectl get service nginx-nodeport-auto -o wide
echo

# Test the auto-assigned port
echo "🧪 Testing auto-assigned NodePort..."
curl -s http://localhost:$AUTO_PORT | head -1
echo

# Compare traffic policies
echo "🔄 Comparing externalTrafficPolicy effects..."
echo
echo "Cluster Policy Service (nginx-nodeport-service):"
kubectl get service nginx-nodeport-service -o jsonpath='{.spec.externalTrafficPolicy}'
echo
echo "Local Policy Service (nginx-nodeport-auto):"  
kubectl get service nginx-nodeport-auto -o jsonpath='{.spec.externalTrafficPolicy}'
echo
echo

# Create comprehensive NodePort service inventory
echo "📋 Complete NodePort service inventory..."
kubectl get services -l service-type=nodeport -o custom-columns=NAME:.metadata.name,TYPE:.spec.type,CLUSTER-IP:.spec.clusterIP,EXTERNAL-IP:.status.loadBalancer.ingress[0].ip,NODEPORT:.spec.ports[0].nodePort,AGE:.metadata.creationTimestamp --sort-by=.metadata.creationTimestamp
echo

echo "✅ NodePort advanced features exploration complete!"
echo "💡 Learning Tip: Auto-assignment prevents port conflicts in multi-service environments"
echo "🎯 Traffic Policy: 'Local' preserves source IP but may cause uneven load distribution"
echo "🔧 Troubleshooting: Check service endpoints for traffic policy effects"
```

## ☁️ Task 3: LoadBalancer Services - Production External Access

### Subtask 3.1: LoadBalancer Service Configuration

```bash
echo "=== TASK 3: LOADBALANCER SERVICES - PRODUCTION EXTERNAL ACCESS ==="
echo "🏭 Configuring production-grade LoadBalancer services..."
echo

# Create comprehensive LoadBalancer service
echo "⚙️  Creating enterprise LoadBalancer service..."
nano services/nginx-loadbalancer-service.yaml
```

**File Content for nginx-loadbalancer-service.yaml:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer-service
  namespace: services-lab
  labels:
    app: nginx
    service-type: loadbalancer
    environment: production
  annotations:
    description: "Production LoadBalancer service for nginx"
    # AWS-specific annotations (adjust for your cloud provider)
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "http"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-protocol: "http"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-path: "/health"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval-seconds: "10"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-timeout-seconds: "5"
    service.beta.kubernetes.io/aws-load-balancer-healthy-threshold-count: "2"
    service.beta.kubernetes.io/aws-load-balancer-unhealthy-threshold-count: "3"
    # GCP-specific annotations (alternative)
    # cloud.google.com/load-balancer-type: "External"
    # cloud.google.com/neg: '{"ingress": true}'
    # Azure-specific annotations (alternative)
    # service.beta.kubernetes.io/azure-load-balancer-internal: "false"
spec:
  type: LoadBalancer
  selector:
    app: nginx
    tier: frontend
  ports:
  - name: http
    port: 80
    targetPort: http
    protocol: TCP
  - name: https
    port: 443
    targetPort: http
    protocol: TCP
  sessionAffinity: None
  externalTrafficPolicy: Local
  loadBalancerSourceRanges:
  - "0.0.0.0/0"  # Allow all IPs (adjust for production security)
  ipFamilyPolicy: SingleStack
  ipFamilies:
  - IPv4
```

```bash
# Deploy LoadBalancer service
kubectl apply -f services/nginx-loadbalancer-service.yaml
echo

# Monitor LoadBalancer provisioning
echo "🔍 Monitoring LoadBalancer provisioning..."
echo "Note: This may take several minutes in cloud environments..."
kubectl get service nginx-loadbalancer-service --watch --timeout=300s &
WATCH_PID=$!

# Wait a moment then kill the watch process
sleep 10
kill $WATCH_PID 2>/dev/null || true
echo

# Check LoadBalancer status
echo "📊 LoadBalancer service status..."
kubectl get service nginx-loadbalancer-service -o wide
echo
kubectl describe service nginx-loadbalancer-service
echo

# Check for external IP assignment
EXTERNAL_IP=$(kubectl get service nginx-loadbalancer-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}' 2>/dev/null)
EXTERNAL_HOSTNAME=$(kubectl get service nginx-loadbalancer-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}' 2>/dev/null)

if [ -n "$EXTERNAL_IP" ]; then
    echo "✅ External IP assigned: $EXTERNAL_IP"
elif [ -n "$EXTERNAL_HOSTNAME" ]; then
    echo "✅ External hostname assigned: $EXTERNAL_HOSTNAME"
else
    echo "⏳ LoadBalancer still provisioning or not supported in this environment"
    echo "💡 Note: LoadBalancer services require cloud provider support"
fi
echo

echo "✅ LoadBalancer service configuration complete!"
echo "💡 Learning Tip: LoadBalancer services are the preferred method for production external access"
echo "☁️ Cloud Note: Different cloud providers have specific annotation requirements"
echo "🔧 Troubleshooting: Check cloud provider quotas if LoadBalancer remains pending"
```

### Subtask 3.2: MetalLB Installation for Local LoadBalancer Simulation

```bash
echo "=== SUBTASK 3.2: METALLB INSTALLATION FOR LOCAL SIMULATION ==="
echo "🛠️  Installing MetalLB for LoadBalancer simulation in local environments..."
echo

# Check if we're in a cloud environment first
echo "🔍 Detecting environment type..."
kubectl get nodes -o jsonpath='{.items[0].spec.providerID}' | grep -q "aws\|gce\|azure" && CLOUD_ENV=true || CLOUD_ENV=false

if [ "$CLOUD_ENV" = "false" ]; then
    echo "📦 Installing MetalLB for local LoadBalancer simulation..."
    
    # Install MetalLB
    kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.7/config/manifests/metallb-native.yaml
    
    # Wait for MetalLB to be ready
    echo "⏳ Waiting for MetalLB pods to be ready..."
    kubectl wait --namespace metallb-system \
        --for=condition=ready pod \
        --selector=app=metallb \
        --timeout=90s
    
    # Get the node network range for IP pool configuration
    NODE_NETWORK=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}' | cut -d. -f1-3)
    
    # Create MetalLB configuration
    echo "⚙️  Configuring MetalLB IP address pool..."
    nano configs/metallb-config.yaml
fi
```

**File Content for metallb-config.yaml:**
```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: services-lab-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.240-192.168.1.250  # Adjust based on your network
  autoAssign: true
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: services-lab-l2
  namespace: metallb-system
spec:
  ipAddressPools:
  - services-lab-pool
  interfaces:
  - eth0  # Adjust based on your network interface
```

```bash
if [ "$CLOUD_ENV" = "false" ]; then
    # Apply MetalLB configuration
    kubectl apply -f configs/metallb-config.yaml
    
    # Wait for IP assignment
    echo "⏳ Waiting for MetalLB to assign external IP..."
    sleep 30
    
    # Check LoadBalancer status with MetalLB
    echo "🔍 Checking LoadBalancer status with MetalLB..."
    kubectl get service nginx-loadbalancer-service -o wide
    
    METALLB_IP=$(kubectl get service nginx-loadbalancer-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}' 2>/dev/null)
    if [ -n "$METALLB_IP" ]; then
        echo "✅ MetalLB assigned external IP: $METALLB_IP"
        
        # Test MetalLB LoadBalancer
        echo "🧪 Testing MetalLB LoadBalancer..."
        curl -s -w "HTTP Status: %{http_code}\nResponse Time: %{time_total}s\n" http://$METALLB_IP/ | head -3
    else
        echo "⏳ MetalLB still assigning IP address..."
    fi
else
    echo "☁️ Cloud environment detected - using native LoadBalancer support"
fi

echo

echo "✅ LoadBalancer setup complete!"
echo "💡 Learning Tip: MetalLB enables LoadBalancer services in bare-metal environments"
echo "🌐 Production Note: Cloud LoadBalancers provide additional features like SSL termination"
echo "🔧 Troubleshooting: Check MetalLB logs if IP assignment fails"
```

### Subtask 3.3: LoadBalancer Performance Testing and Monitoring

```bash
echo "=== SUBTASK 3.3: LOADBALANCER PERFORMANCE TESTING ==="
echo "⚡ Performing comprehensive LoadBalancer performance analysis..."
echo

# Get LoadBalancer IP (cloud or MetalLB)
LB_IP=$(kubectl get service nginx-loadbalancer-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}' 2>/dev/null)
LB_HOSTNAME=$(kubectl get service nginx-loadbalancer-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}' 2>/dev/null)

if [ -n "$LB_IP" ]; then
    LB_ENDPOINT=$LB_IP
elif [ -n "$LB_HOSTNAME" ]; then
    LB_ENDPOINT=$LB_HOSTNAME
else
    echo "⚠️  LoadBalancer external endpoint not available - using NodePort for testing"
    LB_ENDPOINT="localhost:30080"
fi

echo "🎯 Testing LoadBalancer endpoint: $LB_ENDPOINT"
echo

# Basic connectivity test
echo "🔍 Basic LoadBalancer connectivity test..."
curl -s -w "HTTP Status: %{http_code}\nTotal Time: %{time_total}s\nName Resolution: %{time_namelookup}s\nConnect Time: %{time_connect}s\n" http://$LB_ENDPOINT/ | head -5
echo

# Load distribution testing
echo "⚖️  Testing load distribution across backend pods..."
echo "Making 50 requests to analyze traffic distribution..."
declare -A pod_responses
for i in {1..50}; do
    POD_ID=$(curl -s http://$LB_ENDPOINT/ | grep -o 'nginx-app-[a-z0-9-]*' 2>/dev/null || echo "pod-$((RANDOM % 3))")
    pod_responses[$POD_ID]=$((${pod_responses[$POD_ID]} + 1))
    echo -n "."
    sleep 0.1
done
echo
echo "Load distribution results:"
for pod in "${!pod_responses[@]}"; do
    echo "  $pod: ${pod_responses[$pod]} requests ($(( ${pod_responses[$pod]} * 100 / 50 ))%)"
done
echo

# Performance benchmarking
echo "📊 Performance benchmarking with Apache Bench..."
if command -v ab >/dev/null 2>&1; then
    ab -n 1000 -c 10 -g loadbalancer_results.dat http://$LB_ENDPOINT/ 2>/dev/null | grep -E "(Requests per second|Time per request|Transfer rate)"
else
    echo "Installing Apache Bench..."
    apt-get update && apt-get install -y apache2-utils >/dev/null 2>&1
    ab -n 1000 -c 10 http://$LB_ENDPOINT/ 2>/dev/null | grep -E "(Requests per second|Time per request|Transfer rate)"
fi
echo

# Health check testing
echo "🏥 Testing LoadBalancer health check endpoint..."
curl -s -w "Health Check Status: %{http_code}\n" http://$LB_ENDPOINT/health
echo

# SSL/HTTPS testing (if port 443 is available)
echo "🔒 Testing HTTPS capability (if configured)..."
curl -k -s -w "HTTPS Status: %{http_code}\n" https://$LB_ENDPOINT/ 2>/dev/null || echo "HTTPS not configured or not available"
echo

echo "✅ LoadBalancer performance testing complete!"
echo "💡 Learning Tip: LoadBalancers provide automatic failover and health checking"
echo "📈 Performance Note: External LoadBalancers typically add 1-5ms latency"
echo "🔧 Troubleshooting: Monitor cloud provider LoadBalancer metrics for production issues"
```

## 🔍 Task 4: Advanced Service Discovery and DNS Deep Dive

### Subtask 4.1: Comprehensive DNS Resolution Testing

```bash
echo "=== TASK 4: ADVANCED SERVICE DISCOVERY AND DNS DEEP DIVE ==="
echo "🌐 Exploring Kubernetes DNS architecture and service discovery patterns..."
echo

# Create enhanced DNS testing pod
echo "🛠️  Creating advanced DNS testing environment..."
nano manifests/dns-test-pod.yaml
```

**File Content for dns-test-pod.yaml:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dns-test-advanced
  namespace: services-lab
  labels:
    role: dns-tester
    type: debugging
spec:
  containers:
  - name: dns-tools
    image: tutum/dnsutils:latest
    command: ["/bin/bash"]
    args: ["-c", "sleep 3600"]
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
    env:
    - name: NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
  restartPolicy: Always
  dnsPolicy: ClusterFirst
  dnsConfig:
    options:
    - name: ndots
      value: "2"
    - name: edns0
```

```bash
# Deploy DNS testing pod
kubectl apply -f manifests/dns-test-pod.yaml
kubectl wait --for=condition=ready pod/dns-test-advanced --timeout=60s
echo

# Comprehensive DNS resolution testing
echo "🔬 Performing comprehensive DNS resolution analysis..."

# Test 1: Basic service name resolution
echo "Test 1: Basic service name resolution..."
kubectl exec -it dns-test-advanced -- nslookup nginx-clusterip-service
echo

# Test 2: Fully Qualified Domain Name (FQDN) resolution
echo "Test 2: FQDN resolution testing..."
kubectl exec -it dns-test-advanced -- nslookup nginx-clusterip-service.services-lab.svc.cluster.local
echo

# Test 3: Cross-namespace service discovery
echo "Test 3: Cross-namespace service discovery..."
kubectl exec -it dns-test-advanced -- nslookup kubernetes.default.svc.cluster.local
echo

# Test 4: DNS performance testing
echo "Test 4: DNS resolution performance analysis..."
kubectl exec -it dns-test-advanced -- bash -c 'for i in {1..5}; do echo "Lookup $i:"; time nslookup nginx-clusterip-service >/dev/null; done'
echo

# Test 5: SRV record discovery
echo "Test 5: SRV record discovery for service ports..."
kubectl exec -it dns-test-advanced -- nslookup -type=SRV _http._tcp.nginx-clusterip-service.services-lab.svc.cluster.local
echo

# Test 6: Headless service DNS (create a headless service first)
echo "Creating headless service for DNS testing..."
nano services/nginx-headless-service.yaml
```

**File Content for nginx-headless-service.yaml:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless-service
  namespace: services-lab
  labels:
    app: nginx
    service-type: headless
spec:
  type: ClusterIP
  clusterIP: None  # This makes it headless
  selector:
    app: nginx
    tier: frontend
  ports:
  - name: http
    port: 80
    targetPort: http
    protocol: TCP
```

```bash
kubectl apply -f services/nginx-headless-service.yaml
echo

# Test headless service DNS
echo "Test 6: Headless service DNS resolution (returns all pod IPs)..."
kubectl exec -it dns-test-advanced -- nslookup nginx-headless-service
echo

# Test 7: DNS configuration analysis
echo "Test 7: DNS configuration analysis..."
kubectl exec -it dns-test-advanced -- cat /etc/resolv.conf
echo

# Test 8: CoreDNS testing
echo "Test 8: CoreDNS functionality verification..."
kubectl exec -it dns-test-advanced -- dig @kube-dns.kube-system.svc.cluster.local nginx-clusterip-service.services-lab.svc.cluster.local
echo

echo "✅ DNS resolution testing complete!"
echo "💡 Learning Tip: Kubernetes DNS enables service discovery across the entire cluster"
echo "🎯 Performance Tip: Use short names within the same namespace for faster resolution"
echo "🔧 Troubleshooting: Check CoreDNS pods if DNS resolution fails"
```

### Subtask 4.2: Service Endpoints Deep Dive

```bash
echo "=== SUBTASK 4.2: SERVICE ENDPOINTS DEEP DIVE ==="
echo "🎯 Analyzing service endpoint management and lifecycle..."
echo

# Comprehensive endpoint analysis
echo "🔍 Analyzing current service endpoints..."
kubectl get endpoints -o wide
echo

# Detailed endpoint inspection for each service
echo "📊 Detailed endpoint analysis..."
for service in nginx-clusterip-service nginx-nodeport-service nginx-loadbalancer-service nginx-headless-service; do
    if kubectl get service $service >/dev/null 2>&1; then
        echo "Service: $service"
        kubectl get endpoints $service -o yaml | grep -E "(name:|addresses:|ports:|nodeName:|targetRef:)" || echo "No endpoints found"
        echo
    fi
done

# Test endpoint behavior during pod scaling
echo "⚖️  Testing endpoint behavior during scaling operations..."
echo "Current pod count:"
kubectl get pods -l app=nginx --no-headers | wc -l

# Scale up
echo "Scaling up to 5 replicas..."
kubectl scale deployment nginx-app --replicas=5
kubectl rollout status deployment/nginx-app --timeout=120s

echo "Endpoints after scaling up:"
kubectl get endpoints nginx-clusterip-service -o jsonpath='{.subsets[0].addresses[*].ip}' | tr ' ' '\n' | wc -l
kubectl get endpoints nginx-clusterip-service -o wide
echo

# Scale down
echo "Scaling down to 2 replicas..."
kubectl scale deployment nginx-app --replicas=2
kubectl rollout status deployment/nginx-app --timeout=120s

echo "Endpoints after scaling down:"
kubectl get endpoints nginx-clusterip-service -o jsonpath='{.subsets[0].addresses[*].ip}' | tr ' ' '\n' | wc -l
kubectl get endpoints nginx-clusterip-service -o wide
echo

# Restore original scale
echo "Restoring original scale (3 replicas)..."
kubectl scale deployment nginx-app --replicas=3
kubectl rollout status deployment/nginx-app --timeout=120s
echo

# Test endpoint behavior with unhealthy pods
echo "🏥 Testing endpoint behavior with unhealthy pods..."
echo "Simulating pod failure..."
POD_TO_KILL=$(kubectl get pods -l app=nginx -o jsonpath='{.items[0].metadata.name}')
kubectl delete pod $POD_TO_KILL &

echo "Watching endpoint changes during pod replacement..."
sleep 5
kubectl get endpoints nginx-clusterip-service -o wide
echo

echo "✅ Service endpoints analysis complete!"
echo "💡 Learning Tip: Endpoints are automatically managed by the Endpoints Controller"
echo "⚡ Scaling Tip: Endpoints update immediately as pods become ready/unready"
echo "🔧 Troubleshooting: Check endpoint subsets if services aren't reaching pods"
```

### Subtask 4.3: Advanced Service Discovery Patterns

```bash
echo "=== SUBTASK 4.3: ADVANCED SERVICE DISCOVERY PATTERNS ==="
echo "🎨 Exploring advanced service discovery and communication patterns..."
echo

# Create multi-tier application for service discovery testing
echo "🏗️  Creating multi-tier application for service discovery testing..."
nano manifests/backend-app.yaml
```

**File Content for backend-app.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-app
  namespace: services-lab
  labels:
    app: backend
    tier: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
      tier: backend
  template:
    metadata:
      labels:
        app: backend
        tier: backend
    spec:
      containers:
      - name: backend
        image: httpd:2.4-alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "32Mi"
            cpu: "50m"
          limits:
            memory: "64Mi"
            cpu: "100m"
        env:
        - name: SERVER_TYPE
          value: "backend"
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: services-lab
  labels:
    app: backend
    tier: backend
spec:
  type: ClusterIP
  selector:
    app: backend
    tier: backend
  ports:
  - name: http
    port: 8080
    targetPort: 80
    protocol: TCP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
  namespace: services-lab
  labels:
    app: api-gateway
    tier: api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: api-gateway
      tier: api
  template:
    metadata:
      labels:
        app: api-gateway
        tier: api
    spec:
      containers:
      - name: api-gateway
        image: nginx:alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "32Mi"
            cpu: "50m"
          limits:
            memory: "64Mi"
            cpu: "100m"
        volumeMounts:
        - name: gateway-config
          mountPath: /etc/nginx/conf.d/default.conf
          subPath: default.conf
      volumes:
      - name: gateway-config
        configMap:
          name: api-gateway-config
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-gateway-config
  namespace: services-lab
data:
  default.conf: |
    upstream backend {
        server backend-service.services-lab.svc.cluster.local:8080;
    }
    
    upstream frontend {
        server nginx-clusterip-service.services-lab.svc.cluster.local:80;
    }
    
    server {
        listen 80;
        
        location /api/ {
            proxy_pass http://backend/;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        
        location / {
            proxy_pass http://frontend/;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        
        location /health {
            return 200 "API Gateway Healthy\n";
            add_header Content-Type text/plain;
        }
    }
---
apiVersion: v1
kind: Service
metadata:
  name: api-gateway-service
  namespace: services-lab
  labels:
    app: api-gateway
    tier: api
spec:
  type: NodePort
  selector:
    app: api-gateway
    tier: api
  ports:
  - name: http
    port: 80
    targetPort: 80
    nodePort: 30100
    protocol: TCP
```

```bash
# Deploy multi-tier application
kubectl apply -f manifests/backend-app.yaml
kubectl wait --for=condition=available deployment/backend-app --timeout=120s
kubectl wait --for=condition=available deployment/api-gateway --timeout=120s
echo

# Test service discovery patterns
echo "🔍 Testing service discovery patterns..."

# Test internal service-to-service communication
echo "Test 1: Internal service-to-service communication..."
kubectl exec -it dns-test-advanced -- curl -s http://backend-service.services-lab.svc.cluster.local:8080
echo

# Test API Gateway service discovery
echo "Test 2: API Gateway service discovery and routing..."
curl -s http://localhost:30100/health
echo
curl -s http://localhost:30100/ | head -1
echo

# Test environment-based service discovery
echo "Test 3: Environment variable-based service discovery..."
kubectl exec -it dns-test-advanced -- env | grep -E "(BACKEND_SERVICE|NGINX_CLUSTERIP)" || echo "Environment variables not automatically created (depends on service creation order)"
echo

# Create service discovery test script
echo "Test 4: Programmatic service discovery..."
nano troubleshooting/service-discovery-test.sh
```

**File Content for service-discovery-test.sh:**
```bash
#!/bin/bash

echo "=== Service Discovery Test Suite ==="

# Function to test service connectivity
test_service() {
    local service_name=$1
    local namespace=${2:-services-lab}
    local port=${3:-80}
    
    echo "Testing $service_name in namespace $namespace on port $port..."
    
    # Test short name
    if curl -s --connect-timeout 5 http://$service_name:$port/ >/dev/null; then
        echo "  ✅ Short name resolution: SUCCESS"
    else
        echo "  ❌ Short name resolution: FAILED"
    fi
    
    # Test namespace-qualified name
    if curl -s --connect-timeout 5 http://$service_name.$namespace:$port/ >/dev/null; then
        echo "  ✅ Namespace-qualified resolution: SUCCESS"
    else
        echo "  ❌ Namespace-qualified resolution: FAILED"
    fi
    
    # Test FQDN
    if curl -s --connect-timeout 5 http://$service_name.$namespace.svc.cluster.local:$port/ >/dev/null; then
        echo "  ✅ FQDN resolution: SUCCESS"
    else
        echo "  ❌ FQDN resolution: FAILED"
    fi
    
    echo
}

# Test all services
test_service "nginx-clusterip-service" "services-lab" "80"
test_service "backend-service" "services-lab" "8080"
test_service "api-gateway-service" "services-lab" "80"
test_service "kubernetes" "default" "443"

echo "=== Service Discovery Test Complete ==="
```

```bash
# Make script executable and run it
chmod +x troubleshooting/service-discovery-test.sh
kubectl exec -i dns-test-advanced -- bash < troubleshooting/service-discovery-test.sh
echo

# Test cross-namespace service discovery
echo "Test 5: Cross-namespace service discovery..."
kubectl create namespace test-namespace
kubectl run cross-ns-test --image=busybox --rm -it --restart=Never --namespace=test-namespace -- nslookup nginx-clusterip-service.services-lab.svc.cluster.local
echo

echo "✅ Advanced service discovery patterns testing complete!"
echo "💡 Learning Tip: Use FQDN for cross-namespace service communication"
echo "🎯 Best Practice: Implement health checks for reliable service discovery"
echo "🔧 Troubleshooting: Use dig/nslookup to debug DNS resolution issues"
```

## 🛠️ Task 5: Advanced Troubleshooting and Service Debugging

### Subtask 5.1: Comprehensive Service Debugging Toolkit

```bash
echo "=== TASK 5: ADVANCED TROUBLESHOOTING AND SERVICE DEBUGGING ==="
echo "🔧 Building comprehensive service debugging and troubleshooting toolkit..."
echo

# Create troubleshooting namespace and tools
echo "📦 Setting up debugging environment..."
nano troubleshooting/debug-toolkit.yaml
```

**File Content for debug-toolkit.yaml:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: debug-scripts
  namespace: services-lab
data:
  service-health-check.sh: |
    #!/bin/bash
    SERVICE_NAME=$1
    NAMESPACE=${2:-services-lab}
    
    echo "=== Service Health Check: $SERVICE_NAME ==="
    
    # Check service exists
    if ! kubectl get service $SERVICE_NAME -n $NAMESPACE >/dev/null 2>&1; then
        echo "❌ Service $SERVICE_NAME not found in namespace $NAMESPACE"
        exit 1
    fi
    
    # Service details
    echo "📋 Service Details:"
    kubectl get service $SERVICE_NAME -n $NAMESPACE -o wide
    
    # Endpoints check
    echo "🎯 Endpoints Check:"
    ENDPOINT_COUNT=$(kubectl get endpoints $SERVICE_NAME -n $NAMESPACE -o jsonpath='{.subsets[0].addresses[*].ip}' | wc -w)
    echo "Active endpoints: $ENDPOINT_COUNT"
    kubectl get endpoints $SERVICE_NAME -n $NAMESPACE -o wide
    
    # Pod status
    echo "🔍 Backend Pod Status:"
    SELECTOR=$(kubectl get service $SERVICE_NAME -n $NAMESPACE -o jsonpath='{.spec.selector}' | jq -r 'to_entries[] | "\(.key)=\(.value)"' | tr '\n' ',' | sed 's/,$//')
    kubectl get pods -n $NAMESPACE -l "$SELECTOR" -o wide
    
    echo "✅ Health check complete for $SERVICE_NAME"
  
  network-diagnostics.sh: |
    #!/bin/bash
    echo "=== Network Diagnostics ==="
    
    # DNS testing
    echo "🌐 DNS Configuration:"
    cat /etc/resolv.conf
    echo
    
    # Test cluster DNS
    echo "🔍 Cluster DNS Test:"
    nslookup kubernetes.default.svc.cluster.local
    echo
    
    # Network connectivity
    echo "📡 Network Connectivity:"
    ip route | head -5
    echo
    
    # Active connections
    echo "🔗 Active Connections:"
    netstat -tuln | head -10
    
  service-performance-test.sh: |
    #!/bin/bash
    SERVICE_URL=$1
    REQUESTS=${2:-100}
    CONCURRENCY=${3:-10}
    
    echo "=== Performance Test: $SERVICE_URL ==="
    echo "Requests: $REQUESTS, Concurrency: $CONCURRENCY"
    
    # Basic connectivity test
    echo "🔍 Basic connectivity test..."
    if curl -s --connect-timeout 5 $SERVICE_URL >/dev/null; then
        echo "✅ Service is reachable"
    else
        echo "❌ Service is not reachable"
        exit 1
    fi
    
    # Response time test
    echo "⏱️  Response time analysis..."
    for i in {1..5}; do
        TIME=$(curl -s -w "%{time_total}" -o /dev/null $SERVICE_URL)
        echo "Request $i: ${TIME}s"
    done
    
    echo "✅ Performance test complete"

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: debug-toolkit
  namespace: services-lab
  labels:
    app: debug-toolkit
spec:
  replicas: 1
  selector:
    matchLabels:
      app: debug-toolkit
  template:
    metadata:
      labels:
        app: debug-toolkit
    spec:
      containers:
      - name: debug-tools
        image: nicolaka/netshoot:latest
        command: ["/bin/bash", "-c", "sleep 3600"]
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
        volumeMounts:
        - name: debug-scripts
          mountPath: /scripts
        env:
        - name: NAMESPACE
          value: services-lab
      volumes:
      - name: debug-scripts
        configMap:
          name: debug-scripts
          defaultMode: 0755
      serviceAccountName: debug-service-account

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: debug-service-account
  namespace: services-lab

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: debug-cluster-role
rules:
- apiGroups: [""]
  resources: ["services", "endpoints", "pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: debug-cluster-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: debug-cluster-role
subjects:
- kind: ServiceAccount
  name: debug-service-account
  namespace: services-lab
```

```bash
# Deploy debugging toolkit
kubectl apply -f troubleshooting/debug-toolkit.yaml
kubectl wait --for=condition=available deployment/debug-toolkit --timeout=120s
echo

# Test debugging toolkit
echo "🧪 Testing debugging toolkit functionality..."
DEBUG_POD=$(kubectl get pods -l app=debug-toolkit -o jsonpath='{.items[0].metadata.name}')

echo "Debug pod: $DEBUG_POD"
echo

# Test service health check script
echo "Test 1: Service health check..."
kubectl exec -it $DEBUG_POD -- /scripts/service-health-check.sh nginx-clusterip-service services-lab
echo

# Test network diagnostics
echo "Test 2: Network diagnostics..."
kubectl exec -it $DEBUG_POD -- /scripts/network-diagnostics.sh
echo

echo "✅ Debugging toolkit setup complete!"
echo "💡 Learning Tip: Always have debugging tools ready for production troubleshooting"
echo "🔧 Usage: kubectl exec -it $DEBUG_POD -- /scripts/[script-name]"
```

### Subtask 5.2: Common Service Issues Simulation and Resolution

```bash
echo "=== SUBTASK 5.2: COMMON SERVICE ISSUES SIMULATION ==="
echo "🎭 Simulating and resolving common service issues..."
echo

# Issue 1: Service with wrong selector
echo "Issue 1: Service with incorrect selector..."
nano troubleshooting/broken-service-1.yaml
```

**File Content for broken-service-1.yaml:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: broken-service-selector
  namespace: services-lab
  labels:
    issue-type: wrong-selector
spec:
  type: ClusterIP
  selector:
    app: non-existent-app  # Wrong selector
    tier: wrong-tier
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
```

```bash
kubectl apply -f troubleshooting/broken-service-1.yaml

echo "🔍 Diagnosing wrong selector issue..."
echo "Service details:"
kubectl get service broken-service-selector -o wide
echo
echo "Endpoints (should be empty):"
kubectl get endpoints broken-service-selector
echo

echo "🔧 Fixing wrong selector issue..."
kubectl patch service broken-service-selector -p '{"spec":{"selector":{"app":"nginx","tier":"frontend"}}}'
echo "After fix:"
kubectl get endpoints broken-service-selector -o wide
echo

# Issue 2: Port mismatch
echo "Issue 2: Port mismatch configuration..."
nano troubleshooting/broken-service-2.yaml
```

**File Content for broken-service-2.yaml:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: broken-service-port
  namespace: services-lab
  labels:
    issue-type: wrong-port
spec:
  type: ClusterIP
  selector:
    app: nginx
    tier: frontend
  ports:
  - name: http
    port: 80
    targetPort: 8080  # Wrong target port
    protocol: TCP
```

```bash
kubectl apply -f troubleshooting/broken-service-2.yaml

echo "🔍 Diagnosing port mismatch issue..."
kubectl exec -it $DEBUG_POD -- curl -s --connect-timeout 5 http://broken-service-port/ || echo "❌ Connection failed due to port mismatch"

echo "🔧 Fixing port mismatch..."
kubectl patch service broken-service-port -p '{"spec":{"ports":[{"name":"http","port":80,"targetPort":"http","protocol":"TCP"}]}}'
echo "Testing after fix:"
kubectl exec -it $DEBUG_POD -- curl -s http://broken-service-port/ | head -1
echo

# Issue 3: Network policy blocking traffic (simulate)
echo "Issue 3: Simulating network connectivity issues..."
echo "Creating test pod that can't reach service..."
nano troubleshooting/isolated-pod.yaml
```

**File Content for isolated-pod.yaml:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: isolated-test-pod
  namespace: services-lab
  labels:
    network-access: restricted
spec:
  containers:
  - name: test-client
    image: busybox
    command: ["sleep", "3600"]
    resources:
      requests:
        memory: "16Mi"
        cpu: "25m"
      limits:
        memory: "32Mi"
        cpu: "50m"
```

```bash
kubectl apply -f troubleshooting/isolated-pod.yaml
kubectl wait --for=condition=ready pod/isolated-test-pod --timeout=60s

echo "🔍 Testing connectivity from isolated pod..."
kubectl exec -it isolated-test-pod -- wget -qO- --timeout=5 http://nginx-clusterip-service/ | head -1 && echo "✅ Connection successful" || echo "❌ Connection failed"
echo

# Issue 4: Service without endpoints
echo "Issue 4: Service with no backing pods..."
nano troubleshooting/broken-service-3.yaml
```

**File Content for broken-service-3.yaml:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: orphaned-service
  namespace: services-lab
  labels:
    issue-type: no-pods
spec:
  type: ClusterIP
  selector:
    app: non-existent-deployment
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
```

```bash
kubectl apply -f troubleshooting/broken-service-3.yaml

echo "🔍 Diagnosing service with no endpoints..."
kubectl get endpoints orphaned-service
echo "This service will have no endpoints because no pods match its selector"
echo

# Create comprehensive troubleshooting guide
echo "📋 Creating troubleshooting decision tree..."
nano troubleshooting/service-troubleshooting-guide.md
```

**File Content for service-troubleshooting-guide.md:**
```markdown
# Kubernetes Service Troubleshooting Guide

## Quick Diagnosis Checklist

### 1. Service Not Responding
- [ ] Check if service exists: `kubectl get svc SERVICE_NAME`
- [ ] Verify service has endpoints: `kubectl get endpoints SERVICE_NAME`
- [ ] Confirm pods are running: `kubectl get pods -l SELECTOR`
- [ ] Test from within cluster first

### 2. Empty Endpoints
- [ ] Verify pod labels match service selector
- [ ] Check if pods are in Ready state
- [ ] Confirm target ports match container ports
- [ ] Review pod readiness probes

### 3. DNS Resolution Issues
- [ ] Test with FQDN: `service.namespace.svc.cluster.local`
- [ ] Check CoreDNS pods: `kubectl get pods -n kube-system -l k8s-app=kube-dns`
- [ ] Verify DNS policy in pod spec
- [ ] Test from different namespace

### 4. External Access Problems (NodePort/LoadBalancer)
- [ ] Verify service type is correct
- [ ] Check node firewall rules
- [ ] Confirm LoadBalancer provider support
- [ ] Test internal connectivity first

### 5. Performance Issues
- [ ] Monitor endpoint distribution
- [ ] Check resource limits on pods
- [ ] Review service session affinity settings
- [ ] Analyze network latency

## Common Commands for Troubleshooting

```bash
# Service inspection
kubectl get svc SERVICE_NAME -o yaml
kubectl describe svc SERVICE_NAME

# Endpoint verification
kubectl get endpoints SERVICE_NAME
kubectl describe endpoints SERVICE_NAME

# Pod connectivity testing
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -qO- SERVICE_NAME

# DNS testing
kubectl run dns-test --image=busybox --rm -it --restart=Never -- nslookup SERVICE_NAME

# Network debugging
kubectl exec -it POD_NAME -- netstat -tuln
kubectl exec -it POD_NAME -- ss -tuln

# Service logs and events
kubectl get events --sort-by='.lastTimestamp'
kubectl logs -l app=APP_LABEL
```

## Troubleshooting Decision Tree

1. **Service exists but not responding**
   - Check endpoints → Fix selector/pod labels
   - Verify port configuration → Correct targetPort
   - Test DNS resolution → Fix CoreDNS issues

2. **External access failing**
   - NodePort: Check node accessibility and firewall
   - LoadBalancer: Verify cloud provider support
   - Try internal access first

3. **Intermittent failures**
   - Check pod health and readiness probes
   - Review resource limits and scaling
   - Analyze load balancing behavior
```

```bash
echo

echo "✅ Service issues simulation and troubleshooting complete!"
echo "💡 Learning Tip: Always check endpoints first when services aren't working"
echo "🎯 Best Practice: Use systematic troubleshooting approach from internal to external"
echo "🔧 Quick Debug: kubectl get svc,endpoints,pods -l app=SERVICE_LABEL"
```

### Subtask 5.3: Performance Monitoring and Optimization

```bash
echo "=== SUBTASK 5.3: PERFORMANCE MONITORING AND OPTIMIZATION ==="
echo "📊 Implementing service performance monitoring and optimization strategies..."
echo

# Create service performance monitoring setup
echo "⚙️  Setting up service performance monitoring..."
nano troubleshooting/performance-monitor.yaml
```

**File Content for performance-monitor.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: performance-monitor
  namespace: services-lab
  labels:
    app: performance-monitor
spec:
  replicas: 1
  selector:
    matchLabels:
      app: performance-monitor
  template:
    metadata:
      labels:
        app: performance-monitor
    spec:
      containers:
      - name: monitor
        image: prom/prometheus:latest
        ports:
        - containerPort: 9090
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "400m"
        volumeMounts:
        - name: prometheus-config
          mountPath: /etc/prometheus/prometheus.yml
          subPath: prometheus.yml
        - name: prometheus-storage
          mountPath: /prometheus
      volumes:
      - name: prometheus-config
        configMap:
          name: prometheus-config
      - name: prometheus-storage
        emptyDir: {}

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: services-lab
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s

    scrape_configs:
      - job_name: 'kubernetes-services'
        kubernetes_sd_configs:
          - role: service
            namespaces:
              names:
              - services-lab
        relabel_configs:
          - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
            action: keep
            regex: true
          - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
            action: replace
            target_label: __metrics_path__
            regex: (.+)

      - job_name: 'kubernetes-pods'
        kubernetes_sd_configs:
          - role: pod
            namespaces:
              names:
              - services-lab
        relabel_configs:
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
            action: keep
            regex: true

---
apiVersion: v1
kind: Service
metadata:
  name: prometheus-service
  namespace: services-lab
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9090"
  labels:
    app: performance-monitor
spec:
  type: NodePort
  selector:
    app: performance-monitor
  ports:
  - name: prometheus
    port: 9090
    targetPort: 9090
    nodePort: 30900
    protocol: TCP
```

```bash
# Deploy performance monitoring
kubectl apply -f troubleshooting/performance-monitor.yaml
kubectl wait --for=condition=available deployment/performance-monitor --timeout=120s
echo

# Performance testing and optimization
echo "🏎️  Running comprehensive performance tests..."

# Test 1: Baseline performance measurement
echo "Test 1: Baseline performance measurement..."
echo "Measuring service response times..."
for i in {1..10}; do
    RESPONSE_TIME=$(kubectl exec -it $DEBUG_POD -- curl -s -w "%{time_total}" -o /dev/null http://nginx-clusterip-service/)
    echo "Request $i: ${RESPONSE_TIME}s"
done | tee /tmp/baseline_performance.log
echo

# Calculate average response time
AVERAGE=$(awk '{sum+=$3} END {print sum/NR}' /tmp/baseline_performance.log 2>/dev/null || echo "Unable to calculate")
echo "Average response time: ${AVERAGE}s"
echo

# Test 2: Load testing with concurrent connections
echo "Test 2: Load testing with concurrent connections..."
kubectl exec -it $DEBUG_POD -- bash -c '
echo "Running concurrent load test..."
for i in {1..20}; do
    (curl -s -w "%{time_total}\n" -o /dev/null http://nginx-clusterip-service/) &
done
wait
' | tee /tmp/concurrent_test.log
echo

# Analyze concurrent performance
echo "📊 Concurrent test analysis:"
CONCURRENT_AVERAGE=$(awk '{sum+=$1} END {print sum/NR}' /tmp/concurrent_test.log 2>/dev/null || echo "Unable to calculate")
echo "Average concurrent response time: ${CONCURRENT_AVERAGE}s"
echo

# Test 3: Session affinity performance impact
echo "Test 3: Testing session affinity performance impact..."
echo "Testing service with session affinity..."
kubectl patch service nginx-clusterip-service -p '{"spec":{"sessionAffinity":"ClientIP"}}'

# Wait for patch to take effect
sleep 5

echo "Performance with session affinity:"
for i in {1..5}; do
    RESPONSE_TIME=$(kubectl exec -it $DEBUG_POD -- curl -s -w "%{time_total}" -o /dev/null http://nginx-clusterip-service/)
    echo "Request $i: ${RESPONSE_TIME}s"
done
echo

# Reset session affinity
kubectl patch service nginx-clusterip-service -p '{"spec":{"sessionAffinity":"None"}}'

# Test 4: Service mesh simulation with performance monitoring
echo "Test 4: Advanced load balancing analysis..."
echo "Analyzing load distribution and performance..."
declare -A pod_performance
declare -A pod_count

# Get pod IPs for direct testing
POD_IPS=($(kubectl get pods -l app=nginx -o jsonpath='{.items[*].status.podIP}'))

echo "Testing direct pod access performance:"
for pod_ip in "${POD_IPS[@]}"; do
    echo "Testing pod: $pod_ip"
    TOTAL_TIME=0
    for i in {1..5}; do
        RESPONSE_TIME=$(kubectl exec -it $DEBUG_POD -- curl -s -w "%{time_total}" -o /dev/null http://$pod_ip/ 2>/dev/null || echo "0")
        TOTAL_TIME=$(echo "$TOTAL_TIME + $RESPONSE_TIME" | bc 2>/dev/null || echo "0")
    done
    AVERAGE_TIME=$(echo "scale=4; $TOTAL_TIME / 5" | bc 2>/dev/null || echo "0")
    echo "  Average response time: ${AVERAGE_TIME}s"
    pod_performance[$pod_ip]=$AVERAGE_TIME
done
echo

# Create performance optimization recommendations
echo "📋 Creating performance optimization report..."
nano troubleshooting/performance-report.md
```

**File Content for performance-report.md:**
```markdown
# Service Performance Analysis Report

## Performance Test Results

### Baseline Performance
- **Average Response Time**: Measured during single-request testing
- **Concurrent Performance**: Response time under load
- **Session Affinity Impact**: Performance difference with ClientIP affinity

### Key Findings

1. **Service Overhead Analysis**
   - ClusterIP service adds minimal latency (~0.1ms)
   - Load balancing algorithm impacts performance
   - Session affinity can improve cache performance but may cause uneven distribution

2. **Scaling Recommendations**
   - Increase replicas for high-traffic services
   - Use horizontal pod autoscaling for dynamic load
   - Consider pod disruption budgets for availability

3. **Network Optimization**
   - Use headless services for direct pod access when appropriate
   - Implement proper resource limits to prevent resource contention
   - Consider topology-aware routing for large clusters

### Performance Optimization Strategies

#### Service-Level Optimizations
- **Session Affinity**: Use ClientIP for stateful applications
- **External Traffic Policy**: Use 'Local' to preserve source IP and reduce hops
- **Load Balancing**: Consider IPVS mode for better performance

#### Pod-Level Optimizations
- **Resource Requests/Limits**: Proper sizing prevents throttling
- **Readiness Probes**: Faster probe intervals for quicker endpoint updates
- **Quality of Service**: Use 'Guaranteed' QoS for critical services

#### Infrastructure Optimizations
- **Node Placement**: Use node affinity for latency-sensitive workloads
- **Network Policies**: Minimize unnecessary network hops
- **CNI Selection**: Choose appropriate CNI plugin for performance requirements

## Monitoring Recommendations

### Key Metrics to Monitor
- Service response time percentiles (p50, p95, p99)
- Request rate and error rate
- Endpoint availability and health
- Pod resource utilization

### Alerting Thresholds
- Response time > 500ms (warning)
- Response time > 1000ms (critical)
- Error rate > 1% (warning)
- Error rate > 5% (critical)
- Endpoint availability < 100% (warning)

### Tools and Implementation
- Prometheus for metrics collection
- Grafana for visualization
- AlertManager for alerting
- Service mesh (Istio/Linkerd) for advanced observability
```

```bash
echo

# Generate performance summary
echo "📊 Performance Test Summary:"
echo "=========================="
echo "✅ Baseline performance measured"
echo "✅ Concurrent load testing completed"
echo "✅ Session affinity impact analyzed"
echo "✅ Direct pod access performance tested"
echo "📋 Performance report generated"
echo

# Access Prometheus metrics (if available)
echo "🔍 Accessing performance metrics..."
echo "Prometheus web UI available at: http://localhost:30900"
echo "Use the following queries for service metrics:"
echo "  - up{job=\"kubernetes-services\"}"
echo "  - rate(http_requests_total[5m])"
echo "  - histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))"
echo

echo "✅ Performance monitoring and optimization complete!"
echo "💡 Learning Tip: Regular performance testing helps identify bottlenecks early"
echo "📈 Optimization Tip: Monitor p95 and p99 response times, not just averages"
echo "🔧 Troubleshooting: Use Prometheus queries to identify performance patterns"
```

## 🧹 Task 6: Advanced Cleanup and Best Practices Implementation

### Subtask 6.1: Comprehensive Resource Management

```bash
echo "=== TASK 6: ADVANCED CLEANUP AND BEST PRACTICES ==="
echo "🧹 Implementing comprehensive resource management and cleanup procedures..."
echo

# Create service inventory before cleanup
echo "📋 Creating comprehensive service inventory..."
echo "=== CURRENT SERVICE INVENTORY ===" > /tmp/service_inventory.txt
kubectl get services -o wide >> /tmp/service_inventory.txt
echo "" >> /tmp/service_inventory.txt
echo "=== ENDPOINTS SUMMARY ===" >> /tmp/service_inventory.txt
kubectl get endpoints -o wide >> /tmp/service_inventory.txt
echo "" >> /tmp/service_inventory.txt
echo "=== RESOURCE USAGE ===" >> /tmp/service_inventory.txt
kubectl top pods --no-headers 2>/dev/null | awk '{print $1, $2, $3}' >> /tmp/service_inventory.txt || echo "Metrics server not available" >> /tmp/service_inventory.txt

echo "📊 Service inventory created:"
cat /tmp/service_inventory.txt
echo

# Gradual cleanup with verification
echo "🔄 Performing gradual cleanup with verification..."

# Cleanup custom services first
echo "Step 1: Cleaning up custom services..."
CUSTOM_SERVICES=(
    "nginx-nodeport-service"
    "nginx-nodeport-auto" 
    "nginx-loadbalancer-service"
    "broken-service-selector"
    "broken-service-port"
    "orphaned-service"
    "nginx-headless-service"
    "backend-service"
    "api-gateway-service"
)

for service in "${CUSTOM_SERVICES[@]}"; do
    if kubectl get service "$service" >/dev/null 2>&1; then
        echo "  Deleting service: $service"
        kubectl delete service "$service"
        echo "  ✅ $service deleted"
    else
        echo "  ℹ️  Service $service not found"
    fi
done
echo

# Cleanup deployments
echo "Step 2: Cleaning up deployments..."
DEPLOYMENTS=(
    "nginx-app"
    "backend-app" 
    "api-gateway"
    "debug-toolkit"
    "performance-monitor"
)

for deployment in "${DEPLOYMENTS[@]}"; do
    if kubectl get deployment "$deployment" >/dev/null 2>&1; then
        echo "  Scaling down: $deployment"
        kubectl scale deployment "$deployment" --replicas=0
        echo "  Deleting deployment: $deployment"
        kubectl delete deployment "$deployment" --timeout=60s
        echo "  ✅ $deployment deleted"
    else
        echo "  ℹ️  Deployment $deployment not found"
    fi
done
echo

# Cleanup pods
echo "Step 3: Cleaning up standalone pods..."
STANDALONE_PODS=(
    "network-debug"
    "dns-test-advanced"
    "isolated-test-pod"
)

for pod in "${STANDALONE_PODS[@]}"; do
    if kubectl get pod "$pod" >/dev/null 2>&1; then
        echo "  Deleting pod: $pod"
        kubectl delete pod "$pod" --grace-period=30
        echo "  ✅ $pod deleted"
    else
        echo "  ℹ️  Pod $pod not found"
    fi
done
echo

# Cleanup ConfigMaps
echo "Step 4: Cleaning up ConfigMaps..."
kubectl delete configmap nginx-config api-gateway-config debug-scripts prometheus-config metallb-config 2>/dev/null || echo "  ℹ️  Some ConfigMaps not found"
echo

# Cleanup RBAC resources
echo "Step 5: Cleaning up RBAC resources..."
kubectl delete serviceaccount debug-service-account 2>/dev/null || echo "  ℹ️  ServiceAccount not found"
kubectl delete clusterrole debug-cluster-role 2>/dev/null || echo "  ℹ️  ClusterRole not found"
kubectl delete clusterrolebinding debug-cluster-role-binding 2>/dev/null || echo "  ℹ️  ClusterRoleBinding not found"
echo

# Final verification
echo "Step 6: Final verification..."
kubectl get all
echo

# Clean up remaining service if needed
if kubectl get service nginx-clusterip-service >/dev/null 2>&1; then
    echo "Cleaning up remaining nginx-clusterip-service..."
    kubectl delete service nginx-clusterip-service
fi

echo "Step 7: Cleaning up MetalLB (if installed)..."
kubectl delete -f https://raw.githubusercontent.com/metallb/metallb/v0.13.7/config/manifests/metallb-native.yaml 2>/dev/null || echo "  ℹ️  MetalLB not installed"

# Clean up test namespace
echo "Step 8: Cleaning up test resources..."
kubectl delete namespace test-namespace 2>/dev/null || echo "  ℹ️  test-namespace not found"
echo

echo "✅ Comprehensive cleanup complete!"
echo "🔍 Remaining resources in services-lab namespace:"
kubectl get all
echo
```

### Subtask 6.2: Production-Ready Service Configuration Template

```bash
echo "=== SUBTASK 6.2: PRODUCTION-READY SERVICE TEMPLATES ==="
echo "📝 Creating production-ready service configuration templates and best practices..."
echo

# Create production service template
echo "⚙️  Creating production service configuration template..."
nano configs/production-service-template.yaml
```

**File Content for production-service-template.yaml:**
```yaml
# Production-Ready Kubernetes Service Template
# This template includes best practices for production deployments

apiVersion: apps/v1
kind: Deployment
metadata:
  name: production-app
  namespace: production
  labels:
    app: production-app
    version: v1.0.0
    tier: frontend
    environment: production
    component: web-server
  annotations:
    description: "Production web application with best practices"
    owner: "platform-team"
    contact: "platform@company.com"
    version: "1.0.0"
    changelog: "Initial production deployment"
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0  # Ensure zero downtime
  selector:
    matchLabels:
      app: production-app
      tier: frontend
  template:
    metadata:
      labels:
        app: production-app
        version: v1.0.0
        tier: frontend
        environment: production
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      serviceAccountName: production-app-sa
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 2000
      containers:
      - name: app
        image: nginx:1.21-alpine
        imagePullPolicy: Always
        ports:
        - name: http
          containerPort: 80
          protocol: TCP
        - name: metrics
          containerPort: 8080
          protocol: TCP
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
            ephemeral-storage: "1Gi"
          limits:
            memory: "256Mi"
            cpu: "200m"
            ephemeral-storage: "2Gi"
        livenessProbe:
          httpGet:
            path: /health
            port: http
            scheme: HTTP
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
          successThreshold: 1
        readinessProbe:
          httpGet:
            path: /ready
            port: http
            scheme: HTTP
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
          successThreshold: 1
        startupProbe:
          httpGet:
            path: /startup
            port: http
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 30
        env:
        - name: ENVIRONMENT
          value: "production"
        - name: LOG_LEVEL
          value: "info"
        - name: METRICS_PORT
          value: "8080"
        volumeMounts:
        - name: app-config
          mountPath: /etc/nginx/conf.d
          readOnly: true
        - name: tmp
          mountPath: /tmp
        - name: var-cache
          mountPath: /var/cache/nginx
        - name: var-run
          mountPath: /var/run
      volumes:
      - name: app-config
        configMap:
          name: production-app-config
          defaultMode: 0644
      - name: tmp
        emptyDir: {}
      - name: var-cache
        emptyDir: {}
      - name: var-run
        emptyDir: {}
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - production-app
              topologyKey: kubernetes.io/hostname
      tolerations:
      - key: "high-priority"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: production-app
      terminationGracePeriodSeconds: 60

---
# Production ClusterIP Service
apiVersion: v1
kind: Service
metadata:
  name: production-app-clusterip
  namespace: production
  labels:
    app: production-app
    service-type: clusterip
    tier: frontend
    environment: production
  annotations:
    description: "Internal service for production application"
    service.kubernetes.io/load-balancer-class: "internal"
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/metrics"
spec:
  type: ClusterIP
  selector:
    app: production-app
    tier: frontend
  ports:
  - name: http
    port: 80
    targetPort: http
    protocol: TCP
  - name: metrics
    port: 8080
    targetPort: metrics
    protocol: TCP
  sessionAffinity: None
  ipFamilyPolicy: SingleStack
  ipFamilies:
  - IPv4

---
# Production LoadBalancer Service
apiVersion: v1
kind: Service
metadata:
  name: production-app-loadbalancer
  namespace: production
  labels:
    app: production-app
    service-type: loadbalancer
    tier: frontend
    environment: production
  annotations:
    description: "External LoadBalancer service for production application"
    # AWS Load Balancer Controller annotations
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
    service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "http"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-protocol: "http"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-path: "/health"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval-seconds: "10"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-timeout-seconds: "5"
    service.beta.kubernetes.io/aws-load-balancer-healthy-threshold-count: "2"
    service.beta.kubernetes.io/aws-load-balancer-unhealthy-threshold-count: "3"
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: "arn:aws:acm:region:account:certificate/cert-id"
    service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "443"
    service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy: "ELBSecurityPolicy-TLS-1-2-2017-01"
    # Security annotations
    service.beta.kubernetes.io/aws-load-balancer-access-log-enabled: "true"
    service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-name: "company-lb-access-logs"
    service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-prefix: "production-app"
spec:
  type: LoadBalancer
  selector:
    app: production-app
    tier: frontend
  ports:
  - name: http
    port: 80
    targetPort: http
    protocol: TCP
  - name: https
    port: 443
    targetPort: http
    protocol: TCP
  sessionAffinity: None
  externalTrafficPolicy: Local  # Preserve source IP and reduce hops
  loadBalancerSourceRanges:
  - "10.0.0.0/8"      # Internal corporate network
  - "172.16.0.0/12"   # Private network range
  - "192.168.0.0/16"  # Private network range
  ipFamilyPolicy: SingleStack
  ipFamilies:
  - IPv4

---
# ServiceAccount with minimal permissions
apiVersion: v1
kind: ServiceAccount
metadata:
  name: production-app-sa
  namespace: production
  labels:
    app: production-app
  annotations:
    description: "Service account for production application"

---
# Role with minimal required permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: production-app-role
  labels:
    app: production-app
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list", "watch"]

---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: production-app-rolebinding
  namespace: production
  labels:
    app: production-app
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: production-app-role
subjects:
- kind: ServiceAccount
  name: production-app-sa
  namespace: production

---
# Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: production-app-hpa
  namespace: production
  labels:
    app: production-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: production-app
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
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
      - type: Pods
        value: 2
        periodSeconds: 60
      selectPolicy: Max

---
# Pod Disruption Budget
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: production-app-pdb
  namespace: production
  labels:
    app: production-app
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: production-app
      tier: frontend

---
# Network Policy for security
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: production-app-netpol
  namespace: production
  labels:
    app: production-app
spec:
  podSelector:
    matchLabels:
      app: production-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-system
    - namespaceSelector:
        matchLabels:
          name: monitoring
    ports:
    - protocol: TCP
      port: 80
    - protocol: TCP
      port: 8080
  egress:
  - to: []
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
  - to:
    - namespaceSelector:
        matchLabels:
          name: database
    ports:
    - protocol: TCP
      port: 5432
```

```bash
# Create production best practices documentation
echo "📚 Creating production best practices documentation..."
nano configs/service-best-practices.md
```

**File Content for service-best-practices.md:**
```markdown
# Kubernetes Services Production Best Practices

## 🏭 Production Service Configuration Checklist

### Service Design Principles
- [ ] **Single Responsibility**: Each service should have a single, well-defined purpose
- [ ] **Statelessness**: Services should be stateless to enable horizontal scaling
- [ ] **Health Checks**: Implement comprehensive health, readiness, and startup probes
- [ ] **Resource Limits**: Define appropriate resource requests and limits
- [ ] **Security Context**: Run containers with minimal privileges

### Service Types Selection Guide

#### ClusterIP (Default)
- **Use for**: Internal microservice communication
- **Benefits**: Secure, fast, simple
- **Best practices**: 
  - Use for service-to-service communication
  - Implement proper DNS naming conventions
  - Use headless services for direct pod access when needed

#### NodePort
- **Use for**: Development, testing, debugging
- **Benefits**: Direct external access without load balancer
- **Best practices**:
  - Restrict source IP ranges with firewall rules
  - Use high-numbered ports (30000-32767)
  - Document port assignments to avoid conflicts

#### LoadBalancer
- **Use for**: Production external access
- **Benefits**: Automatic provisioning, health checking, SSL termination
- **Best practices**:
  - Use cloud provider annotations for advanced features
  - Implement proper SSL/TLS configuration
  - Configure health check paths and intervals
  - Use externalTrafficPolicy: Local for source IP preservation

### Labeling and Annotation Standards

#### Required Labels
```yaml
labels:
  app: application-name           # Application identifier
  version: v1.0.0                # Application version
  component: web-server          # Component type
  tier: frontend                 # Application tier
  environment: production        # Environment
  managed-by: kubernetes         # Management tool
```

#### Recommended Annotations
```yaml
annotations:
  description: "Service description"
  owner: "team-name"
  contact: "team@company.com"
  prometheus.io/scrape: "true"   # Monitoring
  prometheus.io/port: "8080"
  prometheus.io/path: "/metrics"
```

### Security Best Practices

#### Network Policies
- Implement default deny-all policies
- Allow only necessary traffic between services
- Use namespace isolation for multi-tenant environments

#### Service Accounts
- Create dedicated service accounts for each application
- Follow principle of least privilege
- Use workload identity where available (GKE, AKS, EKS)

#### Pod Security
- Run containers as non-root users
- Use read-only root filesystems when possible
- Implement security contexts with appropriate user/group IDs
- Use Pod Security Standards/Pod Security Policies

### High Availability Configuration

#### Deployment Strategy
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0  # Zero downtime deployments
```

#### Pod Distribution
```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: your-app
      topologyKey: kubernetes.io/hostname
```

#### Disruption Budgets
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 50%
  selector:
    matchLabels:
      app: your-app
```

### Performance Optimization

#### Resource Management
- Set appropriate CPU and memory requests/limits
- Use Guaranteed QoS class for critical services
- Monitor and adjust based on actual usage

#### Auto-scaling
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: your-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### Monitoring and Observability

#### Essential Metrics
- **Service-level**: Request rate, error rate, response time
- **Infrastructure-level**: CPU, memory, network utilization
- **Business-level**: User satisfaction, conversion rates

#### Health Check Implementation
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

### Service Mesh Integration

#### Istio Configuration
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: app-vs
spec:
  http:
  - match:
    - uri:
        prefix: /api/
    route:
    - destination:
        host: backend-service
  - route:
    - destination:
        host: frontend-service
```

### Disaster Recovery

#### Backup Strategy
- Regular etcd backups
- Application data backup procedures
- Configuration management in version control

#### Multi-Region Deployment
- Cross-region service replication
- DNS-based failover mechanisms
- Data synchronization strategies

### Documentation Requirements

#### Service Documentation
- API documentation with OpenAPI/Swagger
- Runbooks for common operations
- Troubleshooting guides
- Architecture diagrams

#### Change Management
- Version control for all configurations
- Automated testing pipelines
- Rollback procedures
- Change approval processes

## 🔧 Production Deployment Checklist

### Pre-Deployment
- [ ] Code review completed
- [ ] Security scan passed
- [ ] Performance testing completed
- [ ] Documentation updated
- [ ] Monitoring configured
- [ ] Alerting rules defined

### Deployment
- [ ] Blue-green or canary deployment strategy
- [ ] Health checks configured
- [ ] Resource limits appropriate
- [ ] Service mesh integration (if applicable)
- [ ] SSL/TLS certificates valid

### Post-Deployment
- [ ] Service functionality verified
- [ ] Monitoring dashboards updated
- [ ] Load testing completed
- [ ] Rollback plan tested
- [ ] Team notification sent
- [ ] Documentation updated

## 🚨 Common Production Issues

### Issue: Service Unavailable
**Symptoms**: 502/503 errors, connection timeouts
**Troubleshooting**:
1. Check pod status and logs
2. Verify service endpoints
3. Test internal connectivity
4. Check resource constraints
5. Review network policies

### Issue: Poor Performance
**Symptoms**: High response times, timeouts
**Troubleshooting**:
1. Monitor resource utilization
2. Check auto-scaling configuration
3. Analyze traffic patterns
4. Review database performance
5. Check network latency

### Issue: Intermittent Failures
**Symptoms**: Random 5xx errors, partial outages
**Troubleshooting**:
1. Check pod health and readiness
2. Review load balancing configuration
3. Analyze traffic distribution
4. Check for resource contention
5. Review auto-scaling behavior
```

```bash
echo

# Create deployment validation script
echo "🔍 Creating deployment validation script..."
nano troubleshooting/validate-production-service.sh
```

**File Content for validate-production-service.sh:**
```bash
#!/bin/bash

# Production Service Validation Script
# This script validates a Kubernetes service deployment against best practices

SERVICE_NAME=${1:-"production-app-clusterip"}
NAMESPACE=${2:-"production"}

echo "=== Production Service Validation ==="
echo "Service: $SERVICE_NAME"
echo "Namespace: $NAMESPACE"
echo "Timestamp: $(date)"
echo

# Color codes for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Function to print colored output
print_status() {
    local status=$1
    local message=$2
    case $status in
        "PASS")
            echo -e "${GREEN}✅ PASS:${NC} $message"
            ;;
        "FAIL")
            echo -e "${RED}❌ FAIL:${NC} $message"
            ;;
        "WARN")
            echo -e "${YELLOW}⚠️  WARN:${NC} $message"
            ;;
        "INFO")
            echo -e "${BLUE}ℹ️  INFO:${NC} $message"
            ;;
    esac
}

# Test 1: Service Existence
echo "1. Service Existence Check"
if kubectl get service $SERVICE_NAME -n $NAMESPACE >/dev/null 2>&1; then
    print_status "PASS" "Service $SERVICE_NAME exists"
else
    print_status "FAIL" "Service $SERVICE_NAME not found"
    exit 1
fi
echo

# Test 2: Service Configuration Validation
echo "2. Service Configuration Validation"

# Check labels
LABELS=$(kubectl get service $SERVICE_NAME -n $NAMESPACE -o jsonpath='{.metadata.labels}')
if [[ "$LABELS" == *"app"* ]]; then
    print_status "PASS" "Service has app label"
else
    print_status "WARN" "Service missing app label"
fi

if [[ "$LABELS" == *"version"* ]]; then
    print_status "PASS" "Service has version label"
else
    print_status "WARN" "Service missing version label"
fi

# Check annotations
ANNOTATIONS=$(kubectl get service $SERVICE_NAME -n $NAMESPACE -o jsonpath='{.metadata.annotations}')
if [[ "$ANNOTATIONS" == *"description"* ]]; then
    print_status "PASS" "Service has description annotation"
else
    print_status "WARN" "Service missing description annotation"
fi
echo

# Test 3: Endpoint Validation
echo "3. Endpoint Validation"
ENDPOINT_COUNT=$(kubectl get endpoints $SERVICE_NAME -n $NAMESPACE -o jsonpath='{.subsets[0].addresses[*].ip}' 2>/dev/null | wc -w)

if [ "$ENDPOINT_COUNT" -gt 0 ]; then
    print_status "PASS" "Service has $ENDPOINT_COUNT active endpoints"
else
    print_status "FAIL" "Service has no active endpoints"
fi

# Check endpoint health
READY_ENDPOINTS=$(kubectl get endpoints $SERVICE_NAME -n $NAMESPACE -o jsonpath='{.subsets[0].addresses[*].ip}' 2>/dev/null | wc -w)
NOT_READY_ENDPOINTS=$(kubectl get endpoints $SERVICE_NAME -n $NAMESPACE -o jsonpath='{.subsets[0].notReadyAddresses[*].ip}' 2>/dev/null | wc -w)

if [ "$NOT_READY_ENDPOINTS" -gt 0 ]; then
    print_status "WARN" "$NOT_READY_ENDPOINTS endpoints are not ready"
else
    print_status "PASS" "All endpoints are ready"
fi
echo

# Test 4: Pod Health Check
echo "4. Backend Pod Health Check"
SELECTOR=$(kubectl get service $SERVICE_NAME -n $NAMESPACE -o jsonpath='{.spec.selector}' | jq -r 'to_entries[] | "\(.key)=\(.value)"' | tr '\n' ',' | sed 's/,$//' 2>/dev/null)

if [ -n "$SELECTOR" ]; then
    POD_COUNT=$(kubectl get pods -n $NAMESPACE -l "$SELECTOR" --field-selector=status.phase=Running --no-headers 2>/dev/null | wc -l)
    TOTAL_PODS=$(kubectl get pods -n $NAMESPACE -l "$SELECTOR" --no-headers 2>/dev/null | wc -l)
    
    if [ "$POD_COUNT" -eq "$TOTAL_PODS" ] && [ "$POD_COUNT" -gt 0 ]; then
        print_status "PASS" "All $POD_COUNT pods are running"
    else
        print_status "WARN" "Only $POD_COUNT out of $TOTAL_PODS pods are running"
    fi
    
    # Check pod readiness
    READY_PODS=$(kubectl get pods -n $NAMESPACE -l "$SELECTOR" -o jsonpath='{.items[*].status.conditions[?(@.type=="Ready")].status}' 2>/dev/null | grep -c "True")
    if [ "$READY_PODS" -eq "$POD_COUNT" ]; then
        print_status "PASS" "All pods are ready"
    else
        print_status "WARN" "Only $READY_PODS out of $POD_COUNT pods are ready"
    fi
else
    print_status "WARN" "Cannot determine pod selector"
fi
echo

# Test 5: Service Connectivity
echo "5. Service Connectivity Test"
SERVICE_IP=$(kubectl get service $SERVICE_NAME -n $NAMESPACE -o jsonpath='{.spec.clusterIP}')
SERVICE_PORT=$(kubectl get service $SERVICE_NAME -n $NAMESPACE -o jsonpath='{.spec.ports[0].port}')

# Test internal connectivity (if possible)
if kubectl run connectivity-test --image=busybox --rm -it --restart=Never --timeout=30s -- wget -qO- --timeout=5 "http://$SERVICE_IP:$SERVICE_PORT/" >/dev/null 2>&1; then
    print_status "PASS" "Service is reachable internally"
else
    print_status "WARN" "Service connectivity test failed or timed out"
fi
echo

# Test 6: Resource Usage Check
echo "6. Resource Usage Check"
if kubectl top pods -n $NAMESPACE -l "$SELECTOR" >/dev/null 2>&1; then
    print_status "INFO" "Resource usage data available"
    kubectl top pods -n $NAMESPACE -l "$SELECTOR" 2>/dev/null | head -5
else
    print_status "WARN" "Resource usage data not available (metrics-server required)"
fi
echo

# Test 7: Security Validation
echo "7. Security Configuration Check"

# Check if running as root
ROOT_PODS=$(kubectl get pods -n $NAMESPACE -l "$SELECTOR" -o jsonpath='{.items[*].spec.securityContext.runAsUser}' 2>/dev/null | grep -c "^0$" || echo "0")
if [ "$ROOT_PODS" -eq 0 ]; then
    print_status "PASS" "No pods running as root user"
else
    print_status "WARN" "$ROOT_PODS pods running as root user"
fi

# Check for security context
SECURITY_CONTEXT=$(kubectl get pods -n $NAMESPACE -l "$SELECTOR" -o jsonpath='{.items[0].spec.securityContext}' 2>/dev/null)
if [ -n "$SECURITY_CONTEXT" ] && [ "$SECURITY_CONTEXT" != "null" ]; then
    print_status "PASS" "Security context configured"
else
    print_status "WARN" "Security context not configured"
fi
echo

# Test 8: High Availability Check
echo "8. High Availability Configuration"

# Check replica count
REPLICA_COUNT=$(kubectl get deployment -n $NAMESPACE -l "$SELECTOR" -o jsonpath='{.items[0].spec.replicas}' 2>/dev/null)
if [ "$REPLICA_COUNT" -ge 3 ]; then
    print_status "PASS" "High availability: $REPLICA_COUNT replicas configured"
elif [ "$REPLICA_COUNT" -ge 2 ]; then
    print_status "WARN" "Medium availability: $REPLICA_COUNT replicas configured"
else
    print_status "WARN" "Low availability: $REPLICA_COUNT replicas configured"
fi

# Check for PodDisruptionBudget
if kubectl get pdb -n $NAMESPACE -l "$SELECTOR" >/dev/null 2>&1; then
    print_status "PASS" "PodDisruptionBudget configured"
else
    print_status "WARN" "No PodDisruptionBudget found"
fi
echo

# Summary
echo "=== Validation Summary ==="
echo "Service: $SERVICE_NAME"
echo "Namespace: $NAMESPACE"
echo "Validation completed at: $(date)"
echo
echo "Review any WARN or FAIL items above for production readiness."
echo "For detailed troubleshooting, use:"
echo "  kubectl describe service $SERVICE_NAME -n $NAMESPACE"
echo "  kubectl describe endpoints $SERVICE_NAME -n $NAMESPACE"
echo "  kubectl get pods -n $NAMESPACE -l \"$SELECTOR\" -o wide"
```

```bash
# Make validation script executable
chmod +x troubleshooting/validate-production-service.sh

echo "✅ Production service template and best practices documentation created!"
echo
echo "📋 Files created:"
echo "  - configs/production-service-template.yaml (Complete production template)"
echo "  - configs/service-best-practices.md (Comprehensive best practices guide)"
echo "  - troubleshooting/validate-production-service.sh (Validation script)"
echo
echo "💡 Usage examples:"
echo "  Apply template: kubectl apply -f configs/production-service-template.yaml"
echo "  Validate service: ./troubleshooting/validate-production-service.sh service-name namespace"
echo "  Best practices: cat configs/service-best-practices.md"
```

### Subtask 6.3: Service Management Automation and Final Summary

```bash
echo "=== SUBTASK 6.3: SERVICE MANAGEMENT AUTOMATION ==="
echo "🤖 Creating service management automation and final lab summary..."
echo

# Create service management automation script
echo "⚙️  Creating comprehensive service management automation..."
nano troubleshooting/service-manager.sh
```

**File Content for service-manager.sh:**
```bash
#!/bin/bash

# Kubernetes Service Management Automation Script
# Provides comprehensive service lifecycle management

set -e

# Configuration
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
LOG_FILE="/tmp/service-manager-${TIMESTAMP}.log"

# Color codes
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
PURPLE='\033[0;35m'
CYAN='\033[0;36m'
NC='\033[0m'

# Logging function
log() {
    echo -e "$1" | tee -a "$LOG_FILE"
}

# Help function
show_help() {
    cat << EOF
Kubernetes Service Manager

USAGE:
    $0 [COMMAND] [OPTIONS]

COMMANDS:
    deploy      Deploy a new service
    validate    Validate existing service
    monitor     Monitor service health
    scale       Scale service replicas
    update      Update service configuration
    cleanup     Clean up services
    backup      Backup service configurations
    restore     Restore service configurations
    test        Run comprehensive service tests
    help        Show this help message

OPTIONS:
    -n, --namespace NAMESPACE    Target namespace (default: default)
    -s, --service SERVICE        Service name
    -r, --replicas COUNT         Number of replicas for scaling
    -f, --file FILE             Configuration file
    -v, --verbose               Verbose output
    -d, --dry-run              Show what would be done without executing

EXAMPLES:
    $0 deploy -f production-service.yaml -n production
    $0 validate -s nginx-service -n default
    $0 scale -s nginx-service -r 5 -n production
    $0 monitor -s nginx-service -n production
    $0 test -n services-lab

EOF
}

# Service deployment function
deploy_service() {
    local file=$1
    local namespace=${2:-default}
    
    log "${BLUE}🚀 Deploying service from $file to namespace $namespace${NC}"
    
    if [ ! -f "$file" ]; then
        log "${RED}❌ Error: Configuration file $file not found${NC}"
        return 1
    fi
    
    # Validate YAML
    kubectl apply --dry-run=client -f "$file" >/dev/null 2>&1
    if [ $? -ne 0 ]; then
        log "${RED}❌ Error: Invalid YAML configuration${NC}"
        return 1
    fi
    
    # Apply configuration
    kubectl apply -f "$file" -n "$namespace"
    log "${GREEN}✅ Service deployed successfully${NC}"
    
    # Wait for rollout
    local deployment_name=$(kubectl get -f "$file" -o jsonpath='{.items[?(@.kind=="Deployment")].metadata.name}' 2>/dev/null)
    if [ -n "$deployment_name" ]; then
        log "${YELLOW}⏳ Waiting for rollout to complete...${NC}"
        kubectl rollout status deployment/"$deployment_name" -n "$namespace" --timeout=300s
    fi
}

# Service validation function
validate_service() {
    local service_name=$1
    local namespace=${2:-default}
    
    log "${BLUE}🔍 Validating service $service_name in namespace $namespace${NC}"
    
    if [ -f "$SCRIPT_DIR/validate-production-service.sh" ]; then
        bash "$SCRIPT_DIR/validate-production-service.sh" "$service_name" "$namespace"
    else
        log "${YELLOW}⚠️  Validation script not found, performing basic validation${NC}"
        
        # Basic validation
        kubectl get service "$service_name" -n "$namespace" >/dev/null 2>&1
        if [ $? -eq 0 ]; then
            log "${GREEN}✅ Service exists${NC}"
            kubectl get endpoints "$service_name" -n "$namespace"
        else
            log "${RED}❌ Service not found${NC}"
            return 1
        fi
    fi
}

# Service monitoring function
monitor_service() {
    local service_name=$1
    local namespace=${2:-default}
    
    log "${BLUE}📊 Monitoring service $service_name in namespace $namespace${NC}"
    
    # Service status
    log "${CYAN}Service Status:${NC}"
    kubectl get service "$service_name" -n "$namespace" -o wide
    
    # Endpoints
    log "${CYAN}Endpoints:${NC}"
    kubectl get endpoints "$service_name" -n "$namespace" -o wide
    
    # Backend pods
    log "${CYAN}Backend Pods:${NC}"
    local selector=$(kubectl get service "$service_name" -n "$namespace" -o jsonpath='{.spec.selector}' | jq -r 'to_entries[] | "\(.key)=\(.value)"' | tr '\n' ',' | sed 's/,$//' 2>/dev/null)
    if [ -n "$selector" ]; then
        kubectl get pods -n "$namespace" -l "$selector" -o wide
        
        # Resource usage
        log "${CYAN}Resource Usage:${NC}"
        kubectl top pods -n "$namespace" -l "$selector" 2>/dev/null || log "${YELLOW}⚠️  Metrics server not available${NC}"
    fi
    
    # Recent events
    log "${CYAN}Recent Events:${NC}"
    kubectl get events -n "$namespace" --sort-by='.lastTimestamp' | grep -i "$service_name" | tail -5
}

# Service scaling function
scale_service() {
    local service_name=$1
    local replicas=$2
    local namespace=${3:-default}
    
    log "${BLUE}⚖️  Scaling service $service_name to $replicas replicas in namespace $namespace${NC}"
    
    # Find deployment
    local selector=$(kubectl get service "$service_name" -n "$namespace" -o jsonpath='{.spec.selector}' | jq -r 'to_entries[] | "\(.key)=\(.value)"' | tr '\n' ',' | sed 's/,$//' 2>/dev/null)
    local deployment=$(kubectl get deployments -n "$namespace" -l "$selector" -o jsonpath='{.items[0].metadata.name}' 2>/dev/null)
    
    if [ -n "$deployment" ]; then
        kubectl scale deployment "$deployment" --replicas="$replicas" -n "$namespace"
        kubectl rollout status deployment/"$deployment" -n "$namespace" --timeout=300s
        log "${GREEN}✅ Service scaled successfully${NC}"
    else
        log "${RED}❌ No deployment found for service $service_name${NC}"
        return 1
    fi
}

# Service testing function
test_service() {
    local namespace=${1:-default}
    
    log "${BLUE}🧪 Running comprehensive service tests in namespace $namespace${NC}"
    
    # List all services
    log "${CYAN}Services in namespace $namespace:${NC}"
    kubectl get services -n "$namespace" -o wide
    
    # Test each service
    while IFS= read -r service; do
        if [ -n "$service" ]; then
            log "${PURPLE}Testing service: $service${NC}"
            
            # Basic connectivity test
            local cluster_ip=$(kubectl get service "$service" -n "$namespace" -o jsonpath='{.spec.clusterIP}' 2>/dev/null)
            local port=$(kubectl get service "$service" -n "$namespace" -o jsonpath='{.spec.ports[0].port}' 2>/dev/null)
            
            if [ -n "$cluster_ip" ] && [ -n "$port" ]; then
                # Create test pod
                kubectl run service-test-${TIMESTAMP} --image=busybox --rm -it --restart=Never --timeout=30s --namespace="$namespace" -- wget -qO- --timeout=5 "http://$cluster_ip:$port/" >/dev/null 2>&1
                if [ $? -eq 0 ]; then
                    log "${GREEN}✅ $service: Connectivity test passed${NC}"
                else
                    log "${RED}❌ $service: Connectivity test failed${NC}"
                fi
            else
                log "${YELLOW}⚠️  $service: Unable to test (missing IP/port)${NC}"
            fi
        fi
    done < <(kubectl get services -n "$namespace" --no-headers -o custom-columns=":metadata.name" 2>/dev/null)
}

# Backup function
backup_service() {
    local namespace=${1:-default}
    local backup_dir="/tmp/k8s-service-backup-${TIMESTAMP}"
    
    log "${BLUE}💾 Backing up services from namespace $namespace${NC}"
    
    mkdir -p "$backup_dir"
    
    # Backup services
    kubectl get services -n "$namespace" -o yaml > "$backup_dir/services.yaml"
    kubectl get deployments -n "$namespace" -o yaml > "$backup_dir/deployments.yaml"
    kubectl get configmaps -n "$namespace" -o yaml > "$backup_dir/configmaps.yaml"
    kubectl get endpoints -n "$namespace" -o yaml > "$backup_dir/endpoints.yaml"
    
    # Create backup archive
    tar -czf "/tmp/k8s-backup-${namespace}-${TIMESTAMP}.tar.gz" -C "/tmp" "k8s-service-backup-${TIMESTAMP}"
    
    log "${GREEN}✅ Backup completed: /tmp/k8s-backup-${namespace}-${TIMESTAMP}.tar.gz${NC}"
    
    # Cleanup temp directory
    rm -rf "$backup_dir"
}

# Main execution
main() {
    local command=$1
    shift
    
    # Parse arguments
    while [[ $# -gt 0 ]]; do
        case $1 in
            -n|--namespace)
                NAMESPACE="$2"
                shift 2
                ;;
            -s|--service)
                SERVICE="$2"
                shift 2
                ;;
            -r|--replicas)
                REPLICAS="$2"
                shift 2
                ;;
            -f|--file)
                FILE="$2"
                shift 2
                ;;
            -v|--verbose)
                set -x
                shift
                ;;
            -d|--dry-run)
                DRY_RUN=true
                shift
                ;;
            -h|--help)
                show_help
                exit 0
                ;;
            *)
                log "${RED}❌ Unknown option: $1${NC}"
                show_help
                exit 1
                ;;
        esac
    done
    
    # Set defaults
    NAMESPACE=${NAMESPACE:-default}
    
    log "${CYAN}=== Kubernetes Service Manager ===${NC}"
    log "${CYAN}Command: $command${NC}"
    log "${CYAN}Namespace: $NAMESPACE${NC}"
    log "${CYAN}Timestamp: $(date)${NC}"
    log "${CYAN}Log file: $LOG_FILE${NC}"
    echo
    
    # Execute command
    case $command in
        deploy)
            if [ -z "$FILE" ]; then
                log "${RED}❌ Error: -f/--file option required for deploy command${NC}"
                exit 1
            fi
            deploy_service "$FILE" "$NAMESPACE"
            ;;
        validate)
            if [ -z "$SERVICE" ]; then
                log "${RED}❌ Error: -s/--service option required for validate command${NC}"
                exit 1
            fi
            validate_service "$SERVICE" "$NAMESPACE"
            ;;
        monitor)
            if [ -z "$SERVICE" ]; then
                log "${RED}❌ Error: -s/--service option required for monitor command${NC}"
                exit 1
            fi
            monitor_service "$SERVICE" "$NAMESPACE"
            ;;
        scale)
            if [ -z "$SERVICE" ] || [ -z "$REPLICAS" ]; then
                log "${RED}❌ Error: -s/--service and -r/--replicas options required for scale command${NC}"
                exit 1
            fi
            scale_service "$SERVICE" "$REPLICAS" "$NAMESPACE"
            ;;
        test)
            test_service "$NAMESPACE"
            ;;
        backup)
            backup_service "$NAMESPACE"
            ;;
        help)
            show_help
            ;;
        *)
            log "${RED}❌ Error: Unknown command '$command'${NC}"
            show_help
            exit 1
            ;;
    esac
    
    log "${GREEN}✅ Operation completed successfully${NC}"
    log "${CYAN}Log saved to: $LOG_FILE${NC}"
}

# Execute main function with all arguments
main "$@"
```

```bash
# Make service manager executable
chmod +x troubleshooting/service-manager.sh

# Create final lab summary and cheat sheet
echo "📚 Creating comprehensive lab summary and cheat sheet..."
nano troubleshooting/k8s-services-cheatsheet.md
```

**File Content for k8s-services-cheatsheet.md:**
```markdown
# Kubernetes Services Complete Cheat Sheet

## 🚀 Quick Command Reference

### Service Management
```bash
# List all services
kubectl get services
kubectl get svc -A  # All namespaces

# Describe service
kubectl describe service SERVICE_NAME
kubectl describe svc SERVICE_NAME

# Get service YAML
kubectl get service SERVICE_NAME -o yaml

# Delete service
kubectl delete service SERVICE_NAME
```

### Service Types Quick Deploy
```bash
# ClusterIP (default)
kubectl expose deployment DEPLOYMENT_NAME --port=80 --target-port=8080

# NodePort
kubectl expose deployment DEPLOYMENT_NAME --type=NodePort --port=80

# LoadBalancer
kubectl expose deployment DEPLOYMENT_NAME --type=LoadBalancer --port=80
```

### Endpoint Management
```bash
# Check endpoints
kubectl get endpoints
kubectl get ep SERVICE_NAME

# Describe endpoints
kubectl describe endpoints SERVICE_NAME
```

### DNS Testing
```bash
# Test DNS resolution
kubectl run dns-test --image=busybox --rm -it --restart=Never -- nslookup SERVICE_NAME

# Test with FQDN
kubectl run dns-test --image=busybox --rm -it --restart=Never -- nslookup SERVICE_NAME.NAMESPACE.svc.cluster.local
```

### Service Connectivity Testing
```bash
# Test service connectivity
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -qO- http://SERVICE_NAME:PORT/

# Port forward for local testing
kubectl port-forward service/SERVICE_NAME 8080:80
```

## 🔧 Troubleshooting Commands

### Common Issues
```bash
# Check if service exists
kubectl get svc SERVICE_NAME

# Check endpoints (most important!)
kubectl get endpoints SERVICE_NAME

# Check pod labels match service selector
kubectl get pods --show-labels
kubectl describe service SERVICE_NAME | grep Selector

# Test internal connectivity
kubectl exec -it POD_NAME -- curl SERVICE_NAME:PORT

# Check service events
kubectl get events --sort-by='.lastTimestamp' | grep SERVICE_NAME
```

### Performance Analysis
```bash
# Check resource usage
kubectl top pods
kubectl top nodes

# Monitor service endpoints
kubectl get endpoints SERVICE_NAME --watch

# Check service configuration
kubectl get service SERVICE_NAME -o jsonpath='{.spec}' | jq
```

## 📝 YAML Templates

### ClusterIP Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-service
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
```

### NodePort Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080
```

### LoadBalancer Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
  externalTrafficPolicy: Local
```

### Headless Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-headless-service
spec:
  clusterIP: None
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
```

##
