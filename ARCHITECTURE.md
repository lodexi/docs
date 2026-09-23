# Lodexi AI Architecture

## 1. High-Level System Design

Lodexi adopts a decoupled microservice architecture consisting of two primary components:

### A. The Lodexi Portal (Frontend & Identity Provider)
- **Framework:** Laravel (PHP 11), React, Inertia.js.
- **Responsibility:** Handles user authentication, billing, Project (Bot) management, Webhook triggers, API Key generation, and global token tracking.
- **Data Flow:** When a user uploads a document to a Project, Laravel temporarily streams the file to a fast storage buffer and immediately forwards it to the Python Core via HTTP API. It does not store the documents long-term.

### B. The Lodexi Core (AI Engine & Vector Store)
- **Framework:** FastAPI (Python 3.12).
- **Responsibility:** Ingests document chunks, generates embeddings, performs vector similarity search, and synthesizes answers via LLMs.
- **Data Flow:** Exposes isolated endpoints (e.g., `/api/v1/documents`, `/api/v1/ask`) protected by API keys scoped specifically to each Project.

---

## 2. RAG & Multi-Project Implementation Deep-Dive

### 2.1 Embedding Generation
Lodexi Core utilizes local, on-premise sentence-transformers (`all-MiniLM-L6-v2`) to generate 384-dimensional embeddings. This ensures that sensitive corporate data is not sent to external APIs during the vectorization phase, saving significant costs and ensuring privacy.

### 2.2 Vector Storage & Project Isolation
The system uses Qdrant as the vector database.
- **Project Isolation (Multi-Tenant):** Every point in Qdrant is tagged with an immutable `project_id` payload. All search queries strictly filter by this `project_id`, guaranteeing that Project A can never retrieve Project B's documents, even if they belong to the same User.
- **Storage Mode:** Qdrant is configured to run in persistent disk mode (`settings.QDRANT_MODE="disk"`) for localized deployments, avoiding the need for complex external Docker containers during early stages.

### 2.3 Semantic Caching Mechanism
We implemented an enterprise caching layer using a secondary Qdrant collection (`lodex_semantic_cache`).
- **Flow:** User asks a question -> Question is vectorized -> Searched in the Semantic Cache Collection using Cosine Similarity.
- **Condition:** If a previous query matches with `> 0.95` similarity, the system instantly returns the cached string answer.

### 2.4 Inference, LLM Rotation, and Personas
- **Provider:** Supports OpenAI and Google Gemini APIs (via OpenAI compatibility wrappers).
- **Per-Project Configuration:** Each Project maintains its own LLM configuration (Model Provider, API Key, Temperature) and AI Persona (System Prompt).
- **API Key Rotation:** The system parses a comma-separated list of API keys from the project's settings. It randomly selects a key per request to distribute loads across multiple free-tier accounts, effectively creating a zero-cost inference layer.

---

## 3. Third-Party Webhook Integrations (Google Chat, WA)

Because Lodexi is a middleware orchestrator, clients can easily integrate it into external applications without writing code:

1. Client navigates to **Project Settings -> Trigger**.
2. Lodexi generates a unique Webhook URL: `https://lodexi.com/api/v1/projects/{uuid}/chat/google`.
3. Client registers this URL in the Google Cloud Console (Google Chat API) as an App URL.
4. When a user mentions the Bot in Google Chat, Google sends a JSON POST to Lodexi. Lodexi reads the `uuid`, loads the corresponding Project's documents, LLM settings, and AI Persona, processes the question via the Python Core, and replies natively back to Google Chat.
