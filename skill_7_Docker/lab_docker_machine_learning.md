# 🚀 Docker for Machine Learning — Enhanced CLI Lab

## 🎯 Objectives

By the end of this lab, you will be able to:

* Create **Dockerfiles optimized for ML environments**
* Build and run **containerized TensorFlow ML models**
* Use **Jupyter Notebooks inside Docker**
* Implement **persistent storage** with Docker volumes
* Deploy **multi-service ML applications** using Docker Compose (API + DB)
* Apply **best practices** for ML containerization

---

## 🖥️ 1. Prerequisites & Environment Setup (⏱ \~5 min)

### System Requirements

* OS: **Ubuntu 20.04 LTS or later**
* CPU: 2 cores minimum
* RAM: 4GB minimum (8GB recommended)
* Disk: 10GB free
* Internet connectivity

### Installed Software (verify with commands):

```bash
# Verify Docker
docker --version
# Expected: Docker version 24.x.x, build XXXXX

# Verify Docker Compose
docker compose version
# Expected: Docker Compose version v2.x.x

# Verify Python
python3 --version
# Expected: Python 3.8 or later
```

💡 If `docker` not found:

```bash
sudo apt-get update && sudo apt-get install -y docker.io docker-compose-plugin
```

Enable non-root Docker use:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

## 📂 2. Project Setup

### Task 1. Create Project Structure (⏱ \~3 min)

```bash
mkdir ml-docker-lab && cd ml-docker-lab
mkdir -p {notebooks,models,data,src}
touch Dockerfile requirements.txt docker-compose.yml
```

Verify:

```bash
tree .
```

Expected:

```
.
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── data/
├── models/
├── notebooks/
└── src/
```

---

### Task 2. Define Dependencies (⏱ \~2 min)

```bash
nano requirements.txt
```

Paste:

```
tensorflow==2.13.0
numpy==1.24.3
pandas==2.0.3
matplotlib==3.7.2
scikit-learn==1.3.0
jupyterlab==4.0.5
flask==2.3.3
psycopg2-binary==2.9.7
```

---

### Task 3. Build Docker Image (⏱ \~5 min)

```bash
nano Dockerfile
```

Paste:

```dockerfile
FROM python:3.9-slim

LABEL maintainer="ML Docker Lab" \
      description="ML environment with TensorFlow + Jupyter + Flask"

ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    JUPYTER_ENABLE_LAB=yes

WORKDIR /app

RUN apt-get update && apt-get install -y \
    build-essential git curl && \
    rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

RUN mkdir -p /app/{notebooks,models,data,src}

COPY . .

RUN useradd -m -u 1000 mluser && \
    chown -R mluser:mluser /app
USER mluser

EXPOSE 8888 5000
CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", "--NotebookApp.token=''", "--NotebookApp.password=''"]
```

Build:

```bash
docker build -t ml-tensorflow:latest .
```

Verify:

```bash
docker images | grep ml-tensorflow
```

Expected:

```
ml-tensorflow   latest   abc12345   2 minutes ago   2.1GB
```

---

## 🧠 3. Containerized Model Training

### Task 4. Simple ML Model (⏱ \~4 min)

```bash
nano src/simple_ml_model.py
```

(paste original code — unchanged).

Paste:

```
#!/usr/bin/env python3
"""
Simple Machine Learning Model for Docker Demo
"""

import numpy as np
import pandas as pd
import tensorflow as tf
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
import matplotlib.pyplot as plt
import os
import pickle

def create_sample_data():
    """Create sample dataset for demonstration"""
    np.random.seed(42)
    
    # Generate synthetic data
    n_samples = 1000
    X = np.random.randn(n_samples, 4)
    
    # Create target variable with some relationship to features
    y = (X[:, 0] * 2 + X[:, 1] * 1.5 - X[:, 2] * 0.5 + X[:, 3] * 0.8 + 
         np.random.randn(n_samples) * 0.1)
    
    # Convert to binary classification
    y = (y > np.median(y)).astype(int)
    
    return X, y

def build_model(input_shape):
    """Build a simple neural network model"""
    model = tf.keras.Sequential([
        tf.keras.layers.Dense(64, activation='relu', input_shape=(input_shape,)),
        tf.keras.layers.Dropout(0.2),
        tf.keras.layers.Dense(32, activation='relu'),
        tf.keras.layers.Dropout(0.2),
        tf.keras.layers.Dense(1, activation='sigmoid')
    ])
    
    model.compile(
        optimizer='adam',
        loss='binary_crossentropy',
        metrics=['accuracy']
    )
    
    return model

def train_model():
    """Main training function"""
    print("Creating sample data...")
    X, y = create_sample_data()
    
    # Split the data
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42
    )
    
    # Scale the features
    scaler = StandardScaler()
    X_train_scaled = scaler.fit_transform(X_train)
    X_test_scaled = scaler.transform(X_test)
    
    print("Building model...")
    model = build_model(X_train.shape[1])
    
    print("Training model...")
    history = model.fit(
        X_train_scaled, y_train,
        epochs=50,
        batch_size=32,
        validation_split=0.2,
        verbose=1
    )
    
    # Evaluate model
    test_loss, test_accuracy = model.evaluate(X_test_scaled, y_test, verbose=0)
    print(f"Test Accuracy: {test_accuracy:.4f}")
    
    # Save model and scaler
    os.makedirs('/app/models', exist_ok=True)
    model.save('/app/models/simple_model.h5')
    
    with open('/app/models/scaler.pkl', 'wb') as f:
        pickle.dump(scaler, f)
    
    print("Model saved successfully!")
    
    return model, history, test_accuracy

if __name__ == "__main__":
    train_model()
```

Run training:

```bash
docker run -it --rm \
    -v $(pwd)/models:/app/models \
    -v $(pwd)/data:/app/data \
    -v $(pwd)/src:/app/src \
    ml-tensorflow:latest \
    python src/simple_ml_model.py
```

Expected key output:

```
Creating sample data...
Building model...
Training model...
Epoch 1/50 ...
...
Test Accuracy: 0.85
Model saved successfully!
```

Verify saved model:

```bash
ls models/
```

Expected:

```
scaler.pkl  simple_model.h5
```

---

## 📓 4. Interactive Jupyter Lab (⏱ \~5 min)

Run Jupyter with **timeout** (auto-stops after 60s instead of Ctrl+C):

```bash
timeout 60 docker run --rm \
    -p 8888:8888 \
    -v $(pwd)/notebooks:/app/notebooks \
    -v $(pwd)/models:/app/models \
    ml-tensorflow:latest
```

Expected startup log:

```
[I 12:00:00.000 ServerApp] Jupyter Server is running at:
http://0.0.0.0:8888/lab
```

Open: [http://localhost:8888/lab](http://localhost:8888/lab)

---

## 💾 5. Persistent Model Storage (⏱ \~4 min)

```bash
nano src/model_manager.py
```

```bash
#!/usr/bin/env python3
"""
Model Management Script for Docker Volumes
"""

import tensorflow as tf
import numpy as np
import pickle
import os
import json
from datetime import datetime

class ModelManager:
    def __init__(self, models_dir="/app/models"):
        self.models_dir = models_dir
        os.makedirs(models_dir, exist_ok=True)
    
    def save_model_with_metadata(self, model, model_name, metadata=None):
        """Save model with metadata to Docker volume"""
        model_path = os.path.join(self.models_dir, f"{model_name}.h5")
        metadata_path = os.path.join(self.models_dir, f"{model_name}_metadata.json")
        
        # Save the model
        model.save(model_path)
        
        # Create metadata
        if metadata is None:
            metadata = {}
        
        metadata.update({
            "model_name": model_name,
            "saved_at": datetime.now().isoformat(),
            "model_path": model_path,
            "tensorflow_version": tf.__version__
        })
        
        # Save metadata
        with open(metadata_path, 'w') as f:
            json.dump(metadata, f, indent=2)
        
        print(f"Model saved: {model_path}")
        print(f"Metadata saved: {metadata_path}")
    
    def load_model_with_metadata(self, model_name):
        """Load model with metadata from Docker volume"""
        model_path = os.path.join(self.models_dir, f"{model_name}.h5")
        metadata_path = os.path.join(self.models_dir, f"{model_name}_metadata.json")
        
        if not os.path.exists(model_path):
            raise FileNotFoundError(f"Model not found: {model_path}")
        
        # Load model
        model = tf.keras.models.load_model(model_path)
        
        # Load metadata
        metadata = {}
        if os.path.exists(metadata_path):
            with open(metadata_path, 'r') as f:
                metadata = json.load(f)
        
        print(f"Model loaded: {model_path}")
        print(f"Metadata: {metadata}")
        
        return model, metadata
    
    def list_saved_models(self):
        """List all saved models in the volume"""
        models = []
        for file in os.listdir(self.models_dir):
            if file.endswith('.h5'):
                model_name = file.replace('.h5', '')
                models.append(model_name)
        
        return models
    
    def save_preprocessing_objects(self, objects_dict, name):
        """Save preprocessing objects like scalers"""
        file_path = os.path.join(self.models_dir, f"{name}_preprocessing.pkl")
        
        with open(file_path, 'wb') as f:
            pickle.dump(objects_dict, f)
        
        print(f"Preprocessing objects saved: {file_path}")
    
    def load_preprocessing_objects(self, name):
        """Load preprocessing objects"""
        file_path = os.path.join(self.models_dir, f"{name}_preprocessing.pkl")
        
        if not os.path.exists(file_path):
            raise FileNotFoundError(f"Preprocessing objects not found: {file_path}")
        
        with open(file_path, 'rb') as f:
            objects = pickle.load(f)
        
        print(f"Preprocessing objects loaded: {file_path}")
        return objects

def demo_model_persistence():
    """Demonstrate model persistence with Docker volumes"""
    print("=== Model Persistence Demo ===")
    
    # Initialize model manager
    manager = ModelManager()
    
    # Create a simple model
    model = tf.keras.Sequential([
        tf.keras.layers.Dense(64, activation='relu', input_shape=(10,)),
        tf.keras.layers.Dense(32, activation='relu'),
        tf.keras.layers.Dense(1, activation='sigmoid')
    ])
    
    model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])
    
    # Create dummy data for demonstration
    X_dummy = np.random.randn(100, 10)
    y_dummy = np.random.randint(0, 2, 100)
    
    # Train briefly
    print("Training demo model...")
    model.fit(X_dummy, y_dummy, epochs=5, verbose=0)
    
    # Save model with metadata
    metadata = {
        "description": "Demo model for Docker volume persistence",
        "input_shape": [10],
        "output_shape": [1],
        "task_type": "binary_classification"
    }
    
    manager.save_model_with_metadata(model, "demo_model", metadata)
    
    # Save preprocessing objects
    from sklearn.preprocessing import StandardScaler
    scaler = StandardScaler()
    scaler.fit(X_dummy)
    
    preprocessing_objects = {
        "scaler": scaler,
        "feature_names": [f"feature_{i}" for i in range(10)]
    }
    
    manager.save_preprocessing_objects(preprocessing_objects, "demo_model")
    
    # List saved models
    print("\nSaved models:")
    for model_name in manager.list_saved_models():
        print(f"  - {model_name}")
    
    # Load model back
    print("\nLoading model...")
    loaded_model, loaded_metadata = manager.load_model_with_metadata("demo_model")
    loaded_preprocessing = manager.load_preprocessing_objects("demo_model")
    
    # Test loaded model
    print("\nTesting loaded model...")
    test_input = np.random.randn(1, 10)
    prediction = loaded_model.predict(test_input, verbose=0)
    print(f"Prediction: {prediction[0][0]:.4f}")
    
    print("\n=== Demo Complete ===")

if __name__ == "__main__":
    demo_model_persistence()
```


Run:


```bash
docker run --rm \
    -u $(id -u):$(id -g) \
    -v $(pwd)/models:/app/models \
    -v $(pwd)/src:/app/src \
    ml-tensorflow:latest \
    python src/model_manager.py
```

Verify:

```bash
ls models/
```

Expected includes:

```
demo_model.h5  demo_model_metadata.json  demo_model_preprocessing.pkl
```

---

## 🌐 6. Multi-Service ML Application with Docker Compose

### Task 6.1 Flask API for Model Serving (⏱ \~4 min)

```bash
nano src/ml_api.py
```

(paste Flask API code from original).

---

### Task 6.2 PostgreSQL DB Setup (⏱ \~4 min)

```bash
nano src/database_setup.py
```

Paste:

```python
#!/usr/bin/env python3
import psycopg2, os, time
def wait_for_db():
    for i in range(10):
        try:
            conn = psycopg2.connect(
                host=os.getenv('DB_HOST','postgres'),
                dbname=os.getenv('DB_NAME','mldb'),
                user=os.getenv('DB_USER','mluser'),
                password=os.getenv('DB_PASSWORD','mlpassword')
            )
            conn.close()
            print("✅ Database ready")
            return True
        except psycopg2.OperationalError:
            print(f"⏳ Waiting... {i+1}/10")
            time.sleep(2)
    return False
if __name__=="__main__":
    if wait_for_db():
        print("Tables can now be created here...")
```

---

### Task 6.3 Docker Compose File (⏱ \~4 min)

```bash
nano docker-compose.yml
```

Paste:

```yaml
version: "3.9" # Or a newer stable version like "3.8" if 3.9 causes issues with your Docker Compose client

services:
  api:
    build: . # Build context is the current directory, where your Dockerfile should be.
             # Ensure your Dockerfile sets WORKDIR /app and installs dependencies.
             # Example Dockerfile snippet:
             # FROM ml-tensorflow:latest
             # WORKDIR /app
             # COPY requirements.txt .
             # RUN pip install -r requirements.txt
             # COPY . /app  # This copies src if you don't mount it.
             # If you mount src (as below), you don't need 'COPY src /app/src' in Dockerfile.

    # IMPORTANT: Run the container as your host user to avoid permission issues
    # with mounted volumes like 'models' and 'src'.
    # You might need to set UID and GID in your shell before running `docker compose up`:
    # export UID=$(id -u) && export GID=$(id -g)
    user: "${UID:-1000}:${GID:-1000}" # Use current host user/group, default to 1000:1000

    command: python src/ml_api.py # Execute your API script
    ports:
      - "5000:5000" # Map host port 5000 to container port 5000

    volumes:
      - ./models:/app/models # Mount host's models dir to container's /app/models for persistence
      - ./src:/app/src     # <--- ADDED: Mount host's src dir for live code changes
      # If your API reads/writes data files, you might also need:
      # - ./data:/app/data

    environment:
      # Pass Postgres connection details to the API service.
      # 'postgres' as the host name works because Docker Compose creates an internal network
      # where services can resolve each other by their names.
      POSTGRES_HOST: postgres
      POSTGRES_DB: ${POSTGRES_DB:-mldb} # Use default or override from .env
      POSTGRES_USER: ${POSTGRES_USER:-mluser}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-mlpassword}
      # Any other environment variables your API needs...

    depends_on:
      - postgres # Ensure the postgres service starts before the api service

  postgres:
    image: postgres:15 # Using a specific version is good practice
    environment:
      # Set these in your .env file or explicitly here for local development
      POSTGRES_DB: ${POSTGRES_DB:-mldb}
      POSTGRES_USER: ${POSTGRES_USER:-mluser}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-mlpassword}
    ports:
      - "5432:5432" # Expose Postgres to your host machine if needed (e.g., for psql client)
    volumes:
      - pgdata:/var/lib/postgresql/data # Mount named volume for persistent Postgres data

volumes:
  pgdata: # Define the named volume
```

---

### Task 6.4 Run Multi-Service App (⏱ \~5 min)

```bash
docker compose up --build
```

Expected logs:

```
api_1       | Model loaded from /app/models/iris_model.h5
postgres_1  | PostgreSQL init process complete; ready for connections
```

Test API (new terminal):

```bash
curl -s http://localhost:5000/health | jq
```

Expected:

```json
{
  "status": "healthy",
  "model_loaded": true,
  "scaler_loaded": false
}
```

---

## ✅ Final Mastery Checklist

Run these to confirm success:

```bash
# Verify ML image
docker images | grep ml-tensorflow

# Verify trained models persisted
ls models/

# Verify API responds
curl -s http://localhost:5000/health
```

---

🔥 Congratulations! You now have a **production-like ML workflow in Docker**:

* TensorFlow model training inside containers
* Jupyter Lab for exploration
* Model persistence with Docker volumes
* Multi-service app (Flask API + PostgreSQL DB)

---
