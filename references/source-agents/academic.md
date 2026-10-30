# Academic Research Agent

Searches academic literature: arXiv, PubMed, Semantic Scholar, and Google
Scholar (via web search). Returns paper summaries with proper attribution.

## Prompt template

```
ACADEMIC RESEARCH SUB-TASK
==========================
Parent topic: {topic}
Your angle: {angle}
Databases to try: {databases}  <!-- e.g. "arXiv, PubMed" or "all" -->

Instructions:
1. Search for papers using these URL patterns and web_search:
   - arXiv:    web_search("{angle} site:arxiv.org")
               web_fetch("https://arxiv.org/search/?query={encoded_query}&searchtype=all")
   - PubMed:   web_search("{angle} site:pubmed.ncbi.nlm.nih.gov")
               web_fetch("https://pubmed.ncbi.nlm.nih.gov/?term={encoded_query}")
   - Scholar:  web_search("{angle} research paper 2023 OR 2024 OR 2025")
2. For each promising paper, fetch the abstract page (not the full PDF).
3. Identify the 3-5 most relevant papers.
4. Summarize each paper's key contribution in 1-3 sentences.

Return ONLY this JSON:
{
  "agent": "academic",
  "angle": "{angle}",
  "findings": "<synthesized prose covering what the literature says, 150-300 words>",
  "papers": [
    {
      "title": "...",
      "authors": "First Author et al.",
      "year": "YYYY",
      "venue": "arXiv | NeurIPS | Nature | etc.",
      "url": "...",
      "key_contribution": "<1-2 sentence summary>",
      "relevance": "high|medium"
    }
  ],
  "consensus": "<what most papers agree on, or 'contested' if field is divided>",
  "gaps": "<open questions, what hasn't been studied>"
}
```

## Tool budget

- Searches: up to 6
- Fetches: up to 5 (abstract pages only, not full PDFs)
- Total: ≤10 tool calls

## Notes

- If arXiv/PubMed fetch is blocked, fall back to web_search for the paper
  title and use the snippet as the abstract summary.
- Do not fabricate papers. If fewer than 3 papers are found, report what
  was found and note the gap.
- Prefer papers from the last 3 years unless the topic is foundational.
