### **Lab 10: Debugging Docker Containers (Enhanced Version)**

**Objectives**
By the end of this lab, you will be able to:
*   Inspect and analyze Docker container logs using `docker logs`
*   Execute commands inside running containers with `docker exec`
*   Troubleshoot networking issues by examining container network configurations
*   Attach to running containers using `docker attach`
*   Identify and resolve common Docker container issues using built-in debugging tools
*   Apply systematic debugging approaches to containerized applications

**Prerequisites**
Before starting this lab, you should have:
*   Basic understanding of Docker concepts (containers, images, Dockerfile)
*   Familiarity with Linux command line operations
*   Knowledge of basic networking concepts
*   Understanding of log file analysis
*   Previous experience with Docker basic commands (`docker run`, `docker ps`, `docker stop`)

**Lab Environment Setup**
Ready-to-Use Cloud Machines: Al Nafi provides pre-configured Linux-based cloud machines with Docker already installed. Simply click Start Lab to access your environment - no need to build your own VM or install Docker manually.

Your lab environment includes:
*   Ubuntu 20.04 LTS with Docker Engine installed
*   Pre-configured user with Docker permissions
*   Sample applications for debugging exercises
*   Network tools for troubleshooting

---

### **Task 1: Inspect Container Logs with docker logs**

#### **Subtask 1.1: Create a Container with Logging Issues**
First, let's create a container that generates various types of logs to practice debugging.

```bash
echo -e "\n\033[34m---> Step 1: Creating a directory for our test application.\033[0m"
echo "We use 'mkdir' to create a new directory named 'debug-lab' in your home folder (~)."
# Create a directory for our test application
mkdir ~/debug-lab

echo -e "\n\033[34m---> Step 2: Navigating into the new directory.\033[0m"
echo "The 'cd' command changes our current location to the newly created directory."
cd ~/debug-lab

echo -e "\n\033[34m---> Step 3: Creating a Python web application file.\033[0m"
echo "We are using a 'cat' command with a 'here document' (<< 'EOF') to write multiple lines of code directly into a file named 'app.py'."
echo "This app is designed to log different kinds of messages (INFO, WARNING, ERROR) based on the URL you visit."
# Create a simple Python web application with logging
cat > app.py << 'EOF'
import time
import sys
import logging
from http.server import HTTPServer, BaseHTTPRequestHandler

# Configure logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')
logger = logging.getLogger(__name__)

class SimpleHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        logger.info(f"Received GET request for {self.path}")
        
        if self.path == '/':
            self.send_response(200)
            self.send_header('Content-type', 'text/html')
            self.end_headers()
            self.wfile.write(b'<h1>Debug Lab Application</h1>')
        elif self.path == '/error':
            logger.error("Intentional error endpoint accessed")
            self.send_response(500)
            self.end_headers()
            self.wfile.write(b'Internal Server Error')
        elif self.path == '/slow':
            logger.warning("Slow endpoint accessed - simulating delay")
            time.sleep(5)
            self.send_response(200)
            self.end_headers()
            self.wfile.write(b'Slow response completed')
        else:
            logger.warning(f"404 - Path not found: {self.path}")
            self.send_response(404)
            self.end_headers()
            self.wfile.write(b'Not Found')

if __name__ == '__main__':
    server = HTTPServer(('0.0.0.0', 8080), SimpleHandler)
    logger.info("Starting server on port 8080")
    try:
        server.serve_forever()
    except KeyboardInterrupt:
        logger.info("Server stopped")
        server.server_close()
EOF

echo -e "\n\033[34m---> Step 4: Creating a Dockerfile for the application.\033[0m"
echo "This Dockerfile defines the steps to containerize our Python app. It starts from a lean Python image, copies our app code, exposes the port, and defines the startup command."
# Create a Dockerfile for the application
cat > Dockerfile << 'EOF'
FROM python:3.9-slim
WORKDIR /app
COPY app.py .
EXPOSE 8080
CMD ["python", "app.py"]
EOF

echo -e "\n\033[34m---> Step 5: Building the Docker image.\033[0m"
echo "The 'docker build' command creates an image from our Dockerfile."
echo "  -t debug-app: This tags (names) our image 'debug-app' for easy reference."
echo "  .: This tells Docker to use the current directory as the build context."
# Build the image
docker build -t debug-app .

echo -e "\n\033[34m---> Step 6: Running the container from our new image.\033[0m"
echo "The 'docker run' command creates and starts a container."
echo "  -d: Detached mode. The container runs in the background."
echo "  --name debug-container: Assigns a memorable name to our container."
echo "  -p 8080:8080: Port mapping. It connects port 8080 on the host to port 8080 in the container."
echo "  debug-app: The name of the image to use."
# Run the container in detached mode
docker run -d --name debug-container -p 8080:8080 debug-app
```

#### **Subtask 1.2: Basic Log Inspection**

```bash
echo -e "\n\033[34m---> Viewing logs in real-time (like tail -f).\033[0m"
echo "The 'docker logs' command shows a container's output. The '-f' or '--follow' flag streams the logs live."
echo "💡 Tip: Open a NEW terminal to run the 'curl' commands while this one is streaming logs."
# View logs in real-time (similar to tail -f)
docker logs -f debug-container
```

**In a new terminal**, generate some log entries:
```bash
echo -e "\n\033[34m---> Generating log entries by making web requests.\033[0m"
echo "Each 'curl' command sends an HTTP request to our running application, which will generate a log entry that you can see in the other terminal."
# Generate different types of requests to create logs
curl http://localhost:8080/
curl http://localhost:8080/error
curl http://localhost:8080/nonexistent
# The '&' runs this slow command in the background so you can continue working.
curl http://localhost:8080/slow &
```

**Return to your first terminal** and press `Ctrl+C` to stop the log stream.

```bash
echo -e "\n\033[34m---> Viewing logs with timestamps.\033[0m"
echo "The '-t' or '--timestamps' flag adds an RFC3339Nano timestamp to each log line, which is critical for debugging."
# Stop the real-time log viewing (Ctrl+C) and view logs with timestamps
docker logs -t debug-container

echo -e "\n\033[34m---> Viewing only the most recent logs.\033[0m"
echo "The '--tail' flag shows only the last N lines of the logs. Very useful for quickly seeing the latest events."
# View only the last 10 log entries
docker logs --tail 10 debug-container

echo -e "\n\033[34m---> Viewing logs since a certain time.\033[0m"
echo "The '--since' flag allows you to filter logs to a specific timeframe, such as the last 5 minutes ('5m')."
# View logs from the last 5 minutes
docker logs --since 5m debug-container
```

#### **Subtask 1.3: Advanced Log Analysis**

```bash
echo -e "\n\033[34m---> Filtering logs by a specific time range.\033[0m"
echo "You can combine '--since' and '--until' with specific timestamps (YYYY-MM-DDTHH:MM:SS) to isolate logs from a specific incident window."
echo "Note: The following command is an example; it likely won't return logs unless you adjust the date."
# View logs from a specific time (adjust the time to your current time)
docker logs --since "2024-01-01T10:00:00" --until "2024-01-01T11:00:00" debug-container

echo -e "\n\033[34m---> Viewing logs from the last hour.\033[0m"
echo "Using relative times like '1h' (one hour) is very convenient."
# View logs from the last hour
docker logs --since 1h debug-container

echo -e "\n\033[34m---> Searching for specific patterns using 'grep'.\033[0m"
echo "We can pipe the output of 'docker logs' to standard Linux tools like 'grep' for powerful searching."
echo "💡 Tip: '2>&1' redirects the standard error (stderr) stream to standard output (stdout), ensuring 'grep' sees both application logs and application errors."
# Use grep to filter logs for errors
docker logs debug-container 2>&1 | grep -i error

echo -e "\n\033[34m---> Searching for multiple patterns at once.\033[0m"
echo "Here, 'grep' searches for lines containing either '404' or '500'."
# Search for specific HTTP status codes
docker logs debug-container 2>&1 | grep "404\|500"
```

### **Task 2: Use docker exec to Run Commands Inside Containers**

#### **Subtask 2.1: Interactive Container Access**
```bash
echo -e "\n\033[34m---> Getting an interactive shell inside a running container.\033[0m"
echo "'docker exec' runs a new command in a running container."
echo "  -i (interactive): Keeps STDIN open so you can type."
echo "  -t (tty): Allocates a pseudo-terminal for a proper shell experience."
echo "Combining '-it' is the standard way to 'get inside' a container for debugging."
# Execute bash inside the running container
docker exec -it debug-container bash
```

**Once inside the container**, explore the environment with these commands. You will see a new prompt like `root@<container_id>:/app#`.
```bash
# Check the current working directory
pwd

# List files in the application directory
ls -la

# Check running processes
ps aux

# Check network configuration
ip addr show

# Check environment variables
env | grep -E "(PATH|PYTHON|HOME)"

# Exit the container shell and return to your host machine
exit```

#### **Subtask 2.2: Running Specific Commands**
```bash
echo -e "\n\033[34m---> Running single, non-interactive commands.\033[0m"
echo "You don't always need a full shell. You can use 'docker exec' to run a single command and see its output directly. This is great for scripting and quick checks."

echo -e "\n\033[34m1. Check the Python version.\033[0m"
# Check the Python version in the container
docker exec debug-container python --version

echo -e "\n\033[34m2. View the application file's content.\033[0m"
# View the application file
docker exec debug-container cat app.py

echo -e "\n\033[34m3. Check the container's disk usage.\033[0m"
# Check disk usage
docker exec debug-container df -h

echo -e "\n\033[34m4. Check the container's memory usage.\033[0m"
# Check memory usage
docker exec debug-container free -m

echo -e "\n\033[34m---> Using exec for application-specific debugging.\033[0m"

echo -e "\n\033[34m1. Check if the app is listening on the correct port using 'netstat'.\033[0m"
# Check if the application is listening on the correct port
docker exec debug-container netstat -tlnp

echo -e "\n\033[34m2. Test connectivity from *inside* the container to itself.\033[0m"
echo "This helps rule out external network issues and confirms the application is running correctly."
# Test internal connectivity
docker exec debug-container curl http://localhost:8080/

echo -e "\n\033[34m3. Find the process ID of the running application.\033[0m"
# Check application logs from inside the container
docker exec debug-container ps aux | grep python
```

#### **Subtask 2.3: Installing Debug Tools**
```bash
echo -e "\n\033[34m---> Installing temporary debugging tools inside a container.\033[0m"
echo "Production images are often minimal ('slim') and lack common tools. 'docker exec' allows you to install them on-the-fly for a debugging session."
echo "⚠️ Warning: These changes are ephemeral. If the container is restarted, the new tools will be gone. For permanent changes, update the Dockerfile."

echo -e "\n\033[34m1. Get an interactive shell again.\033[0m"
# Enter the container interactively
docker exec -it debug-container bash

# Inside the container, run the following:
echo -e "\n\033[34m2. Update the package manager's list of available packages.\033[0m"
apt-get update

echo -e "\n\033[34m3. Install 'curl', 'wget', 'htop' (a process viewer), and 'strace' (a system call tracer).\033[0m"
apt-get install -y curl wget htop strace

echo -e "\n\033[34m4. Test the newly installed tools.\033[0m"
htop  # Press 'q' to quit
curl http://localhost:8080/

echo -e "\n\033[34m5. Exit the container.\033[0m"
exit
```

### **Task 3: Troubleshoot Networking Issues**

#### **Subtask 3.1: Inspect Container Network Configuration**
```bash
echo -e "\n\033[34m---> Getting detailed information about a container with 'docker inspect'.\033[0m"
echo "'docker inspect' returns a large JSON object containing all configuration details of a container. It's the ultimate source of truth."
# Get detailed information about the container
docker inspect debug-container

echo -e "\n\033[34m---> Focusing on the network configuration section.\033[0m"
echo "We pipe the JSON output to 'grep' to quickly find and display the 'NetworkSettings' block."
# Focus on network configuration
docker inspect debug-container | grep -A 20 "NetworkSettings"

echo -e "\n\033[34m---> Extracting a specific piece of information (the IP address).\033[0m"
echo "The '-f' or '--format' flag lets you use Go template syntax to parse the JSON and pull out exactly what you need."
# Get the container's IP address
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' debug-container

echo -e "\n\033[34m---> Extracting the network name.\033[0m"
# Get network name
docker inspect -f '{{range .NetworkSettings.Networks}}{{.NetworkID}}{{end}}' debug-container
```

#### **Subtask 3.2: Network Connectivity Testing**```bash
echo -e "\n\033[34m---> Creating a second container to act as a 'client' for network testing.\033[0m"
echo "We will use this 'network-test' container to check if it can reach our 'debug-container'."
# Run a simple container for network testing
docker run -d --name network-test alpine sleep 3600

echo -e "\n\033[34m---> Testing connectivity between containers using 'ping'.\033[0m"
echo "We use 'docker exec' to run 'ping' from the 'network-test' container, targeting the IP address of 'debug-container'."
echo "The '$(...)' syntax is a command substitution that gets the IP address dynamically."
# Test connectivity between containers
docker exec network-test ping -c 3 $(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' debug-container)

echo -e "\n\033[34m---> Testing port connectivity with 'curl'.\033[0m"

echo "First, install 'curl' in our test container."
# Install network tools in the test container
docker exec network-test apk add --no-cache curl

echo "Now, test if we can reach the web server on port 8080 from the other container."
# Test HTTP connectivity between containers
docker exec network-test curl http://$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' debug-container):8080/
```

#### **Subtask 3.3: Troubleshoot Common Network Issues**
```bash
echo -e "\n\033[34m---> Simulating a common 'port already in use' error.\033[0m"

echo "First, stop the running container to free up port 8080."
# Stop the current container
docker stop debug-container

echo "Now, try to run a new container on the same port 8080. This should fail if another process (not shown here) were using it, but here we are just setting up."
# Try to run another container on the same port (this will fail)
docker run -d --name debug-container-2 -p 8080:8080 debug-app

echo "Let's check for running containers and processes using port 8080 on the host."
# Check if the port is already in use
docker ps -a
netstat -tlnp | grep 8080

echo -e "\n\033[34m---> The Fix: Run the container on a different host port.\033[0m"
echo "Here, we map host port 8081 to container port 8080 (-p 8081:8080), resolving the conflict."
# Run on a different port
docker run -d --name debug-container-alt -p 8081:8080 debug-app

echo "Test the application on the new host port."
# Test the new port
curl http://localhost:8081/

echo -e "\n\033[34m---> Examining Docker's networking infrastructure.\033[0m"
# List all Docker networks
docker network ls

echo "The 'bridge' network is the default network containers connect to."
# Inspect the default bridge network
docker network inspect bridge

echo -e "\n\033[34m---> Using custom networks for better isolation and service discovery.\033[0m"
echo "It's a best practice to create your own networks for applications."
# Create a custom network for better isolation
docker network create debug-network

echo "Now run a container and attach it to our custom network."
# Run containers on the custom network
docker run -d --name debug-app-custom --network debug-network -p 8082:8080 debug-app
```

### **Task 4: Attach to Running Containers**

#### **Subtask 4.1: Understanding `docker attach`**
```bash
echo -e "\n\033[34m---> Preparing for this task by stopping previous containers.\033[0m"
# Stop previous containers to free up resources
docker stop debug-container debug-container-2 debug-container-alt debug-app-custom network-test

echo -e "\n\033[34m---> Creating a container that continuously prints output.\033[0m"
# Run a container with an interactive process
docker run -d --name interactive-container alpine sh -c "while true; do echo 'Container is running...'; sleep 5; done"

echo -e "\n\033[34m---> Using 'docker attach' to view the container's live output.\033[0m"
echo "'attach' connects your terminal to the container's main running process (its STDIN, STDOUT, and STDERR)."
echo "⚠️ CRITICAL: Pressing Ctrl+C here will send a SIGKILL signal to the main process, which will STOP the container."
# Attach to see the output
docker attach interactive-container
# Note: You'll see the repeating output
# Press Ctrl+C to detach (this will stop the container)
```

#### **Subtask 4.2: Proper Attachment Techniques**
```bash
echo -e "\n\033[34m---> Running a container with a proper interactive shell.\033[0m"
echo "Using '-it' keeps STDIN open and allocates a TTY, and '-d' runs it in the background. This creates a perfect container to attach to."
# Run a container with bash that stays open
docker run -dit --name shell-container ubuntu bash

echo -e "\n\033[34m---> Attaching to the interactive shell.\033[0m"
# Attach to the container
docker attach shell-container

# You now have an interactive shell. Try some commands:
# ls -la
# ps aux
# echo "I'm inside the container"

echo -e "\n\033[32m---> How to DETACH SAFELY: Press Ctrl+P, then Ctrl+Q.\033[0m"
echo "This special key sequence detaches your terminal without stopping the container."
# Detach without stopping the container using Ctrl+P, Ctrl+Q
# (Press Ctrl+P, then Ctrl+Q in sequence)

echo -e "\n\033[34m---> Comparing 'attach' vs 'exec'.\033[0m"
echo "First, verify the container is still running after detaching."
# After detaching, check that the container is still running
docker ps

echo "Now, use 'exec' to get a shell. This creates a NEW, separate 'bash' process inside the container."
# Use exec to enter the same container (recommended approach)
docker exec -it shell-container bash

echo "When you 'exit' this shell, you are only exiting the new process. The original container process remains running."
# This creates a new process, so exiting won't stop the container
exit

echo "Verify the container is still running. 'exec' is much safer for debugging!"
# Verify the container is still running
docker ps
```

### **Task 5: Identify and Fix Common Issues**

#### **Subtask 5.1: Debugging Container Startup Issues**
```bash
echo -e "\n\033[34m---> Simulating a broken container build.\033[0m"
echo "This Dockerfile tries to COPY a file that doesn't exist, which will cause the build to fail."
# Create a Dockerfile with issues
cat > Dockerfile.broken << 'EOF'
FROM python:3.9-slim
WORKDIR /app
COPY nonexistent-file.py .
CMD ["python", "nonexistent-file.py"]
EOF

echo -e "\n\033[34m---> Attempting to build and run the broken image.\033[0m"
echo "The build step itself will fail, but if it had succeeded and only the CMD was wrong, the 'docker run' would fail."
# Try to build and run (this will fail)
docker build -f Dockerfile.broken -t broken-app .
# The following command will fail because the image build failed.
# If the issue was in the CMD, the container would be created but exit immediately.
docker run --name broken-container broken-app

echo -e "\n\033[34m---> How to debug a container that exits immediately.\033[0m"
echo "💡 The Key Steps for 'container won't start' issues:"

echo "1. Check the status of ALL containers, including stopped ones. Look for 'Exited' status."
# Check container status
docker ps -a

echo "2. Use 'docker logs' on the *stopped* container. The error message is almost always here."
# Examine the logs to see what went wrong
docker logs broken-container

echo "3. Use 'docker inspect' to check for misconfigurations (e.g., wrong command, entrypoint)."
# Get detailed information about the failed container
docker inspect broken-container
```

#### **Subtask 5.2: Debugging Resource Issues**
```bash
echo -e "\n\033[34m---> Creating a container designed to consume memory.\033[0m"
echo "The '--memory=50m' flag limits this container to 50 megabytes of RAM. The python script will continuously allocate memory until it hits this limit and is terminated by the Docker daemon (OOMKilled - Out Of Memory)."
# Run a container with limited memory
docker run -d --name memory-limited --memory=50m python:3.9-slim python -c "
import time
data = []
while True:
    data.append('x' * 1024 * 1024)  # Allocate 1MB
    time.sleep(1)
    print(f'Allocated {len(data)} MB')
"

echo -e "\n\033[34m---> Monitoring and debugging resource usage.\033[0m"

echo "1. Use 'docker stats' to see live CPU, Memory, and Network I/O for containers."
# Monitor container resource usage
docker stats memory-limited --no-stream

echo "2. Check the logs. You might see messages about memory allocation before it dies."
# Check container logs for memory issues
docker logs memory-limited

echo "3. Inspect the container to see its resource limits and state. After it's OOMKilled, you can see that in the 'State' section."
# Get detailed resource information
docker inspect memory-limited | grep -A 10 "Memory"
```

#### **Subtask 5.3: Debugging Application Issues**
```bash
echo -e "\n\033[34m---> Creating an application with unpredictable runtime errors.\033[0m"
# Create an application with runtime errors
cat > debug-app.py << 'EOF'
import time
import random
import sys

def problematic_function():
    # Simulate various issues
    issue = random.choice(['memory', 'exception', 'slow', 'success'])
    
    if issue == 'memory':
        print("WARNING: High memory usage detected")
        data = [0] * 1000000  # Allocate memory
        time.sleep(2)
    elif issue == 'exception':
        print("ERROR: About to raise an exception")
        raise Exception("Simulated application error")
    elif issue == 'slow':
        print("INFO: Processing slow operation")
        time.sleep(10)
    else:
        print("INFO: Operation completed successfully")

if __name__ == "__main__":
    print("Starting problematic application...")
    while True:
        try:
            problematic_function()
            time.sleep(3)
        except Exception as e:
            print(f"EXCEPTION: {e}")
            time.sleep(5)
EOF

# Create Dockerfile for the problematic app
cat > Dockerfile.debug << 'EOF'
FROM python:3.9-slim
WORKDIR /app
COPY debug-app.py .
CMD ["python", "debug-app.py"]
EOF

echo -e "\n\033[34m---> Building and running the problematic application.\033[0m"
# Build and run
docker build -f Dockerfile.debug -t debug-problematic .
docker run -d --name problematic-app debug-problematic

echo -e "\n\033[34m---> Debugging the running application.\033[0m"

echo "1. Monitor logs in real-time to see the random errors and messages as they happen."
# Monitor logs in real-time
docker logs -f problematic-app &

echo "2. In another terminal (or here after a moment), 'exec' into the container to inspect its state while the app is running."
# In another terminal, examine the container
docker exec -it problematic-app bash

# Inside the container, you can check:
echo "Check running processes."
ps aux
echo "Check system resources."
free -m
df -h
# Exit the container when done
exit

echo "Stop the background log monitoring by finding its process ID and using 'kill' or by pressing Ctrl+C in the relevant terminal."
# Stop the log monitoring (Ctrl+C in the first terminal)
```

#### **Subtask 5.4: Using Docker System Commands for Debugging**
```bash
echo -e "\n\033[34m---> Checking the overall health of the Docker daemon.\033[0m"
# Get system-wide information
docker system info

echo -e "\n\033[34m---> Checking Docker's disk usage.\033[0m"
echo "'docker system df' shows how much space is being used by images, containers, and volumes. It's essential for preventing 'disk full' errors."
# Check disk usage
docker system df

echo "The '-v' (verbose) flag gives a more detailed breakdown."
# Show detailed disk usage
docker system df -v

echo -e "\n\033[34m---> Cleaning up unused Docker objects to reclaim disk space.\033[0m"
echo "'docker system prune' is a powerful command to remove all dangling images, stopped containers, and unused networks."
# Remove unused containers, networks, images
docker system prune

echo "To remove only stopped containers:"
# Remove all stopped containers
docker container prune

echo "To remove only dangling (un-tagged) images:"
# Remove unused images
docker image prune
```

### **Troubleshooting Common Issues**

*   **Issue 1: Container Won't Start**
    *   **Symptoms**: Container exits immediately after starting
    *   **Debug Steps**:
        1.  `docker ps -a` (Check exit code and status)
        2.  `docker logs <container_name>` (Examine logs for error messages)
        3.  `docker inspect <container_name>` (Inspect container configuration)

*   **Issue 2: Cannot Connect to Application**
    *   **Symptoms**: Application not accessible from host
    *   **Debug Steps**:
        1.  `docker port <container_name>` (Check port bindings)
        2.  `docker exec <container_name> netstat -tlnp` (Verify application is listening inside container)
        3.  `docker exec <container_name> curl http://localhost:<port>` (Test internal connectivity)

*   **Issue 3: High Resource Usage**
    *   **Symptoms**: Container consuming too much CPU/memory
    *   **Debug Steps**:
        1.  `docker stats <container_name>` (Monitor real-time resource usage)
        2.  `docker exec <container_name> ps aux` (Check processes inside container)
        3.  `docker exec <container_name> free -m` and `docker exec <container_name> df -h` (Examine system resources)

### **Lab Cleanup**
Before finishing the lab, clean up the created resources.
```bash
echo -e "\n\033[34m---> Cleaning up all lab resources.\033[0m"

echo "1. Stopping all currently running containers."
echo "The command '$(docker ps -q)' lists the IDs of all running containers, which are then passed to 'docker stop'."
# Stop all running containers
docker stop $(docker ps -q)

echo "2. Removing all containers (both stopped and running)."
echo "'docker ps -aq' lists the IDs of ALL containers."
# Remove all containers
docker rm $(docker ps -aq)

echo "3. Removing the images we built during the lab."
# Remove created images
docker rmi debug-app broken-app debug-problematic

echo "4. Removing the custom network we created."
# Remove custom network
docker network rm debug-network

echo "5. Deleting the lab directory and its contents."
# Clean up files
rm -rf ~/debug-lab

echo -e "\n\033[32mLab cleanup complete!\033[0m"
```

### **Conclusion**
In this lab, you have successfully learned essential Docker debugging techniques that are crucial for maintaining containerized applications in production environments. You have mastered:

*   **Log Analysis**: Using `docker logs` with various options to inspect container output, filter by time, and identify issues through log patterns
*   **Container Interaction**: Leveraging `docker exec` to run commands inside containers, install debugging tools, and investigate runtime issues
*   **Network Troubleshooting**: Examining container network configurations, testing connectivity between containers, and resolving common networking problems
*   **Container Attachment**: Understanding when and how to use `docker attach` versus `docker exec` for different debugging scenarios
*   **Issue Resolution**: Identifying and fixing common container problems including startup failures, resource constraints, and application errors

These debugging skills are fundamental for the Docker Certified Associate (DCA) certification and essential for anyone working with containerized applications in development, testing, or production environments. The systematic approach to debugging you've learned will help you quickly identify and resolve issues, minimizing downtime and improving application reliability.

The techniques covered in this lab form the foundation of container troubleshooting and will serve you well as you advance to more complex Docker deployments and orchestration platforms like Kubernetes.
