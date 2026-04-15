# Scrum Master Bot — Claude Code Operating Manual

## Project overview
n8n automation workflow that substitutes core Scrum Master functions.
Handles daily standup collection, AI-powered summarization, blocker detection,
and Slack delivery. Built as Track 2 of the KudosApp Agentic Engineer assessment.

Platform: n8n 2.16.0 self-hosted via Docker
AI: OpenAI gpt-4o-mini via HTTP Request node
Notifications: Slack Bot (xoxb token) to #standups and #scrum-alerts

## Repository structure
scrum-bot/
├── flows/
│   └── scrum-bot-flow.json     ← n8n exported flow (main deliverable)
├── ai-prompts/
│   └── standup-summary-prompt.md
├── tasks/
│   ├── todo.md
│   └── lessons.md
├── CLAUDE.md
└── README.md

## Flow architecture
Webhook (POST /standup)
  → Edit Fields (extract name, yesterday, today, blockers)
  → Code (validate required fields, detect blockers)
  → HTTP Request (OpenAI gpt-4o-mini — generate structured summary)
  → Code (format response for Slack blocks)
  → IF (hasBlocker?)
      true  → Slack #scrum-alerts (blocker alert) → Slack #standups (report)
      false → Slack #standups (report)

## Core principles
- Flow must work end-to-end in a live demo — reliability over complexity
- Every node has a clear single responsibility
- AI prompt returns strict JSON — no markdown, no code blocks
- Blocker detection is automatic — no human intervention needed
- Flow is exportable and version-controlled as JSON

## How to test
curl -X POST http://localhost:5678/webhook/standup \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Team Member",
    "yesterday": "Completed task X",
    "today": "Will work on task Y",
    "blockers": "None"
  }'
