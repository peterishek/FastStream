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