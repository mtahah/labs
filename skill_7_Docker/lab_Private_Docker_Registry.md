# Lab 18: Docker Registry - Creating a Private Docker Registry

## Objectives
By the end of this lab, students will be able to:
- Set up and configure a private Docker registry on a local machine
- Push and pull Docker images to/from a private registry
- Implement authentication and secure access using TLS certificates
- Configure Docker content trust and image signing policies
- Apply best practices for registry storage management and cleanup
- Understand the security implications of running a private Docker registry

## Prerequisites
**System Requirements:**
- OS: Ubuntu 20.04 LTS or later
- RAM: Minimum 4GB, Recommended 8GB
- CPU: 2+ cores
- Disk Space: 20GB available
- Network: Internet connectivity for initial setup

**Required Knowledge:**
- Basic understanding of Docker concepts (containers, images, Dockerfile)
- Familiarity with Linux command line operations
- Knowledge of basic networking concepts (ports, IP addresses)
- Understanding of SSL/TLS certificates (basic level)
- Experience with Docker CLI commands

**Installation Requirements:**
```bash
# Verify Docker installation
docker --version
# Output: Docker version 20.10.x or later

# Verify system resources
free -h
# Output: Shows available memory (need 4GB+)

df -h
# Output: Shows disk space (need 20GB+)
```

**Account Setup:** None required - all operations use local system

---

## Task 1: Setting Up a Private Docker Registry (15 minutes)

### Why: Private registries provide security, compliance, and performance benefits over public registries by keeping images within your infrastructure.

### Subtask 1.1: Verify Docker Installation and Start Registry Container

```bash
# Check Docker version and status
docker --version
docker info

# Output: Docker version 20.10.x, Server running (confirms Docker daemon active)

# Pull the official Docker registry image
docker pull registry:2

# Output: Status shows download progress, final "Status: Downloaded newer image"

# Create a directory for registry data persistence
sudo mkdir -p /opt/docker-registry/data
sudo mkdir -p /opt/docker-registry/certs
sudo mkdir -p /opt/docker-registry/auth

# Set proper permissions
sudo chown -R $USER:$USER /opt/docker-registry

# Verify directory structure
ls -la /opt/docker-registry/
# Output: Shows three directories (auth, certs, data) owned by current user
```

### Subtask 1.2: Run Basic Registry Container

```bash
# Run a simple registry on port 5000
docker run -d \
  --name registry-basic \
  --restart=always \
  -p 5000:5000 \
  -v /opt/docker-registry/data:/var/lib/registry \
  registry:2

# Output: Container ID (64-character hash) indicates successful start

# Verify the registry is running
docker ps | grep registry-basic
# Output: Shows container status as "Up", port mapping 5000:5000

# Check registry health
curl http://localhost:5000/v2/
# Output: {} (empty JSON response confirms registry API is accessible)
```

### Subtask 1.3: Test Basic Registry Functionality

```bash
# Pull a small test image
docker pull hello-world
# Output: Download progress, final "Status: Downloaded newer image"

# Tag the image for your private registry
docker tag hello-world localhost:5000/hello-world

# Push the image to your private registry
docker push localhost:5000/hello-world
# Output: Push progress with layers, final "latest: digest: sha256:..."

# Remove local images to test pull functionality
docker rmi hello-world localhost:5000/hello-world
# Output: "Untagged" messages for both images

# Pull the image from your private registry
docker pull localhost:5000/hello-world
# Output: Pull progress, confirms image retrieval from private registry

# Verify the image works
docker run localhost:5000/hello-world
# Output: "Hello from Docker!" message confirms successful operation
```

**Verification Point:**
```bash
# Confirm registry contains the image
curl http://localhost:5000/v2/_catalog
# Output: {"repositories":["hello-world"]} (shows successful image storage)
```

---

## Task 2: Implementing Authentication and Security (20 minutes)

### Why: Production registries require authentication and encryption to prevent unauthorized access and protect image integrity during transmission.

### Subtask 2.1: Generate Self-Signed SSL Certificates

```bash
# Navigate to the certs directory
cd /opt/docker-registry/certs

# Generate a private key
openssl genrsa -out domain.key 4096
# Output: Generating RSA private key, 4096 bit long modulus

# Generate a certificate signing request
openssl req -new -key domain.key -out domain.csr -subj "/C=US/ST=CA/L=San Francisco/O=MyOrg/CN=localhost"

# Generate the self-signed certificate
openssl x509 -req -days 365 -in domain.csr -signkey domain.key -out domain.crt
# Output: Signature ok, Getting Private key

# Verify certificate creation
ls -la /opt/docker-registry/certs/
# Output: Shows domain.crt, domain.csr, domain.key files with proper timestamps
```

### Subtask 2.2: Create Authentication Credentials

```bash
# Install htpasswd utility if not available
sudo apt-get update
sudo apt-get install -y apache2-utils

# Create authentication file with username 'testuser' and password 'testpass'
htpasswd -Bbn testuser testpass > /opt/docker-registry/auth/htpasswd

# Verify the auth file
cat /opt/docker-registry/auth/htpasswd
# Output: testuser:$2y$05$... (bcrypt hash confirms password encryption)
```

### Subtask 2.3: Stop Basic Registry and Start Secure Registry

```bash
# Stop the basic registry
docker stop registry-basic
docker rm registry-basic

# Start secure registry with authentication and TLS
docker run -d \
  --name registry-secure \
  --restart=always \
  -p 5000:5000 \
  -v /opt/docker-registry/data:/var/lib/registry \
  -v /opt/docker-registry/certs:/certs \
  -v /opt/docker-registry/auth:/auth \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_PRIVATE_KEY=/certs/domain.key \
  -e REGISTRY_AUTH=htpasswd \
  -e REGISTRY_AUTH_HTPASSWD_REALM="Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry:2

# Output: Container ID indicates successful secure registry start

# Verify the secure registry is running
docker ps | grep registry-secure
# Output: Shows container with multiple volume mounts, confirming security configuration
```

### Subtask 2.4: Configure Docker Client for Self-Signed Certificates

```bash
# Create Docker daemon configuration directory
sudo mkdir -p /etc/docker/certs.d/localhost:5000

# Copy the certificate to Docker's certificate directory
sudo cp /opt/docker-registry/certs/domain.crt /etc/docker/certs.d/localhost:5000/ca.crt

# Restart Docker daemon to pick up the new certificate
sudo systemctl restart docker

# Wait for Docker to restart
sleep 5

# Verify Docker is running
docker info
# Output: Server section shows Docker daemon active, confirms restart successful
```

**Common Failure Scenario:**
```bash
# Attempt to access registry without certificate configuration
curl https://localhost:5000/v2/
# Error: curl: (60) SSL certificate problem: self signed certificate
# This demonstrates why certificate configuration is essential
```

---

## Task 3: Testing Authenticated Registry Operations (15 minutes)

### Why: Authentication testing ensures only authorized users can push/pull images, critical for enterprise security policies.

### Subtask 3.1: Login to Private Registry

```bash
# Login to the private registry
docker login localhost:5000
# Prompt: Username: testuser
# Prompt: Password: testpass
# Output: Login Succeeded

# Verify login was successful by checking Docker config
cat ~/.docker/config.json
# Output: Shows "auths" section with localhost:5000 entry and encoded credentials
```

### Subtask 3.2: Push and Pull Images with Authentication

```bash
# Pull a sample application image
docker pull nginx:alpine
# Output: Download progress for nginx alpine image

# Tag it for your private registry
docker tag nginx:alpine localhost:5000/my-nginx:v1.0

# Push to your private registry
docker push localhost:5000/my-nginx:v1.0
# Output: Push layers, final digest confirms successful upload

# Create a custom image to push
nano Dockerfile
```
```dockerfile
FROM alpine:latest
RUN apk add --no-cache curl
COPY . /app
WORKDIR /app
CMD ["echo", "Hello from private registry!"]
```

```bash
# Build custom image
docker build -t localhost:5000/my-custom-app:latest .
# Output: Step-by-step build process, final "Successfully tagged"

# Push custom image
docker push localhost:5000/my-custom-app:latest
# Output: Push progress, digest confirmation

# List images in registry (using registry API)
curl -k -u testuser:testpass https://localhost:5000/v2/_catalog
# Output: {"repositories":["hello-world","my-custom-app","my-nginx"]}
```

### Subtask 3.3: Test Pull Operations

```bash
# Remove local images
docker rmi localhost:5000/my-nginx:v1.0 localhost:5000/my-custom-app:latest

# Pull images from private registry
docker pull localhost:5000/my-nginx:v1.0
docker pull localhost:5000/my-custom-app:latest
# Output: Pull progress confirms successful retrieval

# Test the pulled images
docker run --rm localhost:5000/my-custom-app:latest
# Output: "Hello from private registry!"

docker run --rm -p 8080:80 -d --name test-nginx localhost:5000/my-nginx:v1.0

# Test nginx is working
curl http://localhost:8080
# Output: nginx welcome page HTML content

# Clean up test container
docker stop test-nginx
```

**Verification Point:**
```bash
# Verify all operations completed successfully
docker images | grep localhost:5000
# Output: Shows multiple localhost:5000 tagged images with sizes and timestamps
```

---

## Task 4: Docker Content Trust and Image Signing (10 minutes)

### Why: Content trust provides cryptographic verification of image integrity and publisher identity, essential for supply chain security.

### Subtask 4.1: Understanding Content Trust

```bash
# Check current content trust setting
echo $DOCKER_CONTENT_TRUST
# Output: (empty) or 0, indicating content trust disabled

# Enable content trust
export DOCKER_CONTENT_TRUST=1

# Try to pull an unsigned image (this should fail)
docker pull localhost:5000/my-nginx:v1.0 || echo "Pull failed due to content trust"
# Error: No trust data for localhost:5000/my-nginx (demonstrates trust verification)

# Disable content trust for this session
export DOCKER_CONTENT_TRUST=0

# Now the pull should work
docker pull localhost:5000/my-nginx:v1.0
# Output: Successful pull without trust verification
```

### Subtask 4.2: Working with Content Trust Policies

```bash
# Create a script to demonstrate content trust bypass
nano test-content-trust.sh
```
```bash
#!/bin/bash

echo "Testing with content trust enabled..."
export DOCKER_CONTENT_TRUST=1
docker pull localhost:5000/my-nginx:v1.0 2>&1 | head -5

echo -e "\nTesting with --disable-content-trust flag..."
docker pull --disable-content-trust localhost:5000/my-nginx:v1.0 2>&1 | head -5

echo -e "\nTesting with content trust disabled..."
export DOCKER_CONTENT_TRUST=0
docker pull localhost:5000/my-nginx:v1.0 2>&1 | head -5
```

```bash
# Make script executable and run it
chmod +x test-content-trust.sh
./test-content-trust.sh
# Output: Shows different behaviors with content trust enabled/disabled
```

### Subtask 4.3: Registry Configuration for Content Trust

```bash
# Create a registry configuration file for content trust
nano /opt/docker-registry/config.yml
```
```yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
  tls:
    certificate: /certs/domain.crt
    key: /certs/domain.key
auth:
  htpasswd:
    realm: basic-realm
    path: /auth/htpasswd
notifications:
  endpoints:
    - name: local-webhook
      disabled: true
```

```bash
# Restart registry with custom configuration
docker stop registry-secure
docker rm registry-secure

docker run -d \
  --name registry-configured \
  --restart=always \
  -p 5000:5000 \
  -v /opt/docker-registry/data:/var/lib/registry \
  -v /opt/docker-registry/certs:/certs \
  -v /opt/docker-registry/auth:/auth \
  -v /opt/docker-registry/config.yml:/etc/docker/registry/config.yml \
  registry:2

# Output: New container ID confirms registry restart with custom configuration
```

---

## Task 5: Registry Storage Management and Best Practices (15 minutes)

### Why: Proper storage management prevents disk space issues and maintains registry performance in production environments.

### Subtask 5.1: Monitoring Registry Storage

```bash
# Check registry storage usage
du -sh /opt/docker-registry/data
# Output: Shows total size (e.g., "234M"), indicating current storage consumption

# List all repositories in the registry
curl -k -u testuser:testpass https://localhost:5000/v2/_catalog | jq '.'
# Output: JSON array of repository names with proper formatting

# Get tags for a specific repository
curl -k -u testuser:testpass https://localhost:5000/v2/my-nginx/tags/list | jq '.'
# Output: Shows available tags for the repository

# Get detailed information about registry contents
find /opt/docker-registry/data -type f -name "*.json" | head -10
# Output: Lists manifest files, showing registry internal structure
```

### Subtask 5.2: Registry Cleanup and Garbage Collection

```bash
# Create a cleanup script
nano registry-cleanup.sh
```
```bash
#!/bin/bash

echo "Registry cleanup starting..."

# Stop the registry
docker stop registry-configured

# Run garbage collection
docker run --rm \
  -v /opt/docker-registry/data:/var/lib/registry \
  registry:2 \
  garbage-collect /etc/docker/registry/config.yml

# Restart the registry
docker start registry-configured

echo "Registry cleanup completed."
```

```bash
# Make script executable
chmod +x registry-cleanup.sh

# Show storage before cleanup
echo "Storage before cleanup:"
du -sh /opt/docker-registry/data
# Output: Current storage size for baseline comparison

# Note: Script created and ready for production use
echo "Cleanup script created and ready to use."
```

### Subtask 5.3: Implementing Storage Limits and Policies

```bash
# Create an advanced registry configuration with storage policies
nano /opt/docker-registry/config-advanced.yml
```
```yaml
version: 0.1
log:
  level: info
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
    maxthreads: 100
  maintenance:
    uploadpurging:
      enabled: true
      age: 168h
      interval: 24h
      dryrun: false
    readonly:
      enabled: false
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
  tls:
    certificate: /certs/domain.crt
    key: /certs/domain.key
auth:
  htpasswd:
    realm: basic-realm
    path: /auth/htpasswd
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

```bash
# Create a monitoring script
nano monitor-registry.sh
```
```bash
#!/bin/bash

echo "=== Registry Monitoring Report ==="
echo "Date: $(date)"
echo

echo "Registry Container Status:"
docker ps | grep registry

echo -e "\nRegistry Storage Usage:"
du -sh /opt/docker-registry/data

echo -e "\nRegistry Repositories:"
curl -s -k -u testuser:testpass https://localhost:5000/v2/_catalog | jq -r '.repositories[]' 2>/dev/null || echo "No repositories found"

echo -e "\nRegistry Health Check:"
curl -s -k https://localhost:5000/v2/ > /dev/null && echo "Registry is healthy" || echo "Registry health check failed"

echo -e "\nDocker System Information:"
docker system df

echo "=== End of Report ==="
```

```bash
chmod +x monitor-registry.sh
./monitor-registry.sh
# Output: Comprehensive report showing registry status, storage, and health metrics
```

### Subtask 5.4: Backup and Recovery Procedures

```bash
# Create backup script
nano backup-registry.sh
```
```bash
#!/bin/bash

BACKUP_DIR="/opt/docker-registry/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="registry_backup_${TIMESTAMP}.tar.gz"

echo "Creating registry backup..."

# Create backup directory
mkdir -p $BACKUP_DIR

# Stop registry for consistent backup
docker stop registry-configured

# Create compressed backup
tar -czf "${BACKUP_DIR}/${BACKUP_FILE}" \
  -C /opt/docker-registry \
  data auth certs config.yml config-advanced.yml

# Restart registry
docker start registry-configured

echo "Backup created: ${BACKUP_DIR}/${BACKUP_FILE}"
echo "Backup size: $(du -sh ${BACKUP_DIR}/${BACKUP_FILE} | cut -f1)"
```

```bash
chmod +x backup-registry.sh

# Create restore script
nano restore-registry.sh
```
```bash
#!/bin/bash

if [ $# -eq 0 ]; then
    echo "Usage: $0 <backup_file>"
    echo "Available backups:"
    ls -la /opt/docker-registry/backups/
    exit 1
fi

BACKUP_FILE=$1

echo "Restoring registry from backup: $BACKUP_FILE"

# Stop registry
docker stop registry-configured

# Backup current state
mv /opt/docker-registry/data /opt/docker-registry/data.old.$(date +%Y%m%d_%H%M%S)

# Restore from backup
tar -xzf "$BACKUP_FILE" -C /opt/docker-registry

# Restart registry
docker start registry-configured

echo "Registry restored successfully"
```

```bash
chmod +x restore-registry.sh

# Run backup
./backup-registry.sh
# Output: Shows backup creation progress and final file size
```

---

## Task 6: Advanced Registry Features and Troubleshooting (10 minutes)

### Subtask 6.1: Registry API Exploration

```bash
# Create API testing script
nano test-registry-api.sh
```
```bash
#!/bin/bash

REGISTRY_URL="https://localhost:5000"
AUTH="testuser:testpass"

echo "=== Docker Registry API Testing ==="

echo -e "\n1. Check registry version:"
curl -s -k -u $AUTH $REGISTRY_URL/v2/ | jq '.' 2>/dev/null || echo "Registry API accessible"

echo -e "\n2. List all repositories:"
curl -s -k -u $AUTH $REGISTRY_URL/v2/_catalog | jq '.repositories[]' 2>/dev/null

echo -e "\n3. Get tags for my-nginx repository:"
curl -s -k -u $AUTH $REGISTRY_URL/v2/my-nginx/tags/list | jq '.' 2>/dev/null

echo -e "\n4. Get manifest for my-nginx:v1.0:"
curl -s -k -u $AUTH \
  -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
  $REGISTRY_URL/v2/my-nginx/manifests/v1.0 | jq '.schemaVersion' 2>/dev/null

echo -e "\n5. Registry statistics:"
echo "Total repositories: $(curl -s -k -u $AUTH $REGISTRY_URL/v2/_catalog | jq '.repositories | length' 2>/dev/null)"
```

```bash
chmod +x test-registry-api.sh
./test-registry-api.sh
# Output: Comprehensive API test results showing registry capabilities
```

### Subtask 6.2: Common Troubleshooting Scenarios

**Intentional Failure #1: Certificate Issues**
```bash
# Remove certificate to simulate common error
sudo mv /etc/docker/certs.d/localhost:5000/ca.crt /tmp/ca.crt.backup

# Attempt image push (will fail)
docker push localhost:5000/test-image:latest 2>&1 | head -3
# Error: x509: certificate signed by unknown authority

# Diagnostic command
openssl s_client -connect localhost:5000 -servername localhost < /dev/null 2>&1 | grep "Verify return code"
# Output: Verify return code: 18 (self signed certificate)

# Recovery procedure
sudo mv /tmp/ca.crt.backup /etc/docker/certs.d/localhost:5000/ca.crt
sudo systemctl restart docker
echo "Certificate restored - operations should work again"
```

**Intentional Failure #2: Authentication Problems**
```bash
# Corrupt authentication file
sudo echo "baduser:invalidhash" > /opt/docker-registry/auth/htpasswd

# Attempt login (will fail)
docker login localhost:5000 2>&1 | grep -i "error\|fail"
# Error: Login failed - authentication credentials rejected

# Recovery procedure
htpasswd -Bbn testuser testpass > /opt/docker-registry/auth/htpasswd
docker restart registry-configured
echo "Authentication restored"
```

```bash
# Create comprehensive troubleshooting guide script
nano troubleshoot-registry.sh
```
```bash
#!/bin/bash

echo "=== Docker Registry Troubleshooting Guide ==="

echo -e "\n1. Check if registry container is running:"
docker ps | grep registry || echo "No registry container found!"

echo -e "\n2. Check registry logs:"
echo "Recent registry logs:"
docker logs --tail 10 registry-configured 2>/dev/null || echo "Cannot access registry logs"

echo -e "\n3. Test registry connectivity:"
curl -k https://localhost:5000/v2/ 2>/dev/null && echo "Registry is accessible" || echo "Registry connection failed"

echo -e "\n4. Check certificate validity:"
openssl x509 -in /opt/docker-registry/certs/domain.crt -text -noout | grep "Not After" || echo "Cannot read certificate"

echo -e "\n5. Verify authentication file:"
[ -f /opt/docker-registry/auth/htpasswd ] && echo "Auth file exists" || echo "Auth file missing"

echo -e "\n6. Check disk space:"
df -h /opt/docker-registry/

echo -e "\n7. Test Docker daemon configuration:"
[ -f /etc/docker/certs.d/localhost:5000/ca.crt ] && echo "Docker certificate configured" || echo "Docker certificate missing"

echo -e "\n8. Network connectivity test:"
netstat -tlnp | grep :5000 || echo "Registry port not listening"

echo "=== End Troubleshooting ==="
```

```bash
chmod +x troubleshoot-registry.sh
./troubleshoot-registry.sh
# Output: Systematic diagnostic report identifying potential issues
```

### Subtask 6.3: Performance Optimization

```bash
# Create performance monitoring script
nano performance-monitor.sh
```
```bash
#!/bin/bash

echo "=== Registry Performance Monitoring ==="

echo -e "\n1. Container resource usage:"
docker stats --no-stream registry-configured 2>/dev/null || echo "Cannot get container stats"

echo -e "\n2. Registry response time test:"
time curl -s -k -u testuser:testpass https://localhost:5000/v2/_catalog > /dev/null

echo -e "\n3. Storage I/O statistics:"
iostat -x 1 1 2>/dev/null | tail -n +4 || echo "iostat not available"

echo -e "\n4. Memory usage:"
free -h

echo -e "\n5. Registry data directory size:"
du -sh /opt/docker-registry/data

echo "=== End Performance Report ==="
```

```bash
chmod +x performance-monitor.sh
./performance-monitor.sh
# Output: Performance metrics showing resource utilization and response times
```

---

## Verification and Testing (5 minutes)

### Final Verification Steps

```bash
# Create comprehensive test script
nano final-verification.sh
```
```bash
#!/bin/bash

echo "=== Final Registry Verification ==="

# Test 1: Registry accessibility
echo -e "\n✓ Testing registry accessibility..."
curl -k https://localhost:5000/v2/ > /dev/null 2>&1 && echo "PASS: Registry is accessible" || echo "FAIL: Registry not accessible"

# Test 2: Authentication
echo -e "\n✓ Testing authentication..."
curl -k -u testuser:testpass https://localhost:5000/v2/_catalog > /dev/null 2>&1 && echo "PASS: Authentication working" || echo "FAIL: Authentication failed"

# Test 3: Push/Pull functionality
echo -e "\n✓ Testing push/pull functionality..."
docker pull alpine:latest > /dev/null 2>&1
docker tag alpine:latest localhost:5000/test-alpine:latest > /dev/null 2>&1
docker push localhost:5000/test-alpine:latest > /dev/null 2>&1 && echo "PASS: Push working" || echo "FAIL: Push failed"

docker rmi localhost:5000/test-alpine:latest > /dev/null 2>&1
docker pull localhost:5000/test-alpine:latest > /dev/null 2>&1 && echo "PASS: Pull working" || echo "FAIL: Pull failed"

# Test 4: TLS/SSL
echo -e "\n✓ Testing TLS/SSL..."
openssl s_client -connect localhost:5000 -servername localhost < /dev/null 2>/dev/null | grep "Verify return code: 0" > /dev/null && echo "PASS: TLS working" || echo "INFO: Self-signed certificate in use"

# Test 5: Storage persistence
echo -e "\n✓ Testing storage persistence..."
[ -d /opt/docker-registry/data/docker ] && echo "PASS: Registry data persisted" || echo "FAIL: No registry data found"

echo -e "\n=== Verification Complete ==="
```

```bash
chmod +x final-verification.sh
./final-verification.sh
# Output: Comprehensive test results showing all registry functions working
```

**Challenge Section - Advanced Integration:**
```bash
# Multi-repository workflow test
docker pull redis:alpine
docker pull postgres:13-alpine
docker tag redis:alpine localhost:5000/production/redis:v6
docker tag postgres:13-alpine localhost:5000/production/postgres:v13
docker push localhost:5000/production/redis:v6
docker push localhost:5000/production/postgres:v13

# Verify multi-tier application setup
curl -k -u testuser:testpass https://localhost:5000/v2/_catalog
# Output: Should show production namespace with multiple services
```

---

## Cleanup (Optional)

```bash
# Stop and remove registry container
docker stop registry-configured
docker rm registry-configured

# Remove test images
docker rmi localhost:5000/my-nginx:v1.0 localhost:5000/my-custom-app:latest localhost:5000/test-alpine:latest 2>/dev/null

# Remove registry data (optional - only if you want to start fresh)
# sudo rm -rf /opt/docker-registry

echo "Lab cleanup completed"
```

---

## Troubleshooting Common Issues

### Issue 1: Certificate Errors
**Problem:** `x509: certificate signed by unknown authority`
**Solution:**
```bash
# Ensure certificate is in the correct location
sudo cp /opt/docker-registry/certs/domain.crt /etc/docker/certs.d/localhost:5000/ca.crt
sudo systemctl restart docker
# Why: Docker daemon needs certificate in specific location to trust registry
```

### Issue 2: Authentication Failures
**Problem:** `unauthorized: authentication required`
**Solution:**
```bash
# Re-login to the registry
docker logout localhost:5000
docker login localhost:5000
# Why: Credentials may have expired or been corrupted in Docker config
```

### Issue 3: Registry Not Starting
**Problem:** Registry container fails to start
**Solution:**
```bash
# Check logs for specific error
docker logs registry-configured
# Common fix: check file permissions
sudo chown -R $USER:$USER /opt/docker-registry
# Why: Registry process needs read/write access to mounted volumes
```

### Issue 4: Push/Pull Timeouts
**Problem:** Operations timeout or fail
**Solution:**
```bash
# Check registry health
curl -k https://localhost:5000/v2/
# Restart registry if needed
docker restart registry-configured
# Why: Network connectivity or resource constraints may cause timeouts
```

---

## Final Mastery Checklist

**Core Skills Achieved:**
- [ ] Private registry deployment and configuration
- [ ] SSL/TLS certificate implementation
- [ ] Authentication system setup and testing
- [ ] Image push/pull operations with security
- [ ] Content trust policy understanding
- [ ] Storage management and cleanup procedures
- [ ] API exploration and monitoring
- [ ] Troubleshooting and recovery procedures
- [ ] Performance optimization techniques
- [ ] Backup and restore procedures

**Real-World Applications Demonstrated:**
- Enterprise-grade registry security implementation
- Multi-environment image management (production namespace)
- Automated monitoring and maintenance workflows
- Disaster recovery planning and execution
- Performance monitoring and optimization

---

## Conclusion

**What You Accomplished:**
You have successfully implemented a production-ready private Docker registry with enterprise-grade security, monitoring, and management capabilities. This includes TLS encryption, authentication, content trust policies, and comprehensive operational procedures.

**Why This Matters:**
Private registries are essential for enterprise security, compliance, and performance. The skills demonstrated here directly apply to DevOps engineering, cloud architecture, and container orchestration in production environments.

**Next Steps:**
- Explore Docker Registry clustering for high availability
- Learn about registry replication and mirroring strategies
- Investigate integration with CI/CD tools like Jenkins, GitLab CI, or GitHub Actions
- Study advanced authentication methods including LDAP and OAuth integration
- Practice registry monitoring and alerting using tools like Prometheus and Grafana

This comprehensive lab provides the foundation for Docker registry management essential for container orchestration roles and Docker certification preparation.
