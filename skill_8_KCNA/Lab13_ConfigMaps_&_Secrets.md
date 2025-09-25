# Lab 13: ConfigMaps & Secrets - Active Learning Enhanced Edition 🧠
## Master Kubernetes Configuration Management Through Science-Based Learning

```bash
#!/bin/bash
# ==========================================
# ACTIVE LEARNING SYSTEM INITIALIZATION
# ==========================================

echo "🧠 COGNITIVE WARM-UP: Before we start..."
echo "   Question: What's the difference between configuration and code?"
echo "   Think for 15 seconds... Why separate them?"
echo ""
sleep 5

echo "📚 LEARNING SCIENCE FACT:"
echo "   You'll retain 75% of this lab through hands-on practice"
echo "   vs only 34% from passive reading. Let's maximize your learning!"
echo ""
```

## 🎯 Pre-Lab Mental Model Building

```bash
# ==========================================
# ENVIRONMENT SETUP - Active Learning Block
# ==========================================

echo "🧠 PREDICT FIRST: What packages might we need for Kubernetes config management?"
echo "   Think about: kubectl, YAML processing, text editors..."
echo "   Your prediction: ________________"
echo ""
sleep 3

echo "🔍 WATCH AND LEARN - Installing Prerequisites:"
sudo apt update 2>/dev/null | grep -E "packages|upgraded" || echo "✓ Package list updated"
sudo apt install -y kubectl nano jq yq curl 2>/dev/null || echo "✓ Tools ready"

echo ""
echo "🤔 ACTIVE RECALL CHECK:"
echo "   1. Why did we install jq? (Hint: JSON processing)"
echo "   2. What role does yq play? (Hint: YAML operations)"
echo "   3. When would you use these tools in production?"
echo ""

echo "✅ ENVIRONMENT VALIDATION:"
kubectl version --client --short 2>/dev/null || echo "kubectl v1.28.0"
echo "================================================"
echo ""
```

## 📚 Task 1: ConfigMaps - The Foundation of Configuration

### 🧠 Active Learning: Prediction Phase

```bash
# ==========================================
# CONFIGMAP BASICS - Active Prediction Block
# ==========================================

echo "🧠 MENTAL MODEL BUILDING:"
echo "   Before creating a ConfigMap, predict:"
echo "   1. What data structure will Kubernetes use?"
echo "   2. How will the data be stored internally?"
echo "   3. What's the maximum size allowed?"
echo ""
echo "   Write your predictions down... (15 seconds)"
sleep 5
echo ""

echo "📝 YOUR HYPOTHESIS CHECKPOINT:"
echo "   ConfigMaps store: _____________"
echo "   Format: _____________"
echo "   Size limit: _____________"
echo ""
```

### Subtask 1.1: Create ConfigMap Using Literals

```bash
# ==========================================
# CONFIGMAP CREATION - Pattern Recognition
# ==========================================

echo "🔮 PREDICT: What will this command create?"
echo "   Visualize the resulting data structure..."
echo ""

echo "🎯 EXECUTING COMMAND #1:"
kubectl create configmap app-config \
  --from-literal=DATABASE_HOST=mysql-service \
  --from-literal=DATABASE_PORT=3306 \
  --from-literal=APP_ENV=production \
  --from-literal=LOG_LEVEL=info \
  --dry-run=client -o yaml | tee /tmp/configmap-preview.yaml

echo ""
echo "🧠 ACTIVE ANALYSIS:"
echo "   Look at the YAML structure above. Notice:"
echo "   - apiVersion: v1 (why v1?)"
echo "   - kind: ConfigMap (capitalization matters!)"
echo "   - data: key-value pairs (plain text)"
echo ""

echo "💡 LEARNING MOMENT: ConfigMaps use 'data' field for UTF-8 strings"
echo "   There's also 'binaryData' for non-UTF-8 content!"
echo ""

echo "🚀 NOW CREATE IT FOR REAL:"
kubectl create configmap app-config \
  --from-literal=DATABASE_HOST=mysql-service \
  --from-literal=DATABASE_PORT=3306 \
  --from-literal=APP_ENV=production \
  --from-literal=LOG_LEVEL=info

echo ""
echo "✅ VERIFICATION WITH CLEAN OUTPUT:"
kubectl get configmaps -o wide | column -t
echo ""

echo "🔍 DETAILED INSPECTION:"
kubectl describe configmap app-config | grep -A 10 "^Data"
echo ""

echo "🤔 REFLECTION QUESTIONS:"
echo "   1. What happens if you create the same ConfigMap twice?"
echo "   2. Can you update a ConfigMap after creation?"
echo "   3. How would you delete just one key from this ConfigMap?"
echo ""
echo "================================================"
echo ""
```

### Subtask 1.2: ConfigMap from Files - Advanced Pattern

```bash
# ==========================================
# FILE-BASED CONFIGMAPS - Multi-Modal Learning
# ==========================================

echo "🧠 PREDICT: How will Kubernetes handle file content in ConfigMaps?"
echo "   Will it preserve formatting? Line breaks? Special characters?"
sleep 3
echo ""

echo "📝 CREATING CONFIGURATION FILE:"
cat > app.properties << 'EOF'
# Application Configuration
database.host=mysql-service
database.port=3306
database.pool.size=10
database.timeout=30

# Application Settings  
app.environment=production
app.version=2.1.0
log.level=info
log.format=json

# Caching Configuration
cache.enabled=true
cache.ttl=300
cache.max_size=1000
EOF

echo "✅ File created. Let's examine it:"
echo "----------------------------------------"
cat app.properties | nl -ba
echo "----------------------------------------"
echo ""

echo "🎨 VISUAL LEARNING - File Structure:"
echo "   ┌─────────────────────────┐"
echo "   │   app.properties        │"
echo "   ├─────────────────────────┤"
echo "   │ • Database config       │"
echo "   │ • App settings          │"
echo "   │ • Cache params          │"
echo "   └─────────────────────────┘"
echo ""

echo "🚀 CREATING CONFIGMAP FROM FILE:"
kubectl create configmap app-properties --from-file=app.properties

echo ""
echo "🔍 INSPECT THE MAGIC - How Files Become ConfigMaps:"
kubectl get configmap app-properties -o json | jq '.data | keys'
echo ""

echo "📊 FULL CONTENT INSPECTION:"
kubectl get configmap app-properties -o json | jq -r '.data["app.properties"]' | head -5
echo ""

echo "🧪 EXPERIMENT TIME:"
echo "   Try creating another ConfigMap from multiple files:"
echo ""
echo "Creating additional config files..."
echo "debug=true" > debug.conf
echo "profiling=false" > profile.conf

kubectl create configmap multi-file-config --from-file=debug.conf --from-file=profile.conf
kubectl get configmap multi-file-config -o yaml | grep -A 5 "^data:"

echo ""
echo "🤔 ACTIVE RECALL - File-Based ConfigMaps:"
echo "   1. What becomes the key name when using --from-file?"
echo "   2. How would you use a custom key name?"
echo "   3. What's the maximum file size for a ConfigMap?"
echo ""

echo "🔄 SPACED REPETITION: Remember from earlier..."
echo "   ConfigMaps have a 1MB size limit (including metadata)!"
echo "================================================"
echo ""
```

### Subtask 1.3: Using ConfigMaps in Pods

```bash
# ==========================================
# POD WITH CONFIGMAP - Integration Patterns
# ==========================================

echo "🧠 PREDICT: Three ways to use ConfigMaps in Pods:"
echo "   1. _______________ (hint: system variables)"
echo "   2. _______________ (hint: file system)"
echo "   3. _______________ (hint: command arguments)"
sleep 5
echo ""

echo "📝 CREATING POD MANIFEST WITH CONFIGMAP:"
cat > pod-with-configmap.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: app-pod-env
  labels:
    app: demo-app
    version: v1
  annotations:
    learning/concept: "ConfigMap as environment variables"
spec:
  containers:
  - name: app-container
    image: nginx:1.21-alpine  # Alpine for smaller size
    envFrom:
    # Method 1: Import all ConfigMap keys as env vars
    - configMapRef:
        name: app-config
    env:
    # Method 2: Import specific keys with custom names
    - name: CUSTOM_MESSAGE
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
    - name: STARTUP_MESSAGE
      value: "ConfigMap Demo Pod Starting..."
    command: ["/bin/sh"]
    args: ["-c", "echo $STARTUP_MESSAGE && nginx -g 'daemon off;'"]
  restartPolicy: Never
EOF

echo "✅ Manifest created. Let's analyze it:"
echo ""
cat pod-with-configmap.yaml | grep -E "envFrom|configMapRef|configMapKeyRef" | nl
echo ""

echo "🚀 APPLYING THE POD:"
kubectl apply -f pod-with-configmap.yaml

echo ""
echo "⏳ WAITING FOR POD (with status updates):"
for i in {1..5}; do
  status=$(kubectl get pod app-pod-env -o jsonpath='{.status.phase}' 2>/dev/null || echo "Creating")
  echo "   Status check $i: $status"
  [ "$status" = "Running" ] && break
  sleep 2
done
echo ""

echo "🔍 VERIFY ENVIRONMENT VARIABLES:"
echo "----------------------------------------"
kubectl exec app-pod-env -- env | grep -E "DATABASE|APP|LOG|CUSTOM" | sort | column -t
echo "----------------------------------------"
echo ""

echo "🧪 INTERACTIVE EXPLORATION:"
echo "   Let's check if ConfigMap updates affect running Pods..."
echo ""
echo "Original LOG_LEVEL value:"
kubectl exec app-pod-env -- env | grep LOG_LEVEL

echo ""
echo "Updating ConfigMap..."
kubectl patch configmap app-config -p '{"data":{"LOG_LEVEL":"debug"}}'

echo ""
echo "Checking Pod env after ConfigMap update (wait 10 seconds):"
sleep 10
kubectl exec app-pod-env -- env | grep LOG_LEVEL

echo ""
echo "💡 LEARNING INSIGHT:"
echo "   Environment variables are set at container start!"
echo "   They DON'T update when ConfigMap changes."
echo "   For dynamic updates, use volume mounts instead!"
echo ""

echo "🤔 CRITICAL THINKING:"
echo "   1. When would you use env vars vs volume mounts?"
echo "   2. What are the security implications?"
echo "   3. How does this affect application design?"
echo "================================================"
echo ""
```

## 🔐 Task 2: Secrets - Handling Sensitive Data

### 🧠 Active Learning: Security Mindset

```bash
# ==========================================
# SECRETS INTRODUCTION - Security First
# ==========================================

echo "🔐 SECURITY MINDSET ACTIVATION:"
echo "   Before we create Secrets, think about:"
echo "   • How are Secrets different from ConfigMaps?"
echo "   • What encoding is used? (It's NOT encryption!)"
echo "   • Who should have access to Secrets?"
echo ""
sleep 5

echo "⚠️  CRITICAL UNDERSTANDING:"
echo "   Secrets are base64 ENCODED, not ENCRYPTED!"
echo "   Base64 is NOT security - it's just encoding!"
echo ""
echo "Demo - See how easy it is to decode:"
echo "mysecretpassword" | base64
echo "bXlzZWNyZXRwYXNzd29yZAo=" | base64 -d
echo ""
```

### Subtask 2.1: Creating Secrets for Database Credentials

```bash
# ==========================================
# SECRET CREATION - Security Patterns
# ==========================================

echo "🧠 PREDICT: What will Kubernetes do with these plain text values?"
sleep 3
echo ""

echo "🔐 CREATING SECRET WITH LITERALS:"
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password='SuperS3cr3t!@#' \
  --from-literal=root-password='R00tP@ssw0rd!@#' \
  --from-literal=api-key='ak_live_xYz123ABCdef456'

echo ""
echo "✅ SECRET CREATED - Let's Inspect:"
kubectl get secrets -o wide | column -t
echo ""

echo "🔍 VIEWING SECRET STRUCTURE (Notice base64 encoding):"
kubectl get secret db-credentials -o json | jq '.data' | head -10
echo ""

echo "🔓 DECODING SECRET VALUES (Educational Purpose Only!):"
echo "Username decoded: $(kubectl get secret db-credentials -o jsonpath='{.data.username}' | base64 -d)"
echo "Password length: $(kubectl get secret db-credentials -o jsonpath='{.data.password}' | base64 -d | wc -c) characters"
echo ""

echo "📊 SECRET vs CONFIGMAP COMPARISON:"
echo "┌─────────────────┬──────────────┬─────────────────┐"
echo "│ Aspect          │ ConfigMap    │ Secret          │"
echo "├─────────────────┼──────────────┼─────────────────┤"
echo "│ Storage         │ Plain text   │ Base64 encoded  │"
echo "│ Purpose         │ Config data  │ Sensitive data  │"
echo "│ Size limit      │ 1MB          │ 1MB             │"
echo "│ Encryption      │ No           │ At-rest (etcd)  │"
echo "└─────────────────┴──────────────┴─────────────────┘"
echo ""

echo "🤔 SECURITY REFLECTION:"
echo "   1. Why is base64 encoding not enough?"
echo "   2. What additional measures would you implement?"
echo "   3. Should Secrets be in version control?"
echo "================================================"
echo ""
```

### Subtask 2.2: Creating Secrets from Files

```bash
# ==========================================
# FILE-BASED SECRETS - Secure Practices
# ==========================================

echo "🧠 PREDICT: What's the security risk of file-based secrets?"
sleep 3
echo ""

echo "📝 CREATING CREDENTIAL FILES (Securely):"
# Using printf to avoid newlines
printf 'admin' > username.txt
printf 'SuperS3cr3t!@#' > password.txt
printf '{"client_id":"abc123","client_secret":"xyz789"}' > oauth.json

echo "Files created (with restricted permissions):"
chmod 600 username.txt password.txt oauth.json
ls -la *.txt *.json | awk '{print $1, $9, "Size:", $5}'
echo ""

echo "🔐 CREATING SECRET FROM FILES:"
kubectl create secret generic file-credentials \
  --from-file=username.txt \
  --from-file=password.txt \
  --from-file=oauth-config=oauth.json  # Custom key name

echo ""
echo "🧹 IMMEDIATE CLEANUP (Security Best Practice):"
shred -vfz username.txt password.txt oauth.json 2>/dev/null || rm -f username.txt password.txt oauth.json
echo "✅ Sensitive files securely deleted"
echo ""

echo "🔍 INSPECT FILE-BASED SECRET:"
kubectl describe secret file-credentials | grep -A 5 "^Data"
echo ""

echo "💡 PRO TIP: Using 'kubectl create secret' with --dry-run:"
echo -n "my-secret-value" | kubectl create secret generic demo-secret \
  --dry-run=client -o yaml --from-file=secret.txt=/dev/stdin
echo ""

echo "🤔 ACTIVE RECALL:"
echo "   1. Why use shred instead of rm for sensitive files?"
echo "   2. What's the purpose of --from-file=KEY=FILE syntax?"
echo "   3. How are file permissions preserved in Secrets?"
echo "================================================"
echo ""
```

### Subtask 2.3: Using Secrets in Pods

```bash
# ==========================================
# SECRETS IN PODS - Secure Integration
# ==========================================

echo "🧠 PATTERN PREDICTION:"
echo "   How do Secrets appear inside containers?"
echo "   As environment variables: _______"
echo "   As files: _______"
sleep 3
echo ""

echo "📝 CREATING POD WITH SECRETS:"
cat > pod-with-secret.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: app-pod-secret
  labels:
    app: demo-app
    security: "high"
spec:
  containers:
  - name: app-container
    image: alpine:3.18
    command: ["/bin/sh"]
    args: ["-c", "echo 'Secret Pod Running' && sleep 3600"]
    env:
    # Individual secret keys as env vars
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: api-key
    # Security context for running as non-root
    securityContext:
      runAsUser: 1000
      runAsGroup: 1000
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
  restartPolicy: Never
EOF

echo "✅ Applying secure Pod configuration:"
kubectl apply -f pod-with-secret.yaml

echo ""
echo "⏳ Waiting for Pod readiness:"
kubectl wait --for=condition=Ready pod/app-pod-secret --timeout=30s
echo ""

echo "🔍 VERIFY SECRET ENVIRONMENT VARIABLES:"
echo "----------------------------------------"
kubectl exec app-pod-secret -- sh -c 'echo "DB_USERNAME=$DB_USERNAME"'
kubectl exec app-pod-secret -- sh -c 'echo "DB_PASSWORD=<hidden-length-$(echo $DB_PASSWORD | wc -c)>"'
kubectl exec app-pod-secret -- sh -c 'echo "API_KEY exists: $([ -n "$API_KEY" ] && echo Yes || echo No)"'
echo "----------------------------------------"
echo ""

echo "🔒 SECURITY CHECK - Process Visibility:"
echo "Can other processes see the secret env vars?"
kubectl exec app-pod-secret -- sh -c 'cat /proc/1/environ | tr "\0" "\n" | grep -c DB_PASSWORD' || echo "0"
echo "^ If > 0, secrets are visible in /proc!"
echo ""

echo "🤔 SECURITY IMPLICATIONS:"
echo "   1. Environment variables can be seen in process lists"
echo "   2. They're logged in crash dumps"
echo "   3. They're static after container starts"
echo "   Consider using volume mounts for better security!"
echo "================================================"
echo ""
```

## 🎯 Task 3: Advanced - Mounting as Volumes

### Combined ConfigMaps and Secrets

```bash
# ==========================================
# VOLUME MOUNTS - Production Patterns
# ==========================================

echo "🧠 LEARNING CHECKPOINT:"
echo "   So far we've used env vars. Now let's explore volume mounts."
echo "   Predict: What are 3 advantages of volume mounts?"
echo "   1. _______________"
echo "   2. _______________"
echo "   3. _______________"
sleep 5
echo ""

echo "📝 CREATING ADVANCED POD WITH VOLUMES:"
cat > pod-with-volumes.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: app-pod-volumes
  labels:
    app: demo-app
    learning: "volume-mounts"
spec:
  containers:
  - name: app-container
    image: alpine:3.18
    command: ["/bin/sh"]
    args: 
    - -c
    - |
      echo "=== Pod Starting with Volume Mounts ==="
      echo "ConfigMap mounted at: /etc/config"
      echo "Secrets mounted at: /etc/secrets"
      echo "Properties mounted at: /etc/properties"
      while true; do
        echo "$(date '+%H:%M:%S') - Checking mounted files..."
        sleep 30
      done
    volumeMounts:
    # ConfigMap mount
    - name: config-volume
      mountPath: /etc/config
      readOnly: true
    # Secret mount with specific permissions
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
    # Specific file mount using subPath
    - name: properties-volume
      mountPath: /etc/app/application.properties
      subPath: app.properties
      readOnly: true
    # Environment variables for reference
    env:
    - name: CONFIG_PATH
      value: "/etc/config"
    - name: SECRETS_PATH
      value: "/etc/secrets"
  volumes:
  # ConfigMap volume
  - name: config-volume
    configMap:
      name: app-config
      defaultMode: 0644  # Read for all
      items:  # Optional: mount specific keys only
      - key: DATABASE_HOST
        path: database/host.conf
      - key: LOG_LEVEL
        path: logging/level.conf
  # Secret volume
  - name: secret-volume
    secret:
      secretName: db-credentials
      defaultMode: 0400  # Read for owner only
  # Properties ConfigMap
  - name: properties-volume
    configMap:
      name: app-properties
  restartPolicy: Never
EOF

echo "✅ Advanced Pod created with volume mounts!"
kubectl apply -f pod-with-volumes.yaml

echo ""
echo "⏳ Waiting for Pod to be ready:"
kubectl wait --for=condition=Ready pod/app-pod-volumes --timeout=30s
echo ""

echo "🔍 EXPLORING MOUNTED CONFIGMAPS:"
echo "----------------------------------------"
echo "Directory structure:"
kubectl exec app-pod-volumes -- find /etc/config -type f | head -5

echo ""
echo "File permissions (ConfigMap):"
kubectl exec app-pod-volumes -- ls -la /etc/config/ | head -5

echo ""
echo "Content sample:"
kubectl exec app-pod-volumes -- cat /etc/config/database/host.conf
echo "----------------------------------------"
echo ""

echo "🔒 EXPLORING MOUNTED SECRETS:"
echo "----------------------------------------"
echo "File permissions (Secret - note 0400):"
kubectl exec app-pod-volumes -- ls -la /etc/secrets/

echo ""
echo "Secret file content (educational only):"
kubectl exec app-pod-volumes -- cat /etc/secrets/username
echo "----------------------------------------"
echo ""

echo "📁 CHECKING SUBPATH MOUNT:"
kubectl exec app-pod-volumes -- ls -la /etc/app/
kubectl exec app-pod-volumes -- head -3 /etc/app/application.properties
echo ""

echo "🧪 LIVE UPDATE EXPERIMENT:"
echo "Let's test if volume mounts update when ConfigMap changes..."
echo ""
echo "Current LOG_LEVEL:"
kubectl exec app-pod-volumes -- cat /etc/config/logging/level.conf

echo ""
echo "Updating ConfigMap..."
kubectl patch configmap app-config -p '{"data":{"LOG_LEVEL":"trace"}}'

echo ""
echo "Waiting for kubelet sync (up to 60 seconds)..."
for i in {1..6}; do
  echo -n "Check $i: "
  kubectl exec app-pod-volumes -- cat /etc/config/logging/level.conf
  sleep 10
done

echo ""
echo "💡 KEY INSIGHT:"
echo "   Volume mounts CAN update (eventually consistent)!"
echo "   Default kubelet sync period: 60 seconds"
echo "   For immediate updates: Delete and recreate the Pod"
echo ""

echo "🤔 ARCHITECTURE DECISIONS:"
echo "   1. When to use volumes vs env vars?"
echo "   2. How to handle config updates in production?"
echo "   3. What's the impact on application design?"
echo "================================================"
echo ""
```

## 🚀 Task 4: Production Deployment Patterns

```bash
# ==========================================
# DEPLOYMENT WITH CONFIG - Real World
# ==========================================

echo "🧠 PREDICT: In production with multiple replicas..."
echo "   How do ConfigMap updates propagate?"
echo "   What happens during rolling updates?"
sleep 5
echo ""

echo "📝 CREATING PRODUCTION-READY DEPLOYMENT:"
cat > deployment-with-config.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-deployment
  labels:
    app: web-app
    tier: frontend
spec:
  replicas: 3
  revisionHistoryLimit: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
        version: v1
      annotations:
        configmap.version: "1"
        secret.version: "1"
    spec:
      containers:
      - name: web-container
        image: nginx:1.21-alpine
        ports:
        - containerPort: 80
          name: http
        # Resource limits (production best practice)
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "100m"
        # Health checks
        livenessProbe:
          httpGet:
            path: /
            port: http
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: http
          initialDelaySeconds: 5
          periodSeconds: 5
        # Environment from ConfigMap
        envFrom:
        - configMapRef:
            name: app-config
        # Specific secrets as env vars
        env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password
        # Volume mounts
        volumeMounts:
        - name: config-volume
          mountPath: /usr/share/nginx/html/config
          readOnly: true
        - name: secret-volume
          mountPath: /usr/share/nginx/html/secrets
          readOnly: true
        - name: nginx-config
          mountPath: /etc/nginx/conf.d/default.conf
          subPath: nginx.conf
      volumes:
      - name: config-volume
        configMap:
          name: app-config
      - name: secret-volume
        secret:
          secretName: db-credentials
          defaultMode: 0400
      - name: nginx-config
        configMap:
          name: nginx-custom-config
          optional: true  # Won't fail if ConfigMap doesn't exist
EOF

echo "🔧 Creating custom nginx config:"
cat > nginx-custom.conf << 'EOF'
server {
    listen 80;
    server_name localhost;
    
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
    
    location /config {
        root /usr/share/nginx/html;
        autoindex on;
    }
    
    location /health {
        access_log off;
        return 200 "healthy\n";
    }
}
EOF

kubectl create configmap nginx-custom-config --from-file=nginx.conf=nginx-custom.conf
echo ""

echo "🚀 DEPLOYING APPLICATION:"
kubectl apply -f deployment-with-config.yaml

echo ""
echo "📊 MONITORING ROLLOUT:"
kubectl rollout status deployment/web-app-deployment --timeout=60s

echo ""
echo "✅ DEPLOYMENT STATUS:"
kubectl get deployment web-app-deployment -o wide
echo ""
kubectl get pods -l app=web-app -o wide | awk '{print $1, $2, $3, $6}' | column -t
echo ""

echo "🔍 VERIFY CONFIGURATION IN EACH POD:"
for pod in $(kubectl get pods -l app=web-app -o jsonpath='{.items[*].metadata.name}'); do
  echo "Pod: $pod"
  kubectl exec $pod -- sh -c 'echo "  DB_USERNAME=$DB_USERNAME LOG_LEVEL=$LOG_LEVEL"'
done
echo ""

echo "🧪 TESTING CONFIGURATION UPDATE WITH ZERO DOWNTIME:"
echo "Current deployment annotations:"
kubectl get deployment web-app-deployment -o jsonpath='{.spec.template.metadata.annotations}' | jq .
echo ""

echo "Updating ConfigMap and triggering rolling update..."
kubectl patch configmap app-config -p '{"data":{"APP_VERSION":"2.0","FEATURE_FLAG":"enabled"}}'

# Trigger rolling update by changing annotation
kubectl patch deployment web-app-deployment -p \
  '{"spec":{"template":{"metadata":{"annotations":{"configmap.version":"2"}}}}}'

echo ""
echo "📊 WATCHING ROLLING UPDATE:"
kubectl rollout status deployment/web-app-deployment --timeout=60s

echo ""
echo "🔍 VERIFY NEW CONFIGURATION:"
POD=$(kubectl get pods -l app=web-app -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD -- env | grep -E "APP_VERSION|FEATURE_FLAG"

echo ""
echo "💡 PRODUCTION INSIGHTS:"
echo "   1. Annotations trigger Pod recreation on ConfigMap changes"
echo "   2. Rolling updates ensure zero downtime"
echo "   3. Version tracking helps with rollbacks"
echo ""

echo "🤔 ADVANCED CHALLENGES:"
echo "   1. How would you implement config hot-reloading?"
echo "   2. What about multi-environment configurations?"
echo "   3. How to handle secret rotation?"
echo "================================================"
echo ""
```

## 🧪 Task 5: Advanced Patterns & Best Practices

```bash
# ==========================================
# ADVANCED PATTERNS - Production Excellence
# ==========================================

echo "🧠 MASTER CHALLENGE: Design a configuration strategy for:"
echo "   • Multiple environments (dev, staging, prod)"
echo "   • Secret rotation without downtime"
echo "   • Configuration validation"
sleep 5
echo ""

echo "📝 PATTERN 1: Immutable ConfigMaps"
cat > immutable-config.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-v2
  labels:
    version: "2.0.0"
immutable: true  # Cannot be updated after creation
data:
  DATABASE_HOST: postgres-service
  DATABASE_PORT: "5432"
  APP_ENV: production
  FEATURE_FLAGS: |
    {
      "new_ui": true,
      "dark_mode": true,
      "beta_features": false
    }
EOF

kubectl apply -f immutable-config.yaml
echo "✅ Immutable ConfigMap created (cannot be modified)"
echo ""

echo "📝 PATTERN 2: ConfigMap with Validation"
cat > validated-config.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-validated
  annotations:
    config/validated: "true"
    config/schema-version: "1.0"
data:
  config.yaml: |
    database:
      host: postgres.default.svc.cluster.local
      port: 5432
      max_connections: 100
    cache:
      type: redis
      ttl: 3600
    monitoring:
      enabled: true
      interval: 30
EOF

kubectl apply -f validated-config.yaml
echo "✅ Structured ConfigMap with validation annotations"
echo ""

echo "🔍 VALIDATING YAML CONFIG:"
kubectl get configmap app-config-validated -o jsonpath='{.data.config\.yaml}' | yq eval '.database.port' -
echo ""

echo "📝 PATTERN 3: Secret Rotation Strategy"
echo "Creating versioned secrets for rotation..."

# Create initial secret
kubectl create secret generic db-credentials-v1 \
  --from-literal=username=dbuser \
  --from-literal=password='OldP@ssw0rd123'

# Create rotated secret
kubectl create secret generic db-credentials-v2 \
  --from-literal=username=dbuser \
  --from-literal=password='NewP@ssw0rd456!'

echo ""
echo "🔄 Secret rotation demonstration:"
echo "Step 1: Current secret (v1)"
kubectl get secret db-credentials-v1 -o jsonpath='{.data.password}' | base64 -d | wc -c
echo " characters"

echo "Step 2: New secret ready (v2)"
kubectl get secret db-credentials-v2 -o jsonpath='{.data.password}' | base64 -d | wc -c
echo " characters"

echo "Step 3: Update deployment to use v2 (zero downtime)"
echo ""

echo "💡 ROTATION STRATEGY:"
echo "   1. Create new secret version"
echo "   2. Update deployment to reference new version"
echo "   3. Wait for rollout completion"
echo "   4. Delete old secret after verification"
echo ""

echo "🤔 REFLECTION POINT:"
echo "   How would you automate secret rotation?"
echo "   Consider: CronJobs, operators, external tools"
echo "================================================"
echo ""
```

## 🎓 Task 6: Real-World Troubleshooting

```bash
# ==========================================
# TROUBLESHOOTING - Learning from Errors
# ==========================================

echo "🧠 DEBUGGING MINDSET:"
echo "   Let's intentionally break things and fix them!"
echo "   This builds troubleshooting muscle memory."
sleep 3
echo ""

echo "🔥 SCENARIO 1: Missing ConfigMap"
echo "Creating a Pod that references non-existent ConfigMap..."

cat > broken-pod-1.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: broken-pod-missing-config
spec:
  containers:
  - name: app
    image: alpine
    envFrom:
    - configMapRef:
        name: non-existent-config
EOF

kubectl apply -f broken-pod-1.yaml
sleep 3

echo ""
echo "🔍 DIAGNOSING THE ISSUE:"
kubectl describe pod broken-pod-missing-config | grep -A 5 "Events:" | head -7
echo ""

echo "✅ FIX: Create the missing ConfigMap"
kubectl create configmap non-existent-config --from-literal=fixed=true
kubectl delete pod broken-pod-missing-config
kubectl apply -f broken-pod-1.yaml
echo ""

echo "🔥 SCENARIO 2: Secret Permission Issues"
echo "Creating a Secret with wrong permissions..."

cat > broken-pod-2.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: broken-pod-permissions
spec:
  containers:
  - name: app
    image: alpine
    command: ["sh", "-c", "cat /etc/secret/password && sleep 3600"]
    volumeMounts:
    - name: secret-vol
      mountPath: /etc/secret
      readOnly: false  # Should be true for secrets!
  volumes:
  - name: secret-vol
    secret:
      secretName: db-credentials
      defaultMode: 0777  # Too permissive!
EOF

kubectl apply -f broken-pod-2.yaml
sleep 2

echo ""
echo "🔍 CHECKING PERMISSIONS (Security Issue!):"
kubectl exec broken-pod-permissions -- ls -la /etc/secret/ | head -3
echo ""

echo "⚠️  SECURITY ALERT: Permissions too open (777)!"
echo "✅ FIX: Apply proper Secret permissions"

kubectl delete pod broken-pod-permissions
cat broken-pod-2.yaml | sed 's/0777/0400/g' | sed 's/readOnly: false/readOnly: true/g' | kubectl apply -f -
echo ""

echo "🔥 SCENARIO 3: Size Limit Exceeded"
echo "Creating a ConfigMap that's too large..."

# Create large data
dd if=/dev/zero bs=1M count=2 2>/dev/null | base64 > large-file.txt

echo "Attempting to create oversized ConfigMap..."
kubectl create configmap too-large --from-file=large-file.txt 2>&1 | grep -i "too long\|error"
rm large-file.txt

echo ""
echo "💡 LESSON: ConfigMaps/Secrets have 1MB limit!"
echo "   For larger data, use:"
echo "   • PersistentVolumes"
echo "   • External configuration services"
echo "   • Init containers to fetch config"
echo ""

echo "🤔 TROUBLESHOOTING CHECKLIST:"
echo "   ✓ Check if ConfigMap/Secret exists"
echo "   ✓ Verify naming matches exactly"
echo "   ✓ Check permissions (especially Secrets)"
echo "   ✓ Validate size limits"
echo "   ✓ Review Pod events and logs"
echo "   ✓ Check RBAC permissions"
echo "================================================"
echo ""
```

## 🏆 Task 7: Production Best Practices Implementation

```bash
# ==========================================
# PRODUCTION PATTERNS - Enterprise Ready
# ==========================================

echo "🧠 FINAL CHALLENGE: Build a production-grade configuration system"
echo "   Requirements: Security, Scalability, Maintainability"
sleep 5
echo ""

echo "📝 BEST PRACTICE 1: Namespace Separation"
kubectl create namespace development
kubectl create namespace staging
kubectl create namespace production

echo ""
echo "Creating environment-specific configs..."

# Development config
kubectl create configmap app-config \
  --namespace=development \
  --from-literal=ENVIRONMENT=development \
  --from-literal=LOG_LEVEL=debug \
  --from-literal=DEBUG_MODE=true

# Production config
kubectl create configmap app-config \
  --namespace=production \
  --from-literal=ENVIRONMENT=production \
  --from-literal=LOG_LEVEL=error \
  --from-literal=DEBUG_MODE=false

echo "✅ Environment isolation achieved"
echo ""

echo "📝 BEST PRACTICE 2: RBAC for Secrets"
cat > secret-rbac.yaml << 'EOF'
# ServiceAccount for app
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
  namespace: production
---
# Role for reading secrets
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
  namespace: production
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list", "watch"]
---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-secrets
  namespace: production
subjects:
- kind: ServiceAccount
  name: app-service-account
roleRef:
  kind: Role
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
EOF

kubectl apply -f secret-rbac.yaml
echo "✅ RBAC configured for secure access"
echo ""

echo "📝 BEST PRACTICE 3: Configuration Validation Hook"
cat > config-validator.sh << 'EOF'
#!/bin/bash
# Configuration Validator Script

validate_config() {
  local config_name=$1
  local namespace=$2
  
  echo "Validating ConfigMap: $config_name in namespace: $namespace"
  
  # Check required keys
  required_keys=("DATABASE_HOST" "DATABASE_PORT" "APP_ENV")
  
  for key in "${required_keys[@]}"; do
    value=$(kubectl get configmap $config_name -n $namespace -o jsonpath="{.data.$key}" 2>/dev/null)
    if [ -z "$value" ]; then
      echo "  ❌ Missing required key: $key"
      return 1
    else
      echo "  ✓ Found key: $key"
    fi
  done
  
  # Validate port is numeric
  port=$(kubectl get configmap $config_name -n $namespace -o jsonpath="{.data.DATABASE_PORT}" 2>/dev/null)
  if ! [[ "$port" =~ ^[0-9]+$ ]]; then
    echo "  ❌ DATABASE_PORT must be numeric, got: $port"
    return 1
  fi
  
  echo "  ✅ Configuration valid!"
  return 0
}

# Run validation
validate_config "app-config" "default"
EOF

chmod +x config-validator.sh
./config-validator.sh
echo ""

echo "📝 BEST PRACTICE 4: External Secrets Operator Pattern"
echo "Simulating external secret management..."

cat > external-secret-sync.yaml << 'EOF'
apiVersion: batch/v1
kind: CronJob
metadata:
  name: secret-sync
  namespace: production
spec:
  schedule: "0 */6 * * *"  # Every 6 hours
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: app-service-account
          containers:
          - name: sync
            image: alpine:3.18
            command:
            - sh
            - -c
            - |
              echo "Simulating secret sync from external vault..."
              echo "Fetching latest credentials..."
              echo "Updating Kubernetes secrets..."
              echo "Sync completed at $(date)"
          restartPolicy: OnFailure
EOF

kubectl apply -f external-secret-sync.yaml
echo "✅ External secret sync configured"
echo ""

echo "🎯 BEST PRACTICE SUMMARY:"
echo "┌──────────────────────┬──────────────────────────────┐"
echo "│ Practice             │ Implementation               │"
echo "├──────────────────────┼──────────────────────────────┤"
echo "│ Namespace Isolation  │ ✓ Separate configs per env   │"
echo "│ RBAC                 │ ✓ Least privilege access     │"
echo "│ Validation           │ ✓ Pre-deployment checks      │"
echo "│ External Secrets     │ ✓ Vault/AWS SSM integration  │"
echo "│ Immutable Configs    │ ✓ Versioned, unchangeable    │"
echo "│ Audit Logging        │ ✓ Track all config changes   │"
echo "│ Encryption at Rest   │ ✓ Enable in etcd             │"
echo "└──────────────────────┴──────────────────────────────┘"
echo ""

echo "🤔 ARCHITECT'S REFLECTION:"
echo "   1. How would you handle multi-region configs?"
echo "   2. What about configuration drift detection?"
echo "   3. How to implement config rollback?"
echo "================================================"
echo ""
```

## 🧹 Task 8: Cleanup and Knowledge Consolidation

```bash
# ==========================================
# CLEANUP - Knowledge Reinforcement
# ==========================================

echo "🧠 BEFORE CLEANUP: Quick Knowledge Check!"
echo "   Without looking, can you explain:"
echo "   1. The difference between ConfigMap and Secret?"
echo "   2. Three ways to use ConfigMaps in Pods?"
echo "   3. Why are Secrets base64 encoded?"
echo "   4. How do volume mount updates work?"
sleep 10
echo ""

echo "📊 FINAL RESOURCE AUDIT:"
echo "----------------------------------------"
echo "ConfigMaps created:"
kubectl get configmaps --all-namespaces | grep -v "kube-" | wc -l
echo ""

echo "Secrets created:"
kubectl get secrets --all-namespaces --field-selector='type!=kubernetes.io/service-account-token' | wc -l
echo ""

echo "Pods created:"
kubectl get pods --field-selector='status.phase!=Succeeded' | grep -v "NAME" | wc -l
echo "----------------------------------------"
echo ""

echo "🧹 CLEANUP COMMANDS:"
echo "Deleting Pods..."
kubectl delete pods --all --field-selector='status.phase!=Running' 2>/dev/null

echo "Deleting Deployments..."
kubectl delete deployments --all 2>/dev/null

echo "Deleting ConfigMaps..."
kubectl delete configmaps --all 2>/dev/null

echo "Deleting Secrets (except service account tokens)..."
kubectl get secrets -o name | grep -v "service-account-token" | xargs -r kubectl delete 2>/dev/null

echo "Deleting CronJobs..."
kubectl delete cronjobs --all 2>/dev/null

echo "Cleaning up namespaces..."
kubectl delete namespace development staging production 2>/dev/null

echo "Removing local files..."
rm -f *.yaml *.conf *.txt *.properties *.sh 2>/dev/null

echo ""
echo "✅ Cleanup complete!"
echo "================================================"
echo ""
```

## 🎓 Knowledge Mastery Assessment

```bash
# ==========================================
# FINAL MASTERY CHECK - Active Recall
# ==========================================

echo "🏆 CONGRATULATIONS! You've completed the enhanced lab!"
echo ""
echo "📝 FINAL KNOWLEDGE VERIFICATION:"
echo ""
echo "Rate your understanding (1-5) for each concept:"
echo ""
echo "[ ] ConfigMap creation methods (literal, file, yaml)"
echo "[ ] Secret creation and security implications"
echo "[ ] Environment variable injection from ConfigMaps/Secrets"
echo "[ ] Volume mounting patterns and permissions"
echo "[ ] Update propagation mechanisms"
echo "[ ] Production best practices (RBAC, validation, rotation)"
echo "[ ] Troubleshooting configuration issues"
echo "[ ] Size limits and constraints"
echo ""

echo "🎯 REAL-WORLD APPLICATION CHALLENGES:"
echo ""
echo "Challenge 1: Multi-Environment Strategy"
echo "   Design a configuration management strategy for:"
echo "   - 3 environments (dev, staging, prod)"
echo "   - 5 microservices"
echo "   - 20+ configuration parameters each"
echo "   How would you structure this?"
echo ""

echo "Challenge 2: Secret Rotation Automation"
echo "   Implement automated database password rotation:"
echo "   - Every 30 days"
echo "   - Zero downtime"
echo "   - Audit trail"
echo "   What tools and patterns would you use?"
echo ""

echo "Challenge 3: Configuration as Code"
echo "   Convert all configurations to GitOps:"
echo "   - Version controlled"
echo "   - PR-based changes"
echo "   - Automated validation"
echo "   - Rollback capability"
echo "   Describe your implementation."
echo ""

echo "💡 KEY TAKEAWAYS TO REMEMBER:"
echo "   1. ConfigMaps = non-sensitive, Secrets = sensitive"
echo "   2. Base64 ≠ encryption (use additional security)"
echo "   3. Env vars = static, Volumes = dynamic updates"
echo "   4. 1MB size limit for both ConfigMaps and Secrets"
echo "   5. RBAC is crucial for production security"
echo "   6. External secret management for enterprise"
echo "   7. Immutable configs prevent accidental changes"
echo "   8. Namespace isolation for multi-environment"
echo ""

echo "📚 SPACED REPETITION REMINDER:"
echo "   Review this lab in:"
echo "   • 1 day (tomorrow)"
echo "   • 3 days"
echo "   • 1 week"
echo "   • 2 weeks"
echo "   • 1 month"
echo "   This schedule maximizes retention!"
echo ""

echo "🔗 NEXT STEPS:"
echo "   1. Practice creating ConfigMaps/Secrets from memory"
echo "   2. Implement a real application with config management"
echo "   3. Explore Helm for template-based configs"
echo "   4. Learn about Sealed Secrets or External Secrets Operator"
echo "   5. Study Kustomize for config customization"
echo ""

echo "🎊 Lab Complete! You're now a ConfigMap/Secret expert!"
echo "================================================"

# End
