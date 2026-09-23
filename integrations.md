# Integration Guides: Google Chat & WhatsApp

Lodexi acts as an AI Orchestration Middleware, meaning you can easily connect your isolated Knowledge Base (Project) to various external chat applications. 

This guide will walk you through setting up a **Zero-Code Webhook Integration for Google Chat** and building a **Custom API Integration for WhatsApp**.

---

## 1. Zero-Code Integration: Google Chat

Lodexi provides native Webhook endpoints that are formatted specifically to respond to Google Chat's payload structure. You don't need to write any code to connect your bot to a Google Workspace!

### Step 1: Get Your Lodexi Webhook URL
1. Log in to your Lodexi Dashboard.
2. Ensure you are in the correct **Project**.
3. Navigate to **Project Settings -> Trigger**.
4. You will see a pre-generated URL for Google Chat. It looks like this:
   `https://lodexi.yourdomain.com/api/v1/projects/{your-project-uuid}/chat/google`
5. Copy this URL.

### Step 2: Create a Google Chat App
1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Select your Google Workspace project (or create a new one).
3. In the search bar, look for **Google Chat API** and **Enable** it.
4. Once enabled, click on the **Configuration** tab for the Google Chat API.
5. Fill in the required details:
   - **App name:** e.g., "Lodexi HR Bot"
   - **Avatar URL:** Provide an image URL for your bot's icon.
   - **Description:** e.g., "AI Assistant for HR inquiries."
6. Under **Interactive features**, toggle "Enable interactive features".
7. Under **Functionality**, check "Receive 1:1 messages" and "Join spaces and group conversations".
8. Under **Connection settings**, select **App URL**.
9. **Paste the Lodexi Webhook URL** you copied in Step 1 into the "App URL" field.
10. Save your configuration.

### Step 3: Test Your Bot
Go to your Google Chat workspace, search for your new Bot's name, and send a message. The bot will automatically consult your Lodexi Project's Knowledge Base, apply your AI Persona, and reply directly in the chat!

---

## 2. API Integration: WhatsApp (via Node.js & Baileys)

Unlike Google Chat, WhatsApp does not have a single standard webhook format (unless you use the official Cloud API, which requires Meta approval). For custom WhatsApp integrations, you can build a simple bridge script using a library like `@whiskeysockets/baileys` and the Lodexi `/ask` endpoint.

### Prerequisites
- Node.js installed.
- A Lodexi **Project API Key** (Generated in Project Settings -> API Keys).

### Step 1: Setup the Node.js Project
Create a new directory and install the required packages:

```bash
mkdir lodexi-whatsapp
cd lodexi-whatsapp
npm init -y
npm install @whiskeysockets/baileys axios qrcode-terminal
```

### Step 2: Create the Bridge Script (`bot.js`)
Create a file named `bot.js` and paste the following code. Make sure to replace `YOUR_LODEXI_API_KEY` and the `LODEXI_URL`.

```javascript
const { makeWASocket, useMultiFileAuthState } = require('@whiskeysockets/baileys');
const qrcode = require('qrcode-terminal');
const axios = require('axios');

// Configure your Lodexi Details
const LODEXI_API_URL = "https://lodexi.yourdomain.com/api/v1/ask";
const LODEXI_API_KEY = "YOUR_LODEXI_API_KEY"; 

async function connectToWhatsApp () {
    const { state, saveCreds } = await useMultiFileAuthState('auth_info_baileys');
    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: true
    });

    sock.ev.on('creds.update', saveCreds);

    sock.ev.on('messages.upsert', async ({ messages }) => {
        const m = messages[0];
        
        // Ignore status updates or messages from the bot itself
        if (!m.message || m.key.fromMe) return;

        const senderId = m.key.remoteJid;
        const text = m.message.conversation || m.message.extendedTextMessage?.text;

        if (text) {
            console.log(`Received message: ${text}`);

            try {
                // Send the text to Lodexi API
                const response = await axios.post(LODEXI_API_URL, {
                    question: text,
                    limit: 3
                }, {
                    headers: {
                        'Authorization': `Bearer ${LODEXI_API_KEY}`,
                        'Content-Type': 'application/json'
                    }
                });

                const aiAnswer = response.data.answer;

                // Reply to the user on WhatsApp
                await sock.sendMessage(senderId, { text: aiAnswer });

            } catch (error) {
                console.error("Error communicating with Lodexi API:", error.message);
                await sock.sendMessage(senderId, { text: "Sorry, I am having trouble connecting to my brain right now." });
            }
        }
    });
}

connectToWhatsApp();
```

### Step 3: Run and Authenticate
Run the script using:
```bash
node bot.js
```
A QR code will appear in your terminal. Scan it with your WhatsApp mobile app (Linked Devices). Once connected, any message sent to that WhatsApp number will be processed by Lodexi's RAG system and replied to automatically!

---

## Need More Integrations?
Because Lodexi is API-first, you can apply the exact same logic from the WhatsApp example to connect Lodexi to **Slack, Discord, Zendesk, or Microsoft Teams** using their respective bot SDKs. Simply capture the user's message, send it to the `/ask` endpoint, and return the `answer`.
