# Configuration & API Keys

Quantum Trader Pro requires API keys and other secrets to connect to exchanges and data providers. To handle these securely, the project uses a `.env` file.

!!! danger "Never Commit Your Secrets"
    The `.env` file should **NEVER** be committed to Git or shared publicly. It contains your private credentials. The project's `.gitignore` file is already configured to ignore this file, but you must always be vigilant.

---

### How to Set Up Your Configuration

1.  **Create the File:** In the root directory of the project, create a new file named `.env`.

2.  **Add Your Keys:** Open the `.env` file and add your keys in the `KEY=VALUE` format. Below is a sample structure you can use as a template.

    ```ini
    # --- Exchange APIs ---
    # Used for fetching market data and executing trades
    BINANCE_API_KEY=your_binance_api_key_here
    BINANCE_API_SECRET=your_binance_api_secret_here

    # --- Data Provider APIs ---
    # (Optional) For additional market data or analytics
    COINMARKETCAP_API_KEY=your_coinmarketcap_api_key

    # --- Notification Systems ---
    # (Optional) For sending alerts
    TELEGRAM_BOT_TOKEN=your_bot_token
    TELEGRAM_CHAT_ID=your_chat_id

    # --- Internal Connections ---
    # This is pre-configured for Docker, no need to change for local dev
    REDIS_URL=redis://redis:6379
    ```

### How the Application Uses These Keys

The Python backend (FastAPI) and various scripts will automatically load these variables from your `.env` file when the application starts. You do not need to hardcode them anywhere in the application logic.
