# 🔒 Istio Service Mesh: Zero-Trust Security & Observability Lab

> 💡 **Master service mesh security with mTLS encryption, comprehensive observability, and authorization policies**
> 
> ⏱️ **Time**: 3-4 hours | 🎯 **Difficulty**: Intermediate-Advanced | 🔧 **Prerequisites**: Kubernetes knowledge, basic networking | 💻 **Environment**: Linux with Kubernetes cluster

## 🎯 Objectives

By completing this lab, you will:

1. **Deploy and configure Istio 1.28.2** to create a production-ready service mesh with automatic sidecar injection and traffic management
2. **Implement zero-trust networking** using strict mutual TLS (mTLS) for encrypted service-to-service communication with cryptographic identity verification
3. **Establish comprehensive observability** through metrics collection (Prometheus), visualization (Grafana), distributed tracing (Jaeger), and service topology (Kiali)
4. **Enforce fine-grained security policies** using PeerAuthentication for mTLS modes and AuthorizationPolicy for access control between microservices
5. **Troubleshoot service mesh issues** including certificate validation, policy conflicts, and performance analysis

**Real-World Value**: Service meshes are critical for enterprise Kubernetes deployments. Organizations like Lyft, eBay, and Spotify use Istio to secure microservices, achieve compliance (PCI-DSS, HIPAA, SOC 2), and gain visibility into complex distributed systems. This lab teaches skills directly applicable to KCSA certification and production environments.

## 📊 Progress Tracker

Track your journey through service mesh mastery:

**Foundation**: ⚪⚪⚪⚪⚪ (Setup + Installation)
**Implementation**: ⚪⚪⚪⚪⚪ (Observability + mTLS)
**Troubleshooting**: ⚪⚪⚪⚪⚪ (Diagnostics + Validation)
**Production**: ⚪⚪⚪⚪⚪ (Testing + Assessment)

---

# 🚀 Phase 1: Environment Setup & Istio Installation

**Duration**: 15-20 minutes

## 🤔 Prediction Challenge (60 seconds)

Before proceeding, consider:
- What dependencies does Istio require? (Hint: Think about control plane, data plane, CLI tools)
- How will Istio inject sidecars into pods? (Hint: Consider Kubernetes admission controllers)
- What network components might conflict with Istio? (Hint: Think about CNI plugins, existing service meshes)
- How can we verify Istio is working correctly? (Hint: Multiple validation layers needed)

Write down your predictions. We'll validate them as we proceed.

---

## **1.1 System Requirements & Verification**

**Context**: Istio requires specific Kubernetes versions and system resources. We'll verify compatibility before installation.

### Verify Kubernetes cluster

```bash
kubectl version --short
```

**Expected Output**:
```
Client Version: v1.29+ 
Server Version: v1.29+ to v1.34
```

**Explanation**: Istio 1.28 officially supports Kubernetes versions 1.29 to 1.34. Version mismatches can cause control plane issues.

### Check cluster resources

```bash
kubectl get nodes
kubectl top nodes 2>/dev/null || echo "Metrics server not installed (optional)"
```

**Expected Output**:
```
NAME           STATUS   ROLES           AGE   VERSION
control-plane  Ready    control-plane   1d    v1.29+
worker-1       Ready    <none>          1d    v1.29+
```

**Requirements**: 
- Minimum: 2 CPUs, 4GB RAM per node
- Recommended: 4 CPUs, 8GB RAM for production-like environment
- Storage: 20GB available

### Verify required permissions

```bash
kubectl auth can-i create namespaces --all-namespaces
kubectl auth can-i create customresourcedefinitions
kubectl auth can-i create clusterroles
```

**All commands should return**: `yes`

**Why This Matters**: Istio creates cluster-scoped resources (CRDs, ClusterRoles). Without these permissions, installation will fail silently.

> ⚠️ **WARNING**: If running on managed Kubernetes (GKE, EKS, AKS), ensure your user has cluster-admin role or equivalent permissions.

---

## **1.2 Download and Install Istio 1.28.2**

**Context**: Istio 1.28.2 is the latest stable release (December 22, 2025) with security patches and enhanced features.

### Step 1: Download Istio

```bash
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.28.2 TARGET_ARCH=x86_64 sh -
```

**Expected Output**:
```
Downloading istio-1.28.2 from https://github.com/istio/istio/releases/download/1.28.2/istio-1.28.2-linux-amd64.tar.gz ...
Istio 1.28.2 Download Complete!
```

**What Happened**: Downloaded Istio binaries, manifests, and sample applications (~30MB).

### Step 2: Navigate to Istio directory

```bash
cd istio-1.28.2
ls -la
```

**Expected Output**:
```
bin/           # Contains istioctl CLI
samples/       # Demo applications
manifests/     # Kubernetes YAML files
tools/         # Additional utilities
```

### Step 3: Add istioctl to PATH

```bash
export PATH=$PWD/bin:$PATH
echo 'export PATH=$PWD/bin:$PATH' >> ~/.bashrc  # Persist for future sessions
```

### Step 4: Verify istioctl installation

```bash
istioctl version
```

**Expected Output**:
```
client version: 1.28.2
control plane version: none (no Istio components detected)
```

**Explanation**: `istioctl` is installed but the control plane isn't deployed yet. The CLI version should match our download (1.28.2).

> 💡 **TIP**: `istioctl` provides powerful debugging commands we'll use throughout this lab (analyze, proxy-config, proxy-status).

---

## **1.3 Install Istio Control Plane**

**Context**: The control plane (istiod) manages configuration, certificates, and service discovery. We'll use the `default` profile for balanced features and resources.

### Step 1: Analyze pre-installation environment

```bash
istioctl analyze
```

**Expected Output**:
```
✔ No validation issues found when analyzing namespace: default.
```

**What This Checks**: Validates cluster compatibility, existing configurations, and potential conflicts before installation.

### Step 2: Install Istio with default profile

```bash
istioctl install --set values.defaultRevision=default -y
```

**Expected Output**:
```
✔ Istio core installed
✔ Istiod installed
✔ Ingress gateways installed
✔ Installation complete
```

**What This Does**:
- Creates `istio-system` namespace
- Deploys istiod (control plane)
- Deploys istio-ingressgateway (traffic ingress)
- Installs 50+ Custom Resource Definitions (CRDs)

**Parameter Explanation**:
- `--set values.defaultRevision=default`: Sets this as the default revision for sidecar injection
- `-y`: Auto-confirms installation prompts

> ℹ️ **NOTE**: Installation takes 2-3 minutes. The control plane will stabilize before sidecars can be injected.

### Step 3: Verify control plane installation

```bash
kubectl get pods -n istio-system
```

**Expected Output**:
```
NAME                                    READY   STATUS    RESTARTS   AGE
istiod-xxxx                            1/1     Running   0          2m
istio-ingressgateway-xxxx              1/1     Running   0          2m
```

**Status Requirements**: All pods must show `Running` with `1/1` READY.

### Step 4: Check Istio components health

```bash
istioctl proxy-status
```

**Expected Output** (before any workloads):
```
NAME                                      CLUSTER        CDS        LDS        EDS        RDS        ECDS       PILOT
istio-ingressgateway-xxxx.istio-system    Kubernetes     SYNCED     SYNCED     SYNCED     SYNCED     SYNCED     istiod-xxxx
```

**Explanation**: This shows Envoy proxy synchronization status with istiod:
- **CDS** (Cluster Discovery Service): Backend endpoints
- **LDS** (Listener Discovery Service): Ports to listen on
- **EDS** (Endpoint Discovery Service): Service endpoints
- **RDS** (Route Discovery Service): HTTP routing rules
- **ECDS** (Extension Config Discovery): Extension configs
- All should show `SYNCED` for healthy state

---

## **1.4 Configure Automatic Sidecar Injection**

**Context**: Istio provides automatic sidecar injection through Kubernetes admission controllers. When enabled on a namespace, all new pods automatically get an Envoy sidecar.

### Step 1: Enable injection for default namespace

```bash
kubectl label namespace default istio-injection=enabled
```

**Expected Output**:
```
namespace/default labeled
```

**What This Does**: Adds a label that triggers the mutating webhook to inject sidecars into pods created in this namespace.

### Step 2: Verify namespace labeling

```bash
kubectl get namespace default --show-labels
```

**Expected Output**:
```
NAME      STATUS   AGE   LABELS
default   Active   1d    istio-injection=enabled,kubernetes.io/metadata.name=default
```

**Key Label**: `istio-injection=enabled`

### Step 3: Test sidecar injection (optional verification)

```bash
kubectl run test-nginx --image=nginx --restart=Never --dry-run=client -o yaml | istioctl kube-inject -f - | kubectl apply -f -
```

**Verify the pod has sidecars**:
```bash
kubectl get pod test-nginx -o jsonpath='{.spec.containers[*].name}'
```

**Expected Output**:
```
nginx istio-proxy
```

**Explanation**: Pod has two containers: application (nginx) and sidecar (istio-proxy).

### Step 4: Clean up test pod

```bash
kubectl delete pod test-nginx
```

---

## **1.5 Installation Validation Checklist**

Run this comprehensive check before proceeding:

```bash
echo "=== Istio Installation Validation ==="
echo ""
echo "1. Control Plane Status:"
kubectl get pods -n istio-system -o wide
echo ""
echo "2. CRDs Installed:"
kubectl get crd | grep istio | wc -l
echo "   (Should be 50+)"
echo ""
echo "3. Services Available:"
kubectl get svc -n istio-system
echo ""
echo "4. Injection Enabled:"
kubectl get namespace default -o jsonpath='{.metadata.labels.istio-injection}'
echo ""
echo ""
echo "5. Istioctl Version Match:"
istioctl version
```

**All checks must pass before proceeding to Phase 2.**

<details>
<summary>🔧 <strong>Troubleshooting: Common Setup Issues</strong></summary>

### Issue: Pods in CrashLoopBackOff

**Symptoms**:
```bash
istiod-xxxx   0/1   CrashLoopBackOff
```

**Diagnosis**:
```bash
kubectl logs -n istio-system <pod-name>
kubectl describe pod -n istio-system <pod-name>
```

**Common Causes & Fixes**:

| Cause | Symptoms | Solution |
|-------|----------|----------|
| Insufficient resources | OOMKilled in events | Increase node memory or reduce resource requests |
| Kubernetes version incompatibility | Validation errors in logs | Use Kubernetes 1.29-1.34 |
| Webhook configuration conflict | Admission webhook errors | Delete conflicting webhooks: `kubectl get mutatingwebhookconfigurations` |
| Previous Istio installation remnants | CRD conflicts | Uninstall cleanly: `istioctl uninstall --purge` |

### Issue: istioctl not in PATH

**Symptoms**:
```bash
bash: istioctl: command not found
```

**Fix**:
```bash
export PATH=$PWD/bin:$PATH
# Or use absolute path
/path/to/istio-1.28.2/bin/istioctl version
```

### Issue: Namespace injection not working

**Symptoms**: Pods only have one container (no sidecar)

**Diagnosis**:
```bash
kubectl get namespace default --show-labels
kubectl get mutatingwebhookconfigurations | grep istio
```

**Fixes**:
```bash
# Verify webhook is installed
kubectl get mutatingwebhookconfigurations istio-sidecar-injector -o yaml

# Recreate the label
kubectl label namespace default istio-injection=enabled --overwrite

# Force pod recreation
kubectl rollout restart deployment/<deployment-name>
```

### Issue: Permission denied errors

**Symptoms**: RBAC errors during installation

**Fix**:
```bash
# For managed Kubernetes, grant cluster-admin
kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole=cluster-admin \
  --user=$(gcloud config get-value account)  # Adjust for your cloud provider
```

</details>

---

**✅ Phase 1 Checkpoint**: Before moving forward, ensure:
- [ ] Istio 1.28.2 installed successfully
- [ ] All istio-system pods are Running (1/1 READY)
- [ ] `istioctl version` shows both client and control plane versions
- [ ] Default namespace labeled with `istio-injection=enabled`
- [ ] `istioctl analyze` reports no issues
- [ ] `istioctl proxy-status` shows ingress gateway SYNCED

**Progress Update**: Foundation ⭐⭐⭐⭐⭐

---

# 📦 Phase 2: Deploy Sample Application

**Duration**: 10-15 minutes

## 🤔 Prediction Challenge (90 seconds)

Before deploying, predict:
- How many containers will each pod have after deployment? Why?
- What happens if a service fails to get a sidecar? How would traffic behave?
- How can we verify sidecars are intercepting traffic correctly?
- What network policies does Istio create by default?

---

## **2.1 Deploy Bookinfo Application**

**Context**: Bookinfo is Istio's canonical demo app with microservices written in different languages (Python, Java, Ruby, Node.js). It demonstrates polyglot service mesh capabilities.

### Architecture Overview

The application has 4 services:
- **productpage**: Frontend (Python) - Entry point
- **details**: Book details microservice (Ruby)
- **reviews**: Book reviews microservice (Java) - Has 3 versions
- **ratings**: Star ratings microservice (Node.js)

### Step 1: Deploy the application

```bash
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

**Expected Output**:
```
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
... (multiple resources created)
```

**What This Creates**:
- 6 Deployments (details, ratings, reviews-v1/v2/v3, productpage)
- 4 Services
- 4 ServiceAccounts (for workload identity)

### Step 2: Verify service deployment

```bash
kubectl get services
```

**Expected Output**:
```
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
details       ClusterIP   10.x.x.x        <none>        9080/TCP   1m
kubernetes    ClusterIP   10.x.x.x        <none>        443/TCP    1d
productpage   ClusterIP   10.x.x.x        <none>        9080/TCP   1m
ratings       ClusterIP   10.x.x.x        <none>        9080/TCP   1m
reviews       ClusterIP   10.x.x.x        <none>        9080/TCP   1m
```

### Step 3: Check pod status and sidecars

```bash
kubectl get pods -o wide
```

**Expected Output**:
```
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-xxxx                   2/2     Running   0          2m
productpage-v1-xxxx               2/2     Running   0          2m
ratings-v1-xxxx                   2/2     Running   0          2m
reviews-v1-xxxx                   2/2     Running   0          2m
reviews-v2-xxxx                   2/2     Running   0          2m
reviews-v3-xxxx                   2/2     Running   0          2m
```

**Critical Observation**: Each pod shows `2/2` READY - one application container, one istio-proxy sidecar.

### Step 4: Wait for all pods to be ready

```bash
kubectl wait --for=condition=Ready pod --all --timeout=300s
```

**Expected Output**:
```
pod/details-v1-xxxx condition met
pod/productpage-v1-xxxx condition met
... (all pods)
```

### Step 5: Inspect sidecar injection details

```bash
kubectl get pod <productpage-pod-name> -o jsonpath='{.spec.containers[*].name}' | tr ' ' '\n'
```

**Expected Output**:
```
productpage
istio-proxy
```

**Detailed inspection**:
```bash
kubectl describe pod <productpage-pod-name> | grep -A 5 "istio-proxy"
```

**Key Sidecar Details**:
- Image: `docker.io/istio/proxyv2:1.28.2`
- Ports: 15090 (Prometheus), 15021 (health), 15000 (admin)
- Init Container: `istio-init` (sets up iptables rules)

---

## **2.2 Configure Ingress Gateway**

**Context**: The Ingress Gateway exposes services outside the mesh. We'll configure it to route external traffic to the productpage service.

### Step 1: Create Gateway and VirtualService

```bash
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

**Expected Output**:
```
gateway.networking.istio.io/bookinfo-gateway created
virtualservice.networking.istio.io/bookinfo created
```

**What This Creates**:
- **Gateway**: Defines how traffic enters the mesh (ports, protocols, hosts)
- **VirtualService**: Defines routing rules from gateway to productpage service

### Step 2: Verify Gateway configuration

```bash
kubectl get gateway
kubectl get virtualservice
```

**Expected Output**:
```
NAME               AGE
bookinfo-gateway   1m

NAME       GATEWAYS               HOSTS   AGE
bookinfo   ["bookinfo-gateway"]   ["*"]   1m
```

### Step 3: Get ingress gateway external access

```bash
kubectl get svc istio-ingressgateway -n istio-system
```

**Expected Output** (varies by environment):
```
NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)
istio-ingressgateway   LoadBalancer   10.x.x.x       <pending>       80:31380/TCP...
```

**Environment-Specific Instructions**:

**For Cloud Providers (AWS, GCP, Azure)**:
```bash
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
```

**For Minikube**:
```bash
export INGRESS_HOST=$(minikube ip)
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
```

**For kind/k3d/no external IP**:
```bash
kubectl port-forward -n istio-system svc/istio-ingressgateway 8080:80 &
export GATEWAY_URL=localhost:8080
```

### Step 4: Test application access

```bash
curl -s "http://${GATEWAY_URL}/productpage" | grep -o "<title>.*</title>"
```

**Expected Output**:
```
<title>Simple Bookstore App</title>
```

**Success Indicator**: HTML title confirms the application is accessible through the ingress gateway.

### Step 5: Generate initial traffic

```bash
for i in {1..10}; do
  curl -s "http://${GATEWAY_URL}/productpage" > /dev/null
  echo "Request $i completed"
  sleep 1
done
```

**Why**: Generates baseline traffic for observability tools we'll deploy next.

---

## **2.3 Verification and Investigation**

### Inspect Gateway routing configuration

```bash
kubectl get gateway bookinfo-gateway -o yaml
```

**Key Configuration**:
```yaml
spec:
  selector:
    istio: ingressgateway  # Targets istio-ingressgateway pods
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"  # Accepts traffic for any hostname
```

### Inspect VirtualService routing

```bash
kubectl get virtualservice bookinfo -o yaml
```

**Key Configuration**:
```yaml
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    route:
    - destination:
        host: productpage
        port:
          number: 9080
```

**Routing Logic**: HTTP requests to `/productpage` → productpage service:9080

### Check Envoy configuration in ingress gateway

```bash
istioctl proxy-config listener istio-ingressgateway-<pod-name>.istio-system
```

**Expected**: Shows listener on 0.0.0.0:8080 (or 80) for HTTP traffic

---

## 🧠 Active Recall Check #1

<details>
<summary>💡 <strong>Click to reveal questions - Try answering before expanding!</strong></summary>

**Question 1: Conceptual Understanding**
Why does each pod show `2/2` containers instead of `1/1`? Explain the purpose of each container and how they communicate.

**Question 2: Parameter Reasoning**
In the VirtualService, we used `host: "*"` in the Gateway. What would happen if we changed it to `host: "bookinfo.example.com"`? How would you test access?

**Question 3: Connection to Previous Learning**
How does automatic sidecar injection (Phase 1.4) relate to the pods showing 2/2 containers? What would happen if we deployed bookinfo in a namespace without the `istio-injection=enabled` label?

**Question 4: Real-World Application**
You're deploying a production application with separate staging and production namespaces. How would you configure Gateway and VirtualService resources to route `staging.company.com` to staging namespace and `app.company.com` to production? What are potential anti-patterns?

</details>

<details>
<summary>🔍 <strong>Answer Key</strong></summary>

**Answer 1**: Each pod has two containers: the application container (productpage, details, etc.) and the Envoy sidecar proxy (istio-proxy). They communicate via localhost: the application sends requests to localhost, iptables rules redirect to the sidecar (port 15001), the sidecar enforces policies/mTLS, then forwards to destination. This sidecar pattern enables traffic management without code changes.

**Answer 2**: Changing to `host: "bookinfo.example.com"` would require:
- DNS resolution: `bookinfo.example.com` must resolve to ingress IP
- Host header: curl must send `Host: bookinfo.example.com` header
- Test command: `curl -H "Host: bookinfo.example.com" http://$INGRESS_HOST/productpage`
- Without matching host header, Istio returns 404

**Answer 3**: Sidecar injection is controlled by the namespace label. When `istio-injection=enabled` is present, Kubernetes' mutating admission webhook automatically adds the istio-proxy container to pod specs. Without this label, pods would have 1/1 containers and would NOT participate in the mesh (no mTLS, no traffic management). Existing pods need recreation (rollout restart) to get sidecars.

**Answer 4**: Proper configuration:
```yaml
# Gateway accepting multiple hosts
spec:
  servers:
  - hosts: ["staging.company.com", "app.company.com"]

# Separate VirtualServices per environment
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: staging
spec:
  hosts: ["staging.company.com"]
  gateways: ["bookinfo-gateway"]
  http:
  - match:
    - headers:
        environment:
          exact: staging
    route:
    - destination:
        host: productpage.staging.svc.cluster.local
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: production
spec:
  hosts: ["app.company.com"]
  gateways: ["bookinfo-gateway"]
  http:
  - route:
    - destination:
        host: productpage.production.svc.cluster.local
```

**Anti-patterns to avoid**:
- Using wildcard `*` hosts in production (security risk)
- Single VirtualService with complex routing (maintenance nightmare)
- Not using TLS termination at gateway (exposes sensitive data)
- Missing timeout and retry configurations
</details>

---

**📝 Summary**: We've deployed a multi-service application with automatic sidecar injection and configured external access via Istio's Ingress Gateway. Every pod now has an Envoy proxy intercepting traffic, enabling advanced features we'll explore next.

**Progress Update**: Foundation ⭐⭐⭐⭐⭐ | Implementation ⭐⭐⚪⚪⚪

---

# 🔍 Phase 3: Comprehensive Observability

**Duration**: 20-25 minutes

## 🤔 Prediction Challenge (90 seconds)

Before enabling observability, predict:
- What metrics would be most valuable for troubleshooting microservices? (Hint: RED method - Rate, Errors, Duration)
- How does Istio collect metrics without application code changes? (Hint: Where does the sidecar sit?)
- What's the difference between metrics, logs, and traces? When would you use each?
- What overhead does observability add to requests? (Hint: Consider sidecar processing, data collection)

---

## **3.1 Install Observability Add-ons**

**Context**: Istio integrates with popular observability tools through pre-configured add-ons. These run in the istio-system namespace.

### Architecture Overview

| Component | Purpose | Port |
|-----------|---------|------|
| Prometheus | Metrics collection & storage | 9090 |
| Grafana | Metrics visualization & dashboards | 3000 |
| Jaeger | Distributed tracing | 16686 |
| Kiali | Service mesh topology & health | 20001 |

### Step 1: Install Prometheus

```bash
kubectl apply -f samples/addons/prometheus.yaml
```

**Expected Output**:
```
serviceaccount/prometheus created
configmap/prometheus created
deployment.apps/prometheus created
service/prometheus created
```

**What This Does**: Deploys Prometheus configured to scrape metrics from Istio components and application sidecars every 15 seconds.

### Step 2: Install Grafana

```bash
kubectl apply -f samples/addons/grafana.yaml
```

**Expected Output**:
```
service/grafana created
deployment.apps/grafana created
configmap/istio-grafana-dashboards created
```

**Includes**: Pre-built Istio dashboards for mesh overview, service performance, workload metrics.

### Step 3: Install Jaeger

```bash
kubectl apply -f samples/addons/jaeger.yaml
```

**Expected Output**:
```
deployment.apps/jaeger created
service/jaeger-collector created
service/jaeger-query created
```

**What This Does**: Enables distributed tracing to track requests across microservices with sub-millisecond precision.

### Step 4: Install Kiali

```bash
kubectl apply -f samples/addons/kiali.yaml
```

**Expected Output**:
```
serviceaccount/kiali created
configmap/kiali created
clusterrole.rbac.authorization.k8s.io/kiali created
deployment.apps/kiali created
```

**What This Does**: Provides service mesh visualization, real-time traffic flow, and configuration validation.

### Step 5: Verify all observability pods are running

```bash
kubectl get pods -n istio-system -l 'app in (prometheus,grafana,jaeger,kiali)'
```

**Expected Output**:
```
NAME                          READY   STATUS    RESTARTS   AGE
grafana-xxxx                  1/1     Running   0          2m
jaeger-xxxx                   1/1     Running   0          2m
kiali-xxxx                    1/1     Running   0          2m
prometheus-xxxx               1/1     Running   0          2m
```

**Wait for all pods to be Running**:
```bash
kubectl wait --for=condition=Ready pod -n istio-system -l app=prometheus --timeout=120s
kubectl wait --for=condition=Ready pod -n istio-system -l app=grafana --timeout=120s
kubectl wait --for=condition=Ready pod -n istio-system -l app=jaeger --timeout=120s
kubectl wait --for=condition=Ready pod -n istio-system -l app=kiali --timeout=120s
```

> ℹ️ **NOTE**: These add-ons are for demonstration purposes. In production, use enterprise-grade observability stacks (Prometheus Operator, Grafana Cloud, Datadog, New Relic).

---

## **3.2 Configure Telemetry Collection**

**Context**: Istio's Telemetry API controls what metrics, traces, and logs are collected. We'll enable comprehensive collection for security and performance monitoring.

### Step 1: Create telemetry configuration file

```bash
nano telemetry-config.yaml
```

**Paste this content**, then **Ctrl+O**, **Enter**, **Ctrl+X**:

```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: default-metrics
  namespace: istio-system
spec:
  metrics:
  - providers:
    - name: prometheus
  - overrides:
    - match:
        metric: ALL_METRICS
      tagOverrides:
        request_id:
          operation: UPSERT
          value: "REQUEST_ID(%REQ(x-request-id)%)"
---
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: default-tracing
  namespace: istio-system
spec:
  tracing:
  - providers:
    - name: jaeger
    randomSamplingPercentage: 100.0
---
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: default-logging
  namespace: istio-system
spec:
  accessLogging:
  - providers:
    - name: envoy
```

**Configuration Breakdown**:
- **default-metrics**: Enables Prometheus metrics with request ID tracking
- **default-tracing**: Enables 100% trace sampling (reduce to 1-10% in production)
- **default-logging**: Enables Envoy access logs

### Step 2: Apply telemetry configuration

```bash
kubectl apply -f telemetry-config.yaml
```

**Expected Output**:
```
telemetry.telemetry.istio.io/default-metrics created
telemetry.telemetry.istio.io/default-tracing created
telemetry.telemetry.istio.io/default-logging created
```

### Step 3: Verify telemetry configuration

```bash
kubectl get telemetry -n istio-system
```

**Expected Output**:
```
NAME              AGE
default-metrics   1m
default-tracing   1m
default-logging   1m
```

### Step 4: Verify Prometheus is scraping metrics

```bash
kubectl exec -n istio-system deployment/prometheus -- wget -qO- http://localhost:9090/api/v1/targets | grep -o "istio-mesh"
```

**Expected Output**: Should show multiple `istio-mesh` targets indicating active scraping.

> 💡 **TIP**: Telemetry configuration applies mesh-wide. For workload-specific settings, use namespace or workload selectors in Telemetry resources.

---

## **3.3 Generate Traffic for Observability**

**Context**: Observability tools need traffic to display meaningful data. We'll generate continuous load to populate dashboards.

### Step 1: Generate sustained traffic

Open a **new terminal** and run:

```bash
watch -n 2 'curl -s "http://${GATEWAY_URL}/productpage" > /dev/null && echo "Request sent at $(date +%T)"'
```

**Alternative** (if watch unavailable):
```bash
while true; do
  curl -s "http://${GATEWAY_URL}/productpage" > /dev/null
  echo "Request completed at $(date +%T)"
  sleep 2
done
```

**What This Does**: Sends requests every 2 seconds, simulating user traffic.

**Keep this running** throughout the observability exploration. Return to the original terminal.

### Step 2: Generate error traffic for dashboard variety

In original terminal:

```bash
for i in {1..5}; do
  curl -s "http://${GATEWAY_URL}/non-existent-page" > /dev/null
  echo "Error request $i sent"
  sleep 1
done
```

**Why**: Creates 404 errors visible in metrics and traces, demonstrating error tracking.

### Step 3: Generate traffic with different latencies

```bash
for i in {1..10}; do
  # Simulate slow requests by adding query parameters that trigger delays
  curl -s "http://${GATEWAY_URL}/productpage?u=test&delay=$((RANDOM % 3))" > /dev/null
  echo "Variable latency request $i completed"
  sleep 1
done
```

---

## **3.4 Access and Explore Observability Dashboards**

**Context**: We'll use kubectl port-forward to access dashboards locally. Each dashboard provides unique insights.

### Step 1: Access Grafana (Metrics Visualization)

```bash
kubectl port-forward -n istio-system svc/grafana 3000:3000 > /dev/null 2>&1 &
echo "✅ Grafana available at: http://localhost:3000"
echo "   Default login: admin / admin (no password for demo)"
```

**Open in browser**: http://localhost:3000

**Dashboards to explore**:
1. **Istio Mesh Dashboard**: Overall mesh health, request rates, success rates
2. **Istio Service Dashboard**: Per-service metrics (RPS, latency percentiles, error rates)
3. **Istio Workload Dashboard**: Per-pod CPU, memory, network usage

**Key Metrics to Note**:
- **Request Rate**: Operations/second per service
- **P50/P95/P99 Latency**: Response time percentiles
- **Success Rate**: % of non-error responses
- **Active Connections**: Current TCP connections

### Step 2: Access Jaeger (Distributed Tracing)

```bash
kubectl port-forward -n istio-system svc/jaeger-query 16686:16686 > /dev/null 2>&1 &
echo "✅ Jaeger available at: http://localhost:16686"
```

**Open in browser**: http://localhost:16686

**Investigation Exercise**:
1. Select Service: `productpage.default`
2. Click "Find Traces"
3. Select a trace to see request flow

**Trace Anatomy**:
- **Span**: Single operation (e.g., HTTP request to details service)
- **Trace**: Collection of spans representing one request through the system
- **Timeline**: Shows sequential and parallel operations

**What to Look For**:
- Total request duration
- Which services were called
- Where time was spent (network vs processing)
- Any errors or retries

### Step 3: Access Kiali (Service Mesh Topology)

```bash
kubectl port-forward -n istio-system svc/kiali 20001:20001 > /dev/null 2>&1 &
echo "✅ Kiali available at: http://localhost:20001"
echo "   Default login: admin / admin (no token needed for demo)"
```

**Open in browser**: http://localhost:20001

**Views to Explore**:
1. **Graph View**: Visual service topology
   - Select Namespace: `default`
   - Display: `Versioned app graph`
   - Traffic Animation: Enable
   - **Observe**: Real-time traffic flow between services

2. **Applications**: See all deployed apps and their health

3. **Workloads**: Detailed pod-level information

4. **Istio Config**: Validation of Istio resources (Gateway, VirtualService, DestinationRule)

**Key Features**:
- **Traffic Percentage**: Shows request distribution (e.g., reviews-v1 vs v2 vs v3)
- **Response Time**: Color-coded by latency
- **Error Rate**: Red indicators for failing services
- **mTLS Status**: Lock icon indicates encrypted connections

### Step 4: Access Prometheus (Raw Metrics)

```bash
kubectl port-forward -n istio-system svc/prometheus 9090:9090 > /dev/null 2>&1 &
echo "✅ Prometheus available at: http://localhost:9090"
```

**Open in browser**: http://localhost:9090

**Useful Queries** (enter in expression browser):
```promql
# Request rate per second for productpage
rate(istio_requests_total{destination_service=~"productpage.*"}[1m])

# P95 latency for all services
histogram_quantile(0.95, sum(rate(istio_request_duration_milliseconds_bucket[1m])) by (le, destination_service))

# Error rate percentage
sum(rate(istio_requests_total{response_code=~"5.*"}[1m])) / sum(rate(istio_requests_total[1m])) * 100

# Active TCP connections
sum(istio_tcp_connections_opened_total) - sum(istio_tcp_connections_closed_total)
```

> 💡 **TIP**: Use Prometheus for ad-hoc queries and alerting. Use Grafana for dashboards and visualization.

---

## **3.5 Investigate Metrics and Traces**

### Exercise 1: Trace a Slow Request

1. **In Jaeger**, find a trace with duration > 100ms
2. Click to expand all spans
3. **Identify the slowest span** (longest bar)

**Questions to Answer**:
- Which service is the bottleneck?
- Is it network latency or processing time?
- Are any services called in parallel?

### Exercise 2: Analyze Service Dependencies

1. **In Kiali Graph View**, select `default` namespace
2. Enable "Service Nodes" and "Traffic Animation"

**Questions to Answer**:
- Which services does productpage call directly?
- What's the request flow: productpage → ? → ?
- Are all 3 versions of reviews receiving traffic equally?

### Exercise 3: Monitor Error Rates

1. **In Grafana Istio Service Dashboard**
2. Filter to `productpage` service
3. Observe "Client Success Rate" panel

**Generate errors**:
```bash
for i in {1..20}; do curl -s "http://${GATEWAY_URL}/bad-endpoint" > /dev/null; done
```

**Watch the dashboard** - error rate should spike and success rate should drop.

---

## 🧠 Active Recall Check #2

<details>
<summary>💡 <strong>Click to reveal questions - Try answering before expanding!</strong></summary>

**Question 1: Conceptual Understanding**
Explain the difference between metrics, logs, and traces. Give an example scenario where each would be most useful for troubleshooting.

**Question 2: Parameter Reasoning**
We set `randomSamplingPercentage: 100.0` for tracing. What are the tradeoffs? When would you use 1%, 10%, or 100%? Calculate the overhead: if you have 10,000 requests/sec, how many traces/sec would 10% sampling generate?

**Question 3: Connection to Previous Learning**
How does the Envoy sidecar (from Phase 2) enable observability without application code changes? Where in the request path are metrics collected?

**Question 4: Real-World Application**
You're investigating a production issue where the p99 latency for a payment service spikes to 5 seconds every hour. Which observability tools would you use in what order? Describe your troubleshooting workflow with specific queries or dashboard views.

</details>

<details>
<summary>🔍 <strong>Answer Key</strong></summary>

**Answer 1**:
- **Metrics**: Aggregated numerical data (counters, gauges, histograms). Best for: Monitoring trends, alerting on thresholds. Example: "Payment service error rate exceeded 1% - alert triggered"
- **Logs**: Discrete event records with timestamp. Best for: Debugging specific errors, audit trails. Example: "This user's payment failed with error X at timestamp Y"
- **Traces**: Request flow through distributed system with timing. Best for: Understanding latency bottlenecks, service dependencies. Example: "The payment request took 3s total: 2.8s waiting for external payment gateway, 0.2s in our code"

Use together: Start with metrics (identify problem), use traces (narrow down service), examine logs (root cause).

**Answer 2**:
Tradeoffs:
- **100% sampling**: Complete visibility, HIGH overhead (storage, network, CPU). Only for dev/testing
- **10% sampling**: Good balance, 1 in 10 requests traced. Overhead: 10,000 req/s × 0.10 = 1,000 traces/s
- **1% sampling**: Production default. May miss rare issues. Overhead: 10,000 req/s × 0.01 = 100 traces/s

**Overhead calculation**:
- Each trace: ~1-5KB depending on span count
- At 1,000 traces/sec: 1-5 MB/sec network traffic to Jaeger
- Storage: 86-430 GB/day if retained

**Production best practice**: Use adaptive sampling - 100% for errors, 1-5% for successful requests.

**Answer 3**:
The Envoy sidecar acts as a transparent proxy intercepting all inbound/outbound traffic via iptables rules. For observability:
1. **Request enters pod** → iptables redirect to Envoy (port 15001)
2. **Envoy processes request** → records metrics (latency, status), generates trace spans, writes access logs
3. **Envoy forwards to application** (localhost)
4. **Application responds** → Envoy records response metrics, propagates trace context
5. **Metrics scraped by Prometheus** from Envoy admin port (15090)

Application sees nothing - it receives requests from localhost and sends responses to localhost. All observability is "sidecar-instrumented".

**Answer 4**:
Systematic troubleshooting workflow:

**Step 1: Confirm issue (Metrics - 2 min)**
- Grafana → Istio Service Dashboard → Payment service
- Check p99 latency panel over past 6 hours
- Confirm hourly spike pattern
- Check error rate - is it correlated with latency?

**Step 2: Identify slow component (Tracing - 5 min)**
- Jaeger → Service: payment
- Filter by duration: > 4000ms
- Select trace from spike window
- Identify which span consumes 5 seconds (database? external API? internal processing?)

**Step 3: Correlate with system events (Metrics + Logs - 5 min)**
- Prometheus query: `rate(istio_requests_total{destination_service="payment"}[5m])`
- Check if request rate spike correlates with latency
- Check payment service logs: `kubectl logs -l app=payment --since=1h | grep ERROR`
- Look for: database connection pool exhaustion, rate limit errors, external API timeouts

**Step 4: Check dependencies (Kiali - 2 min)**
- Graph View → Payment service
- Identify upstream/downstream services
- Check if *other* services also slow during spike
- If yes → shared dependency (DB, cache). If no → payment-specific issue

**Step 5: Hypothesis and validation**
- Example finding: External payment gateway has 5s timeout every hour during batch processing
- Validation: Check gateway documentation, contact gateway provider
- Mitigation: Increase timeout, implement circuit breaker, add retry with backoff

**Prometheus queries used**:
```promql
# P99 latency over time
histogram_quantile(0.99, rate(istio_request_duration_milliseconds_bucket{destination_service="payment"}[5m]))

# Request rate
rate(istio_requests_total{destination_service="payment"}[5m])

# Error rate
rate(istio_requests_total{destination_service="payment",response_code=~"5.*"}[5m])
```

</details>

---

**📝 Summary**: We've deployed comprehensive observability with Prometheus metrics, Grafana visualization, Jaeger distributed tracing, and Kiali service topology. The mesh now provides complete visibility into request flows, latency distributions, error rates, and service dependencies - all without application code changes.

**🔄 Spaced Review**: Recall Phase 1 concepts:
- Where are Istio control plane components running? (istio-system namespace)
- What label enables sidecar injection? (istio-injection=enabled)
- How many containers are in each application pod? (2: app + istio-proxy)

**Progress Update**: Implementation ⭐⭐⭐⭐⚪

---

# 🔒 Phase 4: Implement Mutual TLS (mTLS) Security

**Duration**: 25-30 minutes

## 🤔 Prediction Challenge (90 seconds)

Before implementing mTLS, predict:
- What authentication modes does Istio support? What are the security tradeoffs?
- How does mTLS work without touching application code? Where are certificates stored?
- What happens to plain HTTP traffic when we enforce STRICT mTLS?
- How often are certificates rotated? What happens if rotation fails?

---

## **4.1 Understanding mTLS in Istio**

**Conceptual Foundation**:

Mutual TLS (mTLS) provides cryptographic identity, confidentiality, and integrity for service-to-service communication. Unlike standard TLS (used for HTTPS), mTLS requires *both* client and server to present certificates.

**Why mTLS Matters for Zero-Trust**:
1. **Identity**: Each service has a cryptographically verifiable identity (X.509 certificate)
2. **Confidentiality**: Traffic is encrypted end-to-end
3. **Integrity**: Traffic cannot be tampered with in transit
4. **No IP-based trust**: Identity is enforced regardless of topology - no need to track IP addresses

**How Istio Implements mTLS**:
- **Automatic**: Istio's mTLS can be enabled without requiring service code changes
- **Built-in CA**: Istiod acts as Certificate Authority, issues certificates via SDS (Secret Discovery Service)
- **Automatic rotation**: Certificates rotate every 24 hours by default
- **Workload identity**: Based on Kubernetes ServiceAccount mapped to SPIFFE identity

**mTLS Modes**:

| Mode | Behavior | Use Case |
|------|----------|----------|
| **PERMISSIVE** | Accepts both mTLS and plaintext | Migration period, legacy services |
| **STRICT** | Only accepts mTLS traffic | Production security, zero-trust |
| **DISABLE** | No mTLS (plaintext only) | External services, testing |

> 🔒 **SECURITY**: From a security perspective, you shouldn't use DISABLE mode unless you provide your own security solution.

---

## **4.2 Verify Current TLS Status**

**Context**: Before enabling STRICT mode, we'll inspect the current configuration. By default, Istio uses PERMISSIVE mode.

### Step 1: Check current mTLS status with istioctl

```bash
istioctl authn tls-check $(kubectl get pod -l app=productpage -o jsonpath='{.items[0].metadata.name}').default
```

**Expected Output**:
```
HOST:PORT                                  STATUS     SERVER     CLIENT     AUTHN POLICY        DESTINATION RULE
details.default.svc.cluster.local:9080     OK         mTLS       mTLS       default/            -
kubernetes.default.svc.cluster.local:443   OK         -          -          -                   -
productpage.default.svc.cluster.local:9080 OK         mTLS       mTLS       default/            -
ratings.default.svc.cluster.local:9080     OK         mTLS       mTLS       default/            -
reviews.default.svc.cluster.local:9080     OK         mTLS       mTLS       default/            -
```

**Status Explanation**:
- **OK**: mTLS negotiation successful
- **SERVER mTLS**: Server accepts mTLS connections
- **CLIENT mTLS**: Client sends mTLS connections
- **Auto mTLS** is working: Sidecars automatically use mTLS even without explicit configuration

### Step 2: Check for existing PeerAuthentication policies

```bash
kubectl get peerauthentication --all-namespaces
```

**Expected Output** (initially):
```
No resources found
```

**What This Means**: No explicit policies exist. The mesh is using default AUTO behavior (PERMISSIVE mode).

### Step 3: Check DestinationRule configurations

```bash
kubectl get destinationrule --all-namespaces
```

**Expected Output** (initially):
```
No resources found
```

**What This Means**: Auto mTLS works by automatically determining if Istio mutual TLS should be sent without explicit DestinationRule configuration.

### Step 4: Inspect certificate details in a sidecar

```bash
istioctl proxy-config secret $(kubectl get pod -l app=productpage -o jsonpath='{.items[0].metadata.name}')
```

**Expected Output**:
```
RESOURCE NAME     TYPE           STATUS     VALID CERT     SERIAL NUMBER                        NOT AFTER                NOT BEFORE
default           Cert Chain     ACTIVE     true           329939471922583863671240379853       2025-12-31T12:00:00Z     2025-12-30T11:00:00Z
ROOTCA            CA             ACTIVE     true           123456789                            2035-12-28T12:00:00Z     2025-12-28T12:00:00Z
```

**Key Certificate Details**:
- **default**: Workload certificate (client cert for outbound, server cert for inbound)
- **ROOTCA**: Root certificate to verify peer certificates
- **Validity**: Certificates valid for 24 hours by default
- **Serial Number**: Unique identifier for this certificate

### Step 5: View actual certificate content

```bash
istioctl proxy-config secret $(kubectl get pod -l app=productpage -o jsonpath='{.items[0].metadata.name}') -o json | grep -A 10 '"name": "default"'
```

**What to Look For**:
- Certificate chain encoded in base64
- Private key (also base64, used for TLS handshake)
- Trust domain (usually `cluster.local`)

---

## **4.3 Enable STRICT mTLS Mode**

**Context**: STRICT mode enforces mTLS for all communications, blocking plaintext traffic. This is the recommended mode for zero-trust security.

### Step 1: Create PeerAuthentication policy for STRICT mTLS

```bash
nano strict-mtls.yaml
```

**Paste this content**, then **Ctrl+O**, **Enter**, **Ctrl+X**:

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: default
spec:
  mtls:
    mode: STRICT
```

**Configuration Breakdown**:
- **metadata.name**: "default" applies to all workloads in namespace
- **metadata.namespace**: "default" targets our application namespace
- **mtls.mode: STRICT**: Rejects plaintext connections

**Alternative**: Mesh-wide STRICT mTLS (apply to istio-system):
```yaml
metadata:
  name: default
  namespace: istio-system  # Applies to entire mesh
```

### Step 2: Apply STRICT mTLS policy

```bash
kubectl apply -f strict-mtls.yaml
```

**Expected Output**:
```
peerauthentication.security.istio.io/default created
```

### Step 3: Verify policy is active

```bash
kubectl get peerauthentication
```

**Expected Output**:
```
NAME      MODE     AGE
default   STRICT   30s
```

### Step 4: Verify mTLS enforcement

```bash
istioctl authn tls-check $(kubectl get pod -l app=productpage -o jsonpath='{.items[0].metadata.name}').default
```

**Expected Output** (mode should now show STRICT):
```
HOST:PORT                                  STATUS     SERVER     CLIENT     AUTHN POLICY        DESTINATION RULE
details.default.svc.cluster.local:9080     OK         STRICT     mTLS       default/default     -
ratings.default.svc.cluster.local:9080     OK         STRICT     mTLS       default/default     -
reviews.default.svc.cluster.local:9080     OK         STRICT     mTLS       default/default     -
```

**Key Change**: SERVER column now shows `STRICT` instead of `mTLS` (permissive).

---

## **4.4 Configure DestinationRules for mTLS**

**Context**: While PeerAuthentication controls *server-side* mTLS acceptance, DestinationRule controls *client-side* mTLS usage. Best practice is to explicitly configure both.

### Step 1: Create DestinationRule for mTLS traffic

```bash
nano destination-rule-mtls.yaml
```

**Paste this content**, then **Ctrl+O**, **Enter**, **Ctrl+X**:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: default
  namespace: default
spec:
  host: "*.default.svc.cluster.local"
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
```

**Configuration Breakdown**:
- **host**: Wildcard matches all services in default namespace
- **trafficPolicy.tls.mode: ISTIO_MUTUAL**: Use Istio-managed mTLS certificates

**TLS Modes Available**:

| Mode | Description | Use Case |
|------|-------------|----------|
| ISTIO_MUTUAL | Use Istio mTLS | Services within the mesh |
| MUTUAL | Use custom certificates | External services with own PKI |
| SIMPLE | Standard TLS (client only) | External HTTPS services |
| DISABLE | No TLS | Plaintext (testing only) |

### Step 2: Apply destination rule

```bash
kubectl apply -f destination-rule-mtls.yaml
```

**Expected Output**:
```
destinationrule.networking.istio.io/default created
```

### Step 3: Verify destination rule

```bash
kubectl get destinationrule
```

**Expected Output**:
```
NAME      HOST                              AGE
default   *.default.svc.cluster.local       30s
```

### Step 4: Inspect the applied configuration

```bash
kubectl get destinationrule default -o yaml
```

**Verify the trafficPolicy section** shows:
```yaml
trafficPolicy:
  tls:
    mode: ISTIO_MUTUAL
```

---

## **4.5 Test and Validate mTLS Configuration**

**Context**: We'll verify that mTLS is working by testing service-to-service communication and examining TLS certificates.

### Step 1: Test service-to-service communication with mTLS

```bash
kubectl exec -it $(kubectl get pod -l app=productpage -o jsonpath='{.items[0].metadata.name}') -c productpage -- curl -s http://details:9080/details/0
```

**Expected Output** (JSON response):
```json
{"id":0,"author":"William Shakespeare","year":1595,"type":"paperback","pages":200,"publisher":"PublisherA","language":"English","ISBN-10":"1234567890","ISBN-13":"123-1234567890"}
```

**What This Proves**: 
- mTLS is transparent to applications (app uses HTTP, sidecars negotiate mTLS)
- Connection succeeded despite STRICT mode
- Application code unchanged

### Step 2: Verify TLS handshake details

```bash
kubectl exec -it $(kubectl get pod -l app=productpage -o jsonpath='{.items[0].metadata.name}') -c istio-proxy -- curl -v telnet://details:9080 2>&1 | head -20
```

**Look for**: Connection established indicators (though details won't show TLS because we're in the sidecar).

### Step 3: Check certificate being used

```bash
istioctl proxy-config secret $(kubectl get pod -l app=productpage -o jsonpath='{.items[0].metadata.name}') | grep default
```

**Expected Output**:
```
default           Cert Chain     ACTIVE     true           329939471922583863671240379853       2025-12-31T12:00:00Z     2025-12-30T11:00:00Z
```

**Certificate Status**: Must show ACTIVE and true for VALID CERT.

### Step 4: Verify certificate details with openssl

```bash
kubectl exec $(kubectl get pod -l app=productpage -o jsonpath='{.items[0].metadata.name}') -c istio-proxy -- \
  cat /var/run/secrets/istio/root-cert.pem | openssl x509 -text -noout | grep -A 2 "Subject:"
```

**Expected Output**:
```
Subject: O = cluster.local
    Not Before: Dec 28 12:00:00 2025 GMT
    Not After : Dec 28 12:00:00 2035 GMT
```

**Key Details**:
- **O = cluster.local**: Trust domain
- **10-year validity**: Root certificate (workload certs are 24h)

### Step 5: Test that plaintext traffic is blocked

**Create a test pod without Istio sidecar**:

```bash
kubectl run test-plaintext --image=curlimages/curl --rm -it --restart=Never --labels="sidecar.istio.io/inject=false" -- curl -s --max-time 5 http://details:9080/details/0
```

**Expected Behavior**: 
- Connection should **fail** or **timeout**
- Error: `curl: (28) Connection timed out after 5000 milliseconds`

**Why It Fails**: The details service only accepts mTLS connections (STRICT mode). The test pod has no sidecar and cannot perform mTLS handshake.

> 🔒 **SECURITY**: This proves that STRICT mode successfully blocks unauthorized plaintext traffic.

---

## **4.6 Implement Authorization Policies**

**Context**: mTLS provides identity and encryption. AuthorizationPolicy controls *who* can access *what* based on those identities.

### Step 1: Create authorization policies

```bash
nano authorization-policy.yaml
```

**Paste this content**, then **Ctrl+O**, **Enter**, **Ctrl+X**:

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: productpage-policy
  namespace: default
spec:
  selector:
    matchLabels:
      app: productpage
  rules:
  - from:
    - source:
        principals:
          - "cluster.local/ns/default/sa/bookinfo-productpage"
          - "cluster.local/ns/istio-system/sa/istio-ingressgateway-service-account"
    to:
    - operation:
        methods: ["GET", "POST"]
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: details-policy
  namespace: default
spec:
  selector:
    matchLabels:
      app: details
  rules:
  - from:
    - source:
        principals:
          - "cluster.local/ns/default/sa/bookinfo-productpage"
    to:
    - operation:
        methods: ["GET"]
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: ratings-policy
  namespace: default
spec:
  selector:
    matchLabels:
      app: ratings
  rules:
  - from:
    - source:
        principals:
          - "cluster.local/ns/default/sa/bookinfo-reviews"
    to:
    - operation:
        methods: ["GET"]
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: reviews-policy
  namespace: default
spec:
  selector:
    matchLabels:
      app: reviews
  rules:
  - from:
    - source:
        principals:
          - "cluster.local/ns/default/sa/bookinfo-productpage"
    to:
    - operation:
        methods: ["GET"]
```

**Policy Breakdown**:

**productpage-policy**:
- Allows GET/POST from its own ServiceAccount and ingress gateway
- This permits external traffic through the gateway and internal calls

**details-policy**:
- Only allows GET from productpage ServiceAccount
- Blocks direct access from other services

**ratings-policy**:
- Only allows GET from reviews ServiceAccount
- Enforces that ratings is only accessed by reviews service

**reviews-policy**:
- Only allows GET from productpage ServiceAccount
- Prevents unauthorized access to reviews

**SPIFFE Identity Format**:
```
cluster.local/ns/<namespace>/sa/<serviceaccount>
└─ Trust Domain  └─ Namespace  └─ ServiceAccount
```

### Step 2: Apply authorization policies

```bash
kubectl apply -f authorization-policy.yaml
```

**Expected Output**:
```
authorizationpolicy.security.istio.io/productpage-policy created
authorizationpolicy.security.istio.io/details-policy created
authorizationpolicy.security.istio.io/ratings-policy created
authorizationpolicy.security.istio.io/reviews-policy created
```

### Step 3: Verify policies are active

```bash
kubectl get authorizationpolicy
```

**Expected Output**:
```
NAME                 AGE
details-policy       30s
productpage-policy   30s
ratings-policy       30s
reviews-policy       30s
```

### Step 4: Test that authorized traffic still works

```bash
curl -s "http://${GATEWAY_URL}/productpage" | grep -o "<title>.*</title>"
```

**Expected Output**:
```
<title>Simple Bookstore App</title>
```

**Success**: Authorized path (ingress → productpage → details/reviews) works.

### Step 5: Test that unauthorized access is blocked

**Attempt direct access to details (bypassing productpage)**:

```bash
kubectl exec -it $(kubectl get pod -l app=reviews -o jsonpath='{.items[0].metadata.name}') -c reviews -- curl -s --max-time 5 http://details:9080/details/0
```

**Expected Output**:
```
RBAC: access denied
```

**Why Blocked**: reviews ServiceAccount is not in the allowed principals for details-policy. Only productpage can access details.

> 🔒 **SECURITY**: This demonstrates defense-in-depth: even with mTLS encryption, authorization policies enforce least-privilege access.

---

## **4.7 Monitor Security Events in Real-Time**

**Context**: Security policies should be observable. We'll monitor mTLS connections and authorization decisions.

### Step 1: View Envoy access logs with security context

```bash
kubectl logs -l app=details -c istio-proxy --tail=20
```

**Look for fields in logs**:
- **requested_server_name**: TLS SNI
- **downstream_local_subject**: Client certificate subject
- **response_code**: 200 (allowed), 403 (denied)

**Example log entry**:
```
[2025-12-30T12:00:00.000Z] "GET /details/0 HTTP/1.1" 200 - via_upstream ...
downstream_local_subject=O=cluster.local
```

### Step 2: Check for authorization denials

```bash
kubectl logs -l app=details -c istio-proxy --tail=50 | grep "RBAC"
```

**Expected** (if you tested unauthorized access):
```
RBAC: access denied
```

### Step 3: Monitor certificate rotation events

```bash
kubectl get events -n istio-system --field-selector reason=SecretRotated --watch
```

**What to Watch For**: Events indicating certificates were successfully rotated (happens every 24 hours).

**Keep this running** in the background to observe rotation.

### Step 4: Query mTLS metrics in Prometheus

In your browser, open Prometheus (http://localhost:9090) and run:

```promql
# mTLS connection success rate
sum(rate(istio_requests_total{security_policy="mutual_tls",response_code="200"}[5m])) 
/ 
sum(rate(istio_requests_total[5m])) * 100
```

**Expected**: Should be close to 100% (all requests using mTLS).

**Another useful query** - mTLS vs plaintext traffic:
```promql
sum(istio_requests_total{security_policy="mutual_tls"}) by (security_policy)
```

---

## 🧠 Active Recall Check #3

<details>
<summary>💡 <strong>Click to reveal questions - Try answering before expanding!</strong></summary>

**Question 1: Conceptual Understanding**
Explain the relationship between PeerAuthentication, DestinationRule, and AuthorizationPolicy. Why do we need all three for complete zero-trust security?

**Question 2: Parameter Reasoning**
We used `mode: STRICT` in PeerAuthentication. What would happen if we used `mode: PERMISSIVE` instead? When would PERMISSIVE be appropriate? What security risks does it introduce?

**Question 3: Connection to Previous Learning**
How do the Envoy sidecars (Phase 2) and Istio's CA (Phase 4) work together to provide automatic mTLS? Trace the certificate lifecycle from pod creation to certificate rotation.

**Question 4: Real-World Application**
You're migrating a legacy application with 50 microservices to Istio. Some services are external (database, payment gateway) and don't have sidecars. Design a phased mTLS rollout strategy. What mode do you start with? When do you move to STRICT? How do you handle the external services?

</details>

<details>
<summary>🔍 <strong>Answer Key</strong></summary>

**Answer 1**:
The three policies provide different security layers:

**PeerAuthentication** (Server-side):
- Controls: Does the server *accept* mTLS?
- Modes: STRICT (mTLS only), PERMISSIVE (mTLS or plaintext), DISABLE (plaintext only)
- Scope: Workload that receives traffic
- Example: "details service must receive mTLS"

**DestinationRule** (Client-side):
- Controls: Does the client *send* mTLS?
- Modes: ISTIO_MUTUAL, MUTUAL, SIMPLE, DISABLE
- Scope: Workload that sends traffic
- Example: "productpage must send mTLS to details"

**AuthorizationPolicy** (Access Control):
- Controls: Is this specific identity *allowed* to access this resource?
- Based on: SPIFFE identity, HTTP methods, paths, headers
- Scope: Fine-grained access control
- Example: "Only productpage ServiceAccount can GET /details"

**Why all three**:
1. **PeerAuthentication + DestinationRule**: Establishes encrypted channel and verifies identity
2. **AuthorizationPolicy**: Enforces least-privilege access on top of identity

Without PeerAuthentication: No encryption, identity not verified
Without DestinationRule: Client might not use mTLS (depends on auto-mTLS)
Without AuthorizationPolicy: All authenticated services can access everything (not zero-trust)

**Answer 2**:
**PERMISSIVE mode behavior**:
- Server accepts BOTH mTLS and plaintext connections
- Sidecars automatically upgrade to mTLS when possible
- Legacy clients without sidecars can still connect with plaintext

**When PERMISSIVE is appropriate**:
- **Migration phase**: Gradually rolling out Istio to services
- **Incremental adoption**: Some services have sidecars, others don't yet
- **External integrations**: Third-party services sending plaintext
- **Debugging**: Temporarily allow plaintext to troubleshoot connectivity

**Security risks**:
1. **Downgrade attacks**: Attacker might trick system into accepting plaintext
2. **Mixed security posture**: Some traffic encrypted, some not
3. **Compliance violations**: Regulations may require encryption-at-rest and in-transit
4. **False sense of security**: Assuming all traffic is encrypted when some isn't

**Migration strategy**:
```
Week 1-2: PERMISSIVE + monitoring (track mTLS %)
Week 3: PERMISSIVE + alert on plaintext
Week 4: STRICT (after 95%+ mTLS)
```

**Answer 3**:
**Certificate Lifecycle**:

**1. Pod Creation (Initial Certificate Issuance)**:
- Kubernetes creates pod with ServiceAccount token mounted at `/var/run/secrets/kubernetes.io/serviceaccount/token`
- Istio mutating webhook injects init container (istio-init) and sidecar (istio-proxy)
- istio-init sets up iptables rules to redirect traffic to Envoy (port 15001)
- Envoy starts, sends SDS (Secret Discovery Service) request to istiod
- Request includes: ServiceAccount JWT token
- istiod validates JWT, extracts identity: `cluster.local/ns/default/sa/bookinfo-productpage`
- istiod generates CSR (Certificate Signing Request) for this identity
- istiod (acting as CA) signs CSR with its private key
- istiod sends certificate + private key to Envoy via SDS
- Envoy stores cert in memory (not on disk for security)
- Time: ~1-2 seconds from pod start to cert ready

**2. mTLS Connection Establishment**:
- Request: productpage → details
- iptables redirects outbound to Envoy (port 15001)
- Envoy initiates TLS handshake:
  - Client Hello: "I'm productpage (cert)"
  - Server Hello: "I'm details (cert)"
  - Both verify peer cert against ROOTCA
  - Establish encrypted channel
- Application sees normal HTTP on localhost

**3. Certificate Rotation (Every 24 Hours)**:
- Envoy sets timer based on cert's NotAfter (default: 24h validity)
- Before expiry (grace period: 25% of validity, ~6h before), Envoy requests new cert from istiod via SDS
- istiod issues new certificate with same identity
- Envoy performs hot-swap: new connections use new cert, existing connections drain
- Old cert disposed after grace period
- Zero downtime: Applications unaware of rotation

**Components Involved**:
- **Kubernetes**: Provides ServiceAccount tokens
- **istiod**: Certificate Authority, signs certificates
- **Envoy (istio-proxy)**: Requests certs, performs TLS, rotates certs
- **SDS (Secret Discovery Service)**: gRPC protocol between Envoy ↔ istiod

**Answer 4**:
**Phased mTLS Rollout Strategy for 50 Microservices with External Dependencies**:

**Phase 1: Assessment (Week 1)**:
```bash
# Audit current state
kubectl get pods --all-namespaces -o json | \
  jq '.items[] | select(.spec.containers | length > 1) | .metadata.name'
# Count sidecars already present

# Identify external dependencies
# Create inventory: database (PostgreSQL - external), payment-gateway (external), 48 internal services
```

**Phase 2: PERMISSIVE Mode Rollout (Week 2-4)**:
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: PERMISSIVE  # Start here
```

**Why**: Allows coexistence of mTLS (sidecar services) and plaintext (external services, not-yet-migrated services)

**Monitoring**:
```promql
# Track mTLS adoption
sum(istio_requests_total{security_policy="mutual_tls"}) / sum(istio_requests_total) * 100
```

**Target**: 80%+ of requests using mTLS before Phase 3

**Phase 3: Handle External Services (Week 5)**:

**For external database (PostgreSQL)**:
```yaml
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: external-postgres
spec:
  hosts:
  - postgres.external.com
  ports:
  - number: 5432
    name: tcp
    protocol: TCP
  location: MESH_EXTERNAL
  resolution: DNS
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: external-postgres
spec:
  host: postgres.external.com
  trafficPolicy:
    tls:
      mode: DISABLE  # No mTLS to external service
```

**For external payment gateway (HTTPS)**:
```yaml
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: payment-gateway
spec:
  hosts:
  - pay.gateway.com
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  location: MESH_EXTERNAL
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: payment-gateway
spec:
  host: pay.gateway.com
  trafficPolicy:
    tls:
      mode: SIMPLE  # Standard TLS (not mTLS)
```

**Phase 4: Namespace-by-Namespace STRICT Mode (Week 6-10)**:

**Start with low-risk namespaces**:
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: logging  # Low-risk namespace first
spec:
  mtls:
    mode: STRICT
```

**Monitor for connection failures**: If failures occur, rollback to PERMISSIVE and investigate.

**Gradually enable for remaining namespaces**:
```
Week 6: logging, monitoring namespaces (STRICT)
Week 7: staging environments (STRICT)
Week 8: Low-traffic production services (STRICT)
Week 9: High-traffic production services (STRICT)
Week 10: Critical path services (STRICT)
```

**Phase 5: Mesh-Wide STRICT Mode (Week 11)**:
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system  # Mesh-wide
spec:
  mtls:
    mode: STRICT
```

**Exception Handling**:
For workloads that MUST accept plaintext (e.g., legacy service being sunset):
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: legacy-service-exception
  namespace: production
spec:
  selector:
    matchLabels:
      app: legacy-billing
  mtls:
    mode: PERMISSIVE  # Override mesh-wide STRICT
```

**Rollback Plan**:
At each phase, if error rate > 0.1%:
1. Rollback to previous mode immediately
2. Check logs: `kubectl logs -l app=<service> -c istio-proxy | grep "TLS error"`
3. Identify root cause (missing sidecar? external service?)
4. Fix issue, re-enable STRICT

**Success Metrics**:
- mTLS coverage: 100% for internal services
- Error rate: < 0.01% increase after STRICT mode
- External service connectivity: 100% (using DISABLE/SIMPLE modes)
- Certificate rotation: 100% success rate
- Time to rollout: 11 weeks (conservative, can be faster for smaller fleets)

</details>

---

**📝 Summary**: We've implemented comprehensive zero-trust security with STRICT mutual TLS encryption for service-to-service communication and fine-grained authorization policies controlling access based on workload identity. The mesh now enforces encrypted communication and least-privilege access without application code changes.

**🔄 Spaced Review**: Recall Phase 2-3 concepts:
- What are the four Bookinfo services? (productpage, details, reviews, ratings)
- Which observability tool shows service topology? (Kiali)
- What does Prometheus collect? (Metrics - request rate, latency, errors)

**Progress Update**: Implementation ⭐⭐⭐⭐⭐ | Troubleshooting ⭐⚪⚪⚪⚪

---

# 🧪 Phase 5: Comprehensive Testing & Validation

**Duration**: 20-25 minutes

## 🤔 Prediction Challenge (60 seconds)

Before testing, predict:
- What categories of tests should we run? (Hint: Functional, security, performance, operational)
- How do we test that mTLS is actually being enforced?
- What performance overhead does service mesh add?
- How would you test failure scenarios (certificate expiration, policy misconfiguration)?

---

## **5.1 Testing Philosophy & Approach**

**Why Test Service Mesh Deployments**:
1. **Verify security controls**: Ensure mTLS and authorization policies work as designed
2. **Validate performance**: Measure mesh overhead and identify bottlenecks
3. **Confirm observability**: Check that metrics, logs, traces are captured correctly
4. **Test failure modes**: Verify graceful degradation and error handling

**Testing Pyramid**:
```
       /\
      /  \   End-to-End (E2E)
     /----\  Integration
    /------\ Unit/Component
```

**Our Test Strategy**:
- **Component Tests**: Individual Istio resources (Gateway, VirtualService)
- **Integration Tests**: Service-to-service communication
- **Security Tests**: mTLS enforcement, authorization
- **Performance Tests**: Latency, throughput with mesh overhead
- **Operational Tests**: Certificate rotation, policy updates

---

## **5.2 Automated Test Suite**

**Context**: We'll create reusable bash functions to test all aspects of the service mesh.

### Step 1: Create comprehensive test script

```bash
nano service-mesh-tests.sh
```

**Paste this content**, then **Ctrl+O**, **Enter**, **Ctrl+X**:

```bash
#!/bin/bash

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Test counters
TESTS_RUN=0
TESTS_PASSED=0
TESTS_FAILED=0

# Helper functions
print_header() {
    echo -e "\n${YELLOW}=== $1 ===${NC}\n"
}

run_test() {
    local test_name="$1"
    local test_command="$2"
    local expected_result="$3"
    
    TESTS_RUN=$((TESTS_RUN + 1))
    echo -n "Test $TESTS_RUN: $test_name... "
    
    if eval "$test_command" | grep -q "$expected_result"; then
        echo -e "${GREEN}PASSED${NC}"
        TESTS_PASSED=$((TESTS_PASSED + 1))
    else
        echo -e "${RED}FAILED${NC}"
        TESTS_FAILED=$((TESTS_FAILED + 1))
        echo "  Expected: $expected_result"
        echo "  Command: $test_command"
    fi
}

print_summary() {
    echo -e "\n${YELLOW}=== Test Summary ===${NC}"
    echo "Total Tests: $TESTS_RUN"
    echo -e "${GREEN}Passed: $TESTS_PASSED${NC}"
    echo -e "${RED}Failed: $TESTS_FAILED${NC}"
    
    if [ $TESTS_FAILED -eq 0 ]; then
        echo -e "\n${GREEN}✓ All tests passed!${NC}"
        return 0
    else
        echo -e "\n${RED}✗ Some tests failed${NC}"
        return 1
    fi
}

# Test Suite

print_header "1. Basic Connectivity Tests"

run_test "Istio control plane is running" \
    "kubectl get pods -n istio-system -l app=istiod -o jsonpath='{.items[0].status.phase}'" \
    "Running"

run_test "Ingress gateway is running" \
    "kubectl get pods -n istio-system -l app=istio-ingressgateway -o jsonpath='{.items[0].status.phase}'" \
    "Running"

run_test "All Bookinfo pods are running" \
    "kubectl get pods -l 'app in (productpage,details,reviews,ratings)' --field-selector=status.phase=Running | wc -l" \
    "6"

run_test "All pods have sidecars (2 containers)" \
    "kubectl get pod -l app=productpage -o jsonpath='{.items[0].spec.containers[*].name}'" \
    "istio-proxy"

print_header "2. Service Mesh Configuration Tests"

run_test "Gateway exists" \
    "kubectl get gateway bookinfo-gateway -o jsonpath='{.metadata.name}'" \
    "bookinfo-gateway"

run_test "VirtualService exists" \
    "kubectl get virtualservice bookinfo -o jsonpath='{.metadata.name}'" \
    "bookinfo"

run_test "PeerAuthentication is STRICT" \
    "kubectl get peerauthentication default -o jsonpath='{.spec.mtls.mode}'" \
    "STRICT"

run_test "DestinationRule for mTLS exists" \
    "kubectl get destinationrule default -o jsonpath='{.spec.trafficPolicy.tls.mode}'" \
    "ISTIO_MUTUAL"

run_test "Authorization policies exist" \
    "kubectl get authorizationpolicy | wc -l" \
    "4"

print_header "3. mTLS Security Tests"

run_test "mTLS is enabled for productpage" \
    "istioctl authn tls-check \$(kubectl get pod -l app=productpage -o jsonpath='{.items[0].metadata.name}').default details.default.svc.cluster.local:9080 2>/dev/null | grep details" \
    "STRICT"

run_test "Certificates are valid" \
    "istioctl proxy-config secret \$(kubectl get pod -l app=productpage -o jsonpath='{.items[0].metadata.name}') -o json | jq '.dynamicActiveSecrets[0].secret.tlsCertificate != null'" \
    "true"

run_test "Service-to-service communication works with mTLS" \
    "kubectl exec \$(kubectl get pod -l app=productpage -o jsonpath='{.items[0].metadata.name}') -c productpage -- curl -s -o /dev/null -w '%{http_code}' http://details:9080/details/0" \
    "200"

print_header "4. Authorization Policy Tests"

run_test "Authorized access works (productpage → details)" \
    "kubectl exec \$(kubectl get pod -l app=productpage -o jsonpath='{.items[0].metadata.name}') -c productpage -- curl -s -o /dev/null -w '%{http_code}' http://details:9080/details/0" \
    "200"

# Note: This test creates and deletes a pod, which takes time
echo "Test $((TESTS_RUN + 1)): Unauthorized access is blocked (external pod → details)... "
TESTS_RUN=$((TESTS_RUN + 1))
UNAUTHORIZED_RESULT=$(kubectl run test-unauthorized --image=curlimages/curl --rm -i --restart=Never --labels="sidecar.istio.io/inject=false" --command -- curl -s -o /dev/null -w '%{http_code}' --max-time 5 http://details.default:9080/details/0 2>/dev/null || echo "TIMEOUT")
if [[ "$UNAUTHORIZED_RESULT" == *"TIMEOUT"* ]] || [[ "$UNAUTHORIZED_RESULT" == "000" ]]; then
    echo -e "${GREEN}PASSED${NC} (connection blocked as expected)"
    TESTS_PASSED=$((TESTS_PASSED + 1))
else
    echo -e "${RED}FAILED${NC} (expected connection to be blocked, got: $UNAUTHORIZED_RESULT)"
    TESTS_FAILED=$((TESTS_FAILED + 1))
fi

print_header "5. Observability Tests"

run_test "Prometheus is collecting metrics" \
    "kubectl exec -n istio-system deployment/prometheus -- wget -qO- 'http://localhost:9090/api/v1/query?query=istio_requests_total' | grep '\"status\":\"success\"'" \
    "success"

run_test "Grafana dashboards are available" \
    "kubectl get cm -n istio-system istio-grafana-dashboards -o jsonpath='{.metadata.name}'" \
    "istio-grafana-dashboards"

run_test "Jaeger is running" \
    "kubectl get pods -n istio-system -l app=jaeger -o jsonpath='{.items[0].status.phase}'" \
    "Running"

run_test "Kiali is running" \
    "kubectl get pods -n istio-system -l app=kiali -o jsonpath='{.items[0].status.phase}'" \
    "Running"

print_header "6. Performance Tests"

echo "Test $((TESTS_RUN + 1)): Application responds within acceptable latency... "
TESTS_RUN=$((TESTS_RUN + 1))
LATENCY=$(kubectl exec $(kubectl get pod -l app=productpage -o jsonpath='{.items[0].metadata.name}') -c productpage -- time -f "%E" curl -s -o /dev/null http://details:9080/details/0 2>&1 | grep -o "[0-9:\.]*")
LATENCY_MS=$(echo "$LATENCY" | awk -F: '{print ($1 * 60 + $2) * 1000}')
if (( $(echo "$LATENCY_MS < 500" | bc -l) )); then
    echo -e "${GREEN}PASSED${NC} (${LATENCY_MS}ms < 500ms threshold)"
    TESTS_PASSED=$((TESTS_PASSED + 1))
else
    echo -e "${RED}FAILED${NC} (${LATENCY_MS}ms >= 500ms threshold)"
    TESTS_FAILED=$((TESTS_FAILED + 1))
fi

print_header "7. End-to-End Application Tests"

run_test "Application is accessible via ingress gateway" \
    "curl -s 'http://${GATEWAY_URL}/productpage' -o /dev/null -w '%{http_code}'" \
    "200"

run_test "Application returns expected content" \
    "curl -s 'http://${GATEWAY_URL}/productpage'" \
    "Simple Bookstore App"

# Print final summary
print_summary
```

### Step 2: Make script executable and run tests

```bash
chmod +x service-mesh-tests.sh
./service-mesh-tests.sh
```

**Expected Output**:
```
=== 1. Basic Connectivity Tests ===

Test 1: Istio control plane is running... PASSED
Test 2: Ingress gateway is running... PASSED
Test 3: All Bookinfo pods are running... PASSED
Test 4: All pods have sidecars (2 containers)... PASSED

=== 2. Service Mesh Configuration Tests ===

Test 5: Gateway exists... PASSED
...

=== Test Summary ===
Total Tests: 20
Passed: 20
Failed: 0

✓ All tests passed!
```

**If any tests fail**, review the failure details and use the troubleshooting section (Phase 6) to diagnose.

---

## **5.3 Manual Verification Checklist**

Run these manual checks to complement automated tests:

### ✅ Configuration Validation

```bash
# Validate all Istio configurations
istioctl analyze
```

**Expected**: `✔ No validation issues found`

### ✅ Proxy Synchronization

```bash
# Check all proxies are synced with control plane
istioctl proxy-status
```

**Expected**: All proxies show `SYNCED` for CDS, LDS, EDS, RDS, ECDS

### ✅ Certificate Health

```bash
# Check certificate expiration for all workloads
for pod in $(kubectl get pods -l 'app in (productpage,details,reviews,ratings)' -o jsonpath='{.items[*].metadata.name}'); do
    echo "Checking $pod:"
    istioctl proxy-config secret $pod -o json | \
    jq -r '.dynamicActiveSecrets[] | select(.name == "default") | 
    "  Cert Valid Until: " + .secret.tlsCertificate.certificateChain.inlineBytes' | \
    base64 -d 2>/dev/null | openssl x509 -noout -enddate 2>/dev/null || echo "  Unable to decode"
done
```

**Expected**: All certificates valid for ~24 hours from now

### ✅ Observability Data Flow

**Check Prometheus targets**:
```bash
kubectl port-forward -n istio-system svc/prometheus 9090:9090 &
sleep 2
curl -s http://localhost:9090/api/v1/targets | jq '.data.activeTargets | length'
pkill -f "port-forward.*prometheus"
```

**Expected**: Multiple active targets (istiod, envoy sidecars)

**Verify traces are being collected**:
```bash
# Generate traffic
for i in {1..10}; do curl -s "http://${GATEWAY_URL}/productpage" > /dev/null; done

# Check Jaeger has traces
kubectl port-forward -n istio-system svc/jaeger-query 16686:16686 &
sleep 2
curl -s 'http://localhost:16686/api/traces?service=productpage.default' | jq '.data | length'
pkill -f "port-forward.*jaeger"
```

**Expected**: At least 10 traces returned

---

## **5.4 Performance Impact Analysis**

**Context**: Service mesh adds latency and CPU overhead. We'll measure the impact.

### Benchmark 1: Latency Overhead

**Test without mesh** (baseline - requires non-sidecar pod):
```bash
kubectl run perf-baseline --image=curlimages/curl --rm -i --restart=Never \
  --labels="sidecar.istio.io/inject=false" --command -- \
  sh -c 'for i in $(seq 1 100); do time curl -s http://productpage.default:9080/productpage > /dev/null; done' 2>&1 | \
  grep real | awk '{sum += $2; count++} END {print "Average without mesh: " sum/count " seconds"}'
```

**Test with mesh**:
```bash
kubectl exec $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') -c ratings -- \
  sh -c 'for i in $(seq 1 100); do time wget -q -O- http://productpage.default:9080/productpage > /dev/null; done' 2>&1 | \
  grep real | awk '{sum += $2; count++} END {print "Average with mesh: " sum/count " seconds"}'
```

**Typical Results**:
- Without mesh: ~50-100ms average
- With mesh: ~70-120ms average
- **Overhead: 20-30%** (acceptable for security benefits)

### Benchmark 2: Resource Usage

**Check sidecar CPU usage**:
```bash
kubectl top pod -l 'app in (productpage,details,reviews,ratings)' --containers
```

**Expected Output**:
```
POD                    NAME           CPU(cores)   MEMORY(bytes)
productpage-v1-xxx     productpage    5m           50Mi
productpage-v1-xxx     istio-proxy    10m          60Mi
```

**Typical Sidecar Resource Usage**:
- **CPU**: 10-50m (millicores) at moderate load
- **Memory**: 50-100Mi baseline, up to 200Mi under heavy load
- **Network**: Minimal overhead (TLS handshake cost amortized)

### Benchmark 3: Throughput Impact

**Generate sustained load**:
```bash
kubectl run load-test --image=williamyeh/wrk --rm -i --restart=Never -- \
  -t4 -c100 -d30s "http://${GATEWAY_URL}/productpage"
```

**Output Analysis**:
- **Requests/sec**: Compare with vs without mesh (typically 5-15% reduction)
- **Latency distribution**: p50, p99, p999
- **Transfer rate**: Should be consistent

---

## **5.5 Edge Case & Failure Mode Testing**

### Test 1: Certificate Expiration Simulation

**Shorten certificate TTL for testing** (⚠️ don't do in production):
```bash
kubectl patch configmap istio -n istio-system --type merge -p '{"data":{"mesh":"certificateTTL: 1m"}}'
kubectl rollout restart deployment/istiod -n istio-system
```

**Wait 2 minutes and verify rotation occurred**:
```bash
# Check certificate age
istioctl proxy-config secret $(kubectl get pod -l app=productpage -o jsonpath='{.items[0].metadata.name}') | grep default
```

**Restore default TTL**:
```bash
kubectl patch configmap istio -n istio-system --type merge -p '{"data":{"mesh":"certificateTTL: 24h"}}'
kubectl rollout restart deployment/istiod -n istio-system
```

### Test 2: Policy Misconfiguration

**Intentionally break authorization policy**:
```bash
kubectl patch authorizationpolicy details-policy --type merge -p '{"spec":{"rules":[{"from":[{"source":{"principals":["cluster.local/ns/default/sa/nonexistent"]}}]}]}}'
```

**Verify access is blocked**:
```bash
kubectl exec $(kubectl get pod -l app=productpage -o jsonpath='{.items[0].metadata.name}') -c productpage -- \
  curl -s -o /dev/null -w '%{http_code}' http://details:9080/details/0
```

**Expected**: `403` (Forbidden)

**Fix the policy**:
```bash
kubectl apply -f authorization-policy.yaml
```

### Test 3: Control Plane Failure

**Scale down istiod**:
```bash
kubectl scale deployment istiod -n istio-system --replicas=0
```

**Verify existing connections still work** (data plane continues):
```bash
curl -s "http://${GATEWAY_URL}/productpage" | grep -o "<title>.*</title>"
```

**Expected**: Still returns `<title>Simple Bookstore App</title>`

**Why it works**: Envoy proxies cache configuration. Existing workloads continue operating even if control plane is down.

**Restore control plane**:
```bash
kubectl scale deployment istiod -n istio-system --replicas=1
```

---

## **5.6 Testing Checklist Summary**

Before proceeding, verify:

- [ ] All automated tests pass (≥90% success rate)
- [ ] Configuration validation shows no issues (`istioctl analyze`)
- [ ] All proxies are synced (`istioctl proxy-status`)
- [ ] Certificates are valid and rotating
- [ ] mTLS enforcement works (unauthorized access blocked)
- [ ] Authorization policies enforce least-privilege
- [ ] Observability data is being collected (metrics, traces, logs)
- [ ] Performance overhead is acceptable (<30% latency increase)
- [ ] Application functions correctly end-to-end
- [ ] Edge cases handled gracefully (cert rotation, policy updates)

**Progress Update**: Troubleshooting ⭐⭐⭐⭐⭐ | Production ⭐⚪⚪⚪⚪

---

# 🔧 Phase 6: Troubleshooting Guide

**Duration**: Reference (as needed)

## **6.1 Systematic Troubleshooting Approach**

When issues occur, follow this methodical process:

1. **Identify**: What's the symptom? (error message, unexpected behavior, performance issue)
2. **Gather**: Collect logs, metrics, configuration
3. **Hypothesize**: What could cause this? (most likely first)
4. **Test**: Validate hypothesis with targeted checks
5. **Resolve**: Apply fix
6. **Document**: Record issue and solution for future reference

---

## **6.2 Diagnostic Commands**

**Quick Health Check Function**:

```bash
check_mesh_health() {
    echo "=== Istio Mesh Health Check ==="
    
    echo -e "\n1. Control Plane Status:"
    kubectl get pods -n istio-system
    
    echo -e "\n2. Sidecar Injection Status:"
    kubectl get namespace default -o jsonpath='{.metadata.labels.istio-injection}'
    
    echo -e "\n3. Proxy Synchronization:"
    istioctl proxy-status | head -5
    
    echo -e "\n4. Configuration Validation:"
    istioctl analyze
    
    echo -e "\n5. Recent Errors in Control Plane:"
    kubectl logs -n istio-system -l app=istiod --tail=20 | grep -i error
    
    echo -e "\n6. mTLS Status:"
    kubectl get peerauthentication
    
    echo -e "\n7. Recent Events:"
    kubectl get events --sort-by='.lastTimestamp' | tail -10
}

# Run the health check
check_mesh_health
```

---

## **6.3 Common Error Patterns**

<details>
<summary>🔧 <strong>Connection Errors</strong></summary>

### Issue: "Connection refused" or "Connection timeout"

**Symptoms**:
```
curl: (7) Failed to connect to details port 9080: Connection refused
curl: (28) Connection timed out after 5000 milliseconds
```

**Diagnostic Steps**:
```bash
# 1. Check if target pod is running
kubectl get pod -l app=details

# 2. Check if service exists and has endpoints
kubectl get svc details
kubectl get endpoints details

# 3. Check sidecar injection
kubectl get pod -l app=details -o jsonpath='{.items[0].spec.containers[*].name}'
# Should show: details istio-proxy

# 4. Check iptables rules in sidecar
kubectl exec <pod-name> -c istio-proxy -- iptables -t nat -L -n -v | grep 15001

# 5. Check for network policies blocking traffic
kubectl get networkpolicies
```

**Common Causes & Fixes**:

| Cause | Solution |
|-------|----------|
| Pod not running | Check pod logs: `kubectl logs <pod> -c <container>` |
| No sidecar injected | Verify namespace label, recreate pod |
| Service/Endpoints misconfigured | Check service selector matches pod labels |
| mTLS mismatch | Verify PeerAuthentication and DestinationRule align |
| Network policy blocking | Adjust NetworkPolicy or remove it |

**Prevention**:
- Always verify sidecar injection before troubleshooting connectivity
- Use `istioctl analyze` regularly
- Monitor `istio_requests_total{response_code="503"}` metric for connection failures

</details>

<details>
<summary>🔧 <strong>mTLS / Certificate Errors</strong></summary>

### Issue: "TLS handshake failed" or "certificate verify failed"

**Symptoms**:
```
upstream connect error or disconnect/reset before headers. TLS error
```

**Diagnostic Steps**:
```bash
# 1. Check mTLS mode configuration
kubectl get peerauthentication --all-namespaces
kubectl get destinationrule --all-namespaces

# 2. Verify certificate validity
istioctl proxy-config secret <pod-name>

# 3. Check for mTLS conflicts
istioctl authn tls-check <pod-name>.<namespace>

# 4. View detailed proxy logs
kubectl logs <pod-name> -c istio-proxy | grep -i tls

# 5. Check certificate chain
istioctl proxy-config secret <pod-name> -o json | \
  jq '.dynamicActiveSecrets[0].secret.tlsCertificate.certificateChain.inlineBytes' | \
  base64 -d | openssl x509 -text -noout
```

**Common Causes & Fixes**:

| Cause | Solution |
|-------|----------|
| STRICT mode with non-sidecar client | Add sidecar or use PERMISSIVE mode |
| Mismatched PeerAuth and DestinationRule | Align both configs: STRICT + ISTIO_MUTUAL |
| Certificate expired (rare) | Check cert validity, verify auto-rotation |
| Trust domain mismatch | Ensure all workloads use same trust domain |
| Clock skew | Sync system clocks (NTP) across nodes |

**Quick Fix Template**:
```yaml
# Temporarily allow both mTLS and plaintext for debugging
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: debug-permissive
  namespace: <your-namespace>
spec:
  mtls:
    mode: PERMISSIVE
```

**Prevention**:
- Monitor certificate expiration: `histogram_quantile(0.95, istio_cert_expiry_seconds)`
- Alert on TLS errors: `rate(istio_requests_total{response_code="503",connection_security_policy="mutual_tls"}[5m]) > 0.01`
- Test mTLS configuration in staging first

</details>

<details>
<summary>🔧 <strong>Authorization / RBAC Errors</strong></summary>

### Issue: "RBAC: access denied"

**Symptoms**:
```
RBAC: access denied
HTTP 403 Forbidden
```

**Diagnostic Steps**:
```bash
# 1. Check authorization policies
kubectl get authorizationpolicy --all-namespaces

# 2. View specific policy
kubectl get authorizationpolicy <policy-name> -o yaml

# 3. Check workload's ServiceAccount
kubectl get pod <pod-name> -o jsonpath='{.spec.serviceAccountName}'

# 4. View proxy access logs for RBAC decisions
kubectl logs <pod-name> -c istio-proxy | grep RBAC

# 5. Check SPIFFE identity
istioctl proxy-config secret <pod-name> -o json | \
  jq -r '.dynamicActiveSecrets[0].secret.tlsCertificate.certificateChain.inlineBytes' | \
  base64 -d | openssl x509 -text -noout | grep "URI:"
```

**Common Causes & Fixes**:

| Cause | Solution |
|-------|----------|
| ServiceAccount mismatch | Verify principals in AuthorizationPolicy match actual ServiceAccounts |
| Typo in SPIFFE identity | Check format: `cluster.local/ns/<namespace>/sa/<sa-name>` |
| Overly restrictive policy | Add source principal or use `when` conditions |
| Missing default-allow | Create allow-all policy temporarily for debugging |
| Policy not applied | Check policy status: `kubectl get authorizationpolicy` |

**Debug with Permissive Policy**:
```yaml
# Temporarily allow all access for debugging
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-all-debug
  namespace: <your-namespace>
spec:
  action: ALLOW
  rules:
  - {}
```

**Prevention**:
- Test policies in staging with traffic replay
- Use descriptive policy names indicating purpose
- Document required access patterns
- Monitor authorization denials: `rate(istio_requests_total{response_code="403"}[5m])`

</details>

<details>
<summary>🔧 <strong>Configuration / Policy Errors</strong></summary>

### Issue: "Configuration invalid" or policies not taking effect

**Symptoms**:
```
istioctl analyze shows warnings/errors
Policies exist but don't seem to work
```

**Diagnostic Steps**:
```bash
# 1. Validate all configurations
istioctl analyze --all-namespaces

# 2. Check specific resource
istioctl analyze -k <file.yaml>

# 3. View proxy configuration
istioctl proxy-config all <pod-name> --output-directory ./proxy-config

# 4. Check for conflicting policies
kubectl get virtualservice,destinationrule,gateway,authorizationpolicy --all-namespaces

# 5. Verify policy is pushed to proxies
istioctl proxy-config listener <pod-name> -o json | jq '.[] | select(.name | contains("virtualInbound"))'

# 6. Check control plane logs for configuration errors
kubectl logs -n istio-system -l app=istiod --tail=100 | grep -i error
```

**Common Causes & Fixes**:

| Cause | Solution |
|-------|----------|
| YAML syntax error | Use `kubectl apply --dry-run=client` first |
| Conflicting VirtualServices | Ensure unique host/gateway combinations |
| Stale proxy configuration | Force proxy to resync: restart pod or `istioctl proxy-config` |
| Namespace selector mismatch | Verify policy targets correct namespace/labels |
| Resource not in correct namespace | Move resource or adjust namespace references |

**Configuration Cleanup Script**:
```bash
# List all Istio configurations
kubectl get gateway,virtualservice,destinationrule,peerauthentication,authorizationpolicy \
  --all-namespaces -o wide
```

**Prevention**:
- Always run `istioctl analyze` before applying
- Use GitOps (ArgoCD, Flux) for policy management
- Implement policy review process
- Test in dev environment first

</details>

<details>
<summary>🔧 <strong>Observability / Telemetry Issues</strong></summary>

### Issue: No metrics, traces, or logs appearing

**Symptoms**:
```
Grafana dashboards empty
No traces in Jaeger
Prometheus not scraping
```

**Diagnostic Steps**:
```bash
# 1. Check observability pods
kubectl get pods -n istio-system -l 'app in (prometheus,grafana,jaeger,kiali)'

# 2. Verify telemetry configuration
kubectl get telemetry --all-namespaces

# 3. Check if Prometheus is scraping
kubectl exec -n istio-system deployment/prometheus -- wget -qO- http://localhost:9090/api/v1/targets | grep -i istio

# 4. Check sidecar is exposing metrics
kubectl exec <pod-name> -c istio-proxy -- curl -s localhost:15090/stats/prometheus | head

# 5. Verify tracing is enabled
kubectl logs <pod-name> -c istio-proxy | grep -i tracing

# 6. Check sampling rate
kubectl get telemetry -o yaml | grep -i sampling
```

**Common Causes & Fixes**:

| Cause | Solution |
|-------|----------|
| Observability add-ons not installed | Re-apply: `kubectl apply -f samples/addons/` |
| Telemetry disabled | Create Telemetry resource with providers |
| Sampling rate too low | Increase `randomSamplingPercentage` in Telemetry |
| Port-forward not active | Restart port-forward or use LoadBalancer |
| No traffic generated | Generate traffic to populate metrics |

**Quick Fix - Re-enable Telemetry**:
```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: mesh-default
  namespace: istio-system
spec:
  metrics:
  - providers:
    - name: prometheus
  tracing:
  - providers:
    - name: jaeger
    randomSamplingPercentage: 100.0
```

**Prevention**:
- Monitor observability stack health
- Set up alerts for missing metrics
- Use persistent storage for long-term metrics retention

</details>

---

## **6.4 Complete Diagnostic Script**

```bash
nano diagnose-mesh.sh
```

**Paste this content**, then **Ctrl+O**, **Enter**, **Ctrl+X**:

```bash
#!/bin/bash

echo "=== Comprehensive Istio Mesh Diagnostics ==="
echo "Generated at: $(date)"
echo ""

# 1. Cluster Info
echo "=== Cluster Information ==="
kubectl version --short
kubectl get nodes
echo ""

# 2. Istio Components
echo "=== Istio Control Plane Status ==="
kubectl get pods -n istio-system -o wide
echo ""

echo "=== Istio Version ==="
istioctl version
echo ""

# 3. Application Workloads
echo "=== Application Pods and Sidecars ==="
kubectl get pods -o custom-columns=NAME:.metadata.name,READY:.status.containerStatuses[*].ready,CONTAINERS:.spec.containers[*].name
echo ""

# 4. Istio Configuration
echo "=== Istio Resources ==="
echo "Gateways:"
kubectl get gateway --all-namespaces
echo ""
echo "VirtualServices:"
kubectl get virtualservice --all-namespaces
echo ""
echo "DestinationRules:"
kubectl get destinationrule --all-namespaces
echo ""
echo "PeerAuthentications:"
kubectl get peerauthentication --all-namespaces
echo ""
echo "AuthorizationPolicies:"
kubectl get authorizationpolicy --all-namespaces
echo ""

# 5. Proxy Status
echo "=== Proxy Synchronization Status ==="
istioctl proxy-status
echo ""

# 6. Configuration Validation
echo "=== Configuration Analysis ==="
istioctl analyze --all-namespaces
echo ""

# 7. Recent Events
echo "=== Recent Kubernetes Events ==="
kubectl get events --sort-by='.lastTimestamp' --all-namespaces | tail -20
echo ""

# 8. Control Plane Logs (errors only)
echo "=== Recent Istiod Errors ==="
kubectl logs -n istio-system -l app=istiod --tail=50 | grep -i error | tail -10
echo ""

# 9. Sidecar Logs (sample)
echo "=== Sample Sidecar Logs (Productpage) ==="
SAMPLE_POD=$(kubectl get pod -l app=productpage -o jsonpath='{.items[0].metadata.name}')
if [ -n "$SAMPLE_POD" ]; then
    kubectl logs $SAMPLE_POD -c istio-proxy --tail=20 | grep -E "(error|warning|denied)" | tail -10
fi
echo ""

# 10. mTLS Status
echo "=== mTLS Status (Sample) ==="
if [ -n "$SAMPLE_POD" ]; then
    istioctl authn tls-check $SAMPLE_POD.default 2>/dev/null | head -10
fi
echo ""

# 11. Certificate Health
echo "=== Certificate Status ==="
if [ -n "$SAMPLE_POD" ]; then
    istioctl proxy-config secret $SAMPLE_POD | grep default
fi
echo ""

# 12. Resource Usage
echo "=== Resource Usage (Top Pods) ==="
kubectl top pods --all-namespaces --containers 2>/dev/null | grep -E "(istio-proxy|istiod|NAME)" | head -15
echo ""

echo "=== Diagnostic Complete ==="
echo "For detailed investigation, run:"
echo "  kubectl logs <pod-name> -c istio-proxy"
echo "  istioctl proxy-config <pod-name> <type>"
echo "  kubectl describe pod <pod-name>"
```

### Make executable and run

```bash
chmod +x diagnose-mesh.sh
./diagnose-mesh.sh > mesh-diagnostics-$(date +%Y%m%d-%H%M%S).log
```

**Output**: Complete diagnostic report saved to timestamped file for analysis or sharing with support.

---

## **6.5 Resolution Checklist**

When troubleshooting any issue:

1. **Reproduce**: Can you consistently reproduce the issue?
2. **Isolate**: Is it affecting all services or specific ones?
3. **Recent changes**: What changed before the issue started?
4. **Logs**: Check application, sidecar, and control plane logs
5. **Configuration**: Validate with `istioctl analyze`
6. **Network**: Test basic connectivity (`ping`, `curl`)
7. **Certificates**: Verify mTLS certificates are valid
8. **Policies**: Review security and traffic policies
9. **Resources**: Check CPU, memory, disk on nodes
10. **Document**: Record findings and solution

---

## **6.6 Getting Unstuck**

If you're stuck after trying the above:

**Reset to Known-Good State**:
```bash
# Delete problematic application
kubectl delete -f samples/bookinfo/platform/kube/bookinfo.yaml

# Recreate it
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml

# Wait for pods
kubectl wait --for=condition=Ready pod --all --timeout=300s
```

**Consult Official Resources**:
- Istio Documentation: https://istio.io/latest/docs/
- Istio GitHub Issues: https://github.com/istio/istio/issues
- Istio Slack: https://istio.slack.com

**Simplify the Problem**:
- Remove authorization policies temporarily
- Switch to PERMISSIVE mTLS
- Test with simple curl commands
- Use minimal configuration

---

**Progress Update**: Troubleshooting ⭐⭐⭐⭐⭐ | Production ⭐⭐⚪⚪⚪

---

# 🧹 Phase 7: Cleanup

**Duration**: 5-10 minutes

## 🤔 Prediction Challenge (30 seconds)

Before cleaning up, predict:
- What's the correct order for removing components? (Hint: Applications before infrastructure)
- What should we preserve? (Hint: Diagnostics, logs for analysis)
- What are the risks of improper cleanup? (Hint: Orphaned resources, namespace stuck in Terminating)

---

## **7.1 Cleanup Philosophy**

**Why Proper Cleanup Matters**:
1. **Resource reclamation**: Free CPU, memory, storage
2. **Cost reduction**: Especially in cloud environments (LoadBalancers, volumes)
3. **Clean slate**: Prevent interference with future labs
4. **Learning**: Understanding dependencies reinforces architecture knowledge

**Cleanup Principles**:
- Top-down: Applications → Istio components → CRDs
- Verify each step before proceeding
- Save diagnostic data before deletion
- Preserve learning artifacts (notes, scripts)

---

## **7.2 Pre-Cleanup Documentation**

**Context**: Capture current state before cleanup for learning and future reference.

### Step 1: Export current configuration

```bash
mkdir -p istio-lab-backup-$(date +%Y%m%d)
cd istio-lab-backup-$(date +%Y%m%d)

# Export all Istio configurations
kubectl get gateway,virtualservice,destinationrule,peerauthentication,authorizationpolicy,telemetry \
  --all-namespaces -o yaml > istio-configs.yaml

# Export application manifests
kubectl get deployment,service,serviceaccount -n default -o yaml > bookinfo-app.yaml

# Export cluster state
kubectl get pods --all-namespaces -o wide > cluster-pods.txt
kubectl get svc --all-namespaces -o wide > cluster-services.txt

echo "✅ Configuration backed up to $(pwd)"
cd ..
```

### Step 2: Run final diagnostics

```bash
./diagnose-mesh.sh > final-diagnostics-$(date +%Y%m%d-%H%M%S).log
echo "✅ Final diagnostics saved"
```

---

## **7.3 Safe Cleanup Function**

### Create reusable cleanup script

```bash
nano cleanup-mesh.sh
```

**Paste this content**, then **Ctrl+O**, **Enter**, **Ctrl+X**:

```bash
#!/bin/bash

set -e  # Exit on error

# Colors
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m'

echo -e "${YELLOW}=== Istio Service Mesh Cleanup ===${NC}\n"

# Function to wait for resource deletion
wait_for_deletion() {
    local resource_type=$1
    local namespace=$2
    local max_wait=60
    local count=0
    
    while kubectl get $resource_type -n $namespace 2>/dev/null | grep -q .; do
        if [ $count -ge $max_wait ]; then
            echo -e "${RED}Timeout waiting for $resource_type deletion${NC}"
            return 1
        fi
        echo -n "."
        sleep 2
        ((count++))
    done
    echo ""
}

# 1. Stop port-forwarding processes
echo "1. Stopping port-forwarding processes..."
pkill -f "kubectl port-forward" || true
echo -e "${GREEN}✓ Port-forwarding stopped${NC}\n"

# 2. Stop traffic generation
echo "2. Stopping traffic generation..."
pkill -f "watch.*curl.*productpage" || true
pkill -f "while.*curl.*productpage" || true
echo -e "${GREEN}✓ Traffic generation stopped${NC}\n"

# 3. Remove sample application
echo "3. Removing Bookinfo application..."
kubectl delete -f istio-1.28.2/samples/bookinfo/platform/kube/bookinfo.yaml --ignore-not-found=true
wait_for_deletion "pod" "default"
echo -e "${GREEN}✓ Application removed${NC}\n"

# 4. Remove Istio configurations
echo "4. Removing Istio networking configurations..."
kubectl delete -f istio-1.28.2/samples/bookinfo/networking/bookinfo-gateway.yaml --ignore-not-found=true
kubectl delete -f strict-mtls.yaml --ignore-not-found=true 2>/dev/null || true
kubectl delete -f destination-rule-mtls.yaml --ignore-not-found=true 2>/dev/null || true
kubectl delete -f authorization-policy.yaml --ignore-not-found=true 2>/dev/null || true
kubectl delete -f telemetry-config.yaml --ignore-not-found=true 2>/dev/null || true
echo -e "${GREEN}✓ Istio configurations removed${NC}\n"

# 5. Remove observability add-ons
echo "5. Removing observability add-ons..."
kubectl delete -f istio-1.28.2/samples/addons/ --ignore-not-found=true 2>/dev/null || true
wait_for_deletion "pod" "istio-system"
echo -e "${GREEN}✓ Observability add-ons removed${NC}\n"

# 6. Remove sidecar injection label
echo "6. Removing sidecar injection label..."
kubectl label namespace default istio-injection- --ignore-not-found=true
echo -e "${GREEN}✓ Injection label removed${NC}\n"

# 7. Uninstall Istio
echo "7. Uninstalling Istio control plane..."
echo "   This may take 1-2 minutes..."
istioctl uninstall --purge -y 2>/dev/null || echo "Istio may already be uninstalled"
echo -e "${GREEN}✓ Istio uninstalled${NC}\n"

# 8. Delete istio-system namespace
echo "8. Deleting istio-system namespace..."
kubectl delete namespace istio-system --ignore-not-found=true
# Wait up to 60 seconds for namespace deletion
echo -n "   Waiting for namespace deletion"
for i in {1..30}; do
    if ! kubectl get namespace istio-system 2>/dev/null | grep -q istio-system; then
        break
    fi
    echo -n "."
    sleep 2
done
echo ""
echo -e "${GREEN}✓ Namespace deleted${NC}\n"

# 9. Remove test pods (if any)
echo "9. Cleaning up test pods..."
kubectl delete pod test-nginx test-plaintext test-unauthorized load-test --ignore-not-found=true 2>/dev/null || true
echo -e "${GREEN}✓ Test pods removed${NC}\n"

echo -e "${YELLOW}=== Cleanup Summary ===${NC}"
echo "Removed:"
echo "  - Bookinfo sample application"
echo "  - Istio networking configurations (Gateway, VirtualService, DestinationRule)"
echo "  - Security policies (PeerAuthentication, AuthorizationPolicy)"
echo "  - Observability add-ons (Prometheus, Grafana, Jaeger, Kiali)"
echo "  - Istio control plane and data plane components"
echo "  - istio-system namespace"
echo ""
echo "Preserved:"
echo "  - Kubernetes cluster"
echo "  - Other namespaces and resources"
echo "  - Backup configurations in istio-lab-backup-* directories"
echo ""

# 10. Verification
echo -e "${YELLOW}=== Post-Cleanup Verification ===${NC}"

echo "1. Istio components:"
ISTIO_PODS=$(kubectl get pods -n istio-system 2>/dev/null | wc -l)
if [ $ISTIO_PODS -eq 0 ]; then
    echo -e "   ${GREEN}✓ No Istio pods found${NC}"
else
    echo -e "   ${RED}✗ Istio pods still present: $ISTIO_PODS${NC}"
fi

echo "2. Istio namespace:"
if kubectl get namespace istio-system 2>/dev/null | grep -q istio-system; then
    echo -e "   ${YELLOW}⚠ istio-system namespace still exists (may be terminating)${NC}"
else
    echo -e "   ${GREEN}✓ istio-system namespace removed${NC}"
fi

echo "3. Application pods:"
APP_PODS=$(kubectl get pods -n default 2>/dev/null | grep -E "(productpage|details|reviews|ratings)" | wc -l)
if [ $APP_PODS -eq 0 ]; then
    echo -e "   ${GREEN}✓ No application pods found${NC}"
else
    echo -e "   ${RED}✗ Application pods still present: $APP_PODS${NC}"
fi

echo "4. CRDs (Custom Resource Definitions):"
ISTIO_CRDS=$(kubectl get crd 2>/dev/null | grep istio.io | wc -l)
if [ $ISTIO_CRDS -eq 0 ]; then
    echo -e "   ${GREEN}✓ Istio CRDs removed${NC}"
else
    echo -e "   ${YELLOW}⚠ Istio CRDs still present: $ISTIO_CRDS (normal after --purge)${NC}"
fi

echo ""
echo -e "${GREEN}✓✓✓ Cleanup Complete! ✓✓✓${NC}"
echo ""
echo "Your cluster is now ready for new deployments or labs."
echo "Backup files preserved in: istio-lab-backup-* directories"
```

---

## **7.4 Execute Cleanup**

### Make script executable and run

```bash
chmod +x cleanup-mesh.sh
./cleanup-mesh.sh
```

**Expected Output**:
```
=== Istio Service Mesh Cleanup ===

1. Stopping port-forwarding processes...
✓ Port-forwarding stopped

2. Stopping traffic generation...
✓ Traffic generation stopped

3. Removing Bookinfo application...
service "details" deleted
deployment.apps "details-v1" deleted
...
✓ Application removed

...

✓✓✓ Cleanup Complete! ✓✓✓
```

---

## **7.5 Manual Verification Steps**

### Verify all components removed

```bash
# Check for any remaining Istio pods
kubectl get pods --all-namespaces | grep istio

# Check for remaining application pods
kubectl get pods -n default

# Check for Istio CRDs (some may remain, this is normal)
kubectl get crd | grep istio

# Check for Istio services
kubectl get svc --all-namespaces | grep istio

# Check cluster node status
kubectl get nodes
```

**Expected**: No Istio or Bookinfo resources found (CRDs may remain after `--purge`, this is normal).

---

## **7.6 Optional: Complete CRD Cleanup**

**Only if you want to remove ALL Istio traces** (⚠️ this is destructive):

```bash
# Remove all Istio CRDs
kubectl get crd | grep istio.io | awk '{print $1}' | xargs kubectl delete crd

# Verify removal
kubectl get crd | grep istio
```

**Expected**: No output (all Istio CRDs removed)

> ⚠️ **WARNING**: Only do this if you're completely done with Istio. CRD removal is cluster-wide and affects all namespaces.

---

## **7.7 Cleanup Verification Checklist**

Confirm all items before considering cleanup complete:

- [ ] No Istio pods running (`kubectl get pods -n istio-system` returns "No resources found")
- [ ] No Bookinfo pods running (`kubectl get pods -n default | grep bookinfo` returns nothing)
- [ ] No port-forwarding processes active (`ps aux | grep port-forward`)
- [ ] No Istio services remain (`kubectl get svc --all-namespaces | grep istio`)
- [ ] istio-system namespace deleted or empty
- [ ] Backup files saved for future reference
- [ ] Diagnostic logs preserved
- [ ] Scripts saved for reuse

---

**📝 Summary**: The cluster has been restored to its pre-lab state. All Istio components, sample applications, and configurations have been removed cleanly. Backup configurations and diagnostic logs are preserved for future reference and learning.

**Progress Update**: Production ⭐⭐⭐⭐⚪

---

# 🏆 Phase 8: Assessment & Mastery Validation

**Duration**: 30-45 minutes

## **8.1 Reflection Exercise (5 minutes)**

Before proceeding to formal assessment, take time to internalize what you've learned:

### Visualize the Complete Workflow

Close your eyes and mentally trace a request through the mesh:
1. External request arrives at ingress gateway
2. Gateway routes to productpage service
3. Envoy sidecar intercepts (outbound from gateway, inbound to productpage)
4. mTLS handshake (certificate exchange)
5. Authorization policy check
6. Request forwarded to productpage application
7. Application calls details service (repeat interception, mTLS, authorization)
8. Response flows back through sidecars
9. Metrics collected, traces generated at each hop

**Can you explain each step to someone else?**

### Review Your Checklist

- ✅ Installed Istio 1.28.2 with default profile
- ✅ Deployed multi-service application with automatic sidecar injection
- ✅ Configured ingress gateway and routing rules
- ✅ Deployed observability stack (Prometheus, Grafana, Jaeger, Kiali)
- ✅ Enabled comprehensive telemetry collection
- ✅ Implemented STRICT mTLS for zero-trust networking
- ✅ Created authorization policies for least-privilege access
- ✅ Validated security controls through testing
- ✅ Measured performance impact of service mesh
- ✅ Troubleshot common issues and edge cases
- ✅ Cleaned up environment properly

---

## **8.2 Teach-Back Exercise (3-5 minutes)**

Explain the following to someone (or record yourself) as if teaching a colleague:

**"Implementing Service Mesh Security with Istio"**

Your explanation should cover:
1. **The Problem**: Why do microservices need a service mesh? (unencrypted traffic, no identity, visibility gaps)
2. **How It Works**: Sidecar pattern, control plane vs data plane, automatic certificate management
3. **Demo**: "Let me show you mTLS in action..." (trace a request, show certificates, demonstrate policy enforcement)
4. **Troubleshooting**: "If you see 'RBAC: access denied', here's what to check..."

**Time yourself**: Can you explain it clearly in under 5 minutes?

---

## **8.3 Integration Questions**

### Question 1: Architecture & Design

**Scenario**: You're architecting a service mesh for a new e-commerce platform with 30 microservices spanning 5 namespaces (frontend, checkout, inventory, payments, analytics).

**Task**: Draw or describe:
1. **Service mesh architecture diagram** showing:
   - Control plane placement
   - Ingress/egress gateways
   - Namespace boundaries
   - mTLS flows between services
   - Observability data collection points

2. **Explain your design choices**:
   - Where would you deploy istiod? (namespace, replicas)
   - Which services need ingress? Which need egress?
   - How would you handle external APIs (payment gateway, shipping provider)?
   - What mTLS mode per namespace? Why?
   - How would you implement network segmentation (frontend shouldn't call payments directly)?

<details>
<summary>🔍 <strong>Sample Solution</strong></summary>

**Architecture Diagram Description**:
```
┌─────────────────────────────────────────────────────────┐
│                    istio-system                         │
│  ┌──────────┐  ┌─────────────┐  ┌─────────────────┐   │
│  │  istiod  │  │  Ingress GW │  │   Egress GW     │   │
│  │(3 replicas)│  │(HA mode)    │  │(external APIs)  │   │
│  └──────────┘  └─────────────┘  └─────────────────┘   │
└─────────────────────────────────────────────────────────┘
         │                │                    │
         │ (config)       │ (traffic)          │ (external traffic)
         ▼                ▼                    ▼
┌────────────────┬────────────────┬────────────────┐
│   frontend     │   checkout     │   payments     │
│   namespace    │   namespace    │   namespace    │
│ ┌───┐ ┌───┐  │ ┌───┐ ┌───┐   │ ┌───┐ ┌───┐   │
│ │ui │ │api│  │ │cart│ │ord│   │ │pay│ │inv│   │
│ └───┘ └───┘  │ └───┘ └───┘   │ └───┘ └───┘   │
│   PERMISSIVE  │   STRICT       │   STRICT       │
└────────────────┴────────────────┴────────────────┘
                         │
                         ▼
                ┌─────────────────┐
                │   analytics     │
                │   namespace     │
                │ ┌───┐ ┌───┐    │
                │ │log│ │met│    │
                │ └───┘ └───┘    │
                │   STRICT        │
                └─────────────────┘
```

**Design Choices**:

**1. Control Plane (istiod)**:
- **Placement**: Dedicated `istio-system` namespace
- **Replicas**: 3 (HA for production, survives node failures)
- **Resources**: 2CPU, 4GB RAM per replica
- **Why**: Centralized control, isolated from application namespaces

**2. Ingress Gateway**:
- **Services needing ingress**: frontend (UI), checkout (API)
- **HA Mode**: 2 replicas across availability zones
- **TLS Termination**: Handle external HTTPS, internal mTLS
- **Why**: Single entry point, centralized rate limiting/WAF

**3. Egress Gateway**:
- **External APIs**: Payment gateway (Stripe), Shipping (FedEx)
- **ServiceEntry**: Define external hosts
- **DestinationRule**: `mode: SIMPLE` (standard TLS, not mTLS)
- **Why**: Audit external traffic, implement retry/timeout policies

**4. mTLS Modes**:
- **frontend namespace**: PERMISSIVE (may have legacy components, gradual migration)
- **checkout, payments, analytics**: STRICT (sensitive data, enforce zero-trust)
- **Why**: Balance security with migration complexity

**5. Network Segmentation**:
```yaml
# Prevent frontend from directly calling payments
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: payments-isolation
  namespace: payments
spec:
  selector:
    matchLabels:
      app: payment-service
  rules:
  - from:
    - source:
        namespaces: ["checkout"]  # Only checkout can call payments
        principals: ["cluster.local/ns/checkout/sa/checkout-service"]
  - from:
    - source:
        namespaces: ["istio-system"]  # Allow health checks
```

**6. Observability**:
- Prometheus: Scrape all sidecars (metrics)
- Jaeger: 5% sampling (balance cost/visibility)
- Grafana: Dashboards per namespace
- Kiali: Real-time topology view
- **Data flow**: Sidecar → Prometheus → Grafana

</details>

---

### Question 2: Troubleshooting Scenario

**Scenario**: You receive an alert: "Payment service error rate spiked to 15% in the last 10 minutes. Users report checkout failures."

**Your Response**:
1. **Immediate triage** (first 2 minutes):
   - What dashboards do you check first?
   - What initial hypotheses do you form?

2. **Systematic investigation** (next 10 minutes):
   - What commands do you run?
   - What logs do you examine?
   - How do you isolate the issue?

3. **Root cause analysis**:
   - Trace shows 5-second timeout from payment → external gateway
   - mTLS handshake succeeding
   - No authorization denials
   - External payment gateway returning 503 intermittently

4. **Resolution**:
   - Immediate mitigation?
   - Long-term fix?
   - How do you prevent recurrence?

<details>
<summary>🔍 <strong>Sample Solution</strong></summary>

**1. Immediate Triage (2 minutes)**:

**Dashboards to check**:
```
1. Grafana: Istio Service Dashboard → payment-service
   - Check error rate spike (15% confirmed)
   - P99 latency (likely elevated)
   - Success rate by response code (5xx? 4xx?)

2. Kiali: Service Graph
   - payment-service connections
   - Look for red indicators (errors)
   - Check if upstream (checkout) or downstream (external gateway) issue
```

**Initial hypotheses**:
- External payment gateway degraded (most likely)
- New deployment rolled out (check rollout history)
- Certificate expiration (check cert validity)
- Authorization policy change (check recent config changes)
- Network partition (check node health)

**2. Systematic Investigation (10 minutes)**:

**Step 1: Confirm error pattern** (1 min):
```bash
# Check recent errors in payment service
kubectl logs -l app=payment-service --tail=100 | grep -i error

# Check sidecar logs for connection issues
kubectl logs -l app=payment-service -c istio-proxy --tail=50 | grep -E "(503|timeout|refused)"
```

**Finding**: Logs show "upstream connect timeout" to external gateway

**Step 2: Check external connectivity** (2 min):
```bash
# Test external gateway directly from payment pod
kubectl exec -it $(kubectl get pod -l app=payment-service -o jsonpath='{.items[0].metadata.name}') \
  -c payment-service -- curl -v --max-time 10 https://api.stripe.com/v1/health

# Check ServiceEntry for external gateway
kubectl get serviceentry external-payment-gateway -o yaml
```

**Finding**: Curl times out after 5 seconds, ServiceEntry looks correct

**Step 3: Check distributed traces** (3 min):
```
Jaeger → Service: payment-service
Filter: minDuration > 4s
Select recent trace

Trace shows:
  payment-service → egress-gateway → external-payment-api
  └─ Span "external-payment-api call" = 5.2s (timeout)
```

**Finding**: Timeout is from external API, not our infrastructure

**Step 4: Check if issue is affecting all requests** (2 min):
```bash
# Query success rate for payment service
kubectl port-forward -n istio-system svc/prometheus 9090:9090 &
# In Prometheus:
sum(rate(istio_requests_total{destination_service="payment-service",response_code="200"}[5m]))
/
sum(rate(istio_requests_total{destination_service="payment-service"}[5m]))
```

**Finding**: Success rate is 85% (15% failure matches alert)

**Step 5: Check external API status page** (2 min):
```
Visit: https://status.stripe.com
```

**Finding**: Stripe reports "Partial Outage - API Requests Degraded"

**3. Root Cause**:
- **External payment gateway (Stripe)** experiencing partial outage
- Their API responding with 503 Service Unavailable intermittently
- Our 5-second timeout causing failures
- Issue is external, NOT in our service mesh

**4. Resolution**:

**Immediate Mitigation**:
```yaml
# Increase timeout and add retry policy
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: external-payment-gateway
spec:
  host: api.stripe.com
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
    outlierDetection:
      consecutive5xxErrors: 3
      interval: 30s
      baseEjectionTime: 30s
    retries:
      attempts: 3
      perTryTimeout: 10s  # Increased from 5s
      retryOn: "5xx,connect-failure,refused-stream"
    timeout: 30s  # Increased overall timeout
```

```bash
kubectl apply -f destination-rule-payment-gateway.yaml
```

**Why this helps**:
- Longer timeout gives Stripe more time to respond
- Retry logic handles intermittent failures
- Circuit breaker (outlierDetection) prevents cascading failures

**Long-term Fix**:
1. **Implement fallback payment processor**:
   - Add secondary payment gateway (e.g., Braintree)
   - Route traffic if primary fails
   
2. **Better error handling in application**:
   ```python
   try:
       payment_response = stripe.charge()
   except StripeTimeout:
       return {"status": "pending", "message": "Payment processing, check back shortly"}
   ```

3. **User communication**:
   - Show friendly error message
   - Allow retry without re-entering card details

**Prevention**:
- **Monitoring**: Alert on external API error rate (not just our services)
- **Status page integration**: Auto-detect provider outages
- **Load shedding**: Gracefully degrade (skip optional features during outage)
- **Rate limiting**: Protect against retry storms
- **Regular drills**: Test external dependency failure scenarios

**Post-Incident**:
- Document in runbook
- Share findings with team
- Update on-call playbook
- Schedule chaos engineering exercise to simulate external API failures

</details>

---

### Question 3: Production Considerations

**Scenario**: Your team is ready to deploy this service mesh to production. What needs to be addressed?

**Task**: Create a production readiness checklist covering:
1. **Security**: Beyond mTLS, what additional security measures?
2. **Monitoring**: What alerts would you configure? (thresholds, SLOs)
3. **Reliability**: How do you ensure high availability? (HA control plane, failure modes)
4. **Operations**: Day-2 operations (certificate rotation, upgrades, scaling)
5. **Performance**: How do you optimize mesh overhead? (resource limits, caching)
6. **Documentation**: What docs do you create for your team?

<details>
<summary>🔍 <strong>Sample Solution - Production Readiness Checklist</strong></summary>

### **1. Security Hardening**

**Beyond mTLS**:
- [ ] **Egress traffic control**: Block all egress by default, whitelist external services
  ```yaml
  apiVersion: networking.istio.io/v1beta1
  kind: Sidecar
  metadata:
    name: default
    namespace: istio-system
  spec:
    outboundTrafficPolicy:
      mode: REGISTRY_ONLY  # Block unknown destinations
  ```

- [ ] **Network policies**: Kubernetes NetworkPolicy as defense-in-depth
- [ ] **Pod Security Standards**: Enforce restricted pod security
- [ ] **Image scanning**: Scan all container images (Envoy, app containers)
- [ ] **Secrets management**: Use external secrets manager (Vault, AWS Secrets Manager)
- [ ] **RBAC**: Restrict who can modify Istio configurations
- [ ] **Audit logging**: Enable Kubernetes audit logs for config changes
- [ ] **Workload identity**: Use cloud provider workload identity (AWS IRSA, GCP Workload Identity)

### **2. Monitoring & Alerting**

**Critical Alerts**:

| Alert | Threshold | Severity | Action |
|-------|-----------|----------|--------|
| Control plane down | istiod unavailable > 1min | P1 | Page on-call |
| Certificate rotation failure | Cert expires < 1h | P1 | Page on-call |
| mTLS connection failures | > 1% requests failing mTLS | P2 | Investigate ASAP |
| Authorization denials spike | > 100 denials/5min | P3 | Review policy changes |
| Mesh configuration invalid | `istioctl analyze` errors | P2 | Block deployment |
| Sidecar injection failure | Pod without sidecar in 5min | P2 | Auto-remediate or alert |
| Proxy sync failure | CDS/LDS not SYNCED > 2min | P2 | Investigate connectivity |

**SLOs (Service Level Objectives)**:
```yaml
# Example: Payment service SLO
Availability: 99.9% (43min downtime/month)
Latency: p95 < 500ms, p99 < 1s
Error Rate: < 0.1%

# Prometheus alert rules
groups:
- name: payment-slo
  rules:
  - alert: PaymentSLOLatency
    expr: histogram_quantile(0.99, rate(istio_request_duration_milliseconds_bucket{destination_service="payment"}[5m])) > 1000
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Payment service p99 latency exceeds SLO"
      
  - alert: PaymentSLOAvailability
    expr: (sum(rate(istio_requests_total{destination_service="payment",response_code=~"2.."}[5m])) / sum(rate(istio_requests_total{destination_service="payment"}[5m]))) < 0.999
    for: 5m
    labels:
      severity: critical
```

**Dashboards**:
- [ ] Golden signals dashboard (latency, traffic, errors, saturation)
- [ ] Service-level SLO tracking
- [ ] Control plane health
- [ ] Certificate expiration timeline
- [ ] mTLS coverage percentage
- [ ] Resource usage trends

### **3. Reliability & High Availability**

**Control Plane HA**:
```yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
spec:
  components:
    pilot:  # istiod
      k8s:
        replicaCount: 3  # Minimum 3 for HA
        resources:
          requests:
            cpu: 500m
            memory: 2Gi
          limits:
            cpu: 2
            memory: 4Gi
        hpaSpec:
          minReplicas: 3
          maxReplicas: 5
        affinity:
          podAntiAffinity:  # Spread across nodes/zones
            requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchLabels:
                  app: istiod
              topologyKey: kubernetes.io/hostname
```

**Ingress Gateway HA**:
- [ ] Min 2 replicas across availability zones
- [ ] HPA configured (scale 2-10 based on CPU/RPS)
- [ ] PodDisruptionBudget: minAvailable = 1
- [ ] LoadBalancer with health checks

**Failure Modes Testing**:
- [ ] Chaos engineering: Kill istiod pod (data plane continues)
- [ ] Test certificate expiration (rotate before expiry)
- [ ] Simulate zone failure (pods reschedule)
- [ ] Test control plane upgrade (zero-downtime)
- [ ] Network partition testing

**Backup & Disaster Recovery**:
```bash
# Regular backups of Istio configuration
kubectl get gateway,virtualservice,destinationrule,peerauthentication,authorizationpolicy \
  --all-namespaces -o yaml > istio-config-backup-$(date +%Y%m%d).yaml

# Store in version control (GitOps)
```

### **4. Operations (Day-2)**

**Certificate Management**:
- [ ] Monitor expiration: Alert if cert expires < 24h
- [ ] Test rotation: Force rotation in staging monthly
- [ ] Audit logs: Track certificate issuance/revocation
- [ ] Backup CA private key (stored securely)

**Upgrades**:
```bash
# Canary upgrade process
1. Deploy new istiod version alongside old (canary)
2. Label few workloads to use new version
3. Monitor metrics, rollback if issues
4. Gradually shift traffic to new version
5. Remove old version after 100% migration

# Example: Canary control plane
istioctl install --set values.revision=1-28-3 --set values.defaultRevision=1-28-2
kubectl label namespace staging istio.io/rev=1-28-3 --overwrite
```

**Scaling**:
- [ ] HPA for control plane based on proxy connections
- [ ] HPA for gateways based on request rate
- [ ] Node autoscaling for increased workload
- [ ] Test at 2x normal load monthly

**Configuration Management**:
- [ ] GitOps (ArgoCD/Flux): All configs in Git
- [ ] Policy-as-code (OPA/Kyverno): Validate configs before apply
- [ ] Change management: Peer review for policy changes
- [ ] Rollback plan: Keep last 3 config versions

### **5. Performance Optimization**

**Sidecar Resource Tuning**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: istio-sidecar-injector
  namespace: istio-system
data:
  values: |
    global:
      proxy:
        resources:
          requests:
            cpu: 50m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
```

**Reduce Mesh Overhead**:
- [ ] Limit sidecar scope (Sidecar resource with egress hosts)
- [ ] Disable unused features (telemetry you don't need)
- [ ] Tune connection pool sizes
- [ ] Enable caching where possible
- [ ] Use protocol sniffing or explicit protocol

**Performance Testing**:
- [ ] Baseline: Measure latency without mesh
- [ ] With mesh: Measure added latency (target: <20%)
- [ ] Load testing: wrk/k6 at 2x-5x peak load
- [ ] Profile: CPU/memory usage under load

### **6. Documentation**

**Runbooks**:
- [ ] "Service Mesh Incident Response Playbook"
  - Common errors and resolutions
  - Escalation paths
  - Emergency rollback procedures
  
- [ ] "Certificate Expiration Emergency Procedure"
- [ ] "mTLS Troubleshooting Guide"
- [ ] "Adding New Service to Mesh"
- [ ] "Mesh Upgrade Procedure"

**Architecture Docs**:
- [ ] Service mesh architecture diagram
- [ ] Traffic flow diagrams
- [ ] mTLS enforcement policy
- [ ] Network segmentation rules
- [ ] External integrations (payment, shipping APIs)

**Developer Guide**:
- [ ] "Developing Services for Istio"
  - How to test locally with mesh
  - Debugging sidecar issues
  - Understanding observability data
  
- [ ] "Adding Authorization Policies"
- [ ] "Traffic Management Patterns" (canary, blue/green)

**On-Call Guide**:
- [ ] Dashboard links (Grafana, Kiali, Jaeger)
- [ ] Alert runbook (what each alert means, how to respond)
- [ ] Common issues and quick fixes
- [ ] Escalation contacts

</details>

---

### Question 4: Technology Connections

**Part A**: Compare Istio with alternatives:

| Feature | Istio | Linkerd | Consul Connect | AWS App Mesh |
|---------|-------|---------|----------------|--------------|
| Learning curve | ? | ? | ? | ? |
| Resource overhead | ? | ? | ? | ? |
| mTLS implementation | ? | ? | ? | ? |
| Observability | ? | ? | ? | ? |
| Traffic management | ? | ? | ? | ? |
| Best for | ? | ? | ? | ? |

**Part B**: How would you integrate Istio with:
1. **CI/CD pipeline** (Jenkins, GitLab CI, ArgoCD)?
2. **External auth** (OAuth2, JWT validation at gateway)?
3. **WAF** (Web Application Firewall - ModSecurity, AWS WAF)?
4. **Service discovery** (Consul, etcd)?

<details>
<summary>🔍 <strong>Sample Solution</strong></summary>

**Part A: Service Mesh Comparison**:

| Feature | Istio | Linkerd | Consul Connect | AWS App Mesh |
|---------|-------|---------|----------------|--------------|
| **Learning curve** | Steep (complex, many CRDs) | Gentle (simple, opinionated) | Moderate (Consul knowledge needed) | Moderate (AWS-specific) |
| **Resource overhead** | High (Envoy 50-100Mi RAM/pod) | Low (Linkerd-proxy 20-40Mi) | Moderate | Moderate |
| **mTLS implementation** | Automatic, flexible modes | Automatic, always-on | Manual + automatic | Automatic |
| **Observability** | Excellent (Prometheus, Jaeger, Kiali) | Good (built-in, lightweight) | Good (Consul UI) | Good (CloudWatch, X-Ray) |
| **Traffic management** | Excellent (weights, headers, mirrors) | Good (basic routing) | Moderate | Good (route53 integration) |
| **Community** | Large, CNCF graduated | Growing, CNCF graduated | Large (HashiCorp) | AWS ecosystem |
| **Best for** | Complex requirements, features | Simplicity, performance | Multi-platform, VM + K8s | AWS-native workloads |

**When to choose**:
- **Istio**: Need advanced traffic management, multi-cluster, maximum flexibility
- **Linkerd**: Want simplicity, lowest overhead, "it just works"
- **Consul Connect**: Hybrid cloud, VMs + containers, existing Consul use
- **AWS App Mesh**: All-in on AWS, tight integration with AWS services

**Part B: Integration Patterns**

**1. CI/CD Pipeline Integration**:

```yaml
# GitLab CI example - Automated deployment with Istio validation
stages:
  - build
  - validate
  - deploy

validate-istio-config:
  stage: validate
  script:
    - istioctl analyze manifests/*.yaml
    - kubectl apply --dry-run=client -f manifests/
  only:
    - merge_requests

deploy-canary:
  stage: deploy
  script:
    # Deploy new version with canary traffic split
    - kubectl apply -f app-v2-deployment.yaml
    - kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: myapp
        subset: v2
  - route:
    - destination:
        host: myapp
        subset: v1
      weight: 90
    - destination:
        host: myapp
        subset: v2
      weight: 10
EOF
    # Monitor metrics for 10 minutes
    - ./scripts/monitor-canary.sh
    # If successful, shift to 100% v2
  environment:
    name: production
```

**ArgoCD Integration**:
- Use ApplicationSet for multi-cluster deployments
- Health checks for Istio resources
- Automated rollback on failed validation

**2. External Authentication (OAuth2/JWT)**:

```yaml
# JWT validation at ingress gateway
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: jwt-auth
  namespace: istio-system
spec:
  selector:
    matchLabels:
      app: istio-ingressgateway
  jwtRules:
  - issuer: "https://auth.example.com"
    jwksUri: "https://auth.example.com/.well-known/jwks.json"
    audiences:
    - "my-api"
---
# Require valid JWT for all requests
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: require-jwt
  namespace: default
spec:
  action: ALLOW
  rules:
  - from:
    - source:
        requestPrincipals: ["*"]  # Must have valid JWT
```

**OAuth2 Proxy Integration**:
- Deploy oauth2-proxy as sidecar or separate service
- Istio routes unauthenticated requests to proxy
- Proxy handles OAuth flow, sets headers

**3. WAF Integration (ModSecurity)**:

```yaml
# Deploy Envoy with ModSecurity filter
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
spec:
  meshConfig:
    defaultConfig:
      proxyMetadata:
        # Enable ModSecurity wasm filter
        WASM_INSECURE_REGISTRIES: "*"
  components:
    ingressGateways:
    - name: istio-ingressgateway
      k8s:
        podAnnotations:
          sidecar.istio.io/userVolumeMount: '[{"name":"modsec-rules","mountPath":"/etc/modsecurity"}]'
```

**AWS WAF Integration**:
- Place AWS ALB in front of Istio ingress
- ALB → WAF rules → Istio ingress gateway
- Use Istio for internal service-to-service security

**4. Service Discovery Integration (Consul)**:

```yaml
# ServiceEntry to register Consul services in Istio
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: consul-service
spec:
  hosts:
  - database.service.consul
  ports:
  - number: 5432
    name: tcp
    protocol: TCP
  resolution: DNS
  location: MESH_EXTERNAL
---
# Use Consul Connect alongside Istio for hybrid mesh
# Consul for VM workloads, Istio for K8s
```

**Integration Pattern**:
- Use MeshGateway to bridge Istio and Consul meshes
- Federate service discovery across both systems
- Maintain consistent mTLS across both platforms

</details>

---

## **8.4 Hands-On Challenge: Implement Multi-Version Canary Deployment**

**Duration**: 20-30 minutes

**Scenario**: Deploy a new version of the reviews service and gradually shift traffic from v1 to v2 using Istio traffic management.

**Requirements**:
1. Deploy reviews-v2 (already exists in Bookinfo)
2. Create DestinationRule with subsets for v1 and v2
3. Create VirtualService with weighted routing: 90% v1, 10% v2
4. Monitor traffic distribution in Kiali
5. Gradually increase v2 traffic: 50%, then 100%
6. Implement header-based routing: users with header `version: v2` always get v2

**No instructions provided - implement based on lab knowledge!**

<details>
<summary>🔍 <strong>Solution - Only check after attempting!</strong></summary>

```bash
# Step 1: Ensure all reviews versions are deployed
kubectl get pods -l app=reviews

# Step 2: Create DestinationRule with subsets
cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v3
    labels:
      version: v3
EOF

# Step 3: Create VirtualService with 90/10 split
cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 90
    - destination:
        host: reviews
        subset: v2
      weight: 10
EOF

# Step 4: Generate traffic and observe in Kiali
for i in {1..100}; do
  curl -s "http://${GATEWAY_URL}/productpage" > /dev/null
  echo "Request $i sent"
  sleep 1
done

# Open Kiali and view traffic distribution
kubectl port-forward -n istio-system svc/kiali 20001:20001

# Step 5: Increase to 50/50
kubectl patch virtualservice reviews --type merge -p '
{
  "spec": {
    "http": [{
      "route": [
        {"destination": {"host": "reviews", "subset": "v1"}, "weight": 50},
        {"destination": {"host": "reviews", "subset": "v2"}, "weight": 50}
      ]
    }]
  }
}'

# Generate more traffic and verify distribution
for i in {1..50}; do curl -s "http://${GATEWAY_URL}/productpage" > /dev/null; done

# Step 6: Move to 100% v2
kubectl patch virtualservice reviews --type merge -p '
{
  "spec": {
    "http": [{
      "route": [
        {"destination": {"host": "reviews", "subset": "v2"}, "weight": 100}
      ]
    }]
  }
}'

# Step 7: Add header-based routing (bonus)
cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        version:
          exact: "v2"
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
EOF

# Test header-based routing
curl -H "version: v2" "http://${GATEWAY_URL}/productpage"  # Gets v2
curl "http://${GATEWAY_URL}/productpage"                   # Gets v1
```

**Validation**:
- Kiali shows traffic distribution matches configuration
- Prometheus metrics confirm percentages
- Header-based routing works as expected

</details>

---

## **8.5 Teaching Scenario: 5-Minute Mini-Lesson**

**Challenge**: Teach "Mutual TLS in Istio" to a junior engineer in 5 minutes.

Your lesson should include:
1. **Hook** (30s): Grab attention with a compelling problem or analogy
2. **Explain** (2min): Core concept, how it works, key components
3. **Demonstrate** (1.5min): Show a command or diagram
4. **Practice** (1min): Give them something to try
5. **Recap** (30s): Summarize key takeaways

**Record or write out your lesson!**

<details>
<summary>🔍 <strong>Sample Mini-Lesson Script</strong></summary>

**[HOOK - 30s]**
"Imagine you're at a bank. The teller asks for your ID before processing your transaction, but you also want to verify that *they* are a real bank employee, not an imposter. That's mutual authentication - both parties prove their identity. In microservices, without this, any malicious pod could impersonate your payment service. Istio's mTLS solves this."

**[EXPLAIN - 2min]**
"Istio implements mutual TLS automatically using the sidecar pattern. Here's how:

1. **Every pod gets a certificate** - When Kubernetes creates your pod, Istio's mutating webhook injects a sidecar. This sidecar immediately requests a certificate from istiod, which acts as a Certificate Authority.

2. **The certificate contains identity** - It's not just encryption keys; it has a SPIFFE ID like `cluster.local/ns/payments/sa/payment-service`. This cryptographically proves which service is making the request.

3. **Automatic mTLS handshake** - When productpage calls payments, the sidecars perform a TLS handshake:
   - Productpage sidecar: 'Here's my cert, I'm productpage'
   - Payments sidecar: 'Here's my cert, I'm payments'
   - Both verify the other's certificate against the root CA
   - Encrypted channel established

4. **Your application code doesn't change** - Your app still makes plain HTTP calls to `http://payments:9080`. The sidecar intercepts via iptables, encrypts, sends to peer sidecar, which decrypts and forwards to the app.

5. **Certificates auto-rotate** - Every 24 hours, Envoy requests fresh certificates. Zero downtime, no manual management."

**[DEMONSTRATE - 1.5min]**
"Let me show you. First, verify mTLS is working:"

```bash
# Check current mTLS status
istioctl authn tls-check productpage-pod.default payments.default.svc.cluster.local:9080
# You'll see: STATUS: OK, SERVER: STRICT, CLIENT: mTLS
# This means both sides are using mTLS successfully

# Now look at the actual certificate
istioctl proxy-config secret productpage-pod
# See 'default' certificate - that's the workload identity
# Valid for ~24 hours

# View the SPIFFE identity
istioctl proxy-config secret productpage-pod -o json | \
  jq -r '.dynamicActiveSecrets[0].secret.tlsCertificate.certificateChain.inlineBytes' | \
  base64 -d | openssl x509 -text -noout | grep URI
# Shows: URI:spiffe://cluster.local/ns/default/sa/bookinfo-productpage
```

**[PRACTICE - 1min]**
"Your turn! Try this:
1. Check the mTLS status for *your* favorite pod
2. Find its SPIFFE identity
3. Try to call a service from a pod *without* a sidecar - you'll see it gets blocked

Bonus: Look at the PeerAuthentication resource that enforces STRICT mode."

**[RECAP - 30s]**
"Three key takeaways:
1. **mTLS = mutual authentication + encryption** - Both client and server prove identity
2. **Istio automates everything** - Certificates issued, rotated, and verified automatically via sidecars
3. **SPIFFE identity = cryptographic proof** - No more trusting IP addresses

mTLS is your foundation for zero-trust networking. Every service call is authenticated and encrypted, with zero code changes."

</details>

---

## **8.6 Mastery Rubric**

Rate yourself (1-5) on each dimension:

### Conceptual Understanding (1-5)

**Score: _____ / 5**

- [ ] 1 - Can define basic terms (sidecar, mTLS, control plane)
- [ ] 2 - Can explain how Istio works at a high level
- [ ] 3 - Can explain interactions between components (sidecar ↔ istiod ↔ Envoy)
- [ ] 4 - Can design service mesh architectures for realistic scenarios
- [ ] 5 - Can compare tradeoffs, teach others, make production decisions confidently

### Practical Skills (1-5)

**Score: _____ / 5**

- [ ] 1 - Can install Istio following instructions
- [ ] 2 - Can deploy applications and configure basic routing
- [ ] 3 - Can implement mTLS and authorization policies independently
- [ ] 4 - Can implement advanced patterns (canary, circuit breaking, rate limiting)
- [ ] 5 - Can optimize performance, design for HA, implement GitOps workflows

### Troubleshooting Ability (1-5)

**Score: _____ / 5**

- [ ] 1 - Can use provided diagnostic commands
- [ ] 2 - Can interpret common errors and apply known fixes
- [ ] 3 - Can systematically diagnose novel issues using logs, metrics, traces
- [ ] 4 - Can identify root causes in complex scenarios (multi-service failures)
- [ ] 5 - Can mentor others in troubleshooting, create runbooks, predict issues

### Integration & Systems Thinking (1-5)

**Score: _____ / 5**

- [ ] 1 - Understands Istio in isolation
- [ ] 2 - Can connect Istio to observability tools
- [ ] 3 - Can integrate with CI/CD, external auth, other platforms
- [ ] 4 - Can design end-to-end solutions (mesh + monitoring + security + DR)
- [ ] 5 - Can make architecture decisions balancing mesh vs alternatives

**Total Score: _____ / 20**

**Interpretation**:
- **16-20**: Mastery - Ready for production implementations and teaching others
- **12-15**: Proficient - Can work independently, some advanced topics need practice
- **8-11**: Competent - Core skills solid, need more real-world experience
- **4-7**: Developing - Foundational knowledge present, practice needed
- **1-3**: Novice - Review lab and practice fundamentals

---

## **8.7 Mastery Checklist**

### Foundation (Must-Know)

- [ ] Explain sidecar pattern and how traffic is intercepted
- [ ] Describe Istio architecture (control plane vs data plane)
- [ ] Configure automatic sidecar injection
- [ ] Create Gateway and VirtualService for ingress
- [ ] Understand Envoy's role in the mesh

### Intermediate (Should-Know)

- [ ] Implement STRICT mTLS with PeerAuthentication
- [ ] Create DestinationRules for traffic policies
- [ ] Write AuthorizationPolicy for least-privilege access
- [ ] Deploy observability stack (Prometheus, Grafana, Jaeger, Kiali)
- [ ] Configure telemetry collection
- [ ] Troubleshoot common mTLS and authorization errors
- [ ] Interpret distributed traces and metrics
- [ ] Validate configurations with istioctl analyze

### Advanced (Nice-to-Have)

- [ ] Implement canary deployments with traffic shifting
- [ ] Configure circuit breakers and outlier detection
- [ ] Design multi-cluster service mesh architectures
- [ ] Integrate with external auth providers (OAuth2, JWT)
- [ ] Implement rate limiting and quota management
- [ ] Optimize mesh performance (resource limits, caching)
- [ ] Design HA control plane deployments
- [ ] Create runbooks and operational procedures
- [ ] Implement GitOps for mesh configuration
- [ ] Perform chaos engineering on the mesh

---

## **8.8 Spaced Repetition Schedule**

To solidify learning, review and practice on this schedule:

| Timeframe | Activity | Focus Areas |
|-----------|----------|-------------|
| **Day 1** (today) | Complete full lab | All phases, hands-on practice |
| **Day 3** | Review notes, answer recall questions | Concepts, architecture, workflows |
| **Week 2** | Rebuild mesh from scratch (no instructions) | Installation, mTLS, observability |
| **Month 2** | Implement advanced scenario (multi-namespace) | Authorization policies, traffic management |
| **Month 6** | Teach the lab to a colleague | Solidify mastery through teaching |

**Active Recall Exercises** (do without looking at notes):
- Draw Istio architecture from memory
- List all steps to enable STRICT mTLS
- Explain a request flow through the mesh
- Troubleshoot a scenario without diagnostic commands first

---

## **8.9 Learning Journal Template**

Document your learning journey:

```markdown
# Istio Service Mesh Lab - Learning Journal

## Pre-Lab Expectations
- What I expected to learn:
- What I thought would be challenging:
- Prior knowledge:

## Key Discoveries
1. 
2. 
3. 

## Challenges Encountered
| Challenge | How I Solved It | Lesson Learned |
|-----------|----------------|----------------|
|           |                |                |

## Connections to Other Knowledge
- This relates to [X] because...
- This is different from [Y] because...
- I could apply this to...

## Real-World Applications
- Use case 1:
- Use case 2:
- Industry example:

## Questions for Further Learning
1. 
2. 
3. 

## Next Steps
- Practice:
- Deep dive:
- Build project:

## Self-Assessment
- Conceptual Understanding: ___/5
- Practical Skills: ___/5
- Troubleshooting: ___/5
- Integration: ___/5
```

---

## **8.10 Answer Keys for Active Recall Questions**

<details>
<summary>📖 <strong>Active Recall #1 Answers (from Phase 2)</strong></summary>

**See Phase 2, Active Recall Check #1 for full answers**

Quick summary:
- Each pod has 2 containers due to sidecar injection (app + istio-proxy)
- VirtualService host `*` would need host header if changed to specific host
- Sidecar injection controlled by namespace label; pods need recreation to get sidecars
- Multi-environment routing requires separate VirtualServices per host/environment

</details>

<details>
<summary>📖 <strong>Active Recall #2 Answers (from Phase 3)</strong></summary>

**See Phase 3, Active Recall Check #2 for full answers**

Quick summary:
- Metrics = aggregated numbers, Logs = event records, Traces = request flows
- 100% trace sampling has high overhead; use 1-10% in production
- Envoy sidecar collects observability data by intercepting all traffic
- Troubleshoot latency: Start with metrics, use traces to identify bottleneck, check logs for errors

</details>

<details>
<summary>📖 <strong>Active Recall #3 Answers (from Phase 4)</strong></summary>

**See Phase 4, Active Recall Check #3 for full answers**

Quick summary:
- PeerAuthentication = server-side mTLS, DestinationRule = client-side mTLS, AuthorizationPolicy = access control
- PERMISSIVE mode accepts both mTLS and plaintext; use during migration
- Envoy + istiod work together: istiod issues certs via SDS, Envoy performs TLS handshakes, certs rotate every 24h
- Migration strategy: Start PERMISSIVE, monitor adoption, move to STRICT namespace-by-namespace

</details>

---

**🎉 Congratulations!** You've completed the comprehensive Istio Service Mesh Security Lab. You now have hands-on experience with zero-trust networking, observability, and production-grade security patterns used by enterprises worldwide.

**Progress Update**: Foundation ⭐⭐⭐⭐⭐ | Implementation ⭐⭐⭐⭐⭐ | Troubleshooting ⭐⭐⭐⭐⭐ | Production ⭐⭐⭐⭐⭐

---

# 📚 Additional Resources

## Official Documentation
- [Istio Documentation](https://istio.io/latest/docs/)
- [Istio Security Best Practices](https://istio.io/latest/docs/ops/best-practices/security/)
- [Envoy Proxy Documentation](https://www.envoyproxy.io/docs/envoy/latest/)

## Community & Support
- [Istio Slack](https://istio.slack.com) - Active community support
- [Istio GitHub](https://github.com/istio/istio) - Issues, discussions, source code
- [CNCF Istio Project Page](https://www.cncf.io/projects/istio/)

## Advanced Learning
- [Istio Workshop by Solo.io](https://github.com/solo-io/workshops)
- [Service Mesh Patterns](https://www.manning.com/books/istio-in-action) - Book by Solo.io team
- [Tetrate Istio Academy](https://academy.tetrate.io) - Free advanced courses

## Certification Preparation
- **KCSA**: Kubernetes and Cloud Native Security Associate
- **CKA**: Certified Kubernetes Administrator (includes networking)
- **Istio Certification** (when available from CNCF)

## Hands-On Practice
- [Katacoda Istio Scenarios](https://www.katacoda.com/courses/istio)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)
- [killercoda Istio Labs](https://killercoda.com/istio)
