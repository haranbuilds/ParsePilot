# ParsePilot API Design (Proposed)

Base path: `/api/v1`

All endpoints require authentication once identity is implemented. Every document query must enforce organization and document-level authorization.

## Documents

### `POST /documents`
Upload a PDF contract. Returns document ID, status and checksum metadata. Use multipart form upload. Support an idempotency key.

### `GET /documents`
List only documents visible to the caller. Support pagination, status filters and safe sorting.

### `GET /documents/{document_id}`
Return metadata and current lifecycle status.

### `GET /documents/{document_id}/status`
Return current stage, progress information, timestamps and safe failure code.

### `POST /documents/{document_id}/extract`
Request extraction or re-extraction. Return an operation/run ID. Must be idempotent for repeated identical requests where appropriate.

### `GET /documents/{document_id}/extractions`
List immutable extraction runs and their model/schema versions.

### `GET /documents/{document_id}/fields`
Return current extracted/reviewed field values and evidence references.

### `POST /documents/{document_id}/review`
Accept reviewer decisions for one or more fields. Record reviewer identity, timestamp, prior value, corrected value and reason.

## Search

### `POST /search`
Input: query, optional document filters and pagination. Output: authorized matching passages with document ID, page, snippet and retrieval metadata.

## Health

### `GET /health/live`
Indicates that the API process is alive.

### `GET /health/ready`
Checks readiness of required dependencies separately from liveness.

## API conventions

- Use consistent error envelopes with stable machine-readable error codes.
- Never return stack traces or secret values.
- Validate request and response schemas.
- Use pagination for collections.
- Use UTC timestamps in ISO 8601 format.
- Distinguish client errors, dependency failures and internal errors.
- Document status codes and examples in OpenAPI.
- Do not expose internal storage paths.
