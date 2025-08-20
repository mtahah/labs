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

Run training:

```bash
docker run --rm \
    -v $(pwd)/models:/app/models \
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

(paste original ModelManager code).

Run:

```bash
docker run --rm \
    -v $(pwd)/models:/app/models \
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
version: "3.9"
services:
  api:
    build: .
    command: python src/ml_api.py
    ports:
      - "5000:5000"
    volumes:
      - ./models:/app/models
    depends_on:
      - postgres

  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: mldb
      POSTGRES_USER: mluser
      POSTGRES_PASSWORD: mlpassword
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
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
