# pdf-ocr-searchable

A public Hermes-compatible skill package and helper scripts for turning scanned PDFs into searchable OCR PDFs, with optional PaddleOCR-VL layout/Markdown extraction.

## What it does

- Produces searchable PDFs with OCRmyPDF + Tesseract.
- Uses the official PaddleOCR-VL async job API for structured Markdown/layout extraction.
- Keeps a legacy VLM OCR path as a fallback when the official API is unavailable.
- Documents a hybrid OCRmyPDF + PaddleOCR overlay pattern for better phrase search.

This repository is sanitized for public use: it does not include personal file paths, local usernames, private token locations, generated OCR outputs, or API keys.

## Repository layout

```text
README.md
LICENSE
.gitignore
skill/SKILL.md
skill/references/ocrmypdf-paddleocr-hybrid.md
scripts/ocrmypdf_searchable_pdf.sh
scripts/paddleocr_official_job.py
scripts/siliconflow_pdf_ocr.py
```

## Install dependencies

For searchable PDF generation:

```bash
sudo apt-get update
sudo apt-get install -y ocrmypdf tesseract-ocr tesseract-ocr-eng tesseract-ocr-chi-sim
```

For Python helpers, create a local virtual environment and install the packages the scripts require:

```bash
python3 -m venv .venv
. .venv/bin/activate
python -m pip install requests pymupdf pillow
```

If your Python is externally managed, use `uv venv` and `uv pip install` instead of installing into system Python.

## Basic usage

Create a searchable PDF:

```bash
scripts/ocrmypdf_searchable_pdf.sh input.pdf
```

Run official PaddleOCR-VL extraction after setting `PADDLEOCR_TOKEN` outside git:

```bash
python scripts/paddleocr_official_job.py input.pdf -o input_paddleocr
```

Alternatively, store the token in a local dotenv file outside git and point to it with `PADDLEOCR_TOKEN_FILE` at runtime.

Run the legacy SiliconFlow fallback only when needed after setting `SILICONFLOW_API_KEY` outside git:

```bash
python scripts/siliconflow_pdf_ocr.py input.pdf
```

## Token handling

Do not hardcode API tokens in skills, scripts, README files, committed config, examples, or logs.

Use environment variables, a shell secret manager, or local dotenv files excluded by `.gitignore`. The repository only documents placeholders.

## Install as a Hermes skill

Copy the skill directory into your local Hermes skill tree. Example with an explicit destination variable:

```bash
SKILL_DIR="$HOME/.hermes/skills/productivity/pdf-ocr-searchable"
mkdir -p "$SKILL_DIR"
cp skill/SKILL.md "$SKILL_DIR/SKILL.md"
cp -R skill/references "$SKILL_DIR/"
```

Then restart or reload Hermes if your runtime does not auto-discover new skill files.

## Public-safety checklist before publishing changes

Before publishing changes:

- Run Python and shell syntax checks for scripts.
- Remove generated caches such as `__pycache__`.
- Search the repository for real credentials, machine-specific absolute paths, private usernames, generated OCR outputs, and local config files.
- Review every match manually before committing.

## License

MIT.
