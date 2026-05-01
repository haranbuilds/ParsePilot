# ParsePilot Database Design (Initial)

## Core entities

### organizations
- id (UUID, primary key)
- name
- created_at
- updated_at

### users
- id (UUID, primary key)
- organization_id (foreign key)
- email (unique within the intended identity scope)
- password_hash or external identity subject
- role
- is_active
- created_at

### documents
- id (UUID, primary key)
- organization_id (foreign key)
- uploaded_by (user foreign key)
- original_filename
- storage_key
- mime_type
- file_size_bytes
- sha256
- status
- created_at
- updated_at

### document_pages
- id (UUID, primary key)
- document_id (foreign key)
- page_number
- extracted_text
- extraction_method (`native_pdf` or `ocr`)
- extraction_metadata (JSONB)
- created_at

### document_chunks
- id (UUID, primary key)
- document_id (foreign key)
- page_number
- chunk_index
- text
- start_offset
- end_offset
- embedding_model
- created_at

### extraction_runs
- id (UUID, primary key)
- document_id (foreign key)
- schema_version
- provider
- model
- prompt_version
- status
- started_at
- completed_at
- error_code

### extracted_fields
- id (UUID, primary key)
- extraction_run_id (foreign key)
- field_name
- raw_value (JSONB)
- normalized_value (JSONB)
- validation_status
- review_status
- created_at

### evidence_spans
- id (UUID, primary key)
- extracted_field_id (foreign key)
- page_number
- quote
- start_offset
- end_offset
- bounding_box (JSONB, nullable)
- verification_status

### review_events
- id (UUID, primary key)
- extracted_field_id (foreign key)
- reviewer_user_id (foreign key)
- action
- previous_value (JSONB)
- new_value (JSONB)
- reason
- created_at

## Design rules

- Use UUID identifiers for externally exposed resources.
- Index foreign keys and common filter columns.
- Add organization_id filtering to tenant-owned data.
- Preserve extraction runs and review history; avoid destructive overwrites.
- Use JSONB only for flexible values/metadata, not as a substitute for all relational structure.
- Define uniqueness constraints for page numbers per document and chunk indices per document.
- Qdrant is an index, not the source of truth; PostgreSQL and file storage remain authoritative.
- Migrations are version-controlled through Alembic.
