# LinkedIn → WhatsApp AI Agent

An n8n workflow that monitors LinkedIn profiles daily, summarises new posts with Claude, and delivers them to your WhatsApp via Twilio.

## How it works

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

## Example output

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

## Prerequisites

- [n8n](https://n8n.io) (self-hosted or cloud)
- [RapidAPI](https://rapidapi.com) account — LinkedIn Data API (free tier: 100 req/month)
- [Anthropic](https://console.anthropic.com) API key
- [Twilio](https://twilio.com) account with WhatsApp sandbox enabled

---

## Setup

### 1. Import the workflow

1. Open n8n → **Workflows** → **Import from file**
2. Select `linkedin_whatsapp_agent.json`

### 2. Create credentials

**A. LinkedIn Data — RapidAPI**
1. Go to [rapidapi.com](https://rapidapi.com) → search **"LinkedIn Data API"** → subscribe
2. In n8n: **Credentials** → New → **HTTP Header Auth**
   - Header Name: `X-RapidAPI-Key`
   - Header Value: `<your RapidAPI key>`

**B. Claude — Anthropic**
1. Go to [console.anthropic.com](https://console.anthropic.com) → **API Keys** → Create key
2. In n8n: **Credentials** → New → **HTTP Header Auth**
   - Header Name: `x-api-key`
   - Header Value: `<your Anthropic key>`

**C. WhatsApp — Twilio**
1. Go to [twilio.com](https://twilio.com) → **Messaging** → **Try it out** → **Send a WhatsApp message**
2. Follow the sandbox join instructions (send a WhatsApp to the sandbox number from your phone)
3. In n8n: **Credentials** → New → **HTTP Basic Auth**
   - User: `<Account SID>`
   - Password: `<Auth Token>`

### 3. Configure the workflow

**Profiles to monitor** — edit the `Config: Profiles to Monitor` node, replacing the array with LinkedIn usernames (the part after `linkedin.com/in/`):

```json
["andrewyng", "yann-lecun", "jeff-dean"]
```

**Your WhatsApp number** — edit the `Send WhatsApp (Twilio)` node, replacing the **To** field with your number in E.164 format:

```
whatsapp:+447700900000
```

**Twilio Account SID in URL** — in the same node, replace `YOUR_TWILIO_ACCOUNT_SID` in the request URL with your actual Account SID from the Twilio console.

### 4. Activate

Toggle the workflow **Active**. It will now check every hour and send a WhatsApp summary for any new posts.

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

- **No posts returned** — LinkedIn profile IDs are case-sensitive; verify the exact username from the URL
- **Claude 401** — check the `x-api-key` header value in the Anthropic credential
- **Twilio 401** — Account SID goes in the username field, Auth Token in the password field
- **WhatsApp not delivered** — Twilio sandbox requires the recipient to have joined the sandbox first
