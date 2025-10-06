# 🖥️ Kubernetes Data Plane & Node Architecture

> **Expert-Level Technical Reference**  
> Deep dive into node components, container runtime architecture, resource management, and workload execution in Kubernetes.

---

## 📋 Table of Contents

1. [Node Architecture Overview](#node-architecture-overview)
2. [kubelet - The Node Agent](#kubelet-node-agent)
3. [Container Runtime Interface (CRI)](#container-runtime-interface)
4. [kube-proxy - Service Networking](#kube-proxy-service-networking)
5. [Resource Management](#resource-management)
6. [Pod Lifecycle Management](#pod-lifecycle-management)
7. [Node Failure & Recovery](#node-failure-recovery)

---

## Node Architecture Overview

### **Components Running on Every Node**

```
┌────────────────────── Kubernetes Node ─────────────────────┐
│                                                             │
│  ┌────────────────────────────────────────────────────┐   │
│  │ kubelet (Primary Node Agent)                       │   │
│  │ - Pod lifecycle management                         │   │
│  │ - Container health monitoring                      │   │
│  │ - Resource enforcement (cgroups)                   │   │
│  │ - Volume mounting                                  │   │
│  │ - Garbage collection                               │   │
│  └─────────────┬──────────────────────────────────────┘   │
│                │                                            │
│                ▼                                            │
│  ┌────────────────────────────────────────────────────┐   │
│  │ Container Runtime (containerd/CRI-O)               │   │
│  │ - Image management (pull, cache, cleanup)          │   │
│  │ - Container lifecycle (create, start, stop, delete)│   │
│  │ - Resource isolation (cgroups, namespaces)         │   │
│  └─────────────┬──────────────────────────────────────┘   │
│                │                                            │
│                ▼                                            │
│  ┌────────────────────────────────────────────────────┐   │
│  │ Linux Kernel                                       │   │
│  │ - cgroups (CPU, memory, I/O limits)                │   │
│  │ - namespaces (process, network, mount isolation)   │   │
│  │ - iptables/nftables (packet filtering)             │   │
│  │ - Network bridges/OVS (packet forwarding)          │   │
│  └────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌────────────────────────────────────────────────────┐   │
│  │ kube-proxy (Network Proxy)                         │   │
│  │ - Service discovery and load balancing             │   │
│  │ - iptables/IPVS rule management                    │   │
│  │ - ClusterIP implementation                         │   │
│  └────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌────────────────────────────────────────────────────┐   │
│  │ CNI Plugin (Container Network Interface)           │   │
│  │ - Pod IP allocation                                │   │
│  │ - Network namespace setup                          │   │
│  │ - Route configuration                              │   │
│  └────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌────────────────────────────────────────────────────┐   │
│  │ CSI Plugin (Container Storage Interface)           │   │
│  │ - Volume provisioning                              │   │
│  │ - Volume attachment                                │   │
│  │ - Filesystem mounting                              │   │
│  └────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### **Data Plane vs Control Plane Independence**

```
Key Architectural Principle:
Data plane operates independently of control plane

Scenario: Control Plane Failure
T+0s:   All masters crash (API Server, etcd, scheduler unavailable)
T+10s:  kubelet cannot reach API Server
        - Running pods continue executing
        - Containers restart per restartPolicy
        - Health probes continue
        - Network still functional
        
What Stops Working:
- New pod scheduling
- Pod deletion/creation
- Configuration updates
- Scaling operations

What Keeps Working:
- All running pods
- Service networking (kube-proxy rules cached)
- Pod-to-pod communication
- Liveness/readiness probes
- Container restarts

Result: Application availability maintained during control plane outage
```

---

## kubelet - The Node Agent

### **Core Responsibilities**

kubelet is the primary agent running on each node, responsible for ensuring containers are running according to PodSpec.

**Architecture:**

```
kubelet Main Loop (every 10 seconds):

1. Sync Loop:
   - Watch API Server for pod assignments (nodeName == this-node)
   - Compare desired pods vs actual containers
   - Take corrective actions

2. Pod Lifecycle:
   - Create/delete pod sandboxes
   - Pull container images
   - Mount volumes
   - Start/stop containers
   
3. Status Updates:
   - Execute health probes
   - Report pod status to API Server
   - Update node conditions
   
4. Resource Management:
   - Enforce resource limits (cgroups)
   - Evict pods under resource pressure
   - Garbage collect unused images/containers

5. Volume Management:
   - Mount/unmount volumes
   - Coordinate with CSI drivers
```

### **Pod Creation Workflow**

```
Detailed Timeline:

T+0s:  Scheduler assigns pod to this node
       Pod object updated: spec.nodeName = "node-5"

T+1s:  kubelet's watch mechanism receives pod event
       Event type: ADDED
       Pod: myapp-abc123

T+2s:  Validate pod specification
       - Check resource requirements
       - Verify image pull secrets exist
       - Validate volume references

T+3s:  Create pod sandbox (pause container)
       Call CRI: RunPodSandbox()
       - Create network namespace
       - Set up cgroups
       - Configure pod-level resources

T+4s:  CNI plugin configures networking
       - Allocate pod IP: 10.244.5.8
       - Create veth pair
       - Set up routes

T+5s:  Mount volumes
       - emptyDir: Create tmpfs
       - ConfigMap: Mount from API Server
       - Secret: Mount encrypted data
       - PersistentVolume: Call CSI driver

T+10s: Pull container images (parallel for multiple containers)
       Image: myapp:v1.2.3
       - Check local cache
       - Pull from registry if missing
       - Verify image digest

T+40s: Create init containers (sequential)
       InitContainer 1: database-migration
       - Start container
       - Wait for exit code 0
       - Logs available via kubectl logs

T+60s: InitContainer 1 completes successfully

T+61s: Create application containers (parallel)
       Container 1: app
       Container 2: sidecar-proxy
       
       For each container:
       - Create container config
       - Apply security context
       - Set resource limits (cgroups)
       - Mount volumes into container
       - Start container process

T+65s: Configure health probes
       - Startup probe: Check every 5s
       - Liveness probe: Check every 10s  
       - Readiness probe: Check every 5s

T+75s: Startup probe succeeds
       Enable liveness/readiness probes

T+76s: Readiness probe succeeds
       Pod ready for traffic

T+77s: Update pod status to API Server
       status.phase: Running
       status.conditions:
       - type: PodScheduled (True)
       - type: Initialized (True)
       - type: ContainersReady (True)
       - type: Ready (True)

Total Time: 77 seconds (typical for new image pull)
Warm Start: 15-20 seconds (image cached)
```

### **Image Management**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:v1.2.3
    imagePullPolicy: IfNotPresent  # Always, IfNotPresent, Never
  imagePullSecrets:
  - name: registry-credentials
```

**Image Pull Policies:**

| Policy | Behavior | Use Case |
|--------|----------|----------|
| **Always** | Pull on every pod start | Development, mutable tags (`:latest`) |
| **IfNotPresent** | Pull only if not cached | Production, immutable tags (`:v1.2.3`) |
| **Never** | Only use local cache | Air-gapped environments, pre-loaded images |

**Image Pull Process:**

```
1. Check local image cache
   crictl images | grep myapp:v1.2.3
   
2. If not present, authenticate to registry
   - Use imagePullSecrets if provided
   - Use node credentials if configured

3. Pull image layers (parallel downloads)
   Pulling from registry.example.com/myapp:v1.2.3
   Layer 1/5: Already exists (cached from base image)
   Layer 2/5: Downloading [=====>                ] 45%
   Layer 3/5: Downloading [===========>          ] 67%
   Layer 4/5: Queued
   Layer 5/5: Queued

4. Verify image digest (integrity check)
   Digest: sha256:abc123def456...

5. Extract layers to filesystem

6. Image ready for container creation

Performance:
- Cold pull: 30-120 seconds (depends on image size, network)
- Warm start: 1-3 seconds (image cached)
- Pre-pulling: Use DaemonSet to cache images on all nodes
```

**Image Garbage Collection:**

```bash
# kubelet configuration
--image-gc-high-threshold=85  # Start GC when disk usage > 85%
--image-gc-low-threshold=80   # Stop GC when disk usage < 80%

# GC Process:
1. Monitor disk usage every 5 minutes
2. If usage > 85%:
   - Sort images by last used time
   - Delete oldest unused images
   - Stop when usage < 80%

# Manual cleanup
crictl rmi --prune  # Remove unused images
```

### **Resource Management via cgroups**

kubelet enforces resource requests and limits using Linux cgroups (control groups).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        cpu: 500m      # 0.5 CPU cores (guaranteed)
        memory: 512Mi  # 512 MiB (guaranteed)
      limits:
        cpu: 1000m     # 1.0 CPU cores (maximum)
        memory: 1Gi    # 1 GiB (hard limit)
```

**cgroups Hierarchy:**

```bash
# cgroups v1 hierarchy on node
/sys/fs/cgroup/
├── cpu,cpuacct/
│   └── kubepods/
│       ├── burstable/           # QoS: Burstable pods
│       │   └── pod-abc123/
│       │       └── container-xyz/
│       │           ├── cpu.shares → 512 (500m * 1024)
│       │           └── cpu.cfs_quota_us → 100000 (1000m = 100ms/100ms period)
│       ├── besteffort/          # QoS: BestEffort pods
│       └── guaranteed/          # QoS: Guaranteed pods
├── memory/
│   └── kubepods/
│       └── burstable/
│           └── pod-abc123/
│               └── container-xyz/
│                   ├── memory.limit_in_bytes → 1073741824 (1Gi)
│                   └── memory.soft_limit_in_bytes → 536870912 (512Mi)
└── blkio/  (disk I/O throttling)
```

**CPU vs Memory Behavior:**

```
CPU (Compressible Resource):
- Request: Guaranteed minimum CPU time
- Limit: Maximum CPU time (throttled if exceeded)
- Exceeding limit: Container slowed down (throttled)
- No container termination

Example:
Container requests 500m, limit 1000m
Normal load: Uses 600m (within limit)
High load: Tries to use 1200m
Result: Throttled to 1000m, slower performance

Memory (Incompressible Resource):
- Request: Guaranteed minimum memory
- Limit: Maximum memory (hard cap)
- Exceeding limit: Container killed (OOMKilled)
- Container restarted per restartPolicy

Example:
Container requests 512Mi, limit 1Gi
Normal load: Uses 700Mi (within limit)
Memory leak: Reaches 1.1Gi
Result: OOMKilled, container restarted
```

### **QoS Classes**

Kubernetes assigns Quality of Service (QoS) classes based on resource specifications:

```
1. Guaranteed (Highest Priority):
   - Every container has CPU and memory limits
   - Requests equal limits (requests == limits)
   
   Example:
   resources:
     requests:
       cpu: 1000m
       memory: 1Gi
     limits:
       cpu: 1000m     # Same as request
       memory: 1Gi    # Same as request
   
   Benefits:
   - Never evicted except node failure
   - Predictable performance
   - Use for: Databases, critical services

2. Burstable (Medium Priority):
   - At least one container has CPU or memory request
   - Requests < Limits (can burst above requests)
   
   Example:
   resources:
     requests:
       cpu: 500m
       memory: 512Mi
     limits:
       cpu: 2000m     # Can burst to 2 cores
       memory: 2Gi    # Can use up to 2Gi
   
   Benefits:
   - Cost-effective (overcommit resources)
   - Handle traffic spikes
   - Use for: Web apps, APIs, microservices

3. BestEffort (Lowest Priority):
   - No requests or limits specified
   
   Example:
   resources: {}  # No specification
   
   Benefits:
   - Use spare capacity
   - Maximum flexibility
   - Use for: Batch jobs, disposable workloads
   
   Risks:
   - First to be evicted under pressure
   - No performance guarantees
```

### **Eviction Policy**

When node runs out of resources, kubelet evicts pods based on QoS class and usage:

```
Eviction Thresholds (default):
memory.available < 100Mi
nodefs.available < 10%     (root filesystem)
nodefs.inodesFree < 5%
imagefs.available < 15%    (container image filesystem)

Eviction Order:
1. BestEffort pods (no resource guarantees)
   - Sorted by total usage
   
2. Burstable pods exceeding requests
   - Sorted by usage over requests
   
3. Burstable pods within requests
   - Sorted by priority class
   
4. Guaranteed pods
   - Only evicted if system-critical processes threatened

Example Scenario:
Node: 64Gi memory total
Running pods:
- guaranteed-db: 30Gi (requests=limits=30Gi)
- burstable-api: 25Gi (requests=20Gi, limits=35Gi) ← Exceeding!
- besteffort-batch: 10Gi (no limits)
Total: 65Gi (overcommitted!)

Eviction:
T+0s: memory.available < 100Mi
T+1s: besteffort-batch evicted (10Gi freed)
      Node: 55Gi used, stable
      
If still over:
T+2s: burstable-api evicted (using 25Gi, only guaranteed 20Gi)
```

### **Health Probes**

kubelet executes three types of probes to monitor container health:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-demo
spec:
  containers:
  - name: app
    image: myapp:v1
    ports:
    - containerPort: 8080
    
    # Startup Probe: Slow-starting applications
    startupProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 0
      periodSeconds: 10
      failureThreshold: 30  # 5 minutes max startup time
      successThreshold: 1
    
    # Liveness Probe: Detect deadlocks/crashes
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: X-Probe-Type
          value: liveness
      initialDelaySeconds: 15
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 3
      successThreshold: 1
    
    # Readiness Probe: Traffic management
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      timeoutSeconds: 3
      failureThreshold: 1  # Immediately remove from service
      successThreshold: 1
```

**Probe Types:**

```yaml
# HTTP GET (most common)
httpGet:
  path: /healthz
  port: 8080
  httpHeaders:
  - name: Custom-Header
    value: probe

# TCP Socket (for non-HTTP services)
tcpSocket:
  port: 5432

# Exec Command (custom logic)
exec:
  command:
  - cat
  - /tmp/healthy
```

**Probe Behavior:**

```
Startup Probe:
- Purpose: Give slow-starting apps time to initialize
- Behavior: Disables liveness/readiness until succeeds
- Failure: Container killed after failureThreshold

Timeline:
T+0s:   Container starts
T+10s:  Startup probe: FAILED (app still initializing)
T+20s:  Startup probe: FAILED
T+30s:  Startup probe: SUCCESS (app ready)
T+30s:  Enable liveness/readiness probes

Liveness Probe:
- Purpose: Detect unrecoverable errors (deadlocks, memory leaks)
- Behavior: Restart container on failure
- Failure: kubelet kills container, restartPolicy applies

Timeline:
T+0s:   Liveness probe: SUCCESS
T+10s:  Liveness probe: SUCCESS
T+20s:  Liveness probe: FAILED (app deadlocked)
T+30s:  Liveness probe: FAILED (2nd failure)
T+40s:  Liveness probe: FAILED (3rd failure = threshold)
T+40s:  Container killed (SIGTERM)
T+70s:  Grace period expires (SIGKILL)
T+71s:  New container starts

Readiness Probe:
- Purpose: Control traffic routing
- Behavior: Remove from Service endpoints on failure
- Failure: No traffic sent, container NOT restarted

Timeline:
T+0s:   Readiness probe: SUCCESS (receiving traffic)
T+5s:   Readiness probe: SUCCESS
T+10s:  Readiness probe: FAILED (database connection lost)
T+10s:  Pod removed from Service endpoints (no new traffic)
T+15s:  Readiness probe: SUCCESS (database recovered)
T+15s:  Pod added back to Service endpoints

Result: Graceful degradation without restart
```

**Best Practices:**

```yaml
# Anti-pattern: Expensive probes
livenessProbe:
  exec:
    command: ["sh", "-c", "curl db && curl cache"]  # ❌ Slow, external deps

# Best practice: Lightweight probes
livenessProbe:
  httpGet:
    path: /healthz  # Simple process health check
    port: 8080
  periodSeconds: 10
  timeoutSeconds: 1

readinessProbe:
  httpGet:
    path: /ready  # Checks dependencies
    port: 8080
  periodSeconds: 5
  timeoutSeconds: 3

# Probe endpoints should:
# - Liveness: Check process health only (no external deps)
# - Readiness: Check dependencies (DB, cache, APIs)
# - Return quickly (< 1s for liveness, < 3s for readiness)
# - Not perform heavy operations (no DB queries in liveness!)
```

---

## Container Runtime Interface (CRI)

### **Architecture**

CRI is the abstraction layer between kubelet and container runtimes, enabling pluggable runtimes without changing kubelet code.

```
┌─────────────────────────────────────────────┐
│ kubelet                                     │
│ "I need to create a pod"                    │
└────────────────┬────────────────────────────┘
                 │ gRPC API
                 ▼
┌─────────────────────────────────────────────┐
│ CRI (Container Runtime Interface)           │
│                                             │
│ RuntimeService:                             │
│ - RunPodSandbox()                           │
│ - CreateContainer()                         │
│ - StartContainer()                          │
│ - StopContainer()                           │
│ - RemoveContainer()                         │
│ - ListContainers()                          │
│                                             │
│ ImageService:                               │
│ - PullImage()                               │
│ - RemoveImage()                             │
│ - ListImages()                              │
└────────────────┬────────────────────────────┘
                 │
        ┌────────┴────────┬──────────────┐
        ▼                 ▼              ▼
  ┌──────────┐      ┌──────────┐   ┌──────────┐
  │containerd│      │  CRI-O   │   │  Docker  │
  │          │      │          │   │ (shim)   │
  └──────────┘      └──────────┘   └──────────┘
```

### **Runtime Comparison**

| Runtime | Architecture | Pros | Cons | Production Use |
|---------|-------------|------|------|----------------|
| **containerd** | Daemon + gRPC | Industry standard, Docker-compatible, battle-tested | Requires crictl CLI | **Recommended** (K8s default 1.20+) |
| **CRI-O** | Kubernetes-native | Lightweight, K8s-optimized, minimal footprint | Smaller ecosystem | Growing (RedHat/OpenShift default) |
| **Docker** | Legacy via shim | Familiar tooling, large ecosystem | Deprecated (K8s 1.20), removed (1.24) | **Legacy only** |

### **Pod Sandbox Concept**

```
Pod = Pod Sandbox + Containers

Pod Sandbox (pause container):
- Network namespace (shared by all containers)
- IPC namespace (inter-process communication)
- PID namespace (optional: process visibility)
- Cgroups (resource limits at pod level)

Why Pause Container?
- Holds namespaces open even if app containers restart
- Minimal resource usage (~100KB memory)
- Never exits (keeps pod alive)
- Simplifies container lifecycle management

Example:
kubectl get pods myapp -o json | jq '.status.containerStatuses[].name'
"app"          # Application container
"sidecar"      # Sidecar container
"POD"          # Pause container (infrastructure)
```

### **Container Lifecycle via CRI**

```
Complete Flow:

1. kubelet: "Create pod 'myapp'"
   ↓
2. CRI: RunPodSandbox(config)
   Request:
     metadata: {name: "myapp", namespace: "default"}
     linux: {security_context, cgroup_parent}
   ↓
3. containerd: Creates pause container
   - Create network namespace (netns)
   - Set up cgroups
   - Start pause process (infinite sleep)
   ↓
4. CNI plugin: Configure networking
   - Allocate IP: 10.244.5.8
   - Create veth pair
   - Set up routes
   ↓
5. CRI: CreateContainer("nginx", pod_sandbox_id)
   Request:
     image: "nginx:1.21"
     command: ["/usr/sbin/nginx"]
     mounts: [{volume_mount_path}]
     linux: {resources, security_context}
   ↓
6. containerd: Create container
   - Pull image (if not cached)
   - Create OCI runtime spec
   - Create container filesystem (overlay)
   ↓
7. CRI: StartContainer(container_id)
   ↓
8. containerd: Start container process
   - Execute container entrypoint
   - Join pod sandbox namespaces
   - Apply cgroup limits
   ↓
9. Container running, kubelet monitoring

Termination:
1. kubelet: "Delete pod 'myapp'"
   ↓
2. CRI: StopContainer(container_id, timeout=30s)
   ↓
3. containerd: Send SIGTERM to container
   ↓
4. Wait grace period (30s)
   ↓
5. containerd: Send SIGKILL if still running
   ↓
6. CRI: RemoveContainer(container_id)
   ↓
7. containerd: Delete container filesystem
   ↓
8. CRI: StopPodSandbox(pod_sandbox_id)
   ↓
9. containerd: Stop pause container
   ↓
10. CNI plugin: Cleanup networking
```

### **Container Isolation**

Linux provides isolation through namespaces:

```
Namespace Types:

1. PID Namespace:
   - Container sees itself as PID 1
   - Cannot see host processes
   - Children processes visible within container
   
   Example:
   # Inside container
   ps aux
   PID  COMMAND
   1    /app/server
   25   /app/worker
   # Cannot see node processes!

2. Network Namespace:
   - Isolated network stack (interfaces, routes, iptables)
   - Shared across containers in same pod
   
   Example:
   # Container A and B in same pod
   Container A: localhost:8080
   Container B: localhost:8080  # Same port OK! (different containers)
   
   # Communication
   Container A → localhost:8081 → Container B

3. Mount Namespace:
   - Isolated filesystem view
   - Container sees only its rootfs + mounted volumes
   
   Example:
   # Inside container
   ls /
   bin  etc  lib  usr  var  app/  # Container filesystem
   # Cannot see /host/files

4. UTS Namespace:
   - Isolated hostname
   
   Example:
   # Inside container
   hostname
   myapp-abc123-xyz789  # Pod name
   
   # On host
   hostname
   node-5.cluster.local

5. IPC Namespace:
   - Isolated inter-process communication
   - Shared across containers in same pod
   
   Example:
   # Containers in pod can use shared memory
   # But isolated from other pods

6. User Namespace:
   - Map container root to non-root on host
   
   Example:
   # Inside container
   whoami
   root (UID 0)
   
   # On host (mapped)
   ps aux | grep container-process
   user1000  12345  # Mapped to UID 1000
```

---

## kube-proxy - Service Networking

### **Core Functionality**

kube-proxy implements the Service abstraction, providing stable IPs and load balancing.

**Service Types:**

```yaml
# ClusterIP (default) - Internal only
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP
  clusterIP: 10.96.5.20  # Stable virtual IP
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 8080

# NodePort - External access via node IP
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080  # Accessible at <NodeIP>:30080

# LoadBalancer - Cloud load balancer
apiVersion: v1
kind: Service
metadata:
  name: public-api
spec:
  type: LoadBalancer
  selector:
    app: api
  ports:
  - port: 443
    targetPort: 8443
  # Cloud controller provisions LB
```

### **kube-proxy Modes**

**1. iptables Mode (Default)**

```bash
# Service: backend (10.96.5.20:80)
# Endpoints: 192.168.1.10:8080, 192.168.1.11:8080, 192.168.1.12:8080

# kube-proxy creates iptables rules:
iptables -t nat -A PREROUTING -j KUBE-SERVICES

iptables -t nat -A KUBE-SERVICES \
  -d 10.96.5.20/32 -p tcp --dport 80 \
  -j KUBE-SVC-BACKEND

# Load balancing (random selection)
iptables -t nat -A KUBE-SVC-BACKEND \
  -m statistic --mode random --probability 0.33333 \
  -j KUBE-SEP-1  # 33% → 192.168.1.10:8080

iptables -t nat -A KUBE-SVC-BACKEND \
  -m statistic --mode random --probability 0.50000 \
  -j KUBE-SEP-2  # 50% of remaining → 192.168.1.11:8080

iptables -t nat -A KUBE-SVC-BACKEND \
  -j KUBE-SEP-3  # Remaining 100% → 192.168.1.12:8080

# DNAT (Destination NAT)
iptables -t nat -A KUBE-SEP-1 \
  -p tcp -j DNAT --to-destination 192.168.1.10:8080

# Packet flow:
# Client → 10.96.5.20:80
# iptables DNAT → 192.168.1.10:8080
# Packet forwarded to pod
```

**Characteristics:**
- **Pros:** Kernel-level routing (fast), no userspace proxy
- **Cons:** O(n) rule evaluation, large clusters = many rules

**2. IPVS Mode (Recommended for Large Clusters)**

```bash
# Same service, uses IPVS (IP Virtual Server)
ipvsadm -A -t 10.96.5.20:80 -s rr  # Round-robin

# Add real servers (endpoints)
ipvsadm -a -t 10.96.5.20:80 -r 192.168.1.10:8080 -m
ipvsadm -a -t 10.96.5.20:80 -r 192.168.1.11:8080 -m
ipvsadm -a -t 10.96.5.20:80 -r 192.168.1.12:8080 -m

# Load balancing algorithms:
# - rr: Round-robin (default)
# - lc: Least connections
# - dh: Destination hashing (session affinity)
# - sh: Source hashing
# - lblc: Locality-based least connection
# - sed: Shortest expected delay
```

**Characteristics:**
- **Pros:** O(1) hash table lookup, better performance at scale
- **Cons:** Requires kernel modules (`ip_vs`)

**Performance Comparison:**

| Metric | iptables | IPVS |
|--------|----------|------|
| **Lookup** | O(n) linear | O(1) hash table |
| **5K Services** | 5-10ms latency | <1ms latency |
| **Throughput** | 50 Gbps | 100 Gbps |
| **CPU (10K conn/s)** | 80% | 20% |

### **Service Discovery**

```
Two Mechanisms:

1. Environment Variables (Legacy):
   Every pod gets env vars for services:
   BACKEND_SERVICE_HOST=10.96.5.20
   BACKEND_SERVICE_PORT=80
   
   Limitation: Only for services created BEFORE pod

2. DNS (Recommended):
   CoreDNS provides DNS resolution:
   backend.default.svc.cluster.local → 10.96.5.20
   
   Format: <service>.<namespace>.svc.<cluster-domain>

DNS Resolution Flow:
1. App: curl http://backend
2. Container /etc/resolv.conf:
   nameserver 10.96.0.10  # CoreDNS
   search default.svc.cluster.local svc.cluster.local cluster.local
3. Resolver tries:
   - backend (fails)
   - backend.default.svc.cluster.local (success!)
4. CoreDNS returns: 10.96.5.20
5. App connects to 10.96.5.20:80
6. iptables/IPVS DNAT to pod endpoint
```

---

## Resource Management

### **Node Allocatable Resources**

```
Node Capacity vs Allocatable:

Total Node Resources: 64 CPU, 256Gi Memory
  │
  ├─ System Reserved (--system-reserved)
  │  kubelet, containerd, sshd, systemd# 🖥️ Kubernetes Data Plane & Node Architecture

> **Expert-Level Technical Reference**  
> Deep dive into node components, container runtime architecture, resource management, and workload execution in Kubernetes.
