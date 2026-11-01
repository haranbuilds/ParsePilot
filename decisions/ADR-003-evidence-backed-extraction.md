# ADR-003: Require Evidence for Extracted Fields

- Status: Accepted
- Context: Contract data must be traceable; unsupported model-generated values are unsafe.
- Decision: Every non-null extracted field must carry source evidence (page and quote, with offsets or coordinates when available). Unsupported or ambiguous values are flagged for review or abstained.
- Consequences: More implementation and validation work, but improved traceability and reviewability. Evidence verification itself must be evaluated.
