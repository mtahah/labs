### **Lab 32 (Revised): Docker for Big Data - Running Apache Hadoop in Docker**

This revised lab provides up-to-date instructions for setting up a Hadoop cluster using the latest Docker tools on an Ubuntu system.

### **Objectives**
By the end of this lab, you will be able to:
*   Set up and configure Docker containers for Apache Hadoop components (HDFS and YARN).
*   Deploy a multi-container Hadoop environment using Docker Compose.
*   Interact with the Hadoop Distributed File System (HDFS) through containerized interfaces.
*   Submit and monitor MapReduce jobs to a YARN cluster running in Docker containers.
*   Understand the fundamentals of big data processing in a containerized environment.
*   Troubleshoot common issues when running Hadoop in Docker.

### **Prerequisites**
*   Basic understanding of Linux command-line operations.
*   Familiarity with Docker concepts (containers, images, volumes).
*   Basic knowledge of distributed systems concepts.

### **Technical Requirements**
*   A machine running Ubuntu 22.04 LTS (or a similar Debian-based distribution).
*   At least 4GB of available RAM.
*   10GB of free disk space.
*   Internet access to download Docker images and packages.

---

### **Task 0: Initial Machine Setup**

This task will prepare your system by fixing any existing package issues, updating the system, and installing the latest versions of Docker Engine and Docker Compose.

**Subtask 0.1: Fix and Update System Packages**
First, we'll clean up and update the `apt` package manager to ensure a stable base.

```bash
# Clean up the local repository of retrieved package files
sudo apt-get clean

# Fix any broken package dependencies
sudo apt-get install -f

# Update the package lists for upgrades and new package installations
sudo apt-get update

# Upgrade all installed packages to their latest versions
sudo apt-get upgrade -y

# Install prerequisite packages for adding new repositories over HTTPS
sudo apt-get install -y ca-certificates curl gnupg
```

**Subtask 0.2: Install Docker Engine and Docker Compose**
We will follow the official Docker documentation to install the latest versions.

```bash
# Add Docker's official GPG key to ensure package authenticity
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the Docker repository to your APT sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update the package list again to include the new Docker repository
sudo apt-get update

# Install the latest versions of Docker Engine, CLI, containerd, and the Compose plugin
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**Subtask 0.3: Configure Docker Post-Installation**
To run Docker commands without `sudo`, we'll add your user to the `docker` group.

```bash
# Create the docker group if it doesn't already exist
sudo groupadd docker

# Add your current user to the docker group
sudo usermod -aG docker $USER

# IMPORTANT: You must log out and log back in for this change to take effect.
# You can also run the following command to apply the new group membership immediately:
newgrp docker
```
*Note: After running `newgrp docker`, your shell session will have the correct permissions. For a permanent fix, please log out and back in.*

**Subtask 0.4: Verify the Installation**
Let's check that Docker and Docker Compose are installed correctly.

```bash
# Check the Docker version
docker --version

# Check the Docker Compose version
docker compose version

# Run the hello-world container to verify Docker is working
docker run hello-world
```

### **Task 1: Set up Docker Containers for Hadoop (HDFS and YARN)**

**Subtask 1.1: Create Project Directory Structure**
First, let's create an organized directory structure for our Hadoop Docker setup.

```bash
# Create the main project directory in your home folder
mkdir ~/hadoop-docker-lab
cd ~/hadoop-docker-lab

# Create subdirectories for configuration, data, and logs
mkdir -p config/hadoop
mkdir -p data/namenode
mkdir -p data/datanode
mkdir -p data/logs
```

**Subtask 1.2: Create Hadoop Configuration Files**
We will use the `nano` text editor to create the necessary Hadoop configuration files.

**Create `core-site.xml`:**
```bash
# Open the file in the nano editor
nano config/hadoop/core-site.xml
```
Copy the following XML content into the nano editor. Then press `Ctrl+X`, then `Y`, then `Enter` to save and exit.
```xml
<!-- CORRECT -->
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://namenode:9000</value>
    </property>
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>root</value>
    </property>
</configuration>
```

**Create `hdfs-site.xml`:**
```bash
# Open the file in the nano editor
nano config/hadoop/hdfs-site.xml
```
Copy the following XML content into nano, then save and exit (`Ctrl+X`, `Y`, `Enter`).
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/hadoop/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/hadoop/dfs/data</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.permissions.enabled</name>
        <value>false</value>
    </property>
    <property>
        <name>dfs.namenode.http-address</name>
        <value>namenode:9870</value>
    </property>
</configuration>
```

**Create `mapred-site.xml`:**
```bash
# Open the file in the nano editor
nano config/hadoop/mapred-site.xml
```
Copy the following XML content into nano, then save and exit (`Ctrl+X`, `Y`, `Enter`).
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
</configuration>
```

**Create `yarn-site.xml`:**
```bash
# Open the file in the nano editor
nano config/hadoop/yarn-site.xml
```
Copy the following XML content into nano, then save and exit (`Ctrl+X`, `Y`, `Enter`).
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>resourcemanager</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```

**Subtask 1.3: Verify Configuration Files**
Let's verify that all configuration files were created correctly.

```bash
# List all configuration files to ensure they exist
ls -la config/hadoop/

# Check the content of one configuration file to confirm it's correct
cat config/hadoop/core-site.xml
```

### **Task 2: Start a Multi-Container Hadoop Environment with Docker Compose**

**Subtask 2.1: Create Docker Compose File**
Now, create the `docker-compose.yml` file that defines all our Hadoop services.

```bash
# Open the docker-compose.yml file in the nano editor
nano docker-compose.yml
```
Copy the following YAML content into nano, then save and exit (`Ctrl+X`, `Y`, `Enter`).

```yaml
version: '3.8'

services:
  namenode:
    image: apache/hadoop:3.3.6
    container_name: hadoop-namenode
    hostname: namenode
    ports:
      - "9870:9870"  # NameNode Web UI
      - "9000:9000"  # HDFS
    volumes:
      - ./config/hadoop:/opt/hadoop/etc/hadoop
      - ./data/namenode:/hadoop/dfs/name
      - ./data/logs:/opt/hadoop/logs
    environment:
      - CLUSTER_NAME=hadoop-cluster
    command: >
      bash -c "
        hdfs namenode -format -force &&
        hdfs namenode
      "
    networks:
      - hadoop-network

  datanode:
    image: apache/hadoop:3.3.6
    container_name: hadoop-datanode
    hostname: datanode
    ports:
      - "9864:9864"  # DataNode Web UI
    volumes:
      - ./config/hadoop:/opt/hadoop/etc/hadoop
      - ./data/datanode:/hadoop/dfs/data
      - ./data/logs:/opt/hadoop/logs
    environment:
      - CLUSTER_NAME=hadoop-cluster
    command: hdfs datanode
    depends_on:
      - namenode
    networks:
      - hadoop-network

  resourcemanager:
    image: apache/hadoop:3.3.6
    container_name: hadoop-resourcemanager
    hostname: resourcemanager
    ports:
      - "8088:8088"  # ResourceManager Web UI
    volumes:
      - ./config/hadoop:/opt/hadoop/etc/hadoop
      - ./data/logs:/opt/hadoop/logs
    environment:
      - CLUSTER_NAME=hadoop-cluster
    command: yarn resourcemanager
    depends_on:
      - namenode
    networks:
      - hadoop-network

  nodemanager:
    image: apache/hadoop:3.3.6
    container_name: hadoop-nodemanager
    hostname: nodemanager
    ports:
      - "8042:8042"  # NodeManager Web UI
    volumes:
      - ./config/hadoop:/opt/hadoop/etc/hadoop
      - ./data/logs:/opt/hadoop/logs
    environment:
      - CLUSTER_NAME=hadoop-cluster
    command: yarn nodemanager
    depends_on:
      - resourcemanager
    networks:
      - hadoop-network

  historyserver:
    image: apache/hadoop:3.3.6
    container_name: hadoop-historyserver
    hostname: historyserver
    ports:
      - "19888:19888"  # History Server Web UI
    volumes:
      - ./config/hadoop:/opt/hadoop/etc/hadoop
      - ./data/logs:/opt/hadoop/logs
    environment:
      - CLUSTER_NAME=hadoop-cluster
    command: mapred historyserver
    depends_on:
      - namenode
    networks:
      - hadoop-network

networks:
  hadoop-network:
    driver: bridge
```

**Subtask 2.2: Start the Hadoop Cluster**
Now let's start our multi-container Hadoop environment.

```bash
# Start all services in detached mode (in the background)
docker compose up -d

# Check the status of all containers
docker compose ps
```
You should see all five services (`namenode`, `datanode`, `resourcemanager`, `nodemanager`, `historyserver`) with a status of `Up` or `running`.

**Subtask 2.3: Verify Cluster Status**
Wait 2-3 minutes for all services to initialize, then verify the cluster is running.

```bash
# Check the logs for the namenode to ensure it has started correctly
docker compose logs namenode

# Execute a command inside the namenode container to get an HDFS report
docker exec hadoop-namenode hdfs dfsadmin -report
```
The report should show 1 live datanode.

### **Task 3: Interact with the Hadoop Distributed File System (HDFS)**

**Subtask 3.1: Access HDFS Command Line Interface**
Let's get a shell inside the `namenode` container to run HDFS commands.

```bash
# Access the namenode container's bash shell
docker exec -it hadoop-namenode bash

# Inside the container, check HDFS status again
hdfs dfsadmin -report

# List the contents of the HDFS root directory (it should be empty)
hdfs dfs -ls /

# Create directories in HDFS for our user and data
hdfs dfs -mkdir /user
hdfs dfs -mkdir /user/input
```

**Subtask 3.2: Upload and Manage Files in HDFS**
Create sample data inside the container and upload it to HDFS.

```bash
# Still inside the namenode container, create a sample text file
echo "Hello Hadoop World" > /tmp/sample.txt
echo "This is a test file for HDFS" >> /tmp/sample.txt
echo "Docker makes Hadoop deployment easy" >> /tmp/sample.txt

# Upload the local file from the container to HDFS
hdfs dfs -put /tmp/sample.txt /user/input/

# List files in our HDFS input directory
hdfs dfs -ls /user/input/

# View the content of the file directly from HDFS
hdfs dfs -cat /user/input/sample.txt
```

**Subtask 3.3: Explore HDFS Web Interface**
Exit the container and access the HDFS web UI from your browser.

```bash
# Exit the container's shell
exit

# The following URLs can be opened in your local web browser
echo "Access NameNode Web UI at: http://localhost:9870"
echo "Access DataNode Web UI at: http://localhost:9864"
```
Navigate to `http://localhost:9870` in your browser. Go to "Utilities" -> "Browse the file system" to see the `/user/input/sample.txt` file you created.

### **Task 4: Submit Jobs to the YARN Cluster**

**Subtask 4.1: Prepare Data for a MapReduce Job**
Let's create a larger file for a classic word count job.

```bash
# Access the namenode container again
docker exec -it hadoop-namenode bash

# Create a larger sample file for word counting
# Note: We use 'cat' here inside the container for multi-line input
cat > /tmp/wordcount-input.txt << 'EOF'
Apache Hadoop is a collection of open-source software utilities
that facilitates using a network of many computers to solve problems
involving massive amounts of data and computation. It provides a
software framework for distributed storage and processing of big data
using the MapReduce programming model. Hadoop was originally designed
for computer clusters built from commodity hardware. Docker containers
make it easy to deploy and manage Hadoop clusters in various environments.
EOF

# Upload this new file to HDFS
hdfs dfs -put /tmp/wordcount-input.txt /user/input/

# Verify the upload
hdfs dfs -ls /user/input/
```

**Subtask 4.2: Run a MapReduce Word Count Job**
Execute the word count example that comes with Hadoop.

```bash
# Still inside the namenode container, run the word count job
# It will read from /user/input and write to /user/output/wordcount
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar wordcount /user/input /user/output/wordcount

# Check if the job completed and created an output directory
hdfs dfs -ls /user/output/wordcount/

# View the results of the word count job
hdfs dfs -cat /user/output/wordcount/part-r-00000
```

**Subtask 4.3: Monitor Jobs through YARN Web Interface**
Exit the container to access the YARN web UIs.

```bash
# Exit the container
exit

# Use these URLs in your local browser to monitor the cluster and jobs
echo "Access YARN ResourceManager Web UI at: http://localhost:8088"
echo "Access History Server Web UI at: http://localhost:19888"
```
Navigate to `http://localhost:8088`. You should see your completed WordCount job in the "Applications" list.

**Subtask 4.4: Run an Additional MapReduce Example (Pi Estimation)**
Let's run another example to see a different kind of job.

```bash
# Access the namenode container again
docker exec -it hadoop-namenode bash

# Run a Pi estimation job using the Monte Carlo method
# This job calculates Pi with 10 maps and 100 samples each
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar pi 10 100

# Exit the container
exit
```
You can now check the YARN ResourceManager UI again to see this new job.

### **Task 5: Cleanup and Troubleshooting**

**Subtask 5.1: HDFS Cleanup**
Let's remove the output directories from our jobs in HDFS.

```bash
# Get a shell in the namenode container
docker exec -it hadoop-namenode bash

# Remove the HDFS output directories
hdfs dfs -rm -r /user/output/wordcount

# Check remaining HDFS space
hdfs dfs -df -h

# Exit the container
exit
```

**Subtask 5.2: Troubleshooting**
If you encounter issues, these commands are helpful.
*   **Containers not starting:** Check logs for a specific service.
    ```bash
    docker compose logs namenode
    ```
*   **HDFS in Safe Mode:** This can happen on startup. HDFS will exit safe mode automatically once the datanode reports in. You can check the status.
    ```bash
    docker exec hadoop-namenode hdfs dfsadmin -safemode get
    ```
*   **Web UI not accessible:** Ensure the containers are running and check port mappings.
    ```bash
    docker compose ps
    docker port hadoop-namenode
    ```

### **Lab Cleanup**
When you are finished, run these commands to stop and remove all lab resources.

```bash
# Navigate to your project directory if you aren't already there
cd ~/hadoop-docker-lab

# Stop and remove all containers, networks, and volumes defined in the compose file
docker compose down --volumes

# Optional: Remove the Hadoop Docker image to free up disk space
docker rmi apache/hadoop:3.3.6

# Optional: Remove the entire project directory
rm -rf ~/hadoop-docker-lab
```

### Diagnosis when container doesn't stay up after starting. ERROR in Core-Site

The error message is still pointing to the same issue:

*   **Error:** `WstxParsingException: Illegal processing instruction target ("xml")`
*   **File:** `core-site.xml`
*   **Location:** `[row,col,system-id]: [2,5,...]` (Line 2)

This confirms that the `core-site.xml` file inside your `config/hadoop` directory is still malformed. The previous edit with `nano` either didn't save correctly or the file was not fixed as intended.

### The Foolproof Solution: Overwrite the File

Let's not risk another editing mistake. We will use a command that **completely overwrites the file** with the correct content. This will guarantee that it is 100% correct.

**Step 1: Stop the Cluster**
First, make sure everything is stopped.

```bash
# Make sure you are in the ~/hadoop-docker-lab directory
docker compose down
```

**Step 2: Overwrite `core-site.xml` with the Correct Content**
Run this entire block of code in your terminal. It will replace the contents of `core-site.xml` with the correct version.

```bash
cat > config/hadoop/core-site.xml << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://namenode:9000</value>
    </property>
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>root</value>
    </property>
</configuration>
EOF
```

**Step 3: Verify the File is Correct**
Before starting again, let's look at the file's content to be absolutely sure the command worked.

```bash
cat config/hadoop/core-site.xml
```
The output should look exactly like the block of code above.

**Step 4: Restart the Cluster**
Now that we are certain the file is correct, start the cluster.

```bash
docker compose up -d
```

**Step 5: Check the Status**
Wait 15-20 seconds for the services to initialize properly, then check the status.

```bash
docker ps
```

The containers should now be `Up` and remain running. This time, the `namenode` will be able to parse its configuration correctly and will not crash. You can then proceed with the lab.


### **Conclusion**
Congratulations! You have successfully deployed a multi-container Hadoop cluster using Docker, interacted with HDFS, and run MapReduce jobs on a YARN cluster. This lab demonstrates the power of containerization for simplifying the deployment and management of complex big data infrastructure. You've gained practical skills relevant to both modern DevOps practices and foundational big data technologies.
