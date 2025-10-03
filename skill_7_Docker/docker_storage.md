# 🐋 Docker Storage Mastery Lab: Volumes & Bind Mounts

> 💡 **What You'll Master:** Build production-ready data persistence strategies using Docker volumes and bind mounts, including backup/restore operations and real-world troubleshooting scenarios
>
> ⏱️ **Total Time:** 60 minutes  
> 🎯 **Difficulty:** Beginner to Intermediate  
> 🔧 **Prerequisites:** Basic Linux command line knowledge, text editor familiarity  
> 💻 **Environment:** Ubuntu 20.04+ (64-bit), 2GB RAM minimum, 10GB free disk space

---

## 🎯 Learning Objectives

**By the end of this lab, you'll be able to:**

1. **Implement persistent storage** - Design and deploy Docker volumes that survive container lifecycle operations
2. **Configure bind mounts** - Connect host filesystem paths to containers with proper permissions and security
3. **Execute backup/restore workflows** - Protect critical data using production-grade volume management techniques

**Real-World Value:** Volumes prevent data loss when containers restart and enable data sharing between containers, critical for databases, logs, and stateful applications in production environments.

---

## 📊 Skills Progress Tracker

Track your progress through the lab:

| Skill Area | Progress | Confidence |
|------------|----------|------------|
| **Foundation Concepts** | ⚪⚪⚪⚪⚪ | Low → High |
| **Hands-On Implementation** | ⚪⚪⚪⚪⚪ | Low → High |
| **Troubleshooting** | ⚪⚪⚪⚪⚪ | Low → High |
| **Production Readiness** | ⚪⚪⚪⚪⚪ | Low → High |

---

# 🚀 Phase 1: Environment Setup (10 minutes)

## 🤔 Prediction Challenge #1 (60 seconds)

Before installing Docker, predict:
1. What compatibility issues might arise between your OS and Docker?
2. Which system dependencies are required for Docker to function?
3. How will you verify Docker is installed correctly (not just present)?

**Think deeply, then proceed.**

---

## System Requirements

Before installation, verify your system meets these requirements:

| Requirement | Minimum | Verification Command |
|-------------|---------|---------------------|
| **OS** | Ubuntu 20.04+ (64-bit) | `lsb_release -a` |
| **RAM** | 2GB | `free -h` |
| **Disk Space** | 10GB free | `df -h /` |
| **Architecture** | x86_64/amd64 | `uname -m` |

**Verify your system now:**

```bash
echo "=== System Verification ==="
echo "OS Version:" && lsb_release -a
echo -e "\nArchitecture:" && uname -m
echo -e "\nMemory:" && free -h | grep Mem
echo -e "\nDisk Space:" && df -h / | grep -v Filesystem
```

> ⚠️ **WARNING:** Docker requires 64-bit Ubuntu. 32-bit systems are not supported.

---

## Installation Process

We'll install Docker from the official repository to ensure we get the latest stable version.

### Step 1: Update Package Manager

**Why:** Ensures package lists contain the latest available versions, preventing installation failures.

```bash
echo "=== Updating Package Manager ==="
sudo apt update && sudo apt upgrade -y
```

**Expected output:** Package lists updated, system packages upgraded (may take 2-3 minutes).

---

### Step 2: Install Required Dependencies

**Why:** These packages enable secure package verification (GPG keys) and HTTPS repository access.

```bash
echo "=== Installing Docker Dependencies ==="
sudo apt install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg \
  lsb-release
```

**Package explanations:**
- `apt-transport-https`: Allows apt to retrieve packages over HTTPS
- `ca-certificates`: Common CA certificates for SSL verification
- `curl`: Downloads files from repositories
- `gnupg`: Verifies cryptographic signatures (GPG keys)
- `lsb-release`: Detects Ubuntu version for correct repository

**Expected output:** All packages installed successfully.

---

### Step 3: Add Docker's Official GPG Key

**Why:** Verifies packages are authentically from Docker, preventing malicious package installation.

```bash
echo "=== Adding Docker GPG Key ==="
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

**Command breakdown:**
- `curl -fsSL`: Downloads silently with error handling
- `gpg --dearmor`: Converts ASCII GPG key to binary format
- Output location: `/usr/share/keyrings/` (system keyring directory)

**Verify key was added:**

```bash
echo "=== Verifying GPG Key ==="
ls -lh /usr/share/keyrings/docker-archive-keyring.gpg
```

**Expected output:** File exists with size ~2-3KB.

---

### Step 4: Set Up Docker Repository

**Why:** Tells apt where to download Docker packages from the official stable channel.

```bash
echo "=== Configuring Docker Repository ==="
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**Command breakdown:**
- `arch=amd64`: Specifies 64-bit architecture
- `signed-by=`: Points to GPG key for verification
- `$(lsb_release -cs)`: Auto-detects Ubuntu codename (focal, jammy, etc.)
- `stable`: Uses stable release channel (not edge/test)

**Verify repository was added:**

```bash
echo "=== Verifying Repository Configuration ==="
cat /etc/apt/sources.list.d/docker.list
```

**Expected output:** Docker repository entry displayed.

---

### Step 5: Install Docker Engine

**Why:** Installs the core Docker daemon, CLI tools, and container runtime.

```bash
echo "=== Installing Docker Engine ==="
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

**Component explanations:**
- `docker-ce`: Docker Community Edition (engine)
- `docker-ce-cli`: Command-line interface tools
- `containerd.io`: Container runtime (runs containers)

**Expected output:** Docker packages installed successfully (may take 1-2 minutes).

---

### Step 6: Start and Enable Docker Service

**Why:** Starts Docker daemon immediately and configures automatic startup on boot.

```bash
echo "=== Starting Docker Service ==="
sudo systemctl start docker
sudo systemctl enable docker
```

**Verify service status:**

```bash
echo "=== Checking Docker Service Status ==="
sudo systemctl status docker --no-pager | head -20
```

**Expected output:** Active (running) status displayed.

---

### Step 7: Add User to Docker Group

**Why:** Allows running Docker commands without `sudo`, improving workflow efficiency.

```bash
echo "=== Adding User to Docker Group ==="
sudo usermod -aG docker $USER
```

**Apply group changes without logout:**

```bash
echo "=== Applying Group Changes ==="
newgrp docker
```

> ℹ️ **NOTE:** If `newgrp docker` causes issues in your script, log out and back in, or open a new terminal session.

---

### Step 8: Verify Docker Installation

**Critical verification step** - confirms Docker is fully functional, not just installed.

```bash
echo "=== Verifying Docker Version ==="
docker --version

echo -e "\n=== Testing Docker Functionality ==="
docker run --rm hello-world
```

**Expected output:**
- Docker version (e.g., Docker version 27.5.1+)
- "Hello from Docker!" message with explanation

**Additional verification:**

```bash
echo "=== Docker System Information ==="
docker info | grep -E "Server Version|Storage Driver|Operating System"
```

---

## Environment Test Results

Create a validation report showing all capabilities:

```bash
echo "=== Docker Environment Validation Report ===" > ~/docker-validation.txt
echo "Date: $(date)" >> ~/docker-validation.txt
echo "" >> ~/docker-validation.txt

echo "Docker Version:" >> ~/docker-validation.txt
docker --version >> ~/docker-validation.txt

echo "" >> ~/docker-validation.txt
echo "User Groups:" >> ~/docker-validation.txt
groups | grep docker >> ~/docker-validation.txt

echo "" >> ~/docker-validation.txt
echo "Docker Service Status:" >> ~/docker-validation.txt
systemctl is-active docker >> ~/docker-validation.txt

echo "" >> ~/docker-validation.txt
echo "Container Test:" >> ~/docker-validation.txt
docker run --rm alpine:latest echo "Container execution successful" >> ~/docker-validation.txt 2>&1

cat ~/docker-validation.txt
```

**Capability Summary:**

| Capability | Status | Verification |
|-----------|--------|--------------|
| Docker installed | ✅ | `docker --version` |
| Service running | ✅ | `systemctl status docker` |
| User permissions | ✅ | `docker run` without sudo |
| Container execution | ✅ | hello-world container ran |

---

## 🔧 Troubleshooting Common Setup Issues

<details>
<summary>❌ <strong>Error: Package 'docker-ce' has no installation candidate</strong></summary>

**Cause:** Docker repository not properly configured or Ubuntu version incompatible.

**Fix:**
```bash
# Verify repository configuration
cat /etc/apt/sources.list.d/docker.list

# If missing or incorrect, re-add repository
sudo rm /etc/apt/sources.list.d/docker.list
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```
</details>

<details>
<summary>❌ <strong>Error: Permission denied while trying to connect to Docker daemon</strong></summary>

**Cause:** User not in docker group, or group changes not applied.

**Fix:**
```bash
# Verify docker group exists
getent group docker

# Add user to docker group
sudo usermod -aG docker $USER

# Apply changes (choose one method)
# Method 1: New group session
newgrp docker

# Method 2: Log out and back in
# Method 3: Reboot system

# Verify membership
groups | grep docker
```
</details>

<details>
<summary>❌ <strong>Error: docker.service failed to start</strong></summary>

**Cause:** Conflicting installation, corrupted installation, or system issues.

**Fix:**
```bash
# Check service logs
sudo journalctl -u docker.service -n 50 --no-pager

# Remove conflicting packages
sudo apt remove docker docker-engine docker.io containerd runc

# Reinstall Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Start service
sudo systemctl start docker
sudo systemctl status docker
```
</details>

---

## ✅ Setup Completion Checklist

Verify all items before proceeding:

- [ ] Docker version displays correctly (`docker --version`)
- [ ] Docker service is active (`systemctl status docker`)
- [ ] User can run Docker without sudo (`docker run hello-world`)
- [ ] hello-world container executed successfully
- [ ] Validation report created (`~/docker-validation.txt`)

---

## 💡 Review Your Predictions

<details>
<summary><strong>Compare your predictions to reality</strong></summary>

**Prediction 1: Compatibility issues**
- Reality: Main issue is Ubuntu version (must be 20.04+, 64-bit)
- GPG key and repository configuration are common failure points

**Prediction 2: System dependencies**
- Reality: HTTPS transport, certificates, GPG tools, curl
- These enable secure package download and verification

**Prediction 3: Verification method**
- Reality: Multi-level verification (version, service status, container execution)
- Testing container execution is critical, not just checking installed packages
</details>

---

# 📦 Phase 2: Understanding Docker Storage (15 minutes)

## 🤔 Prediction Challenge #2 (90 seconds)

Before learning about Docker storage, predict:
1. What happens to data inside a container when you delete the container?
2. How would you share a file between your laptop and a running container?
3. What's the difference between "storing data in Docker" vs "on your computer"?
4. Why might you need two different storage methods for containers?

**Write down your predictions before continuing.**

---

## Core Concept: Container Ephemeral Nature

**Definition:** Containers are *ephemeral* - they're designed to be temporary and disposable. By default, any data written inside a container is lost when that container is removed.

**Why this matters:** Imagine running a database container. If every container restart deleted all your data, databases would be useless in Docker! This is why Docker provides persistent storage solutions.

**Mental Model:** Think of containers like a hotel room - you can rearrange furniture while you're there, but when you check out (delete the container), the room resets. Docker storage is like having a storage unit (external to the room) where you keep things permanently.

---

## Two Storage Approaches

Docker provides two primary methods for persistent data:

| Feature | Volumes | Bind Mounts |
|---------|---------|-------------|
| **Managed by** | Docker | Host OS |
| **Location** | Docker-controlled directory | Anywhere on host |
| **Creation** | `docker volume create` | Directory must exist first |
| **Best for** | Databases, app data | Config files, development |
| **Performance** | Optimized for containers | Depends on host filesystem |
| **Backup** | Docker-native commands | Standard host backup tools |
| **Security** | Isolated from host | Direct host filesystem access |
| **Portability** | Works on any Docker host | Path must exist on host |

**Key Decision:** Bind mounts are appropriate for sharing source code between development environments and containers, or persisting files onto the host's filesystem.

---

## Visual Flow: How Storage Works

```
┌─────────────────────────────────────────────────────────────┐
│                      CONTAINER                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Container Filesystem (Ephemeral)                     │   │
│  │  /app, /var/log, /etc - LOST ON DELETE              │   │
│  └──────────────────────────────────────────────────────┘   │
│                          ↓                                   │
│  ┌──────────────────┐           ┌─────────────────────┐    │
│  │  Mounted Volume  │           │ Mounted Bind Path   │    │
│  │  /data           │           │ /app/config         │    │
│  └────────┬─────────┘           └──────────┬──────────┘    │
└───────────┼─────────────────────────────────┼───────────────┘
            ↓                                 ↓
   ┌────────────────────┐           ┌──────────────────────┐
   │  Docker Volume     │           │  Host Directory      │
   │  (Managed by       │           │  /home/user/configs  │
   │   Docker)          │           │  (Direct path)       │
   │  PERSISTS          │           │  PERSISTS            │
   └────────────────────┘           └──────────────────────┘
```

---

## Setup Working Environment

Create organized directories for this lab:

**Context:** We'll organize our work into logical folders to keep volumes, bind mounts, and test data separate.

```bash
# Create lab directory structure
echo "=== Creating Lab Directory Structure ==="
mkdir -p ~/docker-lab/volumes        # For volume backup files
mkdir -p ~/docker-lab/bind-mounts    # For bind mount testing
mkdir -p ~/docker-lab/host-data      # For shared host data
cd ~/docker-lab

# Verify structure
echo -e "\n=== Verifying Directory Structure ==="
tree ~/docker-lab 2>/dev/null || ls -R ~/docker-lab
```

**Expected output:** Three directories created under `~/docker-lab/`.

---

## Examine Current Docker Storage

**Context:** Before creating new storage, see what Docker is currently managing.

```bash
# List existing volumes (probably none yet)
echo "=== Current Docker Volumes ==="
docker volume ls

# Show Docker disk usage
echo -e "\n=== Docker Disk Usage ==="
docker system df

# Detailed volume information
echo -e "\n=== Detailed Storage Information ==="
docker system df -v
```

**Expected output:**
- `docker volume ls`: Empty or system volumes only
- `docker system df`: Shows images, containers, volumes, and build cache usage

**What to observe:**
- Volume count (VOLUMES column)
- Disk space used by each category
- Reclaimable space (unused resources)

---

## 🧠 Active Recall Questions

Test your understanding before moving forward:

<details>
<summary><strong>Q1: Explain container ephemerality in your own words</strong></summary>

**Sample Answer:**
Containers are designed to be temporary and replaceable. Any data written to the container's filesystem (outside mounted storage) disappears when the container is deleted. This design allows containers to be scaled, updated, and replaced without worrying about state, but requires external storage for data that needs to persist.

**Why this matters:** Understanding ephemerality prevents data loss surprises and informs storage architecture decisions.
</details>

<details>
<summary><strong>Q2: When would you choose a volume over a bind mount?</strong></summary>

**Sample Answer:**
Choose volumes when:
- Running production databases (PostgreSQL, MySQL, MongoDB)
- You want Docker to manage the storage location
- You need to backup/restore using Docker commands
- Security is critical (volumes are isolated from host)
- Working across different host operating systems

Choose bind mounts when:
- Developing locally (live code reloading)
- Sharing configuration files from host
- You need direct access to files from host OS
- The exact file location matters

**Key insight:** Volumes are production-focused, bind mounts are development-friendly.
</details>

<details>
<summary><strong>Q3: What happens to volume data when you delete a container?</strong></summary>

**Sample Answer:**
Volume data persists even after deleting the container. Volumes are independent objects managed by Docker, separate from container lifecycle. The data remains until you explicitly delete the volume with `docker volume rm` or `docker volume prune`.

**Critical point:** This separation is what makes volumes useful for databases and stateful applications.
</details>

<details>
<summary><strong>Q4: Real-world scenario - Database container setup</strong></summary>

**Question:** You're deploying a PostgreSQL database container for production. Should you use a volume or bind mount for the database files? Explain your reasoning.

**Sample Answer:**
Use a volume for these reasons:

1. **Data safety:** Volumes are isolated from host filesystem changes
2. **Performance:** Volumes are optimized for container I/O
3. **Backup/restore:** Docker provides native volume backup commands
4. **Portability:** Works consistently across different Docker hosts
5. **Security:** Reduces attack surface by limiting host filesystem access

Anti-pattern to avoid: Using bind mount for production database would expose data to host filesystem issues, complicate backups, and create portability problems.
</details>

---

## 📝 Block Summary

**Concepts Mastered:**
- Container ephemeral nature and why it exists
- Difference between volumes and bind mounts
- When to use each storage type
- Docker storage architecture and management

**Critical Insights:**
1. Containers lose data by default - this is intentional design
2. Volumes are Docker-managed, bind mounts are host-managed
3. Choose storage type based on use case (production vs development)

**Progress Update:**
- Foundation Concepts: ⭐⭐⭐⚪⚪ (60% complete)

**Next Up:** Hands-on volume creation and data persistence testing

---

# 🔨 Phase 3: Working with Docker Volumes (15 minutes)

## 🤔 Prediction Challenge #3 (90 seconds)

Before creating volumes, predict:
1. Where on your computer will Docker store volume data?
2. Can two containers use the same volume simultaneously? What could go wrong?
3. If you create a file in a volume from container A, will container B see it?
4. What happens if you mount a volume to a directory that already has files?

**Consider these scenarios carefully before proceeding.**

---

## Block 3.1: Creating Named Volumes

### Core Concept: Named Volumes

**Definition:** Named volumes are Docker-managed storage locations with human-readable names, making them easy to reference and reuse across containers.

**Why named volumes matter:** Unlike anonymous volumes (auto-generated names), named volumes are easy to identify, back up, and manage in production environments.

---

### Hands-On: Create Your First Volume

**Context:** We'll create a volume named `my-data-volume` and inspect its properties to understand Docker's storage management.

**Expected behavior:** Docker creates a managed storage location in its data directory (`/var/lib/docker/volumes/` on Linux).

```bash
# Create a named volume
echo "=== Creating Named Volume ==="
docker volume create my-data-volume

# Verify volume creation
echo -e "\n=== Listing All Volumes ==="
docker volume ls

# Inspect volume details
echo -e "\n=== Inspecting Volume Details ==="
docker volume inspect my-data-volume
```

**Expected output:**
```
my-data-volume           # Volume created
DRIVER    VOLUME NAME
local     my-data-volume # Volume listed

[
    {
        "CreatedAt": "2025-10-03T...",
        "Driver": "local",
        "Mountpoint": "/var/lib/docker/volumes/my-data-volume/_data",
        "Name": "my-data-volume",
        ...
    }
]
```

**Parameter explanations:**
- `Driver: local`: Volume stored on local filesystem (other drivers exist for cloud storage)
- `Mountpoint`: Physical location where Docker stores this volume's data
- `Name`: Human-readable identifier for referencing the volume

**Key observation:** Notice the `Mountpoint` path - Docker manages this location, you don't interact with it directly.

---

### Use Volume in a Container

**Context:** Mount the volume to a container, write data, then verify the volume preserves data after container deletion.

**Expected behavior:** Files written to `/data` inside container persist in the volume after container removal.

```bash
# Start container with mounted volume
echo "=== Starting Container with Volume ==="
docker run -it --name volume-test-1 \
  -v my-data-volume:/data \
  ubuntu:20.04 bash
```

**Inside the container, execute:**

```bash
# Update package lists and install basic tools
apt update && apt install -y file

# Create test files with timestamps
echo "=== Creating Test Files ==="
echo "This is persistent data from container 1" > /data/test-file.txt
echo "Created at: $(date)" >> /data/test-file.txt
echo "Volume test data" > /data/volume-data.txt

# Verify files created
echo -e "\n=== Listing Volume Contents ==="
ls -lh /data/

# Display file contents
echo -e "\n=== File Contents ==="
cat /data/test-file.txt
cat /data/volume-data.txt

# Check file type
echo -e "\n=== File Type Information ==="
file /data/*

# Exit container
exit
```

**Expected output inside container:**
- Files created successfully in `/data/`
- `ls -lh` shows two files with timestamps
- `cat` displays file contents
- `file` confirms they're ASCII text files

**Command breakdown:**
- `-it`: Interactive terminal (allows bash session)
- `--name volume-test-1`: Names container for easy reference
- `-v my-data-volume:/data`: Mounts volume to `/data` inside container
- `ubuntu:20.04 bash`: Uses Ubuntu 20.04 image, runs bash shell

---

### Verify Data Persistence

**Context:** This is the critical test - remove the container and create a new one. If volumes work correctly, the data should still exist.

```bash
# Remove first container
echo "=== Removing First Container ==="
docker rm volume-test-1

# Create new container with same volume
echo -e "\n=== Creating New Container with Same Volume ==="
docker run -it --name volume-test-2 \
  -v my-data-volume:/data \
  ubuntu:20.04 bash
```

**Inside the new container:**

```bash
# Install file utility
apt update && apt install -y file

# Verify old data persists
echo "=== Verifying Persistent Data ==="
ls -lh /data/

echo -e "\n=== Reading Original Files ==="
cat /data/test-file.txt
cat /data/volume-data.txt

# Append new data
echo -e "\n=== Appending Data from Container 2 ==="
echo "Data from container 2 at: $(date)" >> /data/test-file.txt

# Show updated content
echo -e "\n=== Updated File Contents ==="
cat /data/test-file.txt

# Exit container
exit
```

**Expected result:** All original files exist, appended data appears, proving volume persisted across container lifecycle.

---

## Block 3.2: Multiple Volumes

### Create Additional Volumes

**Context:** Real applications often need separate volumes for different data types (logs, configs, databases). This separation improves organization and backup strategies.

```bash
# Create volumes for different purposes
echo "=== Creating Multiple Volumes ==="
docker volume create app-logs
docker volume create app-config
docker volume create database-data

# List all volumes
echo -e "\n=== All Volumes ==="
docker volume ls

# Inspect each volume
echo -e "\n=== Volume Details ==="
for vol in app-logs app-config database-data; do
  echo "--- $vol ---"
  docker volume inspect $vol | grep -E "Name|Mountpoint"
done
```

**Expected output:**
- Four volumes total (including my-data-volume)
- Each has unique Mountpoint location
- All use "local" driver

---

### Use Multiple Volumes in One Container

**Context:** Mounting multiple volumes simulates a real application with separated concerns (logs separate from configs separate from data).

**Expected behavior:** Each mounted path is independent, allowing logical data organization.

```bash
# Run container with three volumes
echo "=== Starting Multi-Volume Container ==="
docker run -it --name multi-volume-test \
  -v app-logs:/var/log/app \
  -v app-config:/etc/app \
  -v database-data:/var/lib/database \
  ubuntu:20.04 bash
```

**Inside container:**

```bash
# Install necessary tools
apt update && apt install -y tree

# Create data in each volume
echo "=== Creating Data in Multiple Volumes ==="

# Logs volume
mkdir -p /var/log/app
echo "Application started at $(date)" > /var/log/app/app.log
echo "INFO: System initialized" >> /var/log/app/app.log

# Config volume
mkdir -p /etc/app
cat > /etc/app/config.ini << 'EOF'
[database]
host=localhost
port=5432
environment=production

[logging]
level=INFO
output=/var/log/app/app.log
EOF

# Database volume
mkdir -p /var/lib/database
echo "database_version=1.0" > /var/lib/database/version.txt
echo "tables=users,orders,products" > /var/lib/database/schema.txt

# Verify all volumes
echo -e "\n=== Volume Contents ==="
echo "--- Logs ---"
ls -lh /var/log/app/
cat /var/log/app/app.log

echo -e "\n--- Config ---"
ls -lh /etc/app/
cat /etc/app/config.ini

echo -e "\n--- Database ---"
ls -lh /var/lib/database/
cat /var/lib/database/*

# Show directory structure
echo -e "\n=== Directory Tree ==="
tree /var/log/app /etc/app /var/lib/database

exit
```

**Expected output:**
- Three separate directories with distinct contents
- Each volume maintains independent data
- Tree structure shows clear separation

**Parameter explanation:**
- Multiple `-v` flags: Each creates a separate volume mount
- Volume naming: Descriptive names indicate purpose (app-logs, app-config, etc.)
- Mount paths: Follow Linux conventions (/var/log, /etc, /var/lib)

---

### Deeper Investigation: Volume Internals

**Context:** Understanding where Docker stores volume data helps with troubleshooting and backup strategies.

```bash
# Check volume physical locations (requires sudo)
echo "=== Volume Physical Locations ==="
for vol in my-data-volume app-logs app-config database-data; do
  echo "--- $vol ---"
  docker volume inspect $vol | grep Mountpoint
done

# Show disk usage per volume
echo -e "\n=== Volume Disk Usage ==="
docker system df -v | grep -A 10 "Local Volumes"

# Clean up test container
echo -e "\n=== Cleaning Up Test Container ==="
docker rm multi-volume-test
```

**Analysis questions:**

1. **Where are volumes stored?**
   - Answer: `/var/lib/docker/volumes/VOLUME_NAME/_data`
   - This is Docker's managed space, not user directories

2. **Why does Docker use a separate location?**
   - Isolation from host filesystem changes
   - Optimized for container I/O
   - Consistent location across hosts

3. **What happens if you delete a volume?**
   - Data is permanently removed
   - No recycle bin or recovery (unless you have backups)

---

## 🧠 Active Recall Questions

<details>
<summary><strong>Q1: Explain volume persistence in your own words</strong></summary>

**Sample Answer:**
Docker volumes are separate from containers - they're independent storage objects. When you mount a volume to a container, Docker connects that storage to the container's filesystem. Any data written to that mount point is stored in the volume, not the container. When you delete the container, the volume remains intact with all its data, ready to be mounted to another container.

**Key analogy:** The volume is like an external hard drive - you can plug it into different computers (containers), but the data stays on the drive.
</details>

<details>
<summary><strong>Q2: Why mount multiple volumes to one container?</strong></summary>

**Sample Answer:**
Mounting multiple volumes provides:

1. **Separation of concerns:** Logs separate from configs separate from database files
2. **Granular backups:** Backup database volume frequently, config volume rarely
3. **Different lifecycle management:** Preserve logs while resetting database
4. **Security:** Apply different permissions to each volume
5. **Organization:** Logical grouping makes troubleshooting easier

**Real-world example:** A PostgreSQL container might use:
- Volume 1: Database files (`/var/lib/postgresql/data`)
- Volume 2: Transaction logs (`/var/log/postgresql`)
- Volume 3: Configuration files (`/etc/postgresql`)

Each can be backed up on different schedules based on criticality.
</details>

<details>
<summary><strong>Q3: Connection to container lifecycle</strong></summary>

**Question:** How does volume behavior differ from container filesystem behavior during start/stop/delete operations?

**Sample Answer:**

| Operation | Container Filesystem | Volume |
|-----------|---------------------|--------|
| **Create** | New empty filesystem | Persists existing data |
| **Stop** | Freezes but data intact | No effect, data safe |
| **Start** | Resumes with existing data | Reconnects existing data |
| **Delete** | All data lost forever | Data remains intact |
| **Restart** | Data persists across restart | Data persists |

**Critical difference:** Container filesystem is ephemeral (tied to container lifecycle), volumes are persistent (independent lifecycle).

**Practical implication:** Always store important data in volumes, never rely on container filesystem for persistence.
</details>

<details>
<summary><strong>Q4: Production scenario - Volume strategy</strong></summary>

**Scenario:** You're deploying a web application with:
- Application code
- User uploaded images
- Application logs
- Database files
- Environment configuration

Which data should use volumes? Which should use other methods?

**Sample Answer:**

| Data Type | Storage Method | Reasoning |
|-----------|---------------|-----------|
| **Application code** | Container image | Code is immutable, baked into image |
| **User images** | Volume (user-uploads) | Must persist, user-generated |
| **Application logs** | Volume (app-logs) | Debugging, compliance, separate lifecycle |
| **Database files** | Volume (db-data) | Critical persistence requirement |
| **Environment config** | Bind mount or secrets | Sensitive, needs version control |

**Anti-patterns to avoid:**
- Storing code in volumes (defeats container immutability)
- Storing secrets in container images (security risk)
- Storing temporary cache in volumes (wastes disk space)
</details>

---

## 📝 Block Summary

**Concepts Mastered:**
- Creating and managing named volumes
- Mounting volumes to containers
- Verifying data persistence across container lifecycle
- Using multiple volumes for separation of concerns

**Critical Insights:**
1. Volumes survive container deletion - this is their primary purpose
2. Multiple volumes enable logical data organization
3. Volume names should indicate purpose for maintainability
4. Data written to volumes persists indefinitely until volume deletion

**Progress Update:**
- Foundation Concepts: ⭐⭐⭐⭐⚪ (80% complete)
- Hands-On Implementation: ⭐⭐⭐⚪⚪ (60% complete)

**Next Up:** Bind mounts for host-container file sharing

---

# 🔗 Phase 4: Working with Bind Mounts (12 minutes)

## 🤔 Prediction Challenge #4 (90 seconds)

Before working with bind mounts, predict:
1. If you edit a file on your host computer, will the container see the change immediately?
2. What permission issues might arise when container processes write to host directories?
3. Can you accidentally delete host files from inside a container with a bind mount?
4. How would bind mounts help during application development?

**Think through these carefully - bind mounts have more power and risk than volumes.**

---

## Block 4.1: Understanding Bind Mounts

### Core Concept: Direct Host Filesystem Access

**Definition:** Bind mounts create a direct connection between a host directory and a container path. Changes in either location are immediately visible in the other.

**Why bind mounts matter:** They enable real-time file synchronization between host and container, critical for development workflows where you edit code on your host and want containers to use those changes immediately.

**Mental model:** Volumes are like hotel safe deposit boxes (managed by hotel). Bind mounts are like pointing to your actual house - the container has direct access to your host files.

**Critical security consideration:** Bind mounts grant containers direct access to host filesystem. Misconfigured bind mounts can expose sensitive host data or allow containers to modify critical system files.

---

### Prepare Host Directory

**Context:** Before creating bind mounts, we need a host directory with test data to demonstrate bidirectional synchronization.

```bash
# Navigate to bind mount workspace
echo "=== Preparing Host Directory ==="
cd ~/docker-lab/host-data

# Create test files with timestamps
echo "This file exists on the host" > host-file.txt
echo "Created: $(date)" >> host-file.txt

echo "Shared configuration data" > config.conf
echo "environment=development" >> config.conf

# Create logs subdirectory
mkdir -p logs
echo "Initial log entry at $(date)" > logs/application.log

# Display created structure
echo -e "\n=== Host Directory Contents ==="
ls -lah ~/docker-lab/host-data/
tree ~/docker-lab/host-data/ 2>/dev/null || ls -R ~/docker-lab/host-data/
```

**Expected output:**
- `host-file.txt` and `config.conf` in main directory
- `logs/` subdirectory with `application.log`
- All files timestamped for tracking changes

---

## Block 4.2: Basic Bind Mount Usage

### Create Bind Mount Container

**Context:** Mount the host directory to `/app/data` inside the container. Changes made in either location will be immediately visible in the other.

**Expected behavior:** Files created on host appear in container, files created in container appear on host.

```bash
# Start container with bind mount
echo "=== Starting Container with Bind Mount ==="
docker run -it --name bind-mount-test \
  -v ~/docker-lab/host-data:/app/data \
  ubuntu:20.04 bash
```

**Inside container:**

```bash
# Install tools
apt update && apt install -y tree file

# Examine mounted data
echo "=== Examining Bind Mount ==="
ls -lah /app/data/
tree /app/data/

# Read host-created files
echo -e "\n=== Reading Host Files ==="
cat /app/data/host-file.txt
cat /app/data/config.conf
cat /app/data/logs/application.log

# Modify existing file
echo -e "\n=== Modifying Host File from Container ==="
echo "Modified from container at $(date)" >> /app/data/host-file.txt

# Create new file
echo -e "\n=== Creating New File from Container ==="
echo "Container created this file at $(date)" > /app/data/container-file.txt

# Append to log
echo "New log entry from container at $(date)" >> /app/data/logs/application.log

# Verify changes
echo -e "\n=== Verifying Changes ==="
ls -lah /app/data/
cat /app/data/host-file.txt

exit
```

**Expected output:**
- Container can read all host files
- Modifications appear in both locations
- New files created by container

**Command breakdown:**
- `-v ~/docker-lab/host-data:/app/data`: Binds host path to container path
- Path before `:` is host location (must be absolute path)
- Path after `:` is container mount point
- Unlike volumes, bind mount directory must exist before running container

---

### Verify Bidirectional Synchronization

**Context:** Confirm changes made in container are visible on host filesystem immediately.

```bash
# Check host directory from host terminal
echo "=== Verifying Host Directory Changes ==="
cd ~/docker-lab/host-data
ls -lah

# Show modified file
echo -e "\n=== Modified Host File ==="
cat host-file.txt

# Show container-created file
echo -e "\n=== Container-Created File ==="
cat container-file.txt

# Show updated log
echo -e "\n=== Updated Log ==="
cat logs/application.log

# Clean up container
docker rm bind-mount-test
```

**Expected output:**
- All container modifications visible on host
- `container-file.txt` exists on host filesystem
- Log file shows both host and container entries

**Key observation:** No data copying occurred - container and host share the exact same files in real-time.

---

## Block 4.3: Real-Time Synchronization Demo

### Multi-Terminal Demonstration

**Context:** This exercise demonstrates the most powerful aspect of bind mounts - real-time bidirectional synchronization. This is why bind mounts are invaluable for development.

**Setup instructions:** You'll need two terminal sessions. If you don't have split terminal support, open two separate terminal windows.

#### Terminal 1: Monitor Host Directory

```bash
# Monitor host directory for changes (updates every 2 seconds)
echo "=== Starting File Monitor ==="
cd ~/docker-lab/host-data

# Create monitoring script
cat > monitor.sh << 'EOF'
#!/bin/bash
while true; do
  clear
  echo "=== File Monitor - Press Ctrl+C to stop ==="
  echo "Time: $(date)"
  echo ""
  echo "--- Directory Contents ---"
  ls -lah
  echo ""
  echo "--- host-file.txt Contents ---"
  cat host-file.txt 2>/dev/null || echo "File not found"
  sleep 2
done
EOF

chmod +x monitor.sh
./monitor.sh
```

**Expected behavior:** Screen refreshes every 2 seconds showing directory contents and `host-file.txt` content.

#### Terminal 2: Make Changes from Container

```bash
# Start container with bind mount
echo "=== Starting Container for Real-Time Testing ==="
docker run -it --name realtime-test \
  -v ~/docker-lab/host-data:/app/data \
  ubuntu:20.04 bash
```

**Inside container:**

```bash
# Install required tools
apt update && apt install -y curl

# Add entries with delays to observe real-time sync
echo "=== Adding Timed Entries ==="
for i in {1..5}; do
  echo "Entry $i from container at $(date)" >> /app/data/host-file.txt
  echo "Added entry $i - check Terminal 1"
  sleep 3
done

# Create new files
echo -e "\n=== Creating Multiple Files ==="
for i in {1..3}; do
  echo "Test file $i created at $(date)" > /app/data/testfile-$i.txt
  echo "Created testfile-$i.txt - check Terminal 1"
  sleep 2
done

exit
```

**Expected observation in Terminal 1:**
- New entries appear in `host-file.txt` as they're written
- New files appear in directory listing immediately
- No delay between container write and host visibility

**Stop monitor:** Return to Terminal 1 and press `Ctrl+C` to stop monitoring script.

```bash
# Clean up
echo "=== Cleaning Up Realtime Test ==="
docker rm realtime-test
rm ~/docker-lab/host-data/monitor.sh
```

---

## Block 4.4: Read-Only Bind Mounts

### Security with Read-Only Mounts

**Context:** Read-only bind mounts prevent containers from modifying host files, crucial for sharing configuration files or source code that shouldn't be altered by containers.

**Expected behavior:** Container can read files but cannot write, create, or delete files in the mounted directory.

```bash
# Create read-only bind mount
echo "=== Starting Container with Read-Only Mount ==="
docker run -it --name readonly-test \
  -v ~/docker-lab/host-data:/app/data:ro \
  ubuntu:20.04 bash
```

**Inside container:**

```bash
# Install basic tools
apt update && apt install -y file

# Verify read access works
echo "=== Testing Read Access ==="
ls -lah /app/data/
cat /app/data/host-file.txt
cat /app/data/config.conf

# Attempt to modify existing file (SHOULD FAIL)
echo -e "\n=== Attempting to Modify File (Should Fail) ==="
echo "This should fail" >> /app/data/host-file.txt
# Expected error: cannot create temp file: Read-only file system

# Attempt to create new file (SHOULD FAIL)
echo -e "\n=== Attempting to Create File (Should Fail) ==="
touch /app/data/new-file.txt
# Expected error: cannot touch: Read-only file system

# Attempt to delete file (SHOULD FAIL)
echo -e "\n=== Attempting to Delete File (Should Fail) ==="
rm /app/data/config.conf
# Expected error: cannot remove: Read-only file system

# Verify mount is read-only
echo -e "\n=== Verifying Read-Only Mount ==="
mount | grep "/app/data"

exit
```

**Expected output:**
- Read operations succeed (ls, cat)
- All write operations fail with "Read-only file system" errors
- `mount` command shows `ro` (read-only) flag

**When to use read-only mounts:**
- Configuration files that containers should read but never modify
- Source code directories in production (code in image, configs mounted)
- Shared reference data across multiple containers
- Security-sensitive directories where write access is unnecessary

```bash
# Clean up
echo "=== Cleaning Up Read-Only Test ==="
docker rm readonly-test
```

---

## Deeper Investigation: Permissions and Ownership

**Context:** Bind mounts can cause permission conflicts because container processes may run as different users than host users.

```bash
# Check current host file ownership
echo "=== Host File Ownership ==="
ls -lhn ~/docker-lab/host-data/

# Create test container to examine permissions
echo -e "\n=== Testing Permission Behavior ==="
docker run --rm \
  -v ~/docker-lab/host-data:/app/data \
  ubuntu:20.04 bash -c '
    echo "=== Container User Info ==="
    id
    echo ""
    echo "=== Container View of Mounted Files ==="
    ls -lhn /app/data/
    echo ""
    echo "=== Creating File as Root ==="
    echo "Created by root in container" > /app/data/root-created.txt
    ls -lhn /app/data/root-created.txt
  '

# Check ownership on host
echo -e "\n=== Host View After Container Creation ==="
ls -lhn ~/docker-lab/host-data/root-created.txt
```

**Analysis questions:**

1. **What user ID (UID) did the container use?**
   - Answer: 0 (root) - containers run as root by default
   
2. **Who owns the file created by the container?**
   - Answer: root (UID 0) on the host filesystem
   
3. **Why might this cause problems?**
   - Non-root users on host may not be able to modify container-created files
   - Security concerns with root-owned files in user directories

**Best practice solution:** Run containers with user mapping or as non-root user:

```bash
# Run container as current user (avoids permission issues)
echo "=== Running Container as Current User ==="
docker run --rm \
  --user $(id -u):$(id -g) \
  -v ~/docker-lab/host-data:/app/data \
  ubuntu:20.04 bash -c '
    echo "=== Current Container User ==="
    id
    echo ""
    echo "=== Creating File as Current User ==="
    echo "Created by non-root user" > /app/data/user-created.txt
    ls -lhn /app/data/user-created.txt
  '

# Verify ownership on host
echo -e "\n=== Host View of User-Created File ==="
ls -lh ~/docker-lab/host-data/user-created.txt
```

**Expected result:** File owned by your host user, no permission conflicts.

---

## 🧠 Active Recall Questions

<details>
<summary><strong>Q1: Explain bind mount synchronization in your own words</strong></summary>

**Sample Answer:**
Bind mounts create a direct link between a host directory and a container directory - they literally point to the same physical storage location. When you modify a file in the container, you're modifying the actual host file, not a copy. The synchronization is instant because there is no copying or syncing happening - both paths reference the same underlying filesystem data.

**Analogy:** It's like creating a shortcut or symbolic link - multiple paths pointing to the same file, not multiple copies of the file.
</details>

<details>
<summary><strong>Q2: When would you use read-only bind mounts?</strong></summary>

**Sample Answer:**
Use read-only bind mounts when:

1. **Configuration files:** Container needs to read config but shouldn't modify it
2. **Security:** Limiting container's ability to affect host filesystem
3. **Source code in production:** Code baked into image, configs mounted read-only
4. **Shared reference data:** Multiple containers reading same data, none should modify
5. **Compliance:** Ensuring containers can't tamper with audit logs

**Real example:** Web application container with:
- Application code: Built into image (immutable)
- TLS certificates: Bind mounted read-only from `/etc/ssl/certs`
- Application config: Bind mounted read-only from `/etc/myapp/config.yml`
- Data storage: Volume (read-write) for database files

This ensures containers can't accidentally corrupt certificates or configurations.
</details>

<details>
<summary><strong>Q3: Compare volumes vs bind mounts</strong></summary>

**Comparison Table:**

| Aspect | Volumes | Bind Mounts |
|--------|---------|-------------|
| **Path** | Docker-managed | User-specified host path |
| **Creation** | Docker creates location | Directory must pre-exist |
| **Portability** | Works on any Docker host | Host-specific paths |
| **Performance** | Optimized for containers | Depends on host FS |
| **Security** | Isolated from host | Direct host access |
| **Use Case** | Production data | Development/configs |
| **Backup** | `docker volume` commands | Standard host backup |
| **Modification** | Container-safe | Permission issues possible |

**Decision tree:**
- Need data to survive container deletion? → Volume or bind mount
- Development with live code reload? → Bind mount
- Production database? → Volume
- Sharing logs with host monitoring? → Bind mount
- Multi-container shared data? → Volume
</details>

<details>
<summary><strong>Q4: Troubleshooting scenario</strong></summary>

**Scenario:** You created a bind mount from `~/app-data` to `/data` in your container. The container is running, but you don't see any files in `/data` inside the container. What could be wrong?

**Diagnostic steps:**

1. **Check if directory exists on host:**
   ```bash
   ls -la ~/app-data
   # If doesn't exist, create it first
   ```

2. **Verify bind mount syntax:**
   ```bash
   docker inspect container-name | grep -A 10 Mounts
   # Look for correct source and destination paths
   ```

3. **Check if using relative path (wrong):**
   ```bash
   # ❌ Wrong: -v app-data:/data (relative path)
   # ✅ Right: -v ~/app-data:/data or -v /home/user/app-data:/data
   ```

4. **Verify container process isn't changing directory:**
   ```bash
   docker exec container-name pwd
   docker exec container-name ls -la /data
   ```

**Common mistakes:**
- Using relative paths (Docker requires absolute paths for bind mounts)
- Typo in host path (Docker creates empty directory if path doesn't exist)
- Wrong mount point inside container
- Container process running in different directory

**Solution:** Always use absolute paths with bind mounts, verify path exists before mounting.
</details>

---

## 📝 Block Summary

**Concepts Mastered:**
- Creating bind mounts with host directories
- Real-time bidirectional synchronization
- Read-only mounts for security
- Permission and ownership considerations

**Critical Insights:**
1. Bind mounts provide instant synchronization - no copying involved
2. Read-only mounts prevent container modifications to host files
3. Permission conflicts arise when container runs as root
4. Bind mounts are powerful but require careful security consideration

**Progress Update:**
- Foundation Concepts: ⭐⭐⭐⭐⭐ (100% complete)
- Hands-On Implementation: ⭐⭐⭐⭐⚪ (80% complete)
- Troubleshooting: ⭐⭐⚪⚪⚪ (40% complete)

**Next Up:** Advanced operations including backup, restore, and lifecycle testing

---

# 🔄 Phase 5: Advanced Volume Operations (10 minutes)

## 🤔 Prediction Challenge #5 (90 seconds)

Before learning backup/restore, predict:
1. How would you backup a Docker volume without stopping the container?
2. What format would be best for volume backups (single file, directory, compressed)?
3. If you restore a volume backup to a new volume, will it have the same name?
4. What could go wrong during a volume restore operation?

**Consider data integrity and portability.**

---

## Block 5.1: Volume Backup

### Core Concept: Volume Backup Strategy

**Definition:** Volume backup involves creating portable copies of volume data that can be restored to new volumes or transferred to different Docker hosts.

**Why backups matter:** Volumes contain critical application data (databases, user uploads, logs). Without backups, hardware failure or accidental deletion causes permanent data loss.

**Standard backup approach:** Use a temporary container to access the volume filesystem, create a compressed archive, and save it to a host directory.

---

### Create Backup of Volume

**Context:** We'll backup the `my-data-volume` we created earlier. The backup will be a compressed tar archive stored on the host.

**Expected behavior:** Temporary container mounts the volume, compresses its contents, saves to host directory, then removes itself.

```bash
# Ensure volume has data
echo "=== Verifying Volume Has Data ==="
docker run --rm -v my-data-volume:/data ubuntu:20.04 ls -lah /data

# Create backup
echo -e "\n=== Creating Volume Backup ==="
docker run --rm \
  -v my-data-volume:/source:ro \
  -v ~/docker-lab/volumes:/backup \
  ubuntu:20.04 \
  tar czf /backup/my-data-backup-$(date +%Y%m%d-%H%M%S).tar.gz -C /source .

# Verify backup was created
echo -e "\n=== Verifying Backup File ==="
ls -lh ~/docker-lab/volumes/
```

**Expected output:**
- Backup file created with timestamp (e.g., `my-data-backup-20251003-143022.tar.gz`)
- File size depends on volume data (typically a few KB for our test data)

**Command breakdown:**
- `--rm`: Automatically remove container after backup completes
- `-v my-data-volume:/source:ro`: Mount volume as read-only (safe during backup)
- `-v ~/docker-lab/volumes:/backup`: Mount host directory for saving backup
- `tar czf`: Create compressed (gzip) tar file
- `-C /source`: Change to source directory before archiving
- `.`: Archive all contents of current directory

**Why this approach works:**
- Temporary container can access volume filesystem
- Read-only mount prevents accidental modifications
- Compressed format reduces storage requirements
- Host-mounted backup directory makes file accessible outside Docker

---

### Verify Backup Contents

**Context:** Before relying on a backup, verify it contains expected data and is not corrupted.

```bash
# List backup contents without extracting
echo "=== Inspecting Backup Contents ==="
tar tzf ~/docker-lab/volumes/my-data-backup-*.tar.gz

# Show detailed listing
echo -e "\n=== Detailed Backup Contents ==="
tar tzvf ~/docker-lab/volumes/my-data-backup-*.tar.gz
```

**Expected output:**
- List of files in backup (test-file.txt, volume-data.txt, etc.)
- File sizes and timestamps preserved

---

## Block 5.2: Volume Restore

### Restore to New Volume

**Context:** Demonstrate disaster recovery by restoring backup to a completely new volume with a different name.

**Expected behavior:** New volume created, backup extracted into it, original data fully restored.

```bash
# Create new empty volume
echo "=== Creating New Volume for Restore ==="
docker volume create restored-volume

# Restore backup to new volume
echo -e "\n=== Restoring Backup ==="
docker run --rm \
  -v restored-volume:/target \
  -v ~/docker-lab/volumes:/backup \
  ubuntu:20.04 \
  tar xzf /backup/my-data-backup-*.tar.gz -C /target

# Verify restored data
echo -e "\n=== Verifying Restored Data ==="
docker run --rm -v restored-volume:/data ubuntu:20.04 ls -lah /data

echo -e "\n=== Checking File Contents ==="
docker run --rm -v restored-volume:/data ubuntu:20.04 cat /data/test-file.txt
docker run --rm -v restored-volume:/data ubuntu:20.04 cat /data/volume-data.txt
```

**Expected output:**
- All original files present in restored volume
- File contents match original data
- Timestamps preserved from backup

**Command breakdown:**
- `-v restored-volume:/target`: Mount new volume as destination
- `tar xzf`: Extract compressed tar file
- `-C /target`: Extract into target directory

**Use cases for restore:**
- Disaster recovery after data loss
- Cloning production data to development environment
- Migrating volumes to new Docker host
- Creating test environments with production-like data

---

## Block 5.3: Volume Inspection and Management

### Detailed Volume Information

**Context:** Production environments require monitoring volume usage and health. These commands provide operational visibility.

```bash
# Inspect all volumes
echo "=== Inspecting All Volumes ==="
for vol in my-data-volume restored-volume app-logs app-config database-data; do
  echo "--- $vol ---"
  docker volume inspect $vol | jq '.[0] | {Name, Driver, Mountpoint, CreatedAt}'
done
```

> ℹ️ **NOTE:** If `jq` is not installed, the command will still show full JSON output.

**Install jq if needed:**

```bash
# Install jq for JSON parsing (optional but recommended)
sudo apt install -y jq
```

**Key metrics to monitor:**
- `Mountpoint`: Physical location on disk
- `CreatedAt`: When volume was created
- `Driver`: Storage driver type (local, nfs, etc.)
- `Labels`: Metadata tags for organization

---

### Volume Usage Analysis

**Context:** Monitor disk space consumption to prevent storage exhaustion.

```bash
# Show Docker disk usage summary
echo "=== Docker Disk Usage Summary ==="
docker system df

# Detailed volume usage
echo -e "\n=== Detailed Volume Usage ==="
docker system df -v | grep -A 20 "Local Volumes"

# Calculate total volume disk usage
echo -e "\n=== Total Volume Disk Usage ==="
docker system df | grep Volumes
```

**Expected output:**
- TYPE, TOTAL, ACTIVE, SIZE, RECLAIMABLE columns
- Identifies volumes consuming most space
- Shows unused (reclaimable) volume space

**When to check volume usage:**
- Before deploying new applications
- During routine maintenance windows
- When disk space alerts trigger
- Before backup operations

---

## Block 5.4: Lifecycle Testing

### Container Restart Persistence

**Context:** Verify data survives container stop/start cycles (simulates crashes and restarts).

```bash
# Create container with volume and background process
echo "=== Starting Container with Persistent Process ==="
docker run -d --name persistence-test \
  -v database-data:/var/lib/data \
  ubuntu:20.04 \
  bash -c 'while true; do echo "$(date): Service running" >> /var/lib/data/db.log; sleep 3; done'

# Let it run and generate data
echo "=== Generating Data (15 seconds) ==="
sleep 15

# Check initial data
echo -e "\n=== Initial Data ==="
docker exec persistence-test cat /var/lib/data/db.log | head -10
echo "..."
docker exec persistence-test cat /var/lib/data/db.log | tail -3
```

**Expected output:** Multiple log entries with timestamps showing continuous operation.

---

### Stop and Restart Test

**Context:** Containers stop for many reasons (updates, crashes, host reboots). Data must persist.

```bash
# Stop container
echo -e "\n=== Stopping Container ==="
docker stop persistence-test

# Restart same container
echo "=== Restarting Container ==="
docker start persistence-test

# Wait for new entries
sleep 10

# Verify old data persists and new data appends
echo -e "\n=== Data After Restart ==="
docker exec persistence-test bash -c 'echo "=== Total log entries:"; wc -l /var/lib/data/db.log; echo ""; echo "=== First 5 entries:"; head -5 /var/lib/data/db.log; echo ""; echo "=== Last 5 entries:"; tail -5 /var/lib/data/db.log'
```

**Expected output:**
- All previous entries still present
- New entries added after restart
- No gap or data loss during stop/start

**What this proves:** Container restarts don't affect volume data.

---

### Complete Container Replacement

**Context:** This is the ultimate persistence test - delete the container entirely and create a new one.

**Expected behavior:** New container sees all data from previous container because they use the same volume.

```bash
# Stop and remove container
echo -e "\n=== Removing Container Completely ==="
docker stop persistence-test
docker rm persistence-test

# Create entirely new container with same volume
echo "=== Creating New Container with Same Volume ==="
docker run -d --name new-persistence-test \
  -v database-data:/var/lib/data \
  ubuntu:20.04 \
  bash -c 'echo "$(date): New container started" >> /var/lib/data/db.log; while true; do echo "$(date): New container running" >> /var/lib/data/db.log; sleep 3; done'

# Wait for new entries
sleep 10

# Verify complete history persists
echo -e "\n=== Complete Data History ==="
docker exec new-persistence-test bash -c 'echo "=== Total entries (old + new):"; wc -l /var/lib/data/db.log; echo ""; echo "=== First entries (from old container):"; head -3 /var/lib/data/db.log; echo ""; echo "=== New container marker:"; grep "New container started" /var/lib/data/db.log; echo ""; echo "=== Latest entries (from new container):"; tail -3 /var/lib/data/db.log'
```

**Expected output:**
- All original log entries from first container
- "New container started" marker
- Continuous log entries from new container
- No data loss despite complete container replacement

**What this proves:**
- Volumes are completely independent of containers
- Container lifecycle doesn't affect volume data
- Same volume can be reused across multiple containers
- This is how database containers maintain data across updates

```bash
# Clean up
echo -e "\n=== Cleaning Up Lifecycle Test ==="
docker stop new-persistence-test
docker rm new-persistence-test
```

---

## 🧠 Active Recall Questions

<details>
<summary><strong>Q1: Explain the backup strategy in your own words</strong></summary>

**Sample Answer:**
The Docker volume backup strategy uses a temporary container as a "bridge" between the volume and the host filesystem. The container mounts the volume (read-only for safety) and a host directory simultaneously. It then uses `tar` to compress the volume contents and writes the archive to the host-mounted directory. The container is automatically removed after backup completes.

**Why this approach:**
- Volumes are managed by Docker, not directly accessible on host
- Temporary container provides filesystem access to volume
- Read-only mount prevents accidental corruption during backup
- Compressed format saves storage space and speeds transfers
- Host-mounted backup directory makes file portable

**Key insight:** This pattern (temporary container + dual mounts) is the standard Docker method for volume backup.
</details>

<details>
<summary><strong>Q2: Why test container lifecycle persistence?</strong></summary>

**Sample Answer:**
Container lifecycle testing proves that volumes work as intended for stateful applications:

1. **Stop/Start test:** Simulates crashes or graceful restarts
2. **Complete replacement test:** Simulates container updates or host migrations
3. **Continuous operation test:** Shows data accumulates correctly over time

**Real-world scenarios this validates:**
- Database container survives host reboot
- Application updates don't lose user data
- Rollback to previous container version retains all data
- Container failure doesn't corrupt volume contents

**Production confidence:** These tests give assurance that critical data will survive routine operations and failures.
</details>

<details>
<summary><strong>Q3: Connection to disaster recovery</strong></summary>

**Question:** How does the backup/restore workflow connect to production disaster recovery planning?

**Sample Answer:**

**Backup frequency based on data criticality:**
- Database volumes: Hourly automated backups
- Log volumes: Daily backups (logs are less critical)
- Configuration volumes: On-change backups (rarely change)
- User content volumes: Multiple times per day

**Complete DR workflow:**
1. **Regular backups:** Automated cron jobs creating timestamped archives
2. **Off-site storage:** Copy backups to cloud storage (S3, Azure Blob)
3. **Backup verification:** Periodically restore to test volumes and verify
4. **Retention policy:** Keep daily for 7 days, weekly for 30 days, monthly for 1 year
5. **Restore procedure:** Documented steps for emergency recovery

**Test regularly:** Untested backups are not backups - practice restoration quarterly.
</details>

<details>
<summary><strong>Q4: Production scenario - Volume management</strong></summary>

**Scenario:** You manage a production Docker environment with 50 containers. How would you implement a volume management strategy?

**Sample Answer:**

**Naming convention:**
```
{service}-{purpose}-{environment}
db-data-prod
db-logs-prod  
api-uploads-prod
api-config-prod
```

**Labeling for automation:**
```bash
docker volume create \
  --label environment=production \
  --label backup-frequency=hourly \
  --label service=database \
  db-data-prod
```

**Monitoring script:**
```bash
# Alert if volumes exceed 80% capacity
docker system df -v | awk '/Local Volumes/,0' | \
  grep -E '[8-9][0-9]%|100%' && echo "WARNING: Volume near capacity"
```

**Backup automation:**
```bash
# Daily backup script
for vol in $(docker volume ls -q --filter label=backup-frequency=daily); do
  docker run --rm \
    -v $vol:/source:ro \
    -v /backups:/backup \
    ubuntu:20.04 \
    tar czf /backup/${vol}-$(date +%Y%m%d).tar.gz -C /source .
done
```

**Cleanup policy:**
- Remove volumes only when explicitly instructed
- Never use `docker volume prune` in production without review
- Maintain backup before any volume deletion
</details>

---

## 📝 Block Summary

**Concepts Mastered:**
- Creating portable volume backups
- Restoring volumes from archives
- Inspecting volume metadata and usage
- Testing data persistence across container lifecycle

**Critical Insights:**
1. Backup strategy uses temporary containers as bridges
2. Volume lifecycle is independent of container lifecycle
3. Testing persistence validates disaster recovery readiness
4. Production requires automated backup workflows

**Progress Update:**
- Foundation Concepts: ⭐⭐⭐⭐⭐ (100% complete)
- Hands-On Implementation: ⭐⭐⭐⭐⭐ (100% complete)
- Troubleshooting: ⭐⭐⭐⚪⚪ (60% complete)

**Next Up:** Comprehensive testing and troubleshooting

---

# 🔄 Phase 6: Spaced Review Checkpoint (3 minutes)

**Review earlier concepts to strengthen retention:**

<details>
<summary><strong>Concept Review - Test Your Memory</strong></summary>

**From Phase 2:** What are the two main types of Docker persistent storage?
- Answer: Volumes (Docker-managed) and Bind Mounts (host filesystem paths)

**From Phase 3:** What happens to volume data when you delete a container?
- Answer: Nothing - volume data persists independently of container lifecycle

**From Phase 4:** Why use read-only bind mounts?
- Answer: Prevent containers from modifying host files, improving security and preventing accidental changes

**From Phase 5:** What's the standard Docker backup approach?
- Answer: Use temporary container with dual mounts (volume + host directory) to create compressed archive

**Connection:** All these concepts work together - understanding ephemerality leads to using volumes, knowing lifecycle independence enables backup strategies, and security considerations guide read-only mounts.
</details>

---

# 🧪 Phase 7: Testing & Validation (5 minutes)

## Testing Philosophy

**Why test:** Validates your understanding, identifies gaps in knowledge, and builds confidence for production usage.

**Testing approach:** Progressive validation from basic functionality to integration scenarios.

---

## Create Comprehensive Test Suite

**Context:** A reusable testing script verifies Docker storage functionality across multiple scenarios.

```bash
# Create testing script
cat > ~/docker-lab/test-storage.sh << 'EOF'
#!/bin/bash

# Color codes for output
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Test counter
PASSED=0
FAILED=0

# Test function
run_test() {
  local test_name="$1"
  local test_command="$2"
  
  echo -e "\n${YELLOW}[TEST]${NC} $test_name"
  
  if eval "$test_command" > /dev/null 2>&1; then
    echo -e "${GREEN}[PASS]${NC} $test_name"
    ((PASSED++))
    return 0
  else
    echo -e "${RED}[FAIL]${NC} $test_name"
    ((FAILED++))
    return 1
  fi
}

# Print summary
print_summary() {
  local total=$((PASSED + FAILED))
  local percentage=$((PASSED * 100 / total))
  
  echo -e "\n=========================================="
  echo -e "           TEST SUMMARY"
  echo -e "=========================================="
  echo -e "Total Tests: $total"
  echo -e "${GREEN}Passed: $PASSED${NC}"
  echo -e "${RED}Failed: $FAILED${NC}"
  echo -e "Success Rate: $percentage%"
  echo -e "==========================================\n"
  
  if [ $percentage -ge 90 ]; then
    echo -e "${GREEN}✓ Excellent! Ready for production usage.${NC}"
  elif [ $percentage -ge 70 ]; then
    echo -e "${YELLOW}⚠ Good, but review failed tests.${NC}"
  else
    echo -e "${RED}✗ Needs improvement. Review concepts.${NC}"
  fi
}

echo "=========================================="
echo "    Docker Storage Test Suite"
echo "=========================================="

# Category 1: Basic Functionality Tests
echo -e "\n${YELLOW}=== Category 1: Basic Functionality ===${NC}"

run_test "Docker service is running" \
  "systemctl is-active --quiet docker"

run_test "Docker command works without sudo" \
  "docker ps > /dev/null"

run_test "Can create named volume" \
  "docker volume create test-vol-basic"

run_test "Volume appears in listing" \
  "docker volume ls | grep -q test-vol-basic"

run_test "Can inspect volume" \
  "docker volume inspect test-vol-basic | grep -q Mountpoint"

run_test "Can use volume in container" \
  "docker run --rm -v test-vol-basic:/data ubuntu:20.04 touch /data/test.txt"

run_test "Data persists in volume" \
  "docker run --rm -v test-vol-basic:/data ubuntu:20.04 ls /data/test.txt"

# Category 2: Bind Mount Tests
echo -e "\n${YELLOW}=== Category 2: Bind Mount Tests ===${NC}"

run_test "Can create bind mount directory" \
  "mkdir -p ~/docker-lab/test-bind && echo 'test' > ~/docker-lab/test-bind/file.txt"

run_test "Bind mount works with container" \
  "docker run --rm -v ~/docker-lab/test-bind:/mnt ubuntu:20.04 cat /mnt/file.txt | grep -q test"

run_test "Container can write to bind mount" \
  "docker run --rm -v ~/docker-lab/test-bind:/mnt ubuntu:20.04 touch /mnt/container-file.txt && [ -f ~/docker-lab/test-bind/container-file.txt ]"

run_test "Read-only bind mount prevents writes" \
  "! docker run --rm -v ~/docker-lab/test-bind:/mnt:ro ubuntu:20.04 touch /mnt/should-fail.txt 2>/dev/null"

# Category 3: Persistence Tests
echo -e "\n${YELLOW}=== Category 3: Persistence Tests ===${NC}"

run_test "Data survives container removal" \
  "docker run --name persist-test -v test-vol-persist:/data ubuntu:20.04 touch /data/persist.txt && \
   docker rm persist-test > /dev/null && \
   docker run --rm -v test-vol-persist:/data ubuntu:20.04 ls /data/persist.txt"

run_test "Multiple containers can share volume" \
  "docker run --name share1 -v test-vol-share:/data ubuntu:20.04 touch /data/shared.txt && \
   docker run --rm -v test-vol-share:/data ubuntu:20.04 ls /data/shared.txt && \
   docker rm share1 > /dev/null"

# Category 4: Backup/Restore Tests
echo -e "\n${YELLOW}=== Category 4: Backup/Restore Tests ===${NC}"

run_test "Can backup volume to tar.gz" \
  "docker run --rm -v test-vol-basic:/source:ro -v ~/docker-lab:/backup ubuntu:20.04 \
   tar czf /backup/test-backup.tar.gz -C /source . && \
   [ -f ~/docker-lab/test-backup.tar.gz ]"

run_test "Can restore volume from backup" \
  "docker volume create test-vol-restored && \
   docker run --rm -v test-vol-restored:/target -v ~/docker-lab:/backup ubuntu:20.04 \
   tar xzf /backup/test-backup.tar.gz -C /target"

run_test "Restored data matches original" \
  "docker run --rm -v test-vol-restored:/data ubuntu:20.04 ls /data/test.txt"

# Category 5: Cleanup Tests
echo -e "\n${YELLOW}=== Category 5: Cleanup Tests ===${NC}"

run_test "Can remove specific volume" \
  "docker volume rm test-vol-restored"

run_test "Removed volume no longer listed" \
  "! docker volume ls | grep -q test-vol-restored"

# Cleanup test artifacts
echo -e "\n${YELLOW}Cleaning up test artifacts...${NC}"
docker volume rm test-vol-basic test-vol-persist test-vol-share 2>/dev/null
rm -rf ~/docker-lab/test-bind ~/docker-lab/test-backup.tar.gz 2>/dev/null

# Print final summary
print_summary

EOF

chmod +x ~/docker-lab/test-storage.sh
```

---

## Run Test Suite

```bash
echo "=== Running Docker Storage Test Suite ==="
~/docker-lab/test-storage.sh
```

**Expected output:**
- Test results with PASS/FAIL indicators
- Categorized test groups
- Final summary with success rate
- Minimum 90% pass rate required

**If tests fail:**
1. Review the failed test category
2. Re-read the corresponding phase in this lab
3. Practice the failed operations manually
4. Run tests again

---

## Manual Verification Checklist

Beyond automated tests, verify these manually:

```bash
# Test 1: Volume listing and inspection
echo "=== Manual Test 1: Volume Management ==="
docker volume ls
docker volume inspect my-data-volume

# Test 2: Container with multiple volumes
echo -e "\n=== Manual Test 2: Multiple Volumes ==="
docker run --rm \
  -v app-logs:/logs \
  -v app-config:/config \
  ubuntu:20.04 bash -c 'touch /logs/test.log /config/test.conf && ls /logs /config'

# Test 3: Bind mount bidirectional sync
echo -e "\n=== Manual Test 3: Bind Mount Sync ==="
echo "Host created: $(date)" > ~/docker-lab/host-data/sync-test.txt
docker run --rm -v ~/docker-lab/host-data:/data ubuntu:20.04 cat /data/sync-test.txt
docker run --rm -v ~/docker-lab/host-data:/data ubuntu:20.04 bash -c 'echo "Container modified: $(date)" >> /data/sync-test.txt'
cat ~/docker-lab/host-data/sync-test.txt

# Test 4: Read-only mount enforcement
echo -e "\n=== Manual Test 4: Read-Only Mount ==="
docker run --rm -v ~/docker-lab/host-data:/data:ro ubuntu:20.04 bash -c 'touch /data/should-fail.txt 2>&1 || echo "PASS: Write correctly prevented"'

echo -e "\n${GREEN}✓ Manual verification complete${NC}"
```

---

## ✅ Testing Completion Checklist

- [ ] Automated test suite passes with ≥90% success rate
- [ ] All manual verification tests complete successfully
- [ ] Volume creation, usage, and deletion work correctly
- [ ] Bind mounts synchronize bidirectionally
- [ ] Read-only mounts prevent writes as expected
- [ ] Backup and restore operations succeed
- [ ] Data persists across container lifecycle operations

---

# 🧹 Phase 8: Cleanup (5 minutes)

## 🤔 Prediction Challenge #6 (30 seconds)

Before cleaning up, predict:
1. What order should you remove resources (containers first? volumes first?)?
2. What risks exist if you remove volumes before containers?
3. What should you preserve from this lab for future reference?

---

## Cleanup Philosophy

**Why proper cleanup matters:**
1. **Prevent resource exhaustion:** Unused volumes consume disk space
2. **Security:** Remove test data that might contain sensitive info
3. **Cost management:** Cloud environments charge for unused storage
4. **System hygiene:** Clean systems are easier to maintain

> ⚠️ **WARNING:** In production, NEVER use bulk cleanup commands (`docker volume prune -f`) without reviewing what will be deleted. Always verify volumes are truly unused.

---

## Pre-Cleanup Inventory

**Context:** Document current state before cleanup to track what was removed.

```bash
# Create inventory report
echo "=== Pre-Cleanup Inventory ===" > ~/docker-lab/cleanup-report.txt
echo "Date: $(date)" >> ~/docker-lab/cleanup-report.txt
echo "" >> ~/docker-lab/cleanup-report.txt

echo "=== Running Containers ===" >> ~/docker-lab/cleanup-report.txt
docker ps >> ~/docker-lab/cleanup-report.txt

echo -e "\n=== All Containers ===" >> ~/docker-lab/cleanup-report.txt
docker ps -a >> ~/docker-lab/cleanup-report.txt

echo -e "\n=== Volumes ===" >> ~/docker-lab/cleanup-report.txt
docker volume ls >> ~/docker-lab/cleanup-report.txt

echo -e "\n=== Disk Usage ===" >> ~/docker-lab/cleanup-report.txt
docker system df >> ~/docker-lab/cleanup-report.txt

cat ~/docker-lab/cleanup-report.txt
```

---

## Safe Cleanup Process

### Step 1: Stop and Remove Containers

**Context:** Containers must be removed before their volumes to prevent orphaned containers.

```bash
# Stop all running containers
echo "=== Stopping All Containers ==="
docker stop $(docker ps -aq) 2>/dev/null || echo "No containers to stop"

# Remove all containers
echo -e "\n=== Removing All Containers ==="
docker rm $(docker ps -aq) 2>/dev/null || echo "No containers to remove"

# Verify containers removed
echo -e "\n=== Verifying Container Removal ==="
docker ps -a
```

**Expected output:** "No containers to stop/remove" or empty container list.

---

### Step 2: Review Volumes Before Deletion

**Context:** Never delete volumes blindly - review what will be removed.

```bash
# List all volumes with details
echo "=== Reviewing Volumes for Cleanup ==="
docker volume ls

# Check which volumes are in use
echo -e "\n=== Checking Volume Usage ==="
docker system df -v | grep -A 50 "Local Volumes"
```

**Decision point:** Identify which volumes to keep (none for this lab, but in production you'd preserve important data).

---

### Step 3: Backup Important Volumes (If Needed)

**Context:** In production, always backup before deletion. We'll demonstrate the process.

```bash
# Example: Backup volumes before cleanup
echo "=== Creating Safety Backups ==="
mkdir -p ~/docker-lab/final-backups

for vol in my-data-volume database-data; do
  if docker volume ls | grep -q $vol; then
    echo "Backing up $vol..."
    docker run --rm \
      -v $vol:/source:ro \
      -v ~/docker-lab/final-backups:/backup \
      ubuntu:20.04 \
      tar czf /backup/${vol}-final.tar.gz -C /source . 2>/dev/null || echo "Volume $vol not found, skipping"
  fi
done

echo -e "\n=== Backup Files Created ==="
ls -lh ~/docker-lab/final-backups/
```

---

### Step 4: Remove Volumes

**Context:** With containers gone and backups created, safely remove volumes.

```bash
# Remove specific volumes from this lab
echo "=== Removing Lab Volumes ==="
docker volume rm my-data-volume restored-volume app-logs app-config database-data 2>/dev/null || echo "Some volumes already removed"

# Verify volumes removed
echo -e "\n=== Remaining Volumes ==="
docker volume ls
```

**Expected output:** Lab volumes removed, only system volumes (if any) remain.

---

### Step 5: Clean Host Directories

**Context:** Remove test files and directories created on host filesystem.

```bash
# Remove bind mount test data
echo "=== Cleaning Host Directories ==="
rm -rf ~/docker-lab/host-data/*
rm -rf ~/docker-lab/bind-mounts/*

# Keep structure but remove test files
ls -la ~/docker-lab/host-data/
ls -la ~/docker-lab/bind-mounts/
```

---

### Step 6: Optional - Remove Docker Images

**Context:** Lab pulled Ubuntu images. Remove only if disk space is critical.

```bash
# Show current images
echo "=== Current Docker Images ==="
docker images

# Optional: Remove Ubuntu images
echo -e "\n=== Removing Ubuntu Images (Optional) ==="
# Uncomment next line to remove:
# docker rmi ubuntu:20.04 alpine:latest 2>/dev/null
```

> ℹ️ **NOTE:** Keeping images speeds up future labs. Only remove if disk space is limited.

---

### Step 7: Verification

**Context:** Confirm cleanup was successful and system is in clean state.

```bash
# Final verification
echo "=== Post-Cleanup Verification ==="

echo "Containers:"
docker ps -a

echo -e "\nVolumes:"
docker volume ls

echo -e "\nDisk Usage:"
docker system df

# Create cleanup summary
echo -e "\n=== Cleanup Summary ===" >> ~/docker-lab/cleanup-report.txt
echo "Cleanup completed: $(date)" >> ~/docker-lab/cleanup-report.txt
echo "" >> ~/docker-lab/cleanup-report.txt
echo "Remaining containers: $(docker ps -a | wc -l)" >> ~/docker-lab/cleanup-report.txt
echo "Remaining volumes: $(docker volume ls | wc -l)" >> ~/docker-lab/cleanup-report.txt
echo "Final disk usage:" >> ~/docker-lab/cleanup-report.txt
docker system df >> ~/docker-lab/cleanup-report.txt

cat ~/docker-lab/cleanup-report.txt
```

---

## What to Preserve

**Keep these for future reference:**

```bash
echo "=== Preserving Lab Artifacts ==="
mkdir -p ~/docker-lab/reference

# Keep useful scripts
cp ~/docker-lab/test-storage.sh ~/docker-lab/reference/
cp ~/docker-lab/cleanup-report.txt ~/docker-lab/reference/

# Create quick reference guide
cat > ~/docker-lab/reference/quick-reference.md << 'EOF'
# Docker Storage Quick Reference

## Volume Commands
```bash
# Create volume
docker volume create volume-name

# List volumes
docker volume ls

# Inspect volume
docker volume inspect volume-name

# Remove volume
docker volume rm volume-name

# Backup volume
docker run --rm -v volume-name:/source:ro -v $(pwd):/backup ubuntu:20.04 \
  tar czf /backup/volume-backup.tar.gz -C /source .

# Restore volume
docker run --rm -v new-volume:/target -v $(pwd):/backup ubuntu:20.04 \
  tar xzf /backup/volume-backup.tar.gz -C /target
```

## Bind Mount Commands
```bash
# Create bind mount (read-write)
docker run -v /host/path:/container/path image

# Create bind mount (read-only)
docker run -v /host/path:/container/path:ro image

# Bind mount with user mapping
docker run --user $(id -u):$(id -g) -v /host/path:/container/path image
```

## Best Practices
- Use volumes for production data (databases, logs)
- Use bind mounts for development (live code reload)
- Always backup volumes before deletion
- Test backups regularly
- Use read-only mounts when containers don't need write access
- Never use `docker volume prune` in production without review
EOF

cat ~/docker-lab/reference/quick-reference.md
```

---

## ✅ Cleanup Completion Checklist

- [ ] All test containers stopped and removed
- [ ] All lab volumes removed (or backed up if needed)
- [ ] Host test directories cleaned
- [ ] Verification shows clean system state
- [ ] Quick reference guide created
- [ ] Cleanup report saved for records

---

## Best Practices Reflection

<details>
<summary><strong>Cleanup Order - Why it matters</strong></summary>

**Correct order:**
1. Stop running containers
2. Remove containers
3. (Optional) Backup volumes
4. Remove volumes
5. Clean host directories
6. (Optional) Remove images

**Why this order:**
- Containers hold references to volumes (remove containers first)
- Running containers must be stopped before removal
- Backups before deletion prevent data loss
- Images can be safely removed last (or kept for reuse)

**Wrong order consequences:**
- Removing volume while container uses it → container errors
- Removing image while container runs → startup failures on restart
</details>

<details>
<summary><strong>Production cleanup considerations</strong></summary>

**Never in production:**
- `docker volume prune -f` (removes ALL unused volumes)
- `docker system prune -a --volumes` (removes everything unused)
- Deleting volumes without backups
- Cleanup during business hours without approval

**Always in production:**
- Review what will be deleted
- Backup critical volumes first
- Use explicit deletion (name each volume)
- Document what was removed and why
- Schedule cleanup during maintenance windows
- Have rollback plan (restored backups)
</details>

---

# 🎓 Phase 9: Final Assessment (10 minutes)

## Part 1: Reflection

Take 2 minutes to visualize the complete workflow:

1. **Close your eyes** and picture the Docker storage architecture
2. **Trace the path** of data from container → volume → host disk
3. **Recall the commands** for creating, using, and backing up volumes
4. **Think through** a real-world scenario where you'd use each storage type

---

## Part 2: Teaching Scenario

**Challenge:** Explain Docker volumes to a junior developer in 3-5 minutes.

**Structure your explanation:**
1. **Hook:** Why containers lose data by default
2. **Core concept:** Volumes are persistent storage separate from containers
3. **Demonstration:** Show create → use → persist workflow
4. **Practice:** Have them create and test a volume
5. **Recap:** When to use volumes vs bind mounts

<details>
<summary><strong>Sample teaching script</strong></summary>

"Imagine you're staying in a hotel. Anything you rearrange in your room resets when you check out, right? That's how containers work - they're temporary by default.

Now imagine you also rent a storage unit across town. That storage unit keeps your stuff safe even after you leave the hotel. That's a Docker volume.

Let me show you:
```bash
# Create a storage unit (volume)
docker volume create my-storage

# Use it in a container
docker run -v my-storage:/data ubuntu bash -c 'echo "Important data" > /data/file.txt'

# Delete the container
docker rm container-name

# Create new container with same volume
docker run -v my-storage:/data ubuntu cat /data/file.txt
```

See? The data survived! That's the power of volumes.

**When to use what:**
- Volumes: Production databases, anything that must survive
- Bind mounts: Development files you edit on your computer

Try it yourself now - create a volume called 'test-volume' and write some data to it..."
</details>

---

## Part 3: Integration Questions

<details>
<summary><strong>Q1: Architecture Design</strong></summary>

**Question:** Design the storage architecture for this application:
- Web application (stateless)
- PostgreSQL database
- Redis cache
- User-uploaded images
- Application logs
- Nginx configuration

**Draw the architecture** (mentally or on paper) showing which storage type to use for each component.

**Sample Answer:**

```
Container Storage Architecture:

1. Web Application Container
   - Code: Baked into image (no volume)
   - Logs: Volume → web-app-logs
   
2. PostgreSQL Container
   - Data: Volume → postgresql-data (critical!)
   - Logs: Volume → postgresql-logs
   
3. Redis Container
   - Data: Volume → redis-data (or cache-only, no volume)
   
4. User Uploads Container
   - Images: Volume → user-uploads (must persist!)
   
5. Nginx Container
   - Config: Bind mount (read-only) → /etc/nginx/nginx.conf
   - Logs: Volume → nginx-logs
   - TLS certs: Bind mount (read-only) → /etc/ssl/certs

Backup Strategy:
- postgresql-data: Hourly backups
- user-uploads: Daily backups
- redis-data: No backups (cache can rebuild)
- Logs: Weekly backups, 30-day retention
```

**Reasoning:**
- Database needs volumes (critical data)
- Configs use read-only bind mounts (security)
- Logs use volumes (separate lifecycle from apps)
- Cache doesn't need persistence (can rebuild)
</details>

<details>
<summary><strong>Q2: Troubleshooting Scenario</strong></summary>

**Scenario:** A developer reports: "My database container keeps losing data when I restart it!"

**Your systematic approach:**

1. **Gather information:**
   ```bash
   # Check if volume is mounted
   docker inspect db-container | grep -A 10 Mounts
   
   # Check container run command
   docker inspect db-container | grep -A 5 "Cmd\|Entrypoint"
   ```

2. **Identify the problem:**
   - No volume mounted → data in container filesystem
   - Anonymous volume → different volume each restart
   - Wrong mount point → data written elsewhere

3. **Common causes:**
   ```bash
   # Problem: No volume at all
   docker run postgres  # ❌ No -v flag
   
   # Problem: Anonymous volume
   docker run -v /var/lib/postgresql/data postgres  # ❌ No volume name
   
   # Solution: Named volume
   docker run -v pgdata:/var/lib/postgresql/data postgres  # ✅ Correct
   ```

4. **Fix and verify:**
   ```bash
   # Create proper volume
   docker volume create db-data
   
   # Run with named volume
   docker run -d --name db -v db-data:/var/lib/postgresql/data postgres
   
   # Test persistence
   docker exec db psql -c "CREATE TABLE test (id INT);"
   docker restart db
   docker exec db psql -c "SELECT * FROM test;"  # Should work
   ```

5. **Prevention:**
   - Always use named volumes for stateful containers
   - Document volume requirements
   - Use docker-compose for complex setups
</details>

<details>
<summary><strong>Q3: Production Considerations</strong></summary>

**Question:** You're deploying the database container to production. What storage considerations must you address?

**Comprehensive answer:**

**Security:**
- Volume encryption at rest
- Access controls (who can mount/backup volumes)
- No sensitive data in container images
- Read-only mounts where possible
- Regular security audits of volume permissions

**Monitoring:**
- Disk space alerts (warn at 80%, critical at 90%)
- Volume growth rate tracking
- I/O performance metrics
- Backup success/failure monitoring
- Restore test results

**Reliability:**
- RAID configuration for volume storage
- Regular automated backups (tested monthly)
- Off-site backup replication
- Point-in-time recovery capability
- Documented restore procedures

**Operations:**
- Volume naming conventions
- Labeling for automation
- Backup retention policy (7 daily, 4 weekly, 12 monthly)
- Change management for volume operations
- Runbooks for common scenarios

**Performance:**
- SSD storage for database volumes
- Separate volumes for logs (different I/O pattern)
- Volume drivers optimized for workload
- Regular performance baseline testing

**Documentation:**
- Architecture diagrams
- Backup/restore procedures
- Troubleshooting guides
- Capacity planning data
- Disaster recovery plan
