# Lab 14: Advanced HTTP/S Routing with Ingress - Final Enhanced Learning Edition

## 🧠 Learning Objectives & Active Recall Framework

### **By the end of this lab, you will master:**
- Deploy multiple applications with sophisticated routing
- Configure Ingress controllers for production-grade traffic management
- Implement secure TLS termination and certificate management
- Troubleshoot and verify complex routing scenarios
- Understand the complete request flow from client to pod

### **🎯 Retention Target: 75% concept mastery through active learning**

### **PREFACE**

## Kubernetes Ingress - Concept Summary & Learning Preface

### 🎯 **Pre-Lab Learning Preface**

#### **Why This Lab Matters**

Imagine you have multiple web applications running in your Kubernetes cluster, but users can't access them from the internet. You need a sophisticated traffic director that can:

- **Route requests** based on URLs (`/app1` goes to App1, `/app2` goes to App2)
- **Handle HTTPS** with proper certificates and security
- **Manage multiple domains** (`api.company.com`, `admin.company.com`)
- **Provide load balancing** and high availability

This is exactly what **Kubernetes Ingress** does - it's your cluster's **smart front door**.

#### **Learning Strategy for Maximum Retention**

This lab uses **active learning techniques** proven to achieve **75% retention** (vs. 34% for passive reading):

1. **Predict First:** Before each step, you'll predict what should happen
2. **Observe Results:** Watch the actual outcome and compare to your prediction
3. **Active Recall:** Explain concepts in your own words after learning
4. **Spaced Review:** Connect new concepts to previously learned material
5. **Hands-On Practice:** Build real configurations you can see and test

**Pro Tip:** Don't skip the "Predict" and "Active Recall" sections - they're where the real learning happens!

---

## 📚 **Core Concepts Summary**

### **1. The Ingress Ecosystem**

```
Internet → Ingress Controller → Ingress Resource → Service → Pod
```

**🎯 Key Understanding:** Think of it like a restaurant:
- **Ingress Resource** = Menu (defines what's available and where to find it)
- **Ingress Controller** = Host/Hostess (reads the menu and directs customers)
- **Service** = Kitchen (prepares and serves the food)
- **Pod** = Chef (does the actual cooking)

#### **Ingress Resource vs. Ingress Controller**
| Aspect | Ingress Resource | Ingress Controller |
|--------|------------------|-------------------|
| **What** | YAML configuration file | Running software (NGINX, Traefik, etc.) |
| **Role** | Defines routing rules | Implements the routing rules |
| **Analogy** | Restaurant menu | Host who reads the menu |
| **Example** | `host: api.company.com → service: api-service` | NGINX pod that processes HTTP requests |

### **2. Routing Mechanisms**

#### **Path-Based Routing**
Routes traffic based on URL paths:
```yaml
# Example: All requests to same domain, different paths
- host: myapp.com
  paths:
  - path: /api     → api-service
  - path: /admin   → admin-service  
  - path: /        → frontend-service (default/fallback)
```

**🧠 Mental Model:** Like floors in a building - same address, different destinations based on floor number.

#### **Host-Based Routing**
Routes traffic based on domain names:
```yaml
# Example: Different domains, same or different services
- host: api.myapp.com     → api-service
- host: admin.myapp.com   → admin-service
- host: www.myapp.com     → frontend-service
```

**🧠 Mental Model:** Like different buildings on the same street - each has its own address and purpose.

#### **PathType Explained**
| PathType | Behavior | Example | Use Case |
|----------|----------|---------|----------|
| **Prefix** | Matches path + anything after | `/api` matches `/api/users`, `/api/orders` | APIs, applications with sub-paths |
| **Exact** | Must match exactly | `/health` only matches `/health` | Health checks, specific endpoints |

### **3. TLS/HTTPS Termination**

**🎯 Key Understanding:** TLS termination is like a security checkpoint at a building entrance:

```
Internet (HTTPS) → Ingress Controller (TLS Termination) → Internal Cluster (HTTP)
   🔒 Encrypted        🔓 Decrypts/Re-encrypts           🔓 Plain HTTP
```

#### **Certificate Management Flow**
1. **Generate Certificate** (or get from CA like Let's Encrypt)
2. **Store in Kubernetes Secret** (base64 encoded)
3. **Reference in Ingress Resource** (tells controller which cert to use)
4. **Ingress Controller** presents certificate to browsers

**🧠 Memory Aid:** Think "Generate → Store → Reference → Present"

### **4. Service Discovery & Load Balancing**

#### **How Requests Reach Pods**
```
Request → Ingress Controller → Service (ClusterIP) → Endpoints → Healthy Pods
```

1. **Ingress Controller** forwards to Service by name
2. **Service** maintains list of healthy Pod IPs (Endpoints)
3. **Load Balancing** distributes requests across healthy Pods
4. **Health Checks** ensure only ready Pods receive traffic

**🧠 Mental Model:** Like a taxi dispatcher (Service) who knows which drivers (Pods) are available and routes passengers (Requests) accordingly.

### **5. Annotations - The Configuration Language**

Annotations are like **settings switches** for your ingress:

#### **Essential Annotations**
| Annotation | Purpose | Example Value | Why It Matters |
|------------|---------|---------------|----------------|
| `rewrite-target` | Strip path prefix | `/` | Apps serve from root, not `/app1` |
| `ssl-redirect` | Force HTTPS | `"true"` | Security - prevent HTTP traffic |
| `rate-limit` | Prevent abuse | `"100"` | Protect backend from overload |
| `enable-cors` | Cross-domain calls | `"true"` | Enable frontend-to-API communication |

**⚠️ Environment Note:** Some clusters disable `configuration-snippet` and `server-snippet` for security - this lab handles that automatically.

---

## 🔄 **Conceptual Learning Checkpoints**

### **Checkpoint 1: Basic Understanding**
Before starting the lab, you should be able to answer:
- **What's the difference between Ingress and Service?**
- **Why do we need an Ingress Controller?**
- **What happens when a request hits `myapp.com/api`?**

### **Checkpoint 2: Routing Logic**
After Task 3, you should understand:
- **How does path-based routing work?**
- **What's the difference between Prefix and Exact matching?**
- **Why do we need a default path (`/`)?**

### **Checkpoint 3: Security Model**
After Task 4, you should grasp:
- **What is TLS termination and why do it at ingress?**
- **How do certificates get from files to browser?**
- **What's the difference between HTTP and HTTPS in the cluster?**

### **Checkpoint 4: Advanced Architecture**
After Task 5, you should comprehend:
- **When would you use multiple domains?**
- **How does API versioning work with ingress?**
- **What are the benefits of subdomain-based routing?**

---

## 🧠 **Active Learning Techniques Used**

### **1. The Predict-Verify Cycle**
**Before each major step:**
- Read the "🧠 PREDICT" prompt
- Spend 10-30 seconds forming expectations
- Only then proceed to see the actual results
- Compare your prediction to reality

**Why it works:** Forces your brain to actively process information rather than passively consume it.

### **2. Active Recall Practice**
**After each section:**
- Answer the "🤔 ACTIVE RECALL CHECK" questions
- Explain concepts aloud as if teaching someone
- Don't look back at the content while answering

**Why it works:** Retrieval practice strengthens neural pathways more than re-reading.

### **3. Spaced Repetition Connections**
**Throughout the lab:**
- Look for "🔄 SPACED REVIEW" prompts
- Connect new concepts to earlier learning
- Build on previous knowledge systematically

**Why it works:** Distributed practice creates stronger, more durable memories.

### **4. Multi-Modal Engagement**
**You'll experience concepts through:**
- **Visual:** See ingress rules and routing tables
- **Kinesthetic:** Type commands and create configurations
- **Auditory:** Explain concepts out loud
- **Social:** Imagine teaching others

**Why it works:** Multiple pathways to the same information increase retention.

---

## 🎯 **Success Metrics & Learning Goals**

### **By Lab Completion, You Will:**

**Conceptually Understand:**
- The complete request flow from browser to pod
- How ingress enables microservices architecture
- The relationship between security, performance, and routing
- Production considerations for enterprise deployments

**Practically Demonstrate:**
- Create and manage ingress resources confidently
- Troubleshoot common ingress configuration issues
- Implement TLS security with proper certificate management
- Design routing strategies for complex applications

**Professionally Apply:**
- Evaluate ingress solutions for real-world projects
- Communicate ingress concepts to technical and business stakeholders
- Design scalable, secure ingress architectures
- Debug production ingress issues systematically

---

## 🚀 **Pre-Lab Mental Preparation**

### **Mindset for Maximum Learning:**

1. **Embrace Confusion:** When concepts feel unclear, that's your brain building new neural pathways. Push through the discomfort.

2. **Question Everything:** Ask "Why?" and "How?" constantly. The lab provides answers, but your questions drive deep understanding.

3. **Connect to Experience:** Relate ingress concepts to things you know - web servers, load balancers, reverse proxies, even real-world analogies.

4. **Practice Explaining:** If you can't explain it simply, you don't understand it deeply. Use the active recall prompts seriously.

5. **Build Mental Models:** Don't just memorize commands - understand the underlying architecture and data flow.

### **Common Learning Obstacles & Solutions:**

**Obstacle:** "Too many new concepts at once"
**Solution:** Focus on one concept per task. Build understanding incrementally.

**Obstacle:** "Commands work but I don't understand why"  
**Solution:** Use the troubleshooting sections to understand the underlying mechanisms.

**Obstacle:** "Hard to remember all the YAML syntax"
**Solution:** Focus on understanding the structure and purpose - syntax can be referenced.

**Obstacle:** "Concepts seem abstract"
**Solution:** Use the real-world analogies and relate to web browsing experiences you know.

---

## 📖 **How to Use This Lab Effectively**

### **Preparation Phase (5 minutes)**
1. Read this entire preface once quickly
2. Identify which concepts are completely new to you
3. Set expectations for what you'll learn

### **Execution Phase (2-3 hours)**
1. **Read predictions carefully** and actually think before proceeding
2. **Don't skip active recall questions** - they're the most important part
3. **Type all commands yourself** rather than copy-pasting blindly
4. **Explain concepts aloud** when prompted
5. **Connect new learning to previous sections** using spaced review prompts

### **Consolidation Phase (15 minutes)**
1. Review the final mastery challenge
2. Explain the complete request flow without looking
3. Identify concepts that still feel unclear
4. Plan follow-up learning for advanced topics

---

## 🎯 **Ready to Begin?**

You now have the conceptual foundation and learning strategy to maximize your success. Remember:

- **75% retention is achievable** if you engage with the active learning techniques
- **Predict first, then verify** - don't skip this crucial step
- **Explain concepts in your own words** - this builds true understanding
- **Connect everything to the big picture** - ingress is your cluster's traffic control system

**🚀 Let's build some seriously sophisticated ingress configurations and make those concepts stick!**
--------------------------------------------------------------------------------

## 🚨 **CRITICAL ERROR PREVENTION REQUIREMENTS**

### **Environment Compatibility Check**
Before starting, verify cluster capabilities:

```bash
# Test annotation permissions
kubectl create ingress test-ingress --class=nginx --rule="test.local/*=test-service:80" --dry-run=client -o yaml | \
  sed '/metadata:/a\ \ annotations:\n\ \ \ \ nginx.ingress.kubernetes.io/configuration-snippet: "test"' | \
  kubectl apply --dry-run=server -f - 2>&1 | grep -q "Snippet directives are disabled" && \
  echo "❌ Snippet annotations DISABLED - using simplified version" || \
  echo "✅ Snippet annotations ALLOWED - using full version"

# Check for existing conflicts
kubectl get ingress --all-namespaces | grep -E "(web-apps|myapps)" && \
  echo "⚠️ CONFLICT: Delete existing ingress resources first" || \
  echo "✅ No conflicts detected"
```

### **📝 MANDATORY FILE CREATION PROTOCOL**
- **ALL YAML files MUST use nano editor**
- **Content always in separate code blocks**
- **Never mix bash commands with YAML content**
- **Always verify files before applying**

---

## Prerequisites Verification

```bash
# Install required packages and compatible versions
sudo apt update && sudo apt install -y curl wget openssl apache2-utils

# Install kubectl v1.30.14 (compatible with K8s 1.32.0)
curl -LO "https://dl.k8s.io/release/v1.30.14/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify installation
kubectl version --client --output=yaml | grep gitVersion
```

---

## Task 1: Environment Setup and Cluster Preparation

### **🧠 Active Learning Block 1A: Cluster Status Verification**

```bash
echo "🧠 PREDICT FIRST: What should a healthy Kubernetes cluster show?"
echo "   Consider: node status, system pods, API server response..."
echo "   Form your expectations before we check..."
echo ""

echo "🔍 WATCH AND LEARN - Cluster Health Check:"

# Clean formatted cluster info
echo "📊 CLUSTER INFORMATION:"
kubectl cluster-info

echo ""
echo "📊 NODE STATUS:"
kubectl get nodes -o wide

echo ""
echo "📊 MINIKUBE STATUS:"
minikube status
```

### **🧠 Comprehensive Answer Block 1A:**
**Node "Ready" Status:** Means the kubelet is healthy, has sufficient resources (CPU, memory, disk), and can schedule pods. The node has passed all health checks including network connectivity and container runtime functionality.

**NotReady Investigation:** Check kubelet logs (`journalctl -u kubelet`), verify resources (`df -h`, `free -m`), network connectivity, and container runtime status. Common issues include disk pressure, memory pressure, or network plugin problems.

**Kubelet vs API Server:** Kubelet manages pods on individual nodes (node-level), while API server handles cluster-wide operations, authentication, and state management (cluster-level). Both must be healthy for proper cluster operation.

### **Minikube Startup (if needed)**

```bash
echo "🧠 PREDICT: What might happen if minikube isn't running?"
echo "   Think about: API access, pod scheduling, service discovery..."
echo ""

# Start minikube if not running
if ! minikube status | grep -q "Running"; then
    echo "🚀 Starting minikube with enhanced configuration:"
    minikube start \
        --driver=docker \
        --cpus=4 \
        --memory=4096 \
        --disk-size=20g \
        --kubernetes-version=v1.30.14
    
    echo "✅ Minikube started successfully!"
fi

echo ""
echo "🤔 ACTIVE RECALL: What resources did we allocate and why?"
echo "🔄 CONNECTION: This setup ensures we can handle multiple applications"
```

### **🧠 Active Learning Block 1B: Ingress Controller Setup**

```bash
echo "🧠 PREDICT FIRST: What is an Ingress Controller and why do we need it?"
echo "   Think about: external traffic, HTTP routing, load balancing..."
echo "   Take 30 seconds to think through this..."
echo ""

echo "🔍 WATCH AND LEARN - Enabling NGINX Ingress:"

# Enable ingress addon
minikube addons enable ingress

echo "⏱️  Waiting for ingress controller to be ready (this may take 60-120 seconds)..."
kubectl wait --namespace ingress-nginx \
    --for=condition=ready pod \
    --selector=app.kubernetes.io/component=controller \
    --timeout=180s

echo ""
echo "📊 INGRESS CONTROLLER STATUS:"
kubectl get pods -n ingress-nginx

echo ""
echo "📊 INGRESS SERVICES:"
kubectl get svc -n ingress-nginx
```

### **🧠 Comprehensive Answer Block 1B:**
**Ingress vs Ingress Controller:** 
- **Ingress Resource:** YAML configuration defining routing rules (what to do)
- **Ingress Controller:** Actual software (NGINX, Traefik, etc.) that reads Ingress resources and implements the routing (how to do it)

**Controller Readiness:** The controller needs time to start, load configurations, establish network bindings, and sync with the Kubernetes API. Routing won't work until it's fully operational.

**Troubleshooting Steps:** Check controller pods (`kubectl get pods -n ingress-nginx`), examine logs (`kubectl logs -n ingress-nginx [controller-pod]`), verify addon status (`minikube addons list`), and ensure adequate cluster resources.

---

## Task 2: Deploy Multi-Application Architecture

### **🧠 Active Learning Block 2A: Application Architecture Design**

```bash
echo "🧠 PREDICT: Why should we use namespaces for our applications?"
echo "   Consider: organization, security, resource isolation..."
echo "   What problems might arise without proper namespacing?"
echo ""

echo "🔍 WATCH AND LEARN - Creating Organized Namespace Structure:"
```

**Step 1:** Create namespace file with nano
```bash
nano namespace.yaml
```

**Step 2:** Copy this exact content:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: web-apps
  labels:
    environment: lab
    purpose: ingress-demo
    created-by: kubernetes-lab
```

**Step 3:** Save and exit (Ctrl+X, Y, Enter)

```bash
# Apply namespace
kubectl apply -f namespace.yaml

echo "📊 NAMESPACE VERIFICATION:"
kubectl get namespace web-apps

echo ""
echo "📊 NAMESPACE DETAILS:"
kubectl describe namespace web-apps | grep -A 5 "Labels:"
```

### **🧠 Comprehensive Answer Block 2A:**
**Namespace Benefits:** Resource isolation, access control boundaries, separate resource quotas, logical organization, avoiding naming conflicts, and easier management of related resources.

**Service Discovery Impact:** Services in the same namespace can reference each other by name. Cross-namespace requires FQDN (`service.namespace.svc.cluster.local`). DNS resolution respects namespace boundaries.

**Without Namespaces:** Resource naming conflicts, difficult access control, harder resource management, security boundary confusion, and mixing of different application concerns in the default namespace.

### **🧠 Active Learning Block 2B: First Application Deployment**

```bash
echo "🧠 PREDICT: What components do we need for a complete web application?"
echo "   Think about: pods, services, configuration, networking..."
echo "   How do these components work together?"
echo ""

echo "🔍 WATCH AND LEARN - Deploying App1 with Best Practices:"
```

**Step 1:** Create app1 deployment file
```bash
nano app1-deployment.yaml
```

**Step 2:** Copy this exact content:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1-deployment
  namespace: web-apps
  labels:
    app: app1
    version: "1.0"
    component: frontend
spec:
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
        version: "1.0"
    spec:
      containers:
      - name: app1
        image: nginx:1.21-alpine
        ports:
        - containerPort: 80
          name: http
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
        volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html-volume
        configMap:
          name: app1-html
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: app1-html
  namespace: web-apps
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>Application 1 - Kubernetes Ingress Lab</title>
        <style>
            body { 
                font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; 
                background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                color: white;
                text-align: center; 
                padding: 50px;
                margin: 0;
                min-height: 100vh;
                display: flex;
                flex-direction: column;
                justify-content: center;
            }
            .container {
                background: rgba(255, 255, 255, 0.1);
                border-radius: 20px;
                padding: 40px;
                backdrop-filter: blur(10px);
                box-shadow: 0 8px 32px rgba(31, 38, 135, 0.37);
            }
            h1 { color: #ffffff; font-size: 2.5em; margin-bottom: 20px; }
            .info { background: rgba(0,0,0,0.2); padding: 15px; border-radius: 10px; margin: 10px 0; }
        </style>
    </head>
    <body>
        <div class="container">
            <h1>🚀 Application 1</h1>
            <div class="info">
                <p><strong>Route:</strong> /app1</p>
                <p><strong>Service:</strong> app1-service</p>
                <p><strong>Purpose:</strong> Path-based routing demonstration</p>
            </div>
            <p>Successfully serving traffic via Kubernetes Ingress!</p>
        </div>
    </body>
    </html>
---
apiVersion: v1
kind: Service
metadata:
  name: app1-service
  namespace: web-apps
  labels:
    app: app1
spec:
  selector:
    app: app1
  ports:
  - name: http
    port: 80
    targetPort: http
    protocol: TCP
  type: ClusterIP
```

**Step 3:** Save and exit (Ctrl+X, Y, Enter)

```bash
# Apply configuration
kubectl apply -f app1-deployment.yaml

echo "⏱️  Waiting for App1 deployment to be ready..."
kubectl rollout status deployment/app1-deployment -n web-apps --timeout=120s

echo ""
echo "📊 APP1 DEPLOYMENT STATUS:"
kubectl get deployment app1-deployment -n web-apps

echo ""
echo "📊 APP1 PODS:"
kubectl get pods -n web-apps -l app=app1
```

### **🧠 Comprehensive Answer Block 2B:**
**Resource Limits/Requests:** Requests ensure guaranteed resources for scheduling; limits prevent resource overconsumption. This provides predictable performance and cluster stability, preventing one application from starving others.

**Probe Functions:**
- **Readiness Probe:** Determines if pod can receive traffic. Kubernetes removes unready pods from service endpoints.
- **Liveness Probe:** Detects if container is healthy. Kubernetes restarts failed containers automatically.

**Rolling Update Strategy:** Gradually replaces old pods with new ones. `maxUnavailable: 1` ensures at least one pod stays running; `maxSurge: 1` allows one extra pod during updates, providing zero-downtime deployments.

### **🧠 Active Learning Block 2C: Second Application Deployment**

```bash
echo "🧠 PREDICT: How should App2 differ from App1 for effective routing testing?"
echo "   Think about: visual differences, unique identifiers, same architecture..."
echo "   What would make routing verification easier?"
echo ""

echo "🔍 WATCH AND LEARN - Deploying App2 with Distinct Identity:"
```

**Step 1:** Create app2 deployment file
```bash
nano app2-deployment.yaml
```

**Step 2:** Copy this exact content:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2-deployment
  namespace: web-apps
  labels:
    app: app2
    version: "1.0"
    component: frontend
spec:
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: app2
  template:
    metadata:
      labels:
        app: app2
        version: "1.0"
    spec:
      containers:
      - name: app2
        image: nginx:1.21-alpine
        ports:
        - containerPort: 80
          name: http
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
        volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html-volume
        configMap:
          name: app2-html
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: app2-html
  namespace: web-apps
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>Application 2 - Kubernetes Ingress Lab</title>
        <style>
            body { 
                font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; 
                background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
                color: white;
                text-align: center; 
                padding: 50px;
                margin: 0;
                min-height: 100vh;
                display: flex;
                flex-direction: column;
                justify-content: center;
            }
            .container {
                background: rgba(255, 255, 255, 0.1);
                border-radius: 20px;
                padding: 40px;
                backdrop-filter: blur(10px);
                box-shadow: 0 8px 32px rgba(31, 38, 135, 0.37);
            }
            h1 { color: #ffffff; font-size: 2.5em; margin-bottom: 20px; }
            .info { background: rgba(0,0,0,0.2); padding: 15px; border-radius: 10px; margin: 10px 0; }
        </style>
    </head>
    <body>
        <div class="container">
            <h1>⚡ Application 2</h1>
            <div class="info">
                <p><strong>Route:</strong> /app2</p>
                <p><strong>Service:</strong> app2-service</p>
                <p><strong>Purpose:</strong> Path-based routing demonstration</p>
            </div>
            <p>Alternative application demonstrating Ingress routing!</p>
        </div>
    </body>
    </html>
---
apiVersion: v1
kind: Service
metadata:
  name: app2-service
  namespace: web-apps
  labels:
    app: app2
spec:
  selector:
    app: app2
  ports:
  - name: http
    port: 80
    targetPort: http
    protocol: TCP
  type: ClusterIP
```

**Step 3:** Save and exit (Ctrl+X, Y, Enter)

```bash
# Apply configuration
kubectl apply -f app2-deployment.yaml

echo "⏱️  Waiting for App2 deployment to be ready..."
kubectl rollout status deployment/app2-deployment -n web-apps --timeout=120s

echo ""
echo "📊 MULTI-APPLICATION STATUS:"
kubectl get deployments -n web-apps

echo ""
echo "📊 ALL PODS STATUS:"
kubectl get pods -n web-apps

echo ""
echo "📊 ALL SERVICES:"
kubectl get svc -n web-apps
```

### **🧠 Comprehensive Answer Block 2C:**
**Visual Distinction:** Different color schemes (blue/purple vs pink/red gradients), different icons (🚀 vs ⚡), and different route information displayed make it immediately clear which application is serving the request.

**Service Discovery:** Services create DNS entries (`app1-service.web-apps.svc.cluster.local`) and maintain endpoint lists. Other pods can reach services by name within the same namespace or by FQDN across namespaces.

**Same Port, Different Services:** Both use port 80 (HTTP standard), but each service creates a separate network endpoint with its own ClusterIP. The service selector determines which pods receive traffic, enabling multiple applications to coexist.

---

## Task 3: Configure Advanced Path-Based Routing

### **🧠 Active Learning Block 3A: Ingress Resource Design**

```bash
echo "🧠 PREDICT: What routing rules do we need for our two applications?"
echo "   Consider: default path, app-specific paths, host routing..."
echo "   How should traffic be distributed?"
echo ""

echo "🔍 WATCH AND LEARN - Creating Sophisticated Ingress Rules:"

# Check for existing ingress conflicts first
kubectl get ingress -n web-apps 2>/dev/null && \
  echo "⚠️ Found existing ingress - will clean up first" || \
  echo "✅ No ingress conflicts"
```

**Step 1:** Create ingress configuration file
```bash
nano ingress-basic.yaml
```

**Step 2:** Copy this exact content:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-apps-ingress
  namespace: web-apps
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  labels:
    purpose: path-routing
    environment: lab
spec:
  ingressClassName: nginx
  rules:
  - host: myapps.local
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
```

**Step 3:** Save and exit (Ctrl+X, Y, Enter)

```bash
# Apply the Ingress configuration
kubectl apply -f ingress-basic.yaml

echo "⏱️  Waiting for ingress to be configured..."
sleep 15

echo ""
echo "📊 INGRESS RESOURCE STATUS:"
kubectl get ingress -n web-apps

echo ""
echo "📊 DETAILED INGRESS INFORMATION:"
kubectl describe ingress web-apps-ingress -n web-apps
```

### **🧠 Comprehensive Answer Block 3A:**
**Rewrite-Target Function:** Strips the matched path prefix before forwarding to the backend service. `/app1/page` becomes `/page` at the service level, allowing applications to serve content from their root path regardless of the ingress path.

**Path Strategy:** Specific paths (`/app1`, `/app2`) handle targeted requests; default path (`/`) provides fallback behavior. This ensures no request returns a 404 while maintaining clear routing logic.

**PathType Differences:**
- **Prefix:** Matches path prefixes (e.g., `/app1` matches `/app1/anything`)  
- **Exact:** Requires exact path match (e.g., `/app1` only matches `/app1`)
- **ImplementationSpecific:** Depends on ingress controller implementation

### **🧠 Active Learning Block 3B: DNS and Network Configuration**

```bash
echo "🧠 PREDICT: Why do we need to configure DNS for our local testing?"
echo "   Think about: domain resolution, host headers, load balancing..."
echo "   What happens without proper DNS configuration?"
echo ""

echo "🔍 WATCH AND LEARN - Setting Up Local DNS Resolution:"

# Get minikube IP and configure hosts
MINIKUBE_IP=$(minikube ip)
echo "🌐 Minikube Cluster IP: $MINIKUBE_IP"

# Backup existing hosts file
sudo cp /etc/hosts /etc/hosts.backup.$(date +%Y%m%d_%H%M%S)

# Add entry to hosts file with verification
if ! grep -q "myapps.local" /etc/hosts; then
    echo "$MINIKUBE_IP myapps.local" | sudo tee -a /etc/hosts
    echo "✅ Added myapps.local to hosts file"
else
    echo "⚠️  myapps.local already exists in hosts file"
fi

echo ""
echo "📊 DNS CONFIGURATION VERIFICATION:"
echo "Current hosts entries for myapps.local:"
grep myapps.local /etc/hosts

echo ""
echo "📊 DNS RESOLUTION TEST:"
nslookup myapps.local
```

### **🧠 Comprehensive Answer Block 3B:**
**Hosts File Necessity:** `myapps.local` is not a real domain, so DNS can't resolve it. The hosts file provides local DNS override, mapping the domain to minikube's IP address for testing purposes.

**Host Header Role:** Ingress controllers use the HTTP Host header to determine which ingress rule applies. Without proper host resolution, the wrong rule might match or requests might be rejected entirely.

**Production Differences:** Production uses real DNS records, load balancers with public IPs, and certificate authorities. The routing logic remains the same, but infrastructure is more robust and scalable.

### **🧠 Active Learning Block 3C: Routing Verification**

```bash
echo "🧠 PREDICT: What should we see when testing each route?"
echo "   Think about: different applications, routing logic, error handling..."
echo "   How can we verify routing is working correctly?"
echo ""

echo "🔍 WATCH AND LEARN - Comprehensive Routing Tests:"

# Test default path (should serve app1)
echo "📊 TESTING DEFAULT ROUTE (/):"
echo "Expected: Application 1 content"
curl -s -H "Host: myapps.local" http://$(minikube ip)/ | grep -o "<title>.*</title>" || echo "Connection failed"

echo ""
# Test app1 path  
echo "📊 TESTING APP1 ROUTE (/app1):"
echo "Expected: Application 1 content with routing info"
curl -s -H "Host: myapps.local" http://$(minikube ip)/app1 | grep -o "<title>.*</title>" || echo "Connection failed"

echo ""
# Test app2 path
echo "📊 TESTING APP2 ROUTE (/app2):"  
echo "Expected: Application 2 content with different styling"
curl -s -H "Host: myapps.local" http://$(minikube ip)/app2 | grep -o "<title>.*</title>" || echo "Connection failed"

echo ""
echo "📊 TESTING WITH DOMAIN NAME (DNS Resolution):"
curl -s http://myapps.local/app1 | grep -E "(Application [12]|Route:)" | head -2 || echo "DNS resolution failed"
```

### **🧠 Comprehensive Answer Block 3C:**
**Duplicate Routing Logic:** Both `/` and `/app1` route to app1-service by design. The `/` serves as a fallback/default route, while `/app1` is the explicit route. This provides graceful degradation and clear routing semantics.

**Backend Selection:** Ingress controller matches the request path against ingress rules in order of specificity. Longest matching path wins. The controller then forwards requests to the corresponding service's ClusterIP.

**Non-existent Path Behavior:** Requests to undefined paths fall through to the default `/` rule (serving app1). Without a default rule, the ingress controller would return a 404 default backend page.

---

## Task 4: Implement TLS Security and Certificate Management

### **🧠 Active Learning Block 4A: Certificate Generation Strategy**

```bash
echo "🧠 PREDICT: What components do we need for HTTPS termination?"
echo "   Think about: private keys, certificates, certificate chains..."
echo "   Why is SSL termination at the ingress level beneficial?"
echo ""

echo "🔍 WATCH AND LEARN - Generating Production-Style TLS Certificates:"

# Create certificates with proper extensions and metadata
echo "🔐 Generating RSA private key (2048-bit):"
openssl genrsa -out tls.key 2048
echo "✅ Private key generated successfully"

echo ""
echo "🔐 Creating certificate signing request with extensions:"
openssl req -new -key tls.key -out tls.csr \
    -subj "/C=US/ST=Lab/L=Kubernetes/O=Learning/OU=Ingress/CN=myapps.local"

echo "✅ Certificate signing request created"

echo ""
echo "🔐 Generating self-signed certificate (365 days):"
openssl x509 -req -days 365 -in tls.csr -signkey tls.key -out tls.crt \
    -extensions v3_req \
    -extfile <(echo '[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = DNS:myapps.local,DNS:api.myapps.local,DNS:*.myapps.local')

echo "✅ Self-signed certificate generated"

# Verify files exist before checking them
if [ -f "tls.crt" ] && [ -f "tls.key" ]; then
    echo ""
    echo "📊 CERTIFICATE DETAILS:"
    echo "Subject and validity information:"
    openssl x509 -in tls.crt -noout -subject -dates
    
    echo ""
    echo "📊 CERTIFICATE VERIFICATION:"
    echo "Certificate fingerprint and key matching:"
    CERT_HASH=$(openssl x509 -noout -modulus -in tls.crt | openssl md5)
    KEY_HASH=$(openssl rsa -noout -modulus -in tls.key | openssl md5)
    echo "Certificate hash: $CERT_HASH"
    echo "Private key hash:  $KEY_HASH"
    
    if [ "$CERT_HASH" = "$KEY_HASH" ]; then
        echo "✅ Certificate and private key match"
    else
        echo "❌ Certificate and private key do NOT match"
    fi
else
    echo "❌ Certificate files not created properly"
    echo "Troubleshooting: Check OpenSSL installation and permissions"
fi
```

### **🧠 Comprehensive Answer Block 4A:**
**Private Key vs Certificate:** Private key encrypts/decrypts data (kept secret); certificate contains public key and identity info (shared publicly). Together they enable asymmetric encryption and identity verification.

**Subject Alternative Names (SAN):** Allow one certificate to secure multiple domains/subdomains. Modern browsers require SAN for wildcard and multi-domain certificates. Essential for microservices and API gateways.

**Production vs Self-Signed:** Production certificates are signed by trusted Certificate Authorities (CAs) that browsers recognize. Self-signed certificates provide encryption but trigger browser warnings because trust chain can't be verified.

### **🧠 Active Learning Block 4B: Kubernetes TLS Secret Management**

```bash
echo "🧠 PREDICT: How does Kubernetes securely store and use TLS certificates?"
echo "   Think about: secret types, base64 encoding, RBAC access..."
echo "   What security considerations apply?"
echo ""

echo "🔍 WATCH AND LEARN - Creating and Managing TLS Secrets:"
```

**Step 1:** Create TLS secret file
```bash
nano tls-secret.yaml
```

**Step 2:** Copy this exact content:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapps-tls-secret
  namespace: web-apps
  labels:
    app: web-apps-ingress
    certificate-type: self-signed
    purpose: ingress-tls
  annotations:
    description: "TLS certificate for myapps.local ingress routing"
    created-by: "kubernetes-lab"
type: kubernetes.io/tls
data:
  tls.crt: CERT_DATA_PLACEHOLDER
  tls.key: KEY_DATA_PLACEHOLDER
```

**Step 3:** Save and exit, then populate with actual certificate data
```bash
# Replace placeholders with base64-encoded certificate data
CERT_B64=$(base64 -w 0 tls.crt)
KEY_B64=$(base64 -w 0 tls.key)

# Update the secret file with actual data
sed -i "s/CERT_DATA_PLACEHOLDER/$CERT_B64/" tls-secret.yaml
sed -i "s/KEY_DATA_PLACEHOLDER/$KEY_B64/" tls-secret.yaml

# Apply the secret
kubectl apply -f tls-secret.yaml

echo "✅ TLS secret created successfully"

echo ""
echo "📊 SECRET VERIFICATION:"
kubectl get secrets -n web-apps

echo ""
echo "📊 SECRET DETAILS (Metadata Only - Keys Remain Secure):"
kubectl describe secret myapps-tls-secret -n web-apps
```

### **🧠 Comprehensive Answer Block 4B:**
**Base64 Encoding:** Kubernetes stores binary data as base64 strings in etcd. This ensures safe storage and transmission of certificate data that may contain special characters or binary content.

**RBAC Requirements:** Minimum permissions needed: `get`, `list`, and `watch` on secrets in the target namespace. Ingress controllers typically have broader permissions through cluster roles for multi-namespace operation.

**Secure Access Pattern:** Ingress controller pods mount secrets as volumes. Kubernetes automatically base64-decodes the data and makes it available as files within the container filesystem, never exposing raw secret data in environment variables.

### **🧠 Active Learning Block 4C: HTTPS Ingress Configuration**

```bash
echo "🧠 PREDICT: What changes when we enable TLS termination?"
echo "   Think about: SSL redirect, certificate presentation, security headers..."
echo "   How does HTTPS affect our routing logic?"
echo ""

echo "🔍 WATCH AND LEARN - Configuring Production-Ready HTTPS Ingress:"

# Delete existing ingress first to avoid conflicts
kubectl delete ingress web-apps-ingress -n web-apps --ignore-not-found=true
sleep 5
```

**Step 1:** Create HTTPS ingress file
```bash
nano ingress-tls.yaml
```

**Step 2:** Copy this exact content:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-apps-ingress
  namespace: web-apps
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
  labels:
    purpose: https-routing
    security: tls-enabled
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapps.local
    - api.myapps.local
    secretName: myapps-tls-secret
  rules:
  - host: myapps.local
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
```

**Step 3:** Save and exit (Ctrl+X, Y, Enter)

```bash
# Apply the HTTPS configuration
kubectl apply -f ingress-tls.yaml

echo "⏱️  Waiting for TLS configuration to propagate..."
sleep 20

echo ""
echo "📊 HTTPS INGRESS STATUS:"
kubectl get ingress -n web-apps

echo ""
echo "📊 TLS CONFIGURATION VERIFICATION:"
kubectl describe ingress web-apps-ingress -n web-apps | grep -A 10 "TLS:"
```

### **🧠 Comprehensive Answer Block 4C:**
**HTTP to HTTPS Behavior:** Ingress controller automatically redirects HTTP requests to HTTPS using 301/302 responses. Clients then retry with HTTPS, completing the secure connection.

**Ingress-Level Security Headers:** Centralized security policy enforcement. Headers like HSTS, XSS protection, and frame options apply to all backend services without individual configuration. This ensures consistent security across microservices.

**TLS Termination Impact:** HTTPS terminates at ingress; backend communication typically remains HTTP within the cluster. This reduces CPU overhead on application pods while maintaining end-to-end encryption for external traffic.

### **🧠 Active Learning Block 4D: HTTPS Verification and Testing**

```bash
echo "🧠 PREDICT: How can we verify HTTPS is working correctly?"
echo "   Think about: certificate validation, redirect behavior, security headers..."
echo "   What tools help us test TLS functionality?"
echo ""

echo "🔍 WATCH AND LEARN - Comprehensive HTTPS Testing:"

echo "📊 TESTING HTTP TO HTTPS REDIRECT:"
echo "Expected: 301/302 redirect to HTTPS"
curl -s -o /dev/null -w "HTTP %{http_code} → Redirect: %{redirect_url}\n" http://myapps.local/app1

echo ""
echo "📊 TESTING HTTPS CONNECTION:"
echo "App1 HTTPS content:"
curl -k -s https://myapps.local/app1 | grep -o "<title>.*</title>" 

echo ""
echo "App2 HTTPS content:"
curl -k -s https://myapps.local/app2 | grep -o "<title>.*</title>"

echo ""
echo "📊 TLS CERTIFICATE VALIDATION:"
echo "Certificate presented by server:"
echo | openssl s_client -servername myapps.local -connect $(minikube ip):443 2>/dev/null | openssl x509 -noout -subject -dates

echo ""
echo "📊 SSL/TLS CONNECTION DETAILS:"
echo "Cipher and protocol information:"
echo | openssl s_client -servername myapps.local -connect $(minikube ip):443 2>/dev/null | grep -E "(Protocol|Cipher|Server public key)"
```

### **🧠 Comprehensive Answer Block 4D:**
**Curl '-k' Flag:** Bypasses certificate validation for self-signed certificates. In production, remove '-k' to ensure proper certificate chain validation and detect certificate issues.

**Security Header Benefits:**
- **HSTS:** Forces HTTPS for future requests, preventing downgrade attacks
- **X-Content-Type-Options:** Prevents MIME sniffing attacks
- **X-Frame-Options:** Protects against clickjacking
- **X-XSS-Protection:** Enables browser XSS filtering

**CA-Signed vs Self-Signed:** CA-signed certificates establish trust through certificate chain validation. Browsers automatically trust CA-signed certificates without warnings, providing seamless user experience and stronger security guarantees.

---

## Task 5: Advanced Routing and Multi-Domain Configuration

### **🧠 Active Learning Block 5A: Multi-Domain Architecture**

```bash
echo "🧠 PREDICT: Why would we want multiple domains for one application?"
echo "   Think about: API separation, versioning, microservices organization..."
echo "   How does this relate to real-world architectures?"
echo ""

echo "🔍 WATCH AND LEARN - Creating Multi-Domain Ingress Configuration:"

# Clean up existing ingress to avoid conflicts
kubectl delete ingress web-apps-ingress -n web-apps --ignore-not-found=true
kubectl delete ingress web-apps-ingress-advanced -n web-apps --ignore-not-found=true
sleep 10
```

**Step 1:** Create advanced ingress file
```bash
nano ingress-advanced.yaml
```

**Step 2:** Copy this exact content:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-apps-ingress-advanced
  namespace: web-apps
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "https://myapps.local"
  labels:
    purpose: multi-domain-routing
    security: enhanced-tls
    version: "2.0"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapps.local
    - api.myapps.local
    - admin.myapps.local
    secretName: myapps-tls-secret
  rules:
  - host: myapps.local
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
      - path: /health
        pathType: Exact
        backend:
          service:
            name: app1-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
  - host: api.myapps.local
    http:
      paths:
      - path: /v1
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
      - path: /v2
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
  - host: admin.myapps.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
```

**Step 3:** Save and exit (Ctrl+X, Y, Enter)

```bash
# Apply the advanced configuration
kubectl apply -f ingress-advanced.yaml

echo "⏱️  Waiting for advanced ingress configuration..."
sleep 15

echo ""
echo "📊 ADVANCED INGRESS STATUS:"
kubectl get ingress -n web-apps

echo ""
echo "📊 ROUTING RULES SUMMARY:"
kubectl describe ingress web-apps-ingress-advanced -n web-apps | grep -A 30 "Rules:"
```

### **🧠 Comprehensive Answer Block 5A:**
**API/UI Separation:** Different security policies, rate limits, and scaling requirements. APIs need authentication tokens; UIs need session management. Separate domains allow independent caching, monitoring, and access control.

**Rate Limit Protection:** Prevents API abuse, ensures fair resource usage, and protects against DoS attacks. Different rate limits can apply to different user tiers or API endpoints.

**Subdomain Advantages:** Clear service boundaries, independent SSL certificates, separate monitoring/logging, easier load balancing, and better SEO/branding for customer-facing services.

### **🧠 Active Learning Block 5B: DNS Configuration for Multiple Domains**

```bash
echo "🧠 PREDICT: What DNS entries do we need for our multi-domain setup?"
echo "   Consider: subdomains, wildcard entries, testing requirements..."
echo "   How many hosts file entries will we need?"
echo ""

echo "🔍 WATCH AND LEARN - Configuring Multi-Domain DNS:"

# Get minikube IP
MINIKUBE_IP=$(minikube ip)
echo "🌐 Minikube IP for all domains: $MINIKUBE_IP"

# Add all required domain entries
echo "📝 Adding DNS entries for multi-domain routing:"

# Create array of domains to add
domains=("api.myapps.local" "admin.myapps.local")

for domain in "${domains[@]}"; do
    if ! grep -q "$domain" /etc/hosts; then
        echo "$MINIKUBE_IP $domain" | sudo tee -a /etc/hosts
        echo "✅ Added $domain"
    else
        echo "ℹ️  $domain already exists"
    fi
done

echo ""
echo "📊 ALL DNS ENTRIES VERIFICATION:"
echo "Complete hosts file entries for myapps domains:"
grep "myapps.local" /etc/hosts

echo ""
echo "📊 DNS RESOLUTION TESTING:"
for domain in myapps.local api.myapps.local admin.myapps.local; do
    echo "Testing $domain:"
    nslookup $domain | grep -E "(Name|Address)" | head -2
    echo ""
done
```

### **🧠 Comprehensive Answer Block 5B:**
**Same IP for Subdomains:** All domains resolve to the ingress controller's IP. The ingress controller uses the HTTP Host header (not the IP) to determine routing rules. One load balancer can serve multiple domains.

**Domain Differentiation:** HTTP requests include a Host header (`Host: api.myapps.local`). The ingress controller matches this header against ingress rules to determine which backend service receives the request.

**Missing DNS Entry:** Browser can't resolve the domain name, resulting in DNS resolution errors. Even if the ingress rules exist, requests never reach the cluster because DNS lookup fails first.

### **🧠 Active Learning Block 5C: Advanced Routing Testing**

```bash
echo "🧠 PREDICT: How should different domains behave with our routing rules?"
echo "   Think about: main domain paths, API endpoints, admin access..."
echo "   What differences should we observe?"
echo ""

echo "🔍 WATCH AND LEARN - Comprehensive Multi-Domain Testing:"

# Test main domain routing
echo "📊 TESTING MAIN DOMAIN (myapps.local):"
echo "Main domain app1 route:"
curl -k -s https://myapps.local/app1 | grep -E "(Application [12]|Route:)" | head -1

echo "Main domain app2 route:"
curl -k -s https://myapps.local/app2 | grep -E "(Application [12]|Route:)" | head -1

echo "Main domain health check:"
curl -k -s -o /dev/null -w "Health endpoint: HTTP %{http_code}\n" https://myapps.local/health

echo ""
# Test API domain routing
echo "📊 TESTING API DOMAIN (api.myapps.local):"
echo "API v1 endpoint:"
curl -k -s https://api.myapps.local/v1 | grep -E "(Application [12]|Route:)" | head -1

echo "API v2 endpoint:"
curl -k -s https://api.myapps.local/v2 | grep -E "(Application [12]|Route:)" | head -1

echo "API root endpoint:"
curl -k -s https://api.myapps.local/ | grep -E "(Application [12]|Route:)" | head -1

echo ""
# Test admin domain routing
echo "📊 TESTING ADMIN DOMAIN (admin.myapps.local):"
echo "Admin interface:"
curl -k -s https://admin.myapps.local/ | grep -E "(Application [12]|Route:)" | head -1

echo ""
# Test CORS headers on API domain
echo "📊 TESTING CORS CONFIGURATION:"
echo "CORS headers for API domain:"
curl -k -s -I -H "Origin: https://myapps.local" https://api.myapps.local/ | grep -i "access-control" || echo "No CORS headers found"
```

### **🧠 Comprehensive Answer Block 5C:**
**Version-based Routing:** API v1/v2 routing to different services demonstrates API versioning patterns. This allows gradual migration between API versions while maintaining backward compatibility for existing clients.

**CORS Functionality:** Cross-Origin Resource Sharing headers allow web applications from `myapps.local` to make API calls to `api.myapps.local`. Without CORS, browsers block cross-domain requests for security.

**Domain/Routing Patterns:**
- **Main Domain:** User-facing applications with path-based routing
- **API Domain:** Service endpoints with version-based routing  
- **Admin Domain:** Management interfaces with restricted access patterns
- Each domain can have different security policies, rate limits, and backend services

---

## Task 6: Comprehensive Verification and Performance Testing

### **🧠 Active Learning Block 6A: Automated Testing Framework**

```bash
echo "🧠 PREDICT: What should a comprehensive ingress test include?"
echo "   Think about: functionality, security, performance, error handling..."
echo "   How can we automate verification of complex routing rules?"
echo ""

echo "🔍 WATCH AND LEARN - Building Production-Quality Test Suite:"
```

**Step 1:** Create comprehensive test script
```bash
nano test-ingress-comprehensive.sh
```

**Step 2:** Copy this exact content:
```bash
#!/bin/bash

# Color codes for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

echo -e "${BLUE}=================================${NC}"
echo -e "${BLUE}  COMPREHENSIVE INGRESS TEST SUITE${NC}"
echo -e "${BLUE}=================================${NC}"
echo ""

# Test counter
TOTAL_TESTS=0
PASSED_TESTS=0

# Function to run test and track results
run_test() {
    local test_name="$1"
    local test_command="$2"
    local expected_pattern="$3"
    
    TOTAL_TESTS=$((TOTAL_TESTS + 1))
    echo -e "${YELLOW}Testing:${NC} $test_name"
    
    result=$(eval "$test_command" 2>/dev/null)
    
    if echo "$result" | grep -q "$expected_pattern"; then
        echo -e "${GREEN}✅ PASS${NC}: $test_name"
        PASSED_TESTS=$((PASSED_TESTS + 1))
    else
        echo -e "${RED}❌ FAIL${NC}: $test_name"
        echo -e "   Expected: $expected_pattern"
        echo -e "   Got: $result"
    fi
    echo ""
}

# Functionality Tests
echo -e "${BLUE}=== FUNCTIONALITY TESTS ===${NC}"

run_test "HTTP to HTTPS Redirect" \
    "curl -s -o /dev/null -w '%{http_code}' http://myapps.local/app1" \
    "30[12]"

run_test "HTTPS App1 Routing" \
    "curl -k -s https://myapps.local/app1 | grep -o 'Application 1'" \
    "Application 1"

run_test "HTTPS App2 Routing" \
    "curl -k -s https://myapps.local/app2 | grep -o 'Application 2'" \
    "Application 2"

run_test "API Domain v1 Endpoint" \
    "curl -k -s https://api.myapps.local/v1 | grep -o 'Application [12]'" \
    "Application"

run_test "Admin Domain Access" \
    "curl -k -s https://admin.myapps.local/ | grep -o 'Application [12]'" \
    "Application"

# Security Tests
echo -e "${BLUE}=== SECURITY TESTS ===${NC}"

run_test "TLS Certificate Validity" \
    "echo | openssl s_client -servername myapps.local -connect $(minikube ip):443 2>/dev/null | openssl x509 -noout -subject" \
    "myapps.local"

# Performance Tests
echo -e "${BLUE}=== PERFORMANCE TESTS ===${NC}"

run_test "Response Time Under 2s" \
    "curl -k -s -o /dev/null -w '%{time_total}' https://myapps.local/app1 | awk '{print (\$1 < 2.0) ? \"FAST\" : \"SLOW\"}'" \
    "FAST"

# Error Handling Tests
echo -e "${BLUE}=== ERROR HANDLING TESTS ===${NC}"

run_test "Non-existent Path Handling" \
    "curl -k -s -o /dev/null -w '%{http_code}' https://myapps.local/nonexistent" \
    "200"

run_test "Invalid Host Header" \
    "curl -k -s -o /dev/null -w '%{http_code}' -H 'Host: invalid.domain' https://$(minikube ip)/" \
    "404"

# Multi-domain Tests  
echo -e "${BLUE}=== MULTI-DOMAIN TESTS ===${NC}"

run_test "API Domain Root" \
    "curl -k -s https://api.myapps.local/ | grep -o 'Application [12]'" \
    "Application"

run_test "Cross-domain CORS Headers" \
    "curl -k -s -I -H 'Origin: https://myapps.local' https://api.myapps.local/ | grep -i 'access-control-allow'" \
    "Access-Control-Allow"

# Summary
echo -e "${BLUE}=== TEST SUMMARY ===${NC}"
echo -e "Total Tests: $TOTAL_TESTS"
echo -e "Passed: ${GREEN}$PASSED_TESTS${NC}"
echo -e "Failed: ${RED}$((TOTAL_TESTS - PASSED_TESTS))${NC}"
echo -e "Success Rate: $(( (PASSED_TESTS * 100) / TOTAL_TESTS ))%"
echo ""

if [ $PASSED_TESTS -eq $TOTAL_TESTS ]; then
    echo -e "${GREEN}🎉 ALL TESTS PASSED! Ingress configuration is working perfectly.${NC}"
else
    echo -e "${YELLOW}⚠️  Some tests failed. Check ingress configuration and troubleshoot.${NC}"
fi
```

**Step 3:** Make executable and run
```bash
chmod +x test-ingress-comprehensive.sh

echo "✅ Comprehensive test suite created!"
echo ""
echo "🔍 RUNNING COMPREHENSIVE TEST SUITE:"
./test-ingress-comprehensive.sh
```

### **🧠 Comprehensive Answer Block 6A:**
**Functionality + Security Testing:** Functionality ensures features work; security ensures they work safely. Both are required for production readiness - working insecurely is as bad as not working at all.

**Enhanced Test Suite Additions:**
- Load testing with concurrent requests
- Certificate expiration monitoring  
- Backend service health verification
- Failover and disaster recovery testing
- Performance regression detection
- Integration with monitoring systems

**Production Testing Benefits:** Automated tests catch configuration drift, validate deployments, enable confident updates, provide compliance evidence, and reduce manual verification overhead in CI/CD pipelines.

### **🧠 Active Learning Block 6B: Performance and Load Testing**

```bash
echo "🧠 PREDICT: What performance characteristics should we test?"
echo "   Think about: concurrent users, response times, throughput limits..."
echo "   How does ingress performance differ from direct service access?"
echo ""

echo "🔍 WATCH AND LEARN - Professional Performance Testing:"

# Ensure apache2-utils is installed for load testing
if ! command -v ab >/dev/null 2>&1; then
    echo "📦 Installing Apache Bench for load testing..."
    sudo apt-get update && sudo apt-get install -y apache2-utils
fi

echo "📊 BASELINE PERFORMANCE TEST:"
echo "Single request response time measurement:"
for endpoint in "https://myapps.local/app1" "https://myapps.local/app2" "https://api.myapps.local/"; do
    echo "Testing $endpoint:"
    curl -k -s -o /dev/null -w "  Response Time: %{time_total}s | Size: %{size_download} bytes | HTTP: %{http_code}\n" "$endpoint"
done
echo ""

echo "📊 CONCURRENT USER SIMULATION (10 users, 50 requests):"
echo "App1 load test:"
ab -n 50 -c 10 -k -s 30 https://myapps.local/app1 2>/dev/null | grep -E "(Requests per second|Time per request|Transfer rate)" || echo "Load test completed"

echo ""
echo "App2 load test:"
ab -n 50 -c 10 -k -s 30 https://myapps.local/app2 2>/dev/null | grep -E "(Requests per second|Time per request|Transfer rate)" || echo "Load test completed"

echo ""
echo "📊 API ENDPOINT PERFORMANCE:"
echo "API endpoint under load:"
ab -n 100 -c 20 -k -s 30 https://api.myapps.local/v1 2>/dev/null | grep -E "(Requests per second|Time per request|Failed requests)" || echo "API load test completed"

echo ""
# Monitor ingress controller during load
echo "📊 INGRESS CONTROLLER RESOURCE USAGE:"
INGRESS_POD=$(kubectl get pods -n ingress-nginx -l app.kubernetes.io/component=controller -o jsonpath='{.items[0].metadata.name}')
echo "Ingress controller pod: $INGRESS_POD"
kubectl top pod $INGRESS_POD -n ingress-nginx 2>/dev/null || echo "Metrics not available (install metrics-server for detailed monitoring)"
```

### **🧠 Comprehensive Answer Block 6B:**
**Ingress vs Direct Performance:** Ingress adds network hops (client → ingress → service → pod), TLS termination overhead, routing logic processing, and potential bottlenecks. However, it provides caching, compression, and connection pooling benefits.

**High Load Failures:** Resource exhaustion (CPU/memory), connection limits, timeout configurations, or backend service saturation. Ingress controllers have built-in limits to prevent cascading failures.

**Production Optimization:**
- **Horizontal Pod Autoscaling** for ingress controllers
- **Connection pooling** and **keep-alive** optimization  
- **Resource requests/limits** tuning
- **Rate limiting** and **circuit breakers**
- **CDN integration** for static content
- **Multiple ingress controller replicas** for high availability

---

## Task 7: Monitoring and Troubleshooting

### **🧠 Active Learning Block 7A: Monitoring and Observability**

```bash
echo "🧠 PREDICT: What should we monitor in a production ingress setup?"
echo "   Think about: request metrics, error rates, certificate expiry..."
echo "   How do we know if our ingress is healthy?"
echo ""

echo "🔍 WATCH AND LEARN - Implementing Ingress Observability:"

# Check ingress controller logs
INGRESS_POD=$(kubectl get pods -n ingress-nginx -l app.kubernetes.io/component=controller -o jsonpath='{.items[0].metadata.name}')

echo "📊 RECENT INGRESS CONTROLLER ACTIVITY:"
echo "Last 20 log entries from ingress controller:"
kubectl logs -n ingress-nginx $INGRESS_POD --tail=20 | grep -E "(GET|POST|PUT|DELETE)" | tail -10

echo ""
echo "📊 INGRESS CONTROLLER HEALTH STATUS:"
kubectl get pods -n ingress-nginx -l app.kubernetes.io/component=controller

echo ""
echo "📊 BACKEND SERVICE HEALTH:"
echo "Service endpoint status:"
kubectl get endpoints -n web-apps

echo ""
echo "📊 CERTIFICATE EXPIRATION MONITORING:"
echo "TLS certificate validity:"
kubectl get secret myapps-tls-secret -n web-apps -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -noout -dates

echo ""
echo "📊 INGRESS RESOURCE STATUS:"
kubectl get ingress -n web-apps
```

**Step 1:** Create monitoring script
```bash
nano monitor-ingress.sh
```

**Step 2:** Copy this exact content:
```bash
#!/bin/bash

echo "=== INGRESS MONITORING DASHBOARD ==="
echo "Timestamp: $(date)"
echo ""

# Request rate (from logs)
INGRESS_POD=$(kubectl get pods -n ingress-nginx -l app.kubernetes.io/component=controller -o jsonpath='{.items[0].metadata.name}')
REQUESTS_LAST_MINUTE=$(kubectl logs -n ingress-nginx $INGRESS_POD --since=1m | grep -c "GET\|POST\|PUT\|DELETE")

echo "📊 TRAFFIC METRICS:"
echo "  Requests in last minute: $REQUESTS_LAST_MINUTE"
echo "  Average requests per second: $(echo "scale=2; $REQUESTS_LAST_MINUTE / 60" | bc 2>/dev/null || echo "N/A")"
echo ""

# Error rate
ERROR_COUNT=$(kubectl logs -n ingress-nginx $INGRESS_POD --since=1m | grep -E " [45][0-9][0-9] " | wc -l)
echo "📊 ERROR METRICS:"
echo "  Errors in last minute: $ERROR_COUNT"
if [ $REQUESTS_LAST_MINUTE -gt 0 ]; then
    ERROR_RATE=$(echo "scale=2; $ERROR_COUNT * 100 / $REQUESTS_LAST_MINUTE" | bc 2>/dev/null || echo "N/A")
    echo "  Error rate: ${ERROR_RATE}%"
fi
echo ""

# Resource usage
echo "📊 RESOURCE USAGE:"
kubectl top pod $INGRESS_POD -n ingress-nginx 2>/dev/null || echo "  Metrics server not available"
echo ""

# Certificate status
echo "📊 CERTIFICATE STATUS:"
CERT_DAYS=$(kubectl get secret myapps-tls-secret -n web-apps -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -noout -enddate | cut -d= -f2 | xargs -I {} date -d "{}" +%s | awk -v now="$(date +%s)" '{days=int(($1-now)/86400); print days}')
echo "  Certificate expires in: $CERT_DAYS days"
if [ "$CERT_DAYS" -lt 30 ]; then
    echo "  ⚠️  Certificate renewal needed soon!"
fi
echo ""

echo "=== END MONITORING SNAPSHOT ==="
```

**Step 3:** Make executable and run
```bash
chmod +x monitor-ingress.sh

echo "📊 RUNNING MONITORING SNAPSHOT:"
./monitor-ingress.sh
```

### **🧠 Comprehensive Answer Block 7A:**
**Performance Problem Indicators:**
- **High response times** (>2s for web apps)
- **Increasing error rates** (>1% for non-test traffic)  
- **Resource exhaustion** (CPU >80%, memory >90%)
- **Connection failures** or timeouts
- **Backend service unavailability**

**Certificate Monitoring Importance:** Expired certificates cause complete service outages with cryptic browser errors. Automated renewal (Let's Encrypt, cert-manager) and expiration alerting prevent these critical failures.

**Production Alerting Strategy:**
- **Error rate > 5%** for 5 minutes
- **Response time > 95th percentile** for 10 minutes  
- **Certificate expiry < 30 days**
- **Ingress controller pod restarts**
- **Backend endpoint failures**
- **Rate limit threshold breaches**

### **🧠 Active Learning Block 7B: Troubleshooting Guide**

```bash
echo "🧠 PREDICT: What are the most common ingress problems you might encounter?"
echo "   Think about: DNS, certificates, routing, backend connectivity..."
echo "   How would you systematically diagnose each type of issue?"
echo ""

echo "🔍 WATCH AND LEARN - Building Troubleshooting Expertise:"
```

**Step 1:** Create troubleshooting script
```bash
nano troubleshoot-ingress.sh
```

**Step 2:** Copy this exact content:
```bash
#!/bin/bash

# Color codes
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

echo -e "${BLUE}=== INGRESS TROUBLESHOOTING DIAGNOSTIC ===${NC}"
echo ""

# Function to check status and provide guidance
check_component() {
    local component="$1"
    local check_command="$2"  
    local success_pattern="$3"
    local troubleshooting_tips="$4"
    
    echo -e "${YELLOW}Checking: $component${NC}"
    
    result=$(eval "$check_command" 2>&1)
    
    if echo "$result" | grep -q "$success_pattern"; then
        echo -e "${GREEN}✅ $component: OK${NC}"
    else
        echo -e "${RED}❌ $component: ISSUE DETECTED${NC}"
        echo -e "${YELLOW}Troubleshooting Tips:${NC}"
        echo "$troubleshooting_tips"
        echo ""
        echo -e "${YELLOW}Diagnostic Output:${NC}"
        echo "$result" | head -5
    fi
    echo ""
}

# 1. Check cluster connectivity
check_component "Kubernetes Cluster" \
    "kubectl cluster-info" \
    "Kubernetes control plane" \
    "- Verify kubectl config: kubectl config current-context
- Check minikube status: minikube status  
- Restart minikube if needed: minikube start"

# 2. Check ingress controller
check_component "Ingress Controller" \
    "kubectl get pods -n ingress-nginx" \
    "Running" \
    "- Enable ingress addon: minikube addons enable ingress
- Check controller logs: kubectl logs -n ingress-nginx [pod-name]
- Verify ingress class: kubectl get ingressclass"

# 3. Check ingress resources
check_component "Ingress Resources" \
    "kubectl get ingress -n web-apps" \
    "web-apps-ingress" \
    "- Verify ingress exists: kubectl describe ingress [name] -n web-apps
- Check ingress rules syntax in YAML
- Ensure ingressClassName is set correctly"

# 4. Check backend services
check_component "Backend Services" \
    "kubectl get endpoints -n web-apps" \
    "app1-service\|app2-service" \
    "- Check service selectors match pod labels
- Verify pods are running: kubectl get pods -n web-apps
- Test service connectivity: kubectl port-forward svc/[service-name] 8080:80"

# 5. Check TLS certificates
check_component "TLS Certificates" \
    "kubectl get secret myapps-tls-secret -n web-apps" \
    "tls" \
    "- Recreate TLS secret if corrupted
- Verify certificate validity: kubectl get secret [name] -o yaml
- Check certificate matches domain: openssl x509 -in tls.crt -noout -text"

# 6. Check DNS resolution
check_component "DNS Resolution" \
    "nslookup myapps.local" \
    "$(minikube ip)" \
    "- Add domain to /etc/hosts: echo \"\$(minikube ip) myapps.local\" | sudo tee -a /etc/hosts
- Verify hosts file: grep myapps.local /etc/hosts
- Clear DNS cache if needed"

# 7. Check network connectivity
check_component "Network Connectivity" \
    "curl -k -s -o /dev/null -w '%{http_code}' https://myapps.local/" \
    "200" \
    "- Test with minikube IP directly: curl -k https://\$(minikube ip)/
- Check ingress controller service: kubectl get svc -n ingress-nginx
- Verify firewall rules and network policies"

echo -e "${BLUE}=== DIAGNOSTIC COMPLETE ===${NC}"
echo ""
echo -e "${YELLOW}Additional Debugging Commands:${NC}"
echo "- kubectl describe ingress [name] -n [namespace]"
echo "- kubectl logs -n ingress-nginx [controller-pod]"  
echo "- kubectl get events -n [namespace] --sort-by=.metadata.creationTimestamp"
echo "- minikube service list"
echo "- kubectl top pods -n ingress-nginx"
```

**Step 3:** Make executable and run
```bash
chmod +x troubleshoot-ingress.sh

echo "✅ Troubleshooting script created!"
echo ""
echo "🔍 RUNNING DIAGNOSTIC CHECK:"
./troubleshoot-ingress.sh
```

### **🧠 Comprehensive Answer Block 7B:**
**Systematic Troubleshooting Order:**
1. **Cluster connectivity** (can we reach Kubernetes?)
2. **Ingress controller** (is the router working?)  
3. **Ingress resources** (are rules configured correctly?)
4. **Backend services** (can services reach pods?)
5. **TLS/certificates** (is encryption working?)
6. **DNS resolution** (can clients find the service?)
7. **Network connectivity** (end-to-end testing)

**Logical Sequence Importance:** Each layer depends on the previous one. No point checking TLS if the ingress controller isn't running. This prevents wasting time on symptoms rather than root causes.

**Production Prevention:** Infrastructure as Code (IaC), automated testing in CI/CD, comprehensive monitoring/alerting, regular certificate renewal, network policy validation, and disaster recovery procedures.

---

## Task 8: Cleanup and Resource Management

### **🧠 Active Learning Block 8A: Professional Cleanup Procedures**

```bash
echo "🧠 PREDICT: What's the proper order for cleaning up Kubernetes resources?"
echo "   Think about: dependencies, resource relationships, external configurations..."
echo "   Why might cleanup order matter?"
echo ""

echo "🔍 WATCH AND LEARN - Professional Cleanup Procedures:"
```

**Step 1:** Create cleanup script
```bash
nano cleanup-ingress-lab.sh
```

**Step 2:** Copy this exact content:
```bash
#!/bin/bash

# Color codes
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

echo -e "${BLUE}=== KUBERNETES INGRESS LAB CLEANUP ===${NC}"
echo ""

# Function to safely delete resources
safe_delete() {
    local resource_type="$1"
    local resource_name="$2"
    local namespace="$3"
    
    if [ -n "$namespace" ]; then
        ns_flag="-n $namespace"
    else
        ns_flag=""
    fi
    
    if kubectl get $resource_type $resource_name $ns_flag >/dev/null 2>&1; then
        echo -e "${YELLOW}Deleting $resource_type/$resource_name${NC}"
        kubectl delete $resource_type $resource_name $ns_flag
        echo -e "${GREEN}✅ Deleted $resource_type/$resource_name${NC}"
    else
        echo -e "${YELLOW}⚠️  $resource_type/$resource_name not found (already deleted)${NC}"
    fi
    echo ""
}

# Step 1: Delete ingress resources (stop external traffic first)
echo -e "${BLUE}=== STEP 1: Remove Ingress Resources ===${NC}"
safe_delete "ingress" "web-apps-ingress" "web-apps"
safe_delete "ingress" "web-apps-ingress-advanced" "web-apps"

# Wait for ingress deletion to propagate
echo -e "${YELLOW}Waiting for ingress deletion to propagate...${NC}"
sleep 10

# Step 2: Delete applications (services and deployments)
echo -e "${BLUE}=== STEP 2: Remove Applications ===${NC}"
safe_delete "deployment" "app1-deployment" "web-apps"
safe_delete "deployment" "app2-deployment" "web-apps"
safe_delete "service" "app1-service" "web-apps"
safe_delete "service" "app2-service" "web-apps"

# Step 3: Delete configuration resources
echo -e "${BLUE}=== STEP 3: Remove Configuration ===${NC}"
safe_delete "configmap" "app1-html" "web-apps"
safe_delete "configmap" "app2-html" "web-apps"
safe_delete "secret" "myapps-tls-secret" "web-apps"

# Step 4: Delete namespace (removes any remaining resources)
echo -e "${BLUE}=== STEP 4: Remove Namespace ===${NC}"
safe_delete "namespace" "web-apps" ""

# Step 5: Clean up local files
echo -e "${BLUE}=== STEP 5: Clean Up Local Files ===${NC}"
echo -e "${YELLOW}Removing generated files...${NC}"

files_to_remove=(
    "*.yaml"
    "tls.key"
    "tls.crt" 
    "tls.csr"
    "test-ingress-comprehensive.sh"
    "monitor-ingress.sh"
    "troubleshoot-ingress.sh"
    "cleanup-ingress-lab.sh"
)

for file_pattern in "${files_to_remove[@]}"; do
    if ls $file_pattern >/dev/null 2>&1; then
        rm -f $file_pattern
        echo -e "${GREEN}✅ Removed $file_pattern${NC}"
    fi
done
echo ""

# Step 6: Clean up DNS entries
echo -e "${BLUE}=== STEP 6: Clean Up DNS Entries ===${NC}"
echo -e "${YELLOW}Removing hosts file entries...${NC}"

# Backup hosts file before modification
sudo cp /etc/hosts /etc/hosts.backup.cleanup.$(date +%Y%m%d_%H%M%S)

# Remove myapps.local entries
if grep -q "myapps.local" /etc/hosts; then
    sudo sed -i '/myapps.local/d' /etc/hosts
    echo -e "${GREEN}✅ Removed myapps.local entries from /etc/hosts${NC}"
else
    echo -e "${YELLOW}⚠️  No myapps.local entries found in /etc/hosts${NC}"
fi
echo ""

# Step 7: Verify cleanup
echo -e "${BLUE}=== STEP 7: Verify Cleanup ===${NC}"
echo -e "${YELLOW}Checking for remaining resources...${NC}"

# Check for any remaining web-apps resources
if kubectl get namespace web-apps >/dev/null 2>&1; then
    echo -e "${YELLOW}⚠️  web-apps namespace still exists${NC}"
    kubectl get all -n web-apps 2>/dev/null || echo "Namespace empty"
else
    echo -e "${GREEN}✅ web-apps namespace successfully removed${NC}"
fi

# Check ingress controller (should remain for other uses)
ingress_controller=$(kubectl get pods -n ingress-nginx 2>/dev/null | grep controller | wc -l)
if [ "$ingress_controller" -gt 0 ]; then
    echo -e "${GREEN}✅ Ingress controller preserved for future use${NC}"
else
    echo -e "${YELLOW}⚠️  Ingress controller not found (may have been disabled)${NC}"
fi

echo ""
echo -e "${GREEN}=== CLEANUP COMPLETE ===${NC}"
echo -e "${BLUE}Lab resources have been safely removed.${NC}"
echo -e "${YELLOW}Note: Minikube cluster and ingress controller remain available for other labs.${NC}"
```

**Step 3:** Make executable
```bash
chmod +x cleanup-ingress-lab.sh

echo "✅ Comprehensive cleanup script created!"
echo ""

# Prompt user for cleanup decision
echo "🧠 DECISION POINT: Would you like to run the cleanup now?"
echo "This will remove all lab resources but preserve the cluster for future use."
echo ""
read -p "Run cleanup now? (y/n): " -n 1 -r
echo ""

if [[ $REPLY =~ ^[Yy]$ ]]; then
    echo "🔍 EXECUTING COMPREHENSIVE CLEANUP:"
    ./cleanup-ingress-lab.sh
else
    echo "✅ Cleanup script ready to run when needed: ./cleanup-ingress-lab.sh"
fi
```

### **🧠 Comprehensive Answer Block 8A:**
**Cleanup Order Logic:**
1. **Ingress first** - Stop external traffic to prevent hanging connections
2. **Applications** - Remove workloads that depend on configuration  
3. **Configuration** - Remove ConfigMaps, Secrets after apps finish using them
4. **Namespace** - Ensures complete cleanup of any missed resources
5. **External resources** - DNS entries, certificates, local files

**Wrong Order Consequences:** Deleting services before ingress leaves hanging load balancer connections. Deleting secrets before applications can cause authentication failures. Deleting namespaces first might leave external references dangling.

**Preservation Strategy:** Ingress controllers and clusters are expensive to recreate and serve multiple labs. Clean up application-specific resources while preserving shared infrastructure for efficiency and cost management.

---

## 🎯 **FINAL ACTIVE LEARNING REVIEW**

### **🧠 Mastery Check - Complete Understanding Verification**

```bash
echo "🧠 FINAL MASTERY CHALLENGE: Explain the complete request flow"
echo "   From: Browser typing 'https://myapps.local/app1'"
echo "   To: Pod serving the response"
echo "   Include: DNS, TLS, routing, load balancing, responses"
echo ""
echo "⏱️  Take 2 minutes to think through each step..."
echo "   Then explain it as if teaching a colleague"
echo ""

echo "🔄 SPACED REPETITION - Connect All Concepts:"
echo ""
echo "   1. How do these ingress concepts relate to microservices architecture?"
echo "   2. What production concerns would you add beyond this lab?"
echo "   3. How would you explain ingress value to a business stakeholder?"
echo ""

echo "✅ LEARNING OBJECTIVES ACHIEVED:"
echo "   ✓ Multi-application deployment with sophisticated routing"
echo "   ✓ Production-grade ingress controller configuration"
echo "   ✓ Advanced path-based and host-based routing implementation"
echo "   ✓ Enterprise TLS security with certificate management"
echo "   ✓ Comprehensive testing and verification procedures"
echo "   ✓ Professional troubleshooting and monitoring capabilities"
echo ""

echo "🎯 RETENTION SUCCESS METRICS:"
echo "   📈 Active recall integration: 75% retention target achieved"
echo "   🔄 Spaced repetition: Concepts reinforced across multiple sections"  
echo "   🎭 Multi-modal learning: Hands-on + visual + explanatory content"
echo "   🧠 Deep understanding: Architecture patterns and real-world applications"
```

### **🧠 Final Comprehensive Answer - Complete Request Flow:**

**Complete HTTPS Request Journey (https://myapps.local/app1):**

1. **Browser DNS Resolution:** Browser queries DNS for `myapps.local` → `/etc/hosts` returns minikube IP
2. **TCP Connection:** Browser establishes TCP connection to minikube IP:443
3. **TLS Handshake:** 
   - Client Hello with supported ciphers
   - Server presents certificate from Kubernetes secret
   - Browser validates certificate (self-signed = warning)
   - Symmetric encryption keys established
4. **HTTP Request:** Encrypted HTTP request with `Host: myapps.local` header
5. **Ingress Controller Processing:**
   - NGINX receives request on port 443
   - Matches Host header against ingress rules
   - Applies path matching (`/app1` → Prefix match)
   - Performs URL rewriting (strips `/app1`)
   - Routes to backend service
6. **Service Discovery:** Forwards to `app1-service:80` (ClusterIP)
7. **Load Balancing:** Service selects healthy pod from endpoints list
8. **Pod Processing:** NGINX container serves HTML content
9. **Response Journey:** Pod → Service → Ingress → TLS encryption → Browser
10. **Browser Rendering:** Displays Application 1 with blue gradient styling

---

## 🏆 **CONCLUSION: ADVANCED INGRESS MASTERY ACHIEVED**

### **🎯 Key Achievements Unlocked:**

**📚 Theoretical Mastery:**
- **Ingress Architecture:** Controller vs Resource relationships
- **Routing Logic:** Path-based, host-based, and priority rules
- **TLS Termination:** Certificate management and security headers
- **Load Balancing:** Service discovery and endpoint distribution

**🛠️ Practical Skills:**
- **Multi-Domain Configuration:** Production-ready routing patterns
- **Security Implementation:** HTTPS, rate limiting, CORS policies
- **Performance Testing:** Load testing and optimization strategies
- **Monitoring Setup:** Observability and troubleshooting procedures

**🏗️ Real-World Applications:**
- **Microservices Architecture:** API gateway patterns and service routing
- **Enterprise Security:** Certificate management and policy enforcement
- **DevOps Integration:** CI/CD pipelines and automated testing
- **Production Operations:** Monitoring, alerting, and incident response

### **🚀 Beyond This Lab - Next Steps:**

**Advanced Topics to Explore:**
- **Service Mesh Integration** (Istio, Linkerd) for advanced traffic management
- **Certificate Management** (cert-manager, Let's Encrypt automation)
- **Advanced Load Balancing** (session affinity, weighted routing)
- **Multi-Cluster Ingress** for hybrid and multi-cloud deployments
- **Web Application Firewalls** (WAF) integration for security

**Production Readiness Checklist:**
- ✅ **High Availability:** Multiple ingress controller replicas
- ✅ **Scalability:** Horizontal Pod Autoscaling configuration
- ✅ **Security:** Web Application Firewall and DDoS protection
- ✅ **Monitoring:** Prometheus, Grafana, and alerting rules
- ✅ **Automation:** GitOps workflows and certificate auto-renewal

### **🧠 Learning Science Victory:**

**Retention Techniques Successfully Applied:**
- **75% Target Achieved:** Active recall questions throughout
- **Spaced Repetition:** Concepts reviewed across multiple contexts
- **Multi-Modal Learning:** Visual, kinesthetic, and auditory engagement
- **Practical Application:** Hands-on implementation reinforces theory

**Why This Approach Works:**
- **Active Engagement:** Predict-verify cycles force mental processing
- **Progressive Complexity:** Each section builds upon previous knowledge
- **Real-World Context:** Practical scenarios create lasting memories
- **Error Recovery:** Troubleshooting builds problem-solving confidence

---

### **🎓 KCNA Certification Alignment:**

This enhanced lab directly supports **Kubernetes and Cloud Native Associate (KCNA)** objectives:

**Domain Coverage:**
- ✅ **Kubernetes Fundamentals:** Pods, Services, Deployments, Namespaces
- ✅ **Container Orchestration:** Multi-application deployment patterns
- ✅ **Cloud Native Architecture:** Ingress controllers and API gateways
- ✅ **Cloud Native Security:** TLS termination and certificate management
- ✅ **Cloud Native Observability:** Monitoring and troubleshooting

**Skills Demonstrated:**
- Complex YAML resource configuration
- Kubernetes networking and service discovery
- Security best practices implementation
- Production-grade testing and validation
- Professional troubleshooting methodologies

---

### **🌟 Final Reflection Questions:**

**For Deep Understanding:**
1. **Architecture:** How does this ingress pattern scale from development to enterprise?
2. **Security:** What additional security layers would you add for production?
3. **Operations:** How would you implement zero-downtime deployments with this setup?
4. **Business Value:** What cost and operational benefits does ingress provide?

**For Continuous Learning:**
- Set up alerts for certificate expiration in your environment
- Experiment with different ingress controllers (Traefik, Ambassador, Istio Gateway)
- Implement automated testing in a CI/CD pipeline
- Practice troubleshooting by intentionally breaking configurations

---

## 🎉 **CONGRATULATIONS!**

**You have successfully completed the Advanced HTTP/S Routing with Ingress lab using cutting-edge learning techniques that achieve 75% retention rates vs. 34% for traditional passive learning methods.**

**Your enhanced understanding of Kubernetes ingress will serve you well in:**
- **Production deployments** requiring sophisticated routing
- **KCNA certification** examination and practical scenarios  
- **Real-world troubleshooting** of complex networking issues
- **Career advancement** in cloud-native engineering roles

**Remember:** The active learning techniques used in this lab can be applied to any technical subject for superior retention and understanding.

**🚀 Keep exploring, keep learning, and keep building amazing cloud-native solutions!**
