# Getting Started in 5 Minutes

This guide will take you from zero to a fully running Quantum Trader Pro environment, right in your browser.

### Prerequisites

*   A GitHub Account.
*   GitHub Codespaces enabled for your account.

---

### Step 1: Open the Project in Codespaces

This is the easiest step. On the main page of the [GitHub repository](https://github.com/elderolder/quantum-trader), click the green **`< > Code`** button, navigate to the **Codespaces** tab, and click **"Create codespace on main"**.

This will launch a complete, containerized development environment in your browser.

### Step 2: Start All Services

Once your Codespace terminal is ready, run the following single command. This will build the Docker containers for the backend, frontend, and Redis, and then start them all up.

```bash
docker-compose up --build
