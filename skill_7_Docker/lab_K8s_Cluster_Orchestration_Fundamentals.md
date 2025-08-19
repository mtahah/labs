# Lab 15: Kubernetes - Container Orchestration Fundamentals

## Lab Objectives
By the end of this lab, you will:
- Deploy and manage a local Kubernetes cluster
- Understand pod, service, and deployment concepts through hands-on practice
- Deploy a realistic multi-tier application (web frontend + database)
- Master essential kubectl commands with detailed output interpretation
- Practice troubleshooting through intentional failure scenarios
- Verify your understanding through built-in CLI assessments

## Prerequisites & System Requirements

### Required Knowledge
- Docker fundamentals (containers, images, basic commands)
- Linux command line basics
- YAML syntax understanding
- Basic networking concepts (ports, IP addresses)

### System Requirements (Ubuntu Linux)
**Minimum:**
- Ubuntu 20.04 LTS or later
- 4GB RAM (8GB recommended)
- 2 CPU cores (4 cores recommended) 
- 20GB free disk space
- Docker installed and running
- Internet connectivity for downloading components

### Required Accounts
**None required** - This lab runs entirely on your local machine.

---

## Task 1: Install and Configure Local Kubernetes

### Subtask 1.1: Install Prerequisites

**Step 1: Verify Docker is running**
```bash
docker --version
docker ps
```

**Expected Output:**
```
Docker version 24.0.x, build xxxxx

CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

**If Docker is not installed, install it:**
```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
```
*Log out and log back in for group changes to take effect*

**Step 2: Install kubectl (Kubernetes CLI)**

```bash
# Update system packages
sudo apt update

# Install required dependencies
sudo apt install -y curl wget apt-transport-https

# Download kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Make executable and move to PATH
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

**Step 3: Verify kubectl installation**
```bash
kubectl version --client --output=yaml
```

**Expected Output:**
```yaml
clientVersion:
  buildDate: "2023-08-15T10:46:23Z"
  compiler: gc
  gitCommit: 1234567890abcdef
  gitTreeState: clean
  gitVersion: v1.28.0
  goVersion: go1.20.7
  major: "1"
  minor: "28"
  platform: linux/amd64
```

### Subtask 1.2: Install and Start Minikube

**Step 1: Install minikube**

```bash
# Download minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# Install minikube
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Verify installation
minikube version
```

**Expected Output:**
```
minikube version: v1.31.2
commit: fd7ecd9c4599bef9f04c0986c4a0187f98a4396e
```

**Step 2: Start your Kubernetes cluster**
```bash
minikube start --driver=docker --memory=4096 --cpus=2
```

**Expected Output (Understanding Each Line):**
```
😄 minikube v1.31.2 on Ubuntu 22.04                    # Minikube version and OS detection
✨ Using the docker driver based on user configuration  # Driver selection (docker/virtualbox/etc)
👍 Starting control plane node minikube in cluster minikube # Creating the master node
🚜 Pulling base image ...                              # Downloading Kubernetes node image
🔥 Creating docker container (CPUs=2, Memory=4096MB) ... # Container resource allocation
🐳 Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...   # K8s and Docker versions being used
    ▪ Generating certificates and keys ...              # TLS setup for secure communication
    ▪ Booting up control plane ...                      # Starting Kubernetes master components
    ▪ Configuring RBAC rules ...                       # Setting up role-based access control
🔎 Verifying Kubernetes components...                   # Health checks on system components
🌟 Enabled addons: storage-provisioner, default-storageclass # Default features activated
🏄 Done! kubectl is now configured to use "minikube" cluster # Success - ready to use
```

**Step 3: Verify cluster status**
```bash
minikube status
```

**Expected Output (Field Explanations):**
```
minikube
type: Control Plane          # This node manages the cluster
host: Running               # Host OS container is operational  
kubelet: Running            # Node agent is running (manages pods)
apiserver: Running          # API server accepts kubectl commands
kubeconfig: Configured      # kubectl knows how to reach this cluster
```

**Step 4: Explore your cluster**
```bash
kubectl cluster-info
```

**Expected Output:**
```
Kubernetes control plane is running at https://192.168.49.2:8443
CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

```bash
kubectl get nodes -o wide
```

**Expected Output (Understanding Each Column):**
```
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
minikube   Ready    control-plane   2m    v1.27.4   192.168.49.2   <none>        Ubuntu 22.04.3 LTS   5.15.0-78-generic   docker://24.0.4
```
- **STATUS**: Node health (Ready/NotReady)
- **ROLES**: control-plane manages the cluster, worker nodes run applications
- **INTERNAL-IP**: Cluster-internal communication address
- **CONTAINER-RUNTIME**: Container engine (docker/containerd/cri-o)

---

## Task 2: Deploy a Realistic Multi-Tier Application

### Subtask 2.1: Create Project Structure

**Step 1: Set up workspace**
```bash
mkdir ~/k8s-webapp-lab && cd ~/k8s-webapp-lab

echo "Current working directory: $(pwd)"
```

**Step 2: Understand what we're building**
We'll deploy a realistic web application with:
- **Frontend**: Nginx web server serving a simple HTML page
- **Backend**: Redis database for session storage
- **Communication**: Frontend connects to Redis for data

### Subtask 2.2: Deploy Single-Container Pod (Foundation)

**Step 1: Create a simple web application pod**
```bash
cat > webapp-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: webapp-frontend
  labels:
    app: webapp
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
      name: http
    env:
    - name: ENVIRONMENT
      value: "development"
EOF
```

**Step 2: Deploy and examine the pod**
```bash
kubectl apply -f webapp-pod.yaml
```

**Expected Output:**
```
pod/webapp-frontend created
```

**Step 3: Watch the pod start up**
```bash
kubectl get pods -w
```
*Press Ctrl+C after the pod shows STATUS as Running*

**Expected Output:**
```
NAME              READY   STATUS              RESTARTS   AGE
webapp-frontend   0/1     ContainerCreating   0          5s
webapp-frontend   1/1     Running             0          15s
```

**Understanding the Status Progression:**
- **ContainerCreating**: Kubernetes is pulling the image and starting the container
- **Running**: Container is healthy and ready to serve traffic
- **READY 1/1**: 1 out of 1 containers in the pod are ready

**Step 4: Get detailed pod information**
```bash
kubectl describe pod webapp-frontend
```

**Key sections to understand in the output:**
- **Events**: Shows the lifecycle (Scheduled → Pulled → Created → Started)
- **Conditions**: Pod and container readiness status
- **IP**: Internal cluster IP assigned to this pod

### Subtask 2.3: Deploy Multi-Container Pod

**Step 1: Create a multi-tier application pod**
```bash
cat > webapp-multi-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: webapp-multi
  labels:
    app: webapp
    tier: full-stack
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
      name: http
  - name: redis
    image: redis:6-alpine
    ports:
    - containerPort: 6379
      name: redis
    command: ["redis-server"]
    args: ["--appendonly", "yes"]
EOF
```

**Step 2: Deploy and monitor startup**
```bash
kubectl apply -f webapp-multi-pod.yaml

# Watch both containers start
kubectl get pods -w
```

**Expected Output:**
```
NAME           READY   STATUS              RESTARTS   AGE
webapp-multi   0/2     ContainerCreating   0          5s
webapp-multi   1/2     Running             0          15s
webapp-multi   2/2     Running             0          20s
```

**Understanding READY 2/2**: Both nginx AND redis containers are running successfully in the same pod.

**Step 3: Examine container logs separately**
```bash
# View nginx logs
kubectl logs webapp-multi -c nginx

echo "--- Separator ---"

# View redis logs  
kubectl logs webapp-multi -c redis
```

**Step 4: Test inter-container communication**
```bash
# Test that nginx can reach redis on localhost
kubectl exec webapp-multi -c nginx -- nc -z localhost 6379 && echo "Redis is reachable from nginx"
```

### 🔧 **Challenge 1: Break and Fix (Intentional Failure)**

**Step 1: Create a broken pod**
```bash
cat > broken-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: broken-webapp
spec:
  containers:
  - name: nginx
    image: nginx:nonexistent-tag
    ports:
    - containerPort: 80
EOF
```

**Step 2: Deploy and observe the failure**
```bash
kubectl apply -f broken-pod.yaml

kubectl get pods
```

**Expected Output:**
```
NAME            READY   STATUS         RESTARTS   AGE
broken-webapp   0/1     ErrImagePull   0          30s
```

**Step 3: Diagnose the problem**
```bash
kubectl describe pod broken-webapp
```

Look for the **Events** section at the bottom. You'll see:
```
Events:
  Type     Reason     Age   From               Message
  ----     ------     ----  ----               -------
  Warning  Failed     30s   kubelet            Failed to pull image "nginx:nonexistent-tag": rpc error: code = NotFound desc = pull access denied for nginx
```

**Step 4: Fix the problem**
```bash
kubectl delete pod broken-webapp

# Fix the image tag in broken-pod.yaml
sed -i 's/nginx:nonexistent-tag/nginx:1.21/' broken-pod.yaml

kubectl apply -f broken-pod.yaml
```

**Verification:**
```bash
kubectl get pods broken-webapp
```

Should now show `STATUS: Running`

---

## Task 3: Scale with Deployments and Expose with Services

### Subtask 3.1: Create a Production-Ready Deployment

**Step 1: Create a deployment for our web application**
```bash
cat > webapp-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
  labels:
    app: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
      tier: frontend
  template:
    metadata:
      labels:
        app: webapp
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
          name: http
        resources:
          limits:
            memory: "128Mi"
            cpu: "100m"
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
EOF
```

**Step 2: Deploy and watch the rollout**
```bash
kubectl apply -f webapp-deployment.yaml

# Watch the deployment create pods
kubectl get deployments -w
```
*Press Ctrl+C after READY shows 3/3*

**Expected Output:**
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
webapp-deployment  0/3     3            0           5s
webapp-deployment  1/3     3            1           15s
webapp-deployment  2/3     3            2           25s
webapp-deployment  3/3     3            3           30s
```

**Understanding Deployment Fields:**
- **READY**: How many replicas are ready to serve traffic
- **UP-TO-DATE**: Replicas updated with latest configuration
- **AVAILABLE**: Replicas available to users

**Step 3: Examine the created pods**
```bash
kubectl get pods -l app=webapp --show-labels
```

**Expected Output:**
```
NAME                                 READY   STATUS    RESTARTS   AGE   LABELS
webapp-deployment-7d4b449c8f-abc12   1/1     Running   0          2m    app=webapp,pod-template-hash=7d4b449c8f,tier=frontend
webapp-deployment-7d4b449c8f-def34   1/1     Running   0          2m    app=webapp,pod-template-hash=7d4b449c8f,tier=frontend
webapp-deployment-7d4b449c8f-ghi56   1/1     Running   0          2m    app=webapp,pod-template-hash=7d4b449c8f,tier=frontend
```

**Notice**: Kubernetes automatically generated unique names and added pod-template-hash labels.

### Subtask 3.2: Create Services for Network Access

**Step 1: Create a ClusterIP service (internal access)**
```bash
cat > webapp-service-cluster.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
  labels:
    app: webapp
spec:
  selector:
    app: webapp
    tier: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: http
      name: web
  type: ClusterIP
EOF
```

**Step 2: Deploy the service**
```bash
kubectl apply -f webapp-service-cluster.yaml

kubectl get services
```

**Expected Output:**
```
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP   30m
webapp-service   ClusterIP   10.96.123.45    <none>        80/TCP    5s
```

**Understanding Service Fields:**
- **TYPE**: ClusterIP = internal only, NodePort = external via node port, LoadBalancer = external via LB
- **CLUSTER-IP**: Internal cluster IP address (reachable only within cluster)
- **EXTERNAL-IP**: External IP (none for ClusterIP)

**Step 3: Test internal service connectivity**
```bash
# Get the service cluster IP
CLUSTER_IP=$(kubectl get service webapp-service -o jsonpath='{.spec.clusterIP}')

echo "Service ClusterIP: $CLUSTER_IP"

# Test from within the cluster
kubectl run test-pod --image=curlimages/curl --rm -it --restart=Never -- curl $CLUSTER_IP
```

**Expected Output:**
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

**Step 4: Create NodePort service for external access**
```bash
cat > webapp-service-nodeport.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: webapp-nodeport
  labels:
    app: webapp
spec:
  selector:
    app: webapp
    tier: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: http
      nodePort: 30080
  type: NodePort
EOF
```

**Step 5: Deploy NodePort service**
```bash
kubectl apply -f webapp-service-nodeport.yaml

kubectl get services
```

**Expected Output:**
```
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
webapp-nodeport   NodePort    10.96.234.56    <none>        80:30080/TCP   5s
```

**Understanding PORT(S) Field:**
`80:30080/TCP` means:
- Port 80: Internal cluster port
- Port 30080: External node port (accessible from outside)

### Subtask 3.3: Test External Access

**Step 1: Get minikube IP and test**
```bash
# Get the minikube node IP
MINIKUBE_IP=$(minikube ip)
echo "Minikube IP: $MINIKUBE_IP"

# Test external access
curl http://$MINIKUBE_IP:30080
```

**Step 2: Use minikube service command**
```bash
minikube service webapp-nodeport --url
```

**Expected Output:**
```
http://192.168.49.2:30080
```

### 🔧 **Challenge 2: Scale and Load Balance**

**Step 1: Scale up the deployment**
```bash
kubectl scale deployment webapp-deployment --replicas=5

kubectl get pods -l app=webapp
```

**Expected**: You should now see 5 pods running.

**Step 2: Test load balancing**
```bash
# Run multiple requests to see different pods responding
for i in {1..10}; do
  kubectl exec webapp-deployment-$(kubectl get pods -l app=webapp --no-headers | head -1 | awk '{print $1}' | cut -d'-' -f3-) -- hostname
done
```

**Step 3: Scale down**
```bash
kubectl scale deployment webapp-deployment --replicas=2

kubectl get pods -l app=webapp -w
```

Watch how Kubernetes gracefully terminates excess pods.

---

## Task 4: Add Database Tier and Service Communication

### Subtask 4.1: Deploy Redis Database

**Step 1: Create Redis deployment**
```bash
cat > redis-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      tier: database
  template:
    metadata:
      labels:
        app: redis
        tier: database
    spec:
      containers:
      - name: redis
        image: redis:6-alpine
        ports:
        - containerPort: 6379
        args: ["redis-server", "--appendonly", "yes"]
        resources:
          limits:
            memory: "256Mi"
            cpu: "100m"
EOF
```

**Step 2: Create Redis service**
```bash
cat > redis-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: redis-service
spec:
  selector:
    app: redis
    tier: database
  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379
  type: ClusterIP
EOF
```

**Step 3: Deploy Redis components**
```bash
kubectl apply -f redis-deployment.yaml
kubectl apply -f redis-service.yaml

kubectl get all -l app=redis
```

### Subtask 4.2: Test Inter-Service Communication

**Step 1: Test Redis connectivity from webapp**
```bash
# Get a webapp pod name
WEBAPP_POD=$(kubectl get pods -l app=webapp --no-headers | head -1 | awk '{print $1}')

echo "Testing from pod: $WEBAPP_POD"

# Test Redis connection using service name (DNS resolution)
kubectl exec $WEBAPP_POD -- nc -z redis-service 6379 && echo "✅ Redis service is reachable"
```

**Step 2: Interact with Redis**
```bash
# Run redis-cli in the cluster
kubectl run redis-client --image=redis:6-alpine --rm -it --restart=Never -- redis-cli -h redis-service

# In the redis-cli prompt, try:
# SET greeting "Hello from Kubernetes"
# GET greeting
# exit
```

### 🔧 **Challenge 3: Service Discovery**

**Step 1: Explore DNS resolution**
```bash
# Create a test pod with network tools
kubectl run netshoot --image=nicolaka/netshoot --rm -it --restart=Never -- bash

# Inside the pod, try:
# nslookup webapp-service
# nslookup redis-service
# dig webapp-service
# exit
```

**Expected**: You'll see how Kubernetes DNS automatically resolves service names to IP addresses.

---

## Task 5: Master kubectl Commands and Troubleshooting

### Subtask 5.1: Essential kubectl Commands

**Step 1: Resource discovery**
```bash
# List all resource types
kubectl api-resources

echo "--- Separator ---"

# Get all resources in default namespace
kubectl get all

echo "--- Separator ---" 

# Get resources with more details
kubectl get all -o wide
```

**Step 2: Output formatting**
```bash
# Get pod information as YAML
kubectl get pod $WEBAPP_POD -o yaml

echo "--- Separator ---"

# Get specific fields using JSONPath
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.phase}{"\n"}{end}'

echo "--- Separator ---"

# Custom columns
kubectl get pods -o custom-columns="NAME:.metadata.name,STATUS:.status.phase,IP:.status.podIP"
```

**Step 3: Filtering and labeling**
```bash
# Get resources by labels
kubectl get all -l app=webapp
kubectl get all -l tier=frontend
kubectl get all -l app=webapp,tier=frontend

echo "--- Separator ---"

# Show labels
kubectl get pods --show-labels
```

### Subtask 5.2: Debugging Commands

**Step 1: Log analysis**
```bash
# Get recent logs
kubectl logs $WEBAPP_POD --tail=20

echo "--- Separator ---"

# Follow logs in real-time (Ctrl+C to stop)
kubectl logs $WEBAPP_POD -f &
LOG_PID=$!

# Generate some traffic
curl http://$(minikube ip):30080 > /dev/null

sleep 5
kill $LOG_PID
```

**Step 2: Pod inspection**
```bash
# Get detailed pod information
kubectl describe pod $WEBAPP_POD

echo "--- Separator ---"

# Check resource usage (requires metrics-server)
kubectl top pods 2>/dev/null || echo "Metrics not available (normal in lab environment)"
```

**Step 3: Interactive debugging**
```bash
# Execute commands in running pod
kubectl exec $WEBAPP_POD -- ps aux
kubectl exec $WEBAPP_POD -- cat /etc/nginx/nginx.conf

echo "--- Separator ---"

# Interactive shell (exit to continue)
kubectl exec -it $WEBAPP_POD -- bash
```

### 🔧 **Challenge 4: Troubleshooting Scenarios**

**Scenario 1: Pod Won't Start**
```bash
# Create a pod with wrong image
cat > faulty-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: faulty-pod
spec:
  containers:
  - name: app
    image: nonexistent/image:latest
EOF

kubectl apply -f faulty-pod.yaml

# Diagnose the issue
kubectl get pods faulty-pod
kubectl describe pod faulty-pod
kubectl events --for pod/faulty-pod
```

**Scenario 2: Service Not Working**
```bash
# Create a service with wrong selector
cat > faulty-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: faulty-service
spec:
  selector:
    app: nonexistent
  ports:
    - port: 80
      targetPort: 80
EOF

kubectl apply -f faulty-service.yaml

# Diagnose
kubectl get endpoints faulty-service
kubectl describe service faulty-service
```

**Fix and verify:**
```bash
kubectl delete service faulty-service
kubectl delete pod faulty-pod
```

---

## Task 6: Cleanup and Verification

### Subtask 6.1: Review What You Built

**Step 1: Examine your complete application**
```bash
kubectl get all -o wide
```

**Step 2: Verify functionality**
```bash
# Test webapp accessibility
curl -s http://$(minikube ip):30080 | grep -o '<title>.*</title>'

# Test Redis connection
kubectl exec $(kubectl get pod -l app=webapp -o jsonpath='{.items[0].metadata.name}') -- nc -z redis-service 6379 && echo "✅ Full stack is working"
```

### 🏆 **Final Assessment: CLI Mastery**

**Test your understanding by completing these tasks:**

1. **Find all pods that are NOT ready:**
```bash
kubectl get pods --field-selector=status.phase!=Running
```

2. **Get the IP addresses of all webapp pods:**
```bash
kubectl get pods -l app=webapp -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.podIP}{"\n"}{end}'
```

3. **Show resource usage of deployments:**
```bash
kubectl get deployments -o custom-columns="NAME:.metadata.name,REPLICAS:.spec.replicas,READY:.status.readyReplicas"
```

4. **List all services and their endpoints:**
```bash
kubectl get services -o wide && echo "--- Endpoints ---" && kubectl get endpoints
```

### Subtask 6.2: Cleanup Resources

**Step 1: Clean up in reverse order**
```bash
# Delete services first
kubectl delete service webapp-nodeport webapp-service redis-service

# Delete deployments (this also deletes pods)
kubectl delete deployment webapp-deployment redis-deployment

# Delete individual pods
kubectl delete pod webapp-frontend webapp-multi broken-webapp

# Verify cleanup
kubectl get all
```

**Expected Output:**
```
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2h
```
*Only the default kubernetes service should remain*

**Step 2: Clean up files**
```bash
cd ~/
rm -rf k8s-webapp-lab
```

**Step 3: Optional - Stop minikube**
```bash
# Stop the cluster (keeps it for later use)
minikube stop

# Or delete completely (removes all data)
# minikube delete
```

---

## Advanced Troubleshooting Guide

### Common Issues and Solutions

**Issue 1: "connection refused" errors**
```bash
# Check if pods are actually running
kubectl get pods -o wide

# Check if services have endpoints
kubectl get endpoints

# Check if ports match
kubectl describe service <service-name>
```

**Issue 2: Pods stuck in "Pending" state**
```bash
# Check resource constraints
kubectl describe pod <pod-name>

# Check node capacity
kubectl describe node minikube

# Look for scheduling events
kubectl get events --sort-by=.metadata.creationTimestamp
```

**Issue 3: Image pull failures**
```bash
# Check exact error
kubectl describe pod <pod-name>

# Verify image name and tag
docker search <image-name>

# Test pull manually
docker pull <image-name:tag>
```

**Issue 4: Minikube networking problems**
```bash
# Restart minikube networking
minikube stop
minikube start

# Check minikube status
minikube status

# Verify docker daemon
docker ps
```

### Emergency Commands

**Reset everything:**
```bash
# Delete all resources in namespace
kubectl delete all --all

# Reset minikube completely  
minikube delete && minikube start --driver=docker
```

**Quick diagnostics:**
```bash
# System overview
kubectl get all -A

# Recent events across cluster
kubectl get events -A --sort-by=.metadata.creationTimestamp

# Node resource usage
kubectl top node

# Check minikube logs
minikube logs
```

---

## Lab Verification Checklist

### ✅ **Core Skills Verification**

Run these commands to verify your mastery:

**1. Cluster Management:**
```bash
kubectl cluster-info
kubectl get nodes -o wide
minikube status
```

**2. Pod Operations:**
```bash
kubectl get pods -A
kubectl describe pod -n kube-system $(kubectl get pods -n kube-system -o name | head -1 | cut -d'/' -f2)
```

**3. Service Discovery:**
```bash
kubectl get services
kubectl get endpoints
nslookup kubernetes.default.svc.cluster.local
```

**4. Resource Management:**
```bash
kubectl get all
kubectl api-resources | head -10
kubectl explain pod.spec.containers
```

**Expected Results:**
- All commands should execute without errors
- You should understand what each output field means
- You can explain the relationship between pods, services, and deployments

---

## Conclusion

### 🎉 **Congratulations! You have successfully completed the Kubernetes Fundamentals Lab.**

### Key Achievements

**Technical Skills Mastered:**
- ✅ **Local Kubernetes Setup**: Deployed minikube with proper configuration
- ✅ **Container Orchestration**: Created and managed pods, deployments, and services  
- ✅ **Multi-Tier Applications**: Built realistic web app with database connectivity
- ✅ **Service Networking**: Implemented ClusterIP and NodePort services for internal/external access
- ✅ **kubectl Proficiency**: Mastered essential commands for cluster interaction
- ✅ **Scaling & Load Balancing**: Demonstrated horizontal pod scaling and traffic distribution
- ✅ **Troubleshooting**: Practiced diagnosing and fixing common Kubernetes issues

**Real-World Applications:**
- **Microservices Architecture**: You now understand how to deploy and connect multiple services
- **High Availability**: You've seen how Kubernetes automatically restarts failed containers
- **Service Discovery**: You've experienced DNS-based service communication
- **Resource Management**: You understand how to set limits and scale applications

### Why This Knowledge Matters

**Industry Relevance:**
- Kubernetes runs 78% of containerized applications in production
- Essential skill for DevOps engineers, cloud architects, and backend developers
- Foundation for cloud-native application development

**Career Impact:**
- Kubernetes skills are among the highest-paying in tech
- Required for modern software deployment and scaling
- Gateway to advanced cloud and containerization technologies

### Next Learning Steps

**Immediate Next Steps:**
1. **Persistent Storage**: Learn about PersistentVolumes and StatefulSets
2. **Configuration Management**: Explore ConfigMaps and Secrets
3. **Ingress Controllers**: Implement advanced routing and load balancing
4. **Helm Package Manager**: Simplify application deployment with charts

**Advanced Topics:**
1. **Kubernetes Security**: RBAC, network policies, pod security standards  
2. **Monitoring & Logging**: Prometheus, Grafana, ELK stack integration
3. **CI/CD Integration**: GitOps workflows with Kubernetes
4. **Multi-Cluster Management**: Federation and cluster mesh technologies

**Production Readiness:**
1. **Managed Kubernetes**: AWS EKS, Google GKE, Azure AKS
2. **Production Patterns**: Blue/green deployments, canary releases
3. **Observability**: Distributed tracing, metrics, and alerting
4. **Cost Optimization**: Resource quotas, vertical pod autoscaling

### Resources for Continued Learning

**Official Documentation:**
- Kubernetes.io tutorials and concepts
- kubectl reference documentation

**Hands-On Practice:**
- Deploy more complex applications (databases, message queues)
- Practice with different service types and networking scenarios
- Experiment with different deployment strategies

**Community:**
- Join Kubernetes Slack community
- Follow CNCF (Cloud Native Computing Foundation) updates
- Participate in local Kubernetes meetups

You now have a solid foundation in Kubernetes and are well-prepared to tackle more advanced container orchestration challenges. The skills you've learned here will serve as the building blocks for modern application deployment and management in production environments.

**Keep practicing, keep learning, and welcome to the world of cloud-native computing!** 🚀
