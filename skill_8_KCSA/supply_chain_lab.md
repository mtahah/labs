# 🔐 Supply Chain Security Lab - Complete Command Groups

## Quick Navigation
- [Phase 1: Environment Setup](#phase-1-environment-setup)
- [Phase 2: Build Application](#phase-2-build-application)
- [Phase 3: Vulnerability Scanning](#phase-3-vulnerability-scanning)
- [Phase 4: Image Signing](#phase-4-image-signing)
- [Phase 5: Secure Pipeline](#phase-5-secure-pipeline)
- [Phase 6: Kubernetes Deployment](#phase-6-kubernetes-deployment)
- [Phase 7: Cleanup](#phase-7-cleanup)

---

## Phase 1: Environment Setup

### Group 1.1: System Requirements Check

```bash
#!/bin/bash
echo "=== CHECKING SYSTEM REQUIREMENTS ==="

echo -e "\n[1/4] System Information"
cat /etc/os-release | grep -E "^(NAME|VERSION)="

echo -e "\n[2/4] Memory Check"
free -h | grep -E "^(Mem|Swap)"

echo -e "\n[3/4] Disk Space Check"
df -h / | tail -n 1

echo -e "\n[4/4] CPU Cores"
nproc

echo -e "\n✅ Requirements check complete"
```

### Group 1.2: Install Base Dependencies

```bash
#!/bin/bash
echo "=== INSTALLING BASE DEPENDENCIES ==="

echo -e "\n[1/3] Updating package manager..."
sudo apt-get update -qq

echo -e "\n[2/3] Installing essential tools..."
sudo apt-get install -y wget apt-transport-https gnupg lsb-release jq curl git

echo -e "\n[3/3] Verifying installations..."
for tool in wget curl jq git; do
  if command -v $tool &> /dev/null; then
    echo "✅ $tool installed"
  else
    echo "❌ $tool NOT FOUND"
  fi
done
```

### Group 1.3: Verify Docker

```bash
#!/bin/bash
echo "=== VERIFYING DOCKER INSTALLATION ==="

echo -e "\n[1/3] Checking Docker command..."
if command -v docker &> /dev/null; then
  echo "✅ Docker found"
  docker --version
else
  echo "❌ Docker not found - Install with:"
  echo "curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh"
  exit 1
fi

echo -e "\n[2/3] Checking Docker daemon..."
if sudo docker info &> /dev/null; then
  echo "✅ Docker daemon running"
  sudo docker info | grep -E "Server Version|Total Memory"
else
  echo "❌ Docker daemon not running"
  echo "Start with: sudo systemctl start docker"
  exit 1
fi

echo -e "\n[3/3] Testing Docker functionality..."
sudo docker ps &> /dev/null && echo "✅ Docker fully operational"
```

### Group 1.4: Verify Kubernetes

```bash
#!/bin/bash
echo "=== VERIFYING KUBERNETES CLUSTER ==="

echo -e "\n[1/3] Checking kubectl..."
if command -v kubectl &> /dev/null; then
  echo "✅ kubectl found"
  kubectl version --client --short 2>/dev/null || kubectl version --client
else
  echo "❌ kubectl not found"
  echo "Install kubectl: https://kubernetes.io/docs/tasks/tools/"
  exit 1
fi

echo -e "\n[2/3] Checking cluster connectivity..."
if kubectl cluster-info &> /dev/null; then
  echo "✅ Cluster accessible"
  kubectl cluster-info
else
  echo "❌ Cannot connect to cluster"
  echo "Start minikube: minikube start"
  exit 1
fi

echo -e "\n[3/3] Checking nodes..."
kubectl get nodes
```

### Group 1.5: Install Trivy

```bash
#!/bin/bash
echo "=== INSTALLING TRIVY VULNERABILITY SCANNER ==="

echo -e "\n[1/4] Adding Trivy GPG key..."
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | \
  sudo gpg --dearmor -o /usr/share/keyrings/trivy-archive-keyring.gpg

echo -e "\n[2/4] Adding Trivy repository..."
echo "deb [signed-by=/usr/share/keyrings/trivy-archive-keyring.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | \
  sudo tee /etc/apt/sources.list.d/trivy.list

echo -e "\n[3/4] Installing Trivy..."
sudo apt-get update -qq
sudo apt-get install -y trivy

echo -e "\n[4/4] Verifying installation..."
trivy --version
trivy image --download-db-only
echo "✅ Trivy installed and database downloaded"
```

### Group 1.6: Install Cosign

```bash
#!/bin/bash
echo "=== INSTALLING COSIGN IMAGE SIGNING TOOL ==="

echo -e "\n[1/3] Downloading Cosign..."
COSIGN_VERSION="v3.0.2"
wget -q "https://github.com/sigstore/cosign/releases/download/${COSIGN_VERSION}/cosign-linux-amd64" \
  -O /tmp/cosign-linux-amd64

echo -e "\n[2/3] Installing Cosign..."
sudo install -o root -g root -m 0755 /tmp/cosign-linux-amd64 /usr/local/bin/cosign
rm /tmp/cosign-linux-amd64

echo -e "\n[3/3] Verifying installation..."
cosign version
echo "✅ Cosign installed successfully"
```

### Group 1.7: Install Go and Crane

```bash
#!/bin/bash
echo "=== INSTALLING GO AND CRANE ==="

echo -e "\n[1/5] Downloading Go..."
GO_VERSION="1.22.0"
wget -q "https://golang.org/dl/go${GO_VERSION}.linux-amd64.tar.gz" -O /tmp/go.tar.gz

echo -e "\n[2/5] Installing Go..."
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf /tmp/go.tar.gz
rm /tmp/go.tar.gz

echo -e "\n[3/5] Configuring Go environment..."
export PATH=$PATH:/usr/local/go/bin
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

if ! grep -q "/usr/local/go/bin" ~/.bashrc 2>/dev/null; then
  echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
  echo 'export GOPATH=$HOME/go' >> ~/.bashrc
  echo 'export PATH=$PATH:$GOPATH/bin' >> ~/.bashrc
fi

echo -e "\n[4/5] Installing crane..."
go install github.com/google/go-containerregistry/cmd/crane@latest
sudo cp ~/go/bin/crane /usr/local/bin/ 2>/dev/null || true

echo -e "\n[5/5] Verifying installations..."
go version
crane version
echo "✅ Go and crane installed successfully"
```

### Group 1.8: Create Lab Directory Structure

```bash
#!/bin/bash
echo "=== CREATING LAB DIRECTORY STRUCTURE ==="

cd ~
mkdir -p supply-chain-lab
cd supply-chain-lab

echo -e "\n[1/2] Creating subdirectories..."
mkdir -p app/src pipeline policies keys manifests reports
mkdir -p config/{dev,staging,prod}

echo -e "\n[2/2] Verifying structure..."
ls -R
echo "✅ Lab directory structure created at: $(pwd)"
```

### Group 1.9: Complete Environment Test

```bash
#!/bin/bash
echo "=== COMPLETE ENVIRONMENT VERIFICATION ==="

check_tool() {
  if command -v $1 &> /dev/null; then
    echo "✅ $1"
    return 0
  else
    echo "❌ $1 - NOT FOUND"
    return 1
  fi
}

echo -e "\n[1/4] Checking installed tools..."
check_tool docker
check_tool kubectl
check_tool trivy
check_tool cosign
check_tool crane
check_tool jq
check_tool go

echo -e "\n[2/4] Checking Docker status..."
if sudo docker info &> /dev/null; then
  echo "✅ Docker daemon running"
else
  echo "❌ Docker daemon not running"
fi

echo -e "\n[3/4] Checking Kubernetes status..."
if kubectl cluster-info &> /dev/null; then
  echo "✅ Cluster accessible"
  kubectl get nodes --no-headers | wc -l | xargs echo "Nodes:"
else
  echo "❌ Cannot connect to cluster"
fi

echo -e "\n[4/4] Checking lab directory..."
if [ -d ~/supply-chain-lab ]; then
  echo "✅ Lab directory exists: ~/supply-chain-lab"
else
  echo "❌ Lab directory not found"
fi

echo -e "\n🎯 Environment verification complete!"
```

---

## Phase 2: Build Application

### Group 2.1: Create Application Source Code

```bash
#!/bin/bash
echo "=== CREATING APPLICATION SOURCE CODE ==="

cd ~/supply-chain-lab/app/src

echo -e "\n[1/3] Creating app.js..."
cat > app.js << 'EOF'
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.json({
    message: 'Supply Chain Security Lab Application',
    version: '1.0.0',
    timestamp: new Date().toISOString(),
    environment: process.env.NODE_ENV || 'development'
  });
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy', uptime: process.uptime() });
});

app.get('/ready', (req, res) => {
  res.json({ 
    status: 'ready',
    checks: {
      memory: process.memoryUsage().heapUsed < 100000000,
      uptime: process.uptime() > 5
    }
  });
});

app.listen(port, '0.0.0.0', () => {
  console.log(`🚀 App listening at http://0.0.0.0:${port}`);
});

process.on('SIGTERM', () => {
  console.log('SIGTERM received: closing');
  process.exit(0);
});
EOF

echo -e "\n[2/3] Creating package.json..."
cat > package.json << 'EOF'
{
  "name": "supply-chain-app",
  "version": "1.0.0",
  "description": "Sample app for supply chain security lab",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "engines": {
    "node": ">=18.0.0"
  }
}
EOF

echo -e "\n[3/3] Creating Dockerfile..."
cat > Dockerfile << 'EOF'
FROM node:18-alpine

WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install --only=production

COPY app.js .

RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
RUN chown -R nodejs:nodejs /usr/src/app
USER nodejs

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (res) => { process.exit(res.statusCode === 200 ? 0 : 1) })"

CMD ["npm", "start"]
EOF

ls -lh app.js package.json Dockerfile
echo "✅ Application files created"
```

### Group 2.2: Build Container Image

```bash
#!/bin/bash
echo "=== BUILDING CONTAINER IMAGE ==="

cd ~/supply-chain-lab/app/src

echo -e "\n[1/2] Building image..."
docker build -t supply-chain-app:v1.0.0 .

echo -e "\n[2/2] Verifying image..."
docker images supply-chain-app:v1.0.0
docker inspect supply-chain-app:v1.0.0 | jq '.[0] | {
  Created: .Created,
  User: .Config.User,
  ExposedPorts: .Config.ExposedPorts
}'

echo "✅ Image built successfully"
```

### Group 2.3: Test Application

```bash
#!/bin/bash
echo "=== TESTING APPLICATION ==="

echo -e "\n[1/5] Starting container..."
docker run -d --name test-app --rm -p 3000:3000 supply-chain-app:v1.0.0

echo -e "\n[2/5] Waiting for startup..."
sleep 5

echo -e "\n[3/5] Testing main endpoint..."
curl -s http://localhost:3000 | jq .

echo -e "\n[4/5] Testing health endpoint..."
curl -s http://localhost:3000/health | jq .

echo -e "\n[5/5] Cleaning up..."
docker stop test-app

echo "✅ Application test complete"
```

---

## Phase 3: Vulnerability Scanning

### Group 3.1: Basic Trivy Scan

```bash
#!/bin/bash
echo "=== BASIC VULNERABILITY SCANNING ==="

cd ~/supply-chain-lab

echo -e "\n[1/3] Updating vulnerability database..."
trivy image --download-db-only

echo -e "\n[2/3] Running basic scan..."
trivy image supply-chain-app:v1.0.0

echo -e "\n[3/3] Running HIGH/CRITICAL only scan..."
trivy image --severity HIGH,CRITICAL supply-chain-app:v1.0.0

echo "✅ Basic scans complete"
```

### Group 3.2: Create Scan Reports

```bash
#!/bin/bash
echo "=== GENERATING SCAN REPORTS ==="

cd ~/supply-chain-lab
mkdir -p reports

echo -e "\n[1/3] Generating JSON report..."
trivy image --format json --output reports/scan-report.json supply-chain-app:v1.0.0

echo -e "\n[2/3] Analyzing results..."
VULN_COUNT=$(jq '[.Results[]?.Vulnerabilities // []] | add | length' reports/scan-report.json 2>/dev/null || echo "0")
echo "Total vulnerabilities found: $VULN_COUNT"

echo -e "\n[3/3] Vulnerability summary by severity..."
jq '[.Results[]?.Vulnerabilities // [] | .[]] | group_by(.Severity) | 
    map({Severity: .[0].Severity, Count: length})' reports/scan-report.json

echo "✅ Report generated: reports/scan-report.json"
```

### Group 3.3: Create Automated Scanning Script

```bash
#!/bin/bash
echo "=== CREATING AUTOMATED SCAN SCRIPT ==="

cd ~/supply-chain-lab/pipeline

cat > scan-image.sh << 'SCRIPT_END'
#!/bin/bash
set -euo pipefail

IMAGE_NAME="${1:-}"
SEVERITY="${2:-HIGH,CRITICAL}"
FORMAT="${3:-table}"

if [ -z "$IMAGE_NAME" ]; then
    echo "Usage: $0 <image_name> [severity] [format]"
    echo "Example: $0 myapp:latest HIGH,CRITICAL json"
    exit 1
fi

echo "===================================="
echo "🔍 VULNERABILITY SCANNING"
echo "===================================="
echo "Image: $IMAGE_NAME"
echo "Severity: $SEVERITY"
echo "Format: $FORMAT"
echo "===================================="

mkdir -p ../reports
REPORT_FILE="../reports/$(basename $IMAGE_NAME | tr ':' '-')-scan.json"

if [ "$FORMAT" = "json" ]; then
    trivy image --severity "$SEVERITY" --format json --output "$REPORT_FILE" "$IMAGE_NAME"
    
    if [ -f "$REPORT_FILE" ]; then
        VULN_COUNT=$(jq '[.Results[]?.Vulnerabilities // []] | add | length' "$REPORT_FILE" 2>/dev/null || echo "0")
        echo ""
        echo "Vulnerabilities Found: $VULN_COUNT"
        echo "Report: $REPORT_FILE"
        
        if [ "$VULN_COUNT" -gt 0 ]; then
            echo "❌ SCAN FAILED"
            exit 1
        else
            echo "✅ SCAN PASSED"
            exit 0
        fi
    fi
else
    trivy image --severity "$SEVERITY" "$IMAGE_NAME"
fi
SCRIPT_END

chmod +x scan-image.sh
echo "✅ Script created: scan-image.sh"

echo -e "\n=== Testing script ==="
./scan-image.sh supply-chain-app:v1.0.0 HIGH,CRITICAL json || echo "⚠️ Vulnerabilities found"
```

### Group 3.4: Create Comprehensive Scan Script

```bash
#!/bin/bash
echo "=== CREATING COMPREHENSIVE SCAN SCRIPT ==="

cd ~/supply-chain-lab/pipeline

cat > comprehensive-scan.sh << 'SCRIPT_END'
#!/bin/bash
set -euo pipefail

IMAGE_NAME="${1:-}"
if [ -z "$IMAGE_NAME" ]; then
    echo "Usage: $0 <image_name>"
    exit 1
fi

echo "==========================================="
echo "🔍 COMPREHENSIVE SECURITY SCAN"
echo "==========================================="
echo "Image: $IMAGE_NAME"
echo "==========================================="

mkdir -p ../reports
SCAN_FAILED=0

echo -e "\n📋 [1/3] Vulnerability scan..."
if trivy image --severity HIGH,CRITICAL --format json \
    --output "../reports/vuln-$(basename $IMAGE_NAME | tr ':' '-').json" "$IMAGE_NAME"; then
    echo "✅ Vulnerability scan complete"
else
    echo "❌ Vulnerabilities found"
    SCAN_FAILED=1
fi

echo -e "\n🔐 [2/3] Secret scan..."
if trivy image --scanners secret --format json \
    --output "../reports/secrets-$(basename $IMAGE_NAME | tr ':' '-').json" "$IMAGE_NAME"; then
    echo "✅ Secret scan complete"
else
    echo "⚠️ Secrets detected"
    SCAN_FAILED=1
fi

echo -e "\n⚙️  [3/3] Configuration scan..."
if trivy image --scanners config --format json \
    --output "../reports/config-$(basename $IMAGE_NAME | tr ':' '-').json" "$IMAGE_NAME"; then
    echo "✅ Configuration scan complete"
else
    echo "⚠️ Configuration issues found"
    SCAN_FAILED=1
fi

SUMMARY_FILE="../reports/summary-$(basename $IMAGE_NAME | tr ':' '-').txt"
cat > "$SUMMARY_FILE" << EOL
COMPREHENSIVE SECURITY SCAN SUMMARY
===================================
Image: $IMAGE_NAME
Date: $(date)
Status: $([ $SCAN_FAILED -eq 0 ] && echo "✅ PASSED" || echo "❌ FAILED")
===================================
EOL

cat "$SUMMARY_FILE"

exit $SCAN_FAILED
SCRIPT_END

chmod +x comprehensive-scan.sh
echo "✅ Script created: comprehensive-scan.sh"

echo -e "\n=== Testing comprehensive scan ==="
./comprehensive-scan.sh supply-chain-app:v1.0.0 || echo "⚠️ Issues found"
```

---

## Phase 4: Image Signing

### Group 4.1: Generate Signing Keys

```bash
#!/bin/bash
echo "=== GENERATING COSIGN SIGNING KEYS ==="

cd ~/supply-chain-lab/keys

echo -e "\n[1/2] Generating key pair..."
echo "⚠️  Use password: lab-password-123"
echo ""

# Generate keys (will prompt for password)
cosign generate-key-pair

echo -e "\n[2/2] Verifying keys..."
ls -lh cosign.key cosign.pub

echo -e "\n=== Public Key Content ==="
cat cosign.pub

echo "✅ Signing keys generated"
```

### Group 4.2: Create Key Management Script

```bash
#!/bin/bash
echo "=== CREATING KEY MANAGEMENT SCRIPT ==="

cd ~/supply-chain-lab/pipeline

cat > manage-keys.sh << 'SCRIPT_END'
#!/bin/bash
set -euo pipefail

KEYS_DIR="../keys"

case "${1:-}" in
    "verify-keys")
        echo "🔍 Verifying key pair..."
        if [ -f "$KEYS_DIR/cosign.key" ] && [ -f "$KEYS_DIR/cosign.pub" ]; then
            echo "✅ Both keys found"
            echo "Private Key: $KEYS_DIR/cosign.key"
            echo "Public Key: $KEYS_DIR/cosign.pub"
            ls -lh "$KEYS_DIR"/cosign.*
        else
            echo "❌ Keys missing"
            exit 1
        fi
        ;;
    
    "backup-keys")
        echo "💾 Creating key backup..."
        BACKUP_DIR="$KEYS_DIR/backup-$(date +%Y%m%d-%H%M%S)"
        mkdir -p "$BACKUP_DIR"
        cp "$KEYS_DIR/cosign.key" "$KEYS_DIR/cosign.pub" "$BACKUP_DIR/"
        chmod 400 "$BACKUP_DIR/cosign.key"
        echo "✅ Keys backed up to: $BACKUP_DIR"
        ls -lh "$BACKUP_DIR"
        ;;
    
    *)
        echo "Usage: $0 {verify-keys|backup-keys}"
        exit 1
        ;;
esac
SCRIPT_END

chmod +x manage-keys.sh
echo "✅ Script created: manage-keys.sh"

echo -e "\n=== Testing key verification ==="
./manage-keys.sh verify-keys
```

### Group 4.3: Sign Container Image

```bash
#!/bin/bash
echo "=== SIGNING CONTAINER IMAGE ==="

cd ~/supply-chain-lab/pipeline

cat > sign-image.sh << 'SCRIPT_END'
#!/bin/bash
set -euo pipefail

IMAGE_NAME="${1:-}"
PRIVATE_KEY="../keys/cosign.key"

if [ -z "$IMAGE_NAME" ]; then
    echo "Usage: $0 <image_name>"
    exit 1
fi

if [ ! -f "$PRIVATE_KEY" ]; then
    echo "❌ Private key not found: $PRIVATE_KEY"
    exit 1
fi

echo "✍️  Signing image: $IMAGE_NAME"
echo "Key: $PRIVATE_KEY"
echo ""

# Sign the image (will prompt for password)
cosign sign --key "$PRIVATE_KEY" "$IMAGE_NAME"

echo ""
echo "✅ Image signed successfully"
SCRIPT_END

chmod +x sign-image.sh
echo "✅ Script created: sign-image.sh"

echo -e "\n=== Signing image (password: lab-password-123) ==="
./sign-image.sh supply-chain-app:v1.0.0
```

### Group 4.4: Verify Image Signature

```bash
#!/bin/bash
echo "=== CREATING SIGNATURE VERIFICATION SCRIPT ==="

cd ~/supply-chain-lab/pipeline

cat > verify-signature.sh << 'SCRIPT_END'
#!/bin/bash
set -euo pipefail

IMAGE_NAME="${1:-}"
PUBLIC_KEY="../keys/cosign.pub"

if [ -z "$IMAGE_NAME" ]; then
    echo "Usage: $0 <image_name>"
    exit 1
fi

if [ ! -f "$PUBLIC_KEY" ]; then
    echo "❌ Public key not found: $PUBLIC_KEY"
    exit 1
fi

echo "🔍 Verifying signature for: $IMAGE_NAME"
echo "Public Key: $PUBLIC_KEY"
echo ""

if cosign verify --key "$PUBLIC_KEY" "$IMAGE_NAME" > /dev/null 2>&1; then
    echo "✅ Signature verification PASSED"
    echo ""
    echo "Signature details:"
    cosign verify --key "$PUBLIC_KEY" "$IMAGE_NAME"
    exit 0
else
    echo "❌ Signature verification FAILED"
    exit 1
fi
SCRIPT_END

chmod +x verify-signature.sh
echo "✅ Script created: verify-signature.sh"

echo -e "\n=== Testing verification ==="
./verify-signature.sh supply-chain-app:v1.0.0
```

### Group 4.5: Sign with Metadata

```bash
#!/bin/bash
echo "=== CREATING ADVANCED SIGNING SCRIPT ==="

cd ~/supply-chain-lab/pipeline

cat > sign-with-metadata.sh << 'SCRIPT_END'
#!/bin/bash
set -euo pipefail

IMAGE_NAME="${1:-}"
BUILD_ID="${2:-$(date +%Y%m%d-%H%M%S)}"
GIT_COMMIT="${3:-unknown}"
PRIVATE_KEY="../keys/cosign.key"

if [ -z "$IMAGE_NAME" ]; then
    echo "Usage: $0 <image_name> [build_id] [git_commit]"
    exit 1
fi

echo "✍️  Signing image with metadata"
echo "Image: $IMAGE_NAME"
echo "Build ID: $BUILD_ID"
echo "Git Commit: $GIT_COMMIT"
echo ""

mkdir -p ../reports

cat > ../reports/build-metadata.json << EOL
{
  "buildId": "$BUILD_ID",
  "gitCommit": "$GIT_COMMIT",
  "buildTime": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "builder": "supply-chain-lab",
  "scanStatus": "passed"
}
EOL

# Sign with annotations (will prompt for password)
cosign sign --key "$PRIVATE_KEY" \
    -a "build-id=$BUILD_ID" \
    -a "git-commit=$GIT_COMMIT" \
    -a "scan-status=passed" \
    "$IMAGE_NAME"

# Attach attestation (will prompt for password again)
cosign attest --key "$PRIVATE_KEY" \
    --predicate ../reports/build-metadata.json \
    "$IMAGE_NAME"

echo ""
echo "✅ Image signed with metadata and attestation"
SCRIPT_END

chmod +x sign-with-metadata.sh
echo "✅ Script created: sign-with-metadata.sh"

echo -e "\n=== Signing with metadata (password: lab-password-123) ==="
./sign-with-metadata.sh supply-chain-app:v1.0.0 "build-001" "abc123"
```

---

## Phase 5: Secure Pipeline

### Group 5.1: Create Complete Secure Pipeline

```bash
#!/bin/bash
echo "=== CREATING SECURE CI/CD PIPELINE ==="

cd ~/supply-chain-lab/pipeline

cat > secure-pipeline.sh << 'SCRIPT_END'
#!/bin/bash
set -euo pipefail

IMAGE_NAME="${1:-supply-chain-app}"
IMAGE_TAG="${2:-v1.0.1}"
FULL_IMAGE="$IMAGE_NAME:$IMAGE_TAG"
BUILD_ID="$(date +%Y%m%d-%H%M%S)"

echo "=========================================="
echo "🚀 SECURE CI/CD PIPELINE"
echo "=========================================="
echo "Image: $FULL_IMAGE"
echo "Build ID: $BUILD_ID"
echo "=========================================="

# Step 1: Build
echo -e "\n📦 [1/4] Building image..."
cd ../app/src
if docker build -t "$FULL_IMAGE" . ; then
    echo "✅ Build successful"
else
    echo "❌ Build failed"
    exit 1
fi

# Step 2: Scan
echo -e "\n🔍 [2/4] Scanning for vulnerabilities..."
cd ../../pipeline
if ./comprehensive-scan.sh "$FULL_IMAGE"; then
    echo "✅ Security scan passed"
else
    echo "❌ Security scan failed"
    exit 1
fi

# Step 3: Sign
echo -e "\n✍️  [3/4] Signing image..."
if ./sign-with-metadata.sh "$FULL_IMAGE" "$BUILD_ID" "pipeline"; then
    echo "✅ Signing successful"
else
    echo "❌ Signing failed"
    exit 1
fi

# Step 4: Verify
echo -e "\n🔍 [4/4] Verifying signature..."
if ./verify-signature.sh "$FULL_IMAGE"; then
    echo "✅ Verification successful"
else
    echo "❌ Verification failed"
    exit 1
fi

# Generate report
REPORT_FILE="../reports/pipeline-$BUILD_ID.txt"
cat > "$REPORT_FILE" << EOL
SECURE CI/CD PIPELINE REPORT
============================
Build ID: $BUILD_ID
Image: $FULL_IMAGE
Date: $(date)
Status: ✅ SUCCESS

Pipeline Steps:
✅ Image Build
✅ Vulnerability Scan
✅ Image Signing
✅ Signature Verification

Next Steps:
- Deploy to staging
- Run integration tests
- Deploy to production (with approval)
EOL

cat "$REPORT_FILE"

echo ""
echo "✅ PIPELINE COMPLETED SUCCESSFULLY"
echo "📋 Report: $REPORT_FILE"
SCRIPT_END

chmod +x secure-pipeline.sh
echo "✅ Script created: secure-pipeline.sh"

echo -e "\n=== Running complete pipeline (password: lab-password-123 when prompted) ==="
./secure-pipeline.sh supply-chain-app v1.0.1
```

---

## Phase 6: Kubernetes Deployment

### Group 6.1: Create Kubernetes Manifests

```bash
#!/bin/bash
echo "=== CREATING KUBERNETES MANIFESTS ==="

cd ~/supply-chain-lab/manifests

echo -e "\n[1/3] Creating deployment manifest..."
cat > app-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: supply-chain-app
  namespace: default
  labels:
    app: supply-chain-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: supply-chain-app
  template:
    metadata:
      labels:
        app: supply-chain-app
      annotations:
        security.policy/image-signed: "true"
    spec:
      containers:
      - name: app
        image: supply-chain-app:v1.0.1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3000
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "100m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
        securityContext:
          allowPrivilegeEscalation: false
          runAsNonRoot: true
          runAsUser: 1001
          readOnlyRootFilesystem: false
          capabilities:
            drop:
            - ALL
---
apiVersion: v1
kind: Service
metadata:
  name: supply-chain-app
  namespace: default
spec:
  selector:
    app: supply-chain-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: ClusterIP
EOF

echo -e "\n[2/3] Creating policy validation script..."
cat > ../policies/validate-deployment.sh << 'EOF'
#!/bin/bash
set -euo pipefail

DEPLOYMENT_FILE="${1:-}"

if [ -z "$DEPLOYMENT_FILE" ]; then
    echo "Usage: $0 <deployment_file>"
    exit 1
fi

echo "🔍 Validating deployment: $DEPLOYMENT_FILE"

# Extract images
IMAGES=$(kubectl get -f "$DEPLOYMENT_FILE" -o jsonpath='{.spec.template.spec.containers[*].image}' --dry-run=client 2>/dev/null || echo "")

if [ -z "$IMAGES" ]; then
    echo "❌ No images found in deployment"
    exit 1
fi

echo "📋 Images to verify: $IMAGES"
echo ""

# Verify each image
for IMAGE in $IMAGES; do
    echo "🔍 Verifying: $IMAGE"
    
    if cosign verify --key ../keys/cosign.pub "$IMAGE" > /dev/null 2>&1; then
        echo "✅ $IMAGE: Signature valid"
    else
        echo "❌ $IMAGE: Signature invalid or missing"
        echo "🚫 Deployment blocked"
        exit 1
    fi
done

echo ""
echo "✅ All images verified - deployment allowed"
EOF

chmod +x ../policies/validate-deployment.sh

echo -e "\n[3/3] Creating namespace and RBAC..."
cat > namespace-rbac.yaml << 'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: cosign-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cosign-webhook
  namespace: cosign-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cosign-webhook
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cosign-webhook
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cosign-webhook
subjects:
- kind: ServiceAccount
  name: cosign-webhook
  namespace: cosign-system
EOF

ls -lh *.yaml
echo "✅ Kubernetes manifests created"
```

### Group 6.2: Validate and Deploy to Kubernetes

```bash
#!/bin/bash
echo "=== VALIDATING AND DEPLOYING TO KUBERNETES ==="

cd ~/supply-chain-lab

echo -e "\n[1/5] Validating deployment file..."
kubectl apply -f manifests/app-deployment.yaml --dry-run=client

echo -e "\n[2/5] Verifying image signatures..."
./policies/validate-deployment.sh manifests/app-deployment.yaml

echo -e "\n[3/5] Creating namespace and RBAC..."
kubectl apply -f manifests/namespace-rbac.yaml

echo -e "\n[4/5] Creating public key secret..."
kubectl create secret generic cosign-public-key \
  --from-file=cosign.pub=keys/cosign.pub \
  --namespace=cosign-system \
  --dry-run=client -o yaml | kubectl apply -f -

echo -e "\n[5/5] Deploying application..."
kubectl apply -f manifests/app-deployment.yaml

echo ""
echo "⏳ Waiting for deployment to be ready..."
kubectl wait --for=condition=available --timeout=60s deployment/supply-chain-app

echo ""
echo "=== Deployment Status ==="
kubectl get deployment supply-chain-app
kubectl get pods -l app=supply-chain-app

echo ""
echo "✅ Application deployed successfully"
```

### Group 6.3: Test Deployed Application

```bash
#!/bin/bash
echo "=== TESTING DEPLOYED APPLICATION ==="

echo -e "\n[1/4] Checking pod status..."
kubectl get pods -l app=supply-chain-app

echo -e "\n[2/4] Viewing pod logs..."
POD_NAME=$(kubectl get pods -l app=supply-chain-app -o jsonpath='{.items[0].metadata.name}')
echo "Pod: $POD_NAME"
kubectl logs $POD_NAME --tail=20

echo -e "\n[3/4] Testing application via port-forward..."
kubectl port-forward service/supply-chain-app 8080:80 &
PF_PID=$!
sleep 3

curl -s http://localhost:8080 | jq .
curl -s http://localhost:8080/health | jq .

kill $PF_PID 2>/dev/null || true

echo -e "\n[4/4] Checking deployment security context..."
kubectl get pod $POD_NAME -o jsonpath='{.spec.containers[0].securityContext}' | jq .

echo ""
echo "✅ Application testing complete"
```

### Group 6.4: Create Image Admission Policy

```bash
#!/bin/bash
echo "=== CREATING IMAGE ADMISSION POLICY ==="

cd ~/supply-chain-lab/policies

cat > image-policy.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: image-policy
  namespace: cosign-system
data:
  policy.yaml: |
    apiVersion: policy.sigstore.dev/v1beta1
    kind: ClusterImagePolicy
    metadata:
      name: signed-images-only
    spec:
      images:
      - glob: "supply-chain-app:*"
      authorities:
      - key:
          data: |
            -----BEGIN PUBLIC KEY-----
            # Public key content here
            -----END PUBLIC KEY-----
---
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: image-signature-webhook
webhooks:
- name: validate-signatures.example.com
  clientConfig:
    service:
      name: cosign-webhook
      namespace: cosign-system
      path: /validate
  rules:
  - operations: ["CREATE", "UPDATE"]
    apiGroups: [""]
    apiVersions: ["v1"]
    resources: ["pods"]
  admissionReviewVersions: ["v1"]
  sideEffects: None
  timeoutSeconds: 10
  failurePolicy: Fail
EOF

echo "✅ Policy manifest created: image-policy.yaml"
echo "Note: In production, use Sigstore Policy Controller or Kyverno"
```

---

## Phase 7: Cleanup

### Group 7.1: Pre-Cleanup Inventory

```bash
#!/bin/bash
echo "=== PRE-CLEANUP INVENTORY ==="

echo -e "\n[1/5] Docker images..."
docker images | grep -E "supply-chain|REPOSITORY"

echo -e "\n[2/5] Running containers..."
docker ps -a | grep -E "supply-chain|CONTAINER"

echo -e "\n[3/5] Kubernetes deployments..."
kubectl get deployments -n default | grep -E "supply-chain|NAME" || echo "None found"

echo -e "\n[4/5] Kubernetes namespaces..."
kubectl get namespace cosign-system 2>/dev/null || echo "cosign-system not found"

echo -e "\n[5/5] Lab directory..."
du -sh ~/supply-chain-lab 2>/dev/null || echo "Directory not found"

echo ""
echo "✅ Inventory complete"
```

### Group 7.2: Clean Kubernetes Resources

```bash
#!/bin/bash
echo "=== CLEANING KUBERNETES RESOURCES ==="

echo -e "\n[1/4] Deleting application deployment..."
kubectl delete -f ~/supply-chain-lab/manifests/app-deployment.yaml --ignore-not-found=true

echo -e "\n[2/4] Deleting service..."
kubectl delete service supply-chain-app --ignore-not-found=true

echo -e "\n[3/4] Deleting cosign-system namespace..."
kubectl delete namespace cosign-system --ignore-not-found=true --timeout=60s

echo -e "\n[4/4] Verifying cleanup..."
kubectl get deployments -n default | grep supply-chain || echo "✅ Deployment removed"
kubectl get namespace cosign-system 2>/dev/null || echo "✅ Namespace removed"

echo ""
echo "✅ Kubernetes resources cleaned"
```

### Group 7.3: Clean Docker Resources

```bash
#!/bin/bash
echo "=== CLEANING DOCKER RESOURCES ==="

echo -e "\n[1/4] Stopping containers..."
docker ps -a | grep supply-chain | awk '{print $1}' | xargs -r docker stop
docker ps -a | grep supply-chain | awk '{print $1}' | xargs -r docker rm

echo -e "\n[2/4] Removing images..."
docker images | grep supply-chain-app | awk '{print $3}' | xargs -r docker rmi -f

echo -e "\n[3/4] Removing dangling images..."
docker image prune -f

echo -e "\n[4/4] Verifying cleanup..."
docker images | grep supply-chain || echo "✅ Images removed"
docker ps -a | grep supply-chain || echo "✅ Containers removed"

echo ""
echo "✅ Docker resources cleaned"
```

### Group 7.4: Clean Lab Directory (Optional - Backup First!)

```bash
#!/bin/bash
echo "=== CLEANING LAB DIRECTORY ==="

echo -e "\n⚠️  WARNING: This will delete the lab directory!"
echo "Press Ctrl+C to cancel, or wait 5 seconds to continue..."
sleep 5

echo -e "\n[1/3] Creating backup..."
BACKUP_DIR=~/supply-chain-lab-backup-$(date +%Y%m%d-%H%M%S)
cp -r ~/supply-chain-lab "$BACKUP_DIR" 2>/dev/null || echo "No directory to backup"
echo "Backup: $BACKUP_DIR"

echo -e "\n[2/3] Removing lab directory..."
rm -rf ~/supply-chain-lab

echo -e "\n[3/3] Verifying removal..."
if [ ! -d ~/supply-chain-lab ]; then
    echo "✅ Lab directory removed"
else
    echo "❌ Directory still exists"
fi

echo ""
echo "✅ Cleanup complete"
echo "📦 Backup available at: $BACKUP_DIR"
```

### Group 7.5: Complete Cleanup Verification

```bash
#!/bin/bash
echo "=== COMPLETE CLEANUP VERIFICATION ==="

echo -e "\n[1/5] Checking Docker images..."
if docker images | grep -q supply-chain; then
    echo "⚠️  Docker images still exist"
    docker images | grep supply-chain
else
    echo "✅ No Docker images found"
fi

echo -e "\n[2/5] Checking Docker containers..."
if docker ps -a | grep -q supply-chain; then
    echo "⚠️  Containers still exist"
    docker ps -a | grep supply-chain
else
    echo "✅ No containers found"
fi

echo -e "\n[3/5] Checking Kubernetes deployments..."
if kubectl get deployment supply-chain-app 2>/dev/null; then
    echo "⚠️  Deployment still exists"
else
    echo "✅ No deployment found"
fi

echo -e "\n[4/5] Checking Kubernetes namespace..."
if kubectl get namespace cosign-system 2>/dev/null; then
    echo "⚠️  Namespace still exists"
else
    echo "✅ No namespace found"
fi

echo -e "\n[5/5] Checking lab directory..."
if [ -d ~/supply-chain-lab ]; then
    echo "⚠️  Lab directory still exists"
    du -sh ~/supply-chain-lab
else
    echo "✅ Lab directory removed"
fi

echo ""
echo "🎯 Cleanup verification complete!"
```

---

## 📚 Quick Reference Commands

### Essential Operations

```bash
# Check environment
cd ~/supply-chain-lab/pipeline && ./manage-keys.sh verify-keys

# Scan image
cd ~/supply-chain-lab/pipeline && ./scan-image.sh IMAGE:TAG HIGH,CRITICAL json

# Sign image (requires password: lab-password-123)
cd ~/supply-chain-lab/pipeline && ./sign-image.sh IMAGE:TAG

# Verify signature
cd ~/supply-chain-lab/pipeline && ./verify-signature.sh IMAGE:TAG

# Run full pipeline (requires password when prompted)
cd ~/supply-chain-lab/pipeline && ./secure-pipeline.sh myapp v1.0.0

# Validate deployment
cd ~/supply-chain-lab && ./policies/validate-deployment.sh manifests/app-deployment.yaml

# Deploy to Kubernetes
kubectl apply -f ~/supply-chain-lab/manifests/app-deployment.yaml

# Check deployment status
kubectl get pods -l app=supply-chain-app
kubectl logs -l app=supply-chain-app --tail=50

# Test application
kubectl port-forward service/supply-chain-app 8080:80
# In another terminal: curl http://localhost:8080
```

### Troubleshooting Commands

```bash
# View scan reports
ls -lh ~/supply-chain-lab/reports/
jq . ~/supply-chain-lab/reports/scan-report.json

# Check signature details
cosign verify --key ~/supply-chain-lab/keys/cosign.pub IMAGE:TAG

# Verify image hasn't been tampered
docker inspect IMAGE:TAG | jq '.[0].RepoDigests'

# Check Kubernetes pod security
POD=$(kubectl get pods -l app=supply-chain-app -o jsonpath='{.items[0].metadata.name}')
kubectl get pod $POD -o jsonpath='{.spec.containers[0].securityContext}' | jq .

# View Trivy database version
trivy --version
trivy image --download-db-only

# Test Docker connectivity
docker info
docker run hello-world

# Test Kubernetes connectivity
kubectl cluster-info
kubectl get nodes
kubectl get all -A
```

---

## 🎯 Lab Completion Checklist

### Phase 1: Environment Setup
- [ ] All tools installed (Docker, kubectl, Trivy, Cosign, Go, crane)
- [ ] Docker daemon running
- [ ] Kubernetes cluster accessible
- [ ] Lab directory structure created
- [ ] All verification tests passed

### Phase 2: Build Application
- [ ] Application source code created
- [ ] Dockerfile created with security best practices
- [ ] Container image built successfully
- [ ] Application tested locally

### Phase 3: Vulnerability Scanning
- [ ] Basic Trivy scans completed
- [ ] Scan automation scripts created
- [ ] Comprehensive scan script working
- [ ] JSON reports generated

### Phase 4: Image Signing
- [ ] Signing keys generated
- [ ] Key management script created
- [ ] Image signed with Cosign
- [ ] Signature verified successfully
- [ ] Metadata and attestations added

### Phase 5: Secure Pipeline
- [ ] Complete pipeline script created
- [ ] Pipeline executed successfully (build → scan → sign → verify)
- [ ] Pipeline report generated

### Phase 6: Kubernetes Deployment
- [ ] Kubernetes manifests created
- [ ] Namespace and RBAC configured
- [ ] Public key secret created
- [ ] Application deployed to cluster
- [ ] Deployment validated and tested

### Phase 7: Cleanup
- [ ] Pre-cleanup inventory completed
- [ ] Kubernetes resources removed
- [ ] Docker resources cleaned
- [ ] Lab directory backed up (optional)
- [ ] Cleanup verified

---

## 🚨 Common Issues and Solutions

### Issue: "permission denied" when running Docker commands
**Solution:**
```bash
sudo usermod -aG docker $USER
newgrp docker
# Or prefix commands with 'sudo'
```

### Issue: Trivy scan fails with "database download error"
**Solution:**
```bash
rm -rf ~/.cache/trivy
trivy image --download-db-only
# Check internet connectivity
```

### Issue: Cosign sign fails with "private key not found"
**Solution:**
```bash
cd ~/supply-chain-lab/pipeline
./manage-keys.sh verify-keys
# If keys missing, regenerate:
cd ~/supply-chain-lab/keys
cosign generate-key-pair
```

### Issue: kubectl cannot connect to cluster
**Solution:**
```bash
# For minikube:
minikube status
minikube start

# For kind:
kind get clusters
kind create cluster

# Check kubeconfig:
kubectl cluster-info
```

### Issue: Image signature verification fails
**Solution:**
```bash
# Verify image exists locally
docker images | grep supply-chain-app

# Check if image was actually signed
cosign verify --key ~/supply-chain-lab/keys/cosign.pub IMAGE:TAG

# Re-sign if needed
cd ~/supply-chain-lab/pipeline
./sign-image.sh IMAGE:TAG
```

### Issue: Pod fails to start in Kubernetes
**Solution:**
```bash
# Check pod status
kubectl get pods -l app=supply-chain-app
kubectl describe pod POD_NAME

# Check logs
kubectl logs POD_NAME

# Common causes:
# 1. Image not available (imagePullPolicy: IfNotPresent requires local image)
# 2. Resource limits too low
# 3. Health check failing
```

---

## 📖 Additional Learning Resources

### Official Documentation
- **Trivy:** https://trivy.dev/docs/
- **Cosign:** https://docs.sigstore.dev/cosign/overview/
- **Kubernetes Security:** https://kubernetes.io/docs/concepts/security/

### Best Practices
- **SLSA Framework:** https://slsa.dev/
- **Supply Chain Levels for Software Artifacts**
- **Container Security:** https://www.cisa.gov/uscert/ncas/current-activity/2021/03/26/container-security-guidance
- **Image Signing Standards:** https://github.com/sigstore/cosign/blob/main/USAGE.md

### Advanced Topics
- **Sigstore Policy Controller:** https://docs.sigstore.dev/policy-controller/overview/
- **Kyverno Image Verification:** https://kyverno.io/docs/writing-policies/verify-images/
- **OPA/Gatekeeper:** https://open-policy-agent.github.io/gatekeeper/

---

## 🎓 What You've Learned

By completing this lab, you've mastered:

1. **Vulnerability Management**
   - Scanning container images with Trivy
   - Interpreting CVE severity levels
   - Creating automated scanning pipelines
   - Generating machine-readable security reports

2. **Image Signing and Verification**
   - Cryptographic key pair generation
   - Signing images with Cosign
   - Verifying signatures before deployment
   - Adding metadata and attestations

3. **Secure CI/CD Integration**
   - Building security into pipelines
   - Implementing security gates
   - Automating scan-sign-verify workflows
   - Generating compliance reports

4. **Kubernetes Security**
   - Deploying signed images
   - Implementing admission controls
   - Enforcing image verification policies
   - Secure pod configurations

5. **Production Best Practices**
   - Non-root container users
   - Minimal base images (Alpine)
   - Resource limits and health checks
   - Key management and rotation

---

## 🚀 Next Steps

### Immediate Actions
1. **Practice key rotation:** Generate new keys and re-sign all images
2. **Test tampered images:** Modify an image and try to deploy it
3. **Experiment with policies:** Create different severity thresholds for dev/staging/prod
4. **Explore Trivy features:** Try filesystem scanning, SBOM generation

### Advanced Challenges
1. **Implement Sigstore Policy Controller** in your Kubernetes cluster
2. **Create a GitLab/GitHub Actions pipeline** with these security checks
3. **Set up Kyverno** for policy-as-code enforcement
4. **Generate and sign SBOMs** (Software Bill of Materials)
5. **Integrate with vulnerability management dashboards** (DefectDojo, Dependency-Track)

### Production Readiness
1. **Key Management:** Move to HashiCorp Vault or cloud KMS
2. **Monitoring:** Set up alerting for failed scans or signature verifications
3. **Automation:** Implement automatic remediation for known CVEs
4. **Compliance:** Map your pipeline to frameworks (SLSA, NIST)
5. **Documentation:** Create runbooks for security incident response

---

**🎉 Congratulations!** You've built a production-grade secure supply chain for container images. These skills are directly applicable to real-world DevSecOps roles.

Remember: **Security is a journey, not a destination.** Keep learning, keep improving, and always verify before you deploy!