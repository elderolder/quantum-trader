# Full Setup: Real-Time Architecture in Codespaces

This document outlines the exact steps to set up the full development environment in GitHub Codespaces to support the entire real-time application stack.

### Overview

This tutorial covers:

*   ⚡ Real-time AI signal generation
*   🌐 FastAPI + Socket.IO WebSocket backend
*   🔁 React frontend with a WebSocket client
*   🔧 A complete, containerized dev environment for scalability

### Prerequisites

*   You have completed the [5-Minute Quickstart](../getting-started/quickstart.md).
*   You have a basic understanding of Docker and `docker-compose`.

---

### 1. Project Structure (Required Layout)

For the system to work correctly, your repository must follow this structure:

quantum-trader/
├── .devcontainer/
│ └── devcontainer.json
├── docker-compose.yml
├── backend/
│ ├── main.py
│ ├── requirements.txt
│ └── app/
│ └── logic/
│ └── cipher.py
└── frontend/
├── index.html
├── main.jsx
└── package.json


### 2. Configure Codespaces: `.devcontainer/devcontainer.json`

This file defines the development container, telling Codespaces how to build the environment.

```json
{
  "name": "quantum-trader-dev",
  "dockerComposeFile": "docker-compose.yml",
  "service": "backend",
  "workspaceFolder": "/workspace/backend",
  "settings": {
    "terminal.integrated.shell.linux": "/bin/bash"
  },
  "extensions": [
    "ms-python.python",
    "ms-azuretools.vscode-docker",
    "esbenp.prettier-vscode"
  ],
  "postCreateCommand": "pip install -r requirements.txt"
}
```

3. Container Composition: docker-compose.yml

This file orchestrates all the services required for the application: the Python backend, the React frontend, and the Redis cache.

```yaml
services:
  backend:
    build:
      context: ./backend
    container_name: fastapi_app
    ports:
      - "8000:8000"
    volumes:
      - ./backend:/workspace/backend
    depends_on:
      - redis
    environment:
      - REDIS_URL=redis://redis:6379
    command: uvicorn main:app --host 0.0.0.0 --port 8000 --reload

  redis:
    image: redis:7-alpine
    container_name: redis_cache
    ports:
      - "6379:6379"

  frontend:
    build:
      context: ./frontend
    container_name: react_ui
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
    command: npm run dev
```

4. FastAPI App Setup (with WebSocket)

The core of the backend is the main.py file, which sets up the FastAPI application and the Socket.IO manager for real-time communication.

```python
# located in backend/main.py
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from fastapi_socketio import SocketManager
import pandas as pd

from app.logic.cipher import fetch_market_data, calculate_quantum_cipher

app = FastAPI(title="Quantum Trader Pro API")
sio = SocketManager(app=app)

# CORS for dev
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
def health_check():
    return {"status": "OK", "message": "Quantum Trader Pro Backend is Live!"}

@sio.on('connect')
async def connect(sid, environ):
    print(f"[WebSocket] Client connected: {sid}")
    await sio.emit('message', {'message': 'Connected to Quantum Trader WS'}, to=sid)

@app.get("/ws-test")
async def test_socket():
    await sio.emit("new_signal", {"type": "test", "data": "🚨 Test signal from backend!"})
    return {"status": "Signal emitted"}
```

This setup provides a health check endpoint, a WebSocket connection handler, and a test route to verify that real-time messages can be sent.
