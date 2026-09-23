# Welcome to Lodexi AI Documentation

Lodexi is an enterprise-grade **Retrieval-Augmented Generation (RAG) platform** and **AI Orchestration middleware** designed for B2B applications. It acts as a highly scalable "Brain" that processes private corporate documents and allows you to build multiple, isolated AI bots (Projects) with custom personas.

## Core Features & Architecture

Instead of building isolated chatbots from scratch, Lodexi is built as a **Multi-Project AI Bot Builder**.

### 1. Multi-Project Isolation (Multi-Tenant)
Each Project you create operates as an independent, isolated bot. API Keys, documents, vector embeddings, and LLM settings are strictly scoped to the Project ID. This guarantees complete data privacy and allows a single company to run multiple specialized bots (e.g., an HR Bot and an IT Support Bot) securely side-by-side.

### 2. Bring Your Own Key (BYOK) Business Model
Lodexi shifts the LLM inference cost entirely to the client. By allowing you to input your own OpenAI or Google Gemini API keys directly into your project settings, the platform operates with **Zero API Overhead** for the SaaS provider, charging solely for RAG infrastructure and document vectorization.

### 3. Zero-Code Integration
Lodexi provides out-of-the-box Webhook URLs for each project, allowing instant, zero-code integration with enterprise messaging platforms like Google Chat, Telegram, and WhatsApp.

---

## Authentication Guide

All programmatic requests to the Lodexi API (such as querying your bot programmatically) require authentication using an API Key scoped to your specific Project.

### How to Generate an API Key

Follow these step-by-step instructions to generate your API Key:

1. **Log in to the Dashboard**
   Navigate to your Lodexi Dashboard and log in with your credentials.

2. **Select Your Project**
   If you have multiple projects, ensure you select the project you want to integrate from the project switcher.

3. **Navigate to API Keys**
   On the left sidebar, click on **Project Settings**, then select the **API Keys** menu.

   ![Navigate to API Keys](./images/nav_api_keys.png) <!-- TODO: Add screenshot here -->

4. **Create a New Key**
   Click the **Create new secret key** button located at the top right of the API Keys page.

   ![Create New Key Button](./images/create_key_button.png) <!-- TODO: Add screenshot here -->

5. **Copy Your Key Safely**
   A modal will appear displaying your new API Key. 
   **Important:** Copy this key immediately and store it securely. For security reasons, you will not be able to view this key again once you close the window.

   ![Copy API Key](./images/copy_api_key.png) <!-- TODO: Add screenshot here -->

### Passing the API Key to the API

Once you have your API Key, you must include it in the `Authorization` header of all your HTTP requests to the Lodexi API. The key must be passed as a **Bearer token**.

Here is how your HTTP headers should look:

```http
Authorization: Bearer YOUR_API_KEY_HERE
Accept: application/json
Content-Type: application/json
```

#### cURL Example:

```bash
curl -X POST "https://lodexi.yourdomain.com/api/v1/ask" \
     -H "Authorization: Bearer YOUR_API_KEY_HERE" \
     -H "Content-Type: application/json" \
     -H "Accept: application/json" \
     -d '{
           "question": "What is the company policy on remote work?"
         }'
```

You are now ready to start sending requests to your isolated Lodexi RAG Knowledge Base! Check out the [Quickstart Guide](./quickstart.md) for more details on available endpoints.
