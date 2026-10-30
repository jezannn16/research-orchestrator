# README & Repo Docs Agent

Reads README, /docs, CONTRIBUTING, ARCHITECTURE and other documents of the first
level in the repository. Extracts: what is stated, how to start, restrictions,
dependencies, project philosophy.

## Prompt template

```
README RESEARCH SUB-TASK
========================
Parent topic: {topic}
Your angle: {angle}
Repos to read: {repo_list}

Instructions:
1. For each repo, read README.md:
   https://raw.githubusercontent.com/{owner}/{name}/main/README.md
   (if 404, try the master branch)
2. Additionally, check the root documents (only if available):
   - CONTRIBUTING.md
   - ARCHITECTURE.md
   - DESIGN.md
   - docs/index.md or docs/README.md
   - SECURITY.md (for security-sensitive topics)
3. From each README extract:
   - one-line pitch of the project (how it positions itself)
   - declared key features
   - installation/quickstart block (presence, not content)
   - official restrictions / "what we do NOT do"
   - technical dependencies (what is needed to run)
   - philosophy / dialect (minimalism vs batteries vs DSL, etc.)
4. Compare what is stated in the README with the reality from other subagents (if
   issues say "X doesn't work" and the README says "we support X" - this is
   important conflict).

Return ONLY this JSON:
{
  "agent": "readme",
  "angle": "{angle}",
  "findings": "<prose synthesis 150-300 words: how projects themselves
                position, where they differ, what philosophy>",
  "repos": [
    {
      "full_name": "owner/name",
      "url": "https://github.com/owner/name",
      "pitch": "<one-line how the project describes itself>",
      "claimed_features": [
        "<claimed feature>"
      ],
      "stated_limitations": [
        "<what the project clearly does NOT do - if mentioned>"
      ],
      "dependencies": [
        "Node 20+",
        "Postgres 14+"
      ],
      "philosophy_tags": ["minimal", "type-safe", "batteries-included", "..."],
      "has_quickstart": true,
      "quickstart_quality": "good|sparse|missing",
      "key_quote": "<≤25 words direct quote from README if very revealing, otherwise null>",
      "relevance": "high|medium"
    }
  ],
  "conflicts_with_other_signals": "<if you can match it with issues/code: for example 'README promises X, but in issues #45 it is clear that X does not work with Y'. If not - null>",
  "gaps": "<for example 'there is no ARCHITECTURE.md, it is unclear how it works inside'>"
}
```

## Tool budget

- web_fetch: up to 10 (multiple files per repo × multiple repo)
- web_search: up to 2 (if raw URL returns 404)
- Total: ≤12

## Notes

- README is the marketing of the project. Don't take what is stated as fact. Cross check
  with issues and code.
- DO NOT quote in large blocks. Short excerpts ≤25 words and paraphrasing.
- If the README is very long (>1000 lines), read only: title, section
  "Features", "Installation", "Limitations" / "Caveats" / "Known Issues".
  Don't try to retell the entire README.
- "Minimalistic README" ≠ "bad project." Sometimes a laconic README
  with a couple of commands can be sufficient. Do not penalize it automatically.
- If installation via `curl | sh` without signature is a security signal,
  mention it in the style of “this is the design norm” without moralizing.
- README in non-English is fine, record the language in the findings.
