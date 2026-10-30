# Subagent-Driven Research

Full-spectrum research orchestrator: routes sub-tasks to specialized subagents
by source type, then synthesizes all findings into one coherent report.

## Source types and their subagent agents

Read the agent file for a source type before spawning that subagent:

| Source type | Agent file | When to use |
|---|---|---|
| Web (general) | `references/source-agents/web.md` | News, product info, general facts |
| Academic | `references/source-agents/academic.md` | arXiv, PubMed, Google Scholar, papers |
| Local files | `references/source-agents/files.md` | Uploaded PDFs, docs, CSVs, images |
| Internal tools | `references/source-agents/internal.md` | Google Drive, Slack, Notion, etc. |
| Documentation | `references/source-agents/docs.md` | Official API/library docs, spec sites |

---

## Workflow

### Phase 1: Source audit

Before planning, take stock of what's available:

1. **Uploads**: Are there files in the conversation or working directory?
   If yes → include a files subagent.
2. **Internal tools**: Does the user mention Drive, Slack, Notion, or similar?
   Check which MCP tools are connected. If yes → include an internal subagent.
3. **Academic need**: Does the query mention papers, studies, literature,
   arXiv, PubMed, or research findings? If yes → include an academic subagent.
4. **Web need**: Is current/external info needed? Almost always yes.
5. **Docs need**: Is the question about how to use a specific tool/library/API?
   If yes → include a docs subagent.

### Phase 2: Research plan

Draft a plan:
- List which source types will be searched
- State the key question each subagent will answer
- Define the structure of the final report (section titles)

For complex or ambiguous topics, show the plan to the user in one short
paragraph before proceeding. For clear requests, proceed directly.

### Phase 3: Spawn subagents in parallel

Read the relevant agent file (`references/source-agents/<type>.md`) for each needed source type,
then spawn all subagents simultaneously.

**Orchestrator rule**: never do the research yourself — delegate entirely to
subagents. Your job is to plan, spawn, and synthesize.

Each subagent returns a structured JSON object. See the agent files for the
exact schema each returns.

### Phase 4: Gap check and optional iteration

After collecting results:
- If a critical angle has major gaps, spawn one targeted follow-up subagent.
- Maximum 1 follow-up round per source type.
- Do not iterate indefinitely — note remaining gaps in the report instead.

### Phase 5: Synthesize

Merge all subagent findings into a unified report. See **Output format** below.

Synthesis rules:
- Write everything in your own words
- Every specific claim gets an inline citation `[Title](url or file ref)`
- Surface conflicts between sources explicitly
- Weight sources: peer-reviewed > official docs > reputable web > other

---

## Output format

### Default: Markdown report

```markdown
# Research Report: <Topic>
*Sources: <comma-separated list of source types used>*

> **Summary**: <3-4 sentence executive summary>

## <Section 1>
<prose with inline citations>

## <Section 2>
...

## Key Findings
- <bullet per major insight, each with citation>

## Gaps & Limitations
<what could not be found, conflicting evidence, caveats>

## Sources
| # | Title / File | Location | Type | Relevance |
|---|---|---|---|---|
```

### Saving to file

If the user says "save", "export", or "write to file", save the report as:
`research-<slugified-topic>-<YYYYMMDD>.md` in the current working directory.
Print the file path when done.

### Length guide

| Scope | Target length |
|---|---|
| Single focused question | 300-500 words |
| Multi-angle topic | 600-1000 words |
| Deep survey / literature review | 1000-2000 words |

---

## Error handling

| Situation | Action |
|---|---|
| Internal tool not connected | Note it, proceed with available sources |
| File unreadable / corrupt | Note in report, skip that file |
| Academic DB returns no results | Try Google Scholar web fallback |
| All web results paywalled | Use abstracts, note limitation |
| Subagent errors out | Retry once; if still failing, note the gap |

---

## Notes for the orchestrating agent

- Always read the relevant `references/source-agents/*.md` file before spawning that agent type
- Subagents return JSON; never pass raw JSON to the user
- If only web sources are needed, prefer **deep web research mode**
  (lighter weight, same web quality)
- This skill is the right choice any time ≥2 source types are involved
