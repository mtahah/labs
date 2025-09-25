# Lab: Production-Ready Multi-Tier Application on Kubernetes (Frontend + Backend + MySQL)

This lab guides you through deploying a secure, production-ready three-tier application on Kubernetes with the following improvements:

- Security hardening: secrets, least-privilege, network policies, non-root, resource limits
- Production readiness: StatefulSet with persistent storage, health checks, graceful shutdown, PDBs, HPA, affinity
- Modern practices: Ingress, Helm-based add-ons, clean YAML structure, images built from Dockerfile (no inline code)
- Observability: metrics (Prometheus/Grafana), centralized logging (Loki/Promtail), optional tracing (Jaeger)
- Enhanced documentation: clear prerequisites, verification, troubleshooting, cleanup

Time to complete: 60–120 minutes (depending on add-ons installed)

Important
- Run commands in a Linux shell with kubectl and (optionally) helm installed. If you’re using Al Nafi’s cloud lab, open the provided terminal. Do not run these commands on Windows PowerShell.
- All code blocks below are copy/paste-friendly bash snippets.


## 0) Prerequisites and Variables

```bash
# Safety options for shell
set -euo pipefail

# Namespace for this lab
NS=multi-tier-app

# Choose image registry for backend image
# Option A: Docker Hub (recommended) – set your username
: "${DOCKERHUB_USERNAME:=your-dockerhub-username}"
BACKEND_IMAGE="docker.io/${DOCKERHUB_USERNAME}/mtk-backend:1.0.0"

# Option B: If using minikube or kind (local dev), you can load a local image instead.
# Adjust as needed; the manifests below reference $BACKEND_IMAGE.

# Storage size for MySQL PVC
MYSQL_STORAGE_SIZE=5Gi

# App config
APP_DB_NAME=webapp_db
APP_DB_USER=webapp_user
APP_PORT=5000
FRONTEND_REPLICAS=3
BACKEND_REPLICAS=2

# Ingress host (uses nip.io to avoid DNS). Resolves to your node IP automatically later.
: "${FQDN:=}"
```


## 1) Create Namespace

```bash
kubectl create namespace "$NS" --dry-run=client -o yaml | kubectl apply -f -
```


## 2) Security: Secrets and ConfigMaps (No hardcoded creds)

Generate strong random passwords and create a Secret without echoing secrets.

```bash
# Generate strong passwords (not printed)
MYSQL_ROOT_PASSWORD=$(openssl rand -hex 24)
MYSQL_APP_PASSWORD=$(openssl rand -hex 24)

# Create database Secret
kubectl -n "$NS" create secret generic database-secret \
  --from-literal=MYSQL_ROOT_PASSWORD="$MYSQL_ROOT_PASSWORD" \
  --from-literal=MYSQL_PASSWORD="$MYSQL_APP_PASSWORD" \
  --dry-run=client -o yaml | kubectl apply -f -

# Immediately unset the secret variables from your shell
unset MYSQL_ROOT_PASSWORD MYSQL_APP_PASSWORD

# Database ConfigMap (non-sensitive)
cat <<EOF | kubectl apply -n "$NS" -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: database-config
  namespace: ${NS}
data:
  MYSQL_DATABASE: "${APP_DB_NAME}"
  MYSQL_USER: "${APP_DB_USER}"
  DB_HOST: "database"
  DB_PORT: "3306"
EOF
```

Optional: Initialize database schema via ConfigMap mounted into MySQL init directory.

```bash
cat <<'EOF' > mysql-init.sql
CREATE TABLE IF NOT EXISTS messages (
  id INT AUTO_INCREMENT PRIMARY KEY,
  content VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
INSERT INTO messages (content) VALUES ('Hello from MySQL!');
EOF

kubectl create configmap mysql-init \
  -n "$NS" \
  --from-file=init.sql=./mysql-init.sql \
  --dry-run=client -o yaml | kubectl apply -f -
```


## 3) Database Tier: MySQL StatefulSet with Persistent Storage

- Uses PersistentVolumeClaim via volumeClaimTemplates
- Mounts init SQL
- Resource limits, probes, non-root filesystem policies

```bash
cat <<EOF | kubectl apply -n "$NS" -f -
apiVersion: v1
kind: Service
metadata:
  name: database
  labels:
    app: database
spec:
  selector:
    app: database
  ports:
  - name: mysql
    port: 3306
    targetPort: 3306
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
  labels:
    app: database
spec:
  serviceName: database
  replicas: 1
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
        tier: database
    spec:
      terminationGracePeriodSeconds: 60
      containers:
      - name: mysql
        image: mysql:8.0.34
        ports:
        - containerPort: 3306
          name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: database-secret
              key: MYSQL_ROOT_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            configMapKeyRef:
              name: database-config
              key: MYSQL_DATABASE
        - name: MYSQL_USER
          valueFrom:
            configMapKeyRef:
              name: database-config
              key: MYSQL_USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: database-secret
              key: MYSQL_PASSWORD
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
        - name: mysql-init
          mountPath: /docker-entrypoint-initdb.d
        resources:
          requests:
            cpu: 250m
            memory: 512Mi
          limits:
            cpu: 1
            memory: 1Gi
        livenessProbe:
          exec:
            command: ["bash", "-c", "mysqladmin ping -h 127.0.0.1 -uroot -p$MYSQL_ROOT_PASSWORD"]
          initialDelaySeconds: 60
          periodSeconds: 15
        readinessProbe:
          exec:
            command: ["bash", "-c", "mysqladmin ping -h 127.0.0.1 -uroot -p$MYSQL_ROOT_PASSWORD"]
          initialDelaySeconds: 30
          periodSeconds: 10
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: false
      # Use only fsGroup to avoid UID mismatch issues in official MySQL image
      securityContext:
        fsGroup: 999
        fsGroupChangePolicy: OnRootMismatch
      volumes:
      - name: mysql-init
        configMap:
          name: mysql-init
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: ${MYSQL_STORAGE_SIZE}
EOF
```


## 4) Backend Tier: Build Image and Deploy Securely

Create a minimal Flask API with Prometheus metrics and MySQL connectivity. Build and push your image.

```bash
# Create backend source files
mkdir -p backend && cd backend

cat <<'EOF' > app.py
from flask import Flask, jsonify
import mysql.connector
import os, time

app = Flask(__name__)

# Optional: Prometheus metrics for Flask
try:
    from prometheus_flask_exporter import PrometheusMetrics
    metrics = PrometheusMetrics(app)
except Exception:
    metrics = None

def get_db_connection(max_retries=10, backoff=3):
    host = os.environ.get('DB_HOST', 'database')
    port = int(os.environ.get('DB_PORT', '3306'))
    db   = os.environ.get('DB_NAME', 'webapp_db')
    user = os.environ.get('DB_USER', 'webapp_user')
    pwd  = os.environ.get('DB_PASSWORD')
    for i in range(max_retries):
        try:
            conn = mysql.connector.connect(host=host, port=port, database=db, user=user, password=pwd)
            return conn
        except Exception as e:
            print(f"DB connection attempt {i+1} failed: {e}")
            time.sleep(backoff)
    return None

@app.route('/api/health')
def health():
    return jsonify({'status': 'ok', 'service': 'backend'})

@app.route('/api/data')
def data():
    conn = get_db_connection()
    if not conn:
        return jsonify({'error': 'DB connection failed'}), 500
    try:
        cur = conn.cursor()
        cur.execute('SELECT content, created_at FROM messages ORDER BY id DESC LIMIT 1')
        row = cur.fetchone()
        conn.close()
        if row:
            return jsonify({'message': row[0], 'created_at': str(row[1])})
        return jsonify({'message': 'No data yet'})
    except Exception as e:
        return jsonify({'error': str(e)}), 500

if __name__ == '__main__':
    port = int(os.environ.get('APP_PORT', '5000'))
    app.run(host='0.0.0.0', port=port)
EOF

cat <<'EOF' > requirements.txt
flask==3.0.3
mysql-connector-python==9.0.0
prometheus-flask-exporter==0.23.0
gunicorn==22.0.0
EOF

cat <<'EOF' > Dockerfile
FROM python:3.11-slim

# Create non-root user
RUN addgroup --system app && adduser --system --ingroup app --uid 10001 app

WORKDIR /app
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py ./

USER 10001:10001
ENV APP_PORT=5000
EXPOSE 5000

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "2", "app:app"]
EOF

# Build and push (Docker Hub). Login first if needed: docker login
IMAGE="$BACKEND_IMAGE"
docker build -t "$IMAGE" .
docker push "$IMAGE"

cd ..
```

Deploy the backend with securityContext, resources, probes, PDB, and HPA.

```bash
cat <<EOF | kubectl apply -n "$NS" -f -
apiVersion: v1
kind: Service
metadata:
  name: backend
  labels:
    app: backend
spec:
  selector:
    app: backend
  ports:
  - name: http
    port: 5000
    targetPort: 5000
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  labels:
    app: backend
spec:
  replicas: ${BACKEND_REPLICAS}
  revisionHistoryLimit: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
        tier: backend
    spec:
      terminationGracePeriodSeconds: 30
      containers:
      - name: app
        image: ${BACKEND_IMAGE}
        ports:
        - containerPort: 5000
          name: http
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: database-config
              key: DB_HOST
        - name: DB_PORT
          valueFrom:
            configMapKeyRef:
              name: database-config
              key: DB_PORT
        - name: DB_NAME
          valueFrom:
            configMapKeyRef:
              name: database-config
              key: MYSQL_DATABASE
        - name: DB_USER
          valueFrom:
            configMapKeyRef:
              name: database-config
              key: MYSQL_USER
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: database-secret
              key: MYSQL_PASSWORD
        - name: APP_PORT
          value: "${APP_PORT}"
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        readinessProbe:
          httpGet:
            path: /api/health
            port: 5000
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /api/health
            port: 5000
          initialDelaySeconds: 30
          periodSeconds: 15
        startupProbe:
          httpGet:
            path: /api/health
            port: 5000
          failureThreshold: 30
          periodSeconds: 5
        securityContext:
          runAsNonRoot: true
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: backend
              topologyKey: kubernetes.io/hostname
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: backend-pdb
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: backend
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
EOF
```


## 5) Frontend Tier: NGINX with Ingress (no NodePort)

- Serves static HTML from ConfigMap
- Proxies /api/ to backend service
- Deployed with resource limits, non-root, affinity, PDB, HPA

```bash
# Frontend HTML
cat <<'EOF' > index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Multi-Tier Kubernetes App</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 40px; background: #f5f5f5; }
    .container { max-width: 800px; margin: auto; background: #fff; padding: 20px; border-radius: 8px; }
    .header { text-align: center; margin-bottom: 24px; }
    .button { background: #007acc; color: white; padding: 10px 16px; border: none; border-radius: 4px; cursor: pointer; margin: 5px; }
    .result { margin-top: 12px; padding: 10px; background: #e8f4f8; border-radius: 4px; white-space: pre-wrap; }
    .success { background: #e6ffe6; }
    .error { background: #ffe6e6; color: #b00020; }
  </style>
</head>
<body>
  <div class="container">
    <div class="header">
      <h1>Multi-Tier Kubernetes Application</h1>
      <p>Frontend → Backend → Database</p>
    </div>
    <button class="button" onclick="test('/api/health')">Test Backend Health</button>
    <button class="button" onclick="test('/api/data')">Test Database Connection</button>
    <div id="out" class="result"></div>
  </div>
  <script>
    async function test(path){
      const out = document.getElementById('out');
      out.textContent = 'Running...'; out.className = 'result';
      try {
        const res = await fetch(path);
        const data = await res.json();
        out.textContent = '✅ ' + JSON.stringify(data, null, 2);
        out.className = 'result success';
      } catch(e){
        out.textContent = '❌ ' + e.message;
        out.className = 'result error';
      }
    }
  </script>
</body>
</html>
EOF

kubectl create configmap frontend-html -n "$NS" --from-file=index.html \
  --dry-run=client -o yaml | kubectl apply -f -

# NGINX config
cat <<'EOF' > default.conf
server {
  listen 80;
  server_name _;
  location / {
    root /usr/share/nginx/html;
    index index.html;
  }
  location /api/ {
    proxy_pass http://backend:5000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
EOF

kubectl create configmap nginx-config -n "$NS" --from-file=default.conf \
  --dry-run=client -o yaml | kubectl apply -f -

# Frontend Service, Deployment, PDB, HPA
cat <<EOF | kubectl apply -n "$NS" -f -
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  selector:
    app: frontend
  ports:
  - name: http
    port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  replicas: ${FRONTEND_REPLICAS}
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
        tier: frontend
    spec:
      terminationGracePeriodSeconds: 15
      containers:
      - name: nginx
        image: nginx:1.25.5-alpine
        ports:
        - containerPort: 80
          name: http
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
        - name: conf
          mountPath: /etc/nginx/conf.d
        resources:
          requests:
            cpu: 50m
            memory: 64Mi
          limits:
            cpu: 200m
            memory: 256Mi
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 20
          periodSeconds: 15
        securityContext:
          runAsNonRoot: true
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
      volumes:
      - name: html
        configMap:
          name: frontend-html
      - name: conf
        configMap:
          name: nginx-config
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: frontend
              topologyKey: kubernetes.io/hostname
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: frontend-pdb
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: frontend
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: frontend
  minReplicas: 3
  maxReplicas: 6
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
EOF
```


## 6) Ingress Controller and Ingress

Install ingress-nginx if not present and publish the frontend via Ingress (no NodePort required).

```bash
# Install ingress-nginx via Helm (requires Helm installed)
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set controller.watchIngressWithoutClass=true

# Determine node IP (fallback to InternalIP) and compute nip.io host
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="ExternalIP")].address}')
if [ -z "${NODE_IP}" ]; then
  NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
fi
FQDN="${FQDN:-${NODE_IP}.nip.io}"
echo "Using host: ${FQDN}"

# Create Ingress
cat <<EOF | kubectl apply -n "$NS" -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend-ingress
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
spec:
  ingressClassName: nginx
  rules:
  - host: ${FQDN}
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80
EOF

# Wait for ingress load balancer IP/port or for NodePort exposure
kubectl -n ingress-nginx get svc -o wide
```


## 7) Network Policies (Zero-trust between tiers)

- Default deny all ingress/egress
- Allow only the minimal required paths: frontend→backend, backend→database, DNS egress, Ingress→frontend

```bash
cat <<EOF | kubectl apply -n "$NS" -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
spec:
  podSelector: {}
  policyTypes: [Egress]
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-policy
spec:
  podSelector:
    matchLabels:
      tier: frontend
  policyTypes: [Ingress, Egress]
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: ingress-nginx
    ports:
    - port: 80
      protocol: TCP
  egress:
  - to:
    - podSelector:
        matchLabels:
          tier: backend
    ports:
    - port: 5000
      protocol: TCP
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes: [Ingress, Egress]
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
    ports:
    - port: 5000
      protocol: TCP
  egress:
  - to:
    - podSelector:
        matchLabels:
          tier: database
    ports:
    - port: 3306
      protocol: TCP
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-policy
spec:
  podSelector:
    matchLabels:
      tier: database
  policyTypes: [Ingress]
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: backend
    ports:
    - port: 3306
      protocol: TCP
EOF
```


## 8) Observability: Metrics and Logging

Install Prometheus + Grafana and Loki + Promtail via Helm. Optional tracing via Jaeger (quickstart).

```bash
# Metrics: kube-prometheus-stack
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade --install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace

# Get Grafana admin password
kubectl get secret -n monitoring monitoring-grafana -o jsonpath='{.data.admin-password}' | base64 -d; echo
kubectl -n monitoring get svc -o wide | grep grafana

# Logging: Loki + Promtail
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm upgrade --install loki grafana/loki-stack \
  --namespace logging --create-namespace \
  --set grafana.enabled=false

# Optional tracing: Jaeger all-in-one (development only)
kubectl create namespace tracing --dry-run=client -o yaml | kubectl apply -f -
kubectl -n tracing apply -f https://github.com/jaegertracing/jaeger-operator/releases/download/v1.57.0/jaeger-operator.yaml
cat <<'EOF' | kubectl apply -n tracing -f -
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: simplest
spec:
  strategy: allInOne
EOF
```


## 9) Verify End-to-End

```bash
# Wait for all pods to be Ready
kubectl -n "$NS" get deploy,statefulset,pods,svc

# Frontend URL
echo "Open: http://${FQDN}"

# Quick sanity checks
kubectl -n "$NS" run tmp --rm -it --image=curlimages/curl --restart=Never -- \
  curl -s http://backend:5000/api/health
kubectl -n "$NS" run tmp --rm -it --image=curlimages/curl --restart=Never -- \
  curl -s http://backend:5000/api/data

# Logs
kubectl -n "$NS" logs deploy/backend --tail=50
kubectl -n "$NS" logs deploy/frontend --tail=50
kubectl -n "$NS" logs statefulset/database --tail=50
```


## 10) Troubleshooting

```bash
# Describe resources
kubectl -n "$NS" get all
kubectl -n "$NS" describe deploy/backend
kubectl -n "$NS" describe statefulset/database
kubectl -n "$NS" describe pod -l app=frontend

# Check endpoints
kubectl -n "$NS" get endpoints backend database frontend

# Exec into backend and test DB connectivity
BACKEND_POD=$(kubectl -n "$NS" get pods -l app=backend -o jsonpath='{.items[0].metadata.name}')
kubectl -n "$NS" exec -it "$BACKEND_POD" -- sh -c "apk add --no-cache mysql-client || true; mysql -h database -u\$DB_USER -p\$DB_PASSWORD \$DB_NAME -e 'SELECT VERSION();'"

# Network policies: verify DNS
FRONTEND_POD=$(kubectl -n "$NS" get pods -l app=frontend -o jsonpath='{.items[0].metadata.name}')
kubectl -n "$NS" exec -it "$FRONTEND_POD" -- nslookup backend || true
```


## 11) CI/CD and GitOps (Optional Quickstarts)

```bash
# Example: GitHub Actions runner step (pseudo)
# - Build and push backend image when app changes
# - Apply manifests on main branch
# This is illustrative; customize for your repo/registry

cat <<'EOF' > .github/workflows/deploy.yaml
name: Deploy Backend
on:
  push:
    paths:
      - backend/**
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: docker.io
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Build and push
        run: |
          IMAGE=docker.io/${{ secrets.DOCKERHUB_USERNAME }}/mtk-backend:${{ github.sha }}
          docker build -t $IMAGE backend
          docker push $IMAGE
      - name: Deploy (kubectl)
        env:
          KUBECONFIG_B64: ${{ secrets.KUBECONFIG_B64 }}
        run: |
          echo "$KUBECONFIG_B64" | base64 -d > kubeconfig
          export KUBECONFIG=$(pwd)/kubeconfig
          kubectl -n ${NS} set image deploy/backend app=docker.io/${{ secrets.DOCKERHUB_USERNAME }}/mtk-backend:${{ github.sha }}
EOF
```

```bash
# GitOps quickstart with Argo CD (illustrative)
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm upgrade --install argocd argo/argo-cd -n argocd --create-namespace
kubectl -n argocd get svc -o wide
# Then point an Application to your Git repo path with manifests.
```


## 12) Cleanup

```bash
kubectl delete namespace "$NS" --wait=true || true
kubectl delete namespace monitoring logging tracing argocd ingress-nginx --wait=false || true
```


## Why These Improvements Matter

- Security: Avoids plaintext secrets, restricts pod communications, runs as non-root, enforces resource limits.
- Reliability: Stateful database with persistence, health probes, graceful termination, PDBs, HPA, and anti-affinity for resilience.
- Modern Practices: Image built from source, Ingress-based exposure, helm-managed add-ons, clean YAML and environment separation.
- Observability: Metrics, logging, and optional tracing enable rapid diagnosis and performance visibility.
- Education: Command-by-command explanations with clear verification and troubleshooting.
