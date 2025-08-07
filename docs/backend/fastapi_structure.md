# FastAPI Application Structure

This document outlines the standard file and folder structure for the Python backend, built with the FastAPI framework. Adhering to this structure ensures the project is organized, scalable, and easy to maintain.

**Audience:** You (Infrastructure), Gemini (API Logic)
**Goal:** To provide a clear map for where to place new files, routes, and business logic.

---

### Backend Directory Layout

The `backend/` directory will contain all Python source code, dependencies, and worker scripts.

backend/
├── app/
│ ├── init.py
│ ├── api/
│ │ ├── init.py
│ │ ├── v1/
│ │ │ ├── init.py
│ │ │ ├── endpoints/
│ │ │ │ ├── init.py
│ │ │ │ ├── signals.py # Handles /signals/... routes
│ │ │ │ └── ai.py # Handles /ai/... routes
│ │ │ └── api.py # Main v1 router
│ ├── logic/
│ │ ├── init.py
│ │ ├── quantum_cipher.py # Core logic for Quantum Cipher
│ │ └── quantum_volume.py # Core logic for Volume Profile
│ ├── services/
│ │ ├── init.py
│ │ ├── redis_service.py # Handles all Redis interactions
│ │ └── websocket_manager.py # Manages WebSocket connections
│ └── workers/
│ ├── init.py
│ └── signal_monitor.py # Background worker for real-time signals
│
├── tests/
│ ├── init.py
│ └── test_api.py
│
├── .env.example # Example environment file
├── Dockerfile # Instructions to build the backend container
├── main.py # Main application entry point
└── requirements.txt # Python dependencies


### Core Components Explained

*   **`main.py`:** The root of the application. It initializes the FastAPI app, includes the main API router, sets up middleware (like CORS), and defines startup/shutdown events.
*   **`app/api/v1/`:** This is the versioned API module.
    *   **`endpoints/`:** Each file here represents a group of related API endpoints (e.g., `signals.py` for all signal-related routes).
    *   **`api.py`:** This file aggregates all the endpoint routers from the `endpoints/` directory into a single main router for `/api/v1`.
*   **`app/logic/`:** Contains the "pure" business logic. These are Python scripts that perform calculations (like our signal indicators) and do not depend on the web framework. This makes them highly reusable and testable.
*   **`app/services/`:** Contains modules that interact with external services like Redis or manage persistent connections like WebSockets.
*   **`app/workers/`:** Contains standalone scripts designed to be run as long-running background processes (e.g., in a separate Docker container).

### Execution Flow for an API Request

1.  A request hits `main.py`.
2.  FastAPI routes it to the main router in `app/api/v1/api.py`.
3.  The v1 router passes it to the specific endpoint function in a file within `app/api/v1/endpoints/`.
4.  The endpoint function calls a "logic" or "service" module to perform the actual work.
5.  The result is returned as a JSON response.
