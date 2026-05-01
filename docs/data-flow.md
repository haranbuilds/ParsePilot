# ParsePilot Data Flow

## A. Upload and ingestion

1. Client sends a PDF to `POST /api/v1/documents`.
2. API authenticates and authorizes the caller.
3. File validation checks size, declared MIME type, file signature and PDF parseability.
4. API computes SHA-256 and checks duplicate policy.
5. Original file is stored through the storage interface.
6. PostgreSQL stores document metadata with status `uploaded`.
7. API enqueues an ingestion job and returns document ID plus status.

## B. Text extraction

1. Worker claims the job.
2. PyMuPDF attempts native text extraction page by page.
3. Pages with insufficient text are marked for OCR.
4. OCR worker converts selected pages to images and invokes Tesseract.
5. Output is normalized conservatively; page association is preserved.
6. Each page stores text, extraction method and processing metadata.
7. Document status advances to `text_extracted` or a stage-specific failure.

## C. Contract extraction

1. Text is segmented into chunks with page and offset metadata.
2. Retrieval/classification identifies passages relevant to target fields.
3. Provider adapter sends selected evidence and a strict output schema to the LLM.
4. Response parser rejects malformed or unexpected structures.
5. Pydantic validates types and field constraints.
6. Evidence verifier checks each answer against source passages.
7. Unsupported, conflicting or ambiguous values are marked for review.
8. Extraction run and field values are persisted.

## D. Indexing and search

1. Chunks are embedded.
2. Qdrant stores vectors with document, organization, page and chunk metadata.
3. Search applies authorization/tenant filters before returning content.
4. Later hybrid retrieval combines lexical and vector results.
5. Reranking and grounded answer generation add citations to source pages.

## E. Human review

1. Reviewer opens flagged fields.
2. UI displays extracted value and source evidence.
3. Reviewer accepts, edits or rejects the field.
4. Review event records actor, timestamp, previous value, new value and reason.
5. Approved correction becomes the current reviewed value; original extraction remains auditable.

## Document lifecycle (initial proposal)

`uploaded -> queued -> extracting_text -> text_extracted -> extracting_fields -> validating -> needs_review | completed`

Failure states:
- `failed_validation`
- `failed_ingestion`
- `failed_extraction`
- `failed_indexing`

State transitions must be explicit and tested. A retry creates an auditable attempt rather than erasing prior failure information.
