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
```

You will see a lot of log output as the services start. This is normal.

Step 3: Verify Everything is Running

The system is now live! GitHub Codespaces automatically forwards the necessary ports.

Check the Backend: Open http://localhost:8000 in a new browser tab. You should see {"status":"OK","message":"Quantum Trader Pro Backend is Live!"}.

Check the Frontend: Open http://localhost:3000. You'll see the basic "Quantum Trader Pro UI".

Confirm the Real-Time Connection: With the frontend tab open, check the browser's developer console (usually F12 or Ctrl+Shift+I). You should see the message: ✅ Connected to WebSocket.

🎉 That's it! You now have a live, real-time connection between the frontend and backend.

What's Next?

Curious how all the pieces fit together? For a detailed explanation of the container setup, WebSocket connections, and real-time architecture, dive into the full tutorial.

View the Full Real-Time Setup Tutorial{ .md-button }
