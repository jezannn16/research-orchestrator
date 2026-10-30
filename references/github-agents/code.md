# Code Search Agent

Searches for real implementations and patterns in source code on GitHub. Returns
short, meaningful quotes linked to a file/line, not code dumps.

## Prompt template

```
CODE RESEARCH SUB-TASK
======================
Parent topic: {topic}
Your angle: {angle}
Search hints: {language, repos_to_focus_on} <!-- for example "language:typescript", "in repo siepie/onion-runtime" -->

Instructions:
1. Sources:
   a) GitHub Code Search via web_fetch:
      https://github.com/search?q={URL_ENCODED_QUERY}&type=code
      Possible modifiers in the request: language:ts, repo:owner/name,
      path:src/, extension:rs, "exact phrase in quotes."
   b) If you know a specific repo, fetch its page with a path filter:
      https://github.com/{owner}/{name}/search?q={query}
2. For the 3-6 most interesting matches, open the raw-URL file:
   https://raw.githubusercontent.com/{owner}/{name}/{branch}/{path}
   (a typical branch is main, sometimes master; if 404, try another)
3. Read ONLY the relevant section (max 50 lines around the match). Don't download
   entire files if they are >300 lines.
4. Extract:
   - short quote ≤15 lines of code
   - what it does (1-2 sentences, in your own words)
   - what pattern/technique does it demonstrate?
5. If you see the same pattern in several repos, mark it,
   This is a signal “this is how it is done.”

Return ONLY this JSON:
{
  "agent": "code",
  "angle": "{angle}",
  "findings": "<prose synthesis of 150-300 words: what patterns were found, general vs
                idiosyncratic, which approaches compete>",
  "snippets": [
    {
      "repo": "owner/name",
      "file_path": "src/auth/refresh.ts",
      "url": "https://github.com/owner/name/blob/main/src/auth/refresh.ts#L42-L58",
      "language": "typescript",
      "snippet": "<≤15 lines of code, no placeholder comments>",
      "what_it_does": "<1-2 sentences>",
      "pattern": "<what is the name of this technique, if applicable: 'rotating refresh token', 'PKCE', etc>",
      "relevance": "high|medium"
    }
  ],
  "common_patterns": [
    "<pattern encountered in ≥2 repo>"
  ],
  "antipatterns_observed": [
    "<if a typical error is clearly visible in the code, mention it>"
  ],
  "gaps": "<for example 'all examples found in JS, but not found for Rust'>"
}
```

## Tool budget

- web_search: up to 3 (direct GitHub Code Search is often enough)
- web_fetch: up to 8 (search page + 3-6 raw files)
- Total: ≤12

## Notes

- GitHub Code Search requires you to be logged in for some requests; if
  the search page returns empty or asks for a login - go to
  `web_search "{query} site:github.com"` and look for code results in the search results.
- Snippets >20 lines long are almost always bad. If the pattern is large -
  describe it in prose and give a link, don't send the whole code.
- NEVER quote license-headers, long comments or README inserts
  - only semantic code.
- Observe the copyright: quotes are short, always with a link to the source.
  Don't reverse engineer someone else's entire code - that's a violation even at MIT.
- If a vulnerability is immediately visible in the code (hardcode of secrets, eval from user input)
  - mention in `antipatterns_observed`, but without moralizing.
- “I didn’t find pattern X” is a valid result. Don't make things up.
