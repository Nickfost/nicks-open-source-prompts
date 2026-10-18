---
title: Example: Summarize Article
---

Description

A prompt to summarize long articles into a concise executive summary.

Prompt

"""
You are an assistant that creates concise executive summaries. Given the following article delimited by triple backticks, produce a 3-5 sentence executive summary focusing on the main findings, implications, and recommended next steps.

Article:
```
{ARTICLE}
```

Format:
- 3-5 sentences.
- No bullet points.
- Include one sentence about the implication for a business decision.
"""

Example Input

Replace {ARTICLE} with the article text.

Example Output

A concise 3-5 sentence summary highlighting findings and business implications.
