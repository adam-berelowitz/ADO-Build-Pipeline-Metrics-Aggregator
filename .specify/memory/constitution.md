<!--
Sync Impact Report
- Version change: unversioned → 1.0.0
- Modified principles: [PRINCIPLE_1_NAME] → Simple, Single-File CLI; [PRINCIPLE_2_NAME] → Deterministic Text I/O; [PRINCIPLE_3_NAME] → Non-Interactive Configuration; [PRINCIPLE_4_NAME] → Minimal Logging & Exit Codes; [PRINCIPLE_5_NAME] → Minimal Dependencies & Security
- Added sections: None
- Removed sections: None
- Templates reviewed:
	- .specify/templates/plan-template.md — ✅ reviewed (no changes needed)
	- .specify/templates/spec-template.md — ✅ reviewed (no changes needed)
	- .specify/templates/tasks-template.md — ✅ reviewed (no changes needed)
	- .specify/templates/checklist-template.md — ✅ reviewed (no changes needed)
	- .specify/templates/commands/ — ⚠ pending (directory not present; no command templates to review)
- Deferred items:
	- TODO(RATIFICATION_DATE): Original adoption date unknown; set when known.
-->

# Get Build Durations Constitution
<!-- Project is a single Python CLI script invoked from the shell. -->

## Core Principles

### Simple, Single-File CLI
- MUST run as a single Python script callable via shell.
- MUST provide `--help` usage and clear argument flags.
- MUST avoid interactive prompts; script is non-daemon, non-service.
- SHOULD keep logic straightforward and small; prefer functions over classes.
Rationale: A simple CLI lowers operational overhead and makes usage obvious.

### Deterministic Text I/O
- Inputs via CLI args and/or stdin; outputs via stdout (CSV or text).
- Errors and logs go to stderr; no mixed streams.
- Exit code `0` on success; non-zero on failure with message.
- Output columns and order are stable across versions unless a MAJOR bump.
Rationale: Deterministic contracts enable piping, scripting, and automation.

### Non-Interactive Configuration
- Secrets (e.g., tokens) read from env vars when args omitted.
- MUST fail fast with actionable error if required inputs are missing.
- MUST support explicit args to override environment values.
- MUST avoid persistent storage of secrets or credentials.
Rationale: Non-interactive config is safer and works in CI and shell pipelines.

### Minimal Logging & Exit Codes
- `--verbose` enables timestamped debug logs to stderr.
- Default mode logs only warnings/errors; no noisy info logs.
- Fatal errors include a concise cause and remediation hint.
- Exit codes are documented and stable.
Rationale: Minimal logging keeps CLI usage clean while aiding troubleshooting.

### Minimal Dependencies & Security
- Prefer Python standard library; only add dependencies when essential.
- Network calls MUST use timeouts and handle rate limits politely.
- No local credential caches; secrets only in process memory.
- Target runtime: Python 3.10+ on Windows PowerShell; stay cross-platform where feasible.
Rationale: Fewer dependencies and basic security reduce risk and friction.

## Additional Constraints
- Runtime: Python 3.10+; compatible with Windows PowerShell shells.
- CSV outputs MUST be parseable by common tools (Excel, `csv` module).
- CLI flags are backward compatible; breaking changes require MAJOR bump.
- Respect API rate limits and add optional `--delay` for throttling.
- Provide sample usage in `--help`; keep `input.txt`/`rest.txt` as examples when applicable.

## Development Workflow
- Small changes: update usage (`--help`) and keep outputs stable.
- Additions to CLI: prefer optional flags; document defaults.
- Before merging: run a local smoke test against sample inputs; verify exit codes.
- If outputs change, update downstream docs/examples and note in changelog.

## Governance
- This constitution supersedes other ad-hoc practices for the script.
- Amendments: propose via PR with rationale, impact analysis, and version bump type.
- Versioning policy (constitution): Semantic — MAJOR for breaking governance changes; MINOR for new sections/principles; PATCH for clarifications.
- Compliance: PR reviewers verify principles are upheld (simplicity, I/O, config, logging, deps/security).
- Migration notes are REQUIRED for MAJOR changes affecting CLI flags or output contracts.

**Version**: 1.0.0 | **Ratified**: TODO(RATIFICATION_DATE): original adoption date unknown | **Last Amended**: 2026-01-23
# [PROJECT_NAME] Constitution
<!-- Example: Spec Constitution, TaskFlow Constitution, etc. -->

## Core Principles

### [PRINCIPLE_1_NAME]
<!-- Example: I. Library-First -->
[PRINCIPLE_1_DESCRIPTION]
<!-- Example: Every feature starts as a standalone library; Libraries must be self-contained, independently testable, documented; Clear purpose required - no organizational-only libraries -->

### [PRINCIPLE_2_NAME]
<!-- Example: II. CLI Interface -->
[PRINCIPLE_2_DESCRIPTION]
<!-- Example: Every library exposes functionality via CLI; Text in/out protocol: stdin/args → stdout, errors → stderr; Support JSON + human-readable formats -->

### [PRINCIPLE_3_NAME]
<!-- Example: III. Test-First (NON-NEGOTIABLE) -->
[PRINCIPLE_3_DESCRIPTION]
<!-- Example: TDD mandatory: Tests written → User approved → Tests fail → Then implement; Red-Green-Refactor cycle strictly enforced -->

### [PRINCIPLE_4_NAME]
<!-- Example: IV. Integration Testing -->
[PRINCIPLE_4_DESCRIPTION]
<!-- Example: Focus areas requiring integration tests: New library contract tests, Contract changes, Inter-service communication, Shared schemas -->

### [PRINCIPLE_5_NAME]
<!-- Example: V. Observability, VI. Versioning & Breaking Changes, VII. Simplicity -->
[PRINCIPLE_5_DESCRIPTION]
<!-- Example: Text I/O ensures debuggability; Structured logging required; Or: MAJOR.MINOR.BUILD format; Or: Start simple, YAGNI principles -->

## [SECTION_2_NAME]
<!-- Example: Additional Constraints, Security Requirements, Performance Standards, etc. -->

[SECTION_2_CONTENT]
<!-- Example: Technology stack requirements, compliance standards, deployment policies, etc. -->

## [SECTION_3_NAME]
<!-- Example: Development Workflow, Review Process, Quality Gates, etc. -->

[SECTION_3_CONTENT]
<!-- Example: Code review requirements, testing gates, deployment approval process, etc. -->

## Governance
<!-- Example: Constitution supersedes all other practices; Amendments require documentation, approval, migration plan -->

[GOVERNANCE_RULES]
<!-- Example: All PRs/reviews must verify compliance; Complexity must be justified; Use [GUIDANCE_FILE] for runtime development guidance -->

**Version**: [CONSTITUTION_VERSION] | **Ratified**: [RATIFICATION_DATE] | **Last Amended**: [LAST_AMENDED_DATE]
<!-- Example: Version: 2.1.1 | Ratified: 2025-06-13 | Last Amended: 2025-07-16 -->
