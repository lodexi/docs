# Lodexi API Quickstart Guide

Welcome to the Lodexi API! This guide will help you connect your external applications (like Telegram Bots, WhatsApp, or Google Apps Script) to your isolated Lodexi Projects (Bots).

## 1. Authentication (Per-Project API Keys)

Lodexi operates on a Multi-Project architecture. This means **API Keys are scoped per Project**, not per user account.

**To get your Project's API Key:**
1. Log in to the Lodexi Dashboard.
2. Select your Project, and navigate to **Project Settings -> API Keys**.
3. Click **Create new secret key**.
4. Copy the generated key. (Keep it safe; you won't be able to see it again!)

Include this key in the `Authorization` header of your HTTP requests as a Bearer token.

```http
Authorization: Bearer YOUR_API_KEY_HERE
Accept: application/json
```

---

## 2. Asking Questions (The `/v1/ask` Endpoint)

This is the primary endpoint for conversational Q&A. It takes a user's question, searches the specific Project's uploaded PDF/TXT documents, and returns a grounded answer.

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

---

## 3. Configuring the AI Persona

Before integrating your bot, you can instruct it on how to behave by configuring its AI Persona (System Prompt).

1. Go to **Project Settings -> AI Persona**.
2. Write a custom instruction. For example:
   > "You are a friendly HR assistant. Always reply in Indonesian and refer to the employee as 'Kak'. If you don't know the answer, politely ask them to contact hr@company.com."
3. Save the settings. Every request sent to the `/v1/ask` endpoint (or via Webhooks) will automatically enforce this persona.

---

## 4. Zero-Code Webhook Integrations (Google Chat)

Lodexi provides native Webhook URLs for instant integration into enterprise platforms, without writing any code.

### Google Chat Integration
1. Go to **Project Settings -> Trigger**.
2. Copy the generated Webhook URL (e.g., `https://lodexi.com/api/v1/projects/{uuid}/chat/google`).
3. Open Google Cloud Console -> Google Chat API -> Configuration.
4. Set the **Connection Settings** to **App URL** and paste your Lodexi Webhook URL.
5. Save. Your bot is now live in Google Chat and will answer questions using your Project's Knowledge Base and AI Persona.

---

## 5. Google Apps Script Integration Example

Want to create custom workflows or process Google Sheets rows? Use this AppScript snippet:

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
```
