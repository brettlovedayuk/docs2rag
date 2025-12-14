# build-training-md

Flatten mixed-format documents into Markdown (and optional JSONL) for downstream uses such as:
- Preparing training corpora for language models
- Building internal search indexes
- Feeding automation tools (for example, n8n) one document at a time

Author and attribution
- Created by Brett Loveday (brett.loveday@thintech.co.uk)
- Licensed under the MIT License (see LICENSE)

Key features
- Two modes:
  - Folder mode: scan a directory tree and generate one consolidated Markdown file with a Table of Contents
  - Single-file mode: process one document and emit the result to stdout (or a file) for automation pipelines
- Supported formats:
  - TXT, MD, JSON, CSV, XLSX/XLSM, DOCX, PDF
- Output options:
  - Markdown sections (safe fenced blocks for text-like formats)
  - Plain text extraction (useful for JSONL / downstream processing)
  - JSON record output (best for automation, includes both Markdown and plain text)
- Optional post-processing:
  - Text normalisation (--clean-text)
  - Legacy citation token stripping (--strip-citations)
  - Best-effort PII removal (--remove-pii / --redact)
  - Per-section size limiting (--max-section-chars)
- Optional JSONL output (--jsonl-out) for downstream pipelines

Repository contents
- build_training_md.py
  - The main CLI script
- README.md
  - Quick start, usage
- INSTRUCTION_MANUAL.md
  - Full reference and operational guidance
- requirements.txt
  - Python dependencies
- LICENSE
  - MIT license text

Installation
- Requirements
  - Python 3.9+ recommended (3.10+ preferred)
- Create a virtual environment
  - python3 -m venv .venv
  - . .venv/bin/activate
- Install dependencies
  - python3 -m pip install -U pip
  - python3 -m pip install -r requirements.txt

Quick start
- Folder mode (build a single consolidated Markdown file)
  - python3 build_training_md.py --root ./docs --out training.md --remove-pii --pii-level extended
- Single-file mode (emit a JSON record to stdout for automation)
  - python3 build_training_md.py --in ./docs/example.pdf --emit json --remove-pii --pii-level extended

n8n example
- Typical flow:
  - Write Binary File (save incoming file to disk)
  - Execute Command:
    - python3 /path/to/build_training_md.py --in "/path/from/previous/node" --emit json --remove-pii --pii-level extended
  - Next node parses stdout as JSON

PII removal notes (important)
- The PII removal is best-effort and regex-based.
- It is designed to reduce risk, not to guarantee that all personal data is removed.
- Always review the output before using it in sensitive workflows.

Licence
- MIT Licence. See LICENSE.
