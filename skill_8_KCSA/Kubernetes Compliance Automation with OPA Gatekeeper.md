# 🔒 Lab: Kubernetes Compliance Automation with OPA Gatekeeper

> 💡 **What You'll Master:** Deploy policy-as-code in Kubernetes to automatically enforce security standards and prevent non-compliant workloads
>
> ⏱️ **Total Time:** 2-3 hours  
> 🎯 **Difficulty:** Intermediate  
> 🔧 **Prerequisites:** Fresh Ubuntu on AWS, basic Linux commands  
> 💻 **Environment:** Ubuntu on AWS (t2.medium or larger)

---

## 🎯 Learning Objectives

**By the end of this lab, you'll be able to:**

1. **Deploy Kubernetes** and OPA Gatekeeper on a single node
2. **Create security policies** using Rego language to restrict container privileges
3. **Automate compliance checks** that prevent policy violations at deployment time
4. **Troubleshoot** policy enforcement and understand admission webhooks

**Real-World Value:** These skills directly apply to enterprise environments requiring SOC 2, PCI DSS, or HIPAA compliance.

---

## 📊 Skills Progress Tracker

| Skill Area | Progress | Confidence |
|------------|----------|------------|
| **Kubernetes Setup** | ⚪⚪⚪⚪⚪ | Low → High |
| **OPA Gatekeeper** | ⚪⚪⚪⚪⚪ | Low → High |
| **Policy Creation** | ⚪⚪⚪⚪⚪ | Low → High |
| **Compliance Testing** | ⚪⚪⚪⚪⚪ | Low → High |

---

# 📦 Phase 0: Kubernetes Cluster Setup (20 min)

> ⚠️ **NOTE:** Your school assignment assumes you already have a Kubernetes cluster. This phase sets that up on your fresh Ubuntu instance.

## 🤔 Prediction Challenge (60 seconds)

Before proceeding, think about:
1. What components does Kubernetes need to run?
2. Why use KIND (Kubernetes in Docker) vs full installation?
3. How will you verify the cluster is working?

<details>
<summary>💡 <strong>Answers</strong></summary>

1. **Components**: Container runtime (Docker), kubectl CLI, control plane (API server, scheduler, controller manager), etcd database
2. **Why KIND**: Fast setup (2 min vs 30+ min), runs in containers (no VMs), perfect for learning, easy cleanup
3. **Verification**: `kubectl cluster-info`, check node status, verify system pods running
</details>

---

## 💻 Complete Setup Script

**This installs everything needed for your school assignment:**

```bash
#!/bin/bash
# Save as: 00-complete-setup.sh

set -e  # Exit on any error

echo "============================================"
echo "  KUBERNETES CLUSTER SETUP FOR LAB"
echo "============================================"
echo ""

# ============================================
echo "▶ STEP 1: System Update"
# ============================================
echo "Updating package lists..."
sudo apt-get update -qq
sudo apt-get upgrade -y -qq
echo "✅ System updated"
echo ""

# ============================================
echo "▶ STEP 2: Install Docker"
# ============================================
if command -v docker &> /dev/null; then
    echo "Docker already installed: $(docker --version)"
else
    echo "Installing Docker..."
    
    # Install prerequisites
    sudo apt-get install -y -qq \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
    
    # Add Docker GPG key
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    
    # Add Docker repository
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    
    # Install Docker
    sudo apt-get update -qq
    sudo apt-get install -y -qq docker-ce docker-ce-cli containerd.io docker-compose-plugin
    
    # Add user to docker group
    sudo usermod -aG docker $USER
    
    # Start Docker
    sudo systemctl start docker
    sudo systemctl enable docker
    
    echo "✅ Docker installed: $(docker --version)"
fi
echo ""

# ============================================
echo "▶ STEP 3: Install kubectl"
# ============================================
if command -v kubectl &> /dev/null; then
    echo "kubectl already installed: $(kubectl version --client --short 2>/dev/null || kubectl version --client)"
else
    echo "Installing kubectl..."
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    rm kubectl
    echo "✅ kubectl installed: $(kubectl version --client --short 2>/dev/null || kubectl version --client)"
fi
echo ""

# ============================================
echo "▶ STEP 4: Install KIND (Kubernetes in Docker)"
# ============================================
if command -v kind &> /dev/null; then
    echo "KIND already installed: $(kind version)"
else
    echo "Installing KIND v0.20.0..."
    [ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
    chmod +x ./kind
    sudo mv ./kind /usr/local/bin/kind
    echo "✅ KIND installed: $(kind version)"
fi
echo ""

# ============================================
echo "▶ STEP 5: Install jq (JSON processor)"
# ============================================
if command -v jq &> /dev/null; then
    echo "jq already installed: $(jq --version)"
else
    sudo apt-get install -y -qq jq
    echo "✅ jq installed: $(jq --version)"
fi
echo ""

# ============================================
echo "▶ STEP 6: Apply Docker Group Permissions"
# ============================================
echo "Applying docker group permissions..."
newgrp docker << EOFGROUP
echo "✅ Docker permissions applied"
EOFGROUP
echo ""

# ============================================
echo "▶ STEP 7: Create Kubernetes Cluster"
# ============================================
if kind get clusters 2>/dev/null | grep -q "gatekeeper-lab"; then
    echo "Cluster 'gatekeeper-lab' already exists"
else
    echo "Creating Kubernetes cluster (this takes ~2 minutes)..."
    kind create cluster --name gatekeeper-lab --wait 2m
    echo "✅ Cluster created"
fi
echo ""

# ============================================
echo "▶ STEP 8: Verify Setup"
# ============================================
echo "Cluster Info:"
kubectl cluster-info --context kind-gatekeeper-lab

echo ""
echo "Node Status:"
kubectl get nodes

echo ""
echo "System Pods:"
kubectl get pods -n kube-system

echo ""
echo "============================================"
echo "  ✅ SETUP COMPLETE - Ready for Lab Tasks"
echo "============================================"
echo ""
echo "Your Kubernetes cluster is ready!"
echo "Proceed to Task 1 of your school assignment."
```

**Run the setup:**

```bash
chmod +x 00-complete-setup.sh
./00-complete-setup.sh
```

**Expected Output:** One node in "Ready" status, all kube-system pods "Running"

---

## 🔧 Quick Troubleshooting

<details>
<summary><strong>If "permission denied" on docker commands</strong></summary>

```bash
sudo usermod -aG docker $USER
newgrp docker
docker ps  # Should work now
```
</details>

<details>
<summary><strong>If cluster creation fails</strong></summary>

```bash
kind delete cluster --name gatekeeper-lab
kind create cluster --name gatekeeper-lab
```
</details>

---

# 📦 Task 1: Install Open Policy Agent (OPA) Gatekeeper

## 🤔 Prediction Challenge (90 seconds)

Before installing Gatekeeper, consider:
1. What Kubernetes resources will Gatekeeper create?
2. How does Gatekeeper intercept pod creation requests?
3. What happens if Gatekeeper pods are down?

<details>
<summary>💡 <strong>Answers</strong></summary>

1. **Resources**: Namespace (gatekeeper-system), Pods (audit + controllers), CRDs (Custom Resource Definitions), ValidatingWebhookConfiguration
2. **Interception**: Uses admission webhooks - K8s API server forwards all create/update requests to Gatekeeper BEFORE persisting to etcd
3. **Impact**: Depends on failurePolicy setting. Gatekeeper uses "Ignore" to prevent cluster lockout during downtime
</details>

---

## 🔐 Core Concept: OPA Gatekeeper Architecture

**What is it?**
- **Policy Engine**: Validates K8s resources against defined rules
- **Admission Controller**: Intercepts API requests before persistence
- **Audit System**: Continuously scans existing resources for violations

**Mental Model:** Think of airport security - Gatekeeper checks every passenger (resource) before boarding (deployment), and also patrols the airport (cluster) for security issues.

**How it works:**
```
Developer → kubectl apply pod.yaml
              ↓
         K8s API Server
              ↓
     Admission Webhook → Gatekeeper Controller
              ↓
         OPA Engine (Rego evaluation)
              ↓
         Allow ✅ or Deny ❌
```

---

## Subtask 1.1: Verify Kubernetes Cluster Status

**From your school assignment - First, let's ensure your Kubernetes cluster is running properly:**

```bash
#!/bin/bash
# Save as: task1-subtask1-verify-cluster.sh

echo "============================================"
echo "  Task 1 - Subtask 1.1: Verify Cluster"
echo "============================================"
echo ""

echo "▶ Check cluster status"
kubectl cluster-info

echo ""
echo "▶ Verify nodes are ready"
kubectl get nodes

echo ""
echo "▶ Check system pods"
kubectl get pods -n kube-system
```

**Run it:**
```bash
chmod +x task1-subtask1-verify-cluster.sh
./task1-subtask1-verify-cluster.sh
```

**What to look for:**
- Cluster info shows control plane URL
- Node status: "Ready"
- System pods status: All "Running"

---

## Subtask 1.2: Install OPA Gatekeeper

**From your school assignment - OPA Gatekeeper is the policy engine that enforces Open Policy Agent policies in Kubernetes:**

```bash
#!/bin/bash
# Save as: task1-subtask2-install-gatekeeper.sh

echo "============================================"
echo "  Task 1 - Subtask 1.2: Install Gatekeeper"
echo "============================================"
echo ""

echo "▶ Apply the Gatekeeper manifest"
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/release-3.14/deploy/gatekeeper.yaml

echo ""
echo "⏳ Wait for Gatekeeper pods to be ready (this may take 2-3 minutes)"
kubectl wait --for=condition=ready pod -l control-plane=controller-manager -n gatekeeper-system --timeout=300s

echo ""
echo "▶ Verify Gatekeeper installation"
kubectl get pods -n gatekeeper-system

echo ""
echo "Expected output should show three pods running:"
echo "  - gatekeeper-audit (1 pod)"
echo "  - gatekeeper-controller-manager (3 pods)"
```

**Run it:**
```bash
chmod +x task1-subtask2-install-gatekeeper.sh
./task1-subtask2-install-gatekeeper.sh
```

**Expected Output:**
```
NAME                                             READY   STATUS    RESTARTS
gatekeeper-audit-xxx                            1/1     Running   0
gatekeeper-controller-manager-xxx               1/1     Running   0
gatekeeper-controller-manager-xxx               1/1     Running   0
gatekeeper-controller-manager-xxx               1/1     Running   0
```

---

## Subtask 1.3: Verify Gatekeeper Components

**From your school assignment:**

```bash
#!/bin/bash
# Save as: task1-subtask3-verify-components.sh

echo "============================================"
echo "  Task 1 - Subtask 1.3: Verify Components"
echo "============================================"
echo ""

echo "▶ Check Gatekeeper CRDs (Custom Resource Definitions)"
kubectl get crd | grep gatekeeper

echo ""
echo "▶ Verify webhook configuration"
kubectl get validatingadmissionwebhooks | grep gatekeeper

echo ""
echo "▶ Check Gatekeeper logs"
kubectl logs -n gatekeeper-system deployment/gatekeeper-controller-manager --tail=20
```

**Run it:**
```bash
chmod +x task1-subtask3-verify-components.sh
./task1-subtask3-verify-components.sh
```

---

## 🧠 Active Recall - Task 1

<details>
<summary><strong>Q1: What are CRDs and why does Gatekeeper need them?</strong></summary>

**CRDs (Custom Resource Definitions)** extend Kubernetes API with new resource types. Gatekeeper creates CRDs like:
- `ConstraintTemplate`: Defines policy logic
- `Config`: Gatekeeper configuration
- Various constraint kinds (created dynamically)

Without CRDs, you couldn't create custom policy objects in Kubernetes.
</details>

<details>
<summary><strong>Q2: Explain the role of ValidatingWebhookConfiguration</strong></summary>

It tells Kubernetes API server:
- "Send all Pod/Deployment/etc. creation requests to Gatekeeper first"
- Which resources to intercept (scope)
- Failure policy (what to do if webhook is down)
- Network location of webhook service

It's the "doorway" through which all resources must pass.
</details>

<details>
<summary><strong>Q3: Why separate audit and controller pods?</strong></summary>

**Different responsibilities:**
- **Controller**: Real-time admission control (block new violations)
- **Audit**: Periodic scanning of existing resources (detect drift)

Separation allows scaling them independently based on load.
</details>

---

# 📦 Task 2: Create Security Policies to Restrict Container Privileges

## 🤔 Prediction Challenge (90 seconds)

Before creating policies:
1. What's the difference between ConstraintTemplate and Constraint?
2. Why use Rego language instead of YAML?
3. Which pods should be exempt from policies?

<details>
<summary>💡 <strong>Answers</strong></summary>

1. **Difference**:
   - **ConstraintTemplate**: Reusable policy LOGIC (like a function definition)
   - **Constraint**: INSTANCE of that template with parameters (like a function call)
2. **Why Rego**: Declarative logic language designed for policy. More powerful than YAML for complex conditions (AND/OR logic, loops, functions)
3. **Exemptions**: kube-system (cluster components), gatekeeper-system (policy engine itself), sometimes specific trusted images
</details>

---

## Subtask 2.1: Create a Constraint Template for Root User Restriction

**From your school assignment - A Constraint Template defines the logic for a policy, while a Constraint applies that template with specific parameters:**

**Create the template file:**

```bash
#!/bin/bash
# Save as: task2-subtask1-root-template.sh

echo "============================================"
echo "  Task 2 - Subtask 2.1: Root User Template"
echo "============================================"
echo ""

echo "▶ Creating no-root-user-template.yaml"

cat > no-root-user-template.yaml << 'EOF'
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequirenonroot
  annotations:
    description: "Requires containers to run as non-root user"
spec:
  crd:
    spec:
      names:
        kind: K8sRequireNonRoot
      validation:
        type: object
        properties:
          exemptImages:
            description: "List of exempt container images"
            type: array
            items:
              type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequirenonroot

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.securityContext.runAsNonRoot
          not is_exempt(container.image)
          msg := sprintf("Container <%v> must run as non-root user", [container.name])
        }

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          container.securityContext.runAsUser == 0
          not is_exempt(container.image)
          msg := sprintf("Container <%v> cannot run as root (UID 0)", [container.name])
        }

        is_exempt(image) {
          exempt_image := input.parameters.exemptImages[_]
          startswith(image, exempt_image)
        }
EOF

echo "✅ File created: no-root-user-template.yaml"
echo ""

echo "▶ Apply the constraint template"
kubectl apply -f no-root-user-template.yaml

echo ""
echo "▶ Verify the template was created"
kubectl get constrainttemplates
```

**Run it:**
```bash
chmod +x task2-subtask1-root-template.sh
./task2-subtask1-root-template.sh
```

---

## 🔍 Deep Dive: Understanding the Rego Policy

Let's break down the policy logic:

```rego
# Rule 1: Container must have runAsNonRoot set
violation[{"msg": msg}] {
  container := input.review.object.spec.containers[_]  # Loop through containers
  not container.securityContext.runAsNonRoot           # Check if missing
  not is_exempt(container.image)                       # Not in exempt list
  msg := sprintf("Container <%v> must run as non-root user", [container.name])
}
```

**How it works:**
- `input.review.object` = The pod being created
- `[_]` = Loop over all containers
- `not X` = Condition fails if X is false/missing
- `is_exempt()` = Helper function to check exemptions

---

## Subtask 2.2: Create a Constraint to Enforce the Policy

**From your school assignment:**

```bash
#!/bin/bash
# Save as: task2-subtask2-root-constraint.sh

echo "============================================"
echo "  Task 2 - Subtask 2.2: Root User Constraint"
echo "============================================"
echo ""

echo "▶ Creating no-root-user-constraint.yaml"

cat > no-root-user-constraint.yaml << 'EOF'
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequireNonRoot
metadata:
  name: must-run-as-non-root
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    excludedNamespaces: ["kube-system", "gatekeeper-system", "kube-public"]
  parameters:
    exemptImages:
      - "gcr.io/google-containers/"
      - "k8s.gcr.io/"
EOF

echo "✅ File created: no-root-user-constraint.yaml"
echo ""

echo "▶ Apply the constraint"
kubectl apply -f no-root-user-constraint.yaml

echo ""
echo "▶ Verify the constraint was created"
kubectl get k8srequirenonroot

echo ""
echo "▶ Check constraint status"
kubectl describe k8srequirenonroot must-run-as-non-root
```

**Run it:**
```bash
chmod +x task2-subtask2-root-constraint.sh
./task2-subtask2-root-constraint.sh
```

---

## Subtask 2.3: Create Additional Security Policies

**From your school assignment - Let's create another policy to restrict privileged containers:**

```bash
#!/bin/bash
# Save as: task2-subtask3-privileged-policy.sh

echo "============================================"
echo "  Task 2 - Subtask 2.3: Privileged Policy"
echo "============================================"
echo ""

echo "▶ Creating no-privileged-template.yaml"

cat > no-privileged-template.yaml << 'EOF'
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequirenonprivileged
  annotations:
    description: "Prohibits privileged containers"
spec:
  crd:
    spec:
      names:
        kind: K8sRequireNonPrivileged
      validation:
        type: object
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequirenonprivileged

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          container.securityContext.privileged
          msg := sprintf("Container <%v> cannot run in privileged mode", [container.name])
        }
EOF

echo "✅ File created: no-privileged-template.yaml"
echo ""

echo "▶ Creating no-privileged-constraint.yaml"

cat > no-privileged-constraint.yaml << 'EOF'
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequireNonPrivileged
metadata:
  name: no-privileged-containers
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    excludedNamespaces: ["kube-system", "gatekeeper-system"]
EOF

echo "✅ File created: no-privileged-constraint.yaml"
echo ""

echo "▶ Apply both files"
kubectl apply -f no-privileged-template.yaml
kubectl apply -f no-privileged-constraint.yaml

echo ""
echo "▶ Verify both constraints are active"
kubectl get constraints
```

**Run it:**
```bash
chmod +x task2-subtask3-privileged-policy.sh
./task2-subtask3-privileged-policy.sh
```

---

## 🧠 Active Recall - Task 2

<details>
<summary><strong>Q1: Why separate template and constraint?</strong></summary>

**Separation of concerns:**
- **Security team**: Writes templates (complex Rego logic) once
- **Operations team**: Creates constraints (simple YAML config) for different environments/namespaces
- **Reusability**: One template can power multiple constraints with different parameters

Example: Same "require-labels" template can enforce different labels per environment.
</details>

<details>
<summary><strong>Q2: What does `privileged: true` actually allow?</strong></summary>

**Privileged containers bypass security:**
- Access to ALL host devices (/dev/*)
- Can modify kernel parameters
- Can load kernel modules
- Essentially root on the host

**Risk**: Container escape becomes trivial. Should only be used for system-level tools like CNI plugins.
</details>

<details>
<summary><strong>Q3: How would you test a policy before enforcing it?</strong></summary>

**Use dryrun mode:**
```yaml
spec:
  enforcementAction: dryrun  # Logs violations but allows creation
```

**Benefits:**
- See what would be blocked
- Assess blast radius
- Educate teams before enforcement
- Validate policy logic
</details>

---

# 📦 Task 3: Automate Compliance Checks and Test Enforcement

## 🤔 Prediction Challenge (60 seconds)

Before testing:
1. Will `runAsUser: 0` be blocked?
2. What about `privileged: true`?
3. What error message format will you see?

<details>
<summary>💡 <strong>Answers</strong></summary>

1. **Yes** - Explicitly violates non-root policy
2. **Yes** - Violates privileged container policy
3. **Format**: "Error from server... admission webhook denied... [policy name]: [violation message]"
</details>

---

## Subtask 3.1: Create Test Applications

**From your school assignment - Let's create applications that violate our policies to test enforcement:**

```bash
#!/bin/bash
# Save as: task3-subtask1-test-apps.sh

echo "============================================"
echo "  Task 3 - Subtask 3.1: Create Test Apps"
echo "============================================"
echo ""

echo "▶ Creating bad-app.yaml (non-compliant)"

cat > bad-app.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: bad-app
  labels:
    app: bad-app
spec:
  containers:
  - name: nginx
    image: nginx:latest
    securityContext:
      runAsUser: 0  # This violates our policy
      privileged: true  # This also violates our policy
EOF

echo "✅ File created: bad-app.yaml"
echo ""

echo "▶ Try to deploy the non-compliant application"
echo ""
echo "You should see an error message indicating policy violations:"
echo ""
kubectl apply -f bad-app.yaml
```

**Run it:**
```bash
chmod +x task3-subtask1-test-apps.sh
./task3-subtask1-test-apps.sh
```

**Expected:** Deployment **DENIED** with violation messages!

---

## Subtask 3.2: Create a Compliant Application

**From your school assignment:**

```bash
#!/bin/bash
# Save as: task3-subtask2-compliant-app.sh

echo "============================================"
echo "  Task 3 - Subtask 3.2: Compliant App"
echo "============================================"
echo ""

echo "▶ Creating good-app.yaml (compliant)"

cat > good-app.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: good-app
  labels:
    app: good-app
spec:
  containers:
  - name: nginx
    image: nginx:latest
    securityContext:
      runAsNonRoot: true
      runAsUser: 1000
      privileged: false
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: var-cache
      mountPath: /var/cache/nginx
    - name: var-run
      mountPath: /var/run
  volumes:
  - name: tmp
    emptyDir: {}
  - name: var-cache
    emptyDir: {}
  - name: var-run
    emptyDir: {}
EOF

echo "✅ File created: good-app.yaml"
echo ""

echo "▶ Deploy the compliant application"
kubectl apply -f good-app.yaml

echo ""
echo "▶ Verify the pod is running"
kubectl get pods

echo ""
echo "▶ Describe pod details"
kubectl describe pod good-app
```

**Run it:**
```bash
chmod +x task3-subtask2-compliant-app.sh
./task3-subtask2-compliant-app.sh
```

**Expected:** Pod created **successfully**!

---

## 🔍 Deep Dive: Why the Good Pod Works

Let's analyze the compliant security context:

```yaml
securityContext:
  runAsNonRoot: true          # ✅ Satisfies non-root policy
  runAsUser: 1000             # ✅ Non-zero UID
  privileged: false           # ✅ Satisfies privileged policy
  allowPrivilegeEscalation: false  # Extra security
  readOnlyRootFilesystem: true     # Defense in depth
  capabilities:
    drop: [ALL]               # Remove all Linux capabilities
```

**Why volumes?**
nginx needs writable directories. With `readOnlyRootFilesystem: true`, we must explicitly mount writable emptyDir volumes for `/tmp`, `/var/cache`, `/var/run`.

---

## Subtask 3.3: Test Policy Enforcement with Deployments

**From your school assignment:**

```bash
#!/bin/bash
# Save as: task3-subtask3-test-deployments.sh

echo "============================================"
echo "  Task 3 - Subtask 3.3: Test Deployments"
echo "============================================"
echo ""

echo "▶ Creating test-deployment.yaml (violates policies)"

cat > test-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-app
  labels:
    app: test-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: test-app
  template:
    metadata:
      labels:
        app: test-app
    spec:
      containers:
      - name: app
        image: busybox:latest
        command: ["sleep", "3600"]
        securityContext:
          runAsUser: 0  # This will be blocked
EOF

echo "✅ File created: test-deployment.yaml"
echo ""

echo "▶ Try to deploy (should fail)"
kubectl apply -f test-deployment.yaml

echo ""
echo ""
echo "▶ Creating test-deployment-fixed.yaml (compliant)"

cat > test-deployment-fixed.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-app-fixed
  labels:
    app: test-app-fixed
spec:
  replicas: 2
  selector:
    matchLabels:
      app: test-app-fixed
  template:
    metadata:
      labels:
        app: test-app-fixed
    spec:
      containers:
      - name: app
        image: busybox:latest
        command: ["sleep", "3600"]
        securityContext:
          runAsNonRoot: true
          runAsUser: 1000
          privileged: false
          allowPrivilegeEscalation: false
EOF

echo "✅ File created: test-deployment-fixed.yaml"
echo ""

echo "▶ Deploy the fixed version"
kubectl apply -f test-deployment-fixed.yaml

echo ""
echo "▶ Verify deployment success"
kubectl get deployments

echo ""
kubectl get pods -l app=test-app-fixed
```

**Run it:**
```bash
chmod +x task3-subtask3-test-deployments.sh
./task3-subtask3-test-deployments.sh
```

---

## Subtask 3.4: Monitor Compliance with Gatekeeper Audit

**From your school assignment - Gatekeeper continuously audits existing resources for policy violations:**

```bash
#!/bin/bash
# Save as: task3-subtask4-audit-compliance.sh

echo "============================================"
echo "  Task 3 - Subtask 3.4: Audit Compliance"
echo "============================================"
echo ""

echo "▶ Check audit results"
kubectl get k8srequirenonroot must-run-as-non-root -o yaml

echo ""
echo ""
echo "▶ View violations in the constraint status"
kubectl describe k8srequirenonroot must-run-as-non-root

echo ""
echo ""
echo "▶ Check privileged container violations"
kubectl describe k8srequirenonprivileged no-privileged-containers
```

**Run it:**
```bash
chmod +x task3-subtask4-audit-compliance.sh
./task3-subtask4-audit-compliance.sh
```

---

## Subtask 3.5: Create a Compliance Dashboard Script

**From your school assignment:**

```bash
#!/bin/bash
# Save as: task3-subtask5-compliance-dashboard.sh

echo "============================================"
echo "  Task 3 - Subtask 3.5: Compliance Dashboard"
echo "============================================"
echo ""

cat > compliance-check.sh << 'EOF'
#!/bin/bash

echo "=== Kubernetes Compliance Status ==="
echo ""

echo "1. Gatekeeper System Status:"
kubectl get pods -n gatekeeper-system --no-headers | awk '{print $1 ": " $3}'
echo ""

echo "2. Active Constraint Templates:"
kubectl get constrainttemplates --no-headers | awk '{print "- " $1}'
echo ""

echo "3. Active Constraints:"
kubectl get constraints --no-headers | awk '{print "- " $1 " (" $2 ")"}'
echo ""

echo "4. Policy Violations Summary:"
for constraint in $(kubectl get constraints -o name); do
    violations=$(kubectl get $constraint -o jsonpath='{.status.totalViolations}' 2>/dev/null)
    name=$(echo $constraint | cut -d'/' -f2)
    echo "- $name: ${violations:-0} violations"
done
echo ""

echo "5. Non-compliant Pods (if any):"
kubectl get pods --all-namespaces -o json | jq -r '.items[] | select(.spec.containers[]?.securityContext.runAsUser == 0 or .spec.containers[]?.securityContext.privileged == true) | "\(.metadata.namespace)/\(.metadata.name)"' 2>/dev/null || echo "No non-compliant pods found or jq not available"
EOF

chmod +x compliance-check.sh

echo "✅ Created: compliance-check.sh"
echo ""

echo "▶ Running compliance dashboard"
./compliance-check.sh
```

**Run it:**
```bash
chmod +x task3-subtask5-compliance-dashboard.sh
./task3-subtask5-compliance-dashboard.sh
```

---

## 🧠 Active Recall - Task 3

<details>
<summary><strong>Q1: Why did the Deployment (not just pod) fail?</strong></summary>

Gatekeeper validates the **pod template** within deployments. The admission webhook sees the embedded pod spec with `runAsUser: 0` and blocks it before the Deployment controller can create any pods.

**Key insight**: Policies apply to pod specs wherever they appear (Deployments, StatefulSets, DaemonSets, Jobs, CronJobs).
</details>

<details>
<summary><strong>Q2: Explain the audit vs admission difference</strong></summary>

**Admission** (Preventive Control):
- Real-time validation during creation
- Blocks non-compliant resources immediately
- Uses ValidatingWebhook

**Audit** (Detective Control):
- Periodic scanning (default: every 60s)
- Detects drift (resources modified after creation, or policies added after resources)
- Updates constraint status with violation counts
</details>

<details>
<summary><strong>Q3: How would you allow a pod in default namespace to run as root temporarily?</strong></summary>

**Option 1** - Add namespace exclusion:
```yaml
excludedNamespaces: ["kube-system", "gatekeeper-system", "default"]
```

**Option 2** - Add image exemption:
```yaml
parameters:
  exemptImages: ["myapp:v1.0"]
```

**Option 3** - Dry-run mode (logs but allows):
```yaml
enforcementAction: dryrun
```

**Best practice**: Use Option 2 with specific image names, document reason, time-box the exemption.
</details>

---

# 📦 Task 4: Advanced Policy Testing and Troubleshooting

## Subtask 4.1: Test Policy Exemptions

**From your school assignment:**

```bash
#!/bin/bash
# Save as: task4-subtask1-test-exemptions.sh

echo "============================================"
echo "  Task 4 - Subtask 4.1: Test Exemptions"
echo "============================================"
echo ""

echo "▶ Create a test namespace"
kubectl create namespace test-exempt

echo ""
echo "▶ Add the namespace to our constraint exemptions"

cat > updated-constraint.yaml << 'EOF'
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequireNonRoot
metadata:
  name: must-run-as-non-root
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    excludedNamespaces: ["kube-system", "gatekeeper-system", "kube-public", "test-exempt"]
  parameters:
    exemptImages:
      - "gcr.io/google-containers/"
      - "k8s.gcr.io/"
EOF

kubectl apply -f updated-constraint.yaml

echo ""
echo "▶ Test deploying a root container in the exempt namespace"
kubectl run test-root --image=nginx --restart=Never -n test-exempt --overrides='{"spec":{"containers":[{"name":"nginx","image":"nginx","securityContext":{"runAsUser":0}}]}}'

echo ""
echo "▶ Verify it was allowed"
kubectl get pods -n test-exempt
```

**Run it:**
```bash
chmod +x task4-subtask1-test-exemptions.sh
./task4-subtask1-test-exemptions.sh
```

---

## Subtask 4.2: Policy Dry-Run Testing

**From your school assignment - Enable dry-run mode to test policies without enforcement:**

```bash
#!/bin/bash
# Save as: task4-subtask2-dryrun-testing.sh

echo "============================================"
echo "  Task 4 - Subtask 4.2: Dry-Run Testing"
echo "============================================"
echo ""

echo "▶ Creating dry-run constraint"

cat > dry-run-constraint.yaml << 'EOF'
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequireNonRoot
metadata:
  name: must-run-as-non-root-dryrun
spec:
  enforcementAction: dryrun
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    excludedNamespaces: ["kube-system", "gatekeeper-system", "kube-public"]
  parameters:
    exemptImages:
      - "gcr.io/google-containers/"
EOF

kubectl apply -f dry-run-constraint.yaml

echo ""
echo "▶ Test with a violating pod - it should be created but logged as violation"
kubectl run test-dryrun --image=nginx --restart=Never --overrides='{"spec":{"containers":[{"name":"nginx","image":"nginx","securityContext":{"runAsUser":0}}]}}'

echo ""
echo "▶ Check if pod was created (it should be)"
kubectl get pod test-dryrun

echo ""
echo "▶ Check violations in constraint status"
kubectl describe k8srequirenonroot must-run-as-non-root-dryrun
```

**Run it:**
```bash
chmod +x task4-subtask2-dryrun-testing.sh
./task4-subtask2-dryrun-testing.sh
```

---

## Subtask 4.3: Troubleshooting Common Issues

**From your school assignment:**

```bash
#!/bin/bash
# Save as: task4-subtask3-troubleshooting-guide.sh

echo "============================================"
echo "  Task 4 - Subtask 4.3: Troubleshooting"
echo "============================================"
echo ""

cat > troubleshoot.sh << 'EOF'
#!/bin/bash

echo "=== Gatekeeper Troubleshooting Guide ==="
echo ""

echo "1. Check Gatekeeper Controller Logs:"
echo "kubectl logs -n gatekeeper-system deployment/gatekeeper-controller-manager"
echo ""

echo "2. Check Webhook Configuration:"
echo "kubectl get validatingadmissionwebhooks"
echo ""

echo "3. Verify Constraint Template Syntax:"
echo "kubectl describe constrainttemplate <template-name>"
echo ""

echo "4. Check Constraint Status:"
echo "kubectl describe <constraint-kind> <constraint-name>"
echo ""

echo "5. Test Policy Logic:"
echo "Use 'kubectl apply --dry-run=server' to test without creating resources"
echo ""

echo "6. Common Issues and Solutions:"
echo "- Policy not enforcing: Check constraint enforcementAction"
echo "- Template errors: Validate Rego syntax"
echo "- Webhook failures: Check network policies and DNS"
echo "- Performance issues: Review constraint scope and complexity"
EOF

chmod +x troubleshoot.sh

echo "✅ Created: troubleshoot.sh"
echo ""
echo "Run './troubleshoot.sh' for troubleshooting commands"
```

**Run it:**
```bash
chmod +x task4-subtask3-troubleshooting-guide.sh
./task4-subtask3-troubleshooting-guide.sh
```

---

## 🧠 Active Recall - Task 4

<details>
<summary><strong>Q1: When should you use dryrun vs exclude?</strong></summary>

**Use dryrun when:**
- Testing new policies before enforcement
- Assessing blast radius
- Gradual rollout (monitor violations first)

**Use exclude when:**
- Legitimate permanent exemption needed
- System namespaces require privileged access
- Specific trusted images

**Best practice**: Dryrun is temporary, exclusions should be minimized and documented.
</details>

<details>
<summary><strong>Q2: How do you debug a failing Rego policy?</strong></summary>

**Steps:**
1. Check constraint status: `kubectl describe <constraint>`
2. Look for Rego syntax errors in template
3. Test with simple pod YAML: `kubectl apply --dry-run=server`
4. Add `trace(...)` statements in Rego for debugging
5. Check controller logs: `kubectl logs -n gatekeeper-system deployment/gatekeeper-controller-manager`
6. Use OPA Playground to test Rego: https://play.openpolicyagent.org
</details>

---

# 📦 Task 5: Comparison with Kyverno (Optional Advanced Section)

## Subtask 5.1: Understanding Kyverno vs OPA Gatekeeper

**From your school assignment:**

```bash
#!/bin/bash
# Save as: task5-subtask1-comparison-doc.sh

echo "============================================"
echo "  Task 5 - Subtask 5.1: Policy Comparison"
echo "============================================"
echo ""

cat > policy-engines-comparison.md << 'EOF'
# Policy Engine Comparison: OPA Gatekeeper vs Kyverno

## OPA Gatekeeper
**Pros:**
- Uses Rego language (powerful and flexible)
- Part of CNCF graduated project
- Strong community and enterprise support
- Excellent for complex policy logic

**Cons:**
- Steeper learning curve (Rego language)
- More complex setup for simple policies
- Requires understanding of OPA concepts

## Kyverno
**Pros:**
- Uses YAML (familiar to Kubernetes users)
- Easier to learn and implement
- Built-in policy library
- Good for simple to moderate complexity policies

**Cons:**
- Less flexible for complex logic
- Smaller community compared to OPA
- Limited advanced features

## When to Choose Which:
- **OPA Gatekeeper**: Complex compliance requirements, existing OPA knowledge, enterprise environments
- **Kyverno**: Simple policies, YAML preference, quick implementation needs
EOF

echo "✅ Created: policy-engines-comparison.md"
cat policy-engines-comparison.md
```

**Run it:**
```bash
chmod +x task5-subtask1-comparison-doc.sh
./task5-subtask1-comparison-doc.sh
```

---

## Subtask 5.2: Quick Kyverno Installation (Demo Only)

**From your school assignment - Note: This is for demonstration purposes. In production, choose one policy engine:**

```bash
#!/bin/bash
# Save as: task5-subtask2-kyverno-demo.sh

echo "============================================"
echo "  Task 5 - Subtask 5.2: Kyverno Demo"
echo "============================================"
echo ""

echo "▶ Install Kyverno (for comparison)"
kubectl create -f https://github.com/kyverno/kyverno/releases/download/v1.10.0/install.yaml

echo ""
echo "▶ Wait for Kyverno to be ready"
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=kyverno -n kyverno --timeout=300s

echo ""
echo "▶ Create a simple Kyverno policy"

cat > kyverno-policy.yaml << 'EOF'
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-non-root-user
spec:
  validationFailureAction: enforce
  background: true
  rules:
  - name: check-non-root
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "Containers must run as non-root user"
      pattern:
        spec:
          containers:
          - securityContext:
              runAsNonRoot: true
EOF

kubectl apply -f kyverno-policy.yaml

echo ""
echo "✅ Kyverno installed and demo policy applied"
echo ""
echo "Compare the YAML syntax to Gatekeeper's Rego!"
```

**Run it:**
```bash
chmod +x task5-subtask2-kyverno-demo.sh
./task5-subtask2-kyverno-demo.sh
```

---

# 🧹 Cleanup and Resource Management

## Subtask 6.1: Clean Up Test Resources

**From your school assignment:**

```bash
#!/bin/bash
# Save as: task6-subtask1-cleanup-tests.sh

echo "============================================"
echo "  Cleanup - Subtask 6.1: Test Resources"
echo "============================================"
echo ""

echo "▶ Remove test pods and deployments"
kubectl delete pod good-app --ignore-not-found
kubectl delete pod test-dryrun --ignore-not-found
kubectl delete pod test-root -n test-exempt --ignore-not-found
kubectl delete deployment test-app-fixed --ignore-not-found

echo ""
echo "▶ Remove test namespace"
kubectl delete namespace test-exempt --ignore-not-found

echo ""
echo "▶ If you installed Kyverno for demo, remove it"
kubectl delete -f https://github.com/kyverno/kyverno/releases/download/v1.10.0/install.yaml --ignore-not-found

echo ""
echo "✅ Test resources cleaned up"
```

**Run it:**
```bash
chmod +x task6-subtask1-cleanup-tests.sh
./task6-subtask1-cleanup-tests.sh
```

---

## Subtask 6.2: Keep Gatekeeper for Future Use

**From your school assignment:**

```bash
#!/bin/bash
# Save as: task6-subtask2-keep-gatekeeper.sh

echo "============================================"
echo "  Cleanup - Subtask 6.2: Keep Gatekeeper"
echo "============================================"
echo ""

echo "Gatekeeper resources to keep:"
echo ""

echo "▶ Constraint Templates:"
kubectl get constrainttemplates

echo ""
echo "▶ Constraints:"
kubectl get constraints

echo ""
echo "▶ Gatekeeper Pods:"
kubectl get pods -n gatekeeper-system

echo ""
echo "These resources remain active for future labs!"
```

**Run it:**
```bash
chmod +x task6-subtask2-keep-gatekeeper.sh
./task6-subtask2-keep-gatekeeper.sh
```

---

## 🧹 Complete Cleanup (Optional)

**If you want to remove everything including the cluster:**

```bash
#!/bin/bash
# Save as: complete-cleanup.sh

echo "============================================"
echo "  COMPLETE CLEANUP (removes cluster)"
echo "============================================"
echo ""

echo "⚠️  This will delete the entire Kubernetes cluster!"
read -p "Continue? (yes/no): " confirm

if [ "$confirm" = "yes" ]; then
    echo ""
    echo "▶ Deleting KIND cluster..."
    kind delete cluster --name gatekeeper-lab
    
    echo ""
    echo "▶ Removing all YAML files..."
    rm -f *.yaml compliance-check.sh troubleshoot.sh *.md
    
    echo ""
    echo "✅ Complete cleanup finished!"
else
    echo "Cleanup cancelled"
fi
```

**Run it:**
```bash
chmod +x complete-cleanup.sh
./complete-cleanup.sh
```

---

# 🎓 Conclusion
In production Kubernetes environments, manual security reviews are insufficient and error-
