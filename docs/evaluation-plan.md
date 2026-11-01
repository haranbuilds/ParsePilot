# ParsePilot Evaluation Plan

## Primary benchmark

Use CUAD (Contract Understanding Atticus Dataset) as the initial contract clause extraction benchmark. Confirm the official release, license/terms, data format and annotation mapping before downloading or redistributing data.

The product's first ten fields are a selected subset; they do not map one-to-one to all CUAD categories. Create and version an explicit mapping document before scoring.

## Extraction metrics

- Exact match for normalized categorical/date values where appropriate.
- Precision, recall and F1 for clause/span identification.
- Field-level accuracy and macro-F1 across supported fields.
- Evidence support rate: proportion of non-null extracted fields with valid supporting source evidence.
- Evidence localization accuracy: page and span match against labels where available.
- Null/abstention quality: whether the system avoids inventing absent fields.

## Retrieval metrics (later search task)

- Recall@K
- MRR
- nDCG@K
- Citation/page correctness

## System metrics

- End-to-end processing latency (p50, p95)
- OCR latency per page
- LLM latency and token usage
- Cost per document
- Queue wait time
- Retry and failure rates
- Human-review rate

## Evaluation discipline

- Split by contract/document, not random text chunks, to avoid leakage.
- Keep a held-out test set.
- Store model, prompt, schema and preprocessing versions with every run.
- Compare against a simple baseline.
- Report failures by field and document type.
- Do not use model self-reported confidence as the only confidence metric.
- Treat benchmark scores as research measurements, not a guarantee of legal correctness.
