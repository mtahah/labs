# 🏢 Kubernetes Enterprise Use Cases & Production Operations

> **Expert-Level Technical Reference**  
> Enterprise patterns, operator development, production operations, and comprehensive best practices for running Kubernetes at scale.

---

## 📋 Table of Contents

1. [Enterprise Use Cases](#enterprise-use-cases)
2. [Operators & CRDs](#operators-crds)
3. [Production Operations](#production-operations)
4. [Observability](#observability)
5. [Disaster Recovery](#disaster-recovery)
6. [Best Practices](#best-practices)

---

## Enterprise Use Cases

### **Multi-Tenancy Patterns**

**Namespace-Level Isolation:**

```yaml
# Tenant namespace with quotas
apiVersion: v1
kind: Namespace
metadata:
  name: tenant-a
  labels:
    tenant: tenant-a
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: tenant-quota
  namespace: tenant-a
spec:
  hard:
    requests.cpu: "100"
    requests.memory: 200Gi
    limits.cpu: "200"
    limits.memory: 400Gi
    pods: "100"
    services.loadbalancers: "5"
    persistentvolumeclaims: "20"
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: tenant-isolation
  namespace: tenant-a
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          tenant: tenant-a
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          tenant: tenant-a
```

**Hierarchical Namespaces (HNC):**

```yaml
# Parent namespace
apiVersion: hnc.x-k8s.io/v1alpha2
kind: HierarchyConfiguration
metadata:
  name: hierarchy
  namespace: org-engineering
spec:
  parent: ""  # Root namespace

---
# Child namespace (inherits policies)
apiVersion: hnc.x-k8s.io/v1alpha2
kind: HierarchyConfiguration
metadata:
  name: hierarchy
  namespace: team-backend
spec:
  parent: org-engineering

# team-backend inherits:
# - ResourceQuotas from org-engineering
# - NetworkPolicies from org-engineering
# - RBAC roles from org-engineering
```

### **ML/AI Workloads**

**GPU Scheduling:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ml-training
spec:
  nodeSelector:
    accelerator: nvidia-tesla-v100
  tolerations:
  - key: nvidia.com/gpu
    operator: Exists
    effect: NoSchedule
  containers:
  - name: training
    image: tensorflow/tensorflow:latest-gpu
    resources:
      limits:
        nvidia.com/gpu: 4  # Request 4 GPUs
        cpu: 16
        memory: 64Gi
    volumeMounts:
    - name: dataset
      mountPath: /data
    - name: models
      mountPath: /models
  volumes:
  - name: dataset
    persistentVolumeClaim:
      claimName: ml-dataset
  - name: models
    persistentVolumeClaim:
      claimName: ml-models

# GPU Node pool configuration
# Node labels: accelerator=nvidia-tesla-v100
# Taints: nvidia.com/gpu=true:NoSchedule
# GPU operator installed for device plugin
```

**Distributed Training (Kubeflow):**

```yaml
apiVersion: kubeflow.org/v1
kind: PyTorchJob
metadata:
  name: distributed-training
spec:
  pytorchReplicaSpecs:
    Master:
      replicas: 1
      template:
        spec:
          containers:
          - name: pytorch
            image: pytorch/pytorch:latest
            resources:
              limits:
                nvidia.com/gpu: 1
    Worker:
      replicas: 4  # 4 worker nodes
      template:
        spec:
          containers:
          - name: pytorch
            image: pytorch/pytorch:latest
            resources:
              limits:
                nvidia.com/gpu: 1

# Result: 5 pods (1 master + 4 workers)
# Automatic distributed training setup
# MPI/Horovod integration
```

### **Batch Processing**

**Parallel Jobs:**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-processing
spec:
  completions: 100  # Process 100 items
  parallelism: 10   # 10 concurrent pods
  backoffLimit: 3
  template:
    spec:
      restartPolicy: OnFailure
      containers:
      - name: processor
        image: data-processor:v1
        env:
        - name: ITEM_INDEX
          valueFrom:
            fieldRef:
              fieldPath: metadata.labels['batch.kubernetes.io/job-completion-index']
        command:
        - /bin/sh
        - -c
        - |
          # Process item based on ITEM_INDEX
          process-item.sh $ITEM_INDEX

# Execution:
# - Creates 10 pods initially
# - As each completes, creates next until 100 total
# - Efficient for large batch processing
```

**Workflow Orchestration (Argo):**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  name: data-pipeline
spec:
  entrypoint: pipeline
  templates:
  - name: pipeline
    steps:
    - - name: extract
        template: extract-data
    - - name: transform
        template: transform-data
        arguments:
          artifacts:
          - name: raw-data
            from: "{{steps.extract.outputs.artifacts.data}}"
    - - name: load
        template: load-data
        arguments:
          artifacts:
          - name: processed-data
            from: "{{steps.transform.outputs.artifacts.data}}"
  
  - name: extract-data
    container:
      image: extractor:v1
      command: ["/extract.sh"]
    outputs:
      artifacts:
      - name: data
        path: /output/data.csv
  
  - name: transform-data
    inputs:
      artifacts:
      - name: raw-data
        path: /input/data.csv
    container:
      image: transformer:v1
      command: ["/transform.sh"]
    outputs:
      artifacts:
      - name: data
        path: /output/transformed.csv
  
  - name: load-data
    inputs:
      artifacts:
      - name: processed-data
        path: /input/data.csv
    container:
      image: loader:v1
      command: ["/load.sh"]
```

### **Platform Engineering**

**Internal Developer Platform:**

```yaml
# Self-service namespace provisioning
apiVersion: v1
kind: ConfigMap
metadata:
  name: platform-template
data:
  namespace.yaml: |
    apiVersion: v1
    kind: Namespace
    metadata:
      name: {{.TeamName}}
      labels:
        team: {{.TeamName}}
        cost-center: {{.CostCenter}}
    ---
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: quota
      namespace: {{.TeamName}}
    spec:
      hard:
        requests.cpu: "50"
        requests.memory: 100Gi
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: team-admin
      namespace: {{.TeamName}}
    subjects:
    - kind: Group
      name: {{.TeamName}}-admins
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: ClusterRole
      name: admin
      apiGroup: rbac.authorization.k8s.io

# Developer self-service portal
# 1. Submit request via web UI
# 2. Platform controller renders template
# 3. Applies to cluster
# 4. Team gets isolated namespace with quotas
```

---

## Operators & CRDs

### **Custom Resource Definitions**

```yaml
# CRD Definition
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.example.com
spec:
  group: example.com
  names:
    kind: Database
    listKind: DatabaseList
    plural: databases
    singular: database
    shortNames:
    - db
  scope: Namespaced
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            required: ["size", "version"]
            properties:
              size:
                type: string
                enum: ["small", "medium", "large"]
              version:
                type: string
                pattern: '^[0-9]+\.[0-9]+$'
              replicas:
                type: integer
                minimum: 1
                maximum: 5
              backup:
                type: object
                properties:
                  enabled:
                    type: boolean
                  schedule:
                    type: string
          status:
            type: object
            properties:
              phase:
                type: string
              endpoint:
                type: string
    additionalPrinterColumns:
    - name: Size
      type: string
      jsonPath: .spec.size
    - name: Version
      type: string
      jsonPath: .spec.version
    - name: Status
      type: string
      jsonPath: .status.phase
    - name: Age
      type: date
      jsonPath: .metadata.creationTimestamp

---
# Custom Resource Instance
apiVersion: example.com/v1
kind: Database
metadata:
  name: my-postgres
spec:
  size: medium
  version: "15.0"
  replicas: 3
  backup:
    enabled: true
    schedule: "0 2 * * *"
```

### **Operator Pattern**

**Reconciliation Loop:**

```go
// Pseudo-code: Operator reconciliation
func (r *DatabaseReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    // 1. Fetch the Database resource
    db := &examplev1.Database{}
    if err := r.Get(ctx, req.NamespacedName, db); err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }
    
    // 2. Get desired state
    desiredState := r.getDesiredState(db)
    
    // 3. Get actual state
    actualState := r.getActualState(ctx, db)
    
    // 4. Calculate diff
    if !reflect.DeepEqual(desiredState, actualState) {
        // 5. Reconcile differences
        if err := r.reconcileStatefulSet(ctx, db, desiredState); err != nil {
            return ctrl.Result{}, err
        }
        if err := r.reconcileService(ctx, db, desiredState); err != nil {
            return ctrl.Result{}, err
        }
        if err := r.reconcileBackup(ctx, db, desiredState); err != nil {
            return ctrl.Result{}, err
        }
    }
    
    // 6. Update status
    db.Status.Phase = "Ready"
    db.Status.Endpoint = fmt.Sprintf("%s.%s.svc:5432", db.Name, db.Namespace)
    if err := r.Status().Update(ctx, db); err != nil {
        return ctrl.Result{}, err
    }
    
    // 7. Requeue after 30 seconds
    return ctrl.Result{RequeueAfter: 30 * time.Second}, nil
}
```

**Production Operator Example:**

```yaml
# Operator creates:
# 1. StatefulSet for database
# 2. Service for connectivity
# 3. CronJob for backups
# 4. PVCs for storage

# When user creates Database CR:
apiVersion: example.com/v1
kind: Database
metadata:
  name: my-postgres
spec:
  size: medium
  version: "15.0"
  replicas: 3

# Operator automatically creates:
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-postgres
  ownerReferences:  # Garbage collection
  - apiVersion: example.com/v1
    kind: Database
    name: my-postgres
    uid: abc-123
spec:
  replicas: 3
  serviceName: my-postgres
  template:
    spec:
      containers:
      - name: postgres
        image: postgres:15.0
        resources:
          requests:  # Based on size: medium
            cpu: 2000m
            memory: 8Gi
---
apiVersion: v1
kind: Service
metadata:
  name: my-postgres
  ownerReferences:
  - apiVersion: example.com/v1
    kind: Database
    name: my-postgres
spec:
  clusterIP: None
  selector:
    app: my-postgres
  ports:
  - port: 5432
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: my-postgres-backup
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: postgres:15.0
            command: ["/backup.sh"]
```

### **Operator Best Practices**

```yaml
# 1. Idempotency
# Reconciliation should be safe to run multiple times

# 2. Status subresource
status:
  conditions:
  - type: Ready
    status: "True"
    lastTransitionTime: "2025-01-04T10:00:00Z"
  - type: Progressing
    status: "False"
  observedGeneration: 5  # Track which spec version we've processed

# 3. Finalizers for cleanup
metadata:
  finalizers:
  - example.com/database-finalizer

# Operator handles deletion:
if db.DeletionTimestamp != nil {
    // Cleanup external resources (cloud databases, etc.)
    cleanup(db)
    // Remove finalizer
    removeFinalizer(db)
}

# 4. Owner references for garbage collection
ownerReferences:
- apiVersion: example.com/v1
  kind: Database
  name: my-postgres
  uid: abc-123
  controller: true
  blockOwnerDeletion: true

# When Database deleted, all owned resources auto-deleted

# 5. Validation webhooks
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: database-validator
webhooks:
- name: validate.database.example.com
  rules:
  - operations: ["CREATE", "UPDATE"]
    apiGroups: ["example.com"]
    apiVersions: ["v1"]
    resources: ["databases"]
```

---

## Production Operations

### **GitOps with ArgoCD**

```yaml
# Application definition
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: production-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/company/k8s-manifests
    targetRevision: main
    path: apps/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true  # Delete resources not in Git
      selfHeal: true  # Revert manual changes
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m

# Git repository structure:
# apps/
#   production/
#     deployment.yaml
#     service.yaml
#     ingress.yaml
#   staging/
#     deployment.yaml
#     service.yaml

# Workflow:
# 1. Developer commits to Git
# 2. ArgoCD detects change
# 3. ArgoCD syncs to cluster
# 4. Automatic rollback if health checks fail
```

### **Progressive Delivery (Flagger)**

```yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: api
  namespace: production
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  service:
    port: 80
  analysis:
    interval: 1m
    threshold: 5  # Require 5 successful checks
    maxWeight: 50
    stepWeight: 10
    metrics:
    - name: request-success-rate
      thresholdRange:
        min: 99
      interval: 1m
    - name: request-duration
      thresholdRange:
        max: 500  # P99 < 500ms
      interval: 1m
    webhooks:
    - name: load-test
      url: http://flagger-loadtester/
      timeout: 5s
      metadata:
        cmd: "hey -z 1m -q 10 -c 2 http://api.production/"

# Automated canary process:
# 1. New deployment detected
# 2. Create canary deployment (10% traffic)
# 3. Run load tests, check metrics
# 4. If metrics pass: Increase to 20%
# 5. Repeat until 50% or failure
# 6. If success: Full rollout
# 7. If failure: Automatic rollback
```

---

## Observability

### **Metrics (Prometheus)**

```yaml
# ServiceMonitor for application
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: api-metrics
  namespace: production
spec:
  selector:
    matchLabels:
      app: api
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics

---
# PrometheusRule for alerts
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: api-alerts
spec:
  groups:
  - name: api
    interval: 30s
    rules:
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "High error rate on {{ $labels.pod }}"
    
    - alert: HighLatency
      expr: histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m])) > 1
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "P99 latency > 1s on {{ $labels.pod }}"
    
    - alert: PodCrashLooping
      expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Pod {{ $labels.pod }} crash looping"
```

### **Logging (ELK/Loki)**

```yaml
# Fluent Bit DaemonSet for log collection
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
  namespace: logging
spec:
  template:
    spec:
      containers:
      - name: fluent-bit
        image: fluent/fluent-bit:latest
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: config
          mountPath: /fluent-bit/etc/
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      - name: config
        configMap:
          name: fluent-bit-config

---
# Fluent Bit config
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush        5
        Daemon       Off
        Log_Level    info
    
    [INPUT]
        Name              tail
        Path              /var/log/containers/*.log
        Parser            docker
        Tag               kube.*
        Refresh_Interval  5
    
    [FILTER]
        Name                kubernetes
        Match               kube.*
        Kube_URL            https://kubernetes.default.svc:443
        Kube_CA_File        /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        Kube_Token_File     /var/run/secrets/kubernetes.io/serviceaccount/token
    
    [OUTPUT]
        Name  loki
        Match *
        Host  loki.logging.svc
        Port  3100
        Labels job=fluentbit
```

### **Distributed Tracing (Jaeger)**

```yaml
# Jaeger operator deployment
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: jaeger
  namespace: observability
spec:
  strategy: production
  storage:
    type: elasticsearch
    options:
      es:
        server-urls: http://elasticsearch.observability.svc:9200
  ingress:
    enabled: true

# Application instrumentation (example)
# Go: import "github.com/opentracing/opentracing-go"
# Python: from jaeger_client import Config
# Automatic: Service mesh (Istio) injects tracing
```

---

## Disaster Recovery

### **Backup Strategy**

```yaml
# Velero for cluster backups
apiVersion: v1
kind: ConfigMap
metadata:
  name: velero-backup-schedule
data:
  schedule.yaml: |
    apiVersion: velero.io/v1
    kind: Schedule
    metadata:
      name: daily-backup
      namespace: velero
    spec:
      schedule: "0 2 * * *"  # 2 AM daily
      template:
        includedNamespaces:
        - "*"
        excludedNamespaces:
        - kube-system
        - velero
        includedResources:
        - "*"
        includeClusterResources: true
        snapshotVolumes: true
        ttl: 720h  # Retain 30 days

# Restore from backup:
velero restore create --from-backup daily-backup-20250104
```

### **etcd Backup**

```bash
#!/bin/bash
# Automated etcd backup script

ETCDCTL_API=3 etcdctl snapshot save \
  /backup/etcd-$(date +%Y%m%d-%H%M%S).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Upload to S3
aws s3 cp /backup/etcd-*.db s3://k8s-backups/etcd/

# Cleanup old backups (keep last 30 days)
find /backup -name "etcd-*.db" -mtime +30 -delete

# Schedule via CronJob
apiVersion: batch/v1
kind: CronJob
metadata:
  name: etcd-backup
  namespace: kube-system
spec:
  schedule: "0 */6 * * *"  # Every 6 hours
  jobTemplate:
    spec:
      template:
        spec:
          hostNetwork: true
          containers:
          - name: backup
            image: k8s.gcr.io/etcd:3.5.9
            command: ["/backup.sh"]
            volumeMounts:
            - name: etcd-certs
              mountPath: /etc/kubernetes/pki/etcd
          volumes:
          - name: etcd-certs
            hostPath:
              path: /etc/kubernetes/pki/etcd
```

---

## Best Practices

### **Cost Optimization**

```yaml
# 1. Right-size resources
# Use VPA for recommendations
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: api-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  updatePolicy:
    updateMode: "Recommend"  # or "Auto"

# 2. Cluster Autoscaler
# Automatically scale node pools

# 3. Spot/Preemptible instances
nodeSelector:
  kubernetes.io/lifecycle: spot

tolerations:
- key: node.kubernetes.io/spot
  operator: Exists

# 4. Pod Disruption Budgets
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: api-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: api
```

### **Security Hardening**

```yaml
# Comprehensive security policy
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  automountServiceAccountToken: false
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: app:v1
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop: [ALL]
      readOnlyRootFilesystem: true
    resources:
      limits:
        cpu: 500m
        memory: 512Mi
      requests:
        cpu: 100m
        memory: 128Mi
```

### **Troubleshooting Checklist**

```bash
# Pod issues
kubectl describe pod <pod-name>
kubectl logs <pod-name> --previous
kubectl get events --sort-by='.lastTimestamp'

# Network issues
kubectl run netshoot --rm -it --image=nicolaka/netshoot -- /bin/bash
curl -v http://service-name
nslookup service-name

# Resource issues
kubectl top nodes
kubectl top pods
kubectl describe nodes | grep
