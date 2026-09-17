# Lodexi AI: Enterprise RAG API-as-a-Service

Lodexi is an enterprise-grade Retrieval-Augmented Generation (RAG) platform designed specifically for B2B SaaS applications. It acts as a highly scalable "Brain" that processes private corporate documents and exposes them via a secure API.

## Core Vision
Instead of building isolated chatbots, Lodexi is built as an **API-first Provider**.
1. **Upload via Portal:** Clients upload PDF/DOCX documents via the Lodexi Laravel Portal.
2. **Retrieve API Key:** Clients obtain an `X-API-Key` specific to their isolated document tenant.
3. **Connect Anywhere:** Clients can plug this API key into WhatsApp bots, Telegram bots, Make.com, n8n, or Google Apps Script to automate responses based *strictly* on their private knowledge base.

## Key Innovations

### 1. Bring Your Own Key (BYOK) Business Model
Lodexi shifts the LLM inference cost entirely to the client. By allowing clients to input their own OpenAI or Google Gemini API keys in the dashboard, the platform operates with **Zero API Overhead** for the SaaS provider, charging solely for RAG infrastructure and document vectorization.

### 2. Semantic Caching (Speed & Cost Optimization)
Lodexi features an integrated semantic cache built on Qdrant.
When a user asks a question, the system vectorizes the query and checks for semantically identical questions asked previously (Cosine Similarity > 95%).
- **Benefit:** If a cache hit occurs, the answer is returned in `<100ms`, completely bypassing the LLM API, saving 100% of token costs.

### 3. Relevance Threshold Filtering (Anti-Hallucination)
Before sending retrieved document chunks to the LLM, the Qdrant vector store enforces a strict `min_score=0.5` threshold.
- **Benefit:** If a user asks a question completely unrelated to the knowledge base (e.g., asking about recipes in a corporate SOP), the vector store returns an empty result, and the system immediately rejects the question without ever pinging the LLM.

### 4. Strict Citation Prompting
The AI is restricted to a "Document Auditor" persona. It is strictly forbidden from using general knowledge and is forced to explicitly cite the document title for every fact it provides.

## Tech Stack
- **Dashboard & Auth:** Laravel (PHP), React, Inertia.js, MySQL
- **Core AI Engine:** FastAPI (Python), Qdrant (Vector Database), Sentence-Transformers, OpenAI / Gemini APIs
