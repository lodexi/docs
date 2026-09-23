# Lodexi AI: Multi-Tenant, Multi-Project AI Bot Builder

Lodexi is an enterprise-grade Retrieval-Augmented Generation (RAG) platform and AI Orchestration middleware designed for B2B applications. It acts as a highly scalable "Brain" that processes private corporate documents and allows you to build multiple, isolated AI bots (Projects) with custom personas.

## Core Vision
Instead of building isolated chatbots from scratch, Lodexi is built as a **Multi-Project AI Bot Builder**.
1. **Create Projects:** Users can create multiple Projects (Bots) within their workspace.
2. **Upload Knowledge:** Clients upload PDF/DOCX documents specifically scoped to each Project.
3. **Configure Persona:** Each Project can have its own AI Persona (System Prompt) and LLM Model settings.
4. **Connect Anywhere:** Each Project generates its own isolated `X-API-Key` and Webhook Endpoints. Connect your Project directly to Google Chat, WhatsApp, Telegram, or custom APIs.

## Key Innovations

### 1. Bring Your Own Key (BYOK) Business Model
Lodexi shifts the LLM inference cost entirely to the client. By allowing clients to input their own OpenAI or Google Gemini API keys per project, the platform operates with **Zero API Overhead** for the SaaS provider, charging solely for RAG infrastructure and document vectorization.

### 2. Multi-Project Isolation
Each Project operates as an independent tenant. API Keys, documents, vector embeddings, and LLM settings are strictly scoped to the Project ID. This guarantees complete data privacy and allows a single company to run multiple specialized bots (e.g., an HR Bot and an IT Support Bot) securely.

### 3. Webhook & Integration Ready
Lodexi provides out-of-the-box Webhook URLs for each project, allowing instant, zero-code integration with enterprise messaging platforms like Google Chat.

### 4. Semantic Caching (Speed & Cost Optimization)
Lodexi features an integrated semantic cache built on Qdrant.
When a user asks a question, the system vectorizes the query and checks for semantically identical questions asked previously.
- **Benefit:** If a cache hit occurs, the answer is returned instantly, bypassing the LLM API and saving 100% of token costs.

## Tech Stack
- **Dashboard & Auth:** Laravel (PHP), React, Inertia.js, MySQL
- **Core AI Engine:** FastAPI (Python), Qdrant (Vector Database), Sentence-Transformers, OpenAI / Gemini APIs
