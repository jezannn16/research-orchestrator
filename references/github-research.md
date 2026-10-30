# GitHub research mode

Multi-agent GitHub explorer: decomposes the issue into angles, runs
parallel subagents for different types of GitHub artifacts, synthesizes one
structured report with a table of repositories and citations.

GitHub is not "just another website": code, issues, PR's, releases, discussions and
Repositories themselves are different sources with different semantics. This skill separates them and assigns a subagent to each relevant type.

---

## When to use this skill

- Search or compare OSS libraries/frameworks/CLI utilities
- Search for real examples of feature implementation in code
- Study of known library problems (issues, discussions)
- Analysis of project activity and health (releases, PRs, frequency of commits)
- Search for forks and alternatives to a popular project
- Dependency audit (licenses, maintainers, security)
- "Has anyone solved X yet?" — search for reference implementations

If the question is **purely factual** (“when did React 19 come out”) - use
`quick web search mode`. If you need **web, docs, and GitHub** -
use `multi-source research mode` and give him this skill as one of
sources.

---

## GitHub Artifact Types and Their Subagents

Before starting a subagent, read its agent file. Everyone has their own budget
tools and JSON schema.

| Artifact Type | Agent File | When needed |
|---|---|---|
| Repositories (search, metadata) | `references/github-agents/repos.md` | "find a library for X", comparing projects, searching for alternatives |
| Source code | `references/github-agents/code.md` | "show me how X is implemented", reference examples, patterns |
| Issues and Discussions | `references/github-agents/issues.md` | Known bugs, workarounds, feature gaps, FAQ |
| Pull Requests | `references/github-agents/prs.md` | How the community solves the problem, what's in the works, recent merges |
| Releases and Changelog | `references/github-agents/releases.md` | Version history, breaking changes, release frequency |
| README and docs repo | `references/github-agents/readme.md` | How the project was announced, getting started, announced features |

---

## Workflow

### Phase 1 - Understand the issue and make a plan

1. **Clarify if it is not clear.** “Find a library” without a language / task - ask
   one clarifying question. If the intention is clear, move on.
2. **Decompose** the question into 2–5 angles, each = a different type of artifact.
3. **Assign** subagents. Don't launch an agent "just to be there" - everyone
   the agent must cover a specific corner.

Decomposition examples:

> *"Find a Rust library for the TON SDK and compare with competitors"*
> - repos: search for candidates by `topic:ton language:rust`
> - readme: what is stated in the README of the top 3
> - releases: activity, latest release
> - issues: open critical bugs
> - code: how they access liteservers (patterns)

> *"How do they usually implement JWT refresh-token rotation in Node.js?"*
> - code: search for a pattern by code
> - repos: top auth libraries
> - issues: discussion of known holes in naive implementations

### Phase 2 - Run subagents in parallel

Read the required `references/github-agents/*.md` and launch all agents **simultaneously**.
The orchestrator does not search itself - it only plans, spawns and synthesizes.

Each agent returns strictly JSON according to its own schema. Raw JSON to user
We don't show it.

### Phase 3 - Whitespace Check (Optional)

After collecting the results:
- If there is an important gap left in `gaps` of a critical subagent - spawn ONE
  follow-up of the subagent exactly under this space.
- Maximum of one follow-up round. Don't get too hung up.

### Phase 4 - Synthesis

Collect all your findings in one report using the format below. Rules:

- Each specific statement - with a link to the repo/issue/PR/commit
- Quotes from the README or code are short (≤25 words), everything else should be rewritten
  in your own words
- Summary table of repositories - required if >1 repo is mentioned
- Conflicts between sources (for example, README says one thing, issues says another)
  raise it clearly

---

## Report format

```markdown
# GitHub Research: <topic>
*Angles: <list of artifact types that were used>*

> **Summary (3-4 sentences):** <what was found, main conclusion>

## Repository Summary

| # | Repo | ⭐ | Latest release | License | Activity | Relevance |
|---|------|---|-----------------|---------|-----------|---------------|
| 1 | [`owner/name`](url) | 12.4k | v2.3.1 (2025-09) | MIT | 🟢 active | high |
| 2 | ... | ... | ... | ... | ... | ... |

## <Section for each corner>

<prose with inline links [owner/name#42](url)>

## Implementation findings
- <key pattern with reference to specific file/line, if applicable>

## Risks and pitfalls
- <what was found in issues - workarounds, known bugs, limitations>

## Gaps and limitations
<which could not be verified, contradictions between sources>

## Sources
| # | Artifact | Type | Link |
|---|---------|-----|--------|
```

### Saving to file

If the user says “save”, “save”, “record”, “export” - save
as `github-research-<slug>-<YYYYMMDD>.md` in the working directory. Lead the way.

### Length

| Swipe | Target Length |
|---|---|
| One point question | 400–700 words |
| Comparison of 3-5 repos | 800–1200 words |
| Full landscape (10+ repo) | 1200–2500 words |

---

## Error Handling

| Situation | Action |
|---|---|
| GitHub Search API rate-limit with fetch | Go to `web_search "site:github.com {query}"` |
| Repo private/404 | Mark, exclude from report |
| README on a non-standard branch (`main` vs `master`) | Try both |
| Huge code file (>2000 lines) | Don't read the whole thing - fetch only the required fragment via line-range |
| Archived repo | Mark ⚠️ archived in the table |
| Subagent returned empty | One retry with an extended request; if empty again, mark gap |

---

## Notes for the orchestrating agent

- Be sure to read the required `references/github-agents/<type>.md` BEFORE starting that subagent -
  there is the exact JSON schema and tool budget
- Never show raw JSON to the user
- If you only need one angle (for example, just “compare these 3 repos”) - that’s it
  Also use this skill with one subagent to save the report format
- When there are more than four angles, consider dividing the question into two runs
  (first landscape, then deep-dive to finalists)
- This skill is the right choice if the question contains at least one of
  markers: github, repository, OSS, library, “see how it’s done”,
  "compare forks", "what's in issues"
