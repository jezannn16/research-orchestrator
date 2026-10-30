# Internal Tools Agent

Searches internal company sources: Google Drive, Slack, Notion, email, and
other connected MCP tools. Returns relevant internal context.

## Before spawning this agent

The orchestrator must first confirm which internal MCP tools are available in
the current session. Only include tools that are actually connected — do not
guess. Common tools:

- `gdrive_search` / `gdrive_fetch` — Google Drive
- `slack_search` — Slack messages and channels
- `notion_search` — Notion pages and databases
- `gmail_search` — Gmail

Pass the list of available tools to the subagent prompt.

## Prompt template

```
INTERNAL TOOLS SUB-TASK
=======================
Parent topic: {topic}
Your angle: {angle}
Available tools: {tool_list}  <!-- only tools confirmed available -->

Instructions:
1. For each available tool, run 1-2 searches relevant to the angle.
   Vary search terms — don't repeat the same keywords.
2. Retrieve the most relevant documents/messages (up to 5 items per tool).
3. Skim each result for relevance to the angle before quoting it.
4. Note when each document/message was created or last modified.

Return ONLY this JSON:
{
  "agent": "internal",
  "angle": "{angle}",
  "findings": "<synthesized prose of what internal sources say, 150-300 words>",
  "items": [
    {
      "tool": "gdrive|slack|notion|gmail|other",
      "title_or_subject": "...",
      "date": "YYYY-MM-DD or approximate",
      "author_or_channel": "...",
      "url_or_id": "...",
      "excerpt": "<key quote or summary ≤50 words>",
      "relevance": "high|medium"
    }
  ],
  "gaps": "<what was searched for but not found internally>"
}
```

## Tool budget

- MCP search calls: up to 6
- Fetch/read calls: up to 4
- Total: ≤10 tool calls

## Privacy note

Do not include personally identifiable information (PII), credentials, or
confidential data in the findings JSON beyond what is necessary to answer
the research question. The orchestrator will handle user-facing presentation.
