# ADR-004: Prefer Native PDF Text and Use OCR as Fallback

- Status: Accepted
- Context: Many PDFs contain embedded text, while scanned PDFs require OCR.
- Decision: Attempt native page text extraction first. Run OCR only for pages with insufficient or unusable text. Preserve page-level extraction method and metadata.
- Consequences: Lower compute cost and often better text fidelity. Detection thresholds and OCR quality need evaluation on representative scanned contracts.
