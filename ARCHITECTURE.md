# Lodexi AI Architecture

## 1. High-Level System Design

Lodexi adopts a decoupled microservice architecture consisting of two primary components:

### A. The Lodexi Portal (Frontend & Identity Provider)
- **Framework:** Laravel (PHP 11), React, Inertia.js.
- **Responsibility:** Handles user authentication, billing, document ingestion UI, and API Key generation.
- **Data Flow:** When a user uploads a document, Laravel temporarily streams the file to a fast storage buffer and immediately forwards it to the Python Core via HTTP API. It does not store the documents long-term.

### B. The Lodexi Core (AI Engine & Vector Store)
- **Framework:** FastAPI (Python 3.12).
- **Responsibility:** Ingests document chunks, generates embeddings, performs vector similarity search, and synthesizes answers via LLMs.
- **Data Flow:** Exposes isolated endpoints (e.g., `/api/v1/documents`, `/api/v1/ask`) protected by API keys.

---

## 2. RAG Implementation Deep-Dive

### 2.1 Embedding Generation
Lodexi Core utilizes local, on-premise sentence-transformers (`all-MiniLM-L6-v2`) to generate 384-dimensional embeddings. This ensures that sensitive corporate data is not sent to external APIs during the vectorization phase, saving significant costs and ensuring privacy.

### 2.2 Vector Storage & Tenant Isolation
The system uses Qdrant as the vector database.
- **Tenant Isolation:** Every point in Qdrant is tagged with an immutable `tenant_id` payload. All search queries strictly filter by this `tenant_id`, guaranteeing that Company A can never retrieve Company B's documents.
- **Storage Mode:** Qdrant is configured to run in persistent disk mode (`settings.QDRANT_MODE="disk"`) for localized deployments, avoiding the need for complex external Docker containers during early stages.

### 2.3 Semantic Caching Mechanism
We implemented an enterprise caching layer using a secondary Qdrant collection (`lodex_semantic_cache`).
- **Flow:** User asks a question -> Question is vectorized -> Searched in the Semantic Cache Collection using Cosine Similarity.
- **Condition:** If a previous query matches with `> 0.95` similarity, the system instantly returns the cached string answer.

### 2.4 Inference & LLM Rotation
- **Provider:** Supports OpenAI and Google Gemini APIs (via OpenAI compatibility wrappers).
- **API Key Rotation:** The system parses a comma-separated list of API keys from the environment or tenant settings. It randomly selects a key per request to distribute loads across multiple free-tier accounts, effectively creating a zero-cost inference layer.

---

## 3. Third-Party Integration Workflow (AppScript, Telegram, WA)

Because Lodexi Core is an API-first service, clients can easily integrate it into their own automated workflows:

1. Client extracts their `X-API-Key` from the Laravel Dashboard.
2. Client sends a POST request to `https://core.lodexi.com/api/v1/ask`:
   ```json
   {
       "question": "What is the return policy?",
       "limit": 5
   }
   ```
3. The response contains the generated answer and the source citations, which the client can format and send directly to their WhatsApp customer or Telegram channel.
