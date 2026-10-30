# Documentation Agent

Searches official technical documentation: API references, library docs,
language specs, framework guides, and changelog pages.

## Prompt template

```
DOCUMENTATION SUB-TASK
======================
Parent topic: {topic}
Your angle: {angle}
Target docs: {doc_urls_or_names}  <!-- e.g. "React docs", "https://docs.anthropic.com", "Postgres 16 docs" -->

Instructions:
1. If a URL is given, web_fetch it directly.
2. If only a name is given, first web_search "<name> official documentation"
   and fetch the top result.
3. Navigate the docs to find sections relevant to the angle:
   - Try fetching /docs, /reference, /api, /guide subpaths if the root is given.
   - Use web_search "site:<docs_domain> {angle_keywords}" to find specific pages.
4. Read the relevant sections closely. Check for:
   - Version-specific behavior
   - Deprecation notices
   - Known limitations or caveats
   - Code examples

Return ONLY this JSON:
{
  "agent": "docs",
  "angle": "{angle}",
  "findings": "<synthesized prose of what the official docs say, 150-300 words>",
  "key_sections": [
    {
      "doc_name": "...",
      "section": "...",
      "url": "...",
      "version": "vX.Y or 'latest'",
      "summary": "<1-2 sentence summary of this section's relevance>",
      "code_example": "<paste a short code snippet if one is critical, else null>"
    }
  ],
  "version_notes": "<any version-specific gotchas or deprecations found>",
  "gaps": "<questions the official docs don't answer clearly>"
}
```

## Tool budget

- Searches: up to 4
- Fetches: up to 6
- Total: ≤10 tool calls

## Notes

- Always note the documentation version or "latest" if unspecified.
- Prefer the official first-party docs over third-party tutorials.
- If the docs site blocks fetching, fall back to web_search with
  `site:<domain>` to get snippets.
