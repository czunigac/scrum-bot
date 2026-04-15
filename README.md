# KudosApp — Scrum Master Bot

n8n automation workflow that handles daily standup rituals and substitutes
core Scrum Master functions using AI-powered summarization and Slack integration.

Built as Track 2 of the KudosApp Agentic Engineer technical assessment.

---

## What it does

- Collects answers to the 3 classic standup questions from team members
- Validates required fields and detects blockers automatically
- Generates a structured AI summary using OpenAI gpt-4o-mini
- Delivers formatted standup report to the #standups Slack channel
- Routes blocker alerts immediately to #scrum-alerts for Scrum Master attention

---

## Flow architecture

Webhook → Edit Fields → Code (validate) → HTTP Request (OpenAI)
  → Code (format) → IF (blocker?)
      → true:  Slack #scrum-alerts + Slack #standups
      → false: Slack #standups

Total nodes: 8
Trigger: HTTP Webhook (POST)
AI model: gpt-4o-mini
Notification: Slack Bot

---

## Platform choice — why n8n

Chosen over Microsoft Copilot Studio because:
- Full control over flow logic via Code nodes (JavaScript)
- Exported JSON flow is version-controllable in git
- Self-hosted — no external service dependency for the demo
- Visual flow canvas is easy to walk through in a live demo
- Native HTTP Request node for direct OpenAI API integration

---

## Setup

### Prerequisites
- Docker installed
- n8n running locally: `docker run -d --name kudos_n8n -p 5678:5678 n8nio/n8n`
- OpenAI API key
- Slack Bot Token (xoxb-...) with channels:scopes write permission

### Import the flow
1. Open n8n at http://localhost:5678
2. Click the menu → Import workflow
3. Select `flows/scrum-bot-flow.json`
4. Configure credentials:
   - OpenAI: Settings → Credentials → Add → OpenAI → paste API key
   - Slack: Settings → Credentials → Add → Slack API → paste Bot Token
5. Invite the Slack bot to both channels:
   - In #standups: `/invite @YourBotName`
   - In #scrum-alerts: `/invite @YourBotName`
6. Activate the workflow (toggle in top right)

---

## Trigger a standup

```bash
# Normal standup — delivers to #standups
curl -X POST http://localhost:5678/webhook/standup \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Christopher Zuniga",
    "yesterday": "Completed Kudos Coach SSE streaming and fixed Supabase migration",
    "today": "Build leaderboard page and badges system",
    "blockers": "None"
  }'

# Standup with blocker — triggers alert to #scrum-alerts
curl -X POST http://localhost:5678/webhook/standup \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Maria Lopez",
    "yesterday": "Reviewed authentication PR and updated tests",
    "today": "Deploy notification system to staging",
    "blockers": "Waiting for DevOps to provide staging API keys, cannot proceed"
  }'
```

---

## Slack output

### #standups channel
Each standup produces a formatted message with:
- Status indicator (✅ on-track / ⚠️ at-risk / 🚫 blocked)
- Team member name
- AI-generated 2-3 sentence summary
- Key points extracted from the update

### #scrum-alerts channel
When blockers are detected:
- 🚨 Alert header with member name
- Blocker description extracted by AI
- Immediate attention flag for the Scrum Master

---

## AI prompt design

The OpenAI system prompt is documented in `ai-prompts/standup-summary-prompt.md`.

Key design decisions:
- JSON-only response format prevents parsing failures
- status field (on-track/blocked/at-risk) drives the IF node branching
- blockerAlert field is null when no blocker — clean boolean check
- Language detection is automatic — works for Spanish and English teams
- gpt-4o-mini chosen for speed and cost efficiency in daily automation

---

## AI tools used

| Tool | Usage |
|---|---|
| Claude | Flow design, prompt engineering, Code node logic, README |
| OpenAI gpt-4o-mini | Standup summary generation and blocker detection |
| GitHub Copilot | JavaScript code completions in Code nodes |

---

## What I would add next

- **Scheduled trigger** — cron job at 9am to send standup questions automatically
- **Reminder flow** — if a member hasn't responded by 10am, send a reminder
- **Google Sheets logging** — persist all standups historically for sprint review
- **Sprint health report** — AI analysis of weekly patterns, delivered every Friday
- **Sentiment analysis** — detect team stress levels from standup tone
- **KudosApp integration** — link standup blockers to kudos given when resolved
- **Multi-team support** — separate webhook paths per team with team-specific channels

---

## Repository structure

scrum-bot/
├── flows/
│   └── scrum-bot-flow.json     ← Import this into n8n
├── ai-prompts/
│   └── standup-summary-prompt.md
├── tasks/
│   ├── todo.md
│   └── lessons.md
├── CLAUDE.md
└── README.md
