# pdf-ocr-searchable

[English](#english) | [中文](#中文)

---

## English

A public Hermes-compatible skill package and helper scripts for turning scanned PDFs into searchable OCR PDFs, with optional PaddleOCR-VL layout and Markdown extraction.

### What it does

- Produces searchable PDFs with OCRmyPDF + Tesseract.
- Uses the official PaddleOCR-VL async job API for structured Markdown/layout extraction.
- Keeps a legacy VLM OCR path as a fallback when the official API is unavailable.
- Documents a hybrid OCRmyPDF + PaddleOCR overlay pattern for better Chinese phrase search.

This repository is sanitized for public use: it does not include personal file paths, local usernames, private token locations, generated OCR outputs, or API keys.

### Repository layout

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

### Install dependencies

For searchable PDF generation:

```bash
sudo apt-get update
sudo apt-get install -y ocrmypdf tesseract-ocr tesseract-ocr-eng tesseract-ocr-chi-sim
```

For Python helpers, create a local virtual environment and install the required packages:

```bash
python3 -m venv .venv
. .venv/bin/activate
python -m pip install requests pymupdf pillow
```

If your Python is externally managed, use `uv venv` and `uv pip install` instead of installing into system Python.

### Basic usage

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

### Token handling

Do not hardcode API tokens in skills, scripts, README files, committed config, examples, or logs.

Use environment variables, a shell secret manager, or local dotenv files excluded by `.gitignore`. The repository only documents placeholders.

### Install as a Hermes skill

Copy the skill directory into your local Hermes skill tree. Example with an explicit destination variable:

```bash
SKILL_DIR="$HOME/.hermes/skills/productivity/pdf-ocr-searchable"
mkdir -p "$SKILL_DIR"
cp skill/SKILL.md "$SKILL_DIR/SKILL.md"
cp -R skill/references "$SKILL_DIR/"
```

Then restart or reload Hermes if your runtime does not auto-discover new skill files.

### Public-safety checklist before publishing changes

Before publishing changes:

- Run Python and shell syntax checks for scripts.
- Remove generated caches such as `__pycache__`.
- Search the repository for real credentials, machine-specific absolute paths, private usernames, generated OCR outputs, and local config files.
- Review every match manually before committing.

### License

MIT.

---

## 中文

一个公开的 Hermes 兼容 skill 包，附带 PDF OCR 辅助脚本。它可以把扫描版 / 图片版 PDF 转成可搜索、可复制的 OCR PDF，也可以按需调用 PaddleOCR-VL 提取版面结构和 Markdown。

### 能做什么

- 用 OCRmyPDF + Tesseract 生成带隐藏文字层的可搜索 PDF。
- 用官方 PaddleOCR-VL 异步任务 API 提取结构化 Markdown 和版面信息。
- 保留一条旧版 VLM OCR 兜底路径，在官方 API 不可用时使用。
- 文档中包含 OCRmyPDF + PaddleOCR 叠加方案，用来改善中文短语搜索效果。

这个仓库已经按公开发布标准清理：不包含个人路径、本机用户名、私有 token 路径、生成的 OCR 结果或 API 密钥。

### 仓库结构

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

### 安装依赖

生成可搜索 PDF 需要：

```bash
sudo apt-get update
sudo apt-get install -y ocrmypdf tesseract-ocr tesseract-ocr-eng tesseract-ocr-chi-sim
```

Python 辅助脚本建议放在本地虚拟环境里运行：

```bash
python3 -m venv .venv
. .venv/bin/activate
python -m pip install requests pymupdf pillow
```

如果系统 Python 受外部管理，优先用 `uv venv` 和 `uv pip install`，不要直接往系统 Python 里装包。

### 基础用法

生成可搜索 PDF：

```bash
scripts/ocrmypdf_searchable_pdf.sh input.pdf
```

设置好 `PADDLEOCR_TOKEN` 后，运行官方 PaddleOCR-VL 提取：

```bash
python scripts/paddleocr_official_job.py input.pdf -o input_paddleocr
```

也可以把 token 放在 git 之外的本地 dotenv 文件里，并在运行时通过 `PADDLEOCR_TOKEN_FILE` 指向它。

只有在需要兜底时，才运行旧版 SiliconFlow 路径；运行前需要在 git 之外设置 `SILICONFLOW_API_KEY`：

```bash
python scripts/siliconflow_pdf_ocr.py input.pdf
```

### Token 处理

不要把 API token 硬编码进 skill、脚本、README、已提交配置、示例或日志。

推荐使用环境变量、shell 密钥管理器，或被 `.gitignore` 排除的本地 dotenv 文件。仓库里只保留占位说明，不放真实密钥。

### 安装为 Hermes skill

把 `skill/` 目录复制到本地 Hermes skill 目录即可。下面示例用变量显式指定目标目录：

```bash
SKILL_DIR="$HOME/.hermes/skills/productivity/pdf-ocr-searchable"
mkdir -p "$SKILL_DIR"
cp skill/SKILL.md "$SKILL_DIR/SKILL.md"
cp -R skill/references "$SKILL_DIR/"
```

如果当前 Hermes 运行环境不会自动发现新 skill，复制后重启或重新加载 Hermes。

### 公开发布前检查清单

发布修改前建议检查：

- 对 Python 和 shell 脚本做语法检查。
- 删除 `__pycache__` 等生成缓存。
- 搜索仓库里是否残留真实凭据、机器相关绝对路径、私有用户名、生成的 OCR 输出和本地配置文件。
- 所有命中都要人工确认后再提交。

### 许可证

MIT.
