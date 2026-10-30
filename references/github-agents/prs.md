# Pull Requests Agent

Studying the Pull Requests of the repository: what is currently being worked on, how it was recently decided
typical tasks, what approaches maintainers compete with.

## Prompt template

```
PRS RESEARCH SUB-TASK
=====================
Parent topic: {topic}
Your angle: {angle}
Repos in scope: {repo_list}
State filter: open|merged|closed-unmerged|all <!-- default "merged" for "as decided", "open" for "work in progress" -->

Instructions:
1. For each repo, open the PR page with a search filter:
   https://github.com/{owner}/{name}/pulls?q={URL_ENCODED_QUERY}
   Useful filters:
     is:pr is:open|is:merged|is:closed
     label:"bug" label:"breaking-change"
     author:{username}
     sort:updated-desc
2. For the 3-6 most interesting PRs, open their page via web_fetch:
   https://github.com/{owner}/{name}/pull/{number}
   and count:
   - title, author, dates opened/merged
   - "Files changed" - which files were affected (number and nature)
   - Reviewers' comments (especially if there is a reasoned pushback)
   - resolution: merged | closed without merge | open at work
   - bindings to issues (Fixes #1234)
3. SEPARATELY valuable:
   a) **Recently merged PRs** - as is now decided
   b) **Open PRs with old sign-off** - what is stuck in the discussion
   c) **Closed-without-merge PRs** - rejected approaches (explanation in discussion)

Return ONLY this JSON:
{
  "agent": "prs",
  "angle": "{angle}",
  "findings": "<synthesis in prose 150-300 words: what's in the works, what's happened recently,
                which approaches compete, which are rejected and why>",
  "items": [
    {
      "repo": "owner/name",
      "number": 5678,
      "url": "https://github.com/owner/name/pull/5678",
      "title": "<full title>",
      "author": "username",
      "state": "open|merged|closed-unmerged",
      "opened": "YYYY-MM-DD",
      "merged_or_closed": "YYYY-MM-DD or null",
      "files_changed": 8,
      "additions_minus_deletions": "+450 -120",
      "linked_issues": [1234, 5670],
      "summary": "<1-2 sentences about the essence of the changes>",
      "review_outcome": "<for example 'merged after 2 review rounds', 'closed: maintainer preferred alternative #5689'>",
      "approach": "<what technical approach is used in this PR>",
      "relevance": "high|medium"
    }
  ],
  "trends": [
    "<a trend visible through multiple PRs - for example 'abandonment of X in favor of Y'>"
  ],
  "stuck_prs": [
    {
      "repo": "owner/name",
      "number": 1111,
      "why_stuck": "<brief explanation from discussion>"
    }
  ],
  "gaps": "<for example 'no open PR on this topic, last merged 3 years ago'>"
}
```

## Tool budget

- web_search: up to 3
- web_fetch: up to 8 (search pages + specific PRs)
- Total: ≤12

## Notes

- A large PR (>50 files) is usually either a refactoring or a new feature. Small
  PR in hot-zone (auth, payments) - often security-fix; worth a look.
- "Closed without merge" is a goldmine of context. If the offer
  rejected, in the comments there is usually an explanation “why you shouldn’t do this.”
- If there is little external PR in the repo, this is not bad. Perhaps the project is being carried out by one
  maintainer who commits directly. Record this as a signal
  bus-factor.
- Do not quote other people's code-review comments in long blocks. Rephrase
  the bottom line, leave a short quote of ≤20 words if it is directly indicative.
- The PR ↔ Issue (`Fixes #N`) binding is useful for tracing: the progress is visible
  "problem → solution". If you can, link it in findings.
