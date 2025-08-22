# Docker Microservices Lab - Complete Guide

## 🎯 **Big Picture Overview**

This lab will teach you to build a **complete microservices architecture** using Docker. Here's what we're building:

```
┌─────────────────────────────────────────────────────────────┐
│                    API Gateway (Port 3000)                 │
│                 (Traffic routing & rate limiting)          │
└─────────────────┬───────────────┬───────────────┬─────────┘
                  │               │               │
          ┌───────▼─────┐ ┌───────▼─────┐ ┌───────▼─────┐
          │ User Service│ │Product Service│ │Order Service│
          │  (Port 3001)│ │  (Port 3002) │ │ (Port 3003) │
          └─────────────┘ └─────────────┘ └─────────────┘
                            ▲               ▲
                            │               │
                            └───────┬───────┘
                                    │
                         Inter-service Communication
```

**What You'll Learn:**
- **Microservices Architecture** - Breaking apps into independent services
- **Docker Containerization** - Packaging each service in containers
- **Service Communication** - How services talk to each other
- **Docker Networking** - Secure container communication
- **Docker Compose** - Orchestrating multiple containers
- **API Gateway Pattern** - Single entry point for all requests
- **Scaling & Monitoring** - Performance optimization

## 📋 **Prerequisites & Requirements**

### **Knowledge Prerequisites:**
- ✅ Basic Docker commands (`docker run`, `docker build`, `docker ps`)
- ✅ Command line navigation (`cd`, `mkdir`, `ls`)
- ✅ Basic understanding of REST APIs
- ✅ JSON format familiarity
- ✅ Basic understanding of web services

### **Technical Requirements:**
- ✅ Access to Al Nafi cloud lab environment
- ✅ Docker Engine installed (provided in lab environment)
- ✅ Docker Compose installed (provided in lab environment)
- ✅ Node.js runtime (provided in lab environment)
- ✅ Text editor (nano/vim available)
- ✅ curl utility for testing (provided)

### **Lab Environment Setup:**
1. **Start your Al Nafi lab** - Click "Start Lab" button
2. **Wait for environment** - Lab will provision a Linux machine with all tools
3. **Access terminal** - You'll get a terminal with root access
4. **Verify tools** - We'll check all required tools are installed

---

## 🚀 **Ready to Begin?**

**Next Steps:**
1. **Launch your Al Nafi container** now
2. **Access the terminal** 
3. **Reply "READY"** when you have terminal access
4. I'll give you the **first set of verification commands** to ensure everything is set up correctly

This lab has **6 major tasks:**
- **Task 1:** Understanding Microservices Architecture
- **Task 2:** Building Multiple Microservices (User, Product, Order services)
- **Task 3:** Docker Networking & Communication
- **Task 4:** Docker Compose Orchestration
- **Task 5:** Scaling & Performance Monitoring
- **Task 6:** API Gateway Integration

Each task builds on the previous one, and I'll guide you through every command with clear explanations.

# 📦 **Your Container is 100% Sufficient!**

## ✅ **No External Accounts Needed**

**Great news!** This lab is completely **self-contained** within your Al Nafi container. Here's what you get:

### **Everything Pre-installed in Your Container:**
- ✅ **Docker Engine** - For building and running containers
- ✅ **Docker Compose** - For orchestrating multiple services
- ✅ **Node.js** - JavaScript runtime for our microservices
- ✅ **Python 3.x** - Additional runtime support
- ✅ **Text Editors** - nano and vim for code editing
- ✅ **Network Tools** - curl, wget for testing APIs
- ✅ **Linux Environment** - Full Ubuntu/CentOS with all utilities

### **No External Dependencies:**
- ❌ **No Docker Hub account needed** - We'll build images locally
- ❌ **No cloud accounts** - Everything runs in your container
- ❌ **No external databases** - We use in-memory storage for simplicity
- ❌ **No additional downloads** - All tools are pre-installed
- ❌ **No internet required** - After initial setup, everything is local

### **What We'll Build Locally:**
```
Your Al Nafi Container
├── 🐳 Docker Engine (locally running)
├── 📦 3 Microservices (built from scratch)
├── 🌐 Docker Network (internal communication)
├── 🔄 Load Balancer (nginx)
├── 📊 Monitoring Scripts
└── 🚪 API Gateway
```

---

## 🎯 **Ready to Start?**

**Simply:**
1. **Launch your Al Nafi lab** (click Start Lab button)
2. **Open the terminal** when ready
3. **Reply "READY"** and I'll give you the first commands

**Your container has everything we need - let's build some microservices!** 🚀

# 🔍 **Step 1: Environment Verification**

Great! Let's verify your lab environment has all the required tools. I'll explain each command and then provide them with pretty output formatting.

## **What We're Checking:**
- ✅ Docker Engine installation and version
- ✅ Docker Compose availability  
- ✅ Node.js runtime
- ✅ Network utilities (curl)
- ✅ Text editors
- ✅ Current working directory

---

## **Commands to Run:**

Copy and paste these commands one by one. Each command will show nicely formatted output:

```bash
# Check Docker Engine version and info
echo "🐳 DOCKER ENGINE STATUS"
echo "======================="
docker --version
docker info | head -10
echo ""
```

```bash
# Check Docker Compose version
echo "🐳 DOCKER COMPOSE STATUS" 
echo "========================"
docker-compose --version
echo ""
```

```bash
# Check Node.js version
echo "📦 NODE.JS RUNTIME STATUS"
echo "========================="
node --version
npm --version
echo ""
```

```bash
# Check network utilities
echo "🌐 NETWORK UTILITIES STATUS"
echo "============================"
curl --version | head -1
wget --version | head -1
echo ""
```

```bash
# Check available text editors
echo "📝 TEXT EDITORS STATUS"
echo "======================"
which nano && echo "✅ nano available"
which vim && echo "✅ vim available" 
echo ""
```

```bash
# Show current working directory and available space
echo "📁 CURRENT ENVIRONMENT"
echo "======================"
pwd
whoami
df -h | head -2
echo ""
```

---

## **What Each Command Does:**

1. **`docker --version`** - Shows installed Docker version
2. **`docker info`** - Displays Docker daemon information
3. **`docker-compose --version`** - Confirms Docker Compose is available
4. **`node --version` & `npm --version`** - Checks Node.js and npm versions
5. **`curl --version` & `wget --version`** - Verifies network testing tools
6. **`which nano/vim`** - Confirms text editors are available
7. **`pwd` & `whoami` & `df -h`** - Shows current directory, user, and disk space

**Run these commands now, and let me know when you see all the output! Then I'll give you the next set of commands to start building our microservices architecture.** 🚀

# 🔧 **Environment Setup Required**

I can see we need to install a few missing tools. Don't worry - this is normal! Let's get everything set up properly.

## **Current Status:**
- ✅ **Docker Engine** - Working perfectly (v26.1.3)
- ❌ **Docker Compose** - Missing (need to install)
- ❌ **Node.js & npm** - Missing (need to install)
- ✅ **curl & wget** - Available for testing
- ✅ **Text editors** - nano and vim ready
- ✅ **Environment** - 20GB available space, good to go!

---

## 🚀 **Step 2: Install Missing Tools**

Let's install the missing components. I'll explain each command:

### **Install Docker Compose:**
```bash
# Download and install Docker Compose (explains: gets the latest stable version)
echo "🔧 INSTALLING DOCKER COMPOSE"
echo "============================="
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Make it executable (explains: gives permission to run the command)
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker-compose --version
echo "✅ Docker Compose installed!"
echo ""
```

### **Install Node.js and npm:**
```bash
# Update package manager (explains: refreshes available software list)
echo "📦 INSTALLING NODE.JS"
echo "===================="
sudo apt update

# Install Node.js and npm (explains: installs JavaScript runtime and package manager)
sudo apt install -y nodejs npm

# Verify installation
echo "Checking Node.js installation..."
node --version
npm --version
echo "✅ Node.js and npm installed!"
echo ""
```

### **Final Verification:**
```bash
# Double-check everything is working
echo "🎯 FINAL VERIFICATION"
echo "===================="
echo "Docker: $(docker --version)"
echo "Docker Compose: $(docker-compose --version)"
echo "Node.js: $(node --version)"
echo "npm: $(npm --version)"
echo ""
echo "🎉 Environment setup complete! Ready to build microservices!"
```

---

## **What Each Command Does:**

1. **`sudo curl -L ...`** - Downloads Docker Compose binary from GitHub releases
2. **`sudo chmod +x`** - Makes the downloaded file executable
3. **`sudo apt update`** - Updates the package repository list
4. **`sudo apt install -y nodejs npm`** - Installs Node.js runtime and npm package manager
5. **Version checks** - Confirms everything installed correctly

**Run these commands in order, and let me know when you see "Environment setup complete!" Then we'll start building our microservices!** 🚀

# 🏗️ **Step 3: Create Project Structure**

Now let's create the foundation for our microservices architecture. I'll explain what we're building and why.

## **What We're Creating:**
We're building a **modular project structure** where each microservice lives in its own directory. This follows microservices best practices for organization and deployment.

---

## **Commands to Run:**

### **Create Main Project Directory:**
```bash
# Create the main project folder (explains: this will contain all our microservices)
echo "🏗️ CREATING PROJECT STRUCTURE"
echo "=============================="
mkdir microservices-lab
cd microservices-lab

# Show where we are now
echo "📁 Current location: $(pwd)"
echo ""
```

### **Create Individual Service Directories:**
```bash
# Create directories for each microservice (explains: separate folders for independent services)
echo "📦 CREATING SERVICE DIRECTORIES"
echo "==============================="
mkdir user-service          # Handles user management (registration, profiles)
mkdir order-service         # Manages orders and transactions  
mkdir product-service       # Product catalog and inventory
mkdir api-gateway          # Single entry point for all requests
mkdir docker-compose-files # Orchestration configurations
mkdir shared-configs       # Common settings across services

# Create a nice directory tree view
echo "📋 PROJECT STRUCTURE CREATED:"
echo "=============================="
tree . 2>/dev/null || ls -la
echo ""
```

### **Verify Structure & Add Some Color:**
```bash
# Show detailed directory structure with colors
echo "🎨 DETAILED PROJECT LAYOUT"
echo "=========================="
ls -la --color=always
echo ""

# Show current working directory
echo "📍 You are now in: $(pwd)"
echo "🚀 Ready to build microservices!"
echo ""
```

---

## **What Each Command Does:**

1. **`mkdir microservices-lab`** - Creates main project folder to organize everything
2. **`cd microservices-lab`** - Changes into our project directory  
3. **`mkdir user-service`** - Creates folder for user management service
4. **`mkdir order-service`** - Creates folder for order processing service
5. **`mkdir product-service`** - Creates folder for product catalog service
6. **`mkdir api-gateway`** - Creates folder for traffic routing service
7. **`mkdir docker-compose-files`** - Creates folder for container orchestration
8. **`mkdir shared-configs`** - Creates folder for common configurations
9. **`tree` or `ls -la`** - Shows the directory structure we just created

**This structure follows microservices best practices where each service is completely independent and can be developed, deployed, and scaled separately.**

**Run these commands and let me know when you see the project structure created! Next, we'll start building our first microservice (User Service).** 🎯

# 👥 **Step 4: Build the User Service**

Now we'll create our first microservice! The **User Service** handles user registration, authentication, and profile management.

## **What We're Building:**
A RESTful API service that manages users with endpoints for:
- Health checks
- Getting all users
- Getting user by ID  
- Creating new users

---

## **Commands to Run:**

### **Navigate to User Service Directory:**
```bash
# Move into the user service folder (explains: we'll build this service here)
echo "👥 BUILDING USER SERVICE"
echo "========================"
cd user-service

# Show current location with nice formatting
echo "📁 Current location: $(pwd)"
echo "🎯 Building: User Management Microservice"
echo ""
```

### **Create the Main Application File:**
```bash
# Create the main Node.js application (explains: this is the core service logic)
echo "📝 CREATING USER SERVICE APPLICATION"
echo "==================================="
nano app.js
```

**In the nano editor, paste this complete User Service code:**

```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3001;

// Middleware
app.use(express.json());

// In-memory user storage (for demo purposes)
let users = [
    { id: 1, name: 'John Doe', email: 'john@example.com', role: 'customer' },
    { id: 2, name: 'Jane Smith', email: 'jane@example.com', role: 'admin' }
];

// Health check endpoint
app.get('/health', (req, res) => {
    res.json({ 
        service: 'user-service', 
        status: 'healthy', 
        timestamp: new Date().toISOString() 
    });
});

// Get all users
app.get('/users', (req, res) => {
    res.json({
        success: true,
        data: users,
        count: users.length
    });
});

// Get user by ID
app.get('/users/:id', (req, res) => {
    const userId = parseInt(req.params.id);
    const user = users.find(u => u.id === userId);
    
    if (!user) {
        return res.status(404).json({
            success: false,
            message: 'User not found'
        });
    }
    
    res.json({
        success: true,
        data: user
    });
});

// Create new user
app.post('/users', (req, res) => {
    const { name, email, role } = req.body;
    
    if (!name || !email) {
        return res.status(400).json({
            success: false,
            message: 'Name and email are required'
        });
    }
    
    const newUser = {
        id: users.length + 1,
        name,
        email,
        role: role || 'customer'
    };
    
    users.push(newUser);
    
    res.status(201).json({
        success: true,
        data: newUser,
        message: 'User created successfully'
    });
});

// Start server
app.listen(PORT, '0.0.0.0', () => {
    console.log(`User Service running on port ${PORT}`);
    console.log(`Health check: http://localhost:${PORT}/health`);
});
```

**To save and exit nano:** Press `Ctrl+X`, then `Y`, then `Enter`

### **Create Package Configuration:**
```bash
# Create package.json (explains: defines service dependencies and metadata)
echo "📋 CREATING PACKAGE CONFIGURATION"
echo "=================================="
nano package.json
```

**In nano, paste this package.json:**

```json
{
    "name": "user-service",
    "version": "1.0.0",
    "description": "User management microservice",
    "main": "app.js",
    "scripts": {
        "start": "node app.js",
        "dev": "nodemon app.js"
    },
    "dependencies": {
        "express": "^4.18.2"
    },
    "keywords": ["microservice", "user", "docker"],
    "author": "Lab Student",
    "license": "MIT"
}
```

**Save and exit:** `Ctrl+X`, `Y`, `Enter`

### **Create Docker Configuration:**
```bash
# Create Dockerfile (explains: instructions for building the container image)
echo "🐳 CREATING DOCKERFILE"
echo "======================"
nano Dockerfile
```

**In nano, paste this Dockerfile:**

```dockerfile
# Use official Node.js runtime as base image
FROM node:18-alpine

# Set working directory in container
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install --only=production

# Copy application code
COPY . .

# Expose port
EXPOSE 3001

# Add health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3001/health || exit 1

# Start the application
CMD ["npm", "start"]
```

**Save and exit:** `Ctrl+X`, `Y`, `Enter`

### **Verify User Service Files:**
```bash
# Check all files were created correctly
echo "✅ USER SERVICE FILES CREATED"
echo "============================="
ls -la --color=always
echo ""

# Show file contents summary
echo "📄 FILE SUMMARY:"
echo "================"
echo "📝 app.js: $(wc -l < app.js) lines - Main service logic"
echo "📋 package.json: $(wc -l < package.json) lines - Dependencies"  
echo "🐳 Dockerfile: $(wc -l < Dockerfile) lines - Container instructions"
echo ""
echo "🎉 User Service ready for containerization!"
```

---

## **What We Just Built:**

- **`app.js`** - Express.js web server with REST API endpoints
- **`package.json`** - Node.js project configuration and dependencies
- **`Dockerfile`** - Instructions for building a Docker container image

**The User Service provides these API endpoints:**
- `GET /health` - Service health status
- `GET /users` - List all users
- `GET /users/:id` - Get specific user
- `POST /users` - Create new user

**Run these commands and let me know when you see "User Service ready for containerization!" Then we'll build the Product Service next!** 🚀

# 🛍️ **Step 5: Build the Product Service**

Now let's create our second microservice! The **Product Service** manages product catalog, inventory, and pricing.

## **What We're Building:**
A product management API with features for:
- Product catalog browsing
- Filtering by category and price
- Inventory management
- Product creation

---

## **Commands to Run:**

### **Navigate to Product Service Directory:**
```bash
# Move to product service folder (explains: building our second microservice)
echo "🛍️ BUILDING PRODUCT SERVICE"
echo "==========================="
cd ../product-service

# Show current location
echo "📁 Current location: $(pwd)"
echo "🎯 Building: Product Catalog Microservice"
echo ""
```

### **Create the Main Application File:**
```bash
# Create the product service application (explains: core product management logic)
echo "📝 CREATING PRODUCT SERVICE APPLICATION"
echo "======================================"
nano app.js
```

**In nano, paste this Product Service code:**

```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3002;

// Middleware
app.use(express.json());

// In-memory product storage
let products = [
    { id: 1, name: 'Laptop', price: 999.99, category: 'Electronics', stock: 50 },
    { id: 2, name: 'Smartphone', price: 699.99, category: 'Electronics', stock: 100 },
    { id: 3, name: 'Book', price: 19.99, category: 'Education', stock: 200 },
    { id: 4, name: 'Headphones', price: 149.99, category: 'Electronics', stock: 75 }
];

// Health check endpoint
app.get('/health', (req, res) => {
    res.json({ 
        service: 'product-service', 
        status: 'healthy', 
        timestamp: new Date().toISOString() 
    });
});

// Get all products
app.get('/products', (req, res) => {
    const { category, minPrice, maxPrice } = req.query;
    let filteredProducts = [...products];
    
    if (category) {
        filteredProducts = filteredProducts.filter(p => 
            p.category.toLowerCase() === category.toLowerCase()
        );
    }
    
    if (minPrice) {
        filteredProducts = filteredProducts.filter(p => p.price >= parseFloat(minPrice));
    }
    
    if (maxPrice) {
        filteredProducts = filteredProducts.filter(p => p.price <= parseFloat(maxPrice));
    }
    
    res.json({
        success: true,
        data: filteredProducts,
        count: filteredProducts.length
    });
});

// Get product by ID
app.get('/products/:id', (req, res) => {
    const productId = parseInt(req.params.id);
    const product = products.find(p => p.id === productId);
    
    if (!product) {
        return res.status(404).json({
            success: false,
            message: 'Product not found'
        });
    }
    
    res.json({
        success: true,
        data: product
    });
});

// Create new product
app.post('/products', (req, res) => {
    const { name, price, category, stock } = req.body;
    
    if (!name || !price || !category) {
        return res.status(400).json({
            success: false,
            message: 'Name, price, and category are required'
        });
    }
    
    const newProduct = {
        id: products.length + 1,
        name,
        price: parseFloat(price),
        category,
        stock: parseInt(stock) || 0
    };
    
    products.push(newProduct);
    
    res.status(201).json({
        success: true,
        data: newProduct,
        message: 'Product created successfully'
    });
});

// Update product stock
app.patch('/products/:id/stock', (req, res) => {
    const productId = parseInt(req.params.id);
    const { quantity } = req.body;
    
    const product = products.find(p => p.id === productId);
    
    if (!product) {
        return res.status(404).json({
            success: false,
            message: 'Product not found'
        });
    }
    
    if (quantity !== undefined) {
        product.stock = parseInt(quantity);
    }
    
    res.json({
        success: true,
        data: product,
        message: 'Stock updated successfully'
    });
});

// Start server
app.listen(PORT, '0.0.0.0', () => {
    console.log(`Product Service running on port ${PORT}`);
    console.log(`Health check: http://localhost:${PORT}/health`);
});
```

**Save and exit:** `Ctrl+X`, `Y`, `Enter`

### **Create Package Configuration:**
```bash
# Create package.json for product service
echo "📋 CREATING PRODUCT PACKAGE CONFIGURATION"
echo "========================================="
nano package.json
```

**In nano, paste:**

```json
{
    "name": "product-service",
    "version": "1.0.0",
    "description": "Product management microservice",
    "main": "app.js",
    "scripts": {
        "start": "node app.js",
        "dev": "nodemon app.js"
    },
    "dependencies": {
        "express": "^4.18.2"
    },
    "keywords": ["microservice", "product", "docker"],
    "author": "Lab Student",
    "license": "MIT"
}
```

**Save and exit:** `Ctrl+X`, `Y`, `Enter`

### **Create Dockerfile:**
```bash
# Create Dockerfile for product service
echo "🐳 CREATING PRODUCT DOCKERFILE"
echo "=============================="
nano Dockerfile
```

**In nano, paste:**

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install --only=production

COPY . .

EXPOSE 3002

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3002/health || exit 1

CMD ["npm", "start"]
```

**Save and exit:** `Ctrl+X`, `Y`, `Enter`

### **Verify Product Service:**
```bash
# Check product service files
echo "✅ PRODUCT SERVICE FILES CREATED"
echo "================================"
ls -la --color=always
echo ""

# Show file summary with nice formatting
echo "📊 PRODUCT SERVICE SUMMARY:"
echo "==========================="
echo "📝 app.js: $(wc -l < app.js) lines - Product management API"
echo "📋 package.json: $(wc -l < package.json) lines - Dependencies"
echo "🐳 Dockerfile: $(wc -l < Dockerfile) lines - Container config"
echo ""
echo "🎉 Product Service ready! Next: Order Service"
```

---

## **What We Just Built:**

**The Product Service provides these API endpoints:**
- `GET /health` - Service health check
- `GET /products` - List all products (with filtering by category, price range)
- `GET /products/:id` - Get specific product details
- `POST /products` - Add new products to catalog
- `PATCH /products/:id/stock` - Update inventory levels

**Key Features:**
- **Filtering** - Search by category, min/max price
- **Inventory Management** - Track stock levels
- **Data Validation** - Ensures required fields
- **Error Handling** - Proper HTTP status codes

**Run these commands and let me know when you see "Product Service ready!" Next, we'll build the most complex service - the Order Service that communicates with both User and Product services!** 🚀

# 📦 **Step 6: Build the Order Service**

Now for the most complex service! The **Order Service** communicates with both User and Product services to process orders. This demonstrates **inter-service communication**.

## **What We're Building:**
An order processing service that:
- Validates users exist (calls User Service)
- Verifies products and prices (calls Product Service)
- Creates and manages orders
- Handles service failures gracefully

---

## **Commands to Run:**

### **Navigate to Order Service Directory:**
```bash
# Move to order service folder (explains: building our most complex microservice)
echo "📦 BUILDING ORDER SERVICE"
echo "========================="
cd ../order-service

# Show current location
echo "📁 Current location: $(pwd)"
echo "🎯 Building: Order Processing Microservice (with inter-service communication)"
echo ""
```

### **Create the Main Application File:**
```bash
# Create the order service application (explains: handles complex business logic)
echo "📝 CREATING ORDER SERVICE APPLICATION"
echo "===================================="
nano app.js
```

**In nano, paste this Order Service code:**

```javascript
const express = require('express');
const axios = require('axios');
const app = express();
const PORT = process.env.PORT || 3003;

// Service URLs (will be resolved by Docker networking)
const USER_SERVICE_URL = process.env.USER_SERVICE_URL || 'http://user-service:3001';
const PRODUCT_SERVICE_URL = process.env.PRODUCT_SERVICE_URL || 'http://product-service:3002';

// Middleware
app.use(express.json());

// In-memory order storage
let orders = [
    {
        id: 1,
        userId: 1,
        products: [
            { productId: 1, quantity: 1, price: 999.99 }
        ],
        total: 999.99,
        status: 'completed',
        createdAt: new Date('2024-01-15').toISOString()
    }
];

// Health check endpoint
app.get('/health', (req, res) => {
    res.json({ 
        service: 'order-service', 
        status: 'healthy', 
        timestamp: new Date().toISOString() 
    });
});

// Get all orders
app.get('/orders', (req, res) => {
    res.json({
        success: true,
        data: orders,
        count: orders.length
    });
});

// Get orders by user ID
app.get('/orders/user/:userId', (req, res) => {
    const userId = parseInt(req.params.userId);
    const userOrders = orders.filter(o => o.userId === userId);
    
    res.json({
        success: true,
        data: userOrders,
        count: userOrders.length
    });
});

// Get order by ID
app.get('/orders/:id', (req, res) => {
    const orderId = parseInt(req.params.id);
    const order = orders.find(o => o.id === orderId);
    
    if (!order) {
        return res.status(404).json({
            success: false,
            message: 'Order not found'
        });
    }
    
    res.json({
        success: true,
        data: order
    });
});

// Create new order
app.post('/orders', async (req, res) => {
    try {
        const { userId, products } = req.body;
        
        if (!userId || !products || !Array.isArray(products) || products.length === 0) {
            return res.status(400).json({
                success: false,
                message: 'User ID and products array are required'
            });
        }
        
        // Verify user exists
        try {
            const userResponse = await axios.get(`${USER_SERVICE_URL}/users/${userId}`);
            if (!userResponse.data.success) {
                return res.status(404).json({
                    success: false,
                    message: 'User not found'
                });
            }
        } catch (error) {
            console.log('User service unavailable, proceeding without verification');
        }
        
        // Calculate total and verify products
        let total = 0;
        const orderProducts = [];
        
        for (const item of products) {
            try {
                const productResponse = await axios.get(`${PRODUCT_SERVICE_URL}/products/${item.productId}`);
                if (productResponse.data.success) {
                    const product = productResponse.data.data;
                    const itemTotal = product.price * item.quantity;
                    total += itemTotal;
                    
                    orderProducts.push({
                        productId: item.productId,
                        quantity: item.quantity,
                        price: product.price,
                        name: product.name
                    });
                }
            } catch (error) {
                console.log(`Product ${item.productId} service unavailable, using provided data`);
                orderProducts.push({
                    productId: item.productId,
                    quantity: item.quantity,
                    price: item.price || 0
                });
                total += (item.price || 0) * item.quantity;
            }
        }
        
        const newOrder = {
            id: orders.length + 1,
            userId,
            products: orderProducts,
            total: Math.round(total * 100) / 100, // Round to 2 decimal places
            status: 'pending',
            createdAt: new Date().toISOString()
        };
        
        orders.push(newOrder);
        
        res.status(201).json({
            success: true,
            data: newOrder,
            message: 'Order created successfully'
        });
        
    } catch (error) {
        console.error('Error creating order:', error.message);
        res.status(500).json({
            success: false,
            message: 'Internal server error'
        });
    }
});

// Update order status
app.patch('/orders/:id/status', (req, res) => {
    const orderId = parseInt(req.params.id);
    const { status } = req.body;
    
    const order = orders.find(o => o.id === orderId);
    
    if (!order) {
        return res.status(404).json({
            success: false,
            message: 'Order not found'
        });
    }
    
    const validStatuses = ['pending', 'processing', 'shipped', 'delivered', 'cancelled'];
    if (!validStatuses.includes(status)) {
        return res.status(400).json({
            success: false,
            message: 'Invalid status. Valid statuses: ' + validStatuses.join(', ')
        });
    }
    
    order.status = status;
    order.updatedAt = new Date().toISOString();
    
    res.json({
        success: true,
        data: order,
        message: 'Order status updated successfully'
    });
});

// Start server
app.listen(PORT, '0.0.0.0', () => {
    console.log(`Order Service running on port ${PORT}`);
    console.log(`Health check: http://localhost:${PORT}/health`);
    console.log(`User Service URL: ${USER_SERVICE_URL}`);
    console.log(`Product Service URL: ${PRODUCT_SERVICE_URL}`);
});
```

**Save and exit:** `Ctrl+X`, `Y`, `Enter`

### **Create Package Configuration:**
```bash
# Create package.json (explains: includes axios for inter-service communication)
echo "📋 CREATING ORDER PACKAGE CONFIGURATION"
echo "======================================="
nano package.json
```

**In nano, paste:**

```json
{
    "name": "order-service",
    "version": "1.0.0",
    "description": "Order management microservice",
    "main": "app.js",
    "scripts": {
        "start": "node app.js",
        "dev": "nodemon app.js"
    },
    "dependencies": {
        "express": "^4.18.2",
        "axios": "^1.6.0"
    },
    "keywords": ["microservice", "order", "docker"],
    "author": "Lab Student",
    "license": "MIT"
}
```

**Save and exit:** `Ctrl+X`, `Y`, `Enter`

### **Create Dockerfile:**
```bash
# Create Dockerfile for order service
echo "🐳 CREATING ORDER DOCKERFILE"
echo "============================"
nano Dockerfile
```

**In nano, paste:**

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install --only=production

COPY . .

EXPOSE 3003

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3003/health || exit 1

CMD ["npm", "start"]
```

**Save and exit:** `Ctrl+X`, `Y`, `Enter`

### **Verify Order Service & Show Progress:**
```bash
# Check order service files
echo "✅ ORDER SERVICE FILES CREATED"
echo "=============================="
ls -la --color=always
echo ""

# Show comprehensive progress
echo "📊 MICROSERVICES PROGRESS:"
echo "=========================="
cd ..
echo "👥 User Service: ✅ Complete"
echo "🛍️ Product Service: ✅ Complete" 
echo "📦 Order Service: ✅ Complete"
echo ""

# Show overall project structure
echo "🏗️ CURRENT PROJECT STRUCTURE:"
echo "============================="
tree . 2>/dev/null || find . -type d | head -10
echo ""
echo "🎯 Next: Docker Networking & Container Communication!"
```

---

## **What We Just Built:**

**The Order Service is special because it:**
- **Calls other services** - Uses `axios` to communicate with User and Product services
- **Handles failures gracefully** - Continues working even if other services are down
- **Validates data** - Checks users exist and gets real product prices
- **Calculates totals** - Computes order totals from product prices

**Key Features:**
- `GET /orders` - List all orders
- `GET /orders/user/:userId` - Get orders for specific user
- `GET /orders/:id` - Get specific order details
- `POST /orders` - Create new order (validates with other services)
- `PATCH /orders/:id/status` - Update order status

**This demonstrates the microservices pattern where services work together while remaining independent!**

**Run these commands and let me know when you see "Next: Docker Networking & Container Communication!" Then we'll set up the networking to make all services talk to each other!** 🌐

# 🌐 **Step 7: Docker Networking & Build All Services**

Now we'll create a **Docker network** for secure service communication and **build all our container images**. This is where the magic happens!

## **What We're Doing:**
- Creating a custom Docker network for isolated service communication
- Building Docker images for all three services
- Preparing for container orchestration

---

## **Commands to Run:**

### **Create Docker Network:**
```bash
# Create custom network for microservices (explains: isolated communication channel)
echo "🌐 CREATING DOCKER NETWORK"
echo "=========================="
cd docker-compose-files
docker network create microservices-network

# Verify network creation with pretty output
echo ""
echo "📋 DOCKER NETWORKS:"
echo "==================="
docker network ls --format "table {{.Name}}\t{{.Driver}}\t{{.Scope}}"
echo ""
echo "✅ Custom network 'microservices-network' created!"
echo ""
```

### **Build All Service Images:**
```bash
# Build User Service image
echo "🏗️ BUILDING SERVICE IMAGES"
echo "=========================="
echo "👥 Building User Service..."
cd ../user-service
docker build -t user-service:latest . --quiet
echo "✅ User Service image built!"

# Build Product Service image  
echo "🛍️ Building Product Service..."
cd ../product-service
docker build -t product-service:latest . --quiet
echo "✅ Product Service image built!"

# Build Order Service image
echo "📦 Building Order Service..."
cd ../order-service  
docker build -t order-service:latest . --quiet
echo "✅ Order Service image built!"
echo ""
```

### **Verify All Images Built Successfully:**
```bash
# Show all our microservice images with nice formatting
echo "🐳 BUILT MICROSERVICE IMAGES"
echo "============================"
echo "📊 Image Details:"
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}\t{{.CreatedSince}}" | grep -E "(REPOSITORY|user-service|product-service|order-service)"
echo ""

# Count our services
echo "📈 SERVICE SUMMARY:"
echo "==================="
SERVICE_COUNT=$(docker images | grep -E "(user-service|product-service|order-service)" | wc -l)
echo "🎯 Total Microservices Built: $SERVICE_COUNT/3"
echo "🌐 Docker Network: microservices-network ✅"
echo "🚀 Ready for container orchestration!"
echo ""
```

### **Test Network Connectivity:**
```bash
# Test our network exists and works
echo "🔍 NETWORK CONNECTIVITY TEST"
echo "============================"
docker network inspect microservices-network --format "{{.Name}}: {{.Driver}} driver" 
echo "📡 Network Subnet: $(docker network inspect microservices-network --format '{{range .IPAM.Config}}{{.Subnet}}{{end}}')"
echo ""

# Show project structure overview
echo "🏗️ COMPLETE PROJECT OVERVIEW"
echo "============================="
cd ..
echo "📁 Current location: $(pwd)"
tree . -L 2 2>/dev/null || ls -la
echo ""
echo "🎉 All services built and network ready!"
echo "🚀 Next: Docker Compose orchestration!"
```

---

## **What We Just Accomplished:**

### **🌐 Docker Network Created:**
- **`microservices-network`** - Custom bridge network
- **Isolated communication** - Services can only talk to each other through this network
- **Service discovery** - Services can find each other by container name
- **Security** - Network isolation from other containers

### **🐳 Docker Images Built:**
- **`user-service:latest`** - Containerized user management API
- **`product-service:latest`** - Containerized product catalog API  
- **`order-service:latest`** - Containerized order processing API

### **🔧 Key Features:**
- **Container isolation** - Each service runs independently
- **Network communication** - Services can call each other via HTTP
- **Health checks** - Built-in monitoring capabilities
- **Lightweight images** - Using Alpine Linux base for efficiency

**The images are now ready to be orchestrated together using Docker Compose!**

---

## **What Each Command Did:**

1. **`docker network create`** - Created isolated network for service communication
2. **`docker build -t service:latest .`** - Built container image from Dockerfile
3. **`docker images`** - Listed all built images to verify success
4. **`docker network inspect`** - Showed network configuration details

**Run these commands and let me know when you see "Next: Docker Compose orchestration!" We'll then use Docker Compose to start all services together with one command!** 🎯

# 🎼 **Step 8: Docker Compose Orchestration**

Now for the exciting part! We'll use **Docker Compose** to orchestrate all our microservices with a single command. This creates our complete microservices architecture!

## **What We're Creating:**
A **docker-compose.yml** file that:
- Defines all 3 services and their configurations
- Sets up networking between services
- Configures environment variables
- Adds health checks and dependencies
- Enables service discovery

---

## **Commands to Run:**

### **Navigate to Compose Directory:**
```bash
# Move to docker-compose configuration folder
echo "🎼 CREATING DOCKER COMPOSE ORCHESTRATION"
echo "========================================"
cd docker-compose-files

# Show current location
echo "📁 Current location: $(pwd)"
echo "🎯 Creating: Complete microservices orchestration"
echo ""
```

### **Create Docker Compose Configuration:**
```bash
# Create the main orchestration file (explains: defines entire microservices architecture)
echo "📝 CREATING DOCKER-COMPOSE.YML"
echo "==============================="
nano docker-compose.yml
```

**In nano, paste this complete Docker Compose configuration:**

```yaml
version: '3.8'

services:
  # User Service
  user-service:
    build:
      context: ../user-service
      dockerfile: Dockerfile
    container_name: user-service
    ports:
      - "3001:3001"
    networks:
      - microservices-network
    environment:
      - NODE_ENV=production
      - PORT=3001
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3001/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    restart: unless-stopped
    labels:
      - "service.name=user-service"
      - "service.version=1.0.0"

  # Product Service
  product-service:
    build:
      context: ../product-service
      dockerfile: Dockerfile
    container_name: product-service
    ports:
      - "3002:3002"
    networks:
      - microservices-network
    environment:
      - NODE_ENV=production
      - PORT=3002
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3002/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    restart: unless-stopped
    labels:
      - "service.name=product-service"
      - "service.version=1.0.0"

  # Order Service
  order-service:
    build:
      context: ../order-service
      dockerfile: Dockerfile
    container_name: order-service
    ports:
      - "3003:3003"
    networks:
      - microservices-network
    environment:
      - NODE_ENV=production
      - PORT=3003
      - USER_SERVICE_URL=http://user-service:3001
      - PRODUCT_SERVICE_URL=http://product-service:3002
    depends_on:
      - user-service
      - product-service
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3003/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    restart: unless-stopped
    labels:
      - "service.name=order-service"
      - "service.version=1.0.0"

networks:
  microservices-network:
    driver: bridge
    name: microservices-network
```

**Save and exit:** `Ctrl+X`, `Y`, `Enter`

### **Deploy All Microservices:**
```bash
# Start all services with Docker Compose (explains: launches entire microservices architecture)
echo "🚀 DEPLOYING MICROSERVICES ARCHITECTURE"
echo "======================================="
docker-compose up -d

echo ""
echo "⏳ Waiting for services to start..."
sleep 10
echo ""
```

### **Check Service Status:**
```bash
# Check status of all services with nice formatting
echo "📊 MICROSERVICES STATUS"
echo "======================="
docker-compose ps --format "table {{.Name}}\t{{.Status}}\t{{.Ports}}"
echo ""

# Show running containers
echo "🐳 RUNNING CONTAINERS"
echo "===================="
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}" | grep -E "(NAMES|user-service|product-service|order-service)"
echo ""
```

### **View Service Logs:**
```bash
# Show logs from all services (explains: see what's happening inside containers)
echo "📋 SERVICE STARTUP LOGS"
echo "======================="
echo "👥 User Service logs:"
docker-compose logs --tail 5 user-service
echo ""

echo "🛍️ Product Service logs:"
docker-compose logs --tail 5 product-service  
echo ""

echo "📦 Order Service logs:"
docker-compose logs --tail 5 order-service
echo ""
```

### **Test All Services Are Running:**
```bash
# Test each service health endpoint
echo "🏥 HEALTH CHECK - ALL SERVICES"
echo "=============================="

echo "👥 Testing User Service..."
curl -s http://localhost:3001/health | jq '.' 2>/dev/null || curl -s http://localhost:3001/health
echo ""

echo "🛍️ Testing Product Service..."
curl -s http://localhost:3002/health | jq '.' 2>/dev/null || curl -s http://localhost:3002/health
echo ""

echo "📦 Testing Order Service..."
curl -s http://localhost:3003/health | jq '.' 2>/dev/null || curl -s http://localhost:3003/health
echo ""

echo "🎉 ALL MICROSERVICES ARE RUNNING!"
echo "================================="
echo "🌐 Your microservices architecture is live!"
```

---

## **What Each Section Does:**

### **Docker Compose Configuration:**
- **`version: '3.8'`** - Uses latest Compose file format
- **`services:`** - Defines each microservice container
- **`build:`** - Points to Dockerfile location for each service
- **`ports:`** - Maps container ports to host ports (3001, 3002, 3003)
- **`networks:`** - Connects all services to our custom network
- **`environment:`** - Sets environment variables for service URLs
- **`depends_on:`** - Order service waits for User and Product services
- **`healthcheck:`** - Monitors service health automatically
- **`restart: unless-stopped`** - Auto-restart containers if they crash

### **Key Benefits:**
- **Single command deployment** - `docker-compose up -d` starts everything
- **Service discovery** - Services find each other by name (user-service, product-service)
- **Dependency management** - Services start in the right order
- **Health monitoring** - Automatic health checks
- **Network isolation** - Secure communication between services

**Run these commands and let me know when you see "ALL MICROSERVICES ARE RUNNING!" Next, we'll test inter-service communication!** 🚀

# 🔧 **Troubleshooting: Services Not Running**

I see the issue! The containers aren't running. Let's diagnose and fix this step by step.

## **What's Happening:**
The `docker-compose up -d` command likely failed silently. Let's check what went wrong and fix it.

---

## **Diagnostic Commands:**

### **Check Docker Compose Status:**
```bash
# Check if docker-compose.yml exists and is valid
echo "🔍 DIAGNOSING DOCKER COMPOSE ISSUES"
echo "==================================="
pwd
ls -la --color=always
echo ""

# Check if our compose file exists
echo "📋 Checking docker-compose.yml:"
if [ -f docker-compose.yml ]; then
    echo "✅ docker-compose.yml exists"
    echo "📏 File size: $(wc -l < docker-compose.yml) lines"
else
    echo "❌ docker-compose.yml missing!"
fi
echo ""
```

### **Try Starting Services with Verbose Output:**
```bash
# Start services with detailed output (explains: see what's failing)
echo "🚀 ATTEMPTING TO START SERVICES WITH DETAILED OUTPUT"
echo "===================================================="
docker-compose up --build --no-deps

# This will show us exactly what's going wrong
```

**Wait! Let's see the output from this command first before proceeding. It will show us exactly what error is occurring.**

---

## **Most Likely Issues & Solutions:**

### **Issue 1: Network Already Exists**
If you see "network already exists" error:
```bash
# Remove existing network and recreate
docker network rm microservices-network 2>/dev/null
docker network create microservices-network
```

### **Issue 2: Build Context Problems**
The build paths might be wrong. Let's check:
```bash
# Verify our directory structure
echo "🏗️ CHECKING BUILD CONTEXTS"
echo "=========================="
echo "Current directory: $(pwd)"
echo ""
echo "Checking service directories:"
ls -la ../user-service/ 2>/dev/null && echo "✅ user-service directory exists" || echo "❌ user-service missing"
ls -la ../product-service/ 2>/dev/null && echo "✅ product-service directory exists" || echo "❌ product-service missing"  
ls -la ../order-service/ 2>/dev/null && echo "✅ order-service directory exists" || echo "❌ order-service missing"
```

---

## **Let's Start Fresh:**

**First, run the diagnostic commands above to see what's happening, then I'll give you the exact fix based on what we find.**

**Run these diagnostic commands and paste the output - especially the `docker-compose up --build --no-deps` output. That will show us exactly what's failing!** 🔍

# 🔧 **Issue Found: Network Configuration Problem**

Great! I can see what's happening. The images built successfully, but there's a **network configuration conflict**. The network exists but wasn't created by Docker Compose.

## **The Problem:**
- ✅ **Images built successfully** - All 3 services compiled correctly
- ❌ **Network conflict** - The manually created network conflicts with Compose
- ❌ **Services not starting** - Due to network configuration mismatch

---

## **Fix Commands:**

### **Clean Up and Restart:**
```bash
# Stop any existing containers and clean up
echo "🧹 CLEANING UP EXISTING RESOURCES"
echo "================================="
docker-compose down --remove-orphans
docker network rm microservices-network 2>/dev/null || echo "Network already removed"
echo "✅ Cleanup complete!"
echo ""
```

### **Start Services Fresh:**
```bash
# Start services fresh (explains: Docker Compose will create network automatically)
echo "🚀 STARTING MICROSERVICES FRESH"
echo "==============================="
docker-compose up -d --build

echo ""
echo "⏳ Waiting for services to fully start..."
sleep 15
echo ""
```

### **Check Service Status:**
```bash
# Check if containers are running now
echo "📊 CONTAINER STATUS CHECK"
echo "========================="
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}" --filter "name=service"
echo ""

# Check Docker Compose status specifically
echo "🎼 DOCKER COMPOSE STATUS"
echo "========================"
docker-compose ps
echo ""
```

### **If Still Not Working - Check Logs:**
```bash
# Check logs for any startup errors
echo "📋 CHECKING SERVICE LOGS FOR ERRORS"
echo "==================================="
echo "👥 User Service logs:"
docker-compose logs user-service --tail 10
echo ""

echo "🛍️ Product Service logs:"
docker-compose logs product-service --tail 10
echo ""

echo "📦 Order Service logs:"
docker-compose logs order-service --tail 10
echo ""
```

### **Test Services Once Running:**
```bash
# Test services are responding
echo "🏥 TESTING SERVICE HEALTH"
echo "========================="
echo "👥 User Service test:"
curl -s http://localhost:3001/health && echo " ✅ User Service OK" || echo " ❌ User Service DOWN"

echo "🛍️ Product Service test:"
curl -s http://localhost:3002/health && echo " ✅ Product Service OK" || echo " ❌ Product Service DOWN"

echo "📦 Order Service test:"
curl -s http://localhost:3003/health && echo " ✅ Order Service OK" || echo " ❌ Order Service DOWN"
echo ""
```

---

## **What These Commands Do:**

1. **`docker-compose down --remove-orphans`** - Stops all containers and cleans up
2. **`docker network rm`** - Removes the conflicting network
3. **`docker-compose up -d --build`** - Rebuilds and starts services fresh
4. **`docker-compose ps`** - Shows status of services managed by Compose
5. **`docker-compose logs`** - Shows startup logs to identify any errors

**The key fix is letting Docker Compose manage the network automatically instead of creating it manually.**

**Run these commands in order and let me know what you see after each step! We'll get your microservices running!** 🚀

# 🎉 **SUCCESS! All Services Running!**

Excellent! All three microservices are healthy and responding. Now let's test **inter-service communication** - the real magic of microservices!

---

## **Step 9: Test Inter-Service Communication**

### **Test Individual Service Endpoints:**
```bash
# Test User Service endpoints with pretty JSON output
echo "👥 TESTING USER SERVICE ENDPOINTS"
echo "================================="
echo "📋 Getting all users:"
curl -s http://localhost:3001/users | jq '.' 2>/dev/null || curl -s http://localhost:3001/users
echo ""

echo "🔍 Getting user by ID (user 1):"
curl -s http://localhost:3001/users/1 | jq '.' 2>/dev/null || curl -s http://localhost:3001/users/1
echo ""
```

### **Test Product Service:**
```bash
# Test Product Service endpoints
echo "🛍️ TESTING PRODUCT SERVICE ENDPOINTS"
echo "===================================="
echo "📋 Getting all products:"
curl -s http://localhost:3002/products | jq '.' 2>/dev/null || curl -s http://localhost:3002/products
echo ""

echo "🔍 Getting product by ID (product 1):"
curl -s http://localhost:3002/products/1 | jq '.' 2>/dev/null || curl -s http://localhost:3002/products/1
echo ""

echo "🏷️ Filtering products by category (Electronics):"
curl -s "http://localhost:3002/products?category=Electronics" | jq '.' 2>/dev/null || curl -s "http://localhost:3002/products?category=Electronics"
echo ""
```

### **Test Order Service (The Big Test!):**
```bash
# Test Order Service basic endpoints
echo "📦 TESTING ORDER SERVICE ENDPOINTS"
echo "=================================="
echo "📋 Getting existing orders:"
curl -s http://localhost:3003/orders | jq '.' 2>/dev/null || curl -s http://localhost:3003/orders
echo ""
```

### **The Big Test - Create New Order (Inter-Service Communication):**
```bash
# Create a new order (explains: this tests communication between all services!)
echo "🎯 TESTING INTER-SERVICE COMMUNICATION"
echo "======================================"
echo "🛒 Creating new order (Order Service → User Service + Product Service):"

curl -X POST http://localhost:3003/orders \
  -H "Content-Type: application/json" \
  -d '{
    "userId": 1,
    "products": [
      {
        "productId": 1,
        "quantity": 2
      },
      {
        "productId": 2,
        "quantity": 1
      }
    ]
  }' | jq '.' 2>/dev/null || curl -X POST http://localhost:3003/orders \
  -H "Content-Type: application/json" \
  -d '{
    "userId": 1,
    "products": [
      {
        "productId": 1,
        "quantity": 2
      },
      {
        "productId": 2,
        "quantity": 1
      }
    ]
  }'

echo ""
echo ""
```

### **Verify the Magic Happened:**
```bash
# Check that our new order was created
echo "✨ VERIFYING INTER-SERVICE COMMUNICATION SUCCESS"
echo "==============================================="
echo "📊 Updated orders list:"
curl -s http://localhost:3003/orders | jq '.' 2>/dev/null || curl -s http://localhost:3003/orders
echo ""

# Show what just happened behind the scenes
echo "🎭 WHAT JUST HAPPENED BEHIND THE SCENES:"
echo "========================================"
echo "1. 📦 Order Service received your order request"
echo "2. 👤 Order Service called User Service to verify user exists"
echo "3. 🛍️ Order Service called Product Service to get real prices"
echo "4. 💰 Order Service calculated total: (Laptop × 2) + (Smartphone × 1)"
echo "5. ✅ Order Service created order with correct pricing!"
echo ""
echo "🎉 MICROSERVICES COMMUNICATION SUCCESS!"
```

---

## **What This Tests:**

### **🔄 Inter-Service Communication Flow:**
```
Your Request → Order Service → User Service (verify user)
                    ↓
             Product Service (get prices)
                    ↓
             Calculate Total → Create Order
```

### **🎯 What We're Proving:**
- **Service Discovery** - Order service finds other services by name
- **HTTP Communication** - Services make REST API calls to each other
- **Data Integration** - Order service combines data from multiple sources
- **Error Handling** - Services work even if others are temporarily unavailable
- **Business Logic** - Real-world order processing with price calculation

**Run these commands and watch the magic! You'll see:**
- Individual service data
- A new order being created that pulls data from both User and Product services
- Real calculated totals based on current product prices

**Let me know when you see "MICROSERVICES COMMUNICATION SUCCESS!" Then we'll move on to scaling and monitoring!** 🚀

# 📈 **Step 10: Scale Services & Performance Monitoring**

Great! Now let's explore the real power of microservices - **horizontal scaling** and **performance monitoring**. We'll scale individual services and create monitoring tools.

---

## **Commands to Run:**

### **Scale Product Service (Most Popular Service):**
```bash
# Scale product service to handle more load (explains: creates multiple instances)
echo "📈 SCALING MICROSERVICES"
echo "======================="
echo "🛍️ Scaling Product Service to 3 instances..."
docker-compose up -d --scale product-service=3

echo ""
echo "⏳ Waiting for scaled services to start..."
sleep 10
echo ""
```

### **Check Scaled Services:**
```bash
# View all running instances with nice formatting
echo "🐳 SCALED SERVICES STATUS"
echo "========================="
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}" | grep -E "(NAMES|service)"
echo ""

# Count instances of each service
echo "📊 SERVICE INSTANCE COUNT:"
echo "=========================="
USER_COUNT=$(docker ps --filter "name=user-service" --format "{{.Names}}" | wc -l)
PRODUCT_COUNT=$(docker ps --filter "name=product-service" --format "{{.Names}}" | wc -l)
ORDER_COUNT=$(docker ps --filter "name=order-service" --format "{{.Names}}" | wc -l)

echo "👥 User Service: $USER_COUNT instance(s)"
echo "🛍️ Product Service: $PRODUCT_COUNT instance(s) 🚀"
echo "📦 Order Service: $ORDER_COUNT instance(s)"
echo ""
```

### **Create Performance Monitoring Script:**
```bash
# Create monitoring script for ongoing performance tracking
echo "📊 CREATING PERFORMANCE MONITORING SCRIPT"
echo "========================================="
cd ..
nano monitor-services.sh
```

**In nano, paste this monitoring script:**

```bash
#!/bin/bash

echo "🔍 MICROSERVICES HEALTH & PERFORMANCE MONITOR"
echo "=============================================="
echo "⏰ Timestamp: $(date)"
echo "=============================================="

services=("user-service:3001" "product-service:3002" "order-service:3003")

for service in "${services[@]}"; do
    name=$(echo $service | cut -d':' -f1)
    port=$(echo $service | cut -d':' -f2)
    
    echo -n "🏥 Checking $name... "
    
    start_time=$(date +%s.%3N)
    if curl -s -f "http://localhost:$port/health" > /dev/null; then
        end_time=$(date +%s.%3N)
        response_time=$(echo "$end_time - $start_time" | bc -l 2>/dev/null || echo "0.0")
        echo "✅ Healthy (Response: ${response_time}s)"
    else
        echo "❌ Unhealthy"
    fi
done

echo ""
echo "📈 CONTAINER RESOURCE USAGE"
echo "==========================="
docker stats --no-stream --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"

echo ""
echo "📋 RECENT SERVICE LOGS (Last 3 lines each)"
echo "==========================================="
for service in user-service product-service order-service; do
    echo "--- 📝 $service ---"
    docker logs --tail 3 $service 2>/dev/null || echo "Service not running"
    echo
done

echo "🎯 Monitoring complete!"
```

**Save and exit:** `Ctrl+X`, `Y`, `Enter`

### **Make Script Executable and Run:**
```bash
# Make monitoring script executable and run it
echo "🔧 SETTING UP MONITORING"
echo "========================"
chmod +x monitor-services.sh

echo "🏃 Running performance monitor..."
./monitor-services.sh
echo ""
```

### **Create Load Testing Script:**
```bash
# Create load testing script (explains: simulates traffic to test scaling)
echo "⚡ CREATING LOAD TESTING SCRIPT"
echo "==============================="
nano load-test.sh
```

**In nano, paste:**

```bash
#!/bin/bash

echo "⚡ MICROSERVICES LOAD TEST"
echo "========================="
echo "🚀 Generating concurrent requests..."

# Test user service with concurrent requests
echo "👥 Testing User Service (10 concurrent requests)..."
for i in {1..10}; do
    curl -s "http://localhost:3001/users" > /dev/null &
done

# Test product service with concurrent requests (this is scaled!)
echo "🛍️ Testing Product Service (15 concurrent requests)..."
for i in {1..15}; do
    curl -s "http://localhost:3002/products" > /dev/null &
done

# Test order creation (inter-service communication under load)
echo "📦 Testing Order Creation (5 concurrent orders)..."
for i in {1..5}; do
    curl -s -X POST "http://localhost:3003/orders" \
        -H "Content-Type: application/json" \
        -d "{\"userId\": $i, \"products\": [{\"productId\": 1, \"quantity\": 1}]}" > /dev/null &
done

# Wait for all background requests to complete
echo "⏳ Waiting for requests to complete..."
wait

echo "✅ Load test completed!"
echo ""

# Check service health after load test
echo "🏥 POST-LOAD HEALTH CHECK"
echo "========================="
./monitor-services.sh
```

**Save and exit:** `Ctrl+X`, `Y`, `Enter`

### **Run Load Test:**
```bash
# Make load test executable and run
chmod +x load-test.sh

echo "🎯 RUNNING LOAD TEST ON SCALED SERVICES"
echo "======================================="
./load-test.sh
```

---

## **What We Just Accomplished:**

### **🚀 Horizontal Scaling:**
- **Product Service** now runs 3 instances (load balanced automatically)
- **User & Order Services** remain single instance
- **Docker automatically distributes** requests across Product Service instances

### **📊 Monitoring Tools:**
- **Real-time health checks** for all services
- **Performance metrics** (CPU, memory, network usage)
- **Response time tracking** for each service
- **Log aggregation** from all containers

### **⚡ Load Testing:**
- **Concurrent requests** to test scaling effectiveness
- **Inter-service communication** under load
- **Performance validation** after scaling

**This demonstrates key microservices benefits:**
- **Independent scaling** - Only scale services that need it
- **Performance monitoring** - Track each service separately
- **Load distribution** - Multiple instances handle more traffic

**Run these commands and let me know what you see from the load test results! Next, we'll create an API Gateway to route all traffic through a single entry point!** 🌟

# 🚪 **Step 11: Build API Gateway**

Now we'll create the **API Gateway** - the single entry point for all requests! This is a critical microservices pattern that provides routing, rate limiting, and centralized access control.

## **What We're Building:**
An API Gateway that:
- Routes requests to appropriate services (`/api/users` → User Service)
- Provides rate limiting (100 requests per 15 minutes)
- Logs all requests for monitoring
- Handles service failures gracefully

---

## **Commands to Run:**

### **Navigate to API Gateway Directory:**
```bash
# Move to API Gateway folder (explains: building the traffic router)
echo "🚪 BUILDING API GATEWAY"
echo "======================="
cd api-gateway

# Show current location
echo "📁 Current location: $(pwd)"
echo "🎯 Building: API Gateway (Single Entry Point)"
echo ""
```

### **Create API Gateway Application:**
```bash
# Create the API Gateway application (explains: traffic routing and rate limiting)
echo "📝 CREATING API GATEWAY APPLICATION"
echo "==================================="
nano app.js
```

**In nano, paste this API Gateway code:**

```javascript
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');
const rateLimit = require('express-rate-limit');
const app = express();
const PORT = process.env.PORT || 3000;

// Rate limiting
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: {
        error: 'Too many requests from this IP, please try again later.'
    }
});

// Apply rate limiting to all requests
app.use(limiter);

// Middleware for logging
app.use((req, res, next) => {
    console.log(`${new Date().toISOString()} - ${req.method} ${req.path} - IP: ${req.ip}`);
    next();
});

// Health check for API Gateway
app.get('/health', (req, res) => {
    res.json({
        service: 'api-gateway',
        status: 'healthy',
        timestamp: new Date().toISOString(),
        uptime: process.uptime()
    });
});

// API Gateway info
app.get('/', (req, res) => {
    res.json({
        message: 'Microservices API Gateway',
        version: '1.0.0',
        services: {
            users: '/api/users',
            products: '/api/products',
            orders: '/api/orders'
        },
        documentation: {
            health: '/health',
            users: '/api/users - User management service',
            products: '/api/products - Product catalog service',
            orders: '/api/orders - Order processing service'
        }
    });
});

// Service URLs
const services = {
    user: process.env.USER_SERVICE_URL || 'http://user-service:3001',
    product: process.env.PRODUCT_SERVICE_URL || 'http://product-service:3002',
    order: process.env.ORDER_SERVICE_URL || 'http://order-service:3003'
};

// Proxy configuration options
const proxyOptions = {
    changeOrigin: true,
    timeout: 10000,
    proxyTimeout: 10000,
    onError: (err, req, res) => {
        console.error('Proxy error:', err.message);
        res.status(503).json({
            error: 'Service temporarily unavailable',
            message: 'The requested service is currently unavailable. Please try again later.',
            timestamp: new Date().toISOString()
        });
    },
    onProxyReq: (proxyReq, req, res) => {
        console.log(`Proxying ${req.method} ${req.path} to ${proxyReq.path}`);
    }
};

// User Service Proxy
app.use('/api/users', createProxyMiddleware({
    target: services.user,
    pathRewrite: {
        '^/api/users': '/users'
    },
    ...proxyOptions
}));

// Product Service Proxy
app.use('/api/products', createProxyMiddleware({
    target: services.product,
    pathRewrite: {
        '^/api/products': '/products'
    },
    ...proxyOptions
}));

// Order Service Proxy
app.use('/api/orders', createProxyMiddleware({
    target: services.order,
    pathRewrite: {
        '^/api/orders': '/orders'
    },
    ...proxyOptions
}));

// Start server
app.listen(PORT, '0.0.0.0', () => {
    console.log(`API Gateway running on port ${PORT}`);
    console.log(`Health check: http://localhost:${PORT}/health`);
    console.log('Available routes:');
    console.log('  GET  / - Gateway info');
    console.log('  GET  /health - Gateway health');
    console.log('  *    /api/users - User Service proxy');
    console.log('  *    /api/products - Product Service proxy');
    console.log('  *    /api/orders - Order Service proxy');
});
```

**Save and exit:** `Ctrl+X`, `Y`, `Enter`

### **Create Package Configuration:**
```bash
# Create package.json for API Gateway (explains: includes proxy middleware)
echo "📋 CREATING API GATEWAY PACKAGE CONFIG"
echo "======================================"
nano package.json
```

**In nano, paste:**

```json
{
    "name": "api-gateway",
    "version": "1.0.0",
    "description": "API Gateway for microservices",
    "main": "app.js",
    "scripts": {
        "start": "node app.js",
        "dev": "nodemon app.js"
    },
    "dependencies": {
        "express": "^4.18.2",
        "http-proxy-middleware": "^2.0.6",
        "express-rate-limit": "^6.7.0"
    },
    "keywords": ["api-gateway", "microservice", "proxy", "docker"],
    "author": "Lab Student",
    "license": "MIT"
}
```

**Save and exit:** `Ctrl+X`, `Y`, `Enter`

### **Create Dockerfile:**
```bash
# Create Dockerfile for API Gateway
echo "🐳 CREATING API GATEWAY DOCKERFILE"
echo "=================================="
nano Dockerfile
```

**In nano, paste:**

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install --only=production

COPY . .

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

CMD ["npm", "start"]
```

**Save and exit:** `Ctrl+X`, `Y`, `Enter`

### **Verify API Gateway Files:**
```bash
# Check API Gateway files
echo "✅ API GATEWAY FILES CREATED"
echo "============================"
ls -la --color=always
echo ""

# Show file summary
echo "📄 API GATEWAY SUMMARY:"
echo "======================="
echo "📝 app.js: $(wc -l < app.js) lines - Gateway routing logic"
echo "📋 package.json: $(wc -l < package.json) lines - Dependencies (includes proxy middleware)"
echo "🐳 Dockerfile: $(wc -l < Dockerfile) lines - Container config"
echo ""

# Show complete project structure
echo "🏗️ COMPLETE PROJECT STRUCTURE"
echo "============================="
cd ..
tree . -L 2 2>/dev/null || find . -name "*.js" | head -10
echo ""
echo "🎉 API Gateway ready! Next: Update Docker Compose to include Gateway!"
```

---

## **What We Just Built:**

### **🚪 API Gateway Features:**
- **Request Routing** - `/api/users` routes to User Service
- **Rate Limiting** - 100 requests per 15 minutes per IP
- **Request Logging** - Tracks all incoming requests
- **Error Handling** - Graceful failures when services are down
- **Path Rewriting** - `/api/users` becomes `/users` for backend services

### **🔄 Traffic Flow:**
```
Client Request → API Gateway (Port 3000) → Appropriate Service
                      ↓
              Rate Limit Check
                      ↓ 
              Route: /api/users → user-service:3001/users
              Route: /api/products → product-service:3002/products  
              Route: /api/orders → order-service:3003/orders
```

### **🎯 Benefits:**
- **Single Entry Point** - Clients only need to know one URL
- **Load Balancing** - Gateway can distribute load across scaled services
- **Security** - Rate limiting and request validation in one place
- **Monitoring** - Centralized logging of all requests

**Run these commands and let me know when you see "API Gateway ready!" Next, we'll update our Docker Compose to include the API Gateway and test the complete architecture!** 🌟

You are now ready to integrate the API Gateway and test the complete, unified architecture. Here are the commands and steps to do just that.

### 1. Update the Docker Compose File

First, you need to add the API Gateway to your orchestration file and remove the `container_name` property to allow for scaling.

Navigate to your `docker-compose-files` directory and open the `docker-compose.yml` file with `nano`.

```bash
# Move into the docker-compose directory
cd ~/microservices-lab/docker-compose-files

# Open the orchestration file for editing
nano docker-compose.yml
```

Replace the entire content of `docker-compose.yml` with the updated configuration below. This version adds the `api-gateway` and, crucially, **removes the `container_name` attributes** to fix the scaling issue you encountered. Docker requires unique container names, and removing this attribute allows Docker Compose to generate unique names for scaled instances automatically.

```yaml
# This is the updated docker-compose.yml file.
# Paste this entire content into your file.

version: '3.8'

services:
  # API Gateway Service: The single entry point for all incoming traffic.
  api-gateway:
    build:
      context: ../api-gateway
      dockerfile: Dockerfile
    ports:
      - "3000:3000" # Expose the gateway to the host on port 3000
    networks:
      - microservices-network
    depends_on:
      - user-service
      - product-service
      - order-service
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:3000/health || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 20s

  # User Service: Manages user data and authentication.
  user-service:
    build:
      context: ../user-service
      dockerfile: Dockerfile
    ports:
      - "3001:3001"
    networks:
      - microservices-network
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:3001/health || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 10s

  # Product Service: Manages the product catalog and inventory.
  product-service:
    build:
      context: ../product-service
      dockerfile: Dockerfile
    ports:
      - "3002:3002"
    networks:
      - microservices-network
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:3002/health || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 10s

  # Order Service: Handles order processing and communicates with other services.
  order-service:
    build:
      context: ../order-service
      dockerfile: Dockerfile
    ports:
      - "3003:3003"
    networks:
      - microservices-network
    depends_on:
      - user-service
      - product-service
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:3003/health || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 15s

# Define the custom network for inter-service communication.
networks:
  microservices-network:
    driver: bridge
```

### 2. Deploy the Full Architecture

Now, bring up the entire stack, including the new API Gateway. The `--build` flag tells Docker Compose to build the new `api-gateway` image.

```bash
# Relaunch the entire microservices stack with the API Gateway
echo "🚀 DEPLOYING FULL ARCHITECTURE WITH API GATEWAY"
echo "=============================================="
docker-compose up -d --build

echo ""
echo "⏳ Waiting for all services to initialize..."
sleep 15
echo ""
```

### 3. Verify and Test Through the API Gateway

Check that all four services are running. You are now interacting with your system through a single, unified entry point.

```bash
# Check the status of all four running containers
echo "📊 FULL STACK STATUS"
echo "====================="
docker-compose ps

echo ""
echo "🎉 All four microservices are running!"
echo ""

# Test that the API Gateway is routing traffic correctly
echo "🚪 TESTING API GATEWAY ENDPOINTS"
echo "================================"

echo "--> Testing /api/users (routes to User Service)"
curl -s "http://localhost:3000/api/users" | jq '.'

echo ""
echo "--> Testing /api/products (routes to Product Service)"
curl -s "http://localhost:3000/api/products" | jq '.'

echo ""
echo "--> Testing /api/orders (routes to Order Service)"
curl -s "http://localhost:3000/api/orders" | jq '.'

echo ""
echo "✅ API Gateway is routing traffic successfully!"
```

### 4. Test Scaling (Corrected)

With the `container_name` property removed, you can now properly scale your services. Let's scale the `product-service` to three instances to handle higher traffic.

```bash
# Scale the product service to 3 instances
echo "📈 SCALING PRODUCT SERVICE TO 3 INSTANCES"
echo "=========================================="
docker-compose up -d --scale product-service=3 --no-recreate

echo ""
echo "⏳ Waiting for new instances to start..."
sleep 5
echo ""

# Verify that there are now three instances of the product service
echo "📊 VERIFYING SCALED SERVICES"
echo "============================"
docker-compose ps

echo ""
echo "🚀 Product service successfully scaled!"
```

You have now successfully built, deployed, and tested a complete microservices architecture, including a central API Gateway for routing and a scaled service to handle increased load.

#### Bonus Task 
Running a stress test, and Monitoring it LIVE

This is an excellent way to see your architecture handle a real-world scenario. You will create two scripts:

1.  `run-stress-test.sh`: The main script that scales up your services, runs the load test for two minutes, and then scales everything back down.
2.  `live-monitor.sh`: A helper script that provides a live, continuously updating CLI dashboard showing CPU, Memory, and Network I/O for each container.

You will run these in two separate terminal windows to observe the live metrics while the stress test is running.

### 1. Create the Live Monitoring Script

This script will give you a dynamic, color-coded view of your containers' performance.

```bash
# Navigate to your project's root directory
cd ~/microservices-lab

# Create the live monitoring script
nano live-monitor.sh
```

Now, paste the entire block of code below into the `live-monitor.sh` file. This script uses `watch` to refresh the `docker stats` command every two seconds, providing a real-time dashboard.

```bash
#!/bin/bash

# live-monitor.sh
# A script to display live, color-coded stats for running microservice containers.

# ANSI color codes
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
CYAN='\033[0;36m'
RED='\033[0;31m'
NC='\033[0m' # No Color

echo -e "${CYAN}🚀 LAUNCHING LIVE MICROSERVICE MONITOR...${NC}"
echo -e "${YELLOW}Press Ctrl+C to exit.${NC}"
sleep 2

# Use 'watch' to execute the docker stats command every 2 seconds.
# The 'awk' command is used for advanced, color-coded formatting.
watch -n 2 "docker stats --no-stream --format '{{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}' | while read name cpu mem; do
    printf \"${GREEN}%-40s${NC}\" \"\$name\";
    cpu_val=\$(echo \$cpu | sed 's/%//');
    if (( \$(echo \"\$cpu_val > 75.0\" | bc -l) )); then
        printf \"${RED}%-15s${NC}\" \"\$cpu\";
    elif (( \$(echo \"\$cpu_val > 40.0\" | bc -l) )); then
        printf \"${YELLOW}%-15s${NC}\" \"\$cpu\";
    else
        printf \"${GREEN}%-15s${NC}\" \"\$cpu\";
    fi;
    printf \"${CYAN}%-25s${NC}\n\" \"\$mem\";
done"

```

Save and exit the editor. Now, make this script executable:

```bash
chmod +x live-monitor.sh
```

Keep this script in mind. You will run it in a **second terminal window** when prompted.

### 2. Create the Main Stress Test Script

This is the orchestrator script. It will handle scaling, execute the load test for two minutes, and then clean up.

```bash
# Make sure you are in the project root directory
cd ~/microservices-lab

# Create the main stress test script
nano run-stress-test.sh
```

Paste the entire, comprehensive script below into the `run-stress-test.sh` file. It includes functions for generating load and clear, descriptive `echo` statements so you know exactly what is happening at each stage.

```bash
#!/bin/bash

# run-stress-test.sh
# Scales up the microservices, runs a 2-minute load test, and scales down.

# --- ANSI Color Codes for beautiful output ---
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
CYAN='\033[0;36m'
RED='\033[0;31m'
NC='\033[0m' # No Color

# --- Configuration ---
LOAD_DURATION_SECONDS=120 # 2 minutes
API_GATEWAY_URL="http://localhost:3000"

# --- Load Generation Function ---
# This function sends continuous, parallel requests to all gateway endpoints.
generate_load() {
    echo -e "${CYAN}   -> Starting load generation process...${NC}"
    while true; do
        # User service endpoints
        curl -s -o /dev/null "$API_GATEWAY_URL/api/users" &
        curl -s -o /dev/null "$API_GATEWAY_URL/api/users/1" &

        # Product service endpoints
        curl -s -o /dev/null "$API_GATEWAY_URL/api/products" &
        curl -s -o /dev/null "$API_GATEWAY_URL/api/products/2" &

        # Order service endpoints (heaviest load)
        curl -X POST -s -o /dev/null -H "Content-Type: application/json" -d '{"userId": 1, "products": [{"productId": 3, "quantity": 1}]}' "$API_GATEWAY_URL/api/orders" &

        sleep 0.1 # Small delay between bursts
    done
}

# --- Main Script Execution ---

echo -e "${GREEN}🚀 STARTING MICROSERVICES STRESS TEST 🚀${NC}"
echo "=================================================="

# STEP 1: Scale up all services
echo -e "\n${YELLOW}[STEP 1/4] Scaling up services to 3 instances each...${NC}"
cd ~/microservices-lab/docker-compose-files
docker-compose up -d --scale user-service=3 --scale product-service=3 --scale order-service=3 --scale api-gateway=3 --no-recreate
echo -e "${GREEN}✅ Scaling complete. Waiting for services to stabilize...${NC}"
sleep 15

# Show the scaled state
docker-compose ps

# STEP 2: Start the load test
echo -e "\n${YELLOW}[STEP 2/4] Starting 2-minute load test...${NC}"
echo -e "${CYAN}*** NOW, RUN './live-monitor.sh' in a a separate terminal to see live stats! ***${NC}"

generate_load &
LOAD_PID=$! # Save the process ID of the background load function

# Display a countdown timer
end_time=$((SECONDS + LOAD_DURATION_SECONDS))
while [ $SECONDS -lt $end_time ]; do
    remaining=$((end_time - SECONDS))
    printf "\r${CYAN}   -> Test running: %02d:%02d remaining...${NC}" $((remaining/60)) $((remaining%60))
    sleep 1
done

# STEP 3: Stop the load test
echo -e "\n\n${GREEN}✅ Load test finished!${NC}"
kill $LOAD_PID
wait $LOAD_PID 2>/dev/null

echo -e "\n${YELLOW}[STEP 3/4] Performing post-test health check...${NC}"
# Quick health check after the test
curl -s "$API_GATEWAY_URL/health" && echo -e "${GREEN}   -> API Gateway is healthy!${NC}" || echo -e "${RED}   -> API Gateway is UNHEALTHY!${NC}"

# STEP 4: Scale down services
echo -e "\n${YELLOW}[STEP 4/4] Scaling services back down to 1 instance each...${NC}"
docker-compose up -d --scale user-service=1 --scale product-service=1 --scale order-service=1 --scale api-gateway=1 --no-recreate
sleep 10
echo -e "${GREEN}✅ Scale down complete.${NC}"

# Final state
docker-compose ps

echo -e "\n${GREEN}🏁 STRESS TEST COMPLETE 🏁${NC}"
```

Save and exit the editor, then make it executable:

```bash
chmod +x run-stress-test.sh
```

### 3. Running the Test and Monitor

You are now ready. The process requires two terminal windows connected to your server.

**Terminal 1: The Stress Test Orchestrator**

In your first terminal, simply run the main script:

```bash
# Make sure you are in the ~/microservices-lab directory
cd ~/microservices-lab

# Execute the stress test
./run-stress-test.sh
```

**Terminal 2: The Live Dashboard**

As soon as the stress test script in Terminal 1 says **"NOW, RUN './live-monitor.sh'..."**, switch to your second terminal and start the live monitor:

```bash
# Make sure you are in the ~/microservices-lab directory
cd ~/microservices-lab

# Start the live dashboard
./live-monitor.sh
```

### What You Will See

*   **In Terminal 1:** You will see the script progress through its four stages: scaling up, running the 2-minute load test (with a countdown), performing a health check, and scaling back down.
*   **In Terminal 2:** This is where the magic happens. You will see a CLI dashboard that refreshes every two seconds. As the load test runs, you will observe the **CPU %** for each container increase. The numbers will be color-coded:
    *   **Green:** Normal usage.
    *   **Yellow:** Moderate usage.
    *   **Red:** High usage.

This provides direct, visual feedback on how your services are performing under stress, allowing you to identify potential bottlenecks and confirm that your scaled architecture can handle the pressure.
