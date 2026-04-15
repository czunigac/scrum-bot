# Standup Summary — OpenAI System Prompt

Used in: n8n HTTP Request node → OpenAI API
Model: gpt-4o-mini
Node: "HTTP Request" (node 4 in the flow)
Purpose: Transform raw standup answers into structured JSON summaries
with automatic blocker detection and status classification.

---

## System prompt (exact text used in production)

You are a Scrum Master assistant. Given a team member's standup update,
produce a concise professional summary.

Respond ONLY with this exact JSON structure, no markdown, no code blocks:
{
  "summary": "2-3 sentence professional summary",
  "status": "on-track | blocked | at-risk",
  "keyPoints": ["point 1", "point 2"],
  "blockerAlert": "description if blocked, null if not"
}

Always respond in the same language as the input.

---

## User prompt template

Team member: {{ $json.memberName }}
Yesterday: {{ $json.yesterday }}
Today: {{ $json.today }}
Blockers: {{ $json.blockers }}

---

## Design decisions

### JSON-only response
The Code node after this HTTP Request parses the response directly with
JSON.parse(). Any markdown wrapping (```json) would break the parser.
The prompt explicitly says "no markdown, no code blocks" to prevent this.
A regex strip is also applied as a safety fallback in the Code node.

### status field values
Three possible values map directly to the IF node branching logic:
- on-track → standard report to #standups only
- at-risk → standard report to #standups (future: could add warning)
- blocked → report to #standups + alert to #scrum-alerts

### blockerAlert field
Returns null when no blocker is present. The Code node checks:
  hasBlocker: !!ai.blockerAlert
This drives the IF node — null is falsy, any string is truthy.

### Language detection
No explicit language instruction needed — gpt-4o-mini automatically
responds in the same language as the input. Tested with Spanish and
English standup responses.

### Model choice — gpt-4o-mini
- Fast response time (~1-2 seconds) — acceptable for async webhook flow
- Low cost — suitable for daily automation at scale
- Sufficient reasoning for structured extraction tasks
- gpt-4o would add latency without meaningful quality improvement here

### Temperature: 0.3
Low temperature ensures consistent JSON structure output.
Higher values risk creative formatting that breaks JSON.parse().

---

## Example input / output

Input:
  memberName: Christopher Zuniga
  yesterday: Completed Kudos Coach SSE streaming and fixed Supabase IPv6 migration issue
  today: Build leaderboard page and badges system
  blockers: None

Output:
{
  "summary": "Christopher successfully completed the Kudos Coach SSE streaming feature and resolved a Supabase IPv6 connection issue. Today he will focus on building the leaderboard page and implementing the badges system.",
  "status": "on-track",
  "keyPoints": [
    "Completed Kudos Coach SSE streaming",
    "Fixed Supabase IPv6 migration issue",
    "Building leaderboard and badges today"
  ],
  "blockerAlert": null
}

---

Input with blocker:
  memberName: Maria Lopez
  yesterday: Reviewed authentication PR and updated unit tests
  today: Deploy notification system to staging
  blockers: Waiting for DevOps to provide staging API keys, cannot proceed

Output:
{
  "summary": "Maria completed her PR review and updated unit tests. She is blocked on deploying the notification system to staging due to missing API keys from DevOps.",
  "status": "blocked",
  "keyPoints": [
    "Completed authentication PR review",
    "Updated unit tests",
    "Blocked on staging deployment"
  ],
  "blockerAlert": "Waiting for DevOps to provide staging API keys — deployment to staging is blocked"
}
