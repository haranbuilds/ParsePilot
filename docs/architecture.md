# ParsePilot System Architecture

## 1. Product goal

ParsePilot accepts a contract PDF, extracts relevant contractual information, attaches evidence from the source document, validates the output, and routes uncertain results for human review. Users can later search across their contracts and ask questions with citations.

The product is an assistance and review system. It does not replace qualified legal review and must not present model-generated interpretations as legal advice.

## 2. Initial users and workflow

Initial user roles:
- Organization administrator: manages organization membership and settings.
- Reviewer: uploads and reviews contracts, corrects extracted fields.
- Read-only user: views permitted documents and results.

Primary workflow:
1. User uploads a PDF contract.
2. API validates the file and creates a document record.
3. Original file is stored behind a storage abstraction.
4. A background job extracts native PDF text; scanned pages use OCR fallback.
5. Text is normalized while retaining page and source-location metadata.
6. Relevant contract passages are identified.
7. Extraction produces typed fields and evidence references.
8. Deterministic validation checks types, dates, required fields, and evidence linkage.
9. Low-confidence or unsupported fields are flagged for human review.
10. Results and processing history are persisted.
11. Chunks are indexed for later search and question answering.

## 3. Architecture style

Use a modular monolith initially. Modules have explicit responsibilities and interfaces, but are deployed as a small number of containers. This avoids premature microservice complexity while keeping boundaries clear enough to extract services later if scale requires it.

Logical modules:
- API: HTTP routes, request/response models, dependency wiring.
- Identity and tenancy: users, organizations, roles, authorization.
- Documents: upload, metadata, lifecycle and storage.
- Ingestion: PDF parsing, OCR, normalization and page representation.
- Extraction: clause selection, LLM invocation, schema validation and evidence linking.
- Review: human corrections, approval state and audit events.
- Search: chunking, embeddings, Qdrant retrieval and hybrid search.
- RAG: context assembly, grounded answer generation and citation checks.
- Workers: asynchronous execution, retries and scheduled/background tasks.
- Observability: structured logs, metrics and traces.

## 4. Proposed technology choices

| Concern | Initial choice | Rationale |
|---|---|---|
| Language | Python 3.12 | Ecosystem and maintainability |
| API | FastAPI | Typed REST APIs and OpenAPI |
| Validation | Pydantic | Runtime schema validation |
| Relational data | PostgreSQL | Durable metadata and transactional state |
| ORM/migrations | SQLAlchemy + Alembic | Explicit models and versioned schema |
| PDF text | PyMuPDF | Native text extraction and page access |
| OCR fallback | Tesseract | Local OCR without mandatory GPU |
| LLM | Provider adapter; Gemini initially | Avoid coupling domain code to one vendor |
| Embeddings | BGE-small-en-v1.5 initially | CPU-friendly English embedding baseline |
| Vector store | Qdrant | Vector retrieval with payload filtering |
| Background jobs | Celery + Redis | Retryable asynchronous processing |
| Frontend | React + TypeScript | Separate user interface |
| Tests | Pytest + HTTPX | Unit and API/integration tests |
| Lint/format | Ruff | Fast Python quality checks |
| Containers | Docker Compose | Reproducible local and initial deployment |

These are initial decisions, not claims that every dependency must be installed in Task 2. Add dependencies only when their task begins.

## 5. Contract extraction scope

Initial product schema fields:
- contracting parties
- agreement date
- effective date
- expiration date
- renewal term
- termination notice period
- governing law
- liability cap
- indemnification
- confidentiality

The product schema is deliberately smaller than CUAD's full taxonomy. It will be versioned and expanded only after evaluation.

Each extracted field should contain:
- field name
- typed value or null
- normalized value when applicable
- source document identifier
- page number
- exact evidence quote
- evidence offsets or bounding box when available
- extraction run identifier
- validation status
- review status

Do not treat an LLM's self-reported confidence as calibrated probability. Confidence must be based on explicit signals such as evidence presence, schema validation, retrieval score, agreement between methods, and human review.

## 6. Reliability and failure handling

- Uploads are validated before processing.
- Each processing request has an idempotency key or equivalent deduplication strategy.
- Background jobs have bounded retries and exponential backoff.
- Permanent failures are recorded with a stage and safe error code.
- Partial outputs are not silently represented as complete.
- Extraction runs are immutable; a new run creates a new version.
- External provider calls have timeouts and quota/error handling.
- Original files and database records must not be orphaned during normal retries.
- Health and readiness checks distinguish API availability from dependency readiness.

## 7. Security and privacy

- Keep secrets in environment/secret management, never in Git.
- Enforce organization-level authorization on every document operation.
- Validate file extension, MIME type, size, and actual file signature.
- Generate server-side storage names; never trust user-supplied paths.
- Do not expose raw internal exceptions or secrets in API responses.
- Restrict CORS and network exposure in deployment.
- Record reviewer actions in an audit trail.
- Define retention and deletion behavior before handling real customer contracts.
- Treat uploaded documents and their contents as untrusted data, including prompt-injection attempts.

## 8. Deployment topology

Initial Docker Compose topology:
- API container
- Worker container
- PostgreSQL
- Redis
- Qdrant
- Frontend container (when frontend task begins)

File storage starts behind an interface with local development storage. Production object storage can be added without changing document-domain logic.

## 9. Key quality attributes

- Correctness: extracted values must be supported by source evidence.
- Traceability: every result maps to a page and extraction run.
- Reliability: failed jobs can be retried safely.
- Security: tenant isolation and document authorization.
- Maintainability: clear modules and typed interfaces.
- Evaluability: metrics are computed against labelled examples.
- Cost control: model calls, OCR and embeddings are measured per document.

## 10. Release boundary

The first useful release supports PDF upload, text/OCR ingestion, contract-field extraction, evidence display, human correction, persistence, and contract search. Advanced RAG, frontend polish, multi-tenant security hardening, observability and deployment are built in later roadmap tasks.
