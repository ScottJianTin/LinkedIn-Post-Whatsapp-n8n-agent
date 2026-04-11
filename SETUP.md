# LinkedIn → WhatsApp AI Agent (n8n + Claude)

## Architecture

```
Schedule (hourly)
  → Config (profile list)
  → Split Profiles
  → Fetch Posts (RapidAPI LinkedIn)
  → Filter New Posts (Code)
  → Has Content? (IF)
    YES → Claude: Summarise Post (claude-sonnet-4-6)
        → Format WhatsApp Message (Code)
        → Send WhatsApp (Twilio)
    NO  → stop
```

---

## Step 1 — Import Workflow into n8n

1. Open n8n → **Workflows** → **Import from file**
2. Select `linkedin_whatsapp_agent.json`

---

## Step 2 — Get API Keys

### A. LinkedIn Data — RapidAPI

1. Go to [rapidapi.com](https://rapidapi.com) → search **"LinkedIn Data API"**
2. Subscribe to **linkedin-data-api** (free tier: 100 req/month)
3. Copy your **RapidAPI Key**
4. In n8n: **Credentials** → New → **HTTP Header Auth**
   - Name: `RapidAPI Key`
   - Header Name: `X-RapidAPI-Key`
   - Header Value: `<your key>`

### B. Claude — Anthropic API

1. Go to [console.anthropic.com](https://console.anthropic.com) → **API Keys** → Create key
2. In n8n: **Credentials** → New → **HTTP Header Auth**
   - Name: `Anthropic API Key`
   - Header Name: `x-api-key`
   - Header Value: `<your key>`

### C. WhatsApp — Twilio

1. Go to [twilio.com](https://twilio.com) → **Console** → **Messaging** → **Try it out** → **Send a WhatsApp message**
2. Follow the sandbox join instructions (send a WhatsApp message to the sandbox number)
3. Note your **Account SID** and **Auth Token**
4. In n8n: **Credentials** → New → **HTTP Basic Auth**
   - Name: `Twilio Basic Auth`
   - User: `<Account SID>`
   - Password: `<Auth Token>`

---

## Step 3 — Configure the Workflow

### Edit "Config: Profiles to Monitor" node

Replace the `linkedinProfileIds` array with the LinkedIn usernames of the people you want to track (the part after `linkedin.com/in/`):

```json
["andrewyng", "yann-lecun", "jeff-dean"]
```

### Edit "Send WhatsApp (Twilio)" node

Replace `+YOUR_WHATSAPP_NUMBER` in the **To** field with your number in E.164 format:
```
whatsapp:+447700900000
```

---

## Step 4 — Activate

Toggle the workflow **Active** in n8n. It will now check every hour for new posts and send you a WhatsApp summary like:

```
*LinkedIn Update* — Andrew Ng
05 Apr 2026  ·  1.2k likes  ·  84 comments

• AI is transitioning from a research curiosity to core business infrastructure.
• Companies that invest in AI talent now will have a 3-year head start.
• The barrier to entry for fine-tuning models has dropped significantly in 2025.
• Open-source models are closing the gap with proprietary ones on most benchmarks.

Why it matters: Teams that wait for "AI to mature" may find the window to build competitive advantage has already closed.

Read full post: https://www.linkedin.com/in/andrew-ng-8aa4b127/
```

---

## Customisation

| Goal | Where to change |
|---|---|
| Check more/less often | `Every Hour` node → change interval |
| Change Claude model | `Claude: Summarise Post` node → `"model"` field |
| Adjust summary style | `Claude: Summarise Post` node → edit the prompt |
| Add more profiles | `Config` node → `linkedinProfileIds` array |
| Send to multiple numbers | Duplicate the `Send WhatsApp` node |

---

## Troubleshooting

- **No posts returned** — LinkedIn profile IDs are case-sensitive; verify the exact username in the URL
- **Claude 401** — check the `x-api-key` header value in the Anthropic credential
- **Twilio 401** — Account SID goes in the username field, Auth Token in the password field
- **WhatsApp not delivered** — Twilio sandbox requires the recipient to have joined the sandbox first
