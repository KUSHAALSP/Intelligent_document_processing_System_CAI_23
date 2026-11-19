# AI-Powered Multi-Document Intelligence and Information Retrieval System

An integrated Streamlit application for document ingestion, OCR/text extraction, classification, field extraction, validation, SQLite storage, search, source-grounded question answering, authenticity-risk screening, human corrections, and manual verification evidence.

## Features that work locally

- Batch upload: PDF, PNG, JPG/JPEG.
- PDF text extraction with PyMuPDF; OCR fallback for scanned PDFs/images when Tesseract OCR is installed and configured.
- Heuristic document classification: Invoice, Receipt, Purchase Order, Contract, KYC, Form, Unknown.
- Basic field extraction and rule validation.
- SQLite document repository with SHA-256 duplicate-file detection.
- Full-text keyword search and CSV export.
- Ask Your Documents: offline lexical retrieval over extracted document chunks; answers display source passages and document IDs. This is extractive Q&A, not a generative LLM.
- Authenticity & Fraud Review: required-field checks, basic GSTIN format check, cautious invoice amount checks, duplicate detection, PDF metadata indicators, review status and JSON/CSV exports.
- Manual official verification evidence: save the source, URL, result, notes and reviewer for GST/MCA/issuer/signature/transaction checks.
- Human field correction workflow with SQLite audit history.
- Document viewer, original-file download, PDF first-page preview and report export.
- Existing database schema is migrated additively at startup.

## Important authenticity limitations

This application is a screening and review tool, not a forensic guarantee. It does **not** automatically access government/issuer APIs, verify GST registration live, validate e-invoice QR codes or digital signatures, or run a trained visual-tampering classifier. The manual evidence form records a human's independently observed result; it does not perform that verification. A heuristic risk score is not a probability of fraud. `Unverified`, `Needs Review`, or `High Risk` must be interpreted with the displayed reasons, not as a definitive fraud verdict.

Official sources that reviewers may consult:
- GST taxpayer search: https://services.gst.gov.in/services/searchtp
- MCA: https://www.mca.gov.in/
- E-invoice QR verification: https://einvoice6.gst.gov.in/content/verify-qr-code/

Portal access and automated APIs may be restricted; use only authorised APIs and permitted verification processes. Avoid uploading sensitive KYC documents to third-party services without proper authorisation and consent.

## Windows setup

Extract the ZIP. Open PowerShell in the extracted project root (the folder containing `requirements.txt` and `app/`).

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
py -m pip install --upgrade pip
py -m pip install -r requirements.txt
py -m streamlit run app/streamlit_app.py
```

Then open http://localhost:8501. Keep the terminal open while using the app.

If PowerShell blocks activation, you can skip activation and run `py -m pip install -r requirements.txt` followed by `py -m streamlit run app/streamlit_app.py`.

### OCR setup

Install Tesseract OCR separately (the Python `pytesseract` package is only a wrapper). Ensure `tesseract.exe` is on PATH, or configure `pytesseract.pytesseract.tesseract_cmd` in `app/processing.py`. Text-based PDFs work without Tesseract.

## Linux/macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m streamlit run app/streamlit_app.py
```

## Tests

```powershell
py -m pytest -q
```

The automated test suite covers processing rules, date handling, OCR-related extraction helpers, retrieval behavior, upload validation, URL validation, and authenticity-status semantics. It does not constitute validation of a production fraud model.

## Project structure

```text
app/
  config.py
  database.py
  processing.py
  authenticity_rules.py
  official_verification.py
  retrieval.py
  streamlit_app.py
data/
  uploads/             # uploaded files are stored here at runtime
  documents.db         # SQLite DB is created at runtime
 tests/
```

## Integration boundaries and next production steps

This release contains functional local checks and manual evidence capture, but it does not pretend that protected external services are connected. See [`docs/INTEGRATION_STATUS.md`](docs/INTEGRATION_STATUS.md) for the exact status of each check and official references.

1. Integrate permitted official verification through an authorised provider, with timeouts, rate limits, and clear `unavailable` states.
2. Add cryptographic PDF signature validation and supported signed e-invoice verification.
3. Train/evaluate a tampering classifier on representative labelled data; report false-positive and false-negative rates.
4. Add a local embedding model or an optional LLM-backed RAG provider if semantic paraphrase search and generated answers are required.
5. Add user authentication, organisation-level access controls, encryption/retention policies and security review before handling real KYC or confidential business documents.


## OCR setup on Windows

The `pytesseract` Python package is only a wrapper; the Tesseract OCR executable must also be installed for PNG/JPG and scanned-PDF text extraction. Install a Windows Tesseract distribution, then add its install directory (commonly `C:\Program Files\Tesseract-OCR`) to PATH. Close and reopen PowerShell and verify with `tesseract --version`. The app sidebar displays whether the executable is actually available. Digital/text PDFs can still be extracted without Tesseract.

## Date handling

Numeric dates are interpreted day-first for this India-focused project (for example, `13-10-2026` becomes `2026-10-13`). Ambiguous dates such as `04/05/2026` cannot be reliably disambiguated from text alone; review the original document.

## Backend hardening added in this build

- Central configuration for data/database paths and maximum upload size (`MAX_UPLOAD_MB`, default 25 MB).
- Filename sanitization and basic content-signature validation before parsing PDF/PNG/JPEG uploads.
- Safer document processing failure handling: detailed exceptions are logged server-side, while the UI receives a non-sensitive message.
- SQLite busy timeout and WAL mode for improved local concurrent-read behaviour.
- `pytest.ini` makes tests discoverable using the standard `python -m pytest -v` command.
- `.env.example`, `.gitignore`, and `docs/ARCHITECTURE.md` document configuration, secret handling and module boundaries.

These are development-grade hardening measures, not a security certification. Do not expose this Streamlit application to the public internet or use it for confidential KYC/business documents until authentication, authorization, deployment hardening, data-retention controls and a security review are completed. See `docs/ARCHITECTURE.md`.
