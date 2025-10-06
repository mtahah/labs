# 🌐 Kubernetes Advanced Networking Architecture

> **Expert-Level Technical Reference**  
> Deep dive into CNI plugins, packet flow, NetworkPolicy, service mesh integration, and production networking patterns.

---

## 📋 Table of Contents

1. [Kubernetes Networking Model](#kubernetes-networking-model)
2. [CNI Plugin Architecture](#cni-plugin-architecture)
3. [Complete Packet Flow](#complete-packet-flow)
4. [NetworkPolicy Deep Dive](#networkpolicy-deep-dive)
5. [Service Mesh Integration](#service-mesh-integration)
6. [Production Networking Patterns](#production-networking-patterns)

---

## Kubernetes Networking Model

### **Fundamental Requirements**

Kubernetes imposes three non-negotiable networking rules:

```
Rule 1: All pods can communicate with all other pods
        - Without NAT
        - Across all nodes
        - Flat network space

Rule 2: All nodes can communicate with all pods
        - Without NAT
        - Bidirectional communication
        
Rule 3: Pod IP = Same IP everywhere
        - IP pod sees itself as = IP others see
        - No port mapping like Docker
        - Simplifies service discovery
```

**Why These Rules?**

```
Traditional Docker Networking (What Kubernetes Avoids):

Container A (172.17.0.2) on host 10.0.1.5
    ↓
docker0 bridge (172.17.0.1)
    ↓
NAT (iptables MASQUERADE)
    ↓
Host IP (10.0.1.5:random_port)
    ↓
External network

Problems:
- Port conflicts (only one container per host port)
- Complex NAT traversal
- IP visibility mismatch
- Doesn't scale to thousands of containers

Kubernetes Flat Network:

Pod A (10.244.1.5) on node-1
    ↓
Direct route via overlay network
    ↓
Pod B (10.244.2.8) on node-2

Benefits:
- No port conflicts (each pod has unique IP)
- No NAT complexity
- Simplified service discovery
- Microservices-friendly architecture
```

### **IP Address Management (IPAM)**

```
Typical IP allocation:

Cluster CIDR: 10.244.0.0/16  (65,536 IPs for pods)
Service CIDR: 10.96.0.0/12   (1,048,576 IPs for services)
Node CIDR: 10.0.0.0/16       (Cloud VPC network)

Per-Node Pod CIDR allocation:
- node-1: 10.244.1.0/24 (254 usable IPs)
- node-2: 10.244.2.0/24 (254 usable IPs)
- node-3: 10.244.3.0/24 (254 usable IPs)
...

Pod IP assignment (node-1):
- pod-a: 10.244.1.5
- pod-b: 10.244.1.6
- pod-c: 10.244.1.7

Service IP assignment (cluster-wide):
- kubernetes: 10.96.0.1 (API Server)
- kube-dns: 10.96.0.10 (CoreDNS)
- my-service: 10.96.5.20

Node IP assignment (cloud provider):
- node-1: 10.0.1.5
- node-2: 10.0.1.6
- node-3: 10.0.1.7
```

---

## CNI Plugin Architecture

### **What is CNI?**

Container Network Interface (CNI) is a specification and set of libraries for configuring network interfaces in Linux containers.

**CNI Workflow:**

```
1. kubelet calls container runtime: "Create pod"
   ↓
2. Runtime creates pod sandbox (pause container)
   ↓
3. Runtime calls CNI plugin: ADD command
   {
     "cniVersion": "0.4.0",
     "name": "k8s-pod-network",
     "type": "calico",
     "ipam": {
       "type": "calico-ipam"
     }
   }
   ↓
4. CNI plugin:
   a. Creates network namespace for pod
   b. Assigns IP address (10.244.1.5)
   c. Creates veth pair (virtual ethernet)
      - One end in pod namespace (eth0)
      - Other end in host namespace (cali-abc123)
   d. Configures routes in pod namespace
   e. Sets up iptables rules (if needed)
   ↓
5. CNI plugin returns result:
   {
     "cniVersion": "0.4.0",
     "interfaces": [...],
     "ips": [{
       "version": "4",
       "address": "10.244.1.5/24",
       "gateway": "10.244.1.1"
     }]
   }
   ↓
6. Runtime starts application containers in pod
   ↓
7. Pod has network connectivity
```

### **CNI Plugin Comparison**

| Plugin | Architecture | Data Plane | Performance | Features | Best For |
|--------|-------------|-----------|-------------|----------|----------|
| **Calico** | BGP or VXLAN | Linux routing or overlay | High (wire speed with BGP) | NetworkPolicy, BGP peering, dual-stack | Large clusters, security focus, hybrid cloud |
| **Cilium** | eBPF-based | XDP/eBPF | Highest (kernel bypass) | L7 policies, service mesh, observability (Hubble) | Modern clusters, performance-critical, L7 security |
| **Flannel** | VXLAN overlay | VXLAN encapsulation | Moderate | Simple, stable, easy setup | Small clusters, simplicity over features |
| **Weave** | Mesh networking | Encrypted mesh | Moderate | Automatic encryption, multi-cloud | Multi-cloud, encryption required |
| **Antrea** | OVS-based | Open vSwitch | High | NetworkPolicy, Windows support | VMware environments, Windows nodes |

---

## Calico Deep Dive

### **Architecture Modes**

**1. BGP Mode (Default - No Encapsulation)**

```
┌─────────────────────────────────────────────────────┐
│ Node 1 (10.244.1.0/24)                              │
│                                                     │
│  ┌────────┐  ┌────────┐  ┌────────┐              │
│  │ Pod A  │  │ Pod B  │  │ Pod C  │              │
│  │.1.5    │  │.1.6    │  │.1.7    │              │
│  └───┬────┘  └───┬────┘  └───┬────┘              │
│      └───────────┼───────────┘                     │
│                  │                                  │
│           ┌──────▼───────┐                         │
│           │ cali-xxx     │ (veth pairs)            │
│           └──────┬───────┘                         │
│                  │                                  │
│           ┌──────▼───────┐                         │
│           │ Linux Kernel │                         │
│           │   Routing    │                         │
│           └──────┬───────┘                         │
│                  │                                  │
│           ┌──────▼───────┐                         │
│           │  BIRD (BGP)  │ ← Advertises routes    │
│           └──────┬───────┘                         │
└──────────────────┼─────────────────────────────────┘
                   │
         BGP Peering (AS 64512)
                   │
┌──────────────────▼─────────────────────────────────┐
│ Node 2 (10.244.2.0/24)                              │
│           ┌──────────┐                              │
│           │BIRD (BGP)│ ← Receives routes           │
│           └────┬─────┘                              │
│                │                                     │
│         Kernel routing table:                       │
│         10.244.1.0/24 via 10.0.1.5 (node-1)        │
│         10.244.2.0/24 dev eth0 (local)             │
└─────────────────────────────────────────────────────┘
```

**Packet Flow (BGP Mode):**

```bash
# Pod A (10.244.1.5 on node-1) → Pod X (10.244.2.8 on node-2)

# Step 1: Pod A routing table
ip route show  # Inside pod
default via 169.254.1.1 dev eth0
169.254.1.1 dev eth0 scope link

# Step 2: Packet exits pod via veth pair to node-1

# Step 3: Node-1 routing table
ip route show  # On node-1
10.244.2.0/24 via 10.0.1.6 dev eth0  # Installed by Calico BGP
# Direct route to node-2

# Step 4: Packet routed to node-2 (no encapsulation!)
# Ethernet: [src: node-1 MAC, dst: node-2 MAC]
# IP: [src: 10.244.1.5, dst: 10.244.2.8]

# Step 5: Node-2 receives packet, routes to local pod
ip route show  # On node-2
10.244.2.8 dev cali-xyz scope link  # Direct to pod veth

# Result: Wire-speed performance, no overhead
```

**2. VXLAN Mode (Overlay Network)**

```
When to use VXLAN:
- BGP not supported in network infrastructure
- Cloud environments without BGP
- Need network isolation

Packet Structure:
┌──────────────────────────────────────────┐
│ Outer Ethernet Header                    │
│ src MAC: node-1, dst MAC: node-2         │
├──────────────────────────────────────────┤
│ Outer IP Header                          │
│ src IP: 10.0.1.5 (node-1)                │
│ dst IP: 10.0.1.6 (node-2)                │
├──────────────────────────────────────────┤
│ UDP Header (port 4789)                   │
├──────────────────────────────────────────┤
│ VXLAN Header (VNI: 4096)                 │
├──────────────────────────────────────────┤
│ Inner Ethernet Header                    │
├──────────────────────────────────────────┤
│ Inner IP Header                          │
│ src IP: 10.244.1.5 (pod-A)               │
│ dst IP: 10.244.2.8 (pod-X)               │
├──────────────────────────────────────────┤
│ TCP/UDP/etc payload                      │
└──────────────────────────────────────────┘

Overhead:
- VXLAN header: 50 bytes
- MTU reduction: 1500 - 50 = 1450 bytes
- Performance impact: 5-10%
```

**Configuration:**

```yaml
# Calico installation with VXLAN
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 10.244.0.0/16
      encapsulation: VXLAN  # or IPIPCrossSubnet, None (BGP)
      natOutgoing: true
      nodeSelector: all()
```

---

## Cilium Deep Dive

### **eBPF Revolution**

Cilium uses eBPF (extended Berkeley Packet Filter) to implement networking in the Linux kernel, bypassing traditional netfilter/iptables.

**Traditional vs eBPF:**

```
Traditional Stack (iptables):
Application → Socket → Netfilter hooks → iptables rules → Network

eBPF Stack:
Application → Socket → eBPF programs (in kernel) → Network
             ↑
        No netfilter traversal!
```

**Performance Benefits:**

```
Benchmark: 10,000 NetworkPolicies

iptables:
- Rule evaluation: O(n) linear scan
- 10,000 rules = check up to 10,000 rules per packet
- Latency: 50ms (p99)
- CPU: 80% at 100K packets/sec

Cilium eBPF:
- Rule evaluation: O(1) hash table lookup
- 10,000 rules = single hash lookup
- Latency: 0.5ms (p99)
- CPU: 15% at 100K packets/sec

Improvement: 100x latency reduction, 5x CPU reduction
```

### **L7-Aware Network Policies**

```yaml
# Traditional NetworkPolicy (L3/L4 only)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: traditional-policy
spec:
  podSelector:
    matchLabels:
      app: api
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
  # Can only filter by: IP, port
  # Cannot inspect: HTTP method, path, headers

---
# Cilium NetworkPolicy (L7-aware!)
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: l7-http-policy
spec:
  endpointSelector:
    matchLabels:
      app: api
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: frontend
    toPorts:
    - ports:
      - port: "80"
        protocol: TCP
      rules:
        http:
        - method: "GET"
          path: "/api/v1/users"  # Regex supported!
        - method: "POST"
          path: "/api/v1/orders"
          headers:
          - "Authorization: Bearer .*"  # Header inspection!
        - method: "GET"
          path: "/api/v1/products.*"
          
  # Result: Block GET /api/v1/admin (not in allow list)
  #         Block POST without Authorization header
```

**How eBPF Implements L7:**

```
Packet arrives at pod:
    ↓
1. XDP (eXpress Data Path) program runs
   - Earliest hook point (NIC driver)
   - Can drop/redirect packets instantly
    ↓
2. Socket-level eBPF program
   - Inspects HTTP headers
   - Matches against policy (hash table lookup)
   - Decision: ALLOW or DROP
    ↓
3. If ALLOW: Packet forwarded to application
   If DROP: Packet dropped (no response)

All in kernel space, no userspace context switches!
```

### **Hubble Observability**

Cilium includes Hubble for network observability:

```bash
# Install Hubble CLI
hubble observe --pod frontend-abc123

# Output: Every packet with full context
TIMESTAMP  FROM            TO              VERDICT  SUMMARY
10:30:01   frontend:52341  api:80          ALLOWED  HTTP/1.1 GET /api/users
10:30:01   api:3306        mysql:3306      ALLOWED  MySQL query
10:30:02   api:80          frontend:52341  ALLOWED  HTTP/1.1 200 OK
10:30:03   frontend:52342  api:80          DROPPED  HTTP/1.1 POST /api/admin (policy denied)

# Service map
hubble observe --output=compact --last=100 | \
  hubble observe --output=json | \
  jq -r '[.source.labels[],.destination.labels[]] | @csv'
  
# Generate visual service graph
hubble observe --follow --output=compact | \
  cilium-servicemap
  
# Result: Real-time network visibility without sidecars
```

---

## Complete Packet Flow

### **Scenario: Pod-to-Service Communication**

```
Setup:
- Pod A: 10.244.1.5 on node-1
- Service "backend": 10.96.5.20
- Backend pods: 10.244.2.8 (node-2), 10.244.3.9 (node-3)

Application: curl http://backend.default.svc.cluster.local
```

**Phase 1: DNS Resolution**

```
T+0s: App initiates: curl http://backend.default.svc.cluster.local

T+1s: DNS query process
      Container /etc/resolv.conf:
      nameserver 10.96.0.10  # CoreDNS
      search default.svc.cluster.local svc.cluster.local cluster.local
      options ndots:5

T+2s: Resolver attempts (due to ndots:5):
      1. "backend" (< 5 dots) → Append search domain
      2. "backend.default.svc.cluster.local" (query DNS)

T+3s: DNS packet structure:
      Source: 10.244.1.5:53241
      Dest: 10.96.0.10:53 (CoreDNS Service IP)
      Query: backend.default.svc.cluster.local A record

T+4s: kube-proxy DNAT (iptables/IPVS)
      Dest: 10.96.0.10:53 → 10.244.2.15:53 (CoreDNS pod)

T+5s: CoreDNS receives query
      Lookup: backend.default.svc.cluster.local
      Response: 10.96.5.20 (Service ClusterIP)

T+6s: DNS response returns to Pod A
      backend.default.svc.cluster.local = 10.96.5.20
      TTL: 30 seconds (cached)

Total DNS time: 6ms (or 0.1ms with NodeLocal DNSCache)
```

**Phase 2: Service Load Balancing**

```
T+7s: Application creates HTTP request
      GET / HTTP/1.1
      Host: backend.default.svc.cluster.local

T+8s: Kernel creates packet
      Source: 10.244.1.5:52345 (ephemeral port)
      Dest: 10.96.5.20:80 (Service ClusterIP)

T+9s: Packet exits pod via veth pair

T+10s: iptables PREROUTING (NAT table)
       KUBE-SERVICES chain:
       -d 10.96.5.20/32 -p tcp --dport 80 -j KUBE-SVC-BACKEND

T+11s: KUBE-SVC-BACKEND chain (load balancing):
       -m statistic --mode random --probability 0.5 -j KUBE-SEP-1
       # 50% chance: endpoint 10.244.2.8:8080
       
       -j KUBE-SEP-2
       # Remaining 50%: endpoint 10.244.3.9:8080

T+12s: Selected KUBE-SEP-1 (10.244.2.8:8080)
       DNAT applied:
       Before: [10.244.1.5:52345 → 10.96.5.20:80]
       After:  [10.244.1.5:52345 → 10.244.2.8:8080]

T+13s: Connection tracking entry created:
       conntrack -L | grep 10.244.1.5
       src=10.244.1.5 dst=10.96.5.20 sport=52345 dport=80 \
       src=10.244.2.8 dst=10.244.1.5 sport=8080 dport=52345
       # Tracks forward and reverse mapping
```

**Phase 3: Inter-Node Routing**

```
T+14s: Node-1 routing decision
       ip route show
       10.244.2.0/24 via 10.0.1.6 dev eth0  # Route to node-2

T+15s: CNI plugin encapsulation (if VXLAN):
       Outer IP: [10.0.1.5 → 10.0.1.6]
       VXLAN header: VNI 4096
       Inner IP: [10.244.1.5 → 10.244.2.8]

T+16s: Packet transmitted to node-2

T+17s: Node-2 receives packet
       VXLAN decapsulation (if applicable)
       Inner packet extracted: [10.244.1.5 → 10.244.2.8]

T+18s: Node-2 routing table
       ip route show
       10.244.2.8 dev cali-xyz scope link
       # Direct delivery to pod veth

T+19s: Packet forwarded through veth pair to pod namespace

T+20s: Backend pod receives packet at 10.244.2.8:8080
```

**Phase 4: Response Journey**

```
T+21s: Backend pod sends HTTP response
       Source: 10.244.2.8:8080
       Dest: 10.244.1.5:52345

T+22s: Packet exits pod via veth pair

T+23s: iptables POSTROUTING (node-2)
       Connection tracking reverse mapping:
       Original dest was Service IP (10.96.5.20)
       
       SNAT reverse:
       Before: [10.244.2.8:8080 → 10.244.1.5:52345]
       After:  [10.96.5.20:80 → 10.244.1.5:52345]
       # Pod A thinks response from Service, not backend pod!

T+24s: Route to node-1 (reverse path)

T+25s: VXLAN encapsulation (if applicable)

T+26s: Transmit to node-1

T+27s: Node-1 receives response

T+28s: VXLAN decapsulation

T+29s: Route to Pod A: 10.244.1.5 dev cali-abc

T+30s: Forward through veth pair to Pod A

T+31s: Pod A receives response
       Source: 10.96.5.20:80  # Sees Service IP!
       Dest: 10.244.1.5:52345
       HTTP/1.1 200 OK

Total round-trip time: 24ms (typical)
```

**Latency Breakdown:**

```
Component                Time
─────────────────────────────
DNS resolution:          6ms
Service load balancing:  5ms
VXLAN encap/decap:      1ms
Network transit:        10ms
Backend processing:      2ms
─────────────────────────────
Total:                  24ms
```

---

## NetworkPolicy Deep Dive

### **Default Behavior**

```
Without NetworkPolicy:
┌─────┐     ┌─────┐     ┌─────┐
│Pod A│────▶│Pod B│────▶│Pod C│
└─────┘     └─────┘     └─────┘
   ▲           ▲           ▲
   └───────────┴───────────┘
   All pods can communicate freely
   (flat network, no restrictions)

With NetworkPolicy:
┌─────┐     ┌─────┐     ┌─────┐
│Pod A│────▶│Pod B│  ✗  │Pod C│
└─────┘     └─────┘     └─────┘
   Explicit allow required
   (default deny when policy applied)
```

### **Policy Structure**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api  # Policy applies to these pods
  
  policyTypes:  # CRITICAL: Specify both!
  - Ingress     # Controls incoming traffic
  - Egress      # Controls outgoing traffic
  
  ingress:
  - from:  # Source selectors (ANY match = allow)
    - podSelector:
        matchLabels:
          app: frontend
    - namespaceSelector:
        matchLabels:
          name: trusted
    - ipBlock:
        cidr: 10.0.0.0/8
        except:
        - 10.0.1.0/24
    ports:  # Destination ports
    - protocol: TCP
      port: 8080
  
  egress:
  - to:  # Destination selectors
    - podSelector:
        matchLabels:
          app: database
    ports:  # Destination ports
    - protocol: TCP
      port: 5432
```

### **Common Patterns**

**Pattern 1: Default Deny All**

```yaml
# Deny all ingress traffic to namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: production
spec:
  podSelector: {}  # Matches ALL pods
  policyTypes:
  - Ingress
  # No ingress rules = deny all

---
# Deny all egress traffic from namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Egress
  # No egress rules = deny all
```

**Pattern 2: Three-Tier Application**

```yaml
# Frontend: Accept from Ingress, talk to Backend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: frontend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector:
        matchLabels:
          tier: backend
    ports:
    - protocol: TCP
      port: 8080
  - to:  # DNS
    - namespaceSelector:
        matchLabels:
          name: kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53

---
# Backend: Accept from Frontend, talk to Database
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          tier: database
    ports:
    - protocol: TCP
      port: 5432
  - to:  # DNS
    - namespaceSelector:
        matchLabels:
          name: kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53

---
# Database: Accept only from Backend, no egress except DNS
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: database
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: backend
    ports:
    - protocol: TCP
      port: 5432
  egress:
  - to:  # DNS only (no external access!)
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

**Pattern 3: External API Whitelisting**

```yaml
# Allow egress only to specific external IPs
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: external-api-whitelist
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: payment-service
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 52.45.67.0/24  # Payment gateway network
    ports:
    - protocol: TCP
      port: 443
  - to:  # DNS
    - namespaceSelector: {}
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
  # Result: Can only reach payment gateway + DNS
  #         Blocked from all other external IPs
```

### **NetworkPolicy Enforcement**

**How CNI Plugins Implement Policies:**

```
Calico (iptables-based):

Pod receives traffic:
    ↓
1. Packet arrives at veth pair (cali-abc123)
    ↓
2. iptables filter chain: cali-fw-cali-abc123
    ↓
3. Check ingress policy rules:
   -A cali-fw-cali-abc123 -m set --match-set cali-allowed-ingress src -j ACCEPT
   -A cali-fw-cali-abc123 -j DROP  # Default deny
    ↓
4. If ACCEPT: Forward to pod
   If DROP: Packet dropped

ipset (for efficiency):
# Instead of 1000 iptables rules, use ipset:
ipset create cali-allowed-ingress hash:net
ipset add cali-allowed-ingress 10.244.1.0/24
ipset add cali-allowed-ingress 10.244.2.0/24

iptables -A cali-fw-cali-abc123 -m set --match-set cali-allowed-ingress src -j ACCEPT
# Single rule checks entire IP set

Cilium (eBPF-based):

Pod receives traffic:
    ↓
1. Packet arrives at veth pair
    ↓
2. eBPF program attached to veth
    ↓
3. Hash table lookup (policy map):
   key = hash(src_ip, dst_ip, port, protocol)
   value = policy_verdict (ALLOW/DROP)
    ↓
4. O(1) lookup time (constant)
    ↓
5. If ALLOW: Forward to pod
   If DROP: Drop packet

Performance:
- Calico: O(n) rule evaluation
- Cilium: O(1) hash lookup
- 10,000 policies: Cilium 100x faster
```

### **Troubleshooting NetworkPolicy**

```bash
# Test connectivity between pods
kubectl run test-client --image=busybox --rm -it -- /bin/sh
wget -O- http://backend-service:8080
# If timeout, check NetworkPolicy

# Verify policy applied
kubectl get networkpolicy -n production
kubectl describe networkpolicy frontend-policy -n production

# Check pod labels match policy selectors
kubectl get pods --show-labels -n production

# CNI-specific debugging

# Calico: Check iptables rules
kubectl exec -n kube-system calico-node-xyz -- iptables-save | grep cali

# Cilium: Check eBPF programs
kubectl exec -n kube-system cilium-xyz -- cilium endpoint list
kubectl exec -n kube-system cilium-xyz -- cilium policy get

# Packet capture
kubectl sniff pod-name -n production
# Requires ksniff plugin

# Common issues:
# 1. Forgot DNS egress rule (pods can't resolve names)
# 2. Only specified Ingress in policyTypes (egress = allow all!)
# 3. Pod labels don't match selector
# 4. Namespace labels missing
```

---

## Service Mesh Integration

### **Why Service Mesh?**

NetworkPolicy provides L3/L4 security, but modern apps need:
- Mutual TLS (mTLS) encryption
- Request-level authorization (L7)
- Distributed tracing
- Traffic management (canary, circuit breaking)
- Observability (metrics without instrumentation)

### **Istio Architecture**

```
┌──────────────────────────────────────────────┐
│ Control Plane (istiod)                       │
│ - Pilot: Service discovery, config          │
│ - Citadel: Certificate management (mTLS)    │
│ - Galley: Configuration validation          │
└────────────────┬─────────────────────────────┘
                 │ xDS API (config push)
                 │
        ┌────────┴────────┬────────────┐
        │                 │            │
┌───────▼────────┐ ┌──────▼───────┐ ┌─▼──────────┐
│ Pod A          │ │ Pod B        │ │ Pod C      │
│ ┌────┐ ┌────┐ │ │ ┌────┐┌────┐│ │┌────┐┌────┐│
│ │App │ │Envoy││ │ │App ││Envoy│││││App ││Envoy││
│ └────┘ └────┘ │ │ └────┘└────┘│ │└────┘└────┘│
└────────────────┘ └──────────────┘ └────────────┘
        │                 │            │
        └────────mTLS─────┴────────────┘
```

**Sidecar Pattern:**

```yaml
# Without Istio
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:v1
    ports:
    - containerPort: 8080

---
# With Istio (automatic injection)
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  annotations:
    sidecar.istio.io/inject: "true"
spec:
  initContainers:
  - name: istio-init  # Sets up iptables to redirect traffic
    image: proxyv2
  containers:
  - name: app
    image: myapp:v1
    ports:
    - containerPort: 8080
  - name: istio-proxy  # Envoy sidecar
    image: proxyv2
    ports:
    - containerPort: 15001  # Outbound traffic
    - containerPort: 15006  # Inbound traffic
    - containerPort: 15020  # Metrics

# Traffic flow:
# App → localhost:8080 → iptables redirect → Envoy (15001)
# → mTLS encryption → Network → Envoy on destination
# → mTLS decryption → App on destination
```

### **Automatic mTLS**

```yaml
# Enable strict mTLS for namespace
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT  # All traffic must be mTLS

# Result:
# - All pod-to-pod traffic automatically encrypted
# - Certificates issued by Istio CA
# - Automatic rotation every 24 hours
# - Zero application code changes
# - Plaintext traffic rejected

# Verification:
kubectl exec pod-a -- openssl s_client -connect pod-b:8080
# Shows TLS handshake, certificate chain
```

### **L7 Authorization**

```yaml
# Fine-grained HTTP-level policies
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: api-authz
  namespace: production
spec:
  selector:
    matchLabels:
      app: api
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/frontend/sa/frontend-sa"]
    to:
    - operation:
        methods: ["GET", "POST"]
        paths: ["/api/v1/*"]
        ports: ["8080"]
    when:
    - key: request.headers[authorization]
      values: ["Bearer *"]
    - key: request.time
      values: ["09:00:00-17:00:00"]  # Business hours only!

# Capabilities NetworkPolicy cannot provide:
# - HTTP method filtering
# - URL path matching
# - Header inspection
# - JWT validation
# - Time-based access
# - Service account identity
```

### **Traffic Management**

```yaml
# Canary deployment: 90% v1, 10% v2
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: api-canary
spec:
  hosts:
  - api-service
  http:
  - match:
    - headers:
        user-agent:
          regex: ".*Mobile.*"
    route:
    - destination:
        host: api-service
        subset: v2
      weight: 100
  - route:
    - destination:
        host: api-service
        subset: v1
      weight: 90
    - destination:
        host: api-service
        subset: v2
      weight: 10

---
# Circuit breaker
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: backend-circuit-breaker
spec:
  host: backend-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        http2MaxRequests: 100
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
```

---

## Production Networking Patterns

### **Multi-Tenancy Isolation**

```yaml
# Namespace-level isolation
apiVersion: v1
kind: Namespace
metadata:
  name: tenant-a
  labels:
    tenant: tenant-a

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
          tenant: tenant-a  # Only same tenant
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          tenant: tenant-a
  - to:  # DNS + Internet
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 10.0.0.0/8  # Block internal networks
    ports:
    - protocol: TCP
      port: 443
```

### **Zero-Trust Architecture**

```
Layers of Security:

Layer 1: NetworkPolicy (L3/L4)
- Deny all by default
- Whitelist required connections
- IP-based access control

Layer 2: Service Mesh mTLS (Transport)
- Encrypt all pod-to-pod traffic
- Certificate-based authentication
- Automatic rotation

Layer 3: Service Mesh Authorization (L7)
- HTTP method/path filtering
- JWT validation
- Service account identity

Layer 4: Application Authentication
- OAuth2/OIDC
- API keys
- User authentication

Result: Defense in depth, multiple security layers
```

### **Performance Optimization**

```yaml
# NodeLocal DNSCache: Reduce DNS latency
apiVersion: v1
kind: ConfigMap
metadata:
  name: node-local-dns
  namespace: kube-system
data:
  Corefile: |
    cluster.local:53 {
        errors
        cache {
                success 9984 30
                denial 9984 5
        }
        reload
        loop
        bind 169.254.20.10
        forward . 10.96.0.10
        prometheus :9253
    }

# Before: Pod → CoreDNS (2-5ms)
# After: Pod → Local cache (0.1ms)
# 20-50x improvement!

---
# IPVS mode for kube-proxy (large clusters)
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-proxy
  namespace: kube-system
data:
  config.conf: |
    apiVersion: kubeproxy.config.k8s.io/v1alpha1
    kind: KubeProxyConfiguration
    mode: "ipvs"
    ipvs:
      scheduler: "rr"  # Round-robin
      # Options: rr, lc (least connection), dh, sh
    # O(1) lookup vs O(n) iptables

---
# Cilium with XDP acceleration
apiVersion: cilium.io/v1alpha1
kind: CiliumConfig
metadata:
  name: cilium-config
spec:
  nativeRoutingCIDR: 10.244.0.0/16
  enableXDPAcceleration: true  # Process packets at NIC level
  # 10x faster than iptables
```

### **Network Troubleshooting Tools**

```bash
# 1. DNS debugging
kubectl run dnsutils --image=gcr.io/kubernetes-e2e-test-images/dnsutils:1.3 --rm -it -- /bin/sh
nslookup backend-service
dig backend-service.default.svc.cluster.local

# 2. Connectivity testing
kubectl run netshoot --image=nicolaka/netshoot --rm -it -- /bin/bash
curl http://backend-service:8080
nc -zv backend-service 8080

# 3. Packet capture
kubectl sniff pod-name -n production -f "port 8080"
# Requires krew plugin: kubectl krew install sniff

# 4. Service endpoint verification
kubectl get endpoints backend-service -o yaml
# Check if pods are listed

# 5. NetworkPolicy testing
kubectl run test-pod --image=busybox --rm -it -- wget -O- http://target-pod:8080
# If timeout, check NetworkPolicy

# 6. CNI plugin logs
# Calico
kubectl logs -n kube-system -l k8s-app=calico-node --tail=100

# Cilium
kubectl logs -n kube-system -l k8s-app=cilium --tail=100

# 7. kube-proxy debugging
kubectl logs -n kube-system kube-proxy-xyz --tail=100
# Check for errors syncing iptables/IPVS rules

# 8. Service mesh debugging
# Istio
istioctl proxy-status
istioctl proxy-config clusters pod-name -n production

# Check Envoy access logs
kubectl logs pod-name -c istio-proxy -n production
```

### **Monitoring & Observability**

```yaml
# Prometheus metrics for networking
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-rules
data:
  network.rules: |
    groups:
    - name: network
      rules:
      # DNS latency
      - alert: HighDNSLatency
        expr: histogram_quantile(0.99, rate(coredns_dns_request_duration_seconds_bucket[5m])) > 0.1
        annotations:
          summary: "DNS queries slow (p99 > 100ms)"
      
      # NetworkPolicy drops
      - alert: NetworkPolicyDrops
        expr: rate(cilium_policy_verdict_total{verdict="drop"}[5m]) > 10
        annotations:
          summary: "High rate of NetworkPolicy drops"
      
      # Service endpoint availability
      - alert: ServiceNoEndpoints
        expr: kube_service_spec_type{type="ClusterIP"} unless on(service,namespace) kube_endpoint_address_available > 0
        annotations:
          summary: "Service {{ $labels.service }} has no endpoints"
      
      # Connection tracking table full
      - alert: ConntrackTableFull
        expr: node_nf_conntrack_entries / node_nf_conntrack_entries_limit > 0.9
        annotations:
          summary: "Conntrack table 90% full on {{ $labels.instance }}"
          description: "Increase nf_conntrack_max"

# Grafana dashboards:
# - Kubernetes Networking (Cluster)
# - Kubernetes Networking (Namespace)
# - Kubernetes Networking (Workload)
# - Service Mesh (Istio/Linkerd)
```

---

## Summary

**Kubernetes Networking Principles:**

1. **Flat Network Model**: All pods can communicate without NAT
2. **Pod IP Stability**: Same IP from pod's perspective and externally
3. **Service Abstraction**: Stable virtual IPs for pod groups
4. **CNI Flexibility**: Pluggable network implementations

**CNI Plugin Selection:**

- **Calico**: Production standard, BGP or VXLAN, excellent NetworkPolicy support
- **Cilium**: Next-generation, eBPF-based, L7 policies, best performance
- **Flannel**: Simple overlay, good for small clusters
- **Choose based on**: Scale, performance requirements, L7 needs, ecosystem

**NetworkPolicy Best Practices:**

- Start with default deny-all policies
- Always include DNS egress rules
- Use namespace selectors for multi-tier apps
- Test policies before production deployment
- Monitor policy drops and adjust accordingly

**Service Mesh Considerations:**

- **Use when**: Need mTLS, L7 policies, advanced traffic management, observability
- **Avoid when**: Simple requirements, resource-constrained, operational complexity concerns
- **Popular choices**: Istio (feature-rich), Linkerd (lightweight), Cilium (eBPF-native)

**Performance Tuning:**

- NodeLocal DNSCache for DNS (20-50x improvement)
- IPVS mode for large clusters (>1000 services)
- Cilium eBPF for maximum performance (100x vs iptables at scale)
- Monitor conntrack table usage and tune limits

**Production Patterns:**

- Implement defense-in-depth (NetworkPolicy + mTLS + L7 authz)
- Use multi-tenancy isolation for shared clusters
- Monitor network metrics (latency, drops, errors)
- Regular security audits of network policies
- Automate policy generation where possible

**Common Pitfalls:**

- Forgetting DNS egress rules (pods can't resolve)
- Not specifying policyTypes (implicit allow)
- Over-complicated policies (hard to debug)
- No monitoring of policy effectiveness
- Assuming NetworkPolicy provides encryption (it doesn't - use mTLS)

This completes the advanced networking architecture deep dive. The next artifacts will cover storage, security, and enterprise use cases.# 🌐 Kubernetes Advanced Networking Architecture
