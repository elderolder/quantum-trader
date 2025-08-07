# Tools, Frameworks & Services

This document provides a comprehensive, categorized list of the tools, platforms, and services used in the Quantum Trader Pro application.

### 🧠 AI & Prompt Engineering

| Tool / Service | Description |
| :--- | :--- |
| **Gemini Build (Google AI)** | AI copilot used for building the application via natural language interfaces. |
| **Gemini Pro API** | External LLM used for conversational assistance and prompt generation. |
| **Mixtral 8x7B API** | High-performance model for internal AI strategy generation. |
| **LangChain** | Framework for building AI agents, managing prompts, memory, and tool usage. |

### 💻 Development Environment

| Tool / Service | Description |
| :--- | :--- |
| **GitHub** | Source control, collaboration, CI/CD base. |
| **GitHub Codespaces** | Cloud-based dev environment where the app runs and builds. |
| **Docker / Docker Compose** | Containerized services for reproducible dev environments. |

### 🧩 Backend

| Tool / Library | Description |
| :--- | :--- |
| **FastAPI** | Python backend framework for APIs and WebSocket endpoints. |
| **Redis** | In-memory data store for caching and real-time messaging (Pub/Sub). |
| **Socket.IO (Python)** | WebSocket framework for sending real-time data to the frontend. |
| **Uvicorn** | ASGI server to run FastAPI apps. |
| **Pandas / NumPy** | Data processing for technical indicators. |
| **CCXT** | Library for connecting to cryptocurrency exchanges. |

### ⚛️ Frontend

| Tool / Library | Description |
| :--- | :--- |
| **React** | UI framework for the dashboard and chatbot interfaces. |
| **Vite / Next.js** | Frontend development tooling (build system, SSR). |
| **Socket.IO Client** | Library for real-time connection to backend signals. |

### 📊 Data & Market Infrastructure

| Tool / Service | Description |
| :--- | :--- |
| **TimescaleDB** | Time-series database (PostgreSQL extension) for historical data. |
| **Market Data APIs** | Data feeds for live crypto prices (e.g., CCXT, CoinMarketCap). |
