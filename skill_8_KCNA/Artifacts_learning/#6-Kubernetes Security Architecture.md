# 🔐 Kubernetes Security Architecture - Complete Guide

> **Expert-Level Security Reference**  
> Comprehensive security architecture covering authentication, authorization, pod security, admission control, secrets management, and runtime security for production Kubernetes environments.

---

## 📋 Table of Contents

1. [Security Philosophy & Defense-in-Depth](#security-philosophy)
2. [Authentication & Authorization](#authentication-authorization)
3. [Pod Security Standards](#pod-security-standards)
4. [Admission Control & Policy Enforcement](#admission-control)
5. [Secrets Management](#secrets-management)
6. [Runtime Security](#runtime-security)
7. [Network Security](#network-security)
8. [Production Security Checklist](#production-checklist)

---

## Security Philosophy

### **Defense-in-Depth Model**

Kubernetes security requires multiple layers of protection. If one layer is compromised, others provide defense:

```
Layer 1: Cluster Perimeter
├─ Network isolation (VPC, security groups)
├─ API Server authentication
└─ TLS everywhere

Layer 2: API Authorization
├─ RBAC (role-based access control)
├─ Admission controllers
└─ Audit logging

Layer 3: Pod Security
├─ Pod Security Standards
├─ Security contexts
└─ Linux security modules (Seccomp, AppArmor)

Layer 4: Network Security
├─ NetworkPolicy (micro-segmentation)
├─ Service mesh mTLS
└─ Egress filtering

Layer 5: Runtime Security
├─ Image scanning
├─ Runtime threat detection (Falco)
└─ Behavioral monitoring

Layer 6: Data Security
├─ Secrets encryption at rest
├─ etcd encryption
└─ Volume encryption
```

**Zero Trust Principles:**
- Never trust, always verify
- Least privilege access
- Assume breach mindset
- Verify explicitly at every layer

---

## Authentication & Authorization

### **ServiceAccount Authentication**

Every pod receives a ServiceAccount token by default - this is often unnecessary and creates attack surface.

**Default Token Mounting:**

```yaml
# Default behavior (INSECURE for most pods)
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  # serviceAccountName: default (implicit)
  # automountServiceAccountToken: true (implicit)
  containers:
  - name: app
    image: nginx

# Token automatically mounted at:
# /var/run/secrets/kubernetes.io/serviceaccount/token
```

**Inside the pod:**

```bash
# Token is a JWT that allows API access
cat /var/run/secrets/kubernetes.io/serviceaccount/token

# Decode the JWT payload
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
echo $TOKEN | cut -d '.' -f 2 | base64 -d | jq
{
  "iss": "kubernetes/serviceaccount",
  "sub": "system:serviceaccount:default:default",
  "aud": ["https://kubernetes.default.svc"],
  "exp": 1735948800,
  "kubernetes.io": {
    "namespace": "default",
    "serviceaccount": {"name": "default"}
  }
}

# Without RBAC, this fails:
curl -k -H "Authorization: Bearer $TOKEN" \
  https://kubernetes.default.svc/api/v1/pods
# Result: 403 Forbidden
```

**Best Practice: Disable Token Mounting**

```yaml
# Recommended for 80% of pods
apiVersion: v1
kind: ServiceAccount
metadata:
  name: webapp-sa
automountServiceAccountToken: false

---
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  serviceAccountName: webapp-sa
  automountServiceAccountToken: false  # Explicit disable
  containers:
  - name: app
    image: nginx
```

**When to Enable Token Mounting:**
- Operators and controllers (need K8s API access)
- CI/CD pods (deploy workloads)
- Monitoring agents (read metrics)
- Service mesh control planes (manage sidecars)

**When to Disable:**
- Web applications
- Databases
- Batch jobs
- Cache servers
- Any pod not using kubectl or K8s client libraries

**Bound Service Account Tokens (K8s 1.20+):**

```yaml
# Modern approach: Short-lived, audience-bound tokens
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  serviceAccountName: secure-sa
  containers:
  - name: app
    image: myapp
    volumeMounts:
    - name: token
      mountPath: /var/run/secrets/tokens
  volumes:
  - name: token
    projected:
      sources:
      - serviceAccountToken:
          path: token
          expirationSeconds: 3600  # 1 hour expiry
          audience: api  # Specific audience
```

Benefits:
- Time-bound (auto-expires)
- Audience-scoped (can't use for other services)
- Automatically rotated
- Reduces blast radius if compromised

### **RBAC: Role-Based Access Control**

RBAC is Kubernetes' authorization mechanism. Understanding RBAC is critical for production security.

**Core Components:**

```
Role/ClusterRole: Defines permissions (what can be done)
RoleBinding/ClusterRoleBinding: Grants permissions (who can do it)

Scope:
- Role + RoleBinding: Namespace-scoped
- ClusterRole + ClusterRoleBinding: Cluster-wide
- ClusterRole + RoleBinding: Namespace-scoped use of cluster role
```

**Role Definition:**

```yaml
# Namespace-scoped role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: production
rules:
- apiGroups: [""]  # "" = core API group
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get"]

---
# Cluster-wide role
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]
```

**RBAC Verbs:**

| Verb | HTTP Method | Meaning |
|------|-------------|---------|
| **get** | GET | Read single resource |
| **list** | GET | Read collection |
| **watch** | GET (stream) | Watch for changes |
| **create** | POST | Create new resource |
| **update** | PUT | Replace resource |
| **patch** | PATCH | Partial update |
| **delete** | DELETE | Delete resource |
| **deletecollection** | DELETE | Delete multiple |

**RoleBinding Example:**

```yaml
# Bind role to ServiceAccount
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: production
subjects:
- kind: ServiceAccount
  name: myapp-sa
  namespace: production
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**Common RBAC Patterns:**

**1. Read-Only Developer Access:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: read-only
rules:
- apiGroups: ["", "apps", "batch"]
  resources:
  - pods
  - services
  - deployments
  - jobs
  - configmaps
  verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developers-readonly
  namespace: staging
subjects:
- kind: Group
  name: developers  # From OIDC
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: read-only
  apiGroup: rbac.authorization.k8s.io
```

**2. CI/CD Deployer:**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ci-deployer
  namespace: production
automountServiceAccountToken: true  # Needs API access

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: ci-deployer
  namespace: production
rules:
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets"]
  verbs: ["get", "list", "create", "update", "patch"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "delete"]  # For rolling restarts
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get"]  # For deployment verification
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list", "create", "update"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]  # Read-only secrets!

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ci-deployer
  namespace: production
subjects:
- kind: ServiceAccount
  name: ci-deployer
  namespace: production
roleRef:
  kind: Role
  name: ci-deployer
  apiGroup: rbac.authorization.k8s.io
```

**3. Namespace Admin (No Cluster Resources):**

```yaml
# Use built-in 'admin' ClusterRole with RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: namespace-admin
  namespace: team-alpha
subjects:
- kind: User
  name: alice@company.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: admin  # Built-in role
  apiGroup: rbac.authorization.k8s.io

# 'admin' ClusterRole provides:
# - Full access to namespace resources
# - Cannot modify ResourceQuotas
# - Cannot modify Namespace itself
# - No cluster-level access
```

**RBAC Anti-Patterns (Avoid These):**

```yaml
# ❌ BAD: Wildcard permissions
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]

# ❌ BAD: cluster-admin for application
subjects:
- kind: ServiceAccount
  name: webapp-sa
roleRef:
  kind: ClusterRole
  name: cluster-admin  # Way too much power!

# ❌ BAD: Overly broad resources
rules:
- apiGroups: [""]
  resources: ["secrets"]  # All secrets!
  verbs: ["get", "list"]

# ✅ GOOD: Specific resources only
rules:
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["app-db-password"]  # Only this secret
  verbs: ["get"]
```

**RBAC Debugging:**

```bash
# Check if user/SA can perform action
kubectl auth can-i list pods --as=system:serviceaccount:production:myapp-sa
# Output: yes or no

# List all permissions for SA
kubectl auth can-i --list --as=system:serviceaccount:production:myapp-sa -n production

# Find who can perform action (requires kubectl-who-can plugin)
kubectl who-can delete deployments -n production

# Audit current bindings
kubectl get rolebindings,clusterrolebindings --all-namespaces -o wide

# Find bindings for specific ServiceAccount
kubectl get rolebindings,clusterrolebindings --all-namespaces -o json | \
  jq '.items[] | select(.subjects[]?.name=="myapp-sa")'
```

**RBAC Best Practices Summary:**

1. **Start with zero permissions, add only required**
2. **Never use cluster-admin except break-glass scenarios**
3. **Bind to groups, not individuals** (easier management)
4. **Use RoleBindings over ClusterRoleBindings** (namespace isolation)
5. **Specify resourceNames when possible** (finest granularity)
6. **Regular RBAC audits** (quarterly reviews)
7. **Document role purposes** (why these permissions?)
8. **Test with `auth can-i` before deployment**

---

## Pod Security Standards

Pod Security Standards (PSS) replaced PodSecurityPolicy in Kubernetes 1.25. PSS defines three profiles for pod security.

### **Three Security Profiles**

**1. Privileged (No Restrictions)**

Use for: System components (CNI, CSI drivers, node monitoring)

Allows:
- Privileged containers
- Host namespaces (network, PID, IPC)
- Host path volumes
- All capabilities

**2. Baseline (Minimal Restrictions)**

Use for: Most applications

Prevents:
- Privileged containers
- Host namespaces
- Host path volumes (except specific safe paths)
- Adding dangerous capabilities
- Running as UID 0 (but doesn't enforce non-root)

Allows:
- Some capabilities (AUDIT_WRITE, CHOWN, DAC_OVERRIDE, etc.)
- Writable root filesystem
- Running as any non-zero UID

**3. Restricted (Hardened)**

Use for: Security-sensitive workloads, production apps

Enforces:
- `runAsNonRoot: true`
- Drop ALL capabilities
- `allowPrivilegeEscalation: false`
- Read-only root filesystem (with exceptions)
- Seccomp profile (RuntimeDefault or Localhost)
- Volume types limited (no hostPath)

This should be the default for production applications.

### **Enabling Pod Security Admission**

```yaml
# Configure via namespace labels
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    # Enforce restricted profile (blocks non-compliant pods)
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: v1.28
    
    # Audit restricted (logs violations to audit log)
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/audit-version: v1.28
    
    # Warn users about violations
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/warn-version: v1.28
```

**Enforcement Modes:**

| Mode | Behavior | Use Case |
|------|----------|----------|
| **enforce** | Rejects non-compliant pods | Production enforcement |
| **audit** | Logs to audit log, allows pod | Monitoring without blocking |
| **warn** | Returns warning to user | Migration/testing phase |

**Recommended Configuration by Environment:**

```yaml
# Production namespaces
enforce: restricted
audit: restricted
warn: restricted

# Staging namespaces (testing migration)
enforce: baseline
audit: restricted
warn: restricted

# Development namespaces
enforce: baseline
audit: baseline
warn: baseline

# System namespaces (kube-system)
enforce: privileged
audit: restricted  # Still log violations
warn: baseline
```

### **Security Context: Making Pods Compliant**

To pass the restricted profile, pods must have specific security settings:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  # Pod-level security context
  securityContext:
    runAsNonRoot: true  # Required for restricted
    runAsUser: 1000     # Non-root UID
    fsGroup: 2000       # Volume ownership
    seccompProfile:
      type: RuntimeDefault  # Required for restricted
  
  # Disable SA token mounting if not needed
  automountServiceAccountToken: false
  
  containers:
  - name: app
    image: myapp:v1
    
    # Container-level security context
    securityContext:
      allowPrivilegeEscalation: false  # Required for restricted
      capabilities:
        drop:
        - ALL  # Required for restricted
      readOnlyRootFilesystem: true  # Best practice
      runAsNonRoot: true
      runAsUser: 1000
    
    # Resource limits (prevents DoS)
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 500m
        memory: 512Mi
    
    # Writable volumes (root FS is read-only)
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: cache
      mountPath: /app/cache
  
  volumes:
  - name: tmp
    emptyDir: {}
  - name: cache
    emptyDir: {}
```

**Key Security Context Fields:**

| Field | Level | Purpose | Restricted Requirement |
|-------|-------|---------|----------------------|
| `runAsNonRoot` | Pod/Container | Enforce non-root | **Required: true** |
| `runAsUser` | Pod/Container | Specify UID | **Required: non-zero** |
| `allowPrivilegeEscalation` | Container | Prevent gaining privileges | **Required: false** |
| `capabilities.drop` | Container | Drop Linux capabilities | **Required: ALL** |
| `readOnlyRootFilesystem` | Container | Immutable container FS | Recommended: true |
| `seccompProfile` | Pod | Syscall filtering | **Required: RuntimeDefault** |
| `fsGroup` | Pod | Volume ownership | Optional |

**Linux Capabilities:**

Capabilities divide root privileges into distinct units. The restricted profile requires dropping ALL.

**Dangerous capabilities (NEVER add in production):**

- `SYS_ADMIN`: Full system administration
- `NET_ADMIN`: Network configuration (change routes, firewall)
- `SYS_PTRACE`: Trace any process (debugging backdoor)
- `DAC_OVERRIDE`: Bypass file read/write/execute permissions
- `SYS_MODULE`: Load/unload kernel modules
- `SYS_RAWIO`: Raw I/O port access
- `SYS_CHROOT`: Change root directory (container escape)

**Safe capabilities (add only if required):**

```yaml
securityContext:
  capabilities:
    drop:
    - ALL
    add:
    - NET_BIND_SERVICE  # Bind to ports < 1024 (e.g., port 80)
```

**Common use cases:**
- `NET_BIND_SERVICE`: Web servers binding to port 80/443
- `CHOWN`: Change file ownership (some legacy apps)
- `SETUID/SETGID`: Change user/group ID (su/sudo functionality)

**Testing Pod Security Compliance:**

```bash
# Try to create non-compliant pod
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: bad-pod
  namespace: production
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      privileged: true  # Violates restricted
EOF

# Error: pods "bad-pod" is forbidden: 
# violates PodSecurity "restricted:v1.28": 
# privileged (container "app" must not set securityContext.privileged=true)

# Check what's wrong
kubectl --dry-run=server create -f bad-pod.yaml

# Tool: kubectl-score for best practices
kubectl score deployment.yaml
```

---

## Admission Control

Admission controllers intercept API requests after authentication/authorization but before persistence to etcd. They can mutate or validate requests.

### **Built-in Admission Controllers**

**Always Enable (Critical):**

- `PodSecurityAdmission`: Enforce Pod Security Standards
- `ResourceQuota`: Enforce namespace quotas
- `LimitRanger`: Set default resource limits
- `ServiceAccount`: Auto-inject default ServiceAccount
- `DefaultStorageClass`: Set default StorageClass
- `MutatingAdmissionWebhook`: Enable custom mutations
- `ValidatingAdmissionWebhook`: Enable custom validation
- `NamespaceLifecycle`: Prevent operations in terminating namespaces

**View enabled controllers:**

```bash
kubectl exec -n kube-system kube-apiserver-master-1 -- \
  kube-apiserver -h | grep enable-admission-plugins
```

### **Dynamic Admission Control**

**Admission Webhook Flow:**

```
kubectl create pod → API Server
    ↓
Authentication (who?)
    ↓
Authorization (allowed?)
    ↓
Mutating Admission Webhooks (modify)
    ↓
Object Schema Validation
    ↓
Validating Admission Webhooks (accept/reject)
    ↓
Persistence to etcd
```

**Mutating Webhook Example (Inject Sidecar):**

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: sidecar-injector
webhooks:
- name: sidecar.example.com
  clientConfig:
    service:
      name: sidecar-injector
      namespace: webhook-system
      path: /mutate
    caBundle: <base64-ca-cert>
  rules:
  - operations: ["CREATE"]
    apiGroups: [""]
    apiVersions: ["v1"]
    resources: ["pods"]
  namespaceSelector:
    matchLabels:
      sidecar-injection: enabled
  admissionReviewVersions: ["v1"]
  sideEffects: None
  failurePolicy: Fail  # Block if webhook unavailable
  timeoutSeconds: 10
```

**Webhook Response (JSON Patch):**

```json
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "request-uid",
    "allowed": true,
    "patchType": "JSONPatch",
    "patch": "<base64-encoded-json-patch>"
  }
}

// Decoded patch (add sidecar container):
[
  {
    "op": "add",
    "path": "/spec/containers/-",
    "value": {
      "name": "sidecar",
      "image": "sidecar:v1",
      "ports": [{"containerPort": 15001}]
    }
  }
]
```

**Validating Webhook Example (Require Labels):**

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: label-validator
webhooks:
- name: validate.labels.example.com
  clientConfig:
    service:
      name: label-validator
      namespace: webhook-system
      path: /validate
  rules:
  - operations: ["CREATE", "UPDATE"]
    apiGroups: ["apps"]
    apiVersions: ["v1"]
    resources: ["deployments"]
  admissionReviewVersions: ["v1"]
  sideEffects: None
  failurePolicy: Fail

// Webhook logic (pseudo-code):
if deployment.metadata.labels["team"] == "" {
  return {
    "allowed": false,
    "status": {
      "message": "Deployment must have 'team' label"
    }
  }
}
```

**Webhook Best Practices:**

1. **Keep webhooks fast** (< 100ms response time)
2. **Implement timeouts** (default: 10s, tune based on load)
3. **Use `failurePolicy: Fail` for security webhooks** (Ignore = security bypass risk)
4. **Monitor webhook latency** (impacts pod creation time)
5. **Implement retry logic** (network failures happen)
6. **Use namespace selectors** (avoid processing kube-system)
7. **Version webhook APIs** (rolling updates)
8. **Test thoroughly** (webhook bugs block cluster operations)

### **Policy Engines**

**OPA Gatekeeper (Open Policy Agent):**

```yaml
# Constraint Template (reusable policy)
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        openAPIV3Schema:
          properties:
            labels:
              type: array
              items: {type: string}
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels
        
        violation[{"msg": msg}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("Missing required labels: %v", [missing])
        }

---
# Constraint (policy instance)
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: require-team-label
spec:
  match:
    kinds:
    - apiGroups: ["apps"]
      kinds: ["Deployment"]
  parameters:
    labels: ["team", "cost-center"]
```

**Kyverno (Kubernetes-native, YAML-based):**

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-resource-limits
spec:
  validationFailureAction: enforce  # or audit
  rules:
  - name: check-cpu-memory-limits
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "All containers must have CPU and memory limits"
      pattern:
        spec:
          containers:
          - resources:
              limits:
                memory: "?*"
                cpu: "?*"

---
# Kyverno mutation policy (add labels)
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-default-labels
spec:
  rules:
  - name: add-team-label
    match:
      any:
      - resources:
          kinds:
          - Deployment
    mutate:
      patchStrategicMerge:
        metadata:
          labels:
            managed-by: kyverno
```

**OPA vs Kyverno:**

| Aspect | OPA Gatekeeper | Kyverno |
|--------|----------------|---------|
| **Language** | Rego (learning curve) | YAML (familiar) |
| **Flexibility** | Very high (code logic) | Moderate (declarative) |
| **Complexity** | Higher (requires Rego) | Lower (YAML only) |
| **Validation** | ✅ Yes | ✅ Yes |
| **Mutation** | ✅ Yes | ✅ Yes |
| **Generation** | ❌ No | ✅ Yes (auto-create resources) |
| **Best For** | Complex logic, reusable | Simple policies, K8s-native |

**Policy Implementation Strategy:**

1. **Start with audit mode** (`validationFailureAction: audit`)
2. **Monitor violations** (metrics, logs)
3. **Fix existing violations**
4. **Graduate to enforce mode**
5. **Iterate on policies** (refine based on feedback)

---

## Secrets Management

### **Kubernetes Secrets (Native)**

**Critical Understanding:** Kubernetes Secrets are base64-encoded, NOT encrypted by default.

```yaml
# Creating a secret
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: cG9zdGdyZXM=  # base64: postgres
  password: c2VjcmV0MTIz  # base64: secret123

# Anyone with 'get secrets' permission can decode:
kubectl get secret db-credentials -o jsonpath='{.data.password}' | base64 -d
# Output: secret123
```

**Secret Types:**

- `Opaque`: Generic key-value (default)
- `kubernetes.io/tls`: TLS certificates
- `kubernetes.io/dockerconfigjson`: Registry credentials
- `kubernetes.io/service-account-token`: SA tokens
- `kubernetes.io/basic-auth`: Basic auth

### **Encryption at Rest**

Without encryption, secrets in etcd are base64-encoded only:

```bash
# On etcd node, read secret directly:
ETCDCTL_API=3 etcdctl get /registry/secrets/default/db-credentials

# Output shows plaintext values!
```

**Enable Encryption:**

```yaml
# /etc/kubernetes/encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources:
  - secrets
  providers:
  - aescbc:  # AES-CBC encryption
      keys:
      - name: key1
        secret: <32-byte-base64-key>  # Generate: head -c 32 /dev/urandom | base64
  - identity: {}  # Fallback (no encryption)

# API Server configuration:
--encryption-provider-config=/etc/kubernetes/encryption-config.yaml

# Verify encryption (secrets should be unreadable):
ETCDCTL_API=3 etcdctl get /registry/secrets/default/db-credentials
# Output: Binary encrypted data
```

**Encryption Provider Options:**

| Provider | Encryption | Performance | Key Management |
|----------|------------|-------------|----------------|
| **identity** | None (plaintext) | Fastest | N/A |
| **aescbc** | AES-CBC | Fast | Manual key in file |
| **aesgcm** | AES-GCM | Fast | Manual key in file |
| **secretbox** | XSalsa20/Poly1305 | Fast | Manual key in file |
| **kms** | Cloud KMS | Slower | AWS KMS, GCP KMS, Azure Key Vault |

**Recommendation:** Use KMS provider for production (automatic key rotation, audit logs, HSM-backed).

**KMS Provider Example (AWS):**

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources:
  - secrets
  providers:
  - kms:
      name: aws-kms
      endpoint: unix:///var/run/kmsplugin/socket.sock
      cachesize: 1000
      timeout: 3s
  - identity: {}
```

### **External Secrets Operator**

Best practice: Don't store secrets in Kubernetes at all. Use external secret stores.

**External Secrets Operator (ESO):**

```yaml
# 1. Define secret store connection
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets-manager
  namespace: production
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets-sa

---
# 2. Sync external secret to K8s Secret
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
  namespace: production
spec:
  refreshInterval: 1h  # Check for updates hourly
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: db-credentials  # K8s Secret name
    creationPolicy: Owner
  data:
  - secretKey: username
    remoteRef:
      key: production/database/username
  - secretKey: password
    remoteRef:
      key: production/database/password

# Result: K8s Secret created and kept in sync
# Source of truth: AWS Secrets Manager (not Git, not K8s)
```

**Benefits:**

- Secrets never in Git or K8s manifests
- Centralized secret management
- Automatic rotation support
- Audit logs in cloud provider
- Access control via cloud IAM

**HashiCorp Vault Integration:**

```yaml
# Vault SecretStore
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-backend
  namespace: production
spec:
  provider:
    vault:
      server: "https://vault.company.com"
      path: "secret"
      version: "v2"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "production-role"
          serviceAccountRef:
            name: vault-auth-sa

---
# External Secret from Vault
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: api-keys
  namespace: production
spec:
  refreshInterval: 15m
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: api-keys
  data:
  - secretKey: stripe-key
    remoteRef:
      key: production/api/stripe
      property: api_key
```

**Secrets Management Best Practices:**

1. **Never commit secrets to Git** (use external secret stores)
2. **Enable etcd encryption at rest** (encryption-config)
3. **Use KMS provider** (AWS KMS, GCP KMS, Azure Key Vault)
4. **Implement secret rotation** (automated via ESO refresh)
5. **Limit secret access** (RBAC - only apps that need them)
6. **Audit secret access** (who accessed what, when)
7. **Use short-lived secrets** (dynamic secrets from Vault)
8. **Scan for secrets in code** (git-secrets, truffleHog)

---

## Runtime Security

### **Seccomp Profiles**

Seccomp (Secure Computing Mode) filters system calls that containers can make.

```yaml
# Use RuntimeDefault profile (recommended)
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault  # Uses container runtime's default profile
  containers:
  - name: app
    image: myapp:v1

# What RuntimeDefault blocks:
# - mount/umount (filesystem operations)
# - reboot/shutdown (system control)
# - load kernel modules
# - raw socket creation
# - ptrace (debugging other processes)
# - ~44 dangerous syscalls blocked
```

**Custom Seccomp Profile:**

```yaml
# Create custom profile (JSON file on node)
# /var/lib/kubelet/seccomp/profiles/custom.json
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "architectures": ["SCMP_ARCH_X86_64"],
  "syscalls": [
    {
      "names": ["accept", "bind", "connect", "socket"],
      "action": "SCMP_ACT_ALLOW"
    },
    {
      "names": ["read", "write", "open", "close"],
      "action": "SCMP_ACT_ALLOW"
    }
  ]
}

# Use custom profile
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/custom.json
```

**Seccomp Best Practices:**

- Always use `RuntimeDefault` minimum
- Test custom profiles thoroughly (apps may break)
- Generate profiles from runtime behavior (tools: oci-seccomp-bpf-hook)
- Monitor seccomp violations (auditd logs)

### **AppArmor**

AppArmor provides Mandatory Access Control (MAC) for file access, network operations, and capabilities.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: apparmor-pod
  annotations:
    # Apply AppArmor profile to container
    container.apparmor.security.beta.kubernetes.io/app: runtime/default
spec:
  containers:
  - name: app
    image: nginx
```

**AppArmor Profiles:**

- `runtime/default`: Default Docker/containerd profile
- `localhost/<profile>`: Custom profile on node
- `unconfined`: No restrictions (dangerous)

**Custom AppArmor Profile Example:**

```bash
# /etc/apparmor.d/nginx-custom
#include <tunables/global>

profile nginx-custom flags=(attach_disconnected) {
  #include <abstractions/base>
  
  # Allow network
  network inet tcp,
  network inet udp,
  
  # Allow read access to specific directories
  /usr/sbin/nginx r,
  /etc/nginx/** r,
  /var/log/nginx/* w,
  
  # Deny everything else
  deny /** w,
}

# Load profile
sudo apparmor_parser -r -W /etc/apparmor.d/nginx-custom
```

**Limitation:** AppArmor not available on all distros (use SELinux on RHEL/CentOS).

### **Image Security**

**Image Scanning in CI/CD:**

```yaml
# GitLab CI example
stages:
  - build
  - scan
  - deploy

build:
  stage: build
  script:
    - docker build -t myapp:$CI_COMMIT_SHA .
    - docker push myapp:$CI_COMMIT_SHA

scan:
  stage: scan
  image: aquasec/trivy:latest
  script:
    - trivy image --exit-code 1 --severity CRITICAL,HIGH myapp:$CI_COMMIT_SHA
  # Fails pipeline if CRITICAL or HIGH vulnerabilities found

deploy:
  stage: deploy
  script:
    - kubectl set image deployment/myapp app=myapp:$CI_COMMIT_SHA
  only:
    - main
```

**Runtime Image Scanning (Admission Controller):**

```yaml
# Trivy admission controller
apiVersion: apps/v1
kind: Deployment
metadata:
  name: trivy-admission
  namespace: trivy-system
spec:
  template:
    spec:
      containers:
      - name: trivy
        image: aquasec/trivy:latest
        args:
        - server
        - --listen=0.0.0.0:8080

---
# ValidatingWebhook
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: trivy-webhook
webhooks:
- name: trivy.webhook.com
  rules:
  - operations: ["CREATE", "UPDATE"]
    apiGroups: [""]
    apiVersions: ["v1"]
    resources: ["pods"]
  # Webhook scans image and rejects if vulnerabilities found
```

**Image Policy (Allow Only Signed Images):**

```yaml
# Connaisseur admission controller (Notary/cosign integration)
apiVersion: v1
kind: ConfigMap
metadata:
  name: connaisseur-config
data:
  policy.yaml: |
    policy:
    - pattern: "myregistry.com/*"
      verify: true  # Require signature
      trust_root: /keys/root.key
    - pattern: "*"
      verify: false
      reject: true  # Block unsigned images
```

**Image Security Best Practices:**

1. **Scan in CI/CD** (fail build on critical CVEs)
2. **Runtime scanning** (admission controller)
3. **Use minimal base images** (distroless, alpine)
4. **Sign images** (cosign, notary)
5. **Private registries** with access control
6. **Block 'latest' tag** in production
7. **Regular base image updates** (automated)
8. **Remove unnecessary tools** (shells, package managers)

### **Runtime Threat Detection (Falco)**

Falco detects anomalous behavior at runtime using eBPF:

```yaml
# Falco DaemonSet
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: falco
  namespace: falco
spec:
  template:
    spec:
      hostNetwork: true
      hostPID: true
      containers:
      - name: falco
        image: falcosecurity/falco:latest
        securityContext:
          privileged: true  # Needs kernel access
        volumeMounts:
        - name: dev
          mountPath: /host/dev
        - name: proc
          mountPath: /host/proc
        - name: boot
          mountPath: /host/boot
        - name: modules
          mountPath: /host/lib/modules
      volumes:
      - name: dev
        hostPath:
          path: /dev
      - name: proc
        hostPath:
          path: /proc
      - name: boot
        hostPath:
          path: /boot
      - name: modules
        hostPath:
          path: /lib/modules
```

**Falco Rules (Examples):**

```yaml
# Detect shell in container
- rule: Terminal shell in container
  desc: Shell spawned in a container
  condition: >
    spawned_process and
    container and
    proc.name in (bash, sh, zsh, csh, ksh, ash)
  output: >
    Shell spawned in container 
    (user=%user.name container=%container.name 
    image=%container.image.repository proc=%proc.name)
  priority: WARNING

# Detect write to /etc
- rule: Write below etc
  desc: Write to /etc directory
  condition: >
    open_write and
    container and
    fd.directory=/etc
  output: >
    Write to /etc directory
    (file=%fd.name container=%container.name)
  priority: WARNING

# Detect privileged container
- rule: Launch Privileged Container
  desc: Privileged container started
  condition: >
    container_started and
    container.privileged=true
  output: >
    Privileged container started
    (container=%container.name image=%container.image)
  priority: CRITICAL

# Detect crypto mining
- rule: Detect crypto miners
  desc: Crypto mining process detected
  condition: >
    spawned_process and
    proc.name in (xmrig, minergate, cryptonight)
  output: >
    Crypto miner detected (proc=%proc.name)
  priority: CRITICAL
```

**Falco Integration:**

```yaml
# Send alerts to Slack
apiVersion: v1
kind: ConfigMap
metadata:
  name: falco-config
data:
  falco.yaml: |
    json_output: true
    json_include_output_property: true
    http_output:
      enabled: true
      url: "https://hooks.slack.com/services/YOUR/WEBHOOK/URL"

# Or send to SIEM (Splunk, Elasticsearch)
```

**Falco Best Practices:**

- Deploy as DaemonSet (every node)
- Tune rules to reduce false positives
- Integrate with incident response (PagerDuty, Slack)
- Regular rule updates
- Monitor Falco resource usage (can be CPU-intensive)

---

## Network Security

### **NetworkPolicy** (Brief - See Networking Artifact)

NetworkPolicy provides Layer 3/4 network segmentation:

```yaml
# Default deny all ingress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress

---
# Allow specific traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

**Critical:** Always include DNS egress rules:

```yaml
egress:
- to:
  - namespaceSelector:
      matchLabels:
        name: kube-system
    podSelector:
      matchLabels:
        k8s-app: kube-dns
  ports:
  - protocol: UDP
    port: 53
```

### **Service Mesh (mTLS)**

Service mesh adds Layer 7 security:

```yaml
# Istio: Enable strict mTLS
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT

# All pod-to-pod traffic automatically encrypted
# Zero application code changes required
```

**Network Security Best Practices:**

1. **Default deny NetworkPolicy** (whitelist approach)
2. **Enable service mesh mTLS** (encryption + identity)
3. **Egress filtering** (control outbound traffic)
4. **Monitor network anomalies** (unexpected connections)

---

## Production Checklist

### **Infrastructure Security**

- ✅ Control plane HA with TLS encryption
- ✅ etcd encryption at rest enabled
- ✅ API Server behind firewall/VPN
- ✅ Node OS hardened (CIS benchmark)
- ✅ Regular security patches applied
- ✅ Separate control plane from worker nodes

### **Authentication & Authorization**

- ✅ OIDC for human users (no static passwords)
- ✅ No cluster-admin bindings (except break-glass)
- ✅ Dedicated ServiceAccount per application
- ✅ Disable default ServiceAccount token mounting
- ✅ Regular RBAC audit (quarterly)
- ✅ Groups-based RBAC (not individual users)

### **Pod Security**

- ✅ Pod Security Standards enforced (restricted profile)
- ✅ All pods run as non-root (runAsNonRoot: true)
- ✅ All capabilities dropped (drop: [ALL])
- ✅ Read-only root filesystem (where possible)
- ✅ Resource limits set (prevent DoS)
- ✅ No privileged containers

### **Admission Control**

- ✅ Policy engine deployed (OPA Gatekeeper or Kyverno)
- ✅ Image scanning webhook enabled
- ✅ Resource validation policies active
- ✅ Custom compliance policies enforced
- ✅ Webhook monitoring (latency, failures)

### **Secrets Management**

- ✅ External secrets operator deployed (ESO)
- ✅ No secrets in Git repositories
- ✅ etcd encryption at rest enabled
- ✅ Secret rotation automated
- ✅ Vault or cloud KMS integration
- ✅ RBAC limits secret access

### **Network Security**

- ✅ NetworkPolicy default deny in place
- ✅ Service mesh with mTLS enabled
- ✅ Ingress TLS termination configured
- ✅ Egress filtering implemented
- ✅ Network monitoring active

### **Runtime Security**

- ✅ Seccomp RuntimeDefault profile
- ✅ Image scanning in CI/CD pipeline
- ✅ Falco deployed for threat detection
- ✅ No privileged containers in production
- ✅ Regular vulnerability patching
- ✅ Incident response plan documented

### **Audit & Compliance**

- ✅ Audit logging enabled (all mutations)
- ✅ Logs sent to external SIEM
- ✅ Regular security audits (quarterly)
- ✅ CIS benchmark compliance verified
- ✅ Vulnerability scanning automated
- ✅ Compliance reports generated

### **Monitoring & Response**

- ✅ Security metrics in Prometheus
- ✅ Alerts for security violations
- ✅ Incident response runbooks
- ✅ Regular security drills
- ✅ Post-incident reviews

---

## Summary

**Defense-in-Depth Layers:**

1. **Authentication**: OIDC for humans, ServiceAccounts for pods, disable unnecessary tokens
2. **Authorization**: RBAC with least privilege, no cluster-admin, regular audits
3. **Admission Control**: Policy enforcement (OPA/Kyverno), image scanning, validation webhooks
4. **Pod Security**: Pod Security Standards (restricted), security contexts, capabilities management
5. **Secrets**: External secret stores (Vault, cloud KMS), encryption at rest, rotation
6. **Runtime**: Seccomp, AppArmor, image scanning, Falco threat detection
7. **Network**: NetworkPolicy micro-segmentation, service mesh mTLS
8. **Audit**: Comprehensive logging, SIEM integration, compliance monitoring

**Critical Security Principles:**

- **Least Privilege**: Start with zero, add only required permissions
- **Zero Trust**: Never trust, always verify at every layer
- **Defense in Depth**: Multiple security layers (one breach doesn't compromise all)
- **Immutable Infrastructure**: Containers are ephemeral, treat as cattle not pets
- **Audit Everything**: Comprehensive logging for forensics and compliance
- **Automate Security**: Manual processes fail, automation scales
- **Regular Testing**: Security drills, penetration testing, vulnerability scanning

**Common Security Mistakes:**

- ❌ Running as root (use runAsNonRoot: true)
- ❌ Mounting ServiceAccount tokens unnecessarily (80% of pods don't need)
- ❌ Using cluster-admin role (overly permissive)
- ❌ Secrets in Git (use external secret stores)
- ❌ No NetworkPolicy (flat network = lateral movement risk)
- ❌ No admission control (no policy enforcement)
- ❌ Privileged containers (container escape risk)
- ❌ No image scanning (vulnerable dependencies)
- ❌ No audit logging (no forensics capability)
- ❌ No security monitoring (blind to attacks)

**Recommended Security Stack:**

- **Authentication**: OIDC (Google, Azure AD, Okta)
- **Authorization**: RBAC with rbac-lookup auditing
- **Policy Engine**: Kyverno (Kubernetes-native) or OPA Gatekeeper (complex logic)
- **Secrets**: External Secrets Operator + Vault/AWS Secrets Manager
- **Admission Control**: Image scanning (Trivy), policy validation (Kyverno/OPA)
- **Runtime Detection**: Falco for anomaly detection
- **Network Security**: Calico NetworkPolicy + Istio/Cilium for mTLS
- **Audit**: API Server audit logs + SIEM integration
- **Compliance**: kube-bench (CIS), kubeaudit, Starboard

This completes the comprehensive Kubernetes security architecture guide.
