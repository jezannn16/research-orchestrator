# Releases & Changelog Agent

Examines the repository's Releases and changelogs: version history, release frequency,
breaking changes, migration guides, stability.

## Prompt template

```
RELEASES RESEARCH SUB-TASK
==========================
Parent topic: {topic}
Your angle: {angle}
Repos in scope: {repo_list}
Lookback: {N_releases or time_window} <!-- for example "last 10 releases" or "last 12 months" -->

Instructions:
1. For each repo, open /releases:
   https://github.com/{owner}/{name}/releases
2. If there are few details on the page, fetch specific releases:
   https://github.com/{owner}/{name}/releases/tag/{tag}
3. Also check the changelog files in the root of the repo (often, but not always):
   https://raw.githubusercontent.com/{owner}/{name}/main/CHANGELOG.md
   https://raw.githubusercontent.com/{owner}/{name}/main/HISTORY.md
4. For each repo, calculate:
   - last N releases with dates
   - frequency: once per week, per month, per quarter, in flux
   - presence of BREAKING CHANGE in release notes
   - is there a migration guide / upgrade notes
   - semver stability (pre-1.0 vs stable)
   - regularity of security-fix releases
5. Compare trajectory: the project accelerates, slows down, is frozen.

Return ONLY this JSON:
{
  "agent": "releases",
  "angle": "{angle}",
  "findings": "<synthesis in prose 150-300 words: how often does it release, what were
                breaking changes, are there culture migration guides,
                project trajectory>",
  "repos": [
    {
      "full_name": "owner/name",
      "latest_version": "vX.Y.Z",
      "latest_release_date": "YYYY-MM-DD",
      "release_cadence": "weekly|monthly|quarterly|sporadic|frozen",
      "is_pre_v1": false,
      "recent_releases": [
        {
          "tag": "vX.Y.Z",
          "date": "YYYY-MM-DD",
          "url": "...",
          "highlights": "<1-2 sentences about key changes>",
          "breaking_changes": true,
          "security_fixes": false
        }
      ],
      "breaking_changes_history": "<for example 'v2.0 removed the callback API in favor of promises'>",
      "migration_quality": "good|sparse|none",
      "stability_signal": "stable|active-but-stable|in-flux|abandoned",
      "relevance": "high|medium"
    }
  ],
  "gaps": "<for example 'release notes are empty for most tags'>"
}
```

## Tool budget

- web_search: up to 2
- web_fetch: up to 8 (releases pages + CHANGELOG)
- Total: ≤10

## Notes

- “Release once a year” can be both a problem (“abandoned”) and a feature (“stable,
  everything works") - depends on the context. Don't judge automatically.
  Cross with the activity of commits from the repos agent.
- Pre-1.0 (v0.x) - signal “API is still moving, don’t go too deep.”
- "Release yesterday" does not mean "stable". Could be a hotfix after regression.
  If you can, look at the last two dates in a row.
- Don't quote full release notes. Excerpts and summary.
- If /releases is empty, but there are tags in the code - sometimes tags without accompanying
  text (rolling releases in monorepo style). This is a valid pattern, please note.
- If the format in CHANGELOG.md is "Keep a Changelog" - this is a good signal
  discipline, please mention.
