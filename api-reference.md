# API Endpoint Reference

This document provides a comprehensive reference for the primary endpoints available in the Lodexi AI API. All requests require authentication using your Project-specific API Key passed via the `Authorization: Bearer` header.

Base URL: `https://lodexi.yourdomain.com`

---

## 1. Chat & Retrieval: `POST /api/v1/ask`

This endpoint allows you to send a natural language question to your isolated Knowledge Base. The AI will retrieve the most relevant documents and generate an answer based strictly on that context, guided by your Project's AI Persona.

### Request Schema

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `question` | string | Yes | The natural language question or prompt you want to send to the bot. |
| `limit` | integer | No | The maximum number of document chunks to retrieve for context. Default is `4`. Max is `10`. |
| `category` | string | No | Optional metadata filter. If provided, the vector search will only retrieve documents tagged with this specific category. |

#### JSON Request Example
```json
{
  "question": "What is the company policy on remote work?",
  "limit": 3,
  "category": "HR_Policies"
}
```

### Response Schema

#### JSON Response Example
```json
{
  "project_id": "proj_a1b2c3d4",
  "question": "What is the company policy on remote work?",
  "answer": "Based on the Employee Handbook, employees are allowed to work remotely for up to 2 days a week with prior manager approval.",
  "grounded": true,
  "cached": false,
  "citations": [
    {
      "document_id": "doc_83j2",
      "title": "Employee_Handbook_2024.pdf",
      "snippet": "Section 4.1: Remote Work. Employees are allowed to work remotely for up to 2 days a week...",
      "score": 0.892
    }
  ],
  "usage": {
    "prompt_tokens": 420,
    "completion_tokens": 35,
    "total_tokens": 455
  }
}
```

### Code Snippets

**cURL**
```bash
curl -X POST "https://lodexi.yourdomain.com/api/v1/ask" \
     -H "Authorization: Bearer YOUR_API_KEY_HERE" \
     -H "Content-Type: application/json" \
     -H "Accept: application/json" \
     -d '{
           "question": "What is the company policy on remote work?",
           "limit": 3
         }'
```

**JavaScript / Fetch**
```javascript
async function askQuestion() {
  const response = await fetch("https://lodexi.yourdomain.com/api/v1/ask", {
    method: "POST",
    headers: {
      "Authorization": "Bearer YOUR_API_KEY_HERE",
      "Content-Type": "application/json",
      "Accept": "application/json"
    },
    body: JSON.stringify({
      question: "What is the company policy on remote work?",
      limit: 3
    })
  });
  
  const data = await response.json();
  console.log(data.answer);
}
```

---

## 2. Ingesting Documents: `POST /api/v1/documents`

While you can upload PDFs and DOCX files directly via the Lodexi Dashboard, this endpoint allows you to programmatically ingest raw text data into your vector database. This is highly useful for integrating with CMS platforms, ticketing systems, or automated scrapers.

### Request Schema

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | string | Yes | The title or source name of the document. This will be used in citations. |
| `content` | string | Yes | The raw text content to be vectorized and stored. The system will automatically chunk this text. |
| `category` | string | No | Optional metadata tag used to filter documents during the `/ask` phase. |

#### JSON Request Example
```json
{
  "title": "IT Support Ticket #1024",
  "content": "To resolve the VPN connection issue on Windows 11, the user must update their network drivers and restart the GlobalProtect service.",
  "category": "IT_Troubleshooting"
}
```

### Response Schema

#### JSON Response Example
```json
{
  "status": "success",
  "message": "Document ingested successfully",
  "data": {
    "document_id": "doc_99x8",
    "chunks_created": 2
  }
}
```

### Code Snippets

**cURL**
```bash
curl -X POST "https://lodexi.yourdomain.com/api/v1/documents" \
     -H "Authorization: Bearer YOUR_API_KEY_HERE" \
     -H "Content-Type: application/json" \
     -H "Accept: application/json" \
     -d '{
           "title": "IT Support Ticket #1024",
           "content": "To resolve the VPN connection issue on Windows 11...",
           "category": "IT_Troubleshooting"
         }'
```

**JavaScript / Fetch**
```javascript
async function ingestDocument() {
  const response = await fetch("https://lodexi.yourdomain.com/api/v1/documents", {
    method: "POST",
    headers: {
      "Authorization": "Bearer YOUR_API_KEY_HERE",
      "Content-Type": "application/json",
      "Accept": "application/json"
    },
    body: JSON.stringify({
      title: "IT Support Ticket #1024",
      content: "To resolve the VPN connection issue on Windows 11...",
      category: "IT_Troubleshooting"
    })
  });
  
  const data = await response.json();
  console.log(data.message);
}
```
