# OCRmyPDF + PaddleOCR Hybrid Searchable PDF Pattern

Use this pattern when OCRmyPDF/Tesseract produces a valid searchable PDF, but exact phrase search remains weak because the OCR text layer contains spacing noise, recognition errors, or poor handling of mixed-language/formula-heavy content.

## Pattern

1. Run OCRmyPDF first to produce the coordinate-aligned searchable PDF and preserve the original page visuals:

```bash
ocrmypdf -l chi_sim+eng \
  --deskew \
  --rotate-pages \
  --skip-text \
  --output-type pdfa \
  input.pdf input_ocrmypdf.pdf
```

2. Run PaddleOCR-VL separately to produce `result.jsonl` and optional Markdown outputs.

3. Create the delivered PDF from `input_ocrmypdf.pdf`, not from the original PDF. Parse PaddleOCR `result.jsonl` and read each page's dimensions, block bounding boxes, and block text. The exact JSON shape may vary by API version, but commonly includes fields similar to:

```text
layoutParsingResults[].prunedResult.width
layoutParsingResults[].prunedResult.height
layoutParsingResults[].prunedResult.parsing_res_list[].block_bbox
layoutParsingResults[].prunedResult.parsing_res_list[].block_content
```

4. Map each OCR block box from image coordinates to PDF page coordinates:

```python
sx = page.rect.width / ocr_width
sy = page.rect.height / ocr_height
rect = fitz.Rect(x0 * sx, y0 * sy, x1 * sx, y1 * sy)
```

5. Insert the cleaner OCR block text invisibly with a font that supports the document's language:

```python
page.insert_textbox(
    rect,
    text,
    fontname="china-s",  # choose a suitable built-in or embedded font
    fontsize=max(4, min(12, rect.height * 0.42)),
    render_mode=3,        # invisible text
    overlay=True,
)
```

If `insert_textbox` reports overflow, fall back to `page.insert_text((rect.x0, rect.y0 + fontsize), ...)` with a smaller font size.

## Why

OCRmyPDF/Tesseract is usually better for coordinate-level selection and PDF normalization. PaddleOCR-VL block text can be cleaner for exact phrase search, tables, formulas, or languages where Tesseract inserts unwanted spacing. Overlaying both layers gives:

- OCRmyPDF/Tesseract: better coordinate fit and PDF/A-style normalization.
- PaddleOCR-VL: cleaner semantic text for search and Markdown extraction.

## Caveats

- The extra invisible layer may overlap the OCRmyPDF layer. Search usually improves, but copy/selection may include duplicate or block-level text.
- PaddleOCR block coordinates may be less precise than word-level OCR coordinates.
- Always keep the original visual page content unchanged.
- Do not use this as a substitute for legal/archive-grade OCR review.

## Verification

Use PyMuPDF or a PDF viewer to verify search and text extraction:

```python
import fitz

doc = fitz.open("output.pdf")
print(doc.page_count)
for needle in ["example phrase", "table heading", "formula label"]:
    print(needle, sum(1 for page in doc if needle in page.get_text()))
```

Also manually check representative pages for rotated pages, formulas, tables, and duplicated hidden text behavior.
