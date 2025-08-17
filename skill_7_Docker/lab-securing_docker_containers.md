### **Lab: Securing Docker Containers** 🔒

-----

### **Objectives**

  * Implement **non-root users** to reduce the attack surface.
  * Use **Docker Content Trust (DCT)** for image integrity and authenticity.
  * Scan images for vulnerabilities with **Docker Scout** and other tools.
  * Limit container privileges by dropping **capabilities**.
  * Configure containers with a **read-only file system**.

-----

### **Prerequisites**

  * Docker Desktop or a Linux machine with Docker Engine installed.
  * A basic understanding of Docker concepts (images, containers, Dockerfile).
  * Familiarity with the command line and basic Linux user/file permissions.

-----

### **Task 1: Run Containers as Non-Root Users**

Running a container with the default **`root`** user is a major security risk. If an attacker compromises your container, they get root access inside it, which could lead to privilege escalation on the host system. The **`USER`** directive in a Dockerfile is the key to preventing this.

#### **Step 1.1: Create a secure Dockerfile**

We'll build a simple Python Flask application that confirms which user it's running as.

1.  Create a project directory and navigate into it.

    ```bash
    mkdir secure-app && cd secure-app
    ```

2.  Create the application and requirements files.

    ```bash
    # Create the Flask application file.
    cat > app.py << 'EOF'
    from flask import Flask
    import os

    app = Flask(__name__)

    @app.route('/')
    def hello():
        # os.getuid() returns the current user's ID.
        return f"Hello from secure container! Running as user: {os.getuid()}"

    if __name__ == '__main__':
        app.run(host='0.0.0.0', port=8080)
    EOF

    # Create the requirements file for dependencies.
    cat > requirements.txt << 'EOF'
    Flask==2.3.3
    EOF
    ```

3.  Create the **`Dockerfile`** with a non-root user.

    ```bash
    cat > Dockerfile << 'EOF'
    # Use a secure, lightweight base image.
    FROM python:3.9-slim

    # Create a non-root user and group. The -r flag creates a system user.
    # Concept: Principle of Least Privilege. We create a dedicated user for the app.
    RUN groupadd -r appuser && useradd -r -g appuser appuser

    # Set the working directory for the application.
    # Concept: Consistency and organization. All subsequent commands will be relative to this path.
    WORKDIR /app

    # Copy and install dependencies as root to avoid permission issues.
    # Concept: Multi-stage build optimization (implied). The root-level commands are for setup.
    COPY requirements.txt .
    RUN pip install --no-cache-dir -r requirements.txt

    # Copy the application code.
    COPY app.py .

    # Change ownership of the application directory to our new user.
    # This is crucial so the non-root user can read/write files.
    RUN chown -R appuser:appuser /app

    # Switch to the non-root user. ALL subsequent commands in this Dockerfile
    # and the running container will execute as 'appuser'.
    # Concept: The USER directive is the most important part of this subtask.
    USER appuser

    # Expose the application port.
    EXPOSE 8080

    # Command to run the application.
    CMD ["python", "app.py"]
    EOF
    ```

#### **Step 1.2: Build and Test the Secure Image**

Now, build the image and run a container to verify the non-root user is active.

1.  Build the Docker image with a tag.

    ```bash
    docker build -t secure-app:v1 .
    ```

      * `docker build`: Builds an image from the `Dockerfile`.
      * `-t secure-app:v1`: Tags the image with a name and version.
      * `.`: Specifies the **build context**, meaning Docker should look in the current directory for the `Dockerfile` and other files.

2.  Run the container and check its status.

    ```bash
    # The -d flag runs the container in 'detached' (background) mode.
    # The -p flag maps host port 8080 to container port 8080.
    # The --name flag gives it a readable name.
    docker run -d -p 8080:8080 --name secure-app-container secure-app:v1

    # Use 'docker ps' to see if the container is running.
    # Concept: 'ps' is short for 'process status'.
    docker ps
    ```

3.  Verify the user inside the container.

    ```bash
    # `docker exec` runs a command inside a running container.
    # `id` shows user and group information.
    docker exec secure-app-container id

    # Expected output: uid=999(appuser) gid=999(appuser) ...
    # A UID of 0 would indicate the root user.
    ```

4.  Access the application via `curl` to confirm it's working and showing the correct user ID.

    ```bash
    curl http://localhost:8080
    ```

      * The output should display the `uid` (User ID), which should not be `0`.

5.  Clean up the container.

    ```bash
    # Always stop and remove containers after use.
    # Use `docker stop` to send a SIGTERM signal, and `docker rm` to delete it.
    docker stop secure-app-container
    docker rm secure-app-container
    ```

-----

### **Task 2: Implement Docker Content Trust (DCT)**

**Docker Content Trust** uses digital signatures to ensure that the images you pull are authentic and untampered with. It's a key part of securing your supply chain.

#### **Step 2.1: Enable and Test Content Trust**

1.  Enable DCT by setting an environment variable.

    ```bash
    # The DOCKER_CONTENT_TRUST variable must be set to '1' to enforce trust.
    export DOCKER_CONTENT_TRUST=1
    ```

2.  Attempt to pull an unsigned public image. This command should fail.

    ```bash
    docker pull alpine:latest
    ```

      * **Concept:** This fails because Docker's default `alpine` image is not signed by a key that your system trusts. This demonstrates that DCT is working.

3.  Disable DCT to allow the pull.

    ```bash
    export DOCKER_CONTENT_TRUST=0
    docker pull alpine:latest
    export DOCKER_CONTENT_TRUST=1 # Re-enable for the next step.
    ```

#### **Step 2.2: Sign and Push Your Image**

You must push a signed image to a registry. We'll use a local registry for this lab.

1.  Start a local Docker registry.

    ```bash
    # Run the official 'registry' image.
    docker run -d -p 5000:5000 --name local-registry registry:2
    ```

2.  Tag your secure image with the registry's address.

    ```bash
    # Tagging the image to include the registry URL (`localhost:5000`).
    docker tag secure-app:v1 localhost:5000/secure-app:signed
    ```

3.  Push the tagged image. This process will trigger a prompt to create new signing keys.

    ```bash
    # Pushing the image to the local registry.
    # Docker Content Trust automatically handles key generation and signing.
    docker push localhost:5000/secure-app:signed
    ```

4.  Verify the image's signature.

    ```bash
    # The `docker trust inspect` command shows who signed an image.
    docker trust inspect localhost:5000/secure-app:signed
    ```

      * **Concept:** This command confirms that the image has been signed and is trustworthy.

5.  Clean up the registry container.

    ```bash
    docker stop local-registry
    docker rm local-registry
    ```

-----

### **Task 3: Security Scanning with Docker Scout**

**Docker Scout** is a powerful tool for finding vulnerabilities (**CVEs**) in your container images. Regularly scanning images is a critical step in a secure **CI/CD pipeline**.

#### **Step 3.1: Scan Your Image**

1.  Scan your `secure-app:v1` image for vulnerabilities.

    ```bash
    # `docker scout cves` scans the image for Common Vulnerabilities and Exposures (CVEs).
    docker scout cves secure-app:v1
    ```

      * The output will show a list of vulnerabilities and their severity levels.

2.  Scan the base image to see its inherent vulnerabilities.

    ```bash
    docker scout cves python:3.9-slim
    ```

      * **Concept:** This helps you understand which vulnerabilities come from your application's dependencies versus the base image itself.

3.  (Optional) Scan with `trivy`, another popular security scanner.

    ```bash
    # Trivy is a powerful, open-source vulnerability scanner.
    # It requires mounting the Docker socket to interact with images.
    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
      aquasec/trivy image secure-app:v1
    ```

-----

### **Task 4: Limit Container Privileges**

By default, Docker containers have a large set of Linux **capabilities**. You can drop unnecessary capabilities to limit what a container can do on the host, reducing the potential damage from a compromise.

#### **Step 4.1: Drop Capabilities**

1.  Create and run a new container with limited capabilities. We'll use the `-it` flags to get an interactive shell.

    ```bash
    # The --cap-drop=ALL option removes all default capabilities.
    # The --cap-add=NET_BIND_SERVICE adds back the specific capability needed to bind to a low port (e.g., < 1024).
    # Since our app uses port 8080, this isn't strictly necessary, but it's a good example.
    docker run --rm -it --name limited-caps --cap-drop=ALL --cap-add=NET_BIND_SERVICE alpine:latest /bin/sh
    ```

2.  Once inside the container's shell, install `libcap` and check the capabilities.

    ```bash
    # Inside the container shell:
    apk add --no-cache libcap
    capsh --print
    ```

      * **Concept:** The `capsh --print` command displays the capabilities of the current process. You should see a much smaller list than a default container.

3.  Exit the container.

    ```bash
    exit
    ```

-----

### **Task 5: Use Read-Only File Systems**

A **read-only file system** prevents a container from writing to any directory other than a temporary file system (`tmpfs`). This significantly limits the ability of an attacker to persist malicious changes.

#### **Step 5.1: Run a Read-Only Container**

1.  Run your existing `secure-app:v1` image with the `--read-only` flag.

    ```bash
    # The --read-only flag mounts the container's root file system as read-only.
    # The --tmpfs flag mounts a temporary, in-memory file system at `/tmp`.
    # This allows the application to write temporary files without compromising security.
    docker run -d --name readonly-container --read-only --tmpfs /tmp:rw,noexec,nosuid,size=100m -p 8081:8080 secure-app:v1
    ```

2.  Try to create a file inside the container's main file system.

    ```bash
    # This command attempts to create a file at the root level.
    # It should fail with a "Read-only file system" error.
    docker exec readonly-container touch /app/test.txt
    ```

3.  Now, try to create a file in the temporary file system.

    ```bash
    # This command should succeed.
    docker exec readonly-container touch /tmp/test.txt
    ```

4.  Check the file system.

    ```bash
    # The `df -h` command shows disk usage and mount points.
    # You should see `/tmp` mounted as a `tmpfs`.
    docker exec readonly-container df -h
    ```

5.  Clean up.

    ```bash
    docker stop readonly-container
    docker rm readonly-container
    ```

-----

### **Task 6: Comprehensive Security with Docker Compose**

**Docker Compose** is an excellent tool for defining and running multi-container applications. It simplifies applying all of these security measures in a single configuration file.

#### **Step 6.1: Create a Secure `docker-compose.yml`**

Create a `docker-compose.yml` file to combine all the security best practices.

```bash
# Create the docker-compose file.
cat > docker-compose.secure.yml << 'EOF'
version: '3.8'

services:
  secure-web-app:
    # Use your previously built image.
    image: secure-app:v1
    ports:
      - "8084:8080"
    
    # Configure security options.
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    security_opt:
      - no-new-privileges:true
      - apparmor:docker-default
    
    # Ensure the container runs as the non-root user.
    user: "appuser"
    
    # Add a health check to verify the application is running.
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/"]
      interval: 30s
      timeout: 10s
      retries: 3
EOF
```

#### **Step 6.2: Deploy and Verify**

1.  Deploy the application using Docker Compose.

    ```bash
    # `docker-compose up -d` builds and runs the services in the background.
    docker-compose -f docker-compose.secure.yml up -d
    ```

2.  Verify the container's security settings.

    ```bash
    # Use `docker-compose ps` to see the running container.
    docker-compose -f docker-compose.secure.yml ps

    # The `docker inspect` command provides a detailed JSON report of the container.
    # We can filter for security-related settings.
    docker inspect $(docker-compose -f docker-compose.secure.yml ps -q) | grep -A 20 "SecurityOpt"
    ```

      * This command will confirm that all the security options (read-only, capabilities, etc.) have been applied correctly.

-----

### **Lab Cleanup**

It's good practice to clean up all resources created during a lab.

1.  Stop and remove the Docker Compose services.

    ```bash
    docker-compose -f docker-compose.secure.yml down
    ```

2.  Remove all remaining containers and images.

    ```bash
    # `docker ps -aq` lists all container IDs (all, quiet).
    docker stop $(docker ps -aq) 2>/dev/null || true
    docker rm $(docker ps -aq) 2>/dev/null || true

    # `docker images -q` lists all image IDs.
    docker rmi $(docker images -q) 2>/dev/null || true
    ```

      * **Note:** The `2>/dev/null || true` part of the command prevents it from showing errors if no containers or images are found. It's a robust way to ensure cleanup.

-----

### **Conclusion: Key Takeaways**

By completing this lab, you've implemented a **defense-in-depth** strategy for container security.  You didn't rely on a single solution but instead created multiple layers of protection.

  * **Non-Root User:** Limits what a compromised process can do.
  * **Content Trust:** Guarantees the image's authenticity from the source.
  * **Vulnerability Scanning:** Proactively identifies and fixes known weaknesses.
  * **Capability Dropping:** Shrinks the attack surface by removing unnecessary privileges.
  * **Read-Only File System:** Prevents attackers from writing and persisting malicious code.

These practices are not just for labs; they are essential for deploying secure, production-ready containerized applications in the real world.
