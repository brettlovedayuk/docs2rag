# Instruction Manual: build-training-md

Created by: Brett Loveday (brett.loveday@thintech.co.uk)
Licence: MIT (see LICENSE)

Purpose
- This tool flattens mixed-format documents into:
  - A single consolidated Markdown file (folder mode), or
  - A single document output (single-file mode) suitable for automation pipelines (for example, n8n)
- The output is designed to be:
  - Easy to parse
  - Safe for Markdown consumers (text is generally wrapped in fenced code blocks)
  - Consistent across runs (stable section anchors and predictable formatting)

Supported file types
- Text-like:
  - .txt
  - .md / .markdown
- Structured:
  - .json (pretty-printed)
  - .csv (rendered as a Markdown table)
  - .xlsx / .xlsm (each sheet rendered as a Markdown table)
- Document:
  - .docx (paragraph text; falls back to a simple table dump if needed)
  - .pdf (text extraction only; no OCR)

Important limitations
- PDF extraction:
  - Image-only or scanned PDFs will not produce meaningful text unless you run OCR upstream.
- Spreadsheet formatting:
  - Complex formatting is not preserved; output is a Markdown table for content capture.
- PII removal:
  - Best-effort and pattern-based; it can miss data or over-redact.
  - Always validate outputs for your risk profile.

Installation
- Prerequisites
  - Python 3.9+ recommended (3.10+ preferred)
- Setup
  - python3 -m venv .venv
  - . .venv/bin/activate
  - python3 -m pip install -U pip
  - python3 -m pip install -r requirements.txt

Command-line usage
- Get help
  - python3 build_training_md.py -h

Modes
- Folder mode
  - --root <folder>
- Single-file mode
  - --in <file>

Only one of --root or --in can be used per run.

Common options (both modes)
- --out <path>
  - Output path
  - Use '-' to write to stdout
  - Default behaviour:
    - Folder mode: thintech_training_data.md
    - Single-file mode: stdout
- --append
  - Append output to --out (useful when calling the script repeatedly in single-file mode)
- --verbose
  - Detailed logging to stderr (keeps stdout clean for automation)
- --clean-text
  - Normalises punctuation and removes control characters for more consistent training text
- --strip-citations
  - Removes legacy inline citation tokens that look like: 62\u2020file:L1-L20
- --max-pdf-pages <n>
  - Safety limit for PDF extraction (default: 200)
- --include-unknown
  - Emits a stub section for unknown file types instead of skipping them
- --max-section-chars <n>
  - Trims large extracted blocks to avoid dataset imbalance

PII removal
- Enable PII removal
  - --remove-pii
  - --redact (alias of --remove-pii, kept for compatibility)
- Configure removal scope
  - --pii-level basic
  - --pii-level extended

PII patterns (best-effort)
- Always removed (basic and extended)
  - Labelled fields at line level, for example:
    - Name:, Full name:, Email:, Phone:, Address:, DOB:, Passport number:
  - Emails
  - Phone numbers (broad pattern, applied after other patterns)
- Additional removals in extended mode
  - Payment card numbers (validated via Luhn check before redacting)
  - IP addresses (IPv4 and IPv6)
  - MAC addresses
  - UK National Insurance numbers (approximate)
  - IBANs

Operational guidance for PII removal
- Treat PII removal as risk reduction, not a guarantee.
- For regulated environments, consider:
  - Upstream OCR redaction for scanned PDFs
  - A second-stage PII scanner (or manual review)
  - Keeping original files out of the training pipeline where possible

Output formats (single-file mode)
- --emit md (default)
  - Emits one Markdown section that includes:
    - A heading
    - Minimal metadata (path, type, size, modified)
    - Extracted content (usually fenced)
- --emit text
  - Emits plain extracted text (outer Markdown fences removed where applicable)
- --emit json
  - Emits a JSON object with:
    - path, type, sha, modified, size
    - markdown (the section Markdown)
    - text (plain extracted text)

JSONL output (both modes)
- --jsonl-out <path>
  - Writes one JSON object per line
- Folder mode
  - One record per processed file
- Single-file mode
  - One record per run
- Notes
  - The "sha" field is a stable identifier derived from the path string, not a content hash.

Examples

Folder mode
- Consolidate a folder into Markdown
  - python3 build_training_md.py --root ./knowledge --out training.md
- Consolidate and produce JSONL sidecar
  - python3 build_training_md.py --root ./knowledge --out training.md --jsonl-out training.jsonl
- Consolidate with PII removal (extended)
  - python3 build_training_md.py --root ./knowledge --remove-pii --pii-level extended --out training.md

Single-file mode (automation-friendly)
- Emit Markdown to stdout
  - python3 build_training_md.py --in ./docs/handbook.docx
- Emit JSON to stdout (recommended for n8n)
  - python3 build_training_md.py --in ./docs/handbook.docx --emit json
- Emit JSON to stdout with PII removal
  - python3 build_training_md.py --in ./docs/handbook.docx --emit json --remove-pii --pii-level extended
- Append multiple runs into one output file
  - python3 build_training_md.py --in ./docs/a.pdf --remove-pii --out training.md --append
  - python3 build_training_md.py --in ./docs/b.docx --remove-pii --out training.md --append

n8n usage pattern
- Recommended for reliability:
  - Save incoming binary file to disk
  - Run single-file mode with --emit json
  - Parse stdout as JSON
- Example Execute Command
  - python3 /path/to/build_training_md.py --in "/data/in/file.pdf" --emit json --remove-pii --pii-level extended

Troubleshooting
- "pypdf not available" / "python-docx not available" / "pandas/openpyxl not available"
  - Install dependencies:
    - python3 -m pip install -r requirements.txt
- "Empty or image-only PDF"
  - The PDF likely requires OCR. Run OCR upstream and feed the OCR output (TXT) into this tool.
- CSV tables look broken
  - The CSV may contain unescaped separators or inconsistent row widths; consider cleaning upstream.

Security and safety notes
- Treat all input documents as untrusted.
- Consider running the tool in a restricted environment if processing third-party documents.
- Review output before sending to external systems.

Contributing
- Keep all source and examples ASCII-only in code blocks.
- Avoid adding dependencies unless required.
- Add new PII patterns carefully to avoid high false-positive rates.

Attribution and licence
- Author: Brett Loveday (brett.loveday@thintech.co.uk)
- Licence: MIT (see LICENSE)
