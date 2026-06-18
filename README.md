# FastStream
FastStream: AI-Powered Asynchronous Video Analytics Engine FastStream is a high-performance, event-driven microservice architecture built to handle concurrent video uploads, asynchronous processing pipelines, and real-time state streaming.


┌───────────────────────┐│   Client Dashboard    │└─────▲───────────┬─────┘│           │(WebSockets)  (HTTP POST Upload)│           ▼┌─────┴─────────────────┐│    FastAPI Gateway    │◄─────► [ Redis Cache ]└─────────┬─────────────┘    (Rate Limits / Sessions)│(Publish Task)▼[ RabbitMQ Broker ]│(Consume Task)▼┌───────────────────────┐│  Celery Task Workers  │◄─────► [ Media Storage ]└───────────────────────┘          (FFmpeg / Analytics)





### Architectural Decisions
* **FastAPI Gateway:** Chosen over Django for its native ASGI asynchronous performance, lower memory footprint, and seamless out-of-the-box support for concurrent persistent WebSocket connections.
* **RabbitMQ Message Broker:** Utilized for enterprise-grade background task guarantees, ensuring zero data loss and robust retry mechanisms for heavy media extraction pipelines.
* **Redis Memory Store:** Dual-purposed as a high-speed data cache to enforce API rate-limiting thresholds and as a fast state-broker for active user sessions.

---

## ✨ Key Features
* ⚡ **Async Chunked Uploads:** Streamlined large file ingestions without blocking the primary server event loop.
* 🔄 **Real-Time WebSocket Updates:** Live progress tracking pushed to clients straight from background processing nodes.
* 🛡️ **Native Data Validation & Security:** Strict payload validation via Pydantic v2 and stateless authentication with OAuth2 JWT.
* 🚦 **Distributed Rate Limiting:** Dynamic API throttling powered by an asynchronous Redis sliding-window middleware.

---

## 🛠️ Tech Stack
* **Core Framework:** Python 3.11+ / FastAPI
* **Task Management:** Celery / RabbitMQ
* **Caching & WebSockets:** Redis
* **Validation & Security:** Pydantic v2 / PyJWT
* **Processing Engine:** FFmpeg

---

## 🚀 Getting Started (Local Setup)

Follow these steps to spin up the entire cluster on your local machine using Docker.

### Prerequisites
Ensure you have the following software installed:
* [Docker Desktop](https://docker.com) (includes Compose v2)
* [git](https://git-scm.com)

### 1. Clone the Repository
```bash
git clone https://github.com
cd faststream-analytics
```

### 2. Configure Environment Variables
Create a `.env` file in the root directory:
```env
PROJECT_NAME="FastStream Analytics"
SECRET_KEY="your-super-secret-immutably-secure-key"
ALGORITHM="HS256"
ACCESS_TOKEN_EXPIRE_MINUTES=30

REDIS_URL="redis://redis:6379/0"
RABBITMQ_URL="amqp://guest:guest@rabbitmq:5672//"
```

### 3. Build and Spin Up Infrastructure
Run the following command to build the custom images and execute the network container orchestration:
```bash
docker compose up --build
```
*Add `-d` at the end if you prefer running it silently in detached background mode.*

---

## 🔍 Verifying the System

Once the Docker containers are healthy, you can interact with the microservices at these entry points:

| Service | Endpoint / URL | Description |
| :--- | :--- | :--- |
| **Interactive API Documentation** | `http://localhost:8000/docs` | Automated Swagger interface to test endpoints |
| **Alternative API Docs** | `http://localhost:8000/redoc` | Clean, readable ReDoc OpenAPI schema |
| **RabbitMQ Management Panel** | `http://localhost:15672` | Monitor message queues (User/Pass: `guest`) |

---

## 🧪 Quick Test Guide

1. Navigate to `http://localhost:8000/docs`.
2. Authenticate using the `/auth/token` endpoint to get your JWT access token.
3. Use the POST `/api/v1/videos/upload` endpoint to push a test `.mp4` file.
4. Open a WebSocket client (or the project's mock UI) pointing to `ws://localhost:8000/api/v1/stream/progress/{video_id}` to watch your Celery worker process processing data in real time.

## 🚀 Getting Started (Local Development)

Follow these steps to set up and run the entire ecosystem directly on your native operating system.

### 📋 Prerequisites

Before running the application, make sure you have the following background services and system dependencies installed:

1. **Python 3.11+**
2. **RabbitMQ Server:** Used as the message broker. 
   * *Mac:* `brew install rabbitmq`
   * *Ubuntu/Debian:* `sudo apt install rabbitmq-server`
   * *Windows:* Install via [Chocolatey](https://chocolatey.org) (`choco install rabbitmq`) or the official installer.
3. **Redis Server:** Used for rate-limiting and session caching.
   * *Mac:* `brew install redis`
   * *Ubuntu/Debian:* `sudo apt install redis-server`
4. **FFmpeg:** System-level multimedia library required by the background workers to process video.
   * *Mac:* `brew install ffmpeg`
   * *Ubuntu/Debian:* `sudo apt install ffmpeg`

Make sure RabbitMQ and Redis are actively running on your machine:
```bash
# Mac/Linux status check examples
brew services start redis
brew services start rabbitmq

# Alternative Linux check
sudo systemctl start redis-server
sudo systemctl start rabbitmq-server
```

---

### 🛠️ Step-by-Step Installation

#### 1. Clone and Navigate to the Repository
```bash
git clone https://github.com
cd faststream-analytics
```

#### 2. Set Up a Python Virtual Environment
Isolate your dependencies using a standard virtual environment:
```bash
# Create the environment
python3 -m venv venv

# Activate the environment
# On macOS/Linux:
source venv/bin/activate
# On Windows (Command Prompt):
venv\Scripts\activate.bat
# On Windows (PowerShell):
.\venv\Scripts\Activate.ps1
```

#### 3. Install Python Dependencies
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

#### 4. Configure Environment Variables
Create a local `.env` file in the root project folder:
```env
PROJECT_NAME="FastStream Analytics"
SECRET_KEY="your-local-development-secret-key-change-in-production"
ALGORITHM="HS256"
ACCESS_TOKEN_EXPIRE_MINUTES=30

# Native localhost service connections
REDIS_URL="redis://localhost:6379/0"
RABBITMQ_URL="amqp://guest:guest@localhost:5672//"
```

---

### 🏃 Running the Application

Because this is an event-driven microservice system, you need to open **two separate terminal windows** (with your virtual environment activated in both) to run the system components.

#### Terminal 1: Launch the FastAPI Web Gateway
This powers the asynchronous REST endpoints, interactive documentation, and WebSocket connection channels.
```bash
uvicorn app.main:app --reload --port 8000
```

#### Terminal 2: Launch the Celery Processing Worker
This worker listens directly to RabbitMQ and executes heavy media tasks (like FFmpeg extractions) in the background.
```bash
celery -A app.worker.celery_app worker --loglevel=info
```

---

## 🐳 Alternative: Run with Container Orchestration (Docker)

If you prefer to skip installing system dependencies like RabbitMQ, Redis, or FFmpeg directly onto your machine, you can spin up the entire pre-configured cluster instantly using Docker:

```bash
# Build the application images and run the infrastructure network
docker compose up --build
```
### 🪟 Special Instructions for Windows Users

Running RabbitMQ and Celery natively on Windows requires specific workarounds due to ecosystem compatibility constraints. Follow these steps carefully:

#### 1. Installing RabbitMQ on Windows
The most reliable native method is using the [Chocolatey Package Manager](https://chocolatey.org):
```powershell
# Run in an administrative PowerShell terminal
choco install rabbitmq
```
*Alternatively, you must install the **Erlang OTP** runtime first, followed by the official **RabbitMQ Windows Installer** executable.*

#### 2. Running Redis on Windows
Redis does not officially support Windows. You must install the latest port via Memurai or use the archived Microsoft Open Tech release:
```powershell
choco install redis-64
redis-server.exe
```

#### 3. Crucial Workaround for Celery on Windows
Celery 5.x **does not officially support Windows** due to limitations with the default pool implementation (`billiard`). If you try to run it normally, your workers will crash instantly with an `AttributeError` or lock up.

**The Fix:** You must force Celery to use the `solo` or `threads` pool execution pool when starting the worker process on Windows.

Launch your worker terminal using this exact flag:
```powershell
# Windows-specific execution command
celery -A app.worker.celery_app worker --loglevel=info -P solo
```
*(Note: The `-P solo` flag forces Celery to process tasks inline on a single thread, bypassing the multi-processing block on Windows. For a true multi-threaded production environment on Windows, utilizing **Docker/WSL2** is highly recommended).*

