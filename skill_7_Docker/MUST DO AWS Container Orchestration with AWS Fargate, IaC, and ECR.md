### **Lab 29: Modern Container Orchestration with AWS Fargate, IaC, and ECR**

Welcome to the definitive guide to running containerized applications on AWS. This lab has been re-architected to teach you the modern, secure, and automated way to use Amazon ECS, moving beyond manual console operations to embrace Infrastructure as Code (IaC) and serverless container execution with AWS Fargate.

#### **Objectives**

By the end of this lab, you will not just know *how* to run containers on AWS, but *why* we use modern best practices to do so. You will be able to:

*   **Provision a Secure Network Infrastructure** using an AWS CloudFormation template.
*   **Build a Custom Docker Image** for a real application and store it in a private Amazon ECR repository.
*   **Define and Deploy a Serverless Container Application** using AWS Fargate, eliminating the need to manage EC2 instances.
*   **Securely Expose Your Application** to the internet using an Application Load Balancer with traffic flowing from public to private subnets.
*   **Implement Robust Auto Scaling** to handle variable workloads and optimize costs.
*   **Centralize Logging and Monitoring** using Amazon CloudWatch for effective troubleshooting.
*   **Understand the Fundamentals of CI/CD** for automated container deployments (Optional Module).

#### **Prerequisites**

*   An AWS account with Administrator-level permissions.
*   AWS CLI installed and configured locally.
*   Docker installed and running on your local machine.
*   A basic understanding of Docker concepts (Dockerfile, build, push).
*   Familiarity with the command line.

---

### **Part A: The Modern Serverless Approach with AWS Fargate and IaC**

In this part, we will deploy our application using AWS Fargate, the serverless, pay-as-you-go compute engine. This is the recommended approach for the vast majority of container workloads on AWS.

---

### **Module 1: Setting Up the Foundation with Infrastructure as Code (IaC)**

We will begin by deploying a secure and well-architected foundation using AWS CloudFormation. This template will create our VPC, subnets, security groups, and an empty ECS cluster.

**Subtask 1.1: Prepare the CloudFormation Template**

1.  Create a new file on your local machine named `ecs-foundation.yaml`.
2.  Copy and paste the entire YAML code block below into this file.

    ```yaml
    AWSTemplateFormatVersion: '2010-09-09'
    Description: >
      Creates a foundational network and ECS cluster for a modern Fargate application.
      Includes a VPC, public/private subnets, security groups, and an ECR repository.

    Resources:
      #---------------------------------------------------------------------------
      # VPC and Networking
      #---------------------------------------------------------------------------
      VPC:
        Type: AWS::EC2::VPC
        Properties:
          CidrBlock: 10.0.0.0/16
          EnableDnsSupport: true
          EnableDnsHostnames: true
          Tags:
            - Key: Name
              Value: ECS-Lab-VPC

      InternetGateway:
        Type: AWS::EC2::InternetGateway

      VPCGatewayAttachment:
        Type: AWS::EC2::VPCGatewayAttachment
        Properties:
          VpcId: !Ref VPC
          InternetGatewayId: !Ref InternetGateway

      PublicSubnetA:
        Type: AWS::EC2::Subnet
        Properties:
          VpcId: !Ref VPC
          CidrBlock: 10.0.1.0/24
          AvailabilityZone: !Select [0, !GetAZs '']
          MapPublicIpOnLaunch: true
          Tags:
            - Key: Name
              Value: ECS-Public-A

      PublicSubnetB:
        Type: AWS::EC2::Subnet
        Properties:
          VpcId: !Ref VPC
          CidrBlock: 10.0.2.0/24
          AvailabilityZone: !Select [1, !GetAZs '']
          MapPublicIpOnLaunch: true
          Tags:
            - Key: Name
              Value: ECS-Public-B

      PrivateSubnetA:
        Type: AWS::EC2::Subnet
        Properties:
          VpcId: !Ref VPC
          CidrBlock: 10.0.3.0/24
          AvailabilityZone: !Select [0, !GetAZs '']
          Tags:
            - Key: Name
              Value: ECS-Private-A

      PrivateSubnetB:
        Type: AWS::EC2::Subnet
        Properties:
          VpcId: !Ref VPC
          CidrBlock: 10.0.4.0/24
          AvailabilityZone: !Select [1, !GetAZs '']
          Tags:
            - Key: Name
              Value: ECS-Private-B

      PublicRouteTable:
        Type: AWS::EC2::RouteTable
        Properties:
          VpcId: !Ref VPC

      PublicRoute:
        Type: AWS::EC2::Route
        DependsOn: VPCGatewayAttachment
        Properties:
          RouteTableId: !Ref PublicRouteTable
          DestinationCidrBlock: 0.0.0.0/0
          GatewayId: !Ref InternetGateway

      PublicSubnetARouteTableAssociation:
        Type: AWS::EC2::SubnetRouteTableAssociation
        Properties:
          SubnetId: !Ref PublicSubnetA
          RouteTableId: !Ref PublicRouteTable

      PublicSubnetBRouteTableAssociation:
        Type: AWS::EC2::SubnetRouteTableAssociation
        Properties:
          SubnetId: !Ref PublicSubnetB
          RouteTableId: !Ref PublicRouteTable

      EIP:
        Type: AWS::EC2::EIP
        Properties:
          Domain: vpc

      NATGateway:
        Type: AWS::EC2::NatGateway
        Properties:
          AllocationId: !GetAtt EIP.AllocationId
          SubnetId: !Ref PublicSubnetA

      PrivateRouteTable:
        Type: AWS::EC2::RouteTable
        Properties:
          VpcId: !Ref VPC

      PrivateRoute:
        Type: AWS::EC2::Route
        Properties:
          RouteTableId: !Ref PrivateRouteTable
          DestinationCidrBlock: 0.0.0.0/0
          NatGatewayId: !Ref NATGateway

      PrivateSubnetARouteTableAssociation:
        Type: AWS::EC2::SubnetRouteTableAssociation
        Properties:
          SubnetId: !Ref PrivateSubnetA
          RouteTableId: !Ref PrivateRouteTable

      PrivateSubnetBRouteTableAssociation:
        Type: AWS::EC2::SubnetRouteTableAssociation
        Properties:
          SubnetId: !Ref PrivateSubnetB
          RouteTableId: !Ref PrivateRouteTable

      #---------------------------------------------------------------------------
      # Security Groups
      #---------------------------------------------------------------------------
      LoadBalancerSecurityGroup:
        Type: AWS::EC2::SecurityGroup
        Properties:
          GroupDescription: Allow HTTP traffic to the Load Balancer
          VpcId: !Ref VPC
          SecurityGroupIngress:
            - IpProtocol: tcp
              FromPort: 80
              ToPort: 80
              CidrIp: 0.0.0.0/0
          Tags:
            - Key: Name
              Value: ECS-ALB-SG

      ContainerSecurityGroup:
        Type: AWS::EC2::SecurityGroup
        Properties:
          GroupDescription: Allow traffic from the Load Balancer to the containers
          VpcId: !Ref VPC
          SecurityGroupIngress:
            - IpProtocol: tcp
              FromPort: 8080 # App port
              ToPort: 8080
              SourceSecurityGroupId: !Ref LoadBalancerSecurityGroup
          Tags:
            - Key: Name
              Value: ECS-Container-SG

      #---------------------------------------------------------------------------
      # ECS Cluster and ECR Repository
      #---------------------------------------------------------------------------
      ECSCluster:
        Type: AWS::ECS::Cluster
        Properties:
          ClusterName: modern-ecs-cluster

      ECRRepository:
        Type: AWS::ECR::Repository
        Properties:
          RepositoryName: ecs-lab-app
          ImageScanningConfiguration:
            ScanOnPush: true

    Outputs:
      VPCId:
        Description: The ID of the VPC
        Value: !Ref VPC
      PublicSubnetA:
        Description: The ID of Public Subnet A
        Value: !Ref PublicSubnetA
      PublicSubnetB:
        Description: The ID of Public Subnet B
        Value: !Ref PublicSubnetB
      PrivateSubnetA:
        Description: The ID of Private Subnet A
        Value: !Ref PrivateSubnetA
      PrivateSubnetB:
        Description: The ID of Private Subnet B
        Value: !Ref PrivateSubnetB
      LoadBalancerSecurityGroup:
        Description: The Security Group for the ALB
        Value: !Ref LoadBalancerSecurityGroup
      ContainerSecurityGroup:
        Description: The Security Group for the ECS Tasks
        Value: !Ref ContainerSecurityGroup
      ECSClusterName:
        Description: The name of the ECS Cluster
        Value: !Ref ECSCluster
      ECRRepositoryUri:
        Description: The URI of the ECR Repository
        Value: !GetAtt ECRRepository.RepositoryUri
    ```

**Subtask 1.2: Deploy the Stack**

1.  Open your terminal.
2.  Navigate to the directory where you saved `ecs-foundation.yaml`.
3.  Run the following AWS CLI command to deploy the stack. Replace `your-region` with your desired AWS region (e.g., `us-east-1`).

    ```bash
    aws cloudformation create-stack \
      --stack-name ecs-lab-foundation \
      --template-body file://ecs-foundation.yaml \
      --capabilities CAPABILITY_IAM \
      --region your-region
    ```

4.  Monitor the stack creation in the AWS Management Console under **CloudFormation**. Wait for the status to become **CREATE_COMPLETE**. This will take 5-10 minutes.
5.  Once complete, go to the **Outputs** tab of the stack and keep this browser tab open. We will need these values later.

> **Why are we doing this?** Using IaC (CloudFormation) ensures your environment is repeatable, version-controlled, and follows best practices. We created a VPC with public subnets for the load balancer and private subnets for our containers, which is a standard secure network design.

---

### **Module 2: Building and Pushing a Container Image to Amazon ECR**

Now we will create a simple Python web application, containerize it with Docker, and push the image to our private ECR repository.

**Subtask 2.1: Create the Application Files**

1.  Create a new directory on your local machine named `ecs-lab-app`.
2.  Inside this directory, create a file named `app.py`:

    ```python
    from flask import Flask
    import os

    app = Flask(__name__)

    @app.route('/')
    def hello_world():
        # Get the ECS Task ID from the container metadata
        task_id = os.environ.get('ECS_CONTAINER_METADATA_URI_V4', 'Not running on ECS').split('/')[-1]
        return f'<h1>Hello from ECS!</h1><p>This response is from Task ID: {task_id}</p>'

    if __name__ == '__main__':
        app.run(host='0.0.0.0', port=8080)
    ```

3.  In the same directory, create a file named `requirements.txt`:

    ```
    Flask==2.2.2
    ```

4.  Finally, create a file named `Dockerfile` in the same directory:

    ```dockerfile
    # Use an official Python runtime as a parent image
    FROM python:3.9-slim

    # Set the working directory in the container
    WORKDIR /app

    # Copy the requirements file into the container
    COPY requirements.txt .

    # Install any needed packages specified in requirements.txt
    RUN pip install --no-cache-dir -r requirements.txt

    # Copy the rest of the application code
    COPY . .

    # Make port 8080 available to the world outside this container
    EXPOSE 8080

    # Define environment variable
    ENV NAME World

    # Run app.py when the container launches
    CMD ["python", "app.py"]
    ```

**Subtask 2.2: Build and Push the Docker Image**

1.  Open your terminal and navigate into the `ecs-lab-app` directory.
2.  Retrieve the ECR Repository URI from your CloudFormation stack's **Outputs** tab.
3.  Authenticate Docker with ECR. Replace `your-region` and `YOUR_AWS_ACCOUNT_ID` and `ecs-lab-app` with your details.

    ```bash
    aws ecr get-login-password --region your-region | docker login --username AWS --password-stdin YOUR_AWS_ACCOUNT_ID.dkr.ecr.your-region.amazonaws.com
    ```

4.  Build the Docker image. Replace the URI with your ECR repository URI.

    ```bash
    docker build -t ecs-lab-app .
    ```

5.  Tag the image for ECR:

    ```bash
    docker tag ecs-lab-app:latest YOUR_ECR_REPOSITORY_URI:latest
    ```

6.  Push the image to ECR:

    ```bash
    docker push YOUR_ECR_REPOSITORY_URI:latest
    ```

7.  Verify the image is in ECR by navigating to the **Elastic Container Registry** console.

> **Why are we doing this?** Storing images in a private registry like ECR is a security best practice. It gives you control over your image assets and allows you to scan them for vulnerabilities.

---

### **Module 3: Defining and Deploying the Application with a Fargate Task Definition**

A Task Definition is the blueprint for our application. It specifies the Docker image to use, CPU/memory allocation, logging configuration, and IAM roles.

**Subtask 3.1: Create IAM Roles**

The Fargate task needs two IAM roles:
*   **Task Execution Role**: Allows ECS to pull the image from ECR and send logs to CloudWatch.
*   **Task Role**: Allows the application inside the container to interact with other AWS services (not needed for this simple app, but a best practice to create).

We will create the required Task Execution Role.
1. Go to the IAM console.
2. Go to **Roles** -> **Create Role**.
3. Select **AWS Service** and for use case, choose **Elastic Container Service**.
4. Select the **Elastic Container Service Task** use case and click **Next**.
5. The `AmazonECSTaskExecutionRolePolicy` will be attached. Click **Next**.
6. Name the role `ecsTaskExecutionRole` and click **Create Role**.

**Subtask 3.2: Create the Task Definition**

1.  Navigate to the **Elastic Container Service** console.
2.  Click **Task Definitions** in the left panel, then **Create new Task Definition**.
3.  **Task Definition Name**: `my-flask-app-task`
4.  **Launch type**: **AWS Fargate**.
5.  **Operating system/Architecture**: Linux/x86_64.
6.  **Task size**:
    *   **Task CPU (unit)**: `.25 vCPU`
    *   **Task memory (MiB)**: `0.5GB`
7.  **Task execution role**: Select the `ecsTaskExecutionRole` you just created.
8.  In the **Container details** section:
    *   **Container name**: `flask-app-container`
    *   **Image URI**: Paste your ECR Repository URI from the CloudFormation outputs (e.g., `123456789012.dkr.ecr.us-east-1.amazonaws.com/ecs-lab-app:latest`).
    *   **Port mappings**:
        *   **Container port**: `8080`
        *   **Protocol**: `TCP`
9.  Expand the **Logging** section under **Log collection**.
10. Check **Use log collection** and ensure **Amazon CloudWatch** is selected. The log group will be created automatically.
11. Click **Create**.

> **Why are we doing this?** The Task Definition decouples your application's configuration from the underlying infrastructure. By specifying resource limits, you ensure predictable performance and cost management.

---

### **Module 4: Creating a Secure ECS Service with an Application Load Balancer**

An ECS Service is responsible for running and maintaining a specified number of instances of a task definition simultaneously.

**Subtask 4.1: Create the Application Load Balancer and Target Group**

1.  Navigate to the **EC2 Console**.
2.  Click **Load Balancers** -> **Create Load Balancer**.
3.  Choose **Application Load Balancer** -> **Create**.
4.  **Configuration**:
    *   **Load balancer name**: `ecs-lab-alb`
    *   **Scheme**: `Internet-facing`
    *   **VPC**: Select `ECS-Lab-VPC` (created by CloudFormation).
    *   **Mappings**: Select the two **public** subnets (e.g., `ECS-Public-A` and `ECS-Public-B`).
    *   **Security groups**: Remove the default and select the `ECS-ALB-SG`.
5.  **Listeners and routing**:
    *   The default listener should be `HTTP` on port `80`.
    *   For the default action, click **Create target group**.
        *   **Choose a target type**: `IP addresses` (because Fargate tasks have their own IPs).
        *   **Target group name**: `ecs-fargate-targets`
        *   **Protocol**: `HTTP`, **Port**: `8080`
        *   **VPC**: Select `ECS-Lab-VPC`.
        *   Click **Next**. Do **not** register any targets. Click **Create target group**.
    *   Back on the Load Balancer creation page, refresh and select the `ecs-fargate-targets` group you just created.
6.  Click **Create load balancer**.

**Subtask 4.2: Create the ECS Service**

1.  Navigate back to the **ECS Console** and click on your `modern-ecs-cluster`.
2.  Under the **Services** tab, click **Create**.
3.  **Deployment configuration**:
    *   **Launch type**: `FARGATE`.
    *   **Task Definition**: Select `my-flask-app-task` (should be revision 1).
    *   **Service name**: `flask-app-service`
    *   **Desired tasks**: `2` (this will launch two instances of our container).
4.  **Networking**:
    *   **VPC**: `ECS-Lab-VPC`.
    *   **Subnets**: Select the two **private** subnets (e.g., `ECS-Private-A` and `ECS-Private-B`).
    *   **Security groups**: Remove the default and select the `ECS-Container-SG`.
5.  **Load balancing**:
    *   **Load balancer type**: `Application Load Balancer`.
    *   Choose the `ecs-lab-alb` load balancer.
    *   **Listener**: Choose the existing listener on port `80`.
    *   **Target group name**: Choose the existing `ecs-fargate-targets` group.
6.  Click **Create**.

**Subtask 4.3: Verify Deployment**

1.  The service will now take a few minutes to provision the Fargate tasks and register them with the load balancer.
2.  Navigate to the **EC2 console** -> **Target Groups** -> `ecs-fargate-targets`.
3.  Click the **Targets** tab. After a few minutes, you should see two IP addresses registered and their status should change to **healthy**.
4.  Go back to the **Load Balancers** page, select `ecs-lab-alb`, and copy its **DNS name**.
5.  Paste the DNS name into a new browser tab. You should see the message: "Hello from ECS! This response is from Task ID: ..."
6.  Refresh the page several times. You should see the Task ID change as the load balancer distributes traffic between your two containers.

> **Why are we doing this?** This architecture is secure and highly available. The load balancer lives in public subnets, while our application containers are isolated in private subnets, only accessible via the load balancer. The service ensures our application is resilient by running multiple copies.

---

### **Module 5: Implementing Auto Scaling**

Now, let's configure the service to automatically scale out (add more tasks) when CPU is high and scale in (remove tasks) when it's low.

1.  In the **ECS Console**, go to your `modern-ecs-cluster` and click on `flask-app-service`.
2.  Click the **Update** button.
3.  Navigate to the **Service auto scaling** section.
4.  Configure as follows:
    *   **Minimum number of tasks**: `2`
    *   **Desired number of tasks**: `2`
    *   **Maximum number of tasks**: `6`
5.  Click **Add scaling policy**.
    *   **Policy name**: `cpu-scale-out`
    *   **ECS service metric**: `ECSServiceAverageCPUUtilization`
    *   **Target value**: `50` (When average CPU exceeds 50%, scale out)
    *   **Scale-out cooldown period**: `60` seconds.
6.  Click **Add scaling policy** again.
    *   **Policy name**: `cpu-scale-in`
    *   **ECS service metric**: `ECSServiceAverageCPUUtilization`
    *   **Target value**: `40` (When average CPU is below 40%, scale in)
    *   **Scale-in cooldown period**: `300` seconds.
7.  Click **Update**.

> **Why are we doing this?** Auto Scaling makes your application elastic. It can handle sudden traffic spikes without manual intervention and saves money by reducing capacity during quiet periods.

---

### **Module 6: Monitoring and Logging**

Let's see where to find our application logs and metrics.

1.  **CloudWatch Logs**:
    *   Navigate to the **CloudWatch Console**.
    *   Click **Log groups** in the left panel.
    *   Find and click the log group named `/ecs/my-flask-app-task`.
    *   You will see log streams for each of your running tasks. Click on one to see the output from the Python application.
2.  **ECS Metrics**:
    *   Navigate back to the **ECS Console** and view your service.
    *   The **Metrics** tab will show you the CPU and Memory utilization for your service over time. This is what Auto Scaling uses to make decisions.
3.  **Create a Dashboard**:
    *   In the **CloudWatch Console**, go to **Dashboards** -> **Create dashboard**.
    *   Name it `ECS-App-Dashboard`.
    *   Add a widget, select **Line**, and choose **Metrics**.
    *   Under **All metrics**, navigate to **ECS** -> **ClusterName, ServiceName**.
    *   Select the `CPUUtilization` and `MemoryUtilization` for your `modern-ecs-cluster` and `flask-app-service`.
    *   Add another widget for the Load Balancer. Navigate to **ALB** -> **Per AppELB Metrics**.
    *   Select `HealthyHostCount`, `UnHealthyHostCount`, and `HTTPCode_Target_2XX_Count` for your `ecs-lab-alb`.
    *   Save the dashboard.

---

### **Module 7 (Optional): Introduction to CI/CD with AWS CodePipeline**

A full CI/CD setup is extensive, but this shows the concept. A pipeline would automate the steps from Module 2:

1.  **Source Stage (CodeCommit/GitHub)**: A developer pushes a code change.
2.  **Build Stage (CodeBuild)**: A trigger starts a CodeBuild project that:
    *   Logs into ECR.
    *   Runs `docker build` and `docker push` to upload a new image version to ECR.
3.  **Deploy Stage (CodeDeploy/ECS)**: CodePipeline triggers a new deployment on the ECS service, which safely drains connections from old tasks and starts new tasks with the updated container image.

This creates a hands-off, automated path from code commit to production deployment.

---

### **Module 8: Comprehensive Cleanup**

To avoid ongoing charges, you must delete the resources we created. The easiest way is to delete the CloudFormation stack.

1.  **Scale Down the ECS Service**:
    *   In the ECS console, update your `flask-app-service`.
    *   Change the **Desired tasks** to `0`.
    *   Wait for the tasks to stop. Then delete the service.
2.  **Delete the CloudFormation Stack**:
    *   Navigate to the **CloudFormation Console**.
    *   Select the `ecs-lab-foundation` stack.
    *   Click **Delete**. This will remove the VPC, subnets, security groups, cluster, and ECR repository (if it's empty).
3.  **Delete Other Resources**:
    *   Delete the **Application Load Balancer** and its **Target Group**.
    *   Delete the **CloudWatch Log Group**.
    *   Delete the **IAM Role** `ecsTaskExecutionRole`.
    *   Delete the **CloudWatch Dashboard**.

### **Conclusion**

Congratulations! You have successfully deployed a containerized application on AWS using modern, best-practice techniques. You have learned how to leverage Infrastructure as Code for a repeatable setup, use AWS Fargate for serverless container execution, build and store images in ECR, and create a secure, scalable, and highly available architecture with an Application Load Balancer. This foundation is crucial for building robust microservices and implementing effective DevOps practices in the cloud.
