# Enhanced Docker Networking Lab - Custom Networks & Advanced Concepts

## Objectives
By the end of this lab, you will be able to:

- **Create and manage** custom Docker bridge networks with advanced configurations
- **Launch containers** within custom networks and verify inter-container connectivity
- **Use Docker network inspection** commands to explore and troubleshoot network configurations
- **Understand host networking** concepts and their practical applications in production
- **Troubleshoot Docker networking** issues using network disconnect/connect commands
- **Apply advanced networking** concepts for real-world container deployments and microservices

## Prerequisites
Before starting this lab, you should have:

- Basic understanding of Docker containers and images
- Familiarity with Linux command line operations
- Knowledge of basic networking concepts (IP addresses, subnets, ports)
- Completed previous Docker labs or equivalent experience
- Understanding of container lifecycle management

## Lab Environment Setup
**Al Nafi Cloud Machines**: This lab uses Al Nafi's pre-configured Linux-based cloud machines with Docker pre-installed. Simply click Start Lab to access your ready-to-use environment. No need to build your own VM or install Docker manually.

Your lab environment includes:
- **Ubuntu 20.04 LTS** with Docker Engine installed
- **Root access** for network configuration
- **All necessary networking tools** pre-installed (ping, wget, curl, netstat, etc.)

---

## Task 1: Create Custom Bridge Networks

### 🎯 Learning Focus: Understanding Docker Network Architecture

**Key Concept**: Docker networking provides isolation and communication channels between containers. While the default bridge network works for simple cases, custom networks offer better control, security, and features like automatic DNS resolution.

### Subtask 1.1: Understanding Docker Network Types

**💡 Learning Tip**: Always start by understanding your current environment. Docker creates three default networks that serve different purposes.

```bash
# Explore the default Docker network landscape
echo "🔍 Discovering default Docker networks..."
docker network ls
```

**Expected Output**:
```
NETWORK ID     NAME      DRIVER    SCOPE
3c7b8f9a2d1e   bridge    bridge    local
5e8d2c9f1a3b   host      host      local
1f4a7e9c2b5d   none      null      local
```

**📚 Understanding Default Networks**:
- **bridge**: Default network for containers, provides basic connectivity
- **host**: Containers share the host's network stack (no isolation)
- **none**: No networking (completely isolated)

```bash
# Get detailed information about the default bridge network
echo "🔍 Examining default bridge network configuration..."
docker network inspect bridge --format='{{json .IPAM.Config}}' | jq '.'
```

**Expected Output**:
```json
[
  {
    "Subnet": "172.17.0.0/16",
    "Gateway": "172.17.0.1"
  }
]
```

**🎓 Key Insight**: The default bridge uses automatic IP assignment, which limits control and doesn't provide container name resolution.

### Subtask 1.2: Create Your First Custom Bridge Network

**Key Concept**: Custom bridge networks provide better isolation, automatic DNS resolution, and more control over IP addressing.

```bash
# Create a custom bridge network for web applications
echo "🌐 Creating custom bridge network for web applications..."
docker network create --driver bridge webapp-network
```

**Expected Output**:
```
f8d9e7c6b4a2c1f5e8d9c7b6a4f2e1d8c9b7a5f3e2d1c8b6a4f2e1d9c8b7a5f4
```

**📚 What Just Happened**:
- Docker created a new bridge network with automatic subnet allocation
- The network gets its own subnet (different from default bridge)
- Containers on this network can resolve each other by name

```bash
# Verify the network creation and compare with defaults
echo "✅ Verifying custom network creation..."
docker network ls
```

**Expected Output**:
```
NETWORK ID     NAME             DRIVER    SCOPE
3c7b8f9a2d1e   bridge           bridge    local
5e8d2c9f1a3b   host             host      local
1f4a7e9c2b5d   none             null      local
f8d9e7c6b4a2   webapp-network   bridge    local
```

**🎉 Success Indicator**: Your `webapp-network` now appears in the list with a unique NETWORK ID.

### Subtask 1.3: Create a Network with Custom Configuration

**🎯 Learning Focus**: Advanced Network Configuration for Production Use

**Key Concept**: Production environments often require specific IP ranges to avoid conflicts with existing infrastructure or to implement security policies.

```bash
# Create an advanced custom network with specific IP configuration
echo "🏗️  Creating backend network with custom IP configuration..."
docker network create \
  --driver bridge \
  --subnet=172.20.0.0/16 \
  --ip-range=172.20.240.0/20 \
  --gateway=172.20.0.1 \
  backend-network
```

**Expected Output**:
```
2a1f8e9d7c6b5a4f3e2d1c9b8a7f6e5d4c3b2a1f9e8d7c6b5a4f3e2d1c0b9a8f7
```

**📚 Configuration Deep Dive**:
- **--subnet=172.20.0.0/16**: Defines the entire network range (65,534 possible IPs)
- **--ip-range=172.20.240.0/20**: Restricts container IPs to a smaller subset (4,094 IPs)
- **--gateway=172.20.0.1**: Sets the default gateway for containers

**💡 Production Tip**: Use specific subnets to avoid conflicts with corporate networks or cloud provider VPCs.

```bash
# Verify the custom configuration
echo "🔍 Inspecting custom backend network configuration..."
docker network inspect backend-network --format='{{json .IPAM}}' | jq '.'
```

**Expected Output**:
```json
{
  "Driver": "default",
  "Options": {},
  "Config": [
    {
      "Subnet": "172.20.0.0/16",
      "IPRange": "172.20.240.0/20",
      "Gateway": "172.20.0.1"
    }
  ]
}
```

### Subtask 1.4: Create Multiple Networks for Different Purposes

**🎯 Learning Focus**: Multi-Tier Architecture Network Design

**Key Concept**: Modern applications use multiple networks to implement security boundaries and separate different tiers (frontend, backend, database).

```bash
# Create a frontend network for user-facing services
echo "🎨 Creating frontend network for user-facing services..."
docker network create frontend-network
```

**Expected Output**:
```
9b8a7f6e5d4c3b2a1f0e9d8c7b6a5f4e3d2c1b0a9f8e7d6c5b4a3f2e1d0c9b8a7
```

```bash
# Create a database network for data persistence layer
echo "🗄️  Creating database network for data persistence..."
docker network create database-network
```

**Expected Output**:
```
c5b4a3f2e1d0c9b8a7f6e5d4c3b2a1f0e9d8c7b6a5f4e3d2c1b0a9f8e7d6c5b4a3
```

```bash
# Verify all networks are created successfully
echo "📊 Complete network inventory..."
docker network ls
```

**Expected Output**:
```
NETWORK ID     NAME               DRIVER    SCOPE
3c7b8f9a2d1e   bridge             bridge    local
2a1f8e9d7c6b   backend-network    bridge    local
c5b4a3f2e1d0   database-network   bridge    local
9b8a7f6e5d4c   frontend-network   bridge    local
5e8d2c9f1a3b   host               host      local
1f4a7e9c2b5d   none               null      local
f8d9e7c6b4a2   webapp-network     bridge    local
```

**🏗️ Architecture Overview**: You now have:
- **webapp-network**: General application network
- **backend-network**: API and business logic services
- **frontend-network**: User interface components
- **database-network**: Data persistence layer

---

## Task 2: Launch Containers Within Custom Networks

### 🎯 Learning Focus: Container Network Assignment and Multi-Network Connectivity

**Key Concept**: Containers can be assigned to specific networks at runtime, and can even belong to multiple networks simultaneously for complex communication patterns.

### Subtask 2.1: Deploy Containers in Custom Networks

```bash
# Deploy a web server container in the custom webapp network
echo "🚀 Deploying web server in webapp-network..."
docker run -d --name web-server --network webapp-network nginx:alpine
```

**Expected Output**:
```
8f7e6d5c4b3a2f1e0d9c8b7a6f5e4d3c2b1a0f9e8d7c6b5a4f3e2d1c0b9a8f7e6
```

```bash
# Deploy a client container in the same network
echo "🖥️  Deploying client container in webapp-network..."
docker run -d --name web-client --network webapp-network alpine:latest sleep 3600
```

**Expected Output**:
```
2d1c0b9a8f7e6d5c4b3a2f1e0d9c8b7a6f5e4d3c2b1a0f9e8d7c6b5a4f3e2d1c0
```

**📚 What's Happening**:
- Both containers are assigned to the same custom network
- They can communicate using container names (automatic DNS)
- They're isolated from containers on other networks

```bash
# Verify containers are running and note their network assignments
echo "✅ Verifying container deployment..."
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Networks}}"
```

**Expected Output**:
```
NAMES        STATUS              NETWORKS
web-client   Up 30 seconds       webapp-network
web-server   Up 1 minute         webapp-network
```

### Subtask 2.2: Deploy Multi-Container Application

**🎯 Learning Focus**: Complex Multi-Network Architecture

**Key Concept**: Real applications often need containers that communicate across multiple network segments. Docker allows containers to connect to multiple networks.

```bash
# Deploy a database container in the backend network
echo "🗄️  Deploying MySQL database in backend network..."
docker run -d --name mysql-db \
  --network backend-network \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=webapp \
  mysql:8.0
```

**Expected Output**:
```
a9f8e7d6c5b4a3f2e1d0c9b8a7f6e5d4c3b2a1f0e9d8c7b6a5f4e3d2c1b0a9f8e7
```

```bash
# Deploy an application server in the backend network
echo "⚙️  Deploying application server in backend network..."
docker run -d --name app-server \
  --network backend-network \
  nginx:alpine
```

**Expected Output**:
```
6d5c4b3a2f1e0d9c8b7a6f5e4d3c2b1a0f9e8d7c6b5a4f3e2d1c0b9a8f7e6d5c4
```

**💡 Advanced Networking**: Now connect the app-server to multiple networks:

```bash
# Connect app-server to frontend network as well (multi-network container)
echo "🌉 Connecting app-server to frontend network..."
docker network connect frontend-network app-server
```

**Expected Output**: (No output indicates success)

**📚 Multi-Network Benefits**:
- **app-server** can now communicate with both frontend and backend services
- Acts as a bridge between network tiers
- Implements proper separation of concerns

### Subtask 2.3: Verify Container Network Assignments

```bash
# Check comprehensive container status
echo "📊 Complete container and network status..."
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}\t{{.Networks}}"
```

**Expected Output**:
```
NAMES        STATUS          PORTS                 NETWORKS
app-server   Up 2 minutes    80/tcp                backend-network,frontend-network
mysql-db     Up 3 minutes    3306/tcp, 33060/tcp   backend-network
web-client   Up 5 minutes                          webapp-network
web-server   Up 6 minutes    80/tcp                webapp-network
```

```bash
# Deep dive into web-server network configuration
echo "🔍 Detailed network inspection of web-server..."
docker inspect web-server --format='{{json .NetworkSettings.Networks}}' | jq '.'
```

**Expected Output**:
```json
{
  "webapp-network": {
    "IPAMConfig": null,
    "Links": null,
    "Aliases": [
      "8f7e6d5c4b3a"
    ],
    "NetworkID": "f8d9e7c6b4a2c1f5e8d9c7b6a4f2e1d8c9b7a5f3e2d1c8b6a4f2e1d9c8b7a5f4",
    "EndpointID": "a1b2c3d4e5f6789012345678901234567890123456789012345678901234567890",
    "Gateway": "172.18.0.1",
    "IPAddress": "172.18.0.2",
    "IPPrefixLen": 16,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "MacAddress": "02:42:ac:12:00:02",
    "DriverOpts": null
  }
}
```

**🎓 Network Analysis**:
- **Gateway**: 172.18.0.1 (network's default route)
- **IPAddress**: 172.18.0.2 (container's assigned IP)
- **Aliases**: Container name for DNS resolution
- **MacAddress**: Unique identifier on the network segment

---

## Task 3: Verify Connectivity Between Containers

### 🎯 Learning Focus: Container Communication Patterns and Network Isolation

**Key Concept**: Custom networks provide automatic DNS resolution and network isolation. Containers can communicate by name within the same network but are isolated from other networks.

### Subtask 3.1: Test Basic Container-to-Container Communication

```bash
# Test connectivity between containers in the same network
echo "🔗 Testing container-to-container communication..."
docker exec -it web-client sh
```

**💡 You're now inside the web-client container. Run these commands:**

```bash
# Test network connectivity using ping
echo "🏓 Testing ping connectivity to web-server..."
ping -c 3 web-server
```

**Expected Output**:
```
PING web-server (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.045 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.043 ms
64 bytes from 172.18.0.2: seq=2 ttl=64 time=0.041 ms

--- web-server ping statistics ---
3 packets transmitted, 3 received, 0% packet loss
round-trip min/avg/max = 0.041/0.043/0.045 ms
```

```bash
# Test HTTP connectivity (application-level communication)
echo "🌐 Testing HTTP connectivity..."
wget -qO- http://web-server
```

**Expected Output**:
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and working...</p>
...
</html>
```

```bash
# Exit the container
exit
```

**🎉 Success Analysis**:
- **DNS Resolution**: `web-server` resolved to IP automatically
- **Network Connectivity**: Ping worked (Layer 3)
- **Application Connectivity**: HTTP worked (Layer 7)

### Subtask 3.2: Test Cross-Network Communication

**🎯 Learning Focus**: Network Isolation Security

**Key Concept**: Containers on different networks cannot communicate by default, providing security isolation.

```bash
# Deploy a container in the frontend network
echo "🎨 Deploying test container in frontend network..."
docker run -d --name frontend-app --network frontend-network alpine:latest sleep 3600
```

```bash
# Test cross-network communication (should fail)
echo "⚠️  Testing cross-network communication (expected to fail)..."
docker exec frontend-app ping -c 2 mysql-db
```

**Expected Output**:
```
ping: bad address 'mysql-db'
```

**📚 Security Insight**: This failure is **intentional and good**! It demonstrates:
- Network isolation working properly
- DNS resolution limited to same network
- Security boundaries enforced

```bash
# Verify the isolation by checking network memberships
echo "🔍 Verifying network isolation..."
echo "Frontend network containers:"
docker network inspect frontend-network --format='{{range .Containers}}{{.Name}} {{end}}'
echo "Backend network containers:"
docker network inspect backend-network --format='{{range .Containers}}{{.Name}} {{end}}'
```

**Expected Output**:
```
Frontend network containers:
frontend-app app-server 

Backend network containers:
mysql-db app-server 
```

### Subtask 3.3: Test Multi-Network Container Communication

**🎯 Learning Focus**: Bridge Container Communication Patterns

**Key Concept**: Containers connected to multiple networks can act as bridges, enabling controlled communication between network segments.

```bash
# Test app-server connectivity to backend services
echo "🌉 Testing multi-network container communication..."
docker exec app-server ping -c 2 mysql-db
```

**Expected Output**:
```
PING mysql-db (172.20.240.2): 56 data bytes
64 bytes from 172.20.240.2: seq=0 ttl=64 time=0.053 ms
64 bytes from 172.20.240.2: seq=1 ttl=64 time=0.047 ms

--- mysql-db ping statistics ---
2 packets transmitted, 2 received, 0% packet loss
round-trip min/avg/max = 0.047/0.050/0.053 ms
```

```bash
# Test frontend to app-server communication
echo "🔗 Testing frontend to bridge container communication..."
docker run --rm --network frontend-network alpine:latest ping -c 3 app-server
```

**Expected Output**:
```
PING app-server (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.048 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.045 ms
64 bytes from 172.19.0.3: seq=2 ttl=64 time=0.043 ms

--- app-server ping statistics ---
3 packets transmitted, 3 received, 0% packet loss
round-trip min/avg/max = 0.043/0.045/0.048 ms
```

**🎯 Architecture Success**: The app-server successfully acts as a bridge:
- Communicates with mysql-db in backend network
- Accessible from frontend network
- Enables controlled cross-network communication

---

## Task 4: Use Docker Network Inspect

### 🎯 Learning Focus: Network Troubleshooting and Configuration Analysis

**Key Concept**: The `docker network inspect` command is your primary tool for understanding network configurations, IP assignments, and connectivity issues.

### Subtask 4.1: Inspect Custom Network Configuration

```bash
# Get comprehensive network configuration
echo "🔍 Deep dive into webapp-network configuration..."
docker network inspect webapp-network
```

**Expected Output** (key sections):
```json
[
    {
        "Name": "webapp-network",
        "Id": "f8d9e7c6b4a2c1f5e8d9c7b6a4f2e1d8c9b7a5f3e2d1c8b6a4f2e1d9c8b7a5f4",
        "Created": "2025-08-17T10:15:30.123456789Z",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Containers": {
            "2d1c0b9a8f7e": {
                "Name": "web-client",
                "EndpointID": "abc123def456",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "8f7e6d5c4b3a": {
                "Name": "web-server",
                "EndpointID": "def456ghi789",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        }
    }
]
```

**📚 Configuration Analysis**:
- **IPAM**: IP Address Management shows automatic subnet allocation
- **Containers**: Shows all connected containers with their IPs
- **Driver**: Bridge driver provides Layer 2 connectivity
- **Scope**: Local means this network exists only on this host

### Subtask 4.2: Analyze Network Settings in Detail

```bash
# Extract specific network information using format filters
echo "🎯 Extracting IPAM configuration..."
docker network inspect webapp-network --format='{{json .IPAM.Config}}' | jq '.'
```

**Expected Output**:
```json
[
  {
    "Subnet": "172.18.0.0/16",
    "Gateway": "172.18.0.1"
  }
]
```

```bash
# List all containers and their IP addresses in the network
echo "🗺️  Container IP mapping in webapp-network..."
docker network inspect webapp-network --format='{{range .Containers}}{{.Name}}: {{.IPv4Address}}{{"\n"}}{{end}}'
```

**Expected Output**:
```
web-client: 172.18.0.3/16
web-server: 172.18.0.2/16
```

**💡 Troubleshooting Tip**: This information is crucial for:
- Debugging connectivity issues
- Understanding IP allocation
- Verifying network membership

### Subtask 4.3: Compare Different Network Types

```bash
# Compare default bridge with custom network configurations
echo "⚖️  Comparing network configurations..."
echo "=== DEFAULT BRIDGE NETWORK ==="
docker network inspect bridge --format='{{json .IPAM.Config}}' | jq '.'
echo ""
echo "=== CUSTOM BACKEND NETWORK ==="
docker network inspect backend-network --format='{{json .IPAM.Config}}' | jq '.'
```

**Expected Output**:
```
=== DEFAULT BRIDGE NETWORK ===
[
  {
    "Subnet": "172.17.0.0/16",
    "Gateway": "172.17.0.1"
  }
]

=== CUSTOM BACKEND NETWORK ===
[
  {
    "Subnet": "172.20.0.0/16",
    "IPRange": "172.20.240.0/20",
    "Gateway": "172.20.0.1"
  }
]
```

**📊 Key Differences**:
- **Default**: Automatic subnet (172.17.x.x)
- **Custom**: Specified subnet with restricted IP range
- **Control**: Custom networks offer precise IP management

```bash
# Check driver differences and capabilities
echo "🔧 Network driver capabilities comparison..."
echo "Default bridge options:"
docker network inspect bridge --format='{{.Options}}'
echo "Custom network options:"
docker network inspect backend-network --format='{{.Options}}'
```

---

## Task 5: Learn About Host Networking

### 🎯 Learning Focus: High-Performance Networking and System Integration

**Key Concept**: Host networking removes network isolation, allowing containers to use the host's network stack directly. This provides maximum performance but sacrifices security isolation.

### Subtask 5.1: Understanding Host Network Mode

```bash
# Deploy a container using host networking
echo "🏠 Creating container with host networking..."
docker run -d --name host-nginx --network host nginx:alpine
```

**Expected Output**:
```
7c6b5a4f3e2d1c0b9a8f7e6d5c4b3a2f1e0d9c8b7a6f5e4d3c2b1a0f9e8d7c6b5
```

**⚠️ Important**: With host networking, the container binds directly to the host's ports!

```bash
# Verify the container is using host networking
echo "🔍 Inspecting host networking configuration..."
docker inspect host-nginx --format='{{.HostConfig.NetworkMode}}'
```

**Expected Output**:
```
host
```

```bash
# Check if nginx is accessible on the host's IP
echo "🌐 Testing host network accessibility..."
curl -I localhost:80
```

**Expected Output**:
```
HTTP/1.1 200 OK
Server: nginx/1.21.6
Date: Sun, 17 Aug 2025 10:30:45 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 25 Jan 2022 15:04:25 GMT
Connection: keep-alive
ETag: "61f0168e-267"
Accept-Ranges: bytes
```

### Subtask 5.2: Compare Host vs Bridge Networking

```bash
# Compare network configurations
echo "🔬 Comparing host vs bridge networking..."
echo "=== HOST NETWORK CONTAINER ==="
docker inspect host-nginx --format='{{.NetworkSettings.NetworkMode}}'
docker inspect host-nginx --format='{{.NetworkSettings.IPAddress}}'

echo ""
echo "=== BRIDGE NETWORK CONTAINER ==="
docker inspect web-server --format='{{.NetworkSettings.NetworkMode}}'
docker inspect web-server --format='{{.NetworkSettings.IPAddress}}'
```

**Expected Output**:
```
=== HOST NETWORK CONTAINER ===
host


=== BRIDGE NETWORK CONTAINER ===
webapp-network
172.18.0.2
```

**📚 Key Differences**:
- **Host**: No IP address (uses host's network directly)
- **Bridge**: Gets assigned container IP within network subnet
- **Port binding**: Host network containers can conflict with host services

```bash
# Demonstrate the networking difference
echo "🔍 Network interface comparison..."
echo "Host container network interfaces:"
docker exec host-nginx ip addr show | grep -E "(eth|lo|docker)"
echo ""
echo "Bridge container network interfaces:"
docker exec web-server ip addr show
```

### Subtask 5.3: Host Network Use Cases

**💡 Learning Tip**: Host networking is ideal for system monitoring, network analysis, and high-performance applications that need direct hardware access.

```bash
# Create a network monitoring container
echo "📊 Creating network monitoring container..."
docker run --rm --network host alpine:latest ip addr show
```

**Expected Output** (shows ALL host network interfaces):
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:12:34:56:78 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
```

```bash
# Compare with bridge container (limited view)
echo "🔍 Bridge container's limited network view..."
docker run --rm --network bridge alpine:latest ip addr show
```

**Expected Output** (only container's interfaces):
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
27: eth0@if28: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

**🎯 Use Case Examples**:
- **Monitoring tools**: Need to see all network interfaces
- **Network scanners**: Require direct network access
- **High-performance apps**: Eliminate network overhead
- **System utilities**: Need host-level network visibility

---

## Task 6: Troubleshoot Networking Issues

### 🎯 Learning Focus: Network Debugging and Dynamic Network Management

**Key Concept**: Docker provides tools to dynamically modify container network connections, which is essential for troubleshooting and maintenance scenarios.

### Subtask 6.1: Identify Network Connectivity Problems

```bash
# Create a completely isolated container (no networking)
echo "🔒 Creating isolated container for troubleshooting demo..."
docker run -d --name isolated-app --network none alpine:latest sleep 3600
```

```bash
# Attempt to test connectivity (should fail)
echo "❌ Testing connectivity from isolated container (expected failure)..."
docker exec isolated-app ping -c 2 8.8.8.8
```

**Expected Output**:
```
ping: bad address '8.8.8.8'
```

```bash
# Check the container's network configuration
echo "🔍 Inspecting isolated container network..."
docker exec isolated-app ip addr show
```

**Expected Output**:
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
```

**📚 Isolation Analysis**: 
- **Only loopback**: No external network interfaces
- **No routing**: Cannot reach external networks
- **Complete isolation**: Maximum security but no connectivity

### Subtask 6.2: Use Docker Network Disconnect

**🎯 Learning Focus**: Dynamic Network Management for Maintenance

**Key Concept**: You can dynamically disconnect containers from networks without stopping them, useful for maintenance, security isolation, or troubleshooting.

```bash
# First, verify app-server's current network connections
echo "🔍 Current app-server network connections..."
docker inspect app-server --format='{{range $net, $conf := .NetworkSettings.Networks}}{{$net}} {{end}}'
```

**Expected Output**:
```
backend-network frontend-network 
```

```bash
# Disconnect app-server from frontend network
echo "🔌 Disconnecting app-server from frontend network..."
docker network disconnect frontend-network app-server
```

**Expected Output**: (No output indicates success)

```bash
# Verify the disconnection
echo "✅ Verifying network disconnection..."
docker network inspect frontend-network --format='{{range .Containers}}{{.Name}} {{end}}'
```

**Expected Output**:
```
frontend-app 
```

**🔍 Notice**: `app-server` is no longer listed in the frontend network.

```bash
# Test connectivity from frontend (should now fail)
echo "❌ Testing frontend to app-server connectivity (should fail now)..."
docker run --rm --network frontend-network alpine:latest ping -c 2 app-server
```

**Expected Output**:
```
ping: bad address 'app-server'
```

**📚 What Happened**:
- Container remained running (no downtime)
- Network interface removed dynamically
- DNS resolution no longer works from frontend network
- Backend connectivity still intact

### Subtask 6.3: Reconnect and Verify

```bash
# Reconnect app-server to frontend network
echo "🔌 Reconnecting app-server to frontend network..."
docker network connect frontend-network app-server
```

**Expected Output**: (No output indicates success)

```bash
# Verify successful reconnection
echo "✅ Verifying network reconnection..."
docker network inspect frontend-network --format='{{range .Containers}}{{.Name}}: {{.IPv4Address}}{{"\n"}}{{end}}'
```

**Expected Output**:
```
frontend-app: 172.19.0.2/16
app-server: 172.19.0.3/16
```

**🎉 Success**: app-server is back with a potentially new IP address.

```bash
# Test restored connectivity
echo "✅ Testing restored frontend to app-server connectivity..."
docker run --rm --network frontend-network alpine:latest ping -c 2 app-server
```

**Expected Output**:
```
PING app-server (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.045 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.043 ms

--- app-server ping statistics ---
2 packets transmitted, 2 received, 0% packet loss
round-trip min/avg/max = 0.043/0.044/0.045 ms
```

**💡 Production Use Case**: This technique is valuable for:
- **Rolling maintenance**: Temporarily isolate containers
- **Security incidents**: Quickly isolate compromised containers
- **Network troubleshooting**: Test connectivity scenarios
- **Blue-green deployments**: Switch traffic between versions

### Subtask 6.4: Advanced Troubleshooting

**🎯 Learning Focus**: Deep Network Analysis and Debugging

```bash
# Examine container's routing table
echo "🗺️  Checking container routing configuration..."
docker exec web-server ip route
```

**Expected Output**:
```
default via 172.18.0.1 dev eth0 
172.18.0.0/16 dev eth0 proto kernel scope link src 172.18.0.2 
```

**📚 Routing Analysis**:
- **Default route**: Traffic goes through network gateway (172.18.0.1)
- **Local network**: Direct connectivity within subnet (172.18.0.0/16)
- **Interface**: All traffic uses eth0 interface

```bash
# Check detailed network interface information
echo "🔍 Detailed network interface analysis..."
docker exec web-server ip addr show eth0
```

**Expected Output**:
```
28: eth0@if29: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.18.0.2/16 brd 172.18.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

**🔍 Interface Details**:
- **MTU 1500**: Standard Ethernet frame size
- **MAC address**: Unique identifier on network segment
- **Broadcast**: Network broadcast address
- **link-netnsid**: Network namespace identifier

```bash
# Test DNS resolution capabilities
echo "🌐 Testing DNS resolution within container..."
docker exec web-server nslookup web-client
```

**Expected Output**:
```
Server:		127.0.0.11
Address:	127.0.0.11:53

Name:	web-client
Address: 172.18.0.3
```

**📚 DNS Analysis**:
- **Server 127.0.0.11**: Docker's embedded DNS resolver
- **Automatic resolution**: Container names resolve to IP addresses
- **Custom network benefit**: Name resolution only works on custom networks

```bash
# Examine host-level network rules (requires host access)
echo "🔧 Checking Docker's iptables rules..."
sudo iptables -t nat -L DOCKER
```

**Expected Output** (partial):
```
Chain DOCKER (2 references)
target     prot opt source               destination         
RETURN     all  --  anywhere             anywhere            
DNAT       tcp  --  anywhere             anywhere             tcp dpt:80 to:172.18.0.2:80
```

**🔍 iptables Analysis**:
- **DNAT rules**: Port forwarding from host to containers
- **RETURN rules**: Traffic within Docker networks
- **Automatic management**: Docker maintains these rules

---

## Task 7: Clean Up and Network Management

### 🎯 Learning Focus: Proper Resource Management and Cleanup Procedures

**Key Concept**: Proper cleanup prevents resource leaks and ensures a clean environment for future deployments.

### Subtask 7.1: Remove Containers

```bash
# Stop all running containers gracefully
echo "🛑 Stopping all running containers..."
docker stop $(docker ps -q)
```

**Expected Output**:
```
host-nginx
app-server
mysql-db
frontend-app
web-client
web-server
isolated-app
```

```bash
# Remove all containers (stopped and running)
echo "🗑️  Removing all containers..."
docker rm $(docker ps -aq)
```

**Expected Output**:
```
host-nginx
app-server
mysql-db
frontend-app
web-client
web-server
isolated-app
```

```bash
# Verify container cleanup
echo "✅ Verifying container cleanup..."
docker ps -a
```

**Expected Output**:
```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

**🎉 Clean State**: No containers remain in the system.

### Subtask 7.2: Remove Custom Networks

**💡 Important**: Networks can only be removed when no containers are using them.

```bash
# Remove custom networks one by one
echo "🌐 Removing custom networks..."
docker network rm webapp-network
docker network rm backend-network
docker network rm frontend-network
docker network rm database-network
```

**Expected Output**:
```
webapp-network
backend-network
frontend-network
database-network
```

```bash
# Verify network cleanup
echo "✅ Verifying network cleanup..."
docker network ls
```

**Expected Output**:
```
NETWORK ID     NAME      DRIVER    SCOPE
3c7b8f9a2d1e   bridge    bridge    local
5e8d2c9f1a3b   host      host      local
1f4a7e9c2b5d   none      null      local
```

**✅ Back to Default**: Only the three default networks remain.

### Subtask 7.3: Network Pruning

```bash
# Use Docker's built-in cleanup for unused networks
echo "🧹 Pruning unused networks..."
docker network prune -f
```

**Expected Output**:
```
Total reclaimed space: 0B
```

**💡 Note**: Since we manually removed networks, pruning finds nothing to clean.

```bash
# Final verification - confirm clean state
echo "🎯 Final verification of clean environment..."
docker network ls
echo ""
docker ps -a
echo ""
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

**Expected Output**:
```
NETWORK ID     NAME      DRIVER    SCOPE
3c7b8f9a2d1e   bridge    bridge    local
5e8d2c9f1a3b   host      host      local
1f4a7e9c2b5d   none      null      local

CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

REPOSITORY   TAG       SIZE
nginx        alpine    23.4MB
mysql        8.0       447MB
alpine       latest    5.6MB
```

---

## Practical Examples and Real-World Use Cases

### Example 1: Microservices Architecture

**🎯 Learning Focus**: Production-Grade Multi-Tier Application Design

```bash
# Create networks for different application tiers
echo "🏗️  Setting up microservices network architecture..."
docker network create --driver bridge microservices-frontend
docker network create --driver bridge microservices-backend
docker network create --driver bridge microservices-database
```

```bash
# Deploy database tier (isolated)
echo "🗄️  Deploying database tier..."
docker run -d --name postgres-db \
  --network microservices-database \
  -e POSTGRES_PASSWORD=dbpass \
  -e POSTGRES_DB=microapp \
  postgres:13
```

```bash
# Deploy API service (connected to backend and database)
echo "⚙️  Deploying API service with multi-network connectivity..."
docker run -d --name api-service \
  --network microservices-backend \
  nginx:alpine

docker network connect microservices-database api-service
```

```bash
# Deploy frontend (connected to frontend and backend)
echo "🎨 Deploying frontend with controlled backend access..."
docker run -d --name frontend-app \
  --network microservices-frontend \
  -p 8080:80 \
  nginx:alpine

docker network connect microservices-backend frontend-app
```

**📊 Architecture Benefits**:
- **Database isolation**: Only API service can access database
- **Layered security**: Frontend cannot directly access database
- **Service communication**: Each tier communicates only with adjacent tiers
- **Scalability**: Each network can be scaled independently

```bash
# Verify the microservices architecture
echo "🔍 Verifying microservices network architecture..."
echo "Database network:"
docker network inspect microservices-database --format='{{range .Containers}}{{.Name}} {{end}}'
echo "Backend network:"
docker network inspect microservices-backend --format='{{range .Containers}}{{.Name}} {{end}}'
echo "Frontend network:"
docker network inspect microservices-frontend --format='{{range .Containers}}{{.Name}} {{end}}'
```

**Expected Output**:
```
Database network:
postgres-db api-service 
Backend network:
api-service frontend-app 
Frontend network:
frontend-app 
```

### Example 2: Development Environment Isolation

**🎯 Learning Focus**: Multi-Environment Development Setup

```bash
# Create isolated development environments
echo "👥 Creating isolated development environments..."
docker network create dev-env-team-a
docker network create dev-env-team-b
```

```bash
# Deploy Team A's development stack
echo "🅰️  Deploying Team A development environment..."
docker run -d --name team-a-web --network dev-env-team-a nginx:alpine
docker run -d --name team-a-db --network dev-env-team-a \
  -e MYSQL_ROOT_PASSWORD=teamApass \
  -e MYSQL_DATABASE=team_a_app \
  mysql:8.0
```

```bash
# Deploy Team B's development stack
echo "🅱️  Deploying Team B development environment..."
docker run -d --name team-b-web --network dev-env-team-b nginx:alpine
docker run -d --name team-b-db --network dev-env-team-b \
  -e MYSQL_ROOT_PASSWORD=teamBpass \
  -e MYSQL_DATABASE=team_b_app \
  mysql:8.0
```

**💡 Development Benefits**:
- **Complete isolation**: Teams can't interfere with each other
- **Identical setup**: Same configuration, different data
- **Resource efficiency**: Share host resources without conflicts
- **Easy cleanup**: Remove entire team environment in one command

```bash
# Test isolation between development environments
echo "🔒 Testing development environment isolation..."
docker exec team-a-web ping -c 1 team-b-web 2>&1 | grep -o "bad address.*" || echo "Isolation successful: no connectivity"
```

### Example 3: Blue-Green Deployment Network Strategy

**🎯 Learning Focus**: Zero-Downtime Deployment Patterns

```bash
# Create networks for blue-green deployment
echo "🔵🟢 Setting up blue-green deployment networks..."
docker network create production-network
docker network create blue-deployment
docker network create green-deployment
```

```bash
# Deploy blue environment
echo "🔵 Deploying BLUE environment..."
docker run -d --name blue-app --network blue-deployment nginx:alpine
docker network connect production-network blue-app
```

```bash
# Deploy green environment (new version)
echo "🟢 Deploying GREEN environment..."
docker run -d --name green-app --network green-deployment nginx:alpine
```

**📚 Blue-Green Process**:
1. **Blue active**: Currently serving production traffic
2. **Green staging**: New version deployed and tested
3. **Switch**: Connect green to production, disconnect blue
4. **Rollback ready**: Blue remains available for quick rollback

```bash
# Simulate traffic switch from blue to green
echo "🔄 Switching traffic from BLUE to GREEN..."
docker network disconnect production-network blue-app
docker network connect production-network green-app
echo "Traffic switch complete - GREEN is now active"
```

---

## Advanced Networking Concepts

### Container Network Namespaces

**🎯 Learning Focus**: Understanding Linux Network Namespaces

```bash
# Create a container and examine its network namespace
echo "🔍 Exploring container network namespaces..."
docker run -d --name namespace-demo alpine:latest sleep 300

# Get container process ID
CONTAINER_PID=$(docker inspect namespace-demo --format='{{.State.Pid}}')
echo "Container PID: $CONTAINER_PID"

# List network namespaces (requires root)
sudo ls -la /proc/$CONTAINER_PID/ns/
```

**Expected Output**:
```
total 0
dr-x--x--x 2 root root 0 Aug 17 10:45 .
dr-xr-xr-x 9 root root 0 Aug 17 10:45 ..
lrwxrwxrwx 1 root root 0 Aug 17 10:45 net -> 'net:[4026532123]'
lrwxrwxrwx 1 root root 0 Aug 17 10:45 pid -> 'pid:[4026532124]'
...
```

### Network Performance Optimization

**💡 Performance Tips**:

1. **Host networking**: For maximum throughput (sacrifice isolation)
2. **Custom MTU**: Optimize for your network infrastructure
3. **Multiple networks**: Separate data and control traffic
4. **Resource limits**: Prevent network resource exhaustion

```bash
# Create performance-optimized network
echo "⚡ Creating performance-optimized network..."
docker network create \
  --driver bridge \
  --opt com.docker.network.driver.mtu=9000 \
  --opt com.docker.network.bridge.name=perf-bridge \
  high-performance-net
```

---

## Key Concepts Summary

### 🎓 Core Networking Principles Mastered

**1. Custom Bridge Networks**
- ✅ **Better isolation** than default bridge network
- ✅ **Automatic DNS resolution** between containers using names
- ✅ **Custom IP addressing** schemes for integration with existing infrastructure
- ✅ **Network segmentation** for security and organization

**2. Network Inspection and Analysis**
- ✅ **docker network inspect** provides comprehensive configuration details
- ✅ **IP Address Management (IPAM)** shows subnet allocation and gateway configuration
- ✅ **Container connectivity mapping** shows which containers are connected where
- ✅ **Network driver capabilities** and limitations understanding

**3. Host Networking Applications**
- ✅ **Maximum performance** by eliminating network virtualization overhead
- ✅ **System monitoring** applications that need host-level network visibility
- ✅ **Legacy application** integration that expects direct host network access
- ✅ **Development scenarios** where network isolation isn't required

**4. Dynamic Network Management**
- ✅ **Live network operations** using connect/disconnect without container downtime
- ✅ **Troubleshooting techniques** for isolating and resolving connectivity issues
- ✅ **Maintenance procedures** for safely updating network configurations
- ✅ **Security incident response** through rapid network isolation

### 🏆 Production-Ready Skills Acquired

**Network Architects**:
- ✅ Multi-tier application network design
- ✅ Security boundary implementation through network segmentation  
- ✅ Performance optimization strategies
- ✅ Integration with existing network infrastructure

**DevOps Engineers**:
- ✅ Microservices communication patterns
- ✅ Blue-green deployment network strategies
- ✅ Development environment isolation
- ✅ Container network troubleshooting

**Security Engineers**:
- ✅ Network-based access controls
- ✅ Incident response and container isolation
- ✅ Network segmentation for defense in depth
- ✅ Monitoring and logging network communications

---

## Common Issues and Solutions

### Issue 1: Container Cannot Resolve Other Container Names
**Symptoms**: `ping: bad address 'container-name'`

**🔍 Diagnosis Steps**:
```bash
# Check if containers are on the same custom network
docker network inspect NETWORK_NAME --format='{{range .Containers}}{{.Name}} {{end}}'

# Verify container network membership
docker inspect CONTAINER_NAME --format='{{.NetworkSettings.Networks}}'
```

**🔧 Solution**: Ensure both containers are on the same custom network (not default bridge):
```bash
docker network connect CUSTOM_NETWORK CONTAINER_NAME
```

### Issue 2: Port Conflicts with Host Networking
**Symptoms**: `Port already in use` errors with `--network host`

**🔍 Diagnosis**:
```bash
# Check what's using the port on host
sudo netstat -tlnp | grep :PORT_NUMBER
sudo ss -tlnp | grep :PORT_NUMBER
```

**🔧 Solutions**:
- Use different ports for applications
- Stop conflicting services
- Use bridge networking instead of host networking

### Issue 3: Cannot Remove Network
**Symptoms**: `network NETWORK_NAME has active endpoints`

**🔍 Diagnosis**:
```bash
# Check which containers are still connected
docker network inspect NETWORK_NAME --format='{{range .Containers}}{{.Name}} {{end}}'
```

**🔧 Solution**: Remove or disconnect all containers first:
```bash
# Disconnect all containers
for container in $(docker network inspect NETWORK_NAME --format='{{range .Containers}}{{.Name}} {{end}}'); do
    docker network disconnect NETWORK_NAME $container
done

# Then remove the network
docker network rm NETWORK_NAME
```

### Issue 4: Slow Container Communication
**Symptoms**: High latency between containers

**🔍 Diagnosis**:
```bash
# Test latency between containers
docker exec CONTAINER1 ping -c 10 CONTAINER2

# Check network configuration
docker network inspect NETWORK_NAME --format='{{.Options}}'
```

**🔧 Solutions**:
- Consider host networking for high-performance applications
- Optimize MTU settings
- Use local volumes instead of network storage
- Monitor host network performance

---

## Troubleshooting Toolkit

### Essential Debugging Commands

```bash
# Network connectivity testing
docker exec CONTAINER ping -c 3 TARGET
docker exec CONTAINER telnet TARGET PORT
docker exec CONTAINER nc -zv TARGET PORT

# DNS resolution testing  
docker exec CONTAINER nslookup TARGET
docker exec CONTAINER dig TARGET

# Network interface analysis
docker exec CONTAINER ip addr show
docker exec CONTAINER ip route
docker exec CONTAINER netstat -rn

# Container network inspection
docker inspect CONTAINER --format='{{.NetworkSettings}}'
docker port CONTAINER

# Host-level network analysis
sudo iptables -t nat -L
sudo netstat -tlnp
sudo ss -tlnp

# Docker network system info
docker system info | grep -i network
docker network ls
docker network inspect NETWORK_NAME
```

---

## Next Steps and Further Learning

### 🎓 Advanced Topics to Explore

**1. Container Network Interface (CNI)**:
- Custom network plugins
- Integration with Kubernetes networking
- Advanced network policies

**2. Service Discovery and Load Balancing**:
- Docker Swarm networking
- External load balancer integration
- Service mesh technologies

**3. Security Hardening**:
- Network policies and firewalling
- Encrypted overlay networks
- Network monitoring and intrusion detection

**4. Performance Optimization**:
- SR-IOV and hardware acceleration
- Network function virtualization (NFV)
- High-performance computing networking

### 📚 Recommended Learning Path

**Immediate Next Steps**:
1. **Docker Swarm Networking**: Learn overlay networks across multiple hosts
2. **Kubernetes Networking**: Understand pods, services, and ingress
3. **Network Monitoring**: Implement logging and monitoring solutions

**Advanced Certification Preparation**:
- **Docker Certified Associate (DCA)**: This lab covers key networking topics
- **Certified Kubernetes Administrator (CKA)**: Network troubleshooting skills
- **Cloud networking certifications**: AWS, Azure, GCP container networking

---

## Conclusion

🎉 **Congratulations!** You have successfully mastered advanced Docker networking concepts and gained hands-on experience with:

✅ **Custom Network Creation**: Built sophisticated network topologies with precise IP management and security boundaries

✅ **Multi-Container Communication**: Implemented secure container-to-container communication with automatic service discovery

✅ **Network Inspection Mastery**: Developed expert-level troubleshooting skills using Docker's network inspection tools

✅ **Host Network Integration**: Understood when and how to use host networking for maximum performance and system integration

✅ **Dynamic Network Management**: Learned to modify container network connectivity without downtime for maintenance and troubleshooting

✅ **Production Architecture Patterns**: Implemented real-world networking patterns like microservices segmentation and blue-green deployments

### 🚀 Why These Skills Matter

**Immediate Impact**:
- **Production readiness**: You can now design and implement enterprise-grade container networking
- **Troubleshooting confidence**: Ability to diagnose and resolve complex network connectivity issues
- **Security awareness**: Understanding of network-based isolation and access controls

**Career Advancement**:
- **Microservices expertise**: Critical skills for modern application architectures
- **DevOps integration**: Network management fits seamlessly into CI/CD pipelines
- **Cloud-native preparation**: Foundation for Kubernetes, service mesh, and cloud container services
- **System architecture**: Capability to design scalable, secure container infrastructures

The networking skills you've developed form the backbone of modern containerized applications. Whether you're building microservices, implementing DevOps pipelines, or managing production container platforms, these networking concepts will be essential tools in your professional toolkit.

**Keep practicing, keep exploring, and remember**: Great network design is invisible to users but essential for application success!
