# Subagent-Driven Web Research

Multi-agent iterative research: plan queries → spawn parallel search subagents
→ synthesize findings → produce a structured cited report.

## When to use this skill

- Topic requires 5+ sources or multiple angles
- User wants a comparison, landscape, or survey
- Simple search returned conflicting/incomplete results
- User explicitly says "research", "report", "comprehensive", "deep dive"

For a single factual question, use **quick web search mode** instead.
For research that also needs local files or internal tools, use
**multi-source research mode**.

---

## Workflow

### Phase 1: Research plan

Before spawning subagents, draft a research plan:

1. **Decompose** the topic into 3-6 distinct sub-questions or angles
2. **Assign** one sub-question per subagent
3. **Decide** the output structure (sections of the final report)

Show the plan to the user briefly if the topic is ambiguous (one line per
sub-question). For clear requests, proceed without asking.

Example plan for "research the best vector databases in 2025":
- Subagent A: feature comparison (indexing, filtering, scalability)
- Subagent B: performance benchmarks and recent evaluations
- Subagent C: pricing and hosting options
- Subagent D: community, docs, ecosystem maturity

### Phase 2: Spawn parallel subagents

Launch all subagents simultaneously. Each receives this prompt:

```
RESEARCH SUB-TASK
=================
Parent topic: <overall research topic>
Your angle: <specific sub-question this agent covers>
Depth: thorough — visit 3-5 sources, fetch full pages where snippets are thin

Instructions:
1. Run 2-3 web searches with different query angles.
2. Use web_fetch on the most informative pages (max 3 fetches).
3. Prioritize: official docs > reputable tech/news sites > blogs > forums.
4. Return JSON:
   {
     "angle": "<your assigned angle>",
     "findings": "<synthesized prose, 150-300 words>",
     "key_facts": ["<specific data point>", "..."],
     "sources": [
       { "title": "...", "url": "...", "relevance": "high|medium" }
     ],
     "gaps": "<what you couldn't find or verify>"
   }
   Return ONLY the JSON.
```

**Tool budget per subagent**: ≤10 tool calls (searches + fetches combined).

### Phase 3: Identify gaps (optional iteration)

After collecting all subagent results:

- If a subagent reported critical `gaps` that matter for the final answer,
  spawn one follow-up subagent to fill that gap specifically.
- Maximum 1 follow-up round. Don't over-iterate.

### Phase 4: Synthesize into report

Combine all findings into a structured Markdown report:

```markdown
# <Topic>

> **Summary**: <2-3 sentence executive summary>

## <Section per angle>

<prose from findings, with inline citations [Source](url)>

## Key Takeaways

- <bullet per major insight>

## Sources

| # | Title | URL | Relevance |
|---|-------|-----|-----------|
| 1 | ...   | ... | high      |
```

Rules for synthesis:
- Write in your own words — do not paste subagent findings verbatim
- Every specific claim needs an inline citation
- If sources conflict, note the disagreement explicitly
- Include a "Limitations" note if important gaps remain

---

## Output format

A **Markdown report** with:
- Executive summary (2-3 sentences)
- One section per research angle (prose + inline citations)
- Key takeaways (bullets)
- Sources table

Length guide: 400-800 words for most topics; up to 1500 for complex ones.

---

## Error handling

| Situation | Action |
|---|---|
| Subagent returns empty/error | Retry that angle once with broader query |
| All sources paywalled on a sub-topic | Note the limitation, use abstracts/summaries |
| Findings massively contradict each other | Surface the disagreement, don't hide it |
| Topic is too broad | Narrow scope, tell user which sub-topics were covered |

---

## Notes for the orchestrating agent

- Spawn all Phase 2 subagents in **parallel** — not sequentially
- The orchestrator synthesizes; subagents only search and return JSON
- If the user wants a `.md` file saved, write it to the working directory
  and note the path
- Do not present raw subagent JSON to the user — always synthesize first
