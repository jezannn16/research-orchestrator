# Local Files Agent

Reads and extracts relevant information from uploaded or local files (PDFs,
Word docs, CSVs, Markdown, images, etc.).

## Prompt template

```
FILES RESEARCH SUB-TASK
=======================
Parent topic: {topic}
Your angle: {angle}
Files to process: {file_paths}  <!-- list of absolute paths -->

Instructions:
1. For each file:
   a. Determine the file type (extension + quick inspection).
   b. Read it using the appropriate method:
      - .pdf:          use the Read tool or pdftotext if available
      - .docx/.doc:    use the Read tool (plain text extraction)
      - .csv/.tsv:     read first 50 rows to understand structure; sample more if needed
      - .md/.txt:      read directly
      - .png/.jpg:     describe what's visible (charts, diagrams, screenshots)
      - .xlsx:         read sheet names, then key sheets
   c. Extract content relevant to the angle.
2. Do NOT read entire large files — skim for relevance, read sections closely.
3. Note file name and page/section for each extracted fact.

Return ONLY this JSON:
{
  "agent": "files",
  "angle": "{angle}",
  "findings": "<synthesized prose of what the files contain relevant to the angle, 150-300 words>",
  "key_excerpts": [
    {
      "source_file": "filename.ext",
      "location": "page 3 / section 'Results' / row 42",
      "excerpt": "<verbatim quote or close paraphrase ≤40 words>",
      "relevance": "high|medium"
    }
  ],
  "file_summaries": [
    {
      "file": "filename.ext",
      "type": "pdf|docx|csv|md|image|other",
      "summary": "<1-2 sentence summary of the whole file>",
      "pages_or_rows": "N"
    }
  ],
  "gaps": "<questions the files don't answer>"
}
```

## Tool budget

- Read calls: up to 10 (read sections, not always whole files)
- No web calls — files agent stays local

## Notes

- If a file is unreadable or corrupt, note it and skip.
- If an image contains a chart or diagram relevant to the topic, describe
  what it shows (axes, trends, key values).
- For large files (>50 pages / >10k rows): skim structure first, then deep-read
  the most relevant sections only.
