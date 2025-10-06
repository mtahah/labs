# 💾 Kubernetes Storage & State Management

> **Expert-Level Technical Reference**  
> Deep dive into persistent storage, StatefulSets, CSI drivers, and production data management patterns.

---

## 📋 Table of Contents

1. [Storage Architecture](#storage-architecture)
2. [Persistent Volume Lifecycle](#persistent-volume-lifecycle)
3. [StatefulSets Deep Dive](#statefulsets-deep-dive)
4. [CSI Driver Architecture](#csi-driver-architecture)
5. [Advanced Operations](#advanced-operations)
6. [Production Patterns](#production-patterns)

---

## Storage Architecture

### **Ephemeral vs Persistent Storage**

```
Ephemeral Storage (Pod-scoped):
- Lives with pod lifecycle
- Deleted when pod deleted
- Types: emptyDir, ConfigMap, Secret

Persistent Storage (Independent):
- Survives pod deletion
- Managed separately from pods
- Types: PersistentVolume (cloud disks, NFS, etc.)
```

### **Storage Abstraction Layers**

```
Application Layer:
    "I need 100GB storage"
           ↓
┌──────────────────────────────┐
│ PersistentVolumeClaim (PVC)  │ (Request)
│ - Requests storage from pool │
│ - Specifies: size, access    │
└──────────┬───────────────────┘
           ↓
┌──────────────────────────────┐
│ PersistentVolume (PV)        │ (Actual storage)
│ - Represents actual storage  │
│ - Bound to single PVC        │
└──────────┬───────────────────┘
           ↓
┌──────────────────────────────┐
│ StorageClass                 │ (Provisioning policy)
│ - Defines how to provision   │
│ - Parameters (type, IOPS)    │
└──────────┬───────────────────┘
           ↓
┌──────────────────────────────┐
│ CSI Driver                   │ (Implementation)
│ - Talks to storage backend   │
│ - Creates/attaches volumes   │
└──────────┬───────────────────┘
           ↓
┌──────────────────────────────┐
│ Storage Backend              │ (Physical)
│ - AWS EBS, GCP PD, NFS, etc. │
└──────────────────────────────┘
```

---

## Persistent Volume Lifecycle

### **Manual Provisioning (Static)**

```yaml
# 1. Admin creates PersistentVolume
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-manual
spec:
  capacity:
    storage: 100Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:  # Don't use in production!
    path: /mnt/data

---
# 2. User creates PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-manual
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
  storageClassName: manual

# 3. Kubernetes binds PVC to PV
# PVC Status: Bound
# PV Status: Bound

---
# 4. Pod uses PVC
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: pvc-manual
```

### **Dynamic Provisioning (Recommended)**

```yaml
# 1. StorageClass defines provisioner
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: ebs.csi.aws.com  # AWS EBS CSI driver
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  encrypted: "true"
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete

---
# 2. User creates PVC (no manual PV needed!)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-dynamic
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: fast-ssd
  resources:
    requests:
      storage: 100Gi

# 3. CSI driver provisions volume automatically
# - Creates EBS volume in AWS
# - Creates PersistentVolume object
# - Binds PVC to PV

---
# 4. Pod uses PVC (same as manual)
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: pvc-dynamic
```

### **Access Modes**

```yaml
accessModes:
- ReadWriteOnce (RWO)   # Single node read/write
- ReadOnlyMany (ROX)    # Multiple nodes read-only
- ReadWriteMany (RWX)   # Multiple nodes read/write
- ReadWriteOncePod (RWOP)  # Single pod (K8s 1.29+)
```

**Access Mode Support by Storage Type:**

| Storage | RWO | ROX | RWX | Use Case |
|---------|-----|-----|-----|----------|
| **AWS EBS** | ✅ | ❌ | ❌ | Databases, single-node apps |
| **GCP Persistent Disk** | ✅ | ✅ | ❌ | Databases, read replicas |
| **Azure Disk** | ✅ | ❌ | ❌ | Databases, single-node apps |
| **AWS EFS** | ✅ | ✅ | ✅ | Shared files, multi-pod access |
| **NFS** | ✅ | ✅ | ✅ | Legacy apps, shared storage |
| **CephFS** | ✅ | ✅ | ✅ | Distributed storage |

**Critical Distinction:**

```
ReadWriteOnce = One NODE, not one POD!

Scenario:
- PVC with RWO
- Pod A on node-1 uses PVC ✅
- Pod B on node-1 uses same PVC ✅ (same node!)
- Pod C on node-2 tries to use PVC ❌ (different node!)

For single-pod access, use ReadWriteOncePod (K8s 1.29+)
```

### **Volume Binding Modes**

```yaml
# Immediate (Legacy - Problematic)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: immediate-binding
provisioner: ebs.csi.aws.com
volumeBindingMode: Immediate

# Problem: Volume provisioned before pod scheduled
# If volume in us-east-1a, pod scheduled to us-east-1b → fails!

---
# WaitForFirstConsumer (Recommended)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: topology-aware
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer

# Benefit: Volume provisioned in same zone as pod
# Scheduler considers volume topology when placing pod
```

**Timeline Comparison:**

```
Immediate Binding:
T+0s:  PVC created
T+1s:  Volume provisioned in random AZ (us-east-1a)
T+5s:  Pod scheduled to node in us-east-1b
T+6s:  Volume attachment fails (cross-AZ EBS not supported)
Result: Pod stuck in ContainerCreating

WaitForFirstConsumer:
T+0s:  PVC created (status: Pending)
T+5s:  Pod scheduled to node in us-east-1a
T+6s:  Volume provisioned in us-east-1a (same AZ as node)
T+10s: Volume attached to node
T+15s: Pod running
Result: Success! Volume and pod co-located
```

---

## StatefulSets Deep Dive

### **Why Deployments Fail for Stateful Apps**

```yaml
# Deployment (Stateless)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: nginx
        image: nginx

# Problems for stateful apps:
# 1. Random pod names (web-abc123, web-def456)
# 2. No stable network identity
# 3. All pods use same PVC (or no persistence)
# 4. No ordering (all start simultaneously)
```

### **StatefulSet Guarantees**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres  # Required!
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 100Gi

# Guarantees:
# 1. Stable names: postgres-0, postgres-1, postgres-2
# 2. Stable DNS: postgres-0.postgres.default.svc.cluster.local
# 3. Ordered deployment: 0 → 1 → 2
# 4. Ordered termination: 2 → 1 → 0
# 5. Per-pod storage: data-postgres-0, data-postgres-1, data-postgres-2
```

### **Deployment Behavior**

```
Scale Up (replicas: 2 → 3):

T+0s:  Ensure postgres-0 is Running and Ready
T+1s:  postgres-0 ready ✓
T+2s:  Ensure postgres-1 is Running and Ready
T+3s:  postgres-1 ready ✓
T+4s:  Create postgres-2
T+5s:  PVC data-postgres-2 created
T+10s: Volume provisioned
T+15s: Volume attached
T+20s: Pod postgres-2 Running
T+30s: postgres-2 Ready (passes readiness probe)
T+31s: StatefulSet reconciliation complete

Scale Down (replicas: 3 → 1):

T+0s:  Delete postgres-2 (highest ordinal first)
T+1s:  Pod status: Terminating
T+31s: Grace period expires, pod deleted
T+32s: PVC data-postgres-2 NOT deleted (persists!)
T+33s: Delete postgres-1
T+64s: postgres-1 deleted
T+65s: Only postgres-0 remains

Note: PVCs remain! Must delete manually to reclaim storage
```

### **Headless Service (Required)**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  clusterIP: None  # Headless service
  selector:
    app: postgres
  ports:
  - port: 5432

# DNS records created:
# postgres.default.svc.cluster.local → All pod IPs
# postgres-0.postgres.default.svc.cluster.local → postgres-0 IP
# postgres-1.postgres.default.svc.cluster.local → postgres-1 IP
# postgres-2.postgres.default.svc.cluster.local → postgres-2 IP

# Application can target specific pod:
psql -h postgres-0.postgres.default.svc.cluster.local
```

### **Update Strategies**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  updateStrategy:
    type: RollingUpdate  # or OnDelete
    rollingUpdate:
      partition: 0  # Update pods with ordinal >= partition

# RollingUpdate (default):
# - Update in reverse order (N → 0)
# - Wait for each pod to be Ready before next

# Example: Update image postgres:14 → postgres:15
T+0s:  Delete postgres-2
T+30s: New postgres-2 created with postgres:15
T+60s: postgres-2 Ready
T+61s: Delete postgres-1
T+91s: New postgres-1 created with postgres:15
T+120s: postgres-1 Ready
T+121s: Delete postgres-0
T+151s: New postgres-0 created with postgres:15
T+180s: postgres-0 Ready
T+181s: Update complete

# Partition (staged rollout):
partition: 2  # Only update pods >= 2
Result: postgres-2 updated, postgres-0 and postgres-1 unchanged
Use: Canary testing (test new version on one pod)
```

---

## CSI Driver Architecture

### **Components**

```
Control Plane (Cluster-wide):
┌────────────────────────────┐
│ CSI Controller             │
│ - CreateVolume()           │
│ - DeleteVolume()           │
│ - ControllerPublishVolume()│
│ - CreateSnapshot()         │
└────────────────────────────┘

Data Plane (Per Node):
┌────────────────────────────┐
│ CSI Node (DaemonSet)       │
│ - NodeStageVolume()        │
│ - NodePublishVolume()      │
│ - NodeUnpublishVolume()    │
└────────────────────────────┘
```

### **Volume Provisioning Flow**

```
T+0s:  User creates PVC
T+1s:  PVC Controller triggers CSI provisioner
T+2s:  CSI Controller: CreateVolume RPC
       Request: {capacity: 100Gi, parameters: {type: gp3}}
       
T+3s:  CSI driver calls cloud API:
       aws ec2 create-volume --size 100 --type gp3
       
T+30s: Cloud provisions volume (vol-abc123)
       
T+31s: CSI driver creates PersistentVolume:
       apiVersion: v1
       kind: PersistentVolume
       spec:
         capacity: {storage: 100Gi}
         csi:
           driver: ebs.csi.aws.com
           volumeHandle: vol-abc123
         nodeAffinity:
           required:
             nodeSelectorTerms:
             - matchExpressions:
               - key: topology.ebs.csi.aws.com/zone
                 operator: In
                 values: [us-east-1a]
       
T+32s: PVC bound to PV
T+33s: Pod scheduled to node in us-east-1a
T+34s: kubelet calls CSI Node: NodeStageVolume
T+35s: CSI driver attaches volume to node:
       aws ec2 attach-volume --volume-id vol-abc123 --instance-id i-node5
       
T+40s: Volume appears as /dev/xvdf
T+41s: CSI Node: Format filesystem (if first use)
       mkfs.ext4 /dev/xvdf
       
T+42s: CSI Node: NodePublishVolume
       mount /dev/xvdf /var/lib/kubelet/pods/.../volumes/...
       
T+43s: kubelet bind mounts into container
T+44s: Pod running with persistent volume
```

---

## Advanced Operations

### **Volume Expansion**

```yaml
# 1. StorageClass must allow expansion
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: expandable
provisioner: ebs.csi.aws.com
allowVolumeExpansion: true  # Required!

---
# 2. Edit PVC to increase size
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-postgres-0
spec:
  resources:
    requests:
      storage: 200Gi  # Increased from 100Gi

# 3. Expansion process (automatic)
T+0s:  PVC update detected
T+1s:  CSI Controller: ControllerExpandVolume
       aws ec2 modify-volume --volume-id vol-abc123 --size 200
       
T+30s: Cloud completes expansion
T+31s: PVC status: FileSystemResizePending
       (Block device is 200GB, filesystem still 100GB)
       
T+32s: Restart pod (triggers filesystem resize)
       kubectl rollout restart statefulset postgres
       
T+33s: kubelet calls CSI Node: NodeExpandVolume
T+34s: CSI Node: resize2fs /dev/xvdf
T+35s: Filesystem expanded to 200GB
T+36s: PVC status: Bound (complete)

Total time: 36 seconds
Downtime: ~20 seconds (pod restart)
```

**Online Resize (No Downtime):**

```bash
# For filesystems supporting online resize (ext4, xfs)
kubectl exec postgres-0 -- resize2fs /dev/xvdf
# Filesystem expands without pod restart
# Downtime: 0 seconds
```

### **Volume Snapshots**

```yaml
# 1. VolumeSnapshotClass
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: ebs-snapshots
driver: ebs.csi.aws.com
deletionPolicy: Retain

---
# 2. Create snapshot
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: postgres-backup-20250104
spec:
  volumeSnapshotClassName: ebs-snapshots
  source:
    persistentVolumeClaimName: data-postgres-0

# Process:
T+0s:  VolumeSnapshot created
T+1s:  CSI Controller: CreateSnapshot
       aws ec2 create-snapshot --volume-id vol-abc123
       
T+60s: AWS completes snapshot (snap-xyz789)
T+61s: VolumeSnapshot status: ReadyToUse

---
# 3. Restore from snapshot
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-restored
spec:
  dataSource:
    name: postgres-backup-20250104
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 100Gi

# CSI creates new volume from snapshot
# New PVC bound to restored volume
```

### **Backup Strategies**

```yaml
# Automated snapshots via CronJob
apiVersion: batch/v1
kind: CronJob
metadata:
  name: postgres-backup
  namespace: production
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: snapshot-creator
          containers:
          - name: create-snapshot
            image: bitnami/kubectl
            command:
            - /bin/sh
            - -c
            - |
              TIMESTAMP=$(date +%Y%m%d-%H%M%S)
              cat <<EOF | kubectl apply -f -
              apiVersion: snapshot.storage.k8s.io/v1
              kind: VolumeSnapshot
              metadata:
                name: postgres-backup-$TIMESTAMP
              spec:
                volumeSnapshotClassName: ebs-snapshots
                source:
                  persistentVolumeClaimName: data-postgres-0
              EOF
              
              # Cleanup old snapshots (keep last 7)
              kubectl get volumesnapshot -o json | \
                jq -r '.items | sort_by(.metadata.creationTimestamp) | reverse | .[7:] | .[].metadata.name' | \
                xargs -I {} kubectl delete volumesnapshot {}
          restartPolicy: OnFailure

# Production backup layers:
# Layer 1: Volume snapshots (every 6h, retain 7 days)
# Layer 2: Application backups (pg_dump daily, retain 30 days)
# Layer 3: Off-site replication (continuous, same as primary)
```

---

## Production Patterns

### **Performance Tuning**

```yaml
# High-performance database storage
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: database-storage
provisioner: ebs.csi.aws.com
parameters:
  type: io2  # Provisioned IOPS
  iops: "20000"  # 20K IOPS
  throughput: "1000"  # 1000 MB/s
  encrypted: "true"
  kmsKeyId: "arn:aws:kms:..."
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer

# Cost: ~$1,425/month per 1TB volume
# Use case: Production databases (PostgreSQL, MySQL)

---
# Cost-effective general purpose
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: general-purpose
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "3000"  # Baseline
  throughput: "125"  # Baseline
  encrypted: "true"
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer

# Cost: ~$80/month per 1TB volume
# Use case: Most applications, logs, cache
```

### **Multi-AZ High Availability**

```yaml
# StatefulSet with zone spreading
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres-ha
spec:
  replicas: 3
  serviceName: postgres
  template:
    spec:
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: postgres
      # Result: postgres-0 in us-east-1a
      #         postgres-1 in us-east-1b
      #         postgres-2 in us-east-1c
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      storageClassName: fast-ssd  # WaitForFirstConsumer
      accessModes: [ReadWriteOnce]
      resources:
        requests:
          storage: 100Gi
# Each replica gets volume in same AZ
# Survives single AZ failure
```

### **Troubleshooting**

```bash
# Pod stuck in ContainerCreating
kubectl describe pod postgres-0
# Events:
# - FailedAttachVolume: Volume not in same AZ as node
# - FailedMount: Volume already attached to another node

# Check PVC status
kubectl get pvc data-postgres-0
# STATUS: Pending (volume not provisioned)
# STATUS: Bound (volume provisioned and bound)

# Check PV
kubectl get pv pvc-abc123 -o yaml
# nodeAffinity: Shows which zone volume is in

# Check StorageClass
kubectl get storageclass fast-ssd -o yaml
# volumeBindingMode: Should be WaitForFirstConsumer

# Force delete pod (node failure scenario)
kubectl delete pod postgres-0 --force --grace-period=0

# Orphaned PVCs (after scaling down)
kubectl get pvc --all-namespaces | grep -v Bound
# Delete manually: kubectl delete pvc data-postgres-2

# Volume not detaching (node unreachable)
# CSI driver will force detach after timeout
aws ec2 detach-volume --volume-id vol-abc123 --force
```

### **Cost Optimization**

```bash
# Monitor disk usage
kubectl exec postgres-0 -- df -h /var/lib/postgresql/data
# Filesystem  Size  Used Avail Use%
# /dev/xvdf   100G   45G   55G  45%

# Right-size volumes (don't over-provision)
# Current: 100GB, Used: 45GB
# Recommendation: 70GB (with 20GB buffer)

# Identify orphaned PVCs
kubectl get pvc --all-namespaces -o json | \
  jq -r '.items[] | select(.status.phase=="Bound") | 
    select(.spec.volumeName as $pv | 
    [env.PVS | contains($pv)] | any | not) | 
    "\(.metadata.namespace)/\(.metadata.name)"'

# Cost per month (AWS EBS gp3):
# 100GB × $0.08 = $8/month per PVC
# 10 orphaned PVCs = $80/month wasted
```

---

## Summary

**Key Concepts:**

- **PV/PVC Abstraction**: Separates storage provisioning from consumption
- **StatefulSets**: Provide stable identity, ordered deployment, persistent storage
- **CSI**: Pluggable storage interface for multiple backends
- **Dynamic Provisioning**: Automatic volume creation via StorageClass

**Production Best Practices:**

- Always use `WaitForFirstConsumer` for zonal storage
- Set `allowVolumeExpansion: true` on StorageClasses
- Implement automated snapshot backups
- Monitor and cleanup orphaned PVCs
- Right-size volumes based on actual usage
- Use topology constraints for multi-AZ HA
- Test disaster recovery procedures regularly

**Common Pitfalls:**

- Using `Immediate` binding mode (topology mismatches)
- Not specifying `volumeBindingMode` (defaults to Immediate)
- Forgetting PVCs persist after scaling down
- Over-provisioning storage (cost waste)
- No backup strategy (data loss risk)
- Not testing restore procedures

This completes the Storage & State Management deep dive.
