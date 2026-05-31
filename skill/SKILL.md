---
name: pdf-ocr-searchable
description: Use when converting scanned or image-only PDFs into searchable/copyable PDFs and optional structured Markdown. Prefer OCRmyPDF/Tesseract for coordinate-aligned PDF text layers; use the official PaddleOCR-VL job API for layout-aware Markdown extraction; keep VLM-only OCR as a fallback.
version: 1.0.1
author: Eone
license: MIT
metadata:
  hermes:
    tags: [pdf, ocr, searchable-pdf, ocrmypdf, paddleocr, vlm]
    related_skills: [ocr-and-documents]
---

# Searchable PDF OCR

## Overview

This skill turns scanned or image-only PDFs into searchable, copyable PDFs while preserving the original page visuals. It separates two concerns:

- **Searchable PDF text layer:** use OCRmyPDF + Tesseract. This is the preferred path when the deliverable must remain a PDF with text selection/search support.
- **Structured extraction:** use the official PaddleOCR-VL async job API when you also need Markdown, layout blocks, tables, formulas, or extracted images.
- **Fallback OCR:** use the legacy VLM fallback only when the official structured API is unavailable or when a rough text/Markdown pass is enough.

The public version of this skill intentionally avoids user-specific paths, private filenames, local account names, and API tokens. Keep local paths and secrets in environment variables, local config files ignored by git, or your agent's private memory—not in this skill.

## When to Use

Use this skill when:

- A PDF is scanned/image-only and needs searchable text.
- A PDF has a poor or missing text layer.
- The user asks for an OCRed PDF rather than plain text.
- Mixed-language, table-heavy, formula-heavy, or layout-heavy documents need optional Markdown extraction.

Do not use it when:

- The PDF already has a good text layer and the user only needs extraction. Use a lightweight PDF text extractor instead.
- The task requires certified/legal-grade OCR accuracy. In that case, state the limitation and add manual review.

## Default Behavior

For a normal “OCR this PDF” request:

- Produce a searchable PDF.
- Preserve the original visual pages.
- Append an OCR marker to the output filename, for example `input.pdf` → `input_ocr.pdf` or `input(ocr).pdf`, depending on local convention.
- Send only the processed PDF by default.
- Generate Markdown only when the user requests it, when structured extraction is needed, or when troubleshooting requires it.
- Keep the final response short: success status, output filename, and any major OCR caveat found during verification.

## Dependencies

Install OCRmyPDF and Tesseract language packs, for example on Debian/Ubuntu:

```bash
sudo apt-get update
sudo apt-get install -y ocrmypdf tesseract-ocr tesseract-ocr-eng tesseract-ocr-chi-sim
```

Create a project-local Python environment for optional helper scripts:

```bash
python3 -m venv .venv
. .venv/bin/activate
python -m pip install requests pymupdf pillow
```

If the system Python is externally managed, use `uv venv` and `uv pip install` instead of installing packages globally.

## Workflow A: Coordinate-Aligned Searchable PDF

Use the wrapper from this repository:

```bash
scripts/ocrmypdf_searchable_pdf.sh input.pdf
```

Or run OCRmyPDF directly:

```bash
ocrmypdf -l chi_sim+eng \
  --deskew \
  --rotate-pages \
  --skip-text \
  --output-type pdfa \
  input.pdf input_ocr.pdf
```

Use `--force-ocr` only when the existing text layer is bad and should be replaced:

```bash
ocrmypdf -l chi_sim+eng --deskew --rotate-pages --force-ocr --output-type pdfa input.pdf input_ocr.pdf
```

Use `--clean` cautiously. It can help black-and-white scans, but may damage color diagrams, annotations, or textbook figures.

## Workflow B: Official PaddleOCR-VL Structured Extraction

Set the token outside git, then run the helper:

```bash
python scripts/paddleocr_official_job.py input.pdf -o input_paddleocr
```

The script can read the token from:

1. `--token TOKEN`
2. `PADDLEOCR_TOKEN` environment variable
3. An optional dotenv-style file specified by `PADDLEOCR_TOKEN_FILE`

Keep any local dotenv file outside git or ignored by `.gitignore`.

Typical outputs:

- `result.jsonl` — raw structured OCR result.
- `job_result.json` — final job metadata.
- `page_0000.md`, `page_0001.md`, ... — page Markdown.
- `combined.md` — merged Markdown.
- `markdown_images/` and `output_images/` — extracted images when available.

Useful options:

```bash
python scripts/paddleocr_official_job.py input.pdf \
  -o input_paddleocr \
  --use-doc-orientation-classify true \
  --use-doc-unwarping true \
  --use-chart-recognition true
```

## Workflow C: Legacy VLM Fallback

Use this only if the official structured API is unavailable and the user mainly needs a rough text/Markdown result or an emergency searchable PDF. Set `SILICONFLOW_API_KEY` outside git, then run:

```bash
python scripts/siliconflow_pdf_ocr.py input.pdf
```

The fallback creates a sidecar Markdown file and a PDF with an invisible page-level text layer. It does not provide the same word-level coordinate fidelity as OCRmyPDF.

## Hybrid Search Improvement

For documents where OCRmyPDF produces a good PDF text layer but exact phrase search is weak, use the hybrid pattern in `references/ocrmypdf-paddleocr-hybrid.md`:

1. Generate the base searchable PDF with OCRmyPDF.
2. Run PaddleOCR-VL to obtain cleaner block text and layout boxes.
3. Overlay PaddleOCR block text invisibly onto the OCRmyPDF output with PyMuPDF.

This can improve exact phrase search, but the hidden text may be less precise for selection/copy behavior than the OCRmyPDF text layer.

## Verification Checklist

After processing a real PDF:

- [ ] Confirm the output PDF exists and opens.
- [ ] Use a PDF viewer or PyMuPDF to search for a known phrase.
- [ ] Try selecting/copying text on representative pages.
- [ ] For structured extraction, confirm `combined.md` and page-level Markdown files exist.
- [ ] Spot-check tables, formulas, page rotations, and unusual symbols.
- [ ] Confirm no API tokens, personal paths, private filenames, or generated OCR outputs are committed.

## Common Pitfalls

1. **Hardcoding tokens.** Keep API credentials in environment variables or local ignored files only.
2. **Publishing personal paths.** Public skills should use repository-relative paths such as `scripts/...`, not machine-specific paths.
3. **Using VLM text as the authoritative PDF text layer.** Use OCRmyPDF for PDF text layers unless you have reliable bounding boxes and coordinate mapping.
4. **OCRing already-text PDFs unnecessarily.** Use `--skip-text` by default; use `--force-ocr` only when replacing a bad text layer.
5. **Over-cleaning color documents.** Avoid `--clean` for color textbooks, annotated pages, and diagram-heavy PDFs unless tested.
6. **Assuming OCR is perfect.** Always verify a few representative pages before reporting success.
