# Smart Facebook Messenger Chatbot with OpenAI

A production-ready n8n workflow that creates an intelligent Facebook Messenger chatbot with message batching, conversation history, and OpenAI integration.

**Import this workflow:**
`https://raw.githubusercontent.com/trendai-au-lab/n8n-workflow/main/workflows/facebook-messenger-openai-chatbot/workflow.json`

## Prerequisites

- n8n instance running (local or cloud)
- Facebook Page (you must be an admin)
- OpenAI account

## Credentials

| Node | Credential Type | Purpose |
|------|----------------|---------|
| Send Seen Indicator | Facebook Graph API | Interact with Messenger |
| Send Typing Indicator | Facebook Graph API | Interact with Messenger |
| Send Response to User | Facebook Graph API | Interact with Messenger |
| OpenAI Chat Model | OpenAI API | Generate AI responses |

---

## Workflow Architecture

This workflow uses **two webhooks** with the same URL path but different HTTP methods:

| Webhook | Method | Purpose |
|---------|--------|---------|
| Facebook Verification Webhook | GET | Handles Facebook's webhook verification |
| Facebook Message Webhook | POST | Receives incoming messages |

```
Facebook GET  → Verification Webhook → Token Check → Challenge/Forbidden
Facebook POST → Message Webhook → Acknowledge → Batch → AI Agent → Response
```

Both webhooks share the same URL: `https://your-n8n.com/webhook/facebook-messenger-webhook`

n8n automatically routes requests based on HTTP method.

---

## Node-by-Node Breakdown

### 1. Facebook Verification Webhook (GET)
- Receives GET requests from Facebook for webhook verification
- Routes to token validation flow

### 2. Facebook Message Webhook (POST)
- Receives POST requests with incoming messages
- Routes to message processing flow

### 3. Is Token Valid? (IF Node)
- Validates `hub.verify_token` matches your configured token
- **TRUE**: Returns the `hub.challenge` to complete verification
- **FALSE**: Returns 403 Forbidden

### 4. Acknowledge Event (Respond to Webhook)
- Immediately returns `EVENT_RECEIVED` (200 OK) to Facebook
- **Critical**: Facebook requires a response within 5 seconds or it will retry

### 5. Filter Valid Messages (IF Node)
- Checks that the message contains text
- Filters out "echo" messages (messages sent by your page)
- Prevents infinite loops

### 6. Store Message for Batching (Code Node)
- Uses workflow static data to batch messages from the same user
- Example: "Hey" + "Can you help me" + "with my order?" → "Hey Can you help me with my order?"

### 7. Send Seen Indicator (HTTP Request)
- Sends `mark_seen` action to Facebook
- Shows blue checkmarks in Messenger

### 8. Wait 3 Seconds (Wait Node)
- Pauses to collect additional messages for batching

### 9. Retrieve Batched Messages (Code Node)
- Retrieves all stored messages, combines them chronologically
- Clears the batch to prevent duplicate processing

### 10. Has Messages to Process? (IF Node)
- Prevents processing if batch was already handled by another execution

### 11. Send Typing Indicator (HTTP Request)
- Shows "typing..." bubble while AI generates response

### 12. AI Agent (LangChain Agent)
- Receives the combined user message
- Connected to:
  - **OpenAI Chat Model**: Generates intelligent responses (gpt-4o-mini)
  - **Conversation Memory**: Maintains 50-message context per user

### 13. Format Response (Code Node)
- Truncates responses to Messenger's 2000 character limit
- Removes markdown formatting (bold, italic, code blocks)

### 14. Send Response to User (HTTP Request)
- Delivers the AI response via Facebook Graph API

---

## Key Features

| Feature | Description |
|---------|-------------|
| **Message Batching** | Combines rapid messages into one AI request |
| **Conversation Memory** | Remembers last 50 messages per user |
| **Seen/Typing Indicators** | Professional UX feedback |
| **Echo Filtering** | Prevents responding to own messages |
| **Response Formatting** | Cleans markdown for Messenger |
| **Quick Acknowledgment** | Responds to Facebook within 5 seconds |
| **Two Webhooks** | Separate GET (verification) and POST (messages) handlers |

---

## Setup Guide

### Step 1: Get Your OpenAI API Key

1. Go to [OpenAI Platform](https://platform.openai.com/api-keys)
2. Click **Create new secret key**
3. Copy and save the API key

### Step 2: Create Facebook App & Get Page Access Token

1. Go to [Facebook Developers](https://developers.facebook.com) → **My Apps** → **Create App**
2. Select **Other** → **Business** → enter app name and email → **Create App**
3. Add the **Messenger** product from the dashboard
4. In Messenger settings, connect your Facebook Page under **Access Tokens**
5. Click **Generate Token** and copy the Page Access Token

### Step 3: Create Your Verify Token

Create a random string (e.g., `my-secret-token-12345`) and save it for later steps.

### Step 4: Create n8n Credentials

1. **Facebook Graph API Credential:** Add credential → search "Facebook Graph API" → paste Page Access Token
2. **OpenAI API Credential:** Add credential → search "OpenAI API" → paste API key

### Step 5: Import & Configure the Workflow

1. In n8n, click **Add Workflow** → **Import from File** → select `workflow.json`
2. In the **"Is Token Valid?"** node, replace `YOUR_VERIFY_TOKEN_HERE` with your verify token
3. Connect credentials to all nodes (see Credentials table above)

### Step 6: Publish the Workflow

1. Click **Save** then **Publish**
2. Copy the webhook URL (e.g., `https://your-n8n.com/webhook/facebook-messenger-webhook`)

### Step 7: Configure Facebook Webhook

1. In Facebook Developers → your App → Messenger Settings → **Webhooks**
2. Click **Add Callback URL**
3. Enter your n8n webhook URL and verify token
4. Click **Verify and Save**
5. After verification, click **Request Permission** and select `pages_messaging`
6. Subscribe to webhook fields: `messages` (required), `messaging_postbacks` (recommended)

### Step 8: Test Your Chatbot

1. Add test users if needed (App Roles → Roles → Add Testers)
2. Open Messenger, find your Page, and send "Hello!"
3. Wait a few seconds — you should receive an AI response

---

## Going Live: Standard vs Advanced Access

### Standard Access (Development Mode)

Only users with app roles (Administrators, Developers, Testers) can message the bot. No business registration required.

### Advanced Access (Production Mode)

To allow the general public to message your bot, you need **Business Verification**:

1. Go to [Meta Business Suite](https://business.facebook.com/)
2. Complete Business Verification with official documents
3. In your Facebook App → **App Review** → request **Advanced Access** for `pages_messaging`

---

## Customization

| Option | Node | Details |
|--------|------|---------|
| AI system prompt | AI Agent | Change personality and behavior |
| Batching time | Wait 3 Seconds | Shorter = faster, Longer = better batching |
| Memory length | Conversation Memory | Default: 50 messages |
| AI model | OpenAI Chat Model | gpt-4o-mini (default), gpt-4o, gpt-3.5-turbo |

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Webhook verification fails | Check verify token matches in n8n and Facebook |
| Messages not received | Verify webhook subscription is active for `messages` |
| No response from bot | Check n8n execution logs for errors |
| "Error validating access token" | Regenerate Page Access Token in Facebook |
| Duplicate responses | Check that `is_echo` field is being filtered correctly |
| AI not responding | Verify OpenAI API key is correct and has credits |

---

## Resources

- [Facebook Messenger Platform](https://developers.facebook.com/docs/messenger-platform)
- [Facebook Graph API - Send API](https://developers.facebook.com/docs/messenger-platform/reference/send-api)
- [OpenAI API Documentation](https://platform.openai.com/docs)
- [n8n Documentation](https://docs.n8n.io/)
