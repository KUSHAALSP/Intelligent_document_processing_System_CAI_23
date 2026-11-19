# Changelog

## Integrated release — 2026-10-13

- Preserved the existing Streamlit application and SQLite repository.
- Hardened document ingestion with filename sanitization, extension/signature checks, and configurable size limits.
- Improved document-number extraction when labels and values appear on separate lines.
- Improved amount extraction for separate-line values and common OCR confusion of the rupee symbol.
- Added subtotal + tax reconciliation checks when all values are available.
- Fixed PDF metadata inspection order so the file exists before its metadata is read.
- Added same-type document-number collision detection in addition to exact SHA-256 duplicate detection.
- Closed SQLite connections deterministically after each operation.
- Refined the workspace visual hierarchy and clarified OCR readiness.
- Added integration status documentation describing what is local, manual, unavailable, and not implemented.
- Added regression tests for document classification, date normalization, amount extraction, reconciliation, and unavailable verification states.

## Manual verification workflow update

- Saved official-source observations now update the matching authenticity report check.
- Evidence records include source details, subject checked, reviewer, notes, and timestamp.
- Matched, mismatch, unable-to-verify, and not-applicable results map to explicit report statuses.
- Company/GST/MCA guidance is provided without scraping portals or falsely claiming automated API verification.
- Added regression tests for manual verification status updates and source URL validation.
