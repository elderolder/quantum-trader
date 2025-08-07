# React Application Structure

This document outlines the standard file and folder structure for the React frontend application, which was initially scaffolded by Gemini Build. This structure promotes a clean, component-based architecture.

**Audience:** You (Frontend Dev), Gemini (UI Components)
**Goal:** To maintain a consistent and organized structure for the user interface code.

---

### Frontend Directory Layout (`frontend/src/`)

All frontend source code resides within the `src/` directory.

src/
├── App.tsx # Main application component, handles routing
│
├── components/
│ ├── shared/ # Small, reusable components used everywhere
│ │ ├── Card.tsx
│ │ ├── MarketChart.tsx
│ │ └── Spinner.tsx
│ │
│ ├── Header.tsx # The top navigation bar
│ └── Sidebar.tsx # The main left-hand navigation menu
│
├── views/ # Top-level page components (the "screens")
│ ├── DashboardView.tsx # Main market dashboard with charts
│ ├── PortfolioView.tsx # Displays user's portfolio and equity
│ ├── TradingView.tsx # The trading terminal for buys/sells
│ └── AnalysisView.tsx # The AI Analysis Center
│
├── hooks/ # Custom React hooks for shared logic
│ └── usePortfolio.ts # Example: hook to manage portfolio state
│
├── services/ # Modules for interacting with APIs
│ └── geminiService.ts # Functions for calling AI backend endpoints
│
├── constants.ts # App-wide constants (e.g., API URLs)
└── types.ts # TypeScript type definitions (e.g., Portfolio, Trade)


### Core Components Explained

*   **`App.tsx`:** The root component of the application. Its primary job is to set up the main layout (e.g., Sidebar + Main Content area) and handle the client-side routing between different `views`.

*   **`components/`:** This directory holds all the reusable UI building blocks.
    *   **`shared/`:** Contains highly generic components that can be used on any page, such as a styled `Card`, a `Spinner` for loading states, or a generic `MarketChart`.
    *   Larger, more specific components like the `Header` and `Sidebar` live at the top level of `components/`.

*   **`views/`:** Each file in this directory represents a full "page" or "screen" in the application. These components are responsible for the layout of a specific page and typically compose smaller components from the `components/` directory. For example, `DashboardView.tsx` would import and use multiple `MarketChart` components.

*   **`hooks/`:** Custom React hooks provide a way to extract and reuse stateful logic. For example, a `usePortfolio` hook could contain all the logic for fetching, updating, and managing the user's portfolio data, which can then be easily used in both `PortfolioView.tsx` and `TradingView.tsx`.

*   **`services/`:** This is the data layer of the frontend. It contains functions that handle all external API calls. This decouples the UI components from the specifics of `fetch` or `axios`, making the code cleaner and easier to test.

This organized structure makes it easy to locate code, manage state, and scale the application by adding new views and components in a predictable way.
