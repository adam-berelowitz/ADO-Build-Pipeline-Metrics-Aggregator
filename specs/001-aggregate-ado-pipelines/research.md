# Phase 0 Research: Azure DevOps Pipeline Aggregation (Mocked Data)

## Decisions

- **Summary file format**: Markdown (`.md`) human-readable report with sections per org and project plus global totals.
- **Summary file location/name**: If `--output` provided as `X.csv`, summary defaults to `X.summary.md`; if stdout used, require `--summary_output` path.
- **CSV schemas**: Stable column order documented; pipeline CSV and job CSV column names defined below.
- **Mocking strategy**: Deterministic generation keyed by `(org_url, begin, end, API_VERSION)`; fixed seed per org; durations sampled from bounded distributions; counts derived from seed.
- **Multiple org input**: Accept repeated `--org-url` flags or a comma-separated list; internally normalize to a list.
- **Delays and threading**: Honor `--delay` between mock "requests" and `--threads` for parallel per-project processing to simulate real behavior.
- **Dependencies**: Retain `requests` (used by script); avoid adding new libraries; use stdlib for CSV and Markdown assembly.

## Rationale

- Markdown is widely readable and diff-friendly; fits "human-readable" requirement.
- Deterministic mocks ensure identical outputs for identical inputs, enabling review without live ADO.
- Repeated flags and comma-separated lists provide ergonomic multi-org input without breaking existing arg parsing.

## Alternatives Considered

- **Plain text summary**: Simpler but less structured; Markdown headings improve scanability.
- **JSON summary**: Machine-readable but less human-friendly; CSVs already serve machine use-cases.
- **YAML schemas**: Chosen for readability; could use JSON Schema, but YAML is concise in contracts docs.

## Unresolved / Follow-ups

- None blocking Phase 1. If later tests request cross-platform guarantees, revisit Windows-specific assumptions.
