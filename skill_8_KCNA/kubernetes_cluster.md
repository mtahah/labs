# Kubernetes Cluster Components Lab

## Objectives
By the end of this lab, you will be able to:
- Identify and understand the core components of a Kubernetes cluster
- Use kubectl commands to inspect cluster architecture and component status
- Examine control plane component logs including API Server and etcd
- Create and deploy a Pod while understanding its interaction with control plane and worker nodes
- Analyze the communication flow between Kubernetes components
- Troubleshoot basic cluster component issues using command-line tools

## Prerequisites
Before starting this lab, you should have:
- Basic understanding of containerization concepts (Docker)
- Familiarity with Linux command line operations
- Basic knowledge of YAML file structure
- Understanding of client-server architecture concepts
- Completion of previous Kubernetes fundamentals labs or equivalent knowledge

## Environment Setup

### Step 1: Install Required Packages and Dependencies

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

echo "Learning Tip: Always update your system before installing new software to avoid conflicts"
```

```bash
# Install essential packages
sudo apt install -y curl wget apt-transport-https ca-certificates gnupg lsb-release software-properties-common

echo "Learning Tip: These packages provide secure communication and package management capabilities"
```

### Step 2: Install Docker

```bash
# Add Docker GPG key and repository
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash
# Update package index and install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

echo "Learning Tip: Docker is the container runtime that Kubernetes uses to run containers"
```

```bash
# Configure Docker for current user
sudo usermod -aG docker $USER
sudo systemctl enable docker
sudo systemctl start docker

echo "Learning Tip: Adding user to docker group allows running docker commands without sudo"
```

### Step 3: Install kubectl

```bash
# Download and install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

echo "Learning Tip: kubectl is the command-line tool for interacting with Kubernetes clusters"
```

```bash
# Verify kubectl installation
kubectl version --client

echo "Learning Tip: Always verify installations to ensure tools are working correctly"
```

### Step 4: Install minikube

```bash
# Download and install minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

echo "Learning Tip: minikube creates a local Kubernetes cluster for development and learning"
```

```bash
# Start minikube cluster
minikube start --driver=docker --cpus=2 --memory=4096

echo "Learning Tip: minikube supports various drivers; docker driver runs everything in containers"
```

```bash
# Verify minikube is running
minikube status

echo "Learning Tip: Always check cluster status after starting to ensure all components are ready"
```

## Task 1: Identifying Kubernetes Cluster Components

### Subtask 1.1: Verify Cluster Status and Basic Information

```bash
# Check cluster status
kubectl cluster-info

echo "Learning Tip: cluster-info shows the API server endpoint and core services locations"
```

```bash
# Display detailed cluster information (first 20 lines)
kubectl cluster-info dump | head -20

echo "Learning Tip: cluster-info dump provides comprehensive cluster configuration data"
```

```bash
# List all nodes in the cluster with detailed information
kubectl get nodes -o wide

echo "Learning Tip: -o wide flag shows additional details like IP addresses and kernel versions"
```

```bash
# Get detailed information about the node
kubectl describe nodes

echo "Learning Tip: describe command shows detailed information including resource allocation and conditions"
```

### Subtask 1.2: Explore Control Plane Components

```bash
# List all pods in the kube-system namespace
kubectl get pods -n kube-system

echo "Learning Tip: kube-system namespace contains essential cluster components and system pods"
```

```bash
# Get detailed information about control plane pods
kubectl get pods -n kube-system -o wide

echo "Learning Tip: Control plane pods manage cluster state and make scheduling decisions"
```

```bash
# Identify specific control plane components
kubectl get pods -n kube-system | grep -E "(apiserver|etcd|scheduler|controller)"

echo "Learning Tip: These four components form the core of Kubernetes control plane"
```

```bash
# Show all system components with their status
kubectl get all -n kube-system

echo "Learning Tip: This shows pods, services, deployments, and other resources in kube-system"
```

### Subtask 1.3: Examine API Server Component

```bash
# Find the API Server pod
kubectl get pods -n kube-system | grep apiserver

echo "Learning Tip: API Server is the central component that exposes the Kubernetes API"
```

```bash
# Get detailed information about the API Server
kubectl describe pod -n kube-system $(kubectl get pods -n kube-system | grep apiserver | awk '{print $1}')

echo "Learning Tip: API Server handles all REST operations and serves as the frontend to the cluster's shared state"
```

```bash
# Check API Server service endpoints
kubectl get endpoints -n kube-system

echo "Learning Tip: Endpoints show which pods are backing each service"
```

```bash
# Check API Server configuration and arguments
kubectl get pod -n kube-system $(kubectl get pods -n kube-system | grep apiserver | awk '{print $1}') -o yaml | grep -A 20 "command:"

echo "Learning Tip: API Server configuration includes security, storage, and networking parameters"
```

### Subtask 1.4: Examine etcd Component

```bash
# Find the etcd pod
kubectl get pods -n kube-system | grep etcd

echo "Learning Tip: etcd is the distributed key-value store that holds all cluster data"
```

```bash
# Get detailed information about etcd
kubectl describe pod -n kube-system $(kubectl get pods -n kube-system | grep etcd | awk '{print $1}')

echo "Learning Tip: etcd stores the entire cluster state and is critical for cluster operation"
```

```bash
# Check etcd health (if accessible)
kubectl exec -n kube-system $(kubectl get pods -n kube-system | grep etcd | awk '{print $1}') -- etcdctl endpoint health

echo "Learning Tip: etcd health checks ensure the cluster's data store is functioning properly"
```

```bash
# Check etcd member status
kubectl exec -n kube-system $(kubectl get pods -n kube-system | grep etcd | awk '{print $1}') -- etcdctl member list

echo "Learning Tip: In production, etcd typically runs as a cluster of 3 or 5 members for high availability"
```

## Task 2: Inspecting Control Plane Component Logs

### Subtask 2.1: Examining API Server Logs

```bash
# View recent API Server logs
kubectl logs -n kube-system $(kubectl get pods -n kube-system | grep apiserver | awk '{print $1}') --tail=50

echo "Learning Tip: API Server logs show all requests made to the Kubernetes API"
```

```bash
# Search for specific events in API Server logs
kubectl logs -n kube-system $(kubectl get pods -n kube-system | grep apiserver | awk '{print $1}') | grep -i "error\|warning" | tail -10

echo "Learning Tip: Filtering logs for errors and warnings helps identify potential issues"
```

```bash
# Check API Server metrics and performance logs
kubectl logs -n kube-system $(kubectl get pods -n kube-system | grep apiserver | awk '{print $1}') | grep -i "latency\|duration" | tail -5

echo "Learning Tip: Performance metrics help identify API Server bottlenecks"
```

### Subtask 2.2: Examining etcd Logs

```bash
# View recent etcd logs
kubectl logs -n kube-system $(kubectl get pods -n kube-system | grep etcd | awk '{print $1}') --tail=30

echo "Learning Tip: etcd logs provide insights into cluster state storage and replication"
```

```bash
# Check for etcd health-related messages
kubectl logs -n kube-system $(kubectl get pods -n kube-system | grep etcd | awk '{print $1}') | grep -i "health\|ready" | tail -5

echo "Learning Tip: etcd health messages indicate the stability of your cluster's data store"
```

```bash
# Monitor etcd performance metrics in logs
kubectl logs -n kube-system $(kubectl get pods -n kube-system | grep etcd | awk '{print $1}') | grep -i "slow\|latency" | tail-5

echo "Learning Tip: etcd performance directly impacts cluster responsiveness"
```

### Subtask 2.3: Examining Scheduler and Controller Manager Logs

```bash
# View Scheduler logs
kubectl logs -n kube-system $(kubectl get pods -n kube-system | grep scheduler | awk '{print $1}') --tail=20

echo "Learning Tip: Scheduler decides which node should run each pod based on resource requirements and constraints"
```

```bash
# View Controller Manager logs
kubectl logs -n kube-system $(kubectl get pods -n kube-system | grep controller-manager | awk '{print $1}') --tail=20

echo "Learning Tip: Controller Manager runs controllers that regulate cluster state and respond to cluster events"
```

```bash
# Search for scheduling decisions in logs
kubectl logs -n kube-system $(kubectl get pods -n kube-system | grep scheduler | awk '{print $1}') | grep -i "scheduled\|binding" | tail -5

echo "Learning Tip: Scheduler logs show pod placement decisions and binding operations"
```

## Task 3: Creating a Pod and Understanding Component Interactions

### Subtask 3.1: Create a Simple Pod

```bash
# Create a Pod manifest file using nano
nano nginx-pod.yaml
```

**Content for nginx-pod.yaml (paste this into nano editor):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-demo
  labels:
    app: nginx-demo
    environment: lab
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    env:
    - name: ENVIRONMENT
      value: "lab-demo"
```

**Save and exit nano (Ctrl+O, Enter, Ctrl+X)**

```bash
echo "Learning Tip: YAML manifests define desired state of Kubernetes resources"
```

```bash
# Verify the YAML file was created correctly
cat nginx-pod.yaml

echo "Learning Tip: Always verify your YAML syntax before applying to avoid errors"
```

```bash
# Apply the Pod manifest
kubectl apply -f nginx-pod.yaml

echo "Learning Tip: kubectl apply creates or updates resources based on manifest files"
```

```bash
# Verify Pod creation
kubectl get pods

echo "Learning Tip: Pod status shows the current phase: Pending, Running, Succeeded, Failed, or Unknown"
```

```bash
# Get detailed Pod information
kubectl describe pod nginx-demo

echo "Learning Tip: describe shows events, conditions, and detailed configuration of the pod"
```

```bash
# Watch pod creation in real-time
kubectl get pods -w &
sleep 5
pkill -f "kubectl get pods -w"

echo "Learning Tip: -w flag watches for changes and updates output in real-time"
```

### Subtask 3.2: Monitor Component Interactions During Pod Creation

```bash
# Check scheduler logs for Pod scheduling decisions
kubectl logs -n kube-system $(kubectl get pods -n kube-system | grep scheduler | awk '{print $1}') | grep nginx-demo

echo "Learning Tip: Scheduler evaluates nodes and selects the best fit for pod placement"
```

```bash
# Check API Server logs for Pod-related API calls
kubectl logs -n kube-system $(kubectl get pods -n kube-system | grep apiserver | awk '{print $1}') | grep nginx-demo | tail-10

echo "Learning Tip: API Server processes all pod-related requests and updates cluster state"
```

```bash
# Check controller manager logs for pod management
kubectl logs -n kube-system $(kubectl get pods -n kube-system | grep controller-manager | awk '{print $1}') | grep nginx-demo | tail-5

echo "Learning Tip: Controllers ensure desired state matches actual state"
```

### Subtask 3.3: Analyze Pod Lifecycle and Component Communication

```bash
# Check Pod events to understand the creation process
kubectl get events --sort-by=.metadata.creationTimestamp | grep nginx-demo

echo "Learning Tip: Events provide a timeline of operations performed on cluster resources"
```

```bash
# Monitor Pod status changes
kubectl get pod nginx-demo -o yaml | grep -A 10 "status:"

echo "Learning Tip: Pod status includes phase, conditions, container statuses, and networking info"
```

```bash
# Verify container runtime interaction
kubectl get pod nginx-demo -o jsonpath='{.status.containerStatuses[0].containerID}'

echo "Learning Tip: Container ID shows successful interaction with container runtime (Docker)"
```

```bash
# Check pod resource usage
kubectl top pod nginx-demo

echo "Learning Tip: Resource monitoring helps ensure pods stay within defined limits"
```

### Subtask 3.4: Test Pod Functionality and Network Communication

```bash
# Execute commands inside the Pod
kubectl exec nginx-demo -- nginx -v

echo "Learning Tip: kubectl exec allows running commands inside running containers"
```

```bash
# Check Pod IP and network configuration
kubectl get pod nginx-demo -o wide

echo "Learning Tip: Each pod gets its own IP address within the cluster network"
```

```bash
# Test network connectivity to the Pod
POD_IP=$(kubectl get pod nginx-demo -o jsonpath='{.status.podIP}')
curl -I http://$POD_IP

echo "Learning Tip: Pod IPs are only accessible from within the cluster network"
```

```bash
# Port-forward to test application accessibility
kubectl port-forward nginx-demo 8080:80 &
sleep 2
curl http://localhost:8080
pkill -f "kubectl port-forward"

echo "Learning Tip: Port-forwarding creates a tunnel from local machine to pod"
```

```bash
# Test pod environment variables
kubectl exec nginx-demo -- env | grep ENVIRONMENT

echo "Learning Tip: Environment variables can be used to configure applications in pods"
```

## Task 4: Advanced Component Analysis

### Subtask 4.1: Examine Resource Usage and Metrics

```bash
# Check node resource usage
kubectl top nodes

echo "Learning Tip: Node metrics show CPU and memory utilization across cluster nodes"
```

```bash
# Check Pod resource usage
kubectl top pods

echo "Learning Tip: Pod metrics help identify resource-hungry applications"
```

```bash
# Get detailed resource information
kubectl describe node | grep -A 5 "Allocated resources"

echo "Learning Tip: Resource allocation shows how much capacity is reserved vs available"
```

```bash
# Check cluster resource quotas (if any)
kubectl get resourcequotas --all-namespaces

echo "Learning Tip: Resource quotas limit resource consumption within namespaces"
```

### Subtask 4.2: Understand Component Dependencies

```bash
# Check service accounts and RBAC
kubectl get serviceaccounts -n kube-system

echo "Learning Tip: Service accounts provide identity for processes running in pods"
```

```bash
# Examine cluster roles and bindings
kubectl get clusterroles | head -10
kubectl get clusterrolebindings | head -10

echo "Learning Tip: RBAC controls what operations users and services can perform in the cluster"
```

```bash
# Check component health endpoints
kubectl get componentstatuses

echo "Learning Tip: Component status shows health of controller-manager, scheduler, and etcd"
```

```bash
# View cluster configuration
kubectl config view

echo "Learning Tip: Config shows cluster connection info and authentication details"
```

### Subtask 4.3: Network and Service Discovery

```bash
# Check DNS configuration
kubectl get pods -n kube-system | grep dns

echo "Learning Tip: CoreDNS provides service discovery within the cluster"
```

```bash
# Test DNS resolution from pod
kubectl exec nginx-demo -- nslookup kubernetes.default

echo "Learning Tip: Kubernetes DNS allows pods to discover services by name"
```

```bash
# Check cluster networking
kubectl get pods -n kube-system | grep -E "(proxy|network|cni)"

echo "Learning Tip: kube-proxy and CNI plugins handle cluster networking"
```

## Troubleshooting Common Issues

### Issue 1: Pod Stuck in Pending State

```bash
# If your Pod remains in Pending state, check Pod events
kubectl describe pod nginx-demo | grep -A 10 Events

echo "Learning Tip: Pending pods often have resource constraints or node selector issues"
```

```bash
# Verify node resources
kubectl describe nodes | grep -A 5 "Allocated resources"

echo "Learning Tip: Insufficient node resources prevent pod scheduling"
```

```bash
# Check scheduler logs
kubectl logs -n kube-system $(kubectl get pods -n kube-system | grep scheduler | awk '{print $1}') | tail-20

echo "Learning Tip: Scheduler logs explain why pods cannot be placed"
```

### Issue 2: Cannot Access Control Plane Components

```bash
# If you cannot access control plane logs, verify cluster status
kubectl cluster-info

echo "Learning Tip: Inaccessible control plane usually indicates cluster connectivity issues"
```

```bash
# Check if components are running
kubectl get pods -n kube-system

echo "Learning Tip: All control plane pods should be in Running status"
```

```bash
# Restart minikube if necessary
# minikube stop
# minikube start

echo "Learning Tip: Sometimes a cluster restart resolves temporary issues"
```

### Issue 3: Network Connectivity Issues

```bash
# If Pod networking doesn't work, check Pod network configuration
kubectl get pod nginx-demo -o yaml | grep -A 5 "podIP"

echo "Learning Tip: Pods without IP addresses indicate networking problems"
```

```bash
# Verify DNS resolution
kubectl exec nginx-demo -- nslookup kubernetes.default

echo "Learning Tip: DNS issues prevent service discovery within the cluster"
```

```bash
# Check network policies (if any)
kubectl get networkpolicies --all-namespaces

echo "Learning Tip: Network policies can block pod-to-pod communication"
```

## Additional Learning Commands

### Monitoring and Observability

```bash
# Watch all pods across all namespaces
kubectl get pods --all-namespaces -w &
sleep 10
pkill -f "kubectl get pods"

echo "Learning Tip: Monitoring all namespaces helps understand cluster-wide activity"
```

```bash
# Check cluster events across all namespaces
kubectl get events --all-namespaces --sort-by=.metadata.creationTimestamp | tail-20

echo "Learning Tip: Cluster-wide events help identify system-level issues"
```

```bash
# Examine node conditions and capacity
kubectl describe nodes | grep -A 10 -E "(Conditions|Capacity|Allocatable)"

echo "Learning Tip: Node conditions indicate health status like disk pressure or memory pressure"
```

### Security and Configuration

```bash
# Check pod security contexts
kubectl get pod nginx-demo -o yaml | grep -A 10 "securityContext"

echo "Learning Tip: Security contexts control privilege and access for pods and containers"
```

```bash
# View cluster API versions
kubectl api-versions

echo "Learning Tip: API versions show which Kubernetes features are available in your cluster"
```

```bash
# Check admission controllers
kubectl exec -n kube-system $(kubectl get pods -n kube-system | grep apiserver | awk '{print $1}') -- kube-apiserver --help | grep -A 5 "admission"

echo "Learning Tip: Admission controllers modify or validate objects before they're stored"
```

## Cleanup

```bash
# Remove the resources created during this lab
kubectl delete pod nginx-demo

echo "Learning Tip: Always clean up resources after labs to maintain a tidy cluster"
```

```bash
# Remove the YAML file
rm nginx-pod.yaml

echo "Learning Tip: Cleaning up files prevents confusion in future labs"
```

```bash
# Optional: Check that pod is fully deleted
kubectl get pods

echo "Learning Tip: Verification ensures resources are properly cleaned up"
```

```bash
# Optional: Stop minikube if you're done with the lab
# minikube stop

echo "Learning Tip: Stopping minikube saves system resources when not needed"
```

## Conclusion

In this lab, you have successfully:

- **Identified Kubernetes cluster components** using kubectl commands and learned how to inspect their status and configuration
- **Examined control plane component logs** including API Server and etcd, understanding how to troubleshoot cluster issues through log analysis  
- **Created a Pod and analyzed its interaction** with control plane and worker nodes, observing the complete lifecycle from creation to running state
- **Understood component communication flow** and how different parts of Kubernetes work together to manage containerized applications
- **Performed advanced monitoring and troubleshooting** using various kubectl commands and techniques

### Why This Matters

Understanding Kubernetes architecture is crucial for:

- **Troubleshooting cluster issues effectively** by knowing which component logs to check
- **Optimizing cluster performance** by understanding resource allocation and component interactions  
- **Securing your cluster** by knowing the role of each component and their communication patterns
- **Preparing for KCNA certification** by demonstrating practical knowledge of Kubernetes internals

### Key Components You've Learned About

1. **API Server** - Central management component that exposes Kubernetes API
2. **etcd** - Distributed key-value store for all cluster data
3. **Scheduler** - Assigns pods to nodes based on resource requirements
4. **Controller Manager** - Runs controllers that regulate cluster state
5. **kubelet** - Node agent that manages pod lifecycle
6. **kube-proxy** - Network proxy that maintains network rules

### Next Steps

- Explore Kubernetes networking concepts in depth
- Learn about advanced scheduling and resource management
- Study Kubernetes security best practices
- Practice troubleshooting scenarios in different cluster configurations

Remember: The more you practice with these components, the better you'll understand how Kubernetes orchestrates containerized applications!
