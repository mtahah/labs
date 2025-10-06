# 🏗️ Kubernetes Control Plane Architecture - Deep Dive

> **Expert-Level Technical Reference**  
> Deep architectural analysis of Kubernetes control plane components, distributed systems principles, and production deployment patterns.

---

## 📋 Table of Contents

1. [Control Plane Overview](#control-plane-overview)
2. [API Server Architecture](#api-server-architecture)
3. [etcd - The Source of Truth](#etcd-source-of-truth)
4. [Scheduler Deep Dive](#scheduler-deep-dive)
5. [Controller Manager](#controller-manager)
6. [Cloud Controller Manager](#cloud-controller-manager)
7. [High Availability Patterns](#high-availability-patterns)

---

## Control Plane Overview

### **Architectural Philosophy**

Kubernetes implements a **declarative** system where users specify desired state and the control plane continuously reconciles actual state to match. This is fundamentally different from imperative systems.

```
Declarative Model:
User: "I want 3 replicas of nginx"
System: Continuously ensures 3 replicas exist (self-healing)

Imperative Model:
User: "Start nginx container on node-5"
System: Executes command once, no self-healing
```

### **Control Plane vs Data Plane**

```
┌─────────────────────────────────────────────────────┐
│ CONTROL PLANE (Brain)                               │
│ - Makes decisions                                   │
│ - Stores desired state                              │
│ - Schedules workloads                               │
│ - Detects failures                                  │
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │   API    │  │   etcd   │  │Scheduler │        │
│  │  Server  │  │          │  │          │        │
│  └──────────┘  └──────────┘  └──────────┘        │
│                                                     │
│  ┌──────────────────────────────────────┐         │
│  │    Controller Manager                │         │
│  │    (Reconciliation Loops)            │         │
│  └──────────────────────────────────────┘         │
└─────────────────────────────────────────────────────┘
                      │
                      │ Instructions
                      ▼
┌─────────────────────────────────────────────────────┐
│ DATA PLANE (Muscles)                                │
│ - Executes workloads                                │
│ - Runs containers                                   │
│ - Manages networking                                │
│ - Handles storage                                   │
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │  Node 1  │  │  Node 2  │  │  Node 3  │        │
│  │          │  │          │  │          │        │
│  │ kubelet  │  │ kubelet  │  │ kubelet  │        │
│  │ runtime  │  │ runtime  │  │ runtime  │        │
│  └──────────┘  └──────────┘  └──────────┘        │
└─────────────────────────────────────────────────────┘
```

**Critical Insight:** Control plane can fail entirely and applications continue running. The control plane reconciles state; it doesn't directly manage running processes.

---

## API Server Architecture

### **Core Responsibilities**

The API Server (`kube-apiserver`) is the **only** component that directly interacts with etcd. Every other component (kubelet, controllers, kubectl) communicates exclusively through the API Server's RESTful interface.

**Why Single Entry Point?**

1. **Centralized Authentication/Authorization** - All requests authenticated once
2. **Admission Control** - Policies enforced consistently
3. **Audit Trail** - Complete logging of all mutations
4. **Simplified Consistency** - Single writer to etcd prevents race conditions
5. **API Versioning** - Backward compatibility managed centrally

### **Request Processing Pipeline**

```
kubectl apply -f deployment.yaml
        │
        ▼
┌─────────────────────────────────────────────────┐
│ PHASE 1: AUTHENTICATION                         │
│ Who are you?                                    │
│                                                 │
│ Methods (tried in order):                       │
│ 1. X.509 Client Certificates                   │
│ 2. Bearer Tokens (ServiceAccount)              │
│ 3. OpenID Connect (OIDC)                       │
│ 4. Authenticating Proxy                        │
│ 5. HTTP Basic Auth (deprecated)                │
│                                                 │
│ Result: User/ServiceAccount identity           │
└────────────────┬────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────┐
│ PHASE 2: AUTHORIZATION                          │
│ What can you do?                                │
│                                                 │
│ Authorization Modes (checked in order):         │
│ 1. RBAC (Role-Based Access Control)            │
│ 2. ABAC (Attribute-Based - legacy)             │
│ 3. Node Authorization (kubelet-specific)       │
│ 4. Webhook (external authorization)            │
│                                                 │
│ Checks: Can this user create deployments?      │
│ Result: Allow or Deny                          │
└────────────────┬────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────┐
│ PHASE 3: MUTATING ADMISSION CONTROLLERS         │
│ Modify the request                              │
│                                                 │
│ Examples:                                       │
│ - ServiceAccount Injection                     │
│   (adds default SA if not specified)           │
│ - PodSecurityPolicy                            │
│   (adds security context)                      │
│ - DefaultStorageClass                          │
│   (sets default SC for PVCs)                   │
│ - Istio Sidecar Injection                      │
│   (injects Envoy proxy)                        │
│                                                 │
│ Webhooks can modify objects before persistence │
└────────────────┬────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────┐
│ PHASE 4: VALIDATING ADMISSION CONTROLLERS       │
│ Accept or reject the request                   │
│                                                 │
│ Examples:                                       │
│ - Pod Security Admission                       │
│   (enforces security standards)                │
│ - ResourceQuota                                │
│   (enforces namespace quotas)                  │
│ - LimitRanger                                  │
│   (validates resource limits)                  │
│ - Custom Policies (OPA, Kyverno)              │
│   (business logic validation)                  │
│                                                 │
│ Any webhook can veto the entire request        │
└────────────────┬────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────┐
│ PHASE 5: SCHEMA VALIDATION                      │
│ Is the YAML valid?                              │
│                                                 │
│ - Checks against OpenAPI schema                │
│ - Required fields present?                     │
│ - Field types correct?                         │
│ - Unknown fields rejected                      │
│                                                 │
│ Result: Valid API object or error              │
└────────────────┬────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────┐
│ PHASE 6: PERSISTENCE (etcd)                     │
│ Store the object                                │
│                                                 │
│ - Serialize to protobuf                        │
│ - Write to etcd with optimistic locking        │
│   (resourceVersion check)                      │
│ - Encryption at rest (if enabled)              │
│                                                 │
│ etcd key: /registry/deployments/default/nginx  │
└────────────────┬────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────┐
│ PHASE 7: RESPONSE                               │
│ Return to client                                │
│                                                 │
│ HTTP 201 Created                                │
│ {                                               │
│   "apiVersion": "apps/v1",                     │
│   "kind": "Deployment",                        │
│   "metadata": {                                │
│     "name": "nginx",                           │
│     "resourceVersion": "12345"                 │
│   }                                            │
│ }                                              │
└────────────────┬────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────┐
│ PHASE 8: WATCH NOTIFICATIONS                    │
│ Notify watchers                                 │
│                                                 │
│ Controllers watching deployments receive:       │
│ - Event type: ADDED                            │
│ - Object: Full deployment spec                 │
│                                                 │
│ Triggers controller reconciliation loops       │
└─────────────────────────────────────────────────┘
```

### **Performance Characteristics**

```
API Server Latency (typical):
- Authentication: 1-5ms (cert validation)
- Authorization (RBAC): 2-10ms (policy evaluation)
- Admission controllers: 5-50ms (depends on webhooks)
- etcd write: 10-30ms (consensus + disk sync)
- Total: 20-100ms per request

Bottlenecks:
1. External admission webhooks (network calls)
2. etcd write latency (disk I/O)
3. Large object serialization (>1MB objects)

Optimization:
- Cache frequently accessed objects
- Use watches instead of polling
- Batch operations where possible
```

### **Horizontal Scaling**

```yaml
# API Server is stateless - scales horizontally
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-apiserver
  namespace: kube-system
spec:
  replicas: 3  # Multiple instances for HA
  template:
    spec:
      containers:
      - name: kube-apiserver
        image: k8s.gcr.io/kube-apiserver:v1.28.0
        command:
        - kube-apiserver
        - --etcd-servers=https://etcd-0:2379,https://etcd-1:2379,https://etcd-2:2379
        - --service-cluster-ip-range=10.96.0.0/12
        - --secure-port=6443
        # All instances share same etcd cluster
```

**Load Balancing:**

```
┌──────────────┐
│ Load Balancer│ (HAProxy, nginx, cloud LB)
└──────┬───────┘
       │
   ┌───┴────┬────────┬────────┐
   │        │        │        │
   ▼        ▼        ▼        ▼
┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│ API  │ │ API  │ │ API  │ │ API  │
│ Srv 1│ │ Srv 2│ │ Srv 3│ │ Srv 4│
└──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘
   │        │        │        │
   └────────┴────────┴────────┘
              │
              ▼
      ┌───────────────┐
      │  etcd Cluster │
      │  (3-5 nodes)  │
      └───────────────┘

Scaling Patterns:
- Small clusters (<100 nodes): 1-2 API Servers
- Medium clusters (100-500 nodes): 3 API Servers
- Large clusters (500-5000 nodes): 5+ API Servers
```

### **Watch Mechanism**

The watch mechanism is how controllers efficiently track changes without polling.

```go
// Pseudo-code: How controllers watch resources
watcher := clientset.CoreV1().Pods("default").Watch(context.TODO(), metav1.ListOptions{})

for event := range watcher.ResultChan() {
    switch event.Type {
    case watch.Added:
        fmt.Printf("Pod added: %s\n", event.Object.(*v1.Pod).Name)
    case watch.Modified:
        fmt.Printf("Pod modified: %s\n", event.Object.(*v1.Pod).Name)
    case watch.Deleted:
        fmt.Printf("Pod deleted: %s\n", event.Object.(*v1.Pod).Name)
    }
}
```

**Watch Implementation:**

```
1. Client opens HTTP connection with watch=true parameter
2. API Server keeps connection open (long-polling)
3. On etcd change, API Server sends event to client
4. Connection stays open indefinitely (with heartbeats)

Benefits:
- No polling overhead
- Instant notification (within milliseconds)
- Scalable (thousands of watchers per API Server)

Challenges:
- Network interruptions require reconnection
- Watch bookmarks for efficiency (periodic sync)
- ResourceVersion tracking for consistency
```

---

## etcd - The Source of Truth

### **Architecture & Raft Consensus**

etcd is a distributed key-value store that implements the **Raft consensus algorithm** to maintain strong consistency across cluster members.

**CAP Theorem Trade-off:**

```
CAP Theorem: Pick 2 of 3
- Consistency
- Availability  
- Partition Tolerance

etcd choice: CP (Consistency + Partition Tolerance)
- Sacrifices availability during network partitions
- Ensures data integrity (no split-brain)
```

### **Raft Consensus Explained**

```
Raft States:
1. Leader: Handles all writes, sends heartbeats
2. Follower: Receives updates from leader
3. Candidate: Election state (when leader fails)

Normal Operation (3-node cluster):

  ┌────────┐         ┌────────┐         ┌────────┐
  │ etcd-0 │         │ etcd-1 │         │ etcd-2 │
  │ LEADER │◄────────│FOLLOWER│         │FOLLOWER│
  └────┬───┘         └────┬───┘         └────┬───┘
       │   Heartbeat      │   Heartbeat      │
       ├──────────────────┤                  │
       │                  │                  │
       └──────────────────┴──────────────────┘

Write Process:
1. Client sends write to leader (etcd-0)
2. Leader appends to its log
3. Leader replicates to followers (etcd-1, etcd-2)
4. Followers acknowledge receipt
5. Leader commits (quorum: 2/3 acknowledged)
6. Leader responds to client (write committed)
7. Leader sends commit index to followers
8. Followers apply to state machine

Quorum: (N/2) + 1
- 3 nodes: need 2 (can tolerate 1 failure)
- 5 nodes: need 3 (can tolerate 2 failures)
- 7 nodes: need 4 (can tolerate 3 failures)
```

### **Leader Election**

```
Timeline: Leader Failure

T+0s:   etcd-0 (leader) crashes
        etcd-1, etcd-2 stop receiving heartbeats

T+150ms: Election timeout expires
         etcd-1 transitions to Candidate state
         etcd-1 increments term number (term=5)
         etcd-1 votes for itself
         etcd-1 requests votes from etcd-2

T+155ms: etcd-2 receives vote request
         etcd-2 validates:
         - Term 5 > current term (4) ✓
         - etcd-2 hasn't voted this term ✓
         - etcd-1's log is up-to-date ✓
         etcd-2 votes for etcd-1

T+160ms: etcd-1 receives vote from etcd-2
         etcd-1 has majority (2/3 votes)
         etcd-1 transitions to Leader
         etcd-1 sends heartbeats to assert leadership

T+165ms: etcd-2 receives heartbeat from new leader
         etcd-2 acknowledges etcd-1 as leader

Result: New leader elected in ~165ms
        Cluster operational, accepting writes
```

**Election Timeout Tuning:**

```bash
# Default election timeout: 1000ms
# Can be tuned based on network latency

# For low-latency networks (same datacenter):
--election-timeout=1000  # Default

# For high-latency networks (cross-region):
--election-timeout=5000  # Prevent false leader failures

# For very fast failover (risk: more elections):
--election-timeout=500   # Aggressive
```

### **Data Model & Performance**

```
etcd Key-Value Structure:

/registry/
├── pods/
│   ├── default/
│   │   ├── nginx-abc123 → JSON blob (pod spec + status)
│   │   └── postgres-xyz789 → JSON blob
│   └── kube-system/
│       └── coredns-123 → JSON blob
├── services/
│   └── default/
│       └── kubernetes → JSON blob
├── deployments/
│   └── default/
│       └── webapp → JSON blob
└── secrets/
    └── default/
        └── db-password → Encrypted JSON blob

Key Characteristics:
- Hierarchical keys (like filesystem paths)
- Values are arbitrary bytes (typically JSON/protobuf)
- Keys are lexicographically ordered
- Supports prefix queries: list all under /registry/pods/
```

**Performance Limits:**

```
etcd Design Constraints:
- Total DB size: 8GB hard limit (default 2GB)
  Reason: Raft requires in-memory replication
  
- Value size: 1.5MB per key (hard limit)
  Reason: gRPC message size limit
  
- Sustained writes: ~10,000 writes/sec
  Bottleneck: Disk fsync latency
  
- Sustained reads: 100,000+ reads/sec
  Read from memory, no disk I/O

Typical Kubernetes Usage:
- 100-node cluster: ~500MB etcd DB
- 1000-node cluster: ~2-3GB etcd DB
- 5000-node cluster: ~5-7GB etcd DB (approaching limits!)
```

**Optimization Strategies:**

```yaml
# 1. Compaction - Remove old revisions
# By default, etcd keeps all historical versions

# Manual compaction
etcdctl compact <revision>

# Automatic compaction (recommended)
--auto-compaction-mode=periodic
--auto-compaction-retention=5m  # Keep last 5 minutes

# 2. Defragmentation - Reclaim disk space
# After compaction, disk space not immediately freed

etcdctl defrag --cluster

# 3. Disk Performance
# Use SSD (NVMe preferred)
# Low latency critical: p99 < 10ms disk write

# Monitor disk latency
etcdctl check perf
# Should show: >90% of disk writes < 10ms

# 4. Separate etcd disks
# Don't share disk with OS or other apps
# Dedicated disk for etcd WAL (write-ahead log)
```

### **Backup & Disaster Recovery**

```bash
# Snapshot (complete backup)
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot-$(date +%Y%m%d-%H%M%S).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify snapshot
etcdctl snapshot status /backup/etcd-snapshot.db
# Output:
# hash: 3456789abc
# revision: 125432
# total keys: 15678
# total size: 2.3 GB

# Restore from snapshot (disaster recovery)
# CRITICAL: Stop all API Servers first!
# Reason: Prevents writes during restore

# On each etcd node:
etcdctl snapshot restore /backup/etcd-snapshot.db \
  --name=etcd-0 \
  --initial-cluster=etcd-0=https://10.0.0.1:2380,etcd-1=https://10.0.0.2:2380,etcd-2=https://10.0.0.3:2380 \
  --initial-advertise-peer-urls=https://10.0.0.1:2380 \
  --data-dir=/var/lib/etcd-restored

# Start etcd with restored data
systemctl start etcd

# Restart API Servers
systemctl start kube-apiserver

# Verify cluster state
kubectl get nodes
kubectl get pods --all-namespaces
```

**Backup Strategy (Production):**

```
Layered Backup Approach:

Layer 1: Continuous snapshots
- Frequency: Every 30 minutes
- Retention: 48 hours (96 snapshots)
- Purpose: Quick recovery from recent issues

Layer 2: Daily snapshots
- Frequency: Daily at 2 AM
- Retention: 30 days
- Purpose: Point-in-time recovery

Layer 3: Weekly snapshots
- Frequency: Sunday at 2 AM
- Retention: 12 weeks (3 months)
- Purpose: Long-term recovery

Layer 4: Off-site replication
- Method: Streaming replication or snapshot copy
- Location: Different region/datacenter
- Purpose: Disaster recovery (datacenter failure)

Automation Example:
apiVersion: batch/v1
kind: CronJob
metadata:
  name: etcd-backup
  namespace: kube-system
spec:
  schedule: "*/30 * * * *"  # Every 30 minutes
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: k8s.gcr.io/etcd:3.5.9
            command:
            - /bin/sh
            - -c
            - |
              BACKUP_FILE="/backup/etcd-$(date +%Y%m%d-%H%M%S).db"
              etcdctl snapshot save $BACKUP_FILE
              # Upload to S3/GCS/Azure Blob
              aws s3 cp $BACKUP_FILE s3://k8s-backups/etcd/
              # Cleanup local old backups
              find /backup -name "etcd-*.db" -mtime +2 -delete
          restartPolicy: OnFailure
```

### **Monitoring & Alerting**

```yaml
# Critical metrics to monitor
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: etcd-alerts
spec:
  groups:
  - name: etcd
    rules:
    # Slow disk writes (critical!)
    - alert: EtcdDiskWriteSlow
      expr: histogram_quantile(0.99, rate(etcd_disk_wal_fsync_duration_seconds_bucket[5m])) > 0.01
      annotations:
        summary: "etcd disk writes slow (p99 > 10ms)"
        description: "Slow disk is #1 cause of etcd issues"
    
    # High leader changes (instability)
    - alert: EtcdHighLeaderChanges
      expr: rate(etcd_server_leader_changes_seen_total[10m]) > 3
      annotations:
        summary: "etcd leader changes frequently"
        description: "Network issues or resource contention"
    
    # No leader (cluster down!)
    - alert: EtcdNoLeader
      expr: etcd_server_has_leader == 0
      annotations:
        summary: "etcd has no leader"
        description: "Cluster unavailable, quorum lost"
    
    # Database size approaching limit
    - alert: EtcdDatabaseSizeHigh
      expr: etcd_mvcc_db_total_size_in_bytes > 6.0e9  # 6GB
      annotations:
        summary: "etcd database approaching 8GB limit"
        description: "Compaction needed or cluster too large"
```

---

## Scheduler Deep Dive

### **Scheduling Algorithm**

The Scheduler (`kube-scheduler`) uses a two-phase algorithm: **Filtering** (Predicates) and **Scoring** (Priorities).

```
┌────────────────────────────────────────────────┐
│ PHASE 1: FILTERING (Predicates)               │
│ Eliminate nodes that cannot run the pod       │
└────────────────────────────────────────────────┘

Available Nodes: [node-1, node-2, node-3, node-4, node-5]
Pod Requirements:
- CPU: 2 cores
- Memory: 4Gi
- GPU: 1x nvidia.com/gpu
- NodeSelector: disktype=ssd
- NodeAffinity: zone=us-east-1a

Filter 1: PodFitsResources
- node-1: 1 CPU available ❌
- node-2: 3 CPU, 8Gi RAM ✓
- node-3: 4 CPU, 2Gi RAM ❌ (insufficient memory)
- node-4: 3 CPU, 8Gi RAM ✓
- node-5: 3 CPU, 8Gi RAM ✓

Filter 2: PodFitsHostPorts
- Check if host ports (if any) are available
- All remaining nodes pass ✓

Filter 3: NodeSelector
- node-2: disktype=ssd ✓
- node-4: disktype=hdd ❌
- node-5: disktype=ssd ✓

Filter 4: NodeAffinity
- node-2: zone=us-east-1a ✓
- node-5: zone=us-east-1b ❌

Filter 5: CheckNodeCondition
- node-2: Ready=True ✓

Filter 6: GPU Available
- node-2: nvidia.com/gpu=2 ✓

Feasible Nodes: [node-2]

┌────────────────────────────────────────────────┐
│ PHASE 2: SCORING (Priorities)                 │
│ Rank remaining nodes by desirability          │
└────────────────────────────────────────────────┘

If multiple nodes pass filtering, score each:

node-2 scores:
- LeastRequestedPriority: 75/100
  (Low resource utilization = better)
  CPU: 50% used, Memory: 30% used
  
- BalancedResourceAllocation: 80/100
  (Balanced CPU/Memory = better)
  CPU and memory usage similar
  
- NodeAffinityPriority: 100/100
  (Matches preferred affinity)
  
- ImageLocalityPriority: 90/100
  (Image already cached on node)
  
- InterPodAffinityPriority: 70/100
  (Matches pod affinity rules)

Total Score: 415/500

Winner: node-2 (highest score)
Bind pod to node-2
```

### **Advanced Scheduling Features**

**1. Node Affinity (Hard and Soft Constraints)**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-workload
spec:
  affinity:
    nodeAffinity:
      # HARD: Must match (pod won't schedule otherwise)
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: gpu-type
            operator: In
            values:
            - nvidia-a100
            - nvidia-v100
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - us-east-1a
            - us-east-1b
      
      # SOFT: Prefer but not required (best effort)
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: instance-type
            operator: In
            values:
            - p3.8xlarge  # Preferred instance type
      - weight: 50
        preference:
          matchExpressions:
          - key: network-speed
            operator: In
            values:
            - 25gbps
  containers:
  - name: training
    image: tensorflow/tensorflow:latest-gpu
    resources:
      limits:
        nvidia.com/gpu: 4
```

**Operators:**

- `In`: Label value in list
- `NotIn`: Label value not in list
- `Exists`: Label key exists (any value)
- `DoesNotExist`: Label key doesn't exist
- `Gt`: Label value > integer
- `Lt`: Label value < integer

**2. Pod Affinity & Anti-Affinity**

```yaml
# Use Case: Co-locate cache pods with application pods
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  affinity:
    podAffinity:
      # Schedule near pods matching this label
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - redis-cache
        topologyKey: kubernetes.io/hostname  # Same node
    
    podAntiAffinity:
      # Don't schedule near pods matching this label
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - webapp  # Don't co-locate webapp replicas
        topologyKey: kubernetes.io/hostname  # Different nodes
```

**Topology Keys:**

- `kubernetes.io/hostname`: Node-level (different nodes)
- `topology.kubernetes.io/zone`: Zone-level (different AZs)
- `topology.kubernetes.io/region`: Region-level (different regions)

**Production Pattern:**

```yaml
# High-availability database: spread replicas across zones
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  replicas: 3
  template:
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: postgres
            topologyKey: topology.kubernetes.io/zone
      # Result: postgres-0 in us-east-1a
      #         postgres-1 in us-east-1b
      #         postgres-2 in us-east-1c
```

**3. Taints and Tolerations**

```bash
# Taint a node (mark it special-purpose)
kubectl taint nodes gpu-node-1 gpu=true:NoSchedule

# Effects:
# - NoSchedule: Don't schedule new pods
# - PreferNoSchedule: Avoid scheduling (soft)
# - NoExecute: Evict existing pods immediately
```

```yaml
# Pod tolerates the taint
apiVersion: v1
kind: Pod
metadata:
  name: ml-training
spec:
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  containers:
  - name: training
    image: tensorflow/tensorflow:latest-gpu
    resources:
      limits:
        nvidia.com/gpu: 2
```

**Common Taint Patterns:**

```bash
# Dedicated nodes for specific workloads
kubectl taint nodes node-1 workload-type=batch:NoSchedule
kubectl taint nodes node-2 workload-type=streaming:NoSchedule

# Nodes under maintenance
kubectl taint nodes node-3 maintenance=true:NoExecute
# Evicts all pods without matching toleration

# Nodes with special hardware
kubectl taint nodes nvme-node-1 storage-type=nvme:NoSchedule

# Remove taint
kubectl taint nodes node-1 gpu=true:NoSchedule-
```

**4. Priority and Preemption**

```yaml
# Define priority classes
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: system-critical
value: 2000000000  # Higher = more important
globalDefault: false
preemptionPolicy: PreemptLowerPriority
description: "System-critical services"

---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: production-high
value: 1000000
globalDefault: false
preemptionPolicy: PreemptLowerPriority
description: "High-priority production workloads"

---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: production-normal
value: 100000
globalDefault: true  # Default for pods without priority
preemptionPolicy: PreemptLowerPriority
description: "Normal production workloads"

---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: batch-jobs
value: 1000
globalDefault: false
preemptionPolicy: Never  # Cannot preempt others
description: "Low-priority batch workloads"
```

**Preemption Scenario:**

```
Cluster State:
- Total: 100 CPU cores
- Used by batch-jobs (priority=1000): 95 cores
- Available: 5 cores

New Pod Arrives:
- Name: payment-service
- Priority: production-high (1000000)
- Requires: 20 cores

Scheduler Process:
1. Filter: No node has 20 cores available
2. Preemption: Calculate which pods to evict
   - Find lowest-priority pods
   - Calculate minimum evictions needed (20 cores)
   - Select 4 batch jobs (5 cores each)
3. Evict: Send termination signal to batch jobs
4. Wait: Grace period (30s default)
5. Schedule: Place payment-service

Result: Critical service never starves for resources
```

### **Scheduler Performance**

```
Scheduling Latency Breakdown:

Small Cluster (50 nodes, 1000 pods):
- Filter phase: 5-10ms
- Score phase: 10-20ms
- Total: 15-30ms per pod

Large Cluster (1000 nodes, 10000 pods):
- Filter phase: 50-100ms
- Score phase: 100-300ms
- Total: 150-400ms per pod

Very Large Cluster (5000 nodes, 50000 pods):
- Filter phase: 200-500ms
- Score phase: 500-2000ms
- Total: 700-2500ms per pod

Optimization Strategies:
1. Percentage of nodes to score (--percentageOfNodesToScore=50)
   - Don't score all nodes, just top 50% from filtering
   
2. Multiple scheduler instances
   - Run 3-5 schedulers with different pod selectors
   
3. Custom schedulers
   - For specialized workloads (ML, batch, etc.)
```

---

## Controller Manager

### **Reconciliation Loop Pattern**

Every controller follows the same fundamental pattern:

```go
// Pseudo-code: Generic Controller Pattern
func (c *Controller) Run(stopCh <-chan struct{}) {
    for {
        select {
        case <-stopCh:
            return
        default:
            // 1. Get desired state from API Server
            desiredState := c.getDesiredState()
            
            // 2. Observe actual state
            actualState := c.observeActualState()
            
            // 3. Calculate diff
            diff := desiredState.Compare(actualState)
            
            // 4. Take corrective action
            if diff.NotEqual() {
                actions := c.calculateActions(diff)
                c.executeActions(actions)
            }
            
            // 5. Update status
            c.updateStatus(actualState)
            
            // 6. Sleep (reconciliation period)
            time.Sleep(30 * time.Second)
        }
    }
}
```

**Key Principles:**

1. **Idempotent**: Applying same desired state multiple times = same result
2. **Level-Triggered**: Operates on current state, not events
3. **Eventually Consistent**: May take several loops to converge
4. **Autonomous**: Each controller operates independently

### **Built-in Controllers**

**1. Deployment Controller**

```
Responsibilities:
- Manages ReplicaSets
- Implements rolling updates
- Handles rollback
- Scales replicas

Reconciliation Logic:
desired := deployment.spec.replicas
actual := len(replicaSet.pods)

if actual < desired:
    create (desired - actual) pods
elif actual > desired:
    delete (actual - desired) pods
    
if deployment.spec.template changed:
    create new ReplicaSet
    scale up new, scale down old (rolling update)
```

**Rolling Update Process:**

```
Initial State:
- Deployment: app-v1, replicas=10
- ReplicaSet-v1: 10 pods running

Update Triggered (app-v1 → app-v2):

T+0s:  Create ReplicaSet-v2
       
T+1s:  Scale up ReplicaSet-v2: 0 → 1 pod
       Wait for pod to be Ready
       
T+10s: Pod v2-0 Ready
       Scale down ReplicaSet-v1: 10 → 9 pods
       
T+11s: Scale up ReplicaSet-v2: 1 → 2 pods
       
T+20s: Pod v2-1 Ready
       Scale down ReplicaSet-v1: 9 → 8 pods
       
... (continues until all pods replaced)

T+120s: ReplicaSet-v2: 10 pods (all Ready)
        ReplicaSet-v1: 0 pods (scaled down)

Parameters:
maxSurge: 25%  # Max extra pods during update
maxUnavailable: 25%  # Max unavailable pods during update
```

**2. ReplicaSet Controller**

```yaml
# Simpler than Deployment - just maintains pod count
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21

# Controller logic:
every 30 seconds:
    desired = 3
    actual = count(pods with label app=nginx)
    
    if actual < desired:
        create (desired - actual) pods
    elif actual > desired:
        delete (actual - desired) pods
```

**3. Node Controller**

```
Responsibilities:
- Monitor node health
- Evict pods from unhealthy nodes
- Update node conditions

Health Check Process:

T+0s:   kubelet sends heartbeat (NodeStatus update)
T+10s:  kubelet sends heartbeat
T+20s:  kubelet sends heartbeat
...

Failure Detection:
T+0s:   kubelet stops sending heartbeats
T+40s:  Node Controller: No heartbeat for 40s
        Node condition: Ready → Unknown
        
T+5m:   Node Controller: Unknown for 5 minutes
        Mark pods on node for eviction
        Pod status: Running → Terminating
        
T+5m10s: Scheduler reschedules pods to healthy nodes

Parameters:
--node-monitor-period=5s          # Check nodes every 5s
--node-monitor-grace-period=40s   # Tolerate 40s of missed heartbeats
--pod-eviction-timeout=5m         # Wait 5m before evicting pods
```

**Node Conditions:**

```yaml
status:
  conditions:
  - type: Ready
    status: "True"  # Node healthy and ready
    reason: KubeletReady
    
  - type: MemoryPressure
    status: "False"  # No memory pressure
    
  - type: DiskPressure
    status: "False"  # No disk pressure
    
  - type: PIDPressure
    status: "False"  # No process ID exhaustion
    
  - type: NetworkUnavailable
    status: "False"  # Network configured correctly
```

**4. StatefulSet Controller**

```
Special Responsibilities:
- Ordered deployment (pod-0, then pod-1, then pod-2)
- Ordered termination (pod-2, then pod-1, then pod-0)
- Stable network identity (pod-name.service.namespace)
- Persistent volume claims (per-pod storage)

Reconciliation Logic:

Scale Up (replicas: 2 → 3):
1. Check pod-0 and pod-1 are Running and Ready
2. Create pod-2
3. Wait for pod-2 to be Running and Ready
4. Done

Scale Down (replicas: 3 → 2):
1. Delete pod-2 (highest ordinal)
2. Wait for pod-2 to terminate completely
3. Done (do NOT delete PVC!)

Update (rolling update):
1. Delete pod-2 (highest ordinal first)
2. Wait for pod-2 to terminate
3. Create new pod-2 with updated spec
4. Wait for pod-2 to be Ready
5. Delete pod-1
... (continues in reverse order)
```

**5. Job Controller**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-processing
spec:
  completions: 10  # Total successful completions needed
  parallelism: 3   # Max pods running simultaneously
  backoffLimit: 4  # Max retries on failure
  template:
    spec:
      restartPolicy: Never  # Don't restart failed pods
      containers:
      - name: processor
        image: data-processor:v1

# Controller behavior:
Running pods: 0, Completed: 0, Failed: 0

T+0s:   Create 3 pods (parallelism=3)
T+30s:  Pod-1 completes (Completed: 1)
        Create pod-4 (maintain parallelism)
T+45s:  Pod-2 fails (Failed: 1)
        Create pod-5 (retry, backoff applied)
T+60s:  Pod-3 completes (Completed: 2)
        Create pod-6
...
T+300s: Completed: 10 (target reached)
        Job status: Complete
        Do not create more pods
```

**6. CronJob Controller**

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  concurrencyPolicy: Forbid  # Don't run concurrent jobs
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:v1
          restartPolicy: OnFailure

# Controller logic:
every minute:
    now = time.Now()
    if schedule_matches(now):
        if concurrencyPolicy == "Forbid" and job_running():
            skip  # Don't create new job
        else:
            create Job from jobTemplate
    
    cleanup_old_jobs(successfulJobsHistoryLimit, failedJobsHistoryLimit)
```

---

## Cloud Controller Manager

### **Purpose & Architecture**

The Cloud Controller Manager (CCM) abstracts cloud-provider-specific code from core Kubernetes. Before CCM, cloud logic was embedded in kube-controller-manager (tight coupling).

```
┌────────────────────────────────────────────┐
│ Cloud Controller Manager                   │
│                                            │
│  ┌──────────────────────────────────────┐ │
│  │ Node Controller                      │ │
│  │ - Node discovery                     │ │
│  │ - Node deletion                      │ │
│  │ - Node labeling (zone, region, etc.)│ │
│  └──────────────────────────────────────┘ │
│                                            │
│  ┌──────────────────────────────────────┐ │
│  │ Route Controller                     │ │
│  │ - Configure cloud routes             │ │
│  │ - Pod CIDR routing                   │ │
│  └──────────────────────────────────────┘ │
│                                            │
│  ┌──────────────────────────────────────┐ │
│  │ Service Controller                   │ │
│  │ - LoadBalancer provisioning          │ │
│  │ - External IP allocation             │ │
│  └──────────────────────────────────────┘ │
└────────────────────────────────────────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │   Cloud Provider API  │
        │   (AWS, GCP, Azure)   │
        └───────────────────────┘
```

### **Node Controller (Cloud-Specific)**

```
Responsibilities:
1. Node Discovery: Query cloud API for instances
2. Node Initialization: Label nodes with cloud metadata
3. Node Deletion: Remove node objects when instances terminate

Workflow:

T+0s:   Cloud admin creates EC2 instance
        Instance boots, kubelet starts
        
T+10s:  kubelet registers with API Server
        Node object created (minimal info)
        
T+15s:  CCM Node Controller detects new node
        Query AWS API for instance details
        
T+20s:  Update node with cloud metadata:
        labels:
          topology.kubernetes.io/region: us-east-1
          topology.kubernetes.io/zone: us-east-1a
          node.kubernetes.io/instance-type: m5.xlarge
          failure-domain.beta.kubernetes.io/zone: us-east-1a
        
        providerID: aws:///us-east-1a/i-1234567890abcdef0

Instance Termination:
T+0s:   Cloud admin terminates EC2 instance
T+10s:  kubelet stops sending heartbeats
T+50s:  Node Controller marks node NotReady
T+5m:   CCM queries AWS API
        Instance state: "terminated"
T+5m10s: CCM deletes Node object from API Server
```

### **Service Controller**

```yaml
# LoadBalancer Service triggers CCM
apiVersion: v1
kind: Service
metadata:
  name: web-public
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"  # Network LB
    service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080

# CCM Workflow:
T+0s:   Service created
T+1s:   CCM detects type=LoadBalancer
        Call AWS API: CreateLoadBalancer
        
T+2s:   AWS begins provisioning NLB
        (takes 1-3 minutes typically)
        
T+120s: AWS NLB ready
        External IP: 52.45.67.89
        
T+121s: CCM updates Service:
        status:
          loadBalancer:
            ingress:
            - hostname: a1234.us-east-1.elb.amazonaws.com
              ip: 52.45.67.89
              
T+122s: CCM configures NLB target group
        Adds node IPs as targets
        Health checks configured

Service Deletion:
T+0s:   kubectl delete service web-public
T+1s:   CCM detects deletion
        Call AWS API: DeleteLoadBalancer
T+60s:  AWS completes deletion
        Resources cleaned up
```

**Multi-Cloud Annotations:**

```yaml
# AWS-specific
annotations:
  service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
  service.beta.kubernetes.io/aws-load-balancer-internal: "true"
  service.beta.kubernetes.io/aws-load-balancer-ssl-cert: "arn:aws:acm:..."

# GCP-specific  
annotations:
  cloud.google.com/load-balancer-type: "Internal"
  networking.gke.io/load-balancer-type: "Internal"

# Azure-specific
annotations:
  service.beta.kubernetes.io/azure-load-balancer-internal: "true"
  service.beta.kubernetes.io/azure-dns-label-name: "myapp"
```

### **Route Controller**

```
Purpose: Configure cloud routes for pod networking

Scenario: 3 nodes with pod CIDR ranges
- node-1: 10.244.1.0/24 (pods 10.244.1.1 - 10.244.1.254)
- node-2: 10.244.2.0/24 (pods 10.244.2.1 - 10.244.2.254)
- node-3: 10.244.3.0/24 (pods 10.244.3.1 - 10.244.3.254)

Without Route Controller:
- Pod on node-1 (10.244.1.5) cannot reach pod on node-2 (10.244.2.8)
- No route exists in VPC routing table

With Route Controller:
- CCM creates VPC routes:
  - 10.244.1.0/24 → node-1 ENI
  - 10.244.2.0/24 → node-2 ENI
  - 10.244.3.0/24 → node-3 ENI
  
- Pod traffic routes correctly through VPC

AWS Example:
aws ec2 create-route \
  --route-table-id rtb-12345 \
  --destination-cidr-block 10.244.1.0/24 \
  --network-interface-id eni-node1

Note: Many CNI plugins handle routing themselves (Calico, Cilium)
      Route Controller less commonly used in modern clusters
```

---

## High Availability Patterns

### **Multi-Master Architecture**

```
Production HA Setup (3 Masters):

                  ┌─────────────┐
                  │Load Balancer│ (VIP: 10.0.0.100)
                  └──────┬──────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
         ▼               ▼               ▼
    ┌─────────┐     ┌─────────┐     ┌─────────┐
    │Master-1 │     │Master-2 │     │Master-3 │
    │         │     │         │     │         │
    │ API Srv │     │ API Srv │     │ API Srv │
    │ Sched   │     │ Sched   │     │ Sched   │
    │ CtrlMgr │     │ CtrlMgr │     │ CtrlMgr │
    │ CCM     │     │ CCM     │     │ CCM     │
    └────┬────┘     └────┬────┘     └────┬────┘
         │               │               │
         └───────────────┼───────────────┘
                         │
                         ▼
                  ┌─────────────┐
                  │etcd Cluster │
                  │(3 or 5 nodes)
                  └─────────────┘

Components:
- API Servers: Stateless, all active (load balanced)
- Scheduler: Active-standby (leader election)
- Controller Manager: Active-standby (leader election)
- CCM: Active-standby (leader election)
- etcd: Active-active (Raft consensus)
```

### **Leader Election**

```yaml
# Schedulers use leader election (only one active)
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-scheduler-leader
  namespace: kube-system
data:
  # Lease holder information
  holderIdentity: "master-1_abc123"
  leaseDurationSeconds: "15"
  acquireTime: "2025-01-04T10:30:00Z"
  renewTime: "2025-01-04T10:30:14Z"

# Process:
1. Scheduler-1 (master-1) acquires lease
2. Scheduler-1 becomes active, starts scheduling
3. Scheduler-2, Scheduler-3 remain standby
4. Scheduler-1 renews lease every 10s (2/3 of duration)
5. If Scheduler-1 fails to renew (crash):
   - Lease expires after 15s
   - Scheduler-2 acquires lease
   - Scheduler-2 becomes active
   - Failover time: ~15-20 seconds
```

**Leader Election Parameters:**

```bash
# kube-scheduler flags
--leader-elect=true
--leader-elect-lease-duration=15s  # How long lease is valid
--leader-elect-renew-deadline=10s  # Deadline to renew before expiry
--leader-elect-retry-period=2s     # Retry interval if not leader

# Similar for kube-controller-manager and cloud-controller-manager
```

### **Failure Scenarios & Recovery**

**Scenario 1: Single API Server Fails**

```
Initial: 3 API Servers (master-1, master-2, master-3)

T+0s:   master-1 API Server crashes
T+1s:   Load balancer detects failure (health check)
T+2s:   Load balancer removes master-1 from rotation
        Requests route to master-2 and master-3
        
Impact: Zero downtime (seamless failover)
        Capacity: 66% (2/3 API Servers)
        
Recovery:
- Restart API Server on master-1
- Health check passes
- Load balancer adds master-1 back
- Capacity: 100%
```

**Scenario 2: etcd Node Fails (3-node cluster)**

```
Initial: 3 etcd nodes (quorum = 2)

T+0s:   etcd-1 crashes
T+150ms: etcd-2 and etcd-3 detect missing heartbeats
T+300ms: New leader election (etcd-2 becomes leader)
        Cluster operational with 2/3 nodes
        
Impact: Zero downtime (quorum maintained)
        Write latency: Slightly higher (2-node consensus)
        
Tolerable: 1 node failure (quorum: 2/3)
Critical: 2 node failures (quorum lost, cluster unavailable)
```

**Scenario 3: Complete Master Failure**

```
Disaster: Entire master-1 host fails
- API Server down
- Scheduler down (was leader)
- Controller Manager down
- etcd-1 down

Recovery Timeline:

T+0s:   master-1 complete failure
T+2s:   Load balancer removes master-1 API Server
        Requests failover to master-2, master-3
T+15s:  Scheduler leader lease expires
        Scheduler on master-2 acquires lease
        Scheduling resumes
T+15s:  Controller Manager leader lease expires
        Controller Manager on master-2 acquires lease
        Reconciliation resumes
T+300ms: etcd-2 elected as leader
        etcd cluster operational (2/3 quorum)

Total Downtime:
- API Server: 2 seconds
- Scheduler: 15 seconds (no new pod scheduling)
- Controllers: 15 seconds (no reconciliation)
- etcd: 300ms

Data Loss: Zero (etcd replicated)
Running Pods: Unaffected (data plane continues)
```

### **Disaster Recovery**

**Backup Strategy:**

```bash
#!/bin/bash
# Complete control plane backup script

# 1. etcd snapshot
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%Y%m%d-%H%M%S).db

# 2. Kubernetes certificates
tar -czf /backup/k8s-certs-$(date +%Y%m%d).tar.gz \
  /etc/kubernetes/pki/

# 3. Control plane manifests
tar -czf /backup/k8s-manifests-$(date +%Y%m%d).tar.gz \
  /etc/kubernetes/manifests/

# 4. API Server configuration
kubectl get all --all-namespaces -o yaml > /backup/cluster-state-$(date +%Y%m%d).yaml

# 5. Upload to off-site storage
aws s3 sync /backup/ s3://k8s-disaster-recovery/$(hostname)/
```

**Recovery Procedure:**

```bash
# Catastrophic failure: Rebuild control plane from scratch

# 1. Restore etcd
etcdctl snapshot restore /backup/etcd-latest.db \
  --data-dir=/var/lib/etcd-restored

# 2. Restore certificates
tar -xzf /backup/k8s-certs-latest.tar.gz -C /

# 3. Start control plane components
systemctl start etcd
systemctl start kube-apiserver
systemctl start kube-controller-manager
systemctl start kube-scheduler

# 4. Verify cluster state
kubectl get nodes
kubectl get pods --all-namespaces

# 5. Nodes and pods should auto-reconcile
# kubelet reconnects and re-registers nodes
# Controllers recreate missing resources
```

### **Split-Brain Prevention**

```
etcd uses Raft consensus - mathematically impossible to have split-brain

Scenario: Network partition

       Master-1, Master-2  |  Master-3
       (etcd-1, etcd-2)    |  (etcd-3)
                           |
         Partition A       |  Partition B
         2 nodes           |  1 node

Raft Behavior:
- Partition A: Has quorum (2/3)
  - Continues accepting writes
  - etcd-1 or etcd-2 is leader
  
- Partition B: No quorum (1/3)
  - Becomes read-only
  - Cannot elect leader
  - Rejects all write operations

When Partition Heals:
- etcd-3 reconnects
- etcd-3 syncs from leader (etcd-1 or etcd-2)
- Cluster returns to normal

Result: No split-brain, consistency maintained
```

**Production Recommendations:**

```
1. Minimum 3 masters (tolerate 1 failure)
2. Prefer 5 masters for large/critical clusters (tolerate 2 failures)
3. Spread masters across availability zones
4. Dedicated etcd nodes (separate from API Servers)
5. Monitor leader elections (frequent elections = problem)
6. Automated backups (every 30 minutes minimum)
7. Test disaster recovery regularly (quarterly)
8. Document runbooks for common failure scenarios
```

---

## Summary

**Key Architectural Principles:**

1. **Separation of Concerns**: Control plane (decisions) vs data plane (execution)
2. **Declarative Model**: Users specify desired state, controllers reconcile
3. **API-Centric**: All interactions through API Server (single entry point)
4. **Eventually Consistent**: System converges to desired state over time
5. **Leader Election**: Active-standby for schedulers and controllers
6. **Strong Consistency**: etcd ensures data integrity via Raft consensus

**Critical Components:**

- **API Server**: Gateway, authentication, authorization, admission control
- **etcd**: Distributed key-value store, source of truth, Raft consensus
- **Scheduler**: Two-phase algorithm (filtering + scoring), affinity/anti-affinity
- **Controllers**: Reconciliation loops, autonomous, idempotent
- **CCM**: Cloud provider abstraction (LoadBalancer, node discovery)

**Production Patterns:**

- Always run HA control plane (3+ masters)
- Use leader election for schedulers and controllers
- Monitor etcd health (disk latency #1 priority)
- Automate backups (etcd snapshots every 30min)
- Test disaster recovery procedures
- Scale API Servers horizontally under load

**Performance Considerations:**

- API Server latency: 20-100ms per request
- Scheduler latency: 15ms-2.5s (depends on cluster size)
- etcd write latency: 10-30ms (disk-bound)
- Controller reconciliation: 30s default period
