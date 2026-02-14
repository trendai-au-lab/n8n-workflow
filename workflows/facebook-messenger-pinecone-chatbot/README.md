# Facebook Messenger AI Chatbot with Pinecone RAG

A production-ready n8n workflow that creates an intelligent Facebook Messenger chatbot with RAG (Retrieval Augmented Generation) capabilities, powered by OpenAI and Pinecone Assistant.

**Import this workflow:**
`https://raw.githubusercontent.com/trendai-au-lab/n8n-workflow/tree/main/workflows/facebook-messenger-pinecone-chatbot/workflow.json`

## Prerequisites

- Self-hosted n8n instance (v1.113.0+ recommended)
- Community nodes enabled in n8n
- Facebook Page (you must be an admin)
- OpenAI account
- Pinecone account (free Starter plan works)

## Credentials

| Node | Credential Type | Purpose |
|------|----------------|---------|
| Send Seen Indicator | Facebook Graph API | Interact with Messenger |
| Send Typing Indicator | Facebook Graph API | Interact with Messenger |
| Send Response to User | Facebook Graph API | Interact with Messenger |
| OpenAI Chat Model | OpenAI API | Generate AI responses |
| Get context snippets in Pinecone Assistant | Pinecone API | Search documents |

---

## What is RAG?

**RAG (Retrieval Augmented Generation)** enhances AI responses by:

1. **Retrieving** relevant information from your documents
2. **Augmenting** the AI's context with that information
3. **Generating** responses grounded in your actual data

This means your chatbot can answer questions about YOUR content (products, policies, FAQs, etc.) rather than just using general AI knowledge.

---

## Workflow Architecture

This workflow uses **two webhooks** with the same URL path but different HTTP methods:

| Webhook | Method | Purpose |
|---------|--------|---------|
| Facebook Verification Webhook | GET | Handles Facebook's webhook verification |
| Facebook Message Webhook | POST | Receives incoming messages |

```
Facebook GET  → Verification Webhook → Token Check → Challenge/Forbidden
Facebook POST → Message Webhook → Acknowledge → Process → AI Agent + Pinecone → Response
```

Both webhooks share the same URL path: `/webhook/facebook-messenger-webhook`

n8n automatically routes requests to the correct webhook based on HTTP method.

---

## Node-by-Node Breakdown

### 1. Facebook Verification Webhook (GET)
- Receives GET requests from Facebook for webhook verification
- Routes to "Is Token Valid?" node

### 2. Facebook Message Webhook (POST)
- Receives POST requests with incoming messages
- Routes to "Acknowledge Event" node

### 3. Is Token Valid? (IF Node)
- Validates `hub.verify_token` matches your configured secret
- **TRUE:** Returns the `hub.challenge` to complete verification
- **FALSE:** Returns 403 Forbidden

### 4. Respond with Challenge / Respond Forbidden
- Completes webhook verification handshake with Facebook

### 5. Acknowledge Event (Respond to Webhook)
- Immediately returns `EVENT_RECEIVED` (200 OK) to Facebook
- **Critical:** Facebook requires a response within 5 seconds

### 6. Filter Valid Messages (IF Node)
- Checks message contains text and is not an "echo" (our own message)
- Prevents infinite loops and filters non-text content

### 7. Store Message for Batching (Code Node)
- Batches rapid consecutive messages from the same user
- Uses workflow static data
- Example: "Hey" + "Can you help" + "with orders?" → "Hey Can you help with orders?"

### 8. Send Seen Indicator (HTTP Request)
- Sends `mark_seen` action via Facebook Graph API
- Shows blue checkmarks in Messenger

### 9. Wait 3 Seconds (Wait Node)
- Pauses to collect additional messages
- Configurable timing

### 10. Retrieve Batched Messages (Code Node)
- Retrieves and combines all batched messages
- Clears the batch to prevent duplicate processing

### 11. Has Messages to Process? (IF Node)
- Prevents processing if batch was already handled

### 12. Send Typing Indicator (HTTP Request)
- Shows "typing..." bubble while AI processes

### 13. AI Agent (LangChain Agent)

The brain of the chatbot. Connected to three components:

#### 13a. OpenAI Chat Model
- **Model:** gpt-4o-mini (configurable)

#### 13b. Conversation Memory
- **Type:** Buffer Window Memory
- **Context:** Last 50 messages per user
- **Session Key:** User's Facebook ID

#### 13c. Get context snippets in Pinecone Assistant
- **Type:** Pinecone Assistant Community Node
- **Operation:** Get Context Snippets
- Searches your documents for relevant information

### 14. Format Response (Code Node)
- Truncates to 1900 characters (Messenger limit is 2000)
- Removes markdown formatting (bold, italic, code blocks)

### 15. Send Response to User (HTTP Request)
- Delivers the formatted AI response via Facebook Graph API Send API

### 16. Success (Set Node)
- Marks successful completion of the flow

---

## Setup Guide

### Step 1: Install the Pinecone Assistant Community Node

1. In n8n, go to **Settings** → **Community Nodes**
2. Click **Install a community node**
3. Enter: `@pinecone-database/n8n-nodes-pinecone-assistant`
4. Click **Install**
5. Restart n8n if prompted

> **Note:** Community nodes must be enabled in your n8n instance. For Docker, set `N8N_COMMUNITY_PACKAGES_ALLOW_INSTALL=true`.

### Step 2: Create Pinecone Account & Assistant

1. Go to [Pinecone](https://www.pinecone.io/) and sign up (Starter plan includes 100 files per assistant)
2. In the Pinecone console, go to **Assistants** → **Create Assistant**
3. Name it `n8n-assistant` (or choose your own name) and select your region
4. Upload your documents (PDFs, text files, etc.) under the **Files** tab
5. Copy your API key from your profile → **API Keys**

### Step 3: Get Your OpenAI API Key

1. Go to [OpenAI Platform](https://platform.openai.com/api-keys)
2. Click **Create new secret key**
3. Copy and save the API key

### Step 4: Create Facebook App & Get Page Access Token

1. Go to [Facebook Developers](https://developers.facebook.com) → **My Apps** → **Create App**
2. Select **Other** → **Business** → enter app name and email → **Create App**
3. Add the **Messenger** product from the dashboard
4. In Messenger settings, connect your Facebook Page under **Access Tokens**
5. Click **Generate Token** and copy the Page Access Token

### Step 5: Create Your Verify Token

Create a random string (e.g., `my-secret-token-12345`) and save it for later steps.

### Step 6: Create n8n Credentials

1. **Pinecone Credential:** Add credential → search "Pinecone" → paste API key
2. **OpenAI API Credential:** Add credential → search "OpenAI API" → paste API key
3. **Facebook Graph API Credential:** Add credential → search "Facebook Graph API" → paste Page Access Token

### Step 7: Import & Configure the Workflow

1. In n8n, click **Add Workflow** → **Import from File** → select `workflow.json`
2. In the **"Is Token Valid?"** node, replace `YOUR_VERIFY_TOKEN_HERE` with your verify token
3. In the **"Get context snippets in Pinecone Assistant"** node, set your assistant name
4. Connect credentials to all nodes (see Credentials table above)

### Step 8: Publish the Workflow

1. Click **Save** then **Publish**
2. Copy the webhook URL (e.g., `https://your-n8n.com/webhook/facebook-messenger-webhook`)

### Step 9: Configure Facebook Webhook

1. In Facebook Developers → your App → Messenger Settings → **Webhooks**
2. Click **Add Callback URL**
3. Enter your n8n webhook URL and verify token
4. Click **Verify and Save**
5. Subscribe to webhook fields: `messages` (required), `messaging_postbacks` (recommended)

### Step 10: Test Your Chatbot

1. Add test users if needed (App Roles → Roles → Add Testers)
2. Open Messenger, find your Page, and send test messages:
   - `"Hello!"` — Should get a friendly greeting
   - `"What information do you have?"` — Should search your documents
   - `"Tell me about [topic in your docs]"` — Should return relevant information

---

## Customization

| Option | Node | Details |
|--------|------|---------|
| AI system prompt | AI Agent1 | Adjust citation style, personality, fallback behavior |
| Batching time | Wait 3 Seconds | Shorter = faster responses, Longer = better batching |
| Memory length | Conversation Memory | Default: 50 messages |
| AI model | OpenAI Chat Model | gpt-4o-mini (default), gpt-4o, gpt-4-turbo |
| Pinecone search | Get context snippets in Pinecone Assistant | Snippet size, number of results, metadata filters |

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Pinecone node not found | Install community node: `@pinecone-database/n8n-nodes-pinecone-assistant` |
| "No snippets returned" | Upload documents to your Pinecone Assistant |
| Webhook verification fails | Ensure verify token matches in n8n and Facebook |
| No response from bot | Check n8n execution logs for errors |
| "Error validating access token" | Regenerate Page Access Token in Facebook |
| AI Agent not using Pinecone tool after import | Open the "AI Agent1" node, make a small edit to the system message, and save to re-initialize tool bindings |

---

## Resources

- [Pinecone Assistant Documentation](https://docs.pinecone.io/guides/assistant/overview)
- [Pinecone Assistant n8n Node](https://github.com/pinecone-io/n8n-nodes-pinecone-assistant)
- [Facebook Messenger Platform](https://developers.facebook.com/docs/messenger-platform)
- [Facebook Graph API - Send API](https://developers.facebook.com/docs/messenger-platform/reference/send-api)
- [OpenAI API Documentation](https://platform.openai.com/docs)
- [n8n LangChain Nodes](https://docs.n8n.io/integrations/builtin/cluster-nodes/)
