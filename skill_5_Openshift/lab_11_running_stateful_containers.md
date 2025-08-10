# Lab 11: Running Stateful Containers

## **Objectives**
* Understand the difference between stateless and stateful containers
* Implement persistent storage using volume mounts and bind mounts
* Run MySQL and PostgreSQL containers with persistent data
* Verify data persistence across container lifecycles
* Learn troubleshooting techniques for stateful containers
* Understand security implications of persistent storage

## **Prerequisites**
* Podman or Docker installed (Podman recommended for OpenShift alignment)
* Basic Linux command line proficiency
* 4GB+ free disk space
* Internet access to pull container images
* Basic understanding of SQL databases

## **Understanding Stateful vs Stateless Containers**

**Stateless Containers**: Data is ephemeral and lost when container is removed
**Stateful Containers**: Data persists beyond container lifecycle using external storage

---

## **Lab Setup**

### **Step 1: Verify Podman Installation**

```bash
podman --version
```

**Command Explanation**: 
- `podman --version`: Displays the installed Podman version
- **Expected Output**: `podman version 4.x.x` (version may vary)

**If Podman is not installed** (RHEL/CentOS/Fedora):
```bash
sudo dnf install -y podman
```

### **Step 2: Check System Resources**

```bash
df -h ~/
free -h
```

**Command Explanation**:
- `df -h ~/`: Shows available disk space in home directory (human-readable format)
- `free -h`: Displays available memory
- **Purpose**: Ensures sufficient resources for database containers

### **Step 3: Create Working Directory Structure**

```bash
mkdir -p ~/stateful-lab/{mysql-data,pg-data,backups}
cd ~/stateful-lab
ls -la
```

**Command Explanation**:
- `mkdir -p`: Creates parent directories if they don't exist
- `{mysql-data,pg-data,backups}`: Bash brace expansion creates multiple directories
- `ls -la`: Lists all files including hidden ones with detailed permissions

### **Step 4: Understand Current Directory Structure**

```bash
pwd
tree . 2>/dev/null || find . -type d
```

**Command Explanation**:
- `pwd`: Prints current working directory path
- `tree .`: Shows directory structure (if installed)
- `2>/dev/null`: Redirects errors to null if tree isn't installed
- `|| find . -type d`: Alternative command if tree fails

---

## **Task 1: Running MySQL Container with Persistent Storage**

### **Subtask 1.1: Understand Volume Types**

**Three types of persistent storage in containers**:
1. **Bind Mounts**: Host directory mounted into container
2. **Named Volumes**: Managed by container runtime
3. **tmpfs Mounts**: Temporary filesystem in memory

We'll use **bind mounts** for direct host access to data.

### **Subtask 1.2: Set Proper Permissions**

```bash
ls -ld mysql-data
sudo chown -R 999:999 mysql-data
ls -ld mysql-data
```

**Command Explanation**:
- `ls -ld`: Shows directory permissions (not contents)
- `chown 999:999`: MySQL container runs as UID/GID 999
- **Purpose**: Prevents permission denied errors when container writes data

### **Subtask 1.3: Pull MySQL Image**

```bash
podman pull docker.io/library/mysql:8.0
podman images | grep mysql
```

**Command Explanation**:
- `podman pull`: Downloads container image from registry
- `docker.io/library/mysql:8.0`: Official MySQL 8.0 image from Docker Hub
- `podman images | grep mysql`: Filters image list to show MySQL images

### **Subtask 1.4: Launch MySQL Container with Persistent Storage**

```bash
podman run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=redhat123 \
  -e MYSQL_DATABASE=testdb \
  -e MYSQL_USER=testuser \
  -e MYSQL_PASSWORD=user123 \
  -v $(pwd)/mysql-data:/var/lib/mysql \
  -p 3306:3306 \
  --health-cmd="mysqladmin ping -h localhost" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  docker.io/library/mysql:8.0
```

**Command Explanation**:
- `-d`: Run in detached mode (background)
- `--name mysql-db`: Assigns container name for easier reference
- `-e MYSQL_ROOT_PASSWORD`: Sets MySQL root password
- `-e MYSQL_DATABASE`: Creates initial database named 'testdb'
- `-e MYSQL_USER/PASSWORD`: Creates non-root user with password
- `-v $(pwd)/mysql-data:/var/lib/mysql`: Bind mounts host directory to MySQL data directory
- `-p 3306:3306`: Maps host port 3306 to container port 3306
- `--health-*`: Configures health checks to monitor container status

### **Subtask 1.5: Monitor Container Startup**

```bash
# Check container status
podman ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}"

# Monitor logs during startup
podman logs -f mysql-db
```

**Command Explanation**:
- `podman ps -a`: Shows all containers (running and stopped)
- `--format "table {{.template}}"`: Custom output format
- `podman logs -f`: Follows log output in real-time
- **Press Ctrl+C** to exit log following

### **Subtask 1.6: Verify Container Health**

```bash
podman inspect mysql-db --format '{{.State.Health.Status}}'
podman port mysql-db
```

**Command Explanation**:
- `podman inspect`: Shows detailed container information
- `--format`: Extracts specific information (health status)
- `podman port`: Lists port mappings for the container

### **Subtask 1.7: Check Host Directory Changes**

```bash
ls -la mysql-data/
du -sh mysql-data/
```

**Command Explanation**:
- `ls -la mysql-data/`: Lists MySQL data files created by container
- `du -sh`: Shows disk usage of directory in human-readable format
- **Observation**: Container created MySQL system databases

### **Subtask 1.8: Test Database Connection and Create Data**

```bash
# Connect to MySQL as testuser
podman exec -it mysql-db mysql -u testuser -puser123 testdb
```

**In MySQL shell, execute**:
```sql
-- Show current database
SELECT DATABASE();

-- Create a test table
CREATE TABLE lab_data (
    id INT AUTO_INCREMENT PRIMARY KEY,
    message VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert test data
INSERT INTO lab_data (message) VALUES 
    ('First persistent record'),
    ('Second persistent record'),
    ('Container restart test data');

-- Verify data insertion
SELECT * FROM lab_data;

-- Show table structure
DESCRIBE lab_data;

-- Exit MySQL shell
exit;
```

**Command Explanation**:
- `podman exec -it`: Execute interactive terminal in running container
- `mysql -u testuser -puser123 testdb`: Connect to MySQL as testuser to testdb database
- SQL commands create table, insert data, and verify success

---

## **Task 2: Verify Data Persistence**

### **Subtask 2.1: Create Backup Before Testing**

```bash
podman exec mysql-db mysqldump -u testuser -puser123 testdb > backups/testdb_backup.sql
cat backups/testdb_backup.sql | head -20
```

**Command Explanation**:
- `mysqldump`: MySQL utility to export database structure and data
- `> backups/testdb_backup.sql`: Redirects output to backup file
- `cat ... | head -20`: Shows first 20 lines of backup file

### **Subtask 2.2: Stop and Remove Container**

```bash
# Check current container status
podman ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Stop container gracefully
podman stop mysql-db

# Remove container (but keep volume data)
podman rm mysql-db

# Verify container is gone
podman ps -a | grep mysql || echo "Container successfully removed"
```

**Command Explanation**:
- `podman stop`: Gracefully stops container (sends SIGTERM, then SIGKILL)
- `podman rm`: Removes container but preserves mounted volumes
- `|| echo`: Shows message if grep finds no matches

### **Subtask 2.3: Verify Data Files Still Exist**

```bash
ls -la mysql-data/ | head -10
echo "Data directory size:"
du -sh mysql-data/
```

**Command Explanation**:
- **Key Learning**: Container removal doesn't affect bind-mounted directories
- Data persists on host filesystem independent of container lifecycle

### **Subtask 2.4: Recreate Container with Same Volume Mount**

```bash
podman run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=redhat123 \
  -v $(pwd)/mysql-data:/var/lib/mysql \
  -p 3306:3306 \
  --health-cmd="mysqladmin ping -h localhost" \
  --health-interval=30s \
  docker.io/library/mysql:8.0

# Wait for container to be ready
echo "Waiting for MySQL to start..."
sleep 10
```

**Command Explanation**:
- **Notice**: We don't need to recreate user/database environment variables
- Existing data files contain all previous configuration
- Container discovers existing databases on startup

### **Subtask 2.5: Verify Data Persistence**

```bash
# Check container is healthy
podman ps --format "table {{.Names}}\t{{.Status}}"

# Test direct SQL query
podman exec mysql-db mysql -u testuser -puser123 testdb -e "SELECT COUNT(*) as 'Total Records', MAX(created_at) as 'Latest Entry' FROM lab_data;"

# Interactive verification
podman exec -it mysql-db mysql -u testuser -puser123 testdb
```

**In MySQL shell**:
```sql
-- Verify all data is present
SELECT * FROM lab_data ORDER BY id;

-- Add new data to prove container is fully functional
INSERT INTO lab_data (message) VALUES ('Data after container recreation');

-- Show all records including new one
SELECT id, message, created_at FROM lab_data ORDER BY id;

exit;
```

**Key Learning Points**:
- All original data survived container removal
- Database relationships, users, and permissions preserved
- New container can immediately read/write existing data

---

## **Task 3: PostgreSQL Implementation**

### **Subtask 3.1: Prepare PostgreSQL Environment**

```bash
# Set proper permissions for PostgreSQL
sudo chown -R 999:999 pg-data
ls -ld pg-data

# Pull PostgreSQL image
podman pull docker.io/library/postgres:15
```

**Command Explanation**:
- PostgreSQL also runs as UID 999 in the container
- Using PostgreSQL 15 for better features and performance

### **Subtask 3.2: Launch PostgreSQL Container**

```bash
podman run -d \
  --name postgres-db \
  -e POSTGRES_PASSWORD=redhat123 \
  -e POSTGRES_USER=testuser \
  -e POSTGRES_DB=testdb \
  -v $(pwd)/pg-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  --health-cmd="pg_isready -U testuser -d testdb" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  docker.io/library/postgres:15

# Monitor startup
podman logs -f postgres-db
```

**Command Explanation**:
- `POSTGRES_*`: Environment variables for PostgreSQL configuration
- `-p 5432:5432`: Standard PostgreSQL port mapping
- `pg_isready`: PostgreSQL utility for health checking

### **Subtask 3.3: Test PostgreSQL Data Persistence**

```bash
# Wait for PostgreSQL to be ready
sleep 5

# Create test data
podman exec -it postgres-db psql -U testuser -d testdb
```

**In PostgreSQL shell**:
```sql
-- Show current database and user
\c
\du

-- Create test table with different data types
CREATE TABLE lab_data (
    id SERIAL PRIMARY KEY,
    message TEXT NOT NULL,
    value NUMERIC(10,2),
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert diverse test data
INSERT INTO lab_data (message, value, is_active) VALUES 
    ('PostgreSQL persistent data', 123.45, true),
    ('Another test record', 67.89, false),
    ('Final test entry', 999.99, true);

-- Verify data with formatting
SELECT id, message, value, is_active, 
       to_char(created_at, 'YYYY-MM-DD HH24:MI:SS') as created 
FROM lab_data;

-- Show table structure
\d lab_data

-- Exit PostgreSQL shell
\q
```

### **Subtask 3.4: Test PostgreSQL Persistence**

```bash
# Create PostgreSQL backup
podman exec postgres-db pg_dump -U testuser testdb > backups/postgres_testdb_backup.sql

# Stop and remove PostgreSQL container
podman stop postgres-db
podman rm postgres-db

# Verify data directory exists
ls -la pg-data/ | head -5
du -sh pg-data/

# Recreate container
podman run -d \
  --name postgres-db \
  -e POSTGRES_PASSWORD=redhat123 \
  -v $(pwd)/pg-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  docker.io/library/postgres:15

# Wait and test data persistence
sleep 10
podman exec postgres-db psql -U testuser -d testdb -c "SELECT COUNT(*) as total_records FROM lab_data;"
```

---

## **Task 4: Advanced Volume Management**

### **Subtask 4.1: Compare Storage Methods**

```bash
# Check disk usage of both databases
echo "MySQL data size:"
du -sh mysql-data/

echo "PostgreSQL data size:"
du -sh pg-data/

# Create named volume for comparison
podman volume create mysql-named-vol
podman volume ls
podman volume inspect mysql-named-vol
```

**Command Explanation**:
- Named volumes are managed by Podman in `/var/lib/containers/storage/volumes/`
- Bind mounts use any host directory path
- Named volumes offer better portability across systems

### **Subtask 4.2: Container Resource Usage**

```bash
# Check container resource usage
podman stats --no-stream mysql-db postgres-db

# Check container processes
podman top mysql-db
podman top postgres-db
```

**Command Explanation**:
- `podman stats`: Shows CPU, memory, network usage
- `podman top`: Shows processes running inside containers

---

## **Task 5: Security and Best Practices**

### **Subtask 5.1: Security Analysis**

```bash
# Check file permissions in mounted directories
ls -la mysql-data/ | head -5
ls -la pg-data/ | head -5

# Check container security information
podman inspect mysql-db --format '{{.HostConfig.SecurityOpt}}'
podman inspect postgres-db --format '{{.HostConfig.SecurityOpt}}'
```

### **Subtask 5.2: Network Security**

```bash
# Check what ports are listening
sudo ss -tulnp | grep -E "(3306|5432)"

# Check container network information
podman inspect mysql-db --format '{{.NetworkSettings.IPAddress}}'
podman inspect postgres-db --format '{{.NetworkSettings.IPAddress}}'
```

---

## **Troubleshooting Guide**

### **Common Issues and Solutions**

#### **1. Permission Denied Errors**
```bash
# Fix ownership issues
sudo chown -R 999:999 mysql-data/
sudo chown -R 999:999 pg-data/

# Alternative: Use named volumes
podman volume create secure-mysql-vol
```

#### **2. Port Conflicts**
```bash
# Check what's using the port
sudo ss -tulnp | grep 3306

# Use different host port
podman run -p 3307:3306 ...
```

#### **3. Container Startup Failures**
```bash
# Check detailed logs
podman logs mysql-db --since="10m"

# Check container events
podman events --since="5m" --filter container=mysql-db

# Inspect container configuration
podman inspect mysql-db --format='{{.Config.Env}}'
```

#### **4. Data Corruption Issues**
```bash
# Stop container gracefully
podman stop --time=30 mysql-db

# Check filesystem for errors
sudo fsck /dev/$(df mysql-data | tail -1 | awk '{print $1}' | sed 's/[0-9]*$//')

# Restore from backup
podman exec -i mysql-db mysql -u testuser -puser123 testdb < backups/testdb_backup.sql
```

---

## **Cleanup**

```bash
# Stop all containers
podman stop mysql-db postgres-db

# Remove containers
podman rm mysql-db postgres-db

# Remove images (optional)
podman rmi mysql:8.0 postgres:15

# Clean up volumes (WARNING: This deletes data)
# rm -rf mysql-data/ pg-data/

# Clean up named volumes
podman volume rm mysql-named-vol
```

---

## **Key Learning Outcomes**

### **What You've Learned**:
1. **Container Storage Types**: Bind mounts vs named volumes vs tmpfs
2. **Data Persistence**: How to maintain data across container lifecycles
3. **Database Containers**: Specific requirements for MySQL and PostgreSQL
4. **Security Considerations**: File permissions and network exposure
5. **Troubleshooting**: Common issues and diagnostic techniques
6. **Best Practices**: Backup strategies and resource management

### **Knowledge Check Questions**

1. **What happens if you omit the `-v` flag when running a database container?**
   - Data is stored in container's writable layer and lost when container is removed

2. **How would you migrate persistent data to another host?**
   - Copy bind mount directories or use database backup/restore tools

3. **What security risks exist with bind mounts?**
   - Host filesystem exposure, incorrect permissions, container breakout potential

4. **Why do we need specific UID/GID permissions?**
   - Container processes run as specific users and need write access to mounted directories

5. **How do named volumes differ from bind mounts?**
   - Named volumes are managed by container runtime, bind mounts use explicit host paths

---

## **Further Exploration**

### **Advanced Topics to Investigate**:
1. **Volume Drivers**: Explore different storage backends
2. **Multi-Host Storage**: Network-attached storage solutions
3. **Backup Automation**: Scheduled database backups in containers
4. **Performance Tuning**: Optimizing database containers for production
5. **Container Orchestration**: Using volumes in Kubernetes/OpenShift
6. **Security Hardening**: Running databases with non-root users

### **Practical Exercises**:
1. Set up automated backups using cron and container exec
2. Create a multi-container application with shared storage
3. Implement database replication using multiple containers
4. Explore different PostgreSQL and MySQL configuration options
5. Set up monitoring for containerized databases
