# Subagent-Driven Web Search

A lightweight skill for spawning a single focused search subagent and returning
a concise, cited answer fast.

## When to use this skill

- One specific question that needs a current web answer
- Fact-checking a single claim
- Looking up a current status, price, version, or role
- Quick news lookup ("did X happen?", "what happened with Y?")

For multiple related questions or synthesis across many sources, use
**deep web research mode** instead.

---

## Workflow

### Step 1: Clarify the query (if needed)

If the user's question is ambiguous (e.g. "find info on Python") — ask one
clarifying question before proceeding. If the intent is clear, proceed directly.

### Step 2: Spawn the search subagent

Launch a subagent with the following prompt template. Pass it via `claude -p`
or the Task tool.

```
SEARCH TASK
===========
Query: <user's question, reworded as a precise 3-6 word search query>
Goal: <one sentence — what does the user actually want to know?>

Instructions:
1. Run web_search with the query above.
2. If the top results don't clearly answer the goal, run 1-2 follow-up
   searches with narrower queries.
3. If a result looks relevant but the snippet is incomplete, use web_fetch
   to read the full page (max 1 fetch).
4. Return a JSON object:
   {
     "answer": "<concise direct answer in 1-3 sentences>",
     "confidence": "high|medium|low",
     "sources": [
       { "title": "...", "url": "...", "snippet": "<key quote ≤20 words>" }
     ],
     "follow_up_needed": false,
     "notes": "<optional: caveats, date of info, conflicting data>"
   }
   Return ONLY the JSON, no preamble.
```

### Step 3: Parse and present the result

- Present the `answer` directly in prose.
- Cite sources inline using markdown links: `[Source Title](url)`
- If `confidence` is `low` or `follow_up_needed` is `true`, say so and offer
  to run a deeper search (→ deep web research mode).
- If `notes` contains important caveats (stale data, conflicting sources),
  include them.

---

## Output format

```
<Direct answer in 1-3 sentences, citing sources inline.>

Sources: [Title](url), [Title](url)
```

Keep it short. No headers, no bullet lists unless the answer is inherently
a list (e.g. "list the top 5 X").

---

## Error handling

| Situation | Action |
|---|---|
| Subagent returns no results | Retry with broader query, inform user |
| Conflicting results across sources | Report the conflict, note most credible source |
| Page fetch blocked/timeout | Proceed with snippet only, note limitation |
| Query requires auth (paywalled) | Note paywall, offer alternative source |

---

## Notes for the orchestrating agent

- Total tool budget: ≤5 tool calls inside the subagent (search + optional fetches)
- Target latency: fast — do not over-research a simple question
- This skill is intentionally minimal; escalate to web-research if the user
  pushes back or needs more depth
