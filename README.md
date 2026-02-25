# AlterEco Publishing Pipeline

> **AI-powered end-to-end publishing automation for Alternativas Económicas magazine. Reduces article processing time from 30 minutes to 2–3 minutes using Claude Vision API, structured extraction, and Drupal REST integration.**

---

## Problem

Each article published by Alternativas Económicas required manual PDF processing: reading the layout, extracting body text, identifying special paragraph types (featured quotes, book reviews, headers), matching with Excel metadata, sourcing images from Google Drive, and publishing to Drupal — 30 minutes of focused work per article.

---

## Solution Architecture

```
PDF Article (uploaded)
        │
        ▼
┌───────────────────────┐
│  Claude Vision API    │  Layout-aware PDF parsing
│  Extraction Pipeline  │  Identifies: body, featured text,
│                       │  book reviews, headers, captions
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│  Metadata Matching    │  Cross-references Excel sheet
│                       │  for author, date, category, tags
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│  Image Matching       │  Searches Google Drive by
│                       │  filename convention / metadata
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│  Drupal REST Publish  │  Structured POST to Drupal API
│                       │  with all fields mapped
└───────────────────────┘
```

---

## Key Technical Components

### Claude Vision PDF Extraction
Standard text extraction loses layout context — featured quotes and book reviews require understanding paragraph position and formatting. Claude Vision API reads the PDF as an image and returns structured JSON describing each content block.

```python
# Structured extraction prompt (simplified)
prompt = """
Analyze this magazine article PDF page and return a JSON object with:
- body_paragraphs: list of main body text blocks
- featured_texts: list of pull quotes or highlighted paragraphs
- book_reviews: any book review sections with title and body
- captions: image captions if present

Return only valid JSON, no preamble.
"""
```

### Excel Metadata Cross-Reference
Extracted content is matched against an Excel metadata sheet (article title, author, publication date, tags, category) using fuzzy title matching to handle slight naming variations.

```python
from rapidfuzz import process

def match_article_metadata(extracted_title, excel_df):
    match, score, idx = process.extractOne(
        extracted_title, excel_df['title'].tolist()
    )
    return excel_df.iloc[idx] if score > 85 else None
```

### Google Drive Image Retrieval
Images are sourced from a structured Google Drive folder using the Drive API, matched by filename convention tied to the article metadata.

### Drupal REST Publishing
The pipeline maps all extracted and matched fields to Drupal's content API schema and POSTs the complete article, including body, featured text blocks, metadata, and media references.

---

## Stack

| Component | Technology |
|---|---|
| Orchestration | n8n (self-hosted) |
| AI / Vision | Claude Vision API (Anthropic) |
| Data Matching | Python, rapidfuzz, openpyxl |
| Image Storage | Google Drive API |
| CMS | Drupal REST API |
| Language | Python, n8n nodes |

---

## Outcome

| Metric | Before | After |
|---|---|---|
| Processing time per article | ~30 minutes | 2–3 minutes |
| Manual steps | ~12 | 1 (review + publish) |
| Paragraph type accuracy | N/A | >95% on standard layouts |
| Operator intervention | Every article | Edge cases only |

---

## Paragraph Types Handled

- Standard body paragraphs
- Featured/pull quote blocks
- Book review sections (title + body extraction)
- Image captions
- Section headers

---

## Note on Code

This repository contains architecture documentation and representative code snippets. Full source is not published to protect client configuration and proprietary workflow details.

---

## Related

- [Soccer Verdun AI Support](https://github.com/danisola/soccer-verdun-ai) — multi-tool RAG agent
- [WorkflowSynth](https://github.com/danisola/workflowsynth) — MSc research on automated AI workflow discovery
