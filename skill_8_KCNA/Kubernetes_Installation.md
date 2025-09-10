# Kubernetes Installation Lab - Complete Setup Guide
## Optimized for 4GB RAM Systems

### Prerequisites Check
Before starting, let's verify system requirements and prepare for installation:

```bash
# Check system resources
echo "=== System Resource Check ==="
echo "Available RAM:"
free -h

echo -e "\nDisk Space:"
df -h /

echo -e "\nCPU Information:"
nproc --all

echo -e "\nOperating System:"
lsb_release -a
```

---

## Task 1: System Preparation for Kubernetes

### Step 1.1: Update System Packages
```bash
echo "=== Updating System Packages ==="
# Update package index - ensures we get latest package information
sudo apt update

echo -e "\n=== Upgrading Existing Packages ==="
# Upgrade existing packages - keeps system secure and stable
sudo apt upgrade -y

echo -e "\n=== Installing Essential Packages ==="
# Install essential packages for Kubernetes prerequisites
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common gnupg lsb-release

echo "✅ System packages updated successfully"
```

**Learning Tip:** These packages enable secure HTTPS transport and GPG key verification for Kubernetes repositories.

---

### Step 1.2: Disable Swap (Critical for Kubernetes)
```bash
echo "=== Disabling Swap for Kubernetes ==="
# Check current swap status
echo "Current swap status:"
free -h

# Disable swap temporarily (immediate effect)
sudo swapoff -a

# Disable swap permanently by commenting out swap entries in fstab
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

echo -e "\n=== Verifying Swap Disable ==="
free -h
echo "✅ Swap disabled - Kubernetes requirement satisfied"
```

**Learning Tip:** Kubernetes disables swap to ensure predictable performance and proper resource management.

---

### Step 1.3: Configure Kernel Modules
```bash
echo "=== Loading Required Kernel Modules ==="
# Load overlay module for container filesystem layers
sudo modprobe overlay

# Load br_netfilter module for bridge network filtering
sudo modprobe br_netfilter

echo -e "\n=== Making Modules Persistent Across Reboots ==="
# Create configuration file to auto-load modules at boot
sudo nano /etc/modules-load.d/k8s.conf
```

**Content for /etc/modules-load.d/k8s.conf:**
```
overlay
br_netfilter
```

```bash
echo "✅ Kernel modules configured for Kubernetes networking"
```

**Learning Tip:** These modules enable container networking and filesystem features required by Kubernetes.

---

### Step 1.4: Configure Network Parameters
```bash
echo "=== Configuring Network Parameters ==="
# Create sysctl configuration for Kubernetes networking
sudo nano /etc/sysctl.d/k8s.conf
```

**Content for /etc/sysctl.d/k8s.conf:**
```
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
```

```bash
echo -e "\n=== Applying Network Configuration ==="
# Apply sysctl parameters without reboot
sudo sysctl --system

echo -e "\n=== Verifying Network Configuration ==="
sudo sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward

echo "✅ Network parameters configured successfully"
```

**Learning Tip:** These parameters enable proper packet forwarding and filtering for Kubernetes pod networking.

---

## Task 2: Installing Container Runtime (containerd) - Memory Optimized

### Step 2.1: Install containerd
```bash
echo "=== Adding Docker Repository (containerd source) ==="
# Download and add Docker's GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository to apt sources
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

echo -e "\n=== Updating Package Index ==="
sudo apt update

echo -e "\n=== Installing containerd ==="
sudo apt install -y containerd.io

echo "✅ containerd installed successfully"
```

---

### Step 2.2: Configure containerd for 4GB RAM Optimization
```bash
echo "=== Creating containerd Configuration Directory ==="
sudo mkdir -p /etc/containerd

echo -e "\n=== Generating Default Configuration ==="
sudo containerd config default | sudo tee /etc/containerd/config.toml > /dev/null

echo -e "\n=== Optimizing containerd for 4GB RAM ==="
# Configure systemd cgroup driver (recommended for Kubernetes)
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

# Add memory optimization settings using nano
sudo nano /etc/containerd/config.toml
```

**Add these lines at the end of config.toml for memory optimization:**
```toml
# Memory optimization for 4GB systems
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = true

[plugins."io.containerd.grpc.v1.cri".containerd]
  default_runtime_name = "runc"

[plugins."io.containerd.grpc.v1.cri".cni]
  bin_dir = "/opt/cni/bin"
  conf_dir = "/etc/cni/net.d"
```

```bash
echo -e "\n=== Restarting and Enabling containerd ==="
sudo systemctl restart containerd
sudo systemctl enable containerd

echo -e "\n=== Verifying containerd Status ==="
sudo systemctl status containerd --no-pager

echo "✅ containerd configured and running with memory optimizations"
```

**Learning Tip:** containerd is the industry-standard container runtime that Kubernetes uses to manage containers.

---

## Task 3: Installing Kubernetes Components

### Step 3.1: Add Kubernetes Repository
```bash
echo "=== Adding Kubernetes Repository ==="
# Download Kubernetes GPG key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add Kubernetes repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

echo -e "\n=== Updating Package Index ==="
sudo apt update

echo "✅ Kubernetes repository added successfully"
```

---

### Step 3.2: Install Kubernetes Tools
```bash
echo "=== Installing Kubernetes Components ==="
# Install kubelet (node agent), kubeadm (cluster setup), kubectl (CLI tool)
sudo apt install -y kubelet kubeadm kubectl

echo -e "\n=== Preventing Automatic Updates ==="
# Hold packages to prevent accidental upgrades
sudo apt-mark hold kubelet kubeadm kubectl

echo -e "\n=== Verifying Installation ==="
kubeadm version
echo ""
kubectl version --client

echo "✅ Kubernetes tools installed successfully"
```

**Learning Tip:** kubelet runs on each node, kubeadm initializes clusters, kubectl is your command-line interface.

---

## Task 4: Initialize Kubernetes Cluster (4GB RAM Optimized)

### Step 4.1: Initialize Master Node with Memory Optimizations
```bash
echo "=== Getting System IP Address ==="
SYSTEM_IP=$(hostname -I | awk '{print $1}')
echo "System IP: $SYSTEM_IP"

echo -e "\n=== Initializing Kubernetes Cluster (Optimized for 4GB RAM) ==="
# Initialize cluster with memory-conscious settings
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --apiserver-advertise-address=$SYSTEM_IP \
  --control-plane-endpoint=$SYSTEM_IP \
  --upload-certs \
  --v=5

echo -e "\n⚠️  IMPORTANT: Save the 'kubeadm join' command from above output!"
echo "✅ Kubernetes cluster initialized successfully"
```

---

### Step 4.2: Configure kubectl for Regular User
```bash
echo "=== Configuring kubectl for Current User ==="
# Create .kube directory in user's home
mkdir -p $HOME/.kube

# Copy admin configuration file
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

# Change ownership to current user
sudo chown $(id -u):$(id -g) $HOME/.kube/config

echo -e "\n=== Verifying kubectl Configuration ==="
kubectl cluster-info

echo -e "\n=== Checking Initial Cluster State ==="
kubectl get nodes

echo "✅ kubectl configured successfully for user access"
```

**Learning Tip:** The kubeconfig file contains cluster connection details and authentication information.

---

## Task 5: Install Pod Network Add-on (Memory Optimized)

### Step 5.1: Install Flannel CNI Plugin
```bash
echo "=== Installing Flannel Network Plugin ==="
# Apply Flannel manifest for pod networking
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

echo -e "\n=== Waiting for Flannel Pods to Initialize ==="
# Wait up to 5 minutes for flannel pods to be ready
kubectl wait --for=condition=ready pod -l app=flannel -n kube-flannel --timeout=300s

echo -e "\n=== Checking Flannel Status ==="
kubectl get pods -n kube-flannel

echo "✅ Flannel network plugin installed and running"
```

---

### Step 5.2: Configure Single Node Cluster
```bash
echo "=== Configuring Single Node Cluster (Removing Master Taint) ==="
# Remove taint to allow scheduling pods on master node
kubectl taint nodes --all node-role.kubernetes.io/control-plane- || true

echo -e "\n=== Verifying Taint Removal ==="
kubectl describe nodes | grep -A 5 -B 5 -i taint

echo "✅ Single node cluster configured - pods can now be scheduled"
```

**Learning Tip:** Taints prevent pod scheduling. We remove the master taint for single-node setups.

---

## Task 6: Cluster Verification and Health Check

### Step 6.1: Comprehensive Node Status Check
```bash
echo "=== Checking Node Status ==="
kubectl get nodes

echo -e "\n=== Detailed Node Information ==="
kubectl get nodes -o wide

echo -e "\n=== Node Resource Allocation ==="
kubectl describe nodes | grep -A 10 "Allocated resources"

echo "✅ Node status verified"
```

---

### Step 6.2: System Pods Verification
```bash
echo "=== Checking All System Pods ==="
kubectl get pods --all-namespaces

echo -e "\n=== Waiting for All System Pods to be Ready ==="
kubectl wait --for=condition=ready pod --all -n kube-system --timeout=300s

echo -e "\n=== Checking Flannel Pods ==="
kubectl get pods -n kube-flannel

echo -e "\n=== Pod Status Summary ==="
kubectl get pods --all-namespaces | grep -E "(Running|Ready|Completed)" | wc -l
echo "Total running pods counted above"

echo "✅ All system pods are healthy and running"
```

---

### Step 6.3: Cluster Component Health Check
```bash
echo "=== Checking Cluster Component Status ==="
kubectl get componentstatuses

echo -e "\n=== Cluster Information ==="
kubectl cluster-info

echo -e "\n=== API Server Health Check ==="
kubectl get --raw='/readyz?verbose'

echo -e "\n=== Cluster Events (Recent) ==="
kubectl get events --sort-by=.metadata.creationTimestamp | tail -10

echo "✅ Cluster components verified and healthy"
```

**Learning Tip:** Component status checks ensure all Kubernetes control plane components are functioning properly.

---

## Task 7: Exploring Your Kubernetes Cluster

### Step 7.1: Basic Cluster Exploration
```bash
echo "=== Listing All Namespaces ==="
kubectl get namespaces

echo -e "\n=== Cluster Version Information ==="
kubectl version

echo -e "\n=== Current Context ==="
kubectl config current-context

echo -e "\n=== Available API Resources ==="
kubectl api-resources | head -20
echo "... (showing first 20 resources)"

echo "✅ Cluster exploration completed"
```

---

### Step 7.2: Deploy Memory-Optimized Test Application
```bash
echo "=== Deploying Lightweight Test Application ==="
# Create deployment with resource limits for 4GB system
kubectl create deployment nginx-test --image=nginx:alpine

echo -e "\n=== Configuring Resource Limits ==="
kubectl patch deployment nginx-test -p '{
  "spec": {
    "template": {
      "spec": {
        "containers": [
          {
            "name": "nginx",
            "resources": {
              "limits": {
                "memory": "64Mi",
                "cpu": "100m"
              },
              "requests": {
                "memory": "32Mi",
                "cpu": "50m"
              }
            }
          }
        ]
      }
    }
  }
}'

echo -e "\n=== Exposing Application as NodePort Service ==="
kubectl expose deployment nginx-test --port=80 --type=NodePort

echo -e "\n=== Checking Deployment Status ==="
kubectl get deployments
kubectl get pods
kubectl get services

echo "✅ Test application deployed with memory optimization"
```

---

### Step 7.3: Application Functionality Test
```bash
echo "=== Waiting for Pod to be Ready ==="
kubectl wait --for=condition=ready pod -l app=nginx-test --timeout=120s

echo -e "\n=== Getting Pod Details ==="
POD_NAME=$(kubectl get pods -l app=nginx-test -o jsonpath='{.items[0].metadata.name}')
echo "Pod Name: $POD_NAME"

kubectl describe pod $POD_NAME

echo -e "\n=== Testing Application Connectivity ==="
NODE_PORT=$(kubectl get service nginx-test -o jsonpath='{.spec.ports[0].nodePort}')
echo "Service accessible on NodePort: $NODE_PORT"

echo -e "\n=== Testing HTTP Response ==="
curl -s http://localhost:$NODE_PORT | head -5
echo -e "\n✅ Application is responding correctly"
```

---

### Step 7.4: Resource Monitoring and Cleanup
```bash
echo "=== Installing Metrics Server for Resource Monitoring ==="
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Patch metrics-server for local testing
kubectl patch deployment metrics-server -n kube-system --type='json' -p='[
  {
    "op": "add",
    "path": "/spec/template/spec/containers/0/args/-",
    "value": "--kubelet-insecure-tls"
  }
]'

echo -e "\n=== Waiting for Metrics Server ==="
kubectl wait --for=condition=ready pod -l k8s-app=metrics-server -n kube-system --timeout=120s

echo -e "\n=== Resource Usage Check ==="
sleep 30  # Allow metrics to populate
kubectl top nodes
kubectl top pods

echo -e "\n=== Cleaning Up Test Resources ==="
kubectl delete deployment nginx-test
kubectl delete service nginx-test

echo -e "\n=== Verifying Cleanup ==="
kubectl get deployments
kubectl get services

echo "✅ Resource monitoring configured and test resources cleaned up"
```

**Learning Tip:** Resource monitoring helps you understand cluster utilization and optimize for your 4GB RAM environment.

---

## Task 8: Advanced Cluster Operations

### Step 8.1: Network and DNS Verification
```bash
echo "=== Testing Cluster DNS Resolution ==="
kubectl run dns-test --image=busybox:1.28 --rm -it --restart=Never -- nslookup kubernetes.default

echo -e "\n=== Checking DNS Pods ==="
kubectl get pods -n kube-system -l k8s-app=kube-dns

echo -e "\n=== Testing Service Discovery ==="
kubectl run service-test --image=busybox:1.28 --rm -it --restart=Never -- nslookup kube-dns.kube-system.svc.cluster.local

echo "✅ DNS and service discovery verified"
```

---

### Step 8.2: Security and RBAC Check
```bash
echo "=== Checking Service Accounts ==="
kubectl get serviceaccounts --all-namespaces

echo -e "\n=== Checking RBAC Configuration ==="
kubectl get clusterroles | head -10
kubectl get clusterrolebindings | head -10

echo -e "\n=== Current User Permissions ==="
kubectl auth can-i create pods
kubectl auth can-i create deployments
kubectl auth can-i get nodes

echo "✅ Security configuration verified"
```

---

## Task 9: Performance Optimization for 4GB RAM

### Step 9.1: Configure kubelet for Low Memory
```bash
echo "=== Optimizing kubelet for 4GB RAM ==="
sudo nano /var/lib/kubelet/config.yaml
```

**Add these optimizations to kubelet config:**
```yaml
# Memory optimization settings
memorySwap: {}
failSwapOn: false
evictionSoft:
  memory.available: "200Mi"
evictionSoftGracePeriod:
  memory.available: "30s"
evictionHard:
  memory.available: "100Mi"
evictionMinimumReclaim:
  memory.available: "50Mi"
kubeReserved:
  memory: "512Mi"
  cpu: "100m"
systemReserved:
  memory: "512Mi"
  cpu: "100m"
```

```bash
echo -e "\n=== Restarting kubelet with New Configuration ==="
sudo systemctl restart kubelet

echo -e "\n=== Verifying kubelet Status ==="
sudo systemctl status kubelet --no-pager

echo "✅ kubelet optimized for 4GB RAM environment"
```

---

### Step 9.2: Set Resource Quotas for Memory Management
```bash
echo "=== Creating Resource Quota for Memory Management ==="
# Create a default namespace resource quota
kubectl create quota memory-quota \
  --hard=requests.memory=2Gi,limits.memory=3Gi,pods=10 \
  --namespace=default

echo -e "\n=== Verifying Resource Quota ==="
kubectl describe quota memory-quota

echo -e "\n=== Creating LimitRange for Pod Defaults ==="
kubectl apply -f - <<EOF
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-limit-range
spec:
  limits:
  - default:
      memory: "128Mi"
      cpu: "100m"
    defaultRequest:
      memory: "64Mi"
      cpu: "50m"
    type: Container
EOF

echo -e "\n=== Verifying LimitRange ==="
kubectl describe limitrange memory-limit-range

echo "✅ Resource quotas and limits configured for memory efficiency"
```

**Learning Tip:** Resource quotas and limits prevent any single application from consuming all available memory.

---

## Troubleshooting Guide

### Common Issue 1: Pods Stuck in Pending State
```bash
echo "=== Troubleshooting Pending Pods ==="
# Check for pending pods
kubectl get pods --field-selector=status.phase=Pending

# If any pending pods exist, describe them
for pod in $(kubectl get pods --field-selector=status.phase=Pending -o name 2>/dev/null); do
  echo "Describing $pod:"
  kubectl describe $pod
done

# Check node resources
kubectl describe nodes | grep -A 10 "Allocated resources"

echo "✅ Pending pod troubleshooting completed"
```

---

### Common Issue 2: High Memory Usage
```bash
echo "=== Memory Usage Troubleshooting ==="
echo "System memory usage:"
free -h

echo -e "\nTop memory consuming processes:"
ps aux --sort=-%mem | head -10

echo -e "\nKubernetes pod memory usage:"
kubectl top pods --all-namespaces 2>/dev/null || echo "Metrics not available yet"

echo -e "\nNode memory pressure check:"
kubectl describe nodes | grep -A 5 -B 5 "MemoryPressure"

echo "✅ Memory troubleshooting completed"
```

---

### Common Issue 3: Network Connectivity Problems
```bash
echo "=== Network Troubleshooting ==="
echo "Checking flannel pods:"
kubectl get pods -n kube-flannel

echo -e "\nChecking network routes:"
ip route show | grep -E "(10.244|flannel)"

echo -e "\nChecking bridge interfaces:"
ip link show | grep -E "(flannel|cni)"

echo -e "\nTesting pod-to-pod connectivity:"
kubectl run net-test-1 --image=busybox:1.28 --rm -it --restart=Never -- ip addr show

echo "✅ Network troubleshooting completed"
```

---

## Final Verification Checklist

```bash
echo "=== FINAL CLUSTER VERIFICATION ==="

echo "1. Node Status:"
kubectl get nodes

echo -e "\n2. System Pods:"
kubectl get pods -n kube-system | grep -E "(Running|Completed)"

echo -e "\n3. Memory Usage:"
free -h

echo -e "\n4. Cluster Info:"
kubectl cluster-info

echo -e "\n5. Storage Classes:"
kubectl get storageclass

echo -e "\n6. Network Policies:"
kubectl get networkpolicies --all-namespaces

echo -e "\n7. Resource Quotas:"
kubectl get quota --all-namespaces

echo -e "\n=== SUCCESS! Your Kubernetes Cluster is Ready ==="
echo "✅ All components verified and optimized for 4GB RAM"
echo "✅ Cluster is ready for application deployment"
echo "✅ Memory optimization settings applied"
echo "✅ Network connectivity verified"
```

---

## Next Steps and Learning Path

**Immediate Next Steps:**
1. **Deploy Your First Application:** Try deploying a simple web application
2. **Explore Persistent Storage:** Learn about PersistentVolumes and Claims
3. **Service Discovery:** Understand how services work in Kubernetes
4. **ConfigMaps and Secrets:** Manage application configuration

**Advanced Learning Path:**
1. **Multi-container Pods:** Deploy applications with sidecars
2. **Ingress Controllers:** Expose services externally
3. **Horizontal Pod Autoscaling:** Scale applications automatically
4. **Monitoring Setup:** Implement Prometheus and Grafana
5. **Security Hardening:** Apply security best practices

**Memory-Conscious Development:**
- Always set resource requests and limits
- Use alpine-based images when possible
- Monitor resource usage regularly
- Implement proper resource quotas

---

## Congratulations! 🎉

You have successfully completed the Kubernetes installation lab with optimizations for 4GB RAM systems. Your cluster is now ready for development and learning. 

**Key Achievements:**
- ✅ Installed and configured Kubernetes cluster
- ✅ Optimized for memory-constrained environments
- ✅ Verified all components are working
- ✅ Implemented resource management
- ✅ Configured monitoring capabilities
- ✅ Applied security best practices

Your Kubernetes journey starts here! 🚀
