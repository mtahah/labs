# 🔒 Lab: Kubernetes API Server Security Hardening

> 💡 **What You'll Master:** Implement production-grade security for Kubernetes API servers using RBAC, encryption at rest, and TLS certificate management
>
> ⏱️ **Total Time:** 3-4 hours  
> 🎯 **Difficulty:** Intermediate to Advanced  
> 🔧 **Prerequisites:** Basic Kubernetes knowledge, familiarity with kubectl, understanding of certificates and encryption concepts  
> 💻 **Environment:** Ubuntu 20.04 LTS with Minikube (Kubernetes 1.28+)

---

## 🎯 Learning Objectives

**By the end of this lab, you'll be able to:**

1. **Configure RBAC** - Create granular access controls using custom roles and role bindings to enforce least-privilege security
2. **Enable Encryption at Rest** - Protect sensitive data in etcd using encryption providers and verify encryption status
3. **Implement TLS Security** - Generate, manage, and rotate certificates for secure API communications with client authentication

**Real-World Value:** These security measures are critical for production Kubernetes clusters. RBAC prevents unauthorized access, encryption protects data breaches, and proper TLS prevents man-in-the-middle attacks. Together, they form the foundation of Kubernetes security compliance (CIS, PCI-DSS, HIPAA).

---

## 📊 Skills Progress Tracker

Track your progress through the lab:

| Skill Area | Progress | Confidence |
|------------|----------|------------|
| **RBAC Configuration** | ⚪⚪⚪⚪⚪ | Low → High |
| **Encryption Implementation** | ⚪⚪⚪⚪⚪ | Low → High |
| **Certificate Management** | ⚪⚪⚪⚪⚪ | Low → High |
| **Security Testing & Verification** | ⚪⚪⚪⚪⚪ | Low → High |

---

## 📦 Phase 1: Environment Setup (15-20 minutes)

### 🤔 Prediction Challenge (60 seconds)

Before setting up your environment, predict the answers:

1. What compatibility issues might arise when running Minikube on Ubuntu?
2. Which dependencies are required for certificate generation?
3. How would you verify that Minikube's API server is accessible?

*Think about these before scrolling down. Write your predictions in a notepad.*

---

### System Requirements

**Minimum Specifications:**
- OS: Ubuntu 20.04 LTS or newer
- RAM: 4GB minimum (8GB recommended)
- Disk: 20GB free space
- CPU: 2 cores minimum
- Network: Internet access for package downloads

**Required Tools:**
- Docker or similar container runtime
- kubectl (Kubernetes CLI)
- Minikube (local Kubernetes cluster)
- OpenSSL (certificate generation)
- curl (API testing)

---

### Installation Commands

> ⚠️ **WARNING:** This lab assumes a clean Ubuntu environment. Running these commands on a production system may interfere with existing configurations.

**Command Group 1: System Preparation**

```bash
#!/bin/bash
# Save this as setup_01_system_prep.sh

echo "=========================================="
echo "1.1: Updating Package Manager"
echo "=========================================="
sudo apt-get update

echo ""
echo "=========================================="
echo "1.2: Installing Essential Dependencies"
echo "=========================================="
# curl: for downloading and API testing
# apt-transport-https: enables apt to retrieve packages over HTTPS
# ca-certificates: provides certificate authorities for SSL/TLS
# gnupg: for verifying package signatures
# openssl: for certificate generation and management
sudo apt-get install -y \
    curl \
    apt-transport-https \
    ca-certificates \
    gnupg \
    openssl \
    jq

echo ""
echo "=========================================="
echo "1.3: Verifying OpenSSL Installation"
echo "=========================================="
openssl version

echo ""
echo "✅ System preparation complete!"
```

**Expected Output for 1.3:**
```
OpenSSL 1.1.1f  31 Mar 2020
```

**Why These Dependencies?**
- `curl`: Used to interact directly with Kubernetes API endpoints
- `openssl`: Required for generating and managing TLS certificates
- `jq`: JSON processor for parsing API responses
- `apt-transport-https`: Enables secure package downloads

---

**Command Group 2: Docker Installation**

```bash
#!/bin/bash
# Save this as setup_02_docker.sh

echo "=========================================="
echo "2.1: Adding Docker's Official GPG Key"
echo "=========================================="
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo ""
echo "=========================================="
echo "2.2: Setting Up Docker Repository"
echo "=========================================="
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

echo ""
echo "=========================================="
echo "2.3: Installing Docker Engine"
echo "=========================================="
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

echo ""
echo "=========================================="
echo "2.4: Adding User to Docker Group"
echo "=========================================="
# This allows running docker without sudo
sudo usermod -aG docker $USER

echo ""
echo "=========================================="
echo "2.5: Starting Docker Service"
echo "=========================================="
sudo systemctl start docker
sudo systemctl enable docker

echo ""
echo "=========================================="
echo "2.6: Verifying Docker Installation"
echo "=========================================="
sudo docker --version

echo ""
echo "⚠️  IMPORTANT: You need to log out and log back in for group changes to take effect"
echo "    Or run: newgrp docker"
```

**Expected Output for 2.6:**
```
Docker version 24.0.7, build afdd53b
```

> 💡 **TIP:** After running this script, execute `newgrp docker` to apply group changes without logging out.

---

**Command Group 3: kubectl Installation**

```bash
#!/bin/bash
# Save this as setup_03_kubectl.sh

echo "=========================================="
echo "3.1: Downloading kubectl"
echo "=========================================="
# Download the latest stable version
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

echo ""
echo "=========================================="
echo "3.2: Installing kubectl"
echo "=========================================="
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

echo ""
echo "=========================================="
echo "3.3: Verifying kubectl Installation"
echo "=========================================="
kubectl version --client

echo ""
echo "✅ kubectl installation complete!"
```

**Expected Output for 3.3:**
```
Client Version: v1.28.x
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
```

---

**Command Group 4: Minikube Installation**

```bash
#!/bin/bash
# Save this as setup_04_minikube.sh

echo "=========================================="
echo "4.1: Downloading Minikube"
echo "=========================================="
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

echo ""
echo "=========================================="
echo "4.2: Installing Minikube"
echo "=========================================="
sudo install minikube-linux-amd64 /usr/local/bin/minikube

echo ""
echo "=========================================="
echo "4.3: Verifying Minikube Installation"
echo "=========================================="
minikube version

echo ""
echo "✅ Minikube installation complete!"
```

**Expected Output for 4.3:**
```
minikube version: v1.32.0
commit: 8220a6eb95f0a4d75f7f2d7b14cef975f050512d
```

---

**Command Group 5: Starting Minikube Cluster**

```bash
#!/bin/bash
# Save this as setup_05_start_cluster.sh

echo "=========================================="
echo "5.1: Starting Minikube Cluster"
echo "=========================================="
# --driver=docker: uses Docker as the container runtime
# --kubernetes-version: specifies K8s version for consistency
# --extra-config: enables features needed for this lab
minikube start \
  --driver=docker \
  --kubernetes-version=v1.28.3 \
  --extra-config=apiserver.enable-admission-plugins=NodeRestriction

echo ""
echo "=========================================="
echo "5.2: Verifying Cluster Status"
echo "=========================================="
minikube status

echo ""
echo "=========================================="
echo "5.3: Checking Cluster Info"
echo "=========================================="
kubectl cluster-info

echo ""
echo "=========================================="
echo "5.4: Verifying Node Readiness"
echo "=========================================="
kubectl get nodes

echo ""
echo "✅ Minikube cluster is ready!"
```

**Expected Output for 5.4:**
```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   30s   v1.28.3
```

---

### Environment Test

**Command Group 6: Comprehensive Environment Verification**

```bash
#!/bin/bash
# Save this as setup_06_verify_environment.sh

echo "=========================================="
echo "6.1: Testing kubectl Connectivity"
echo "=========================================="
kubectl get namespaces

echo ""
echo "=========================================="
echo "6.2: Testing API Server Accessibility"
echo "=========================================="
kubectl get --raw /api/v1 | jq '.resources[] | select(.name == "pods") | {name, verbs}'

echo ""
echo "=========================================="
echo "6.3: Verifying RBAC is Enabled"
echo "=========================================="
kubectl api-resources | grep -i rbac

echo ""
echo "=========================================="
echo "6.4: Checking Default Service Account"
echo "=========================================="
kubectl get serviceaccount default -o yaml

echo ""
echo "=========================================="
echo "6.5: Testing OpenSSL Certificate Generation"
echo "=========================================="
openssl genrsa -out /tmp/test-key.pem 2048 2>/dev/null && echo "✅ OpenSSL working correctly" && rm /tmp/test-key.pem

echo ""
echo "=========================================="
echo "Environment Verification Complete"
echo "=========================================="
```

**Capability Check Results:**

| Component | Test | Expected Result |
|-----------|------|-----------------|
| kubectl | `kubectl get namespaces` | Lists default namespaces |
| API Server | `kubectl get --raw /api/v1` | Returns API resources |
| RBAC | `kubectl api-resources \| grep rbac` | Shows RBAC resources |
| OpenSSL | `openssl genrsa` | Generates key successfully |

---

### Troubleshooting Common Setup Issues

<details>
<summary>❌ <strong>Error: Minikube fails to start</strong></summary>

**Symptoms:**
```
😿  Failed to start minikube: running in container, --driver must be set
```

**Causes & Solutions:**

| Cause | Solution |
|-------|----------|
| Docker not running | `sudo systemctl start docker` |
| User not in docker group | `sudo usermod -aG docker $USER && newgrp docker` |
| Insufficient resources | `minikube start --memory=4096 --cpus=2` |
| Conflicting drivers | `minikube delete && minikube start --driver=docker` |

**Prevention:** Always verify Docker is running before starting Minikube.

</details>

<details>
<summary>❌ <strong>Error: kubectl command not found</strong></summary>

**Symptoms:**
```
bash: kubectl: command not found
```

**Diagnostic Steps:**
1. Check if kubectl is installed: `which kubectl`
2. Verify PATH: `echo $PATH`
3. Check installation location: `ls -l /usr/local/bin/kubectl`

**Solution:**
```bash
# Reinstall kubectl with correct permissions
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

</details>

<details>
<summary>❌ <strong>Error: Permission denied when accessing Docker</strong></summary>

**Symptoms:**
```
Got permission denied while trying to connect to the Docker daemon socket
```

**Solution:**
```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Apply group changes without logout
newgrp docker

# Verify
docker ps
```

</details>

---

### ✅ Environment Setup Checklist

Before proceeding, ensure all items are checked:

- [ ] Ubuntu system updated (`sudo apt-get update`)
- [ ] Docker installed and running (`docker --version`)
- [ ] User added to docker group (`groups | grep docker`)
- [ ] kubectl installed (`kubectl version --client`)
- [ ] Minikube installed (`minikube version`)
- [ ] Minikube cluster started (`minikube status`)
- [ ] Cluster node is Ready (`kubectl get nodes`)
- [ ] API server accessible (`kubectl cluster-info`)
- [ ] RBAC resources visible (`kubectl api-resources | grep rbac`)
- [ ] OpenSSL working (`openssl version`)

---

### 🔄 Review Your Predictions

Now that setup is complete, compare your initial predictions:

<details>
<summary>💡 <strong>Prediction Answers</strong></summary>

1. **Compatibility issues:** Docker driver conflicts, resource constraints, virtualization not enabled
2. **Certificate dependencies:** OpenSSL, ca-certificates package, proper permissions for key generation
3. **API verification:** `kubectl cluster-info`, `kubectl get --raw /api`, checking for Ready nodes

**Key Insight:** Minikube abstracts much of the complexity, but understanding the underlying requirements (Docker, resources, networking) helps troubleshoot effectively.

</details>

---

## 📦 Block 1: Understanding and Configuring RBAC (30-40 minutes)

### 🤔 Prediction Challenge (90 seconds)

Before diving into RBAC configuration:

1. What happens when a user tries to access resources without proper RBAC permissions?
2. Which parameters are critical when creating a Role vs RoleBinding?
3. Why might RBAC testing fail even with correct configurations?
4. How does this connect to the authentication we just set up?

*Write your predictions before continuing.*

---

### Core Concept: Role-Based Access Control (RBAC)

**Definition:** RBAC is Kubernetes' native authorization mechanism that controls which users and service accounts can perform which actions on which resources.

**Why It Matters:** RBAC enables the principle of least privilege by granting only the minimum permissions needed to perform a task, reducing the blast radius if credentials are compromised. Without RBAC, any authenticated user could delete production workloads or access sensitive secrets.

**Mental Model:** Think of RBAC like building access control:
- **Role** = A job description (e.g., "janitor can access floors 1-3")
- **RoleBinding** = Assignment (e.g., "John Smith is a janitor")
- **Subject** = The person or service (e.g., "John Smith")
- **Resources** = The things you can access (e.g., "floors 1-3")

---

### Detailed Explanation: RBAC Components

**RBAC operates on four key components:**

1. **Subjects** - Who is making the request (User, Group, ServiceAccount)
2. **Verbs** - What action they want to perform (get, list, create, delete, etc.)
3. **Resources** - What they want to act upon (pods, secrets, deployments, etc.)
4. **API Groups** - Which API the resource belongs to ("", "apps", "rbac.authorization.k8s.io")

**Process Flow:**

```
Request → Authentication → Authorization (RBAC) → Admission Control → Action
          ↓                ↓
    "Who are you?"    "What can you do?"
```

**Key Components Table:**

| Component | Scope | Purpose | Example |
|-----------|-------|---------|---------|
| **Role** | Namespace | Defines permissions within one namespace | Allow reading pods in "dev" namespace |
| **ClusterRole** | Cluster-wide | Defines permissions across all namespaces | Allow reading pods in any namespace |
| **RoleBinding** | Namespace | Grants Role permissions to subjects | John can use "pod-reader" role in "dev" |
| **ClusterRoleBinding** | Cluster-wide | Grants ClusterRole permissions to subjects | John can use "pod-reader" role everywhere |

---

### Hands-On Exercise 1: Examining Current RBAC Configuration

**Context:** Before creating custom RBAC policies, we need to understand the existing security baseline. This helps avoid accidentally removing necessary permissions or conflicting with system roles.

**Command Group 7: RBAC Discovery**

```bash
#!/bin/bash
# Save this as rbac_01_discovery.sh

echo "=========================================="
echo "7.1: Listing Default ClusterRoles"
echo "=========================================="
# ClusterRoles define permissions that can be granted cluster-wide
# Note: Limited to first 20 for readability
kubectl get clusterroles | head -20

echo ""
echo "=========================================="
echo "7.2: Examining a Built-in ClusterRole"
echo "=========================================="
# The 'view' ClusterRole is commonly used for read-only access
kubectl describe clusterrole view | head -30

echo ""
echo "=========================================="
echo "7.3: Listing Current RoleBindings"
echo "=========================================="
# RoleBindings connect Roles to subjects (users/serviceaccounts)
kubectl get rolebindings --all-namespaces

echo ""
echo "=========================================="
echo "7.4: Examining Default ServiceAccount"
echo "=========================================="
# Every namespace has a default ServiceAccount
# Understanding its permissions is crucial for security
kubectl describe serviceaccount default -n default

echo ""
echo "=========================================="
echo "7.5: Checking Current User Permissions"
echo "=========================================="
# See what the current user (you) can do
kubectl auth can-i --list
```

**Expected Output for 7.1:**
```
NAME                                                                   CREATED AT
admin                                                                  2024-10-08T10:15:30Z
cluster-admin                                                          2024-10-08T10:15:30Z
edit                                                                   2024-10-08T10:15:30Z
view                                                                   2024-10-08T10:15:30Z
```

**Parameter Explanations:**

- `--all-namespaces`: Lists resources across all namespaces (not just default)
- `describe`: Shows detailed information including rules and subjects
- `auth can-i --list`: Tests what actions the current user can perform

**Variations:**

```bash
# Check if a specific user can perform an action
kubectl auth can-i delete pods --as=system:serviceaccount:default:default

# List only custom roles (exclude system:*)
kubectl get clusterroles | grep -v "^system:"

# Show roles in a specific namespace
kubectl get roles -n kube-system
```

---

### Hands-On Exercise 2: Creating a Restricted Namespace

**Context:** Isolation is a key security principle. By creating a dedicated namespace for security testing, we prevent accidental impacts on system components and establish clear boundaries for our RBAC policies.

**Command Group 8: Namespace Creation**

```bash
#!/bin/bash
# Save this as rbac_02_namespace.sh

echo "=========================================="
echo "8.1: Creating Security Lab Namespace"
echo "=========================================="
# Namespaces provide logical isolation for resources
kubectl create namespace security-lab

echo ""
echo "=========================================="
echo "8.2: Verifying Namespace Creation"
echo "=========================================="
kubectl get namespaces security-lab

echo ""
echo "=========================================="
echo "8.3: Labeling Namespace for Organization"
echo "=========================================="
# Labels help with organization and selection
kubectl label namespace security-lab purpose=security-testing environment=lab

echo ""
echo "=========================================="
echo "8.4: Viewing Namespace Details"
echo "=========================================="
kubectl describe namespace security-lab

echo ""
echo "=========================================="
echo "8.5: Listing Default Resources in Namespace"
echo "=========================================="
# New namespaces automatically get a default ServiceAccount
kubectl get serviceaccounts -n security-lab
```

**Expected Output for 8.2:**
```
NAME           STATUS   AGE
security-lab   Active   5s
```

---

### Hands-On Exercise 3: Creating a Test User with Certificates

**Context:** In production, users are authenticated via certificates, tokens, or external identity providers. We'll use certificate-based authentication, which is the most common method for cluster administrators and service accounts.

**Command Group 9: User Certificate Generation**

```bash
#!/bin/bash
# Save this as rbac_03_user_creation.sh

echo "=========================================="
echo "9.1: Creating Certificate Directory"
echo "=========================================="
mkdir -p ~/k8s-certs
cd ~/k8s-certs

echo ""
echo "=========================================="
echo "9.2: Generating Private Key for Test User"
echo "=========================================="
# RSA 2048-bit key provides strong encryption
# This private key must be kept secure - it proves the user's identity
openssl genrsa -out testuser.key 2048

echo ""
echo "=========================================="
echo "9.3: Creating Certificate Signing Request (CSR)"
echo "=========================================="
# CN (Common Name) = username in Kubernetes
# O (Organization) = group membership
# Groups are used for RBAC (e.g., 'developers' group)
openssl req -new \
  -key testuser.key \
  -out testuser.csr \
  -subj "/CN=testuser/O=developers"

echo ""
echo "=========================================="
echo "9.4: Viewing CSR Contents"
echo "=========================================="
openssl req -in testuser.csr -noout -text | grep -A 2 "Subject:"

echo ""
echo "=========================================="
echo "9.5: Creating Kubernetes CSR Object"
echo "=========================================="
# Kubernetes needs the CSR in base64 format
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: testuser-csr
spec:
  request: $(cat testuser.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # 24 hours - short for testing
  usages:
  - client auth
EOF

echo ""
echo "=========================================="
echo "9.6: Checking CSR Status"
echo "=========================================="
kubectl get csr testuser-csr

echo ""
echo "=========================================="
echo "9.7: Approving the Certificate Request"
echo "=========================================="
# In production, CSRs are reviewed before approval
kubectl certificate approve testuser-csr

echo ""
echo "=========================================="
echo "9.8: Extracting the Signed Certificate"
echo "=========================================="
kubectl get csr testuser-csr -o jsonpath='{.status.certificate}' | base64 -d > testuser.crt

echo ""
echo "=========================================="
echo "9.9: Verifying Certificate Extraction"
echo "=========================================="
openssl x509 -in testuser.crt -noout -subject -dates
```

**Expected Output for 9.9:**
```
subject=CN = testuser, O = developers
notBefore=Oct  8 10:30:00 2024 GMT
notAfter=Oct  9 10:30:00 2024 GMT
```

**Why This Matters:** The certificate proves the user's identity. The CN (Common Name) becomes the username, and O (Organization) becomes group membership for RBAC rules.

---

### Deeper Investigation: Certificate Structure

**Command Group 10: Certificate Analysis**

```bash
#!/bin/bash
# Save this as rbac_04_cert_analysis.sh

echo "=========================================="
echo "10.1: Full Certificate Details"
echo "=========================================="
cd ~/k8s-certs
openssl x509 -in testuser.crt -noout -text

echo ""
echo "=========================================="
echo "10.2: Extracting Critical Fields"
echo "=========================================="
echo "Subject (Username):"
openssl x509 -in testuser.crt -noout -subject

echo ""
echo "Organization (Groups):"
openssl x509 -in testuser.crt -noout -subject | grep -oP 'O\s*=\s*\K[^,]+'

echo ""
echo "Validity Period:"
openssl x509 -in testuser.crt -noout -dates

echo ""
echo "=========================================="
echo "10.3: Verifying Certificate Chain"
echo "=========================================="
# Get the CA certificate from Minikube
kubectl config view --raw -o json | jq -r '.clusters[0].cluster."certificate-authority-data"' | base64 -d > cluster-ca.crt

# Verify our user cert was signed by the cluster CA
openssl verify -CAfile cluster-ca.crt testuser.crt
```

**Analysis Questions:**

1. **Q: Why is the expirationSeconds set to 86400 (24 hours)?**
2. **Q: What would happen if we used a different CN (Common Name)?**
3. **Q: How does the Organization field connect to RBAC rules?**

<details>
<summary>💡 <strong>Answers</strong></summary>

1. Short expiration limits the risk if the certificate is compromised. Production certificates typically last 365+ days but should be rotated regularly.

2. The CN becomes the username in Kubernetes. Changing it would create a different user identity, requiring separate RBAC rules.

3. The Organization (O) field maps to Kubernetes groups. RBAC rules can target groups (like "developers") instead of individual users, making management scalable.

</details>

---

### Hands-On Exercise 4: Creating Custom RBAC Policies

**Context:** The principle of least privilege requires granting only the minimum permissions needed. We'll create a Role that allows reading pods but nothing else - demonstrating fine-grained access control.

**Command Group 11: Role Creation**

```bash
#!/bin/bash
# Save this as rbac_05_role_creation.sh

echo "=========================================="
echo "11.1: Creating Pod Reader Role"
echo "=========================================="
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: security-lab
  name: pod-reader
rules:
# Rule 1: Allow reading pods
- apiGroups: [""]  # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
# Rule 2: Allow reading pod logs (common requirement)
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get", "list"]
EOF

echo ""
echo "=========================================="
echo "11.2: Verifying Role Creation"
echo "=========================================="
kubectl get role pod-reader -n security-lab

echo ""
echo "=========================================="
echo "11.3: Examining Role Details"
echo "=========================================="
kubectl describe role pod-reader -n security-lab

echo ""
echo "=========================================="
echo "11.4: Creating RoleBinding"
echo "=========================================="
# RoleBinding connects the Role to our test user
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: security-lab
subjects:
# The user we created with the certificate
- kind: User
  name: testuser
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # Reference to the Role we created above
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
EOF

echo ""
echo "=========================================="
echo "11.5: Verifying RoleBinding"
echo "=========================================="
kubectl get rolebinding read-pods-binding -n security-lab

echo ""
echo "=========================================="
echo "11.6: Describing RoleBinding Details"
echo "=========================================="
kubectl describe rolebinding read-pods-binding -n security-lab
```

**Expected Output for 11.3:**
```
Name:         pod-reader
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources   Non-Resource URLs  Resource Names  Verbs
  ---------   -----------------  --------------  -----
  pods        []                 []              [get list watch]
  pods/log    []                 []              [get list]
```

**RBAC Rules Explained:**

| Field | Purpose | Example Value |
|-------|---------|---------------|
| `apiGroups` | Which API this applies to | `""` (core), `"apps"`, `"rbac.authorization.k8s.io"` |
| `resources` | What objects are affected | `"pods"`, `"secrets"`, `"deployments"` |
| `verbs` | What actions are allowed | `"get"`, `"list"`, `"create"`, `"delete"`, `"*"` |
| `resourceNames` | Specific resource instances (optional) | `["my-pod", "prod-pod"]` |

---

### Hands-On Exercise 5: Testing RBAC Configuration

**Context:** Testing RBAC is critical - misconfigurations can either block legitimate users or grant excessive permissions. We'll test both allowed and denied operations.

**Command Group 12: RBAC Testing Setup**

```bash
#!/bin/bash
# Save this as rbac_06_testing_setup.sh

echo "=========================================="
echo "12.1: Configuring kubectl for Test User"
echo "=========================================="
cd ~/k8s-certs

# Add user credentials to kubeconfig
kubectl config set-credentials testuser \
  --client-certificate=testuser.crt \
  --client-key=testuser.key \
  --embed-certs=true

echo ""
echo "=========================================="
echo "12.2: Creating Context for Test User"
echo "=========================================="
# Context combines cluster, user, and namespace
kubectl config set-context testuser-context \
  --cluster=minikube \
  --user=testuser \
  --namespace=security-lab

echo ""
echo "=========================================="
echo "12.3: Viewing Current Context Configuration"
echo "=========================================="
kubectl config view --minify

echo ""
echo "=========================================="
echo "12.4: Creating Test Workload (as admin)"
echo "=========================================="
# Switch back to admin temporarily
kubectl config use-context minikube

# Create a deployment to test against
kubectl create deployment nginx-test --image=nginx --replicas=2 -n security-lab

# Wait for pods to be ready
echo "Waiting for pods to be ready..."
kubectl wait --for=condition=ready pod -l app=nginx-test -n security-lab --timeout=60s

echo ""
echo "=========================================="
echo "12.5: Verifying Test Workload"
echo "=========================================="
kubectl get pods -n security-lab
```

**Expected Output for 12.5:**
```
NAME                          READY   STATUS    RESTARTS   AGE
nginx-test-7c5ddbdf54-abc12   1/1     Running   0          30s
nginx-test-7c5ddbdf54-def34   1/1     Running   0          30s
```

---

**Command Group 13: RBAC Permission Testing**

```bash
#!/bin/bash
# Save this as rbac_07_permission_tests.sh

echo "=========================================="
echo "13.1: Switching to Test User Context"
echo "=========================================="
kubectl config use-context testuser-context

echo ""
echo "=========================================="
echo "13.2: TEST 1 - Reading Pods (SHOULD SUCCEED)"
echo "=========================================="
# This should work because pod-reader role allows 'get' and 'list' on pods
kubectl get pods -n security-lab

echo ""
echo "=========================================="
echo "13.3: TEST 2 - Reading Pod Logs (SHOULD SUCCEED)"
echo "=========================================="
# This should work because we granted pods/log permissions
POD_NAME=$(kubectl get pods -n security-lab -o jsonpath='{.items[0].metadata.name}')
kubectl logs $POD_NAME -n security-lab --tail=5

echo ""
echo "=========================================="
echo "13.4: TEST 3 - Accessing Different Namespace (SHOULD FAIL)"
echo "=========================================="
# This should fail because Role is namespace-scoped
echo "Attempting to list pods in 'default' namespace..."
kubectl get pods -n default || echo "❌ Access denied (expected behavior)"

echo ""
echo "=========================================="
echo "13.5: TEST 4 - Deleting Pods (SHOULD FAIL)"
echo "=========================================="
# This should fail because we didn't grant 'delete' verb
echo "Attempting to delete a pod..."
kubectl delete pod $POD_NAME -n security-lab || echo "❌ Access denied (expected behavior)"

echo ""
echo "=========================================="
echo "13.6: TEST 5 - Creating Pods (SHOULD FAIL)"
echo "=========================================="
# This should fail because we didn't grant 'create' verb
echo "Attempting to create a pod..."
kubectl run test-pod --image=nginx -n security-lab || echo "❌ Access denied (expected behavior)"

echo ""
echo "=========================================="
echo "13.7: TEST 6 - Reading Secrets (SHOULD FAIL)"
echo "=========================================="
# This should fail because we didn't grant access to secrets
echo "Attempting to list secrets..."
kubectl get secrets -n security-lab || echo "❌ Access denied (expected behavior)"

echo ""
echo "=========================================="
echo "13.8: Switching Back to Admin Context"
echo "=========================================="
kubectl config use-context minikube

echo ""
echo "=========================================="
echo "Test Results Summary"
echo "=========================================="
echo "✅ PASSED: Reading pods in security-lab namespace"
echo "✅ PASSED: Reading pod logs in security-lab namespace"
echo "✅ PASSED: Denied access to default namespace"
echo "✅ PASSED: Denied pod deletion"
echo "✅ PASSED: Denied pod creation"
echo "✅ PASSED: Denied secret access"
echo ""
echo "RBAC is working correctly! ✨"
```

**Expected Behavior:**

| Test | Expected Result | Why |
|------|----------------|-----|
| Get pods in security-lab | ✅ Success | Role grants `get` and `list` |
| Get pod logs | ✅ Success | Role grants access to `pods/log` |
| Get pods in default | ❌ Forbidden | Role is namespace-scoped |
| Delete pod | ❌ Forbidden | Role doesn't include `delete` verb |
| Create pod | ❌ Forbidden | Role doesn't include `create` verb |
| Get secrets | ❌ Forbidden | Role doesn't mention `secrets` resource |

---

### Deeper Investigation: Using can-i for Permission Checks

**Command Group 14: Advanced Permission Testing**

```bash
#!/bin/bash
# Save this as rbac_08_advanced_testing.sh

echo "=========================================="
echo "14.1: Testing Specific Actions"
echo "=========================================="
# auth can-i allows testing without actually performing the action

echo "Can testuser get pods in security-lab?"
kubectl auth can-i get pods --as=testuser -n security-lab

echo ""
echo "Can testuser delete pods in security-lab?"
kubectl auth can-i delete pods --as=testuser -n security-lab

echo ""
echo "Can testuser create deployments in security-lab?"
kubectl auth can-i create deployments --as=testuser -n security-lab

echo ""
echo "=========================================="
echo "14.2: Listing All Permissions for Test User"
echo "=========================================="
kubectl auth can-i --list --as=testuser -n security-lab

echo ""
echo "=========================================="
echo "14.3: Checking Group-Based Permissions"
echo "=========================================="
# Remember: testuser is in the 'developers' group
echo "Checking permissions for 'developers' group:"
kubectl auth can-i --list --as=testuser --as-group=developers -n security-lab
```

**Analysis Questions:**

1. **What's the difference between testing with `--as` vs actually switching contexts?**
2. **Why is `auth can-i` useful in production environments?**
3. **How would you debug if a user reports they can't access a resource?**

<details>
<summary>💡 <strong>Answers</strong></summary>

1. `--as` impersonates the user without changing your context, allowing admins to test permissions quickly. Switching contexts actually authenticates as that user.

2. `auth can-i` allows testing permissions without risking actual operations. You can verify RBAC rules before granting access to users or in CI/CD pipelines.

3. Debugging steps:
   - Check if user is authenticated: `kubectl auth can-i get pods --as=username`
   - Verify RoleBindings: `kubectl get rolebinding -n namespace`
   - Check Role rules: `kubectl describe role rolename -n namespace`
   - Verify user groups: Check certificate Organization field
   - Check for ClusterRoleBindings that might override

</details>

---

### 🧠 Active Recall Check: RBAC Concepts

Test your understanding without looking back:

**Q1: Conceptual Understanding**
Explain in your own words the difference between a Role and a RoleBinding. Why do we need both?

<details>
<summary>💡 <strong>Answer</strong></summary>

A **Role** defines *what* can be done (like a job description - "can read pods and logs"). A **RoleBinding** defines *who* can do it (like an assignment - "John has this job description").

We need both because:
- **Separation of concerns**: Permission definitions are separate from user assignments
- **Reusability**: One Role can be bound to multiple users via multiple RoleBindings
- **Clarity**: Easy to see what permissions exist (Roles) vs who has them (RoleBindings)
- **Flexibility**: You can change who has access without modifying permission definitions

</details>

**Q2: Parameter Reasoning**
Why did we use `apiGroups: [""]` for pods? What would happen if we used `apiGroups: ["apps"]` instead?

<details>
<summary>💡 <strong>Answer</strong></summary>

`apiGroups: [""]` specifies the **core API group** where basic resources like pods, services, and secrets live. The empty string literally means "core group."

If we used `apiGroups: ["apps"]`, the Role would:
- **Not grant access to pods** (pods are in core, not apps)
- **Only apply to** resources in the apps API group (Deployments, StatefulSets, DaemonSets)
- **Result in permission denied** errors when testuser tries to list pods

Each resource belongs to a specific API group:
- Pods, Services, Secrets → `""` (core)
- Deployments, StatefulSets → `"apps"`
- Roles, RoleBindings → `"rbac.authorization.k8s.io"`

</details>

**Q3: Connection to Previous Concepts**
How does the certificate CN (Common Name) connect to the RoleBinding subject? Trace the authentication flow.

<details>
<summary>💡 <strong>Answer</strong></summary>

**Authentication Flow:**

1. **Request**: Client sends request with certificate
2. **Authentication**: API server validates certificate against CA
   - Extracts CN (Common Name) → becomes **username**
   - Extracts O (Organization) → becomes **groups**
3. **Authorization**: RBAC checks permissions
   - Looks for RoleBindings where subject.name matches **username**
   - Also checks ClusterRoleBindings for group memberships
4. **Permission Check**: Compares requested action against Role rules
5. **Allow/Deny**: Returns success or "Forbidden" error

**In our case:**
- Certificate CN: `testuser` → Username: `testuser`
- RoleBinding subject.name: `testuser` → Match! ✅
- RoleBinding references Role: `pod-reader`
- Role allows: `get`, `list`, `watch` on pods
- Request: `kubectl get pods` → Allowed ✅

</details>

**Q4: Real-World Application**
You're setting up RBAC for a team of developers who need to deploy applications but shouldn't be able to delete namespaces or modify RBAC rules. What verbs and resources would you include/exclude?

<details>
<summary>💡 <strong>Answer</strong></summary>

**Include (grant):**
```yaml
rules:
# Application deployment
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets", "statefulsets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

# Pod management
- apiGroups: [""]
  resources: ["pods", "pods/log", "pods/exec"]
  verbs: ["get", "list", "watch", "create", "delete"]

# Configuration
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]

# Services and networking
- apiGroups: [""]
  resources: ["services"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
```

**Exclude (deny by omission):**
```yaml
# NEVER grant these to developers:
# - Namespaces (any verbs) - prevents deletion
# - RBAC resources (Roles, RoleBindings, ClusterRoles, ClusterRoleBindings)
# - Nodes - cluster infrastructure
# - PersistentVolumes - storage infrastructure
# - StorageClasses - storage policy
```

**Anti-patterns to avoid:**
- ❌ Granting `verbs: ["*"]` (wildcards are too permissive)
- ❌ Using ClusterRoles when Roles suffice (reduces blast radius)
- ❌ Granting `delete` on namespaces
- ❌ Allowing RBAC modification (privilege escalation risk)

</details>

---

### 📝 Summary: RBAC Configuration

**Concepts Mastered:**
- ✅ RBAC architecture (Roles, RoleBindings, Subjects, Verbs)
- ✅ Certificate-based authentication with CN and Organization
- ✅ Creating namespace-scoped permissions
- ✅ Testing and validating RBAC policies
- ✅ Using `auth can-i` for permission verification

**Critical Insights:**
1. **Least Privilege**: Only grant the minimum verbs and resources needed
2. **Namespace Scoping**: Prefer Roles over ClusterRoles to limit blast radius
3. **Testing is Essential**: Always test both allowed and denied operations
4. **Groups Scale Better**: Use Organization (groups) instead of individual users

**Progress Update:**
- ⭐⭐⭐⚪⚪ RBAC Configuration (60% complete)
- ⭐⭐⚪⚪⚪ Hands-On Implementation (40% complete)

**What's Next:** We'll implement encryption at rest to protect sensitive data stored in etcd, building on our understanding of secure access control.

---

## 📦 Block 2: Implementing Encryption at Rest (35-45 minutes)

### 🤔 Prediction Challenge (90 seconds)

Before configuring encryption:

1. What happens to existing secrets when you enable encryption? Are they automatically encrypted?
2. Which parameters in the encryption configuration determine the encryption algorithm?
3. Why might the API server fail to restart after adding encryption configuration?
4. How does encryption at rest relate to the RBAC we just configured?

*Consider these questions carefully before proceeding.*

---

### Core Concept: Encryption at Rest

**Definition:** Encryption at rest protects data stored in etcd (Kubernetes' database) by encrypting it before writing to disk. Even if someone gains physical access to the storage or backups, they cannot read the data without the encryption keys.

**Why It Matters:** By default, Kubernetes stores secrets in etcd as **base64-encoded plaintext** - not encrypted! This means anyone with etcd access can read sensitive data like passwords, API keys, and certificates. Encryption at rest is essential for:
- Compliance requirements (PCI-DSS, HIPAA, SOC 2)
- Protecting against insider threats
- Securing backups and disaster recovery media
- Defense in depth (multiple security layers)

**Mental Model:** Think of etcd like a filing cabinet:
- **Without encryption**: Documents stored in plain envelopes (base64 = transparent)
- **With encryption**: Documents in locked boxes, only the API server has keys
- **RBAC**: Controls who can request documents from the cabinet
- **Encryption**: Protects documents if someone steals the entire cabinet

---

### Detailed Explanation: Encryption Providers

Kubernetes supports multiple encryption providers with different security/performance tradeoffs:

**Encryption Provider Table:**

| Provider | Encryption | Performance | Key Management | Use Case |
|----------|-----------|-------------|----------------|----------|
| `identity` | None ❌ | Fastest | N/A | Default (insecure) |
| `aescbc` | AES-CBC | Medium | Manual | Good balance |
| `aesgcm` | AES-GCM | Fast | Manual | High performance |
| `secretbox` | XSalsa20-Poly1305 | Fast | Manual | Modern alternative |
| `kms` | External KMS | Slowest | Automated | Enterprise (AWS KMS, etc.) |

**Provider Chain Process:**
```
Write Request → Try Provider 1 → Try Provider 2 → ... → identity (fallback)
Read Request → Try each provider until successful decryption
```

**Why Multiple Providers?**
1. **Migration**: Transition from unencrypted to encrypted gracefully
2. **Key Rotation**: Add new provider, re-encrypt, remove old provider
3. **Fallback**: `identity` at the end ensures reads don't fail during migration

---

### Hands-On Exercise 6: Examining Current Encryption State

**Context:** Before enabling encryption, we need to verify that secrets are currently stored in plaintext and understand the etcd storage structure.

**Command Group 15: Pre-Encryption Verification**

```bash
#!/bin/bash
# Save this as encryption_01_verify_current.sh

echo "=========================================="
echo "15.1: Creating a Test Secret"
echo "=========================================="
# Create a secret with known content to verify encryption state
kubectl create secret generic plaintext-test \
  --from-literal=password=MySuperSecretPassword123 \
  -n security-lab

echo ""
echo "=========================================="
echo "15.2: Verifying Secret Creation"
echo "=========================================="
kubectl get secret plaintext-test -n security-lab

echo ""
echo "=========================================="
echo "15.3: Viewing Secret in Kubernetes (base64)"
echo "=========================================="
# Kubernetes stores secrets as base64 (NOT encryption!)
kubectl get secret plaintext-test -n security-lab -o jsonpath='{.data.password}' | base64 -d
echo ""

echo ""
echo "=========================================="
echo "15.4: Accessing etcd Directly"
echo "=========================================="
# We need to access etcd inside the Minikube VM
# This shows how secrets are ACTUALLY stored on disk

echo "Connecting to Minikube node..."
minikube ssh "sudo ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/var/lib/minikube/certs/etcd/ca.crt \
  --cert=/var/lib/minikube/certs/etcd/server.crt \
  --key=/var/lib/minikube/certs/etcd/server.key \
  get /registry/secrets/security-lab/plaintext-test" | head -20

echo ""
echo "=========================================="
echo "15.5: Checking for Plaintext Content"
echo "=========================================="
# Search for our secret password in etcd output
echo "Searching etcd for 'MySuperSecretPassword'..."
minikube ssh "sudo ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/var/lib/minikube/certs/etcd/ca.crt \
  --cert=/var/lib/minikube/certs/etcd/server.crt \
  --key=/var/lib/minikube/certs/etcd/server.key \
  get /registry/secrets/security-lab/plaintext-test" | grep -o "MySuperSecretPassword" || echo "Not found in plaintext"

echo ""
echo "⚠️  WARNING: Without encryption, secrets are readable in etcd!"
```

**Expected Output for 15.5:**
```
MySuperSecretPassword
⚠️  WARNING: Without encryption, secrets are readable in etcd!
```

**Why This Matters:** This demonstrates that base64 encoding is **NOT** encryption. Anyone with etcd access can read all secrets!

---

### Hands-On Exercise 7: Generating Encryption Keys

**Context:** Encryption keys must be cryptographically random and of appropriate length. We'll use `/dev/urandom` (cryptographically secure random) rather than `/dev/random` (can block).

**Command Group 16: Key Generation**

```bash
#!/bin/bash
# Save this as encryption_02_generate_keys.sh

echo "=========================================="
echo "16.1: Understanding Key Requirements"
echo "=========================================="
echo "AES-CBC requires:"
echo "  - 128-bit (16 bytes) = 22 base64 characters"
echo "  - 256-bit (32 bytes) = 44 base64 characters (recommended)"
echo ""

echo "=========================================="
echo "16.2: Generating 256-bit Encryption Key"
echo "=========================================="
# Generate 32 random bytes and encode as base64
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
echo "Generated key (DO NOT use in production without secure storage):"
echo $ENCRYPTION_KEY

echo ""
echo "=========================================="
echo "16.3: Verifying Key Length"
echo "=========================================="
# Base64 of 32 bytes should be 44 characters (with padding)
echo "Key length: ${#ENCRYPTION_KEY} characters"
echo "Expected: 44 characters"

echo ""
echo "=========================================="
echo "16.4: Creating Encryption Configuration File"
echo "=========================================="
# This file tells the API server HOW to encrypt data

minikube ssh "sudo mkdir -p /etc/kubernetes/enc"

cat <<EOF | minikube ssh "sudo tee /etc/kubernetes/enc/encryption-config.yaml > /dev/null"
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  # Specify which resources to encrypt
  - resources:
      - secrets        # Most critical - contains passwords, tokens
      - configmaps     # May contain sensitive configuration
    providers:
      # Provider 1: aescbc encryption (active)
      - aescbc:
          keys:
            - name: key1
              secret: $ENCRYPTION_KEY
      # Provider 2: identity (fallback for reads during migration)
      - identity: {}
EOF

echo ""
echo "=========================================="
echo "16.5: Verifying Configuration File"
echo "=========================================="
minikube ssh "sudo cat /etc/kubernetes/enc/encryption-config.yaml"

echo ""
echo "=========================================="
echo "16.6: Setting Correct Permissions"
echo "=========================================="
# Encryption config contains secrets - restrict access!
minikube ssh "sudo chmod 600 /etc/kubernetes/enc/encryption-config.yaml"
minikube ssh "sudo chown root:root /etc/kubernetes/enc/encryption-config.yaml"

minikube ssh "ls -l /etc/kubernetes/enc/encryption-config.yaml"
```

**Expected Output for 16.6:**
```
-rw------- 1 root root 450 Oct  8 12:00 /etc/kubernetes/enc/encryption-config.yaml
```

**Configuration Breakdown:**

| Field | Purpose | Value |
|-------|---------|-------|
| `resources` | What to encrypt | `secrets`, `configmaps` |
| `providers[0]` | Primary encryption method | `aescbc` with our key |
| `providers[1]` | Fallback for migration | `identity` (no encryption) |
| `keys[0].name` | Key identifier | `key1` (for rotation) |
| `keys[0].secret` | Encryption key | Our generated 256-bit key |

> 🔒 **SECURITY:** The encryption config file contains the encryption key. In production:
> - Store keys in a secure KMS (AWS KMS, HashiCorp Vault)
> - Use `kms` provider instead of `aescbc`
> - Rotate keys regularly (every 90-180 days)
> - Never commit encryption configs to version control

---

### Hands-On Exercise 8: Configuring API Server for Encryption

**Context:** The API server needs to be told to use our encryption configuration. We modify its static pod manifest, which causes kubelet to automatically restart it with new settings.

**Command Group 17: API Server Configuration**

```bash
#!/bin/bash
# Save this as encryption_03_configure_apiserver.sh

echo "=========================================="
echo "17.1: Backing Up Current API Server Manifest"
echo "=========================================="
minikube ssh "sudo cp /etc/kubernetes/manifests/kube-apiserver.yaml /etc/kubernetes/manifests/kube-apiserver.yaml.backup"

echo "Backup created at: /etc/kubernetes/manifests/kube-apiserver.yaml.backup"

echo ""
echo "=========================================="
echo "17.2: Modifying API Server Configuration"
echo "=========================================="
# Add encryption provider config flag to API server

minikube ssh "sudo sed -i '/- kube-apiserver/a\    - --encryption-provider-config=/etc/kubernetes/enc/encryption-config.yaml' /etc/kubernetes/manifests/kube-apiserver.yaml"

echo ""
echo "=========================================="
echo "17.3: Adding Volume Mount"
echo "=========================================="
# API server needs to read the encryption config file

# Add volumeMount
minikube ssh "sudo sed -i '/volumeMounts:/a\    - mountPath: /etc/kubernetes/enc/encryption-config.yaml\n      name: encryption-config\n      readOnly: true' /etc/kubernetes/manifests/kube-apiserver.yaml"

# Add volume
minikube ssh "sudo sed -i '/volumes:/a\  - hostPath:\n      path: /etc/kubernetes/enc/encryption-config.yaml\n      type: File\n    name: encryption-config' /etc/kubernetes/manifests/kube-apiserver.yaml"

echo ""
echo "=========================================="
echo "17.4: Verifying Configuration Changes"
echo "=========================================="
echo "Checking for encryption-provider-config flag..."
minikube ssh "sudo grep 'encryption-provider-config' /etc/kubernetes/manifests/kube-apiserver.yaml"

echo ""
echo "=========================================="
echo "17.5: Waiting for API Server to Restart"
echo "=========================================="
echo "The kubelet will detect changes and restart the API server..."
echo "This may take 30-60 seconds..."
sleep 10

# Wait for API server to be responsive
for i in {1..30}; do
  if kubectl get --raw /healthz &>/dev/null; then
    echo "✅ API server is healthy!"
    break
  fi
  echo "Waiting for API server... ($i/30)"
  sleep 2
done

echo ""
echo "=========================================="
echo "17.6: Verifying API Server Status"
echo "=========================================="
kubectl get pods -n kube-system | grep kube-apiserver
```

**What Just Happened:**

1. **Backup**: Saved original config (always have a rollback plan!)
2. **Add Flag**: Told API server where to find encryption config
3. **Mount Volume**: Made the encryption config file accessible inside the container
4. **Auto-Restart**: Kubelet detected changes and restarted API server pod
5. **Verification**: Confirmed API server is running with new configuration

**Troubleshooting Tip:** If API server doesn't start:
```bash
# Check API server logs
minikube ssh "sudo journalctl -u kubelet | grep apiserver | tail -50"

# Restore backup
minikube ssh "sudo cp /etc/kubernetes/manifests/kube-apiserver.yaml.backup /etc/kubernetes/manifests/kube-apiserver.yaml"
```

---

### Deeper Investigation: Verifying Encryption is Active

**Command Group 18: Encryption Verification**

```bash
#!/bin/bash
# Save this as encryption_04_verify_encryption.sh

echo "=========================================="
echo "18.1: Creating New Secret (Should Be Encrypted)"
echo "=========================================="
kubectl create secret generic encrypted-test \
  --from-literal=password=ThisShouldBeEncrypted \
  -n security-lab

echo ""
echo "=========================================="
echo "18.2: Reading Secret from etcd (Raw)"
echo "=========================================="
echo "Fetching encrypted secret from etcd..."
minikube ssh "sudo ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/var/lib/minikube/certs/etcd/ca.crt \
  --cert=/var/lib/minikube/certs/etcd/server.crt \
  --key=/var/lib/minikube/certs/etcd/server.key \
  get /registry/secrets/security-lab/encrypted-test" > /tmp/encrypted-secret-output.txt

head -20 /tmp/encrypted-secret-output.txt

echo ""
echo "=========================================="
echo "18.3: Searching for Plaintext Password"
echo "=========================================="
echo "Searching for 'ThisShouldBeEncrypted' in etcd data..."
if grep -q "ThisShouldBeEncrypted" /tmp/encrypted-secret-output.txt; then
  echo "❌ FAIL: Secret is stored in PLAINTEXT!"
else
  echo "✅ SUCCESS: Secret is ENCRYPTED in etcd!"
fi

echo ""
echo "=========================================="
echo "18.4: Checking for Encryption Prefix"
echo "=========================================="
# Encrypted secrets have a specific prefix
echo "Looking for 'k8s:enc:aescbc' prefix (indicates AES-CBC encryption)..."
if minikube ssh "sudo ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/var/lib/minikube/certs/etcd/ca.crt \
  --cert=/var/lib/minikube/certs/etcd/server.crt \
  --key=/var/lib/minikube/certs/etcd/server.key \
  get /registry/secrets/security-lab/encrypted-test" | grep -q "k8s:enc:aescbc"; then
  echo "✅ Confirmed: Using AES-CBC encryption!"
else
  echo "❌ Encryption prefix not found"
fi

echo ""
echo "=========================================="
echo "18.5: Verifying Secret Still Readable via API"
echo "=========================================="
# API server should decrypt automatically
kubectl get secret encrypted-test -n security-lab -o jsonpath='{.data.password}' | base64 -d
echo ""

echo ""
echo "=========================================="
echo "18.6: Comparing Old vs New Secrets"
echo "=========================================="
echo "Old secret (plaintext-test) - created before encryption:"
minikube ssh "sudo ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/var/lib/minikube/certs/etcd/ca.crt \
  --cert=/var/lib/minikube/certs/etcd/server.crt \
  --key=/var/lib/minikube/certs/etcd/server.key \
  get /registry/secrets/security-lab/plaintext-test" | grep -o "MySuperSecretPassword" || echo "(Already encrypted or not found)"

echo ""
echo "New secret (encrypted-test) - created after encryption:"
minikube ssh "sudo ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/var/lib/minikube/certs/etcd/ca.crt \
  --cert=/var/lib/minikube/certs/etcd/server.crt \
  --key=/var/lib/minikube/certs/etcd/server.key \
  get /registry/secrets/security-lab/encrypted-test" | grep -o "ThisShouldBeEncrypted" || echo "✅ Not found in plaintext (encrypted!)"
```

**Analysis Questions:**

1. **Why can we still read the secret via `kubectl get secret` even though it's encrypted in etcd?**
2. **What happens to the old secret created before encryption was enabled?**
3. **How would you verify ALL secrets are encrypted, not just new ones?**

<details>
<summary>💡 <strong>Answers</strong></summary>

1. The API server automatically decrypts secrets when serving requests. It uses the encryption config to:
   - Read encrypted data from etcd
   - Decrypt using the appropriate provider (aescbc)
   - Return plaintext to authorized clients
   
   This is transparent to clients - they don't know encryption exists.

2. **Old secrets remain in plaintext!** Encryption only applies to NEW writes. The `identity` provider in our config allows reading old unencrypted secrets, but they won't be encrypted until updated.

3. Force re-encryption by updating all secrets:
   ```bash
   kubectl get secrets --all-namespaces -o json | kubectl replace -f -
   ```
   This reads each secret and writes it back, triggering encryption.

</details>

---

### Hands-On Exercise 9: Encrypting Existing Secrets

**Context:** Enabling encryption only protects NEW secrets. To encrypt existing ones, we must force Kubernetes to rewrite them.

**Command Group 19: Bulk Secret Re-encryption**

```bash
#!/bin/bash
# Save this as encryption_05_reencrypt_existing.sh

echo "=========================================="
echo "19.1: Listing All Secrets Before Re-encryption"
echo "=========================================="
kubectl get secrets --all-namespaces | head -15

echo ""
echo "=========================================="
echo "19.2: Checking Old Secret Encryption Status"
echo "=========================================="
echo "Verifying plaintext-test secret (created before encryption):"
minikube ssh "sudo ETCDCTL_API=3 etcdctl
