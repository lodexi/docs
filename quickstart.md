# Lodexi API Quickstart Guide

Welcome to the Lodexi API! This guide will help you connect your external applications (like Telegram Bots, WhatsApp, or Google Apps Script) to your isolated Lodexi RAG Knowledge Base.

## 1. Authentication

All requests to the Lodexi API require an API Key. 

**To get your API Key:**
1. Log in to the Lodexi Dashboard.
2. Navigate to the **API Keys** menu.
3. Click **Create new secret key**.
4. Copy the generated key. (Keep it safe; you won't be able to see it again!)

Include this key in the `Authorization` header of your HTTP requests as a Bearer token.

```http
Authorization: Bearer YOUR_API_KEY_HERE
Accept: application/json
```

---

## 2. Asking Questions (The `/v1/ask` Endpoint)

This is the primary endpoint for conversational Q&A. It takes a user's question, searches your uploaded PDF/TXT documents, and returns a grounded answer.

**Endpoint:** `POST https://lodexi.yourdomain.com/api/v1/ask`

### Request Body (JSON)
```json
{
  "question": "What is the company policy on remote work?",
  "limit": 3
}
```
- `question` (string, required): The question you want to ask your documents.
- `limit` (integer, optional): The maximum number of document chunks to retrieve (default is 4).

### cURL Example
```bash
curl -X POST "https://lodexi.yourdomain.com/api/v1/ask" \
     -H "Authorization: Bearer YOUR_API_KEY_HERE" \
     -H "Content-Type: application/json" \
     -H "Accept: application/json" \
     -d '{
           "question": "What is the company policy on remote work?"
         }'
```

### JSON Response
```json
{
  "tenant_id": "user_123",
  "question": "What is the company policy on remote work?",
  "answer": "Based on the Employee Handbook [Employee_Handbook_2024.pdf], employees are allowed to work remotely for up to 2 days a week with prior manager approval.",
  "grounded": true,
  "cached": false,
  "citations": [
    {
      "external_id": "doc_83j2",
      "title": "Employee_Handbook_2024.pdf",
      "snippet": "Section 4.1: Remote Work. Employees are allowed to work remotely for up to 2 days a week...",
      "score": 0.892
    }
  ],
  "prompt_tokens": 420,
  "completion_tokens": 35
}
```

---

## 3. Google Apps Script Integration Example

Want to create an auto-reply bot in Google Chat or process Google Sheets rows? Use this AppScript snippet:

```javascript
function askLodexi(questionText) {
  var url = "https://lodexi.yourdomain.com/api/v1/ask";
  var apiKey = "YOUR_API_KEY_HERE"; 
  
  var payload = {
    "question": questionText,
    "limit": 3
  };
  
  var options = {
    "method": "post",
    "headers": {
      "Authorization": "Bearer " + apiKey,
      "Content-Type": "application/json",
      "Accept": "application/json"
    },
    "payload": JSON.stringify(payload)
  };
  
  try {
    var response = UrlFetchApp.fetch(url, options);
    var json = JSON.parse(response.getContentText());
    return json.answer;
  } catch (e) {
    return "Error contacting Lodexi AI: " + e.message;
  }
}

// Test the function
function testLodexi() {
  var answer = askLodexi("How many leave days do we get?");
  Logger.log(answer);
}
```

## Next Steps
- Head over to the **Playground** in the Dashboard to test your documents before writing code.
- Remember to top up your OpenAI or Gemini API Keys in the **LLM Settings** menu if you are using the BYOK model.
