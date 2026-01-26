# Implementation Plan: Azure DevOps Pipeline Aggregation (Mocked Data)

**Branch**: `001-aggregate-ado-pipelines` | **Date**: 2026-01-23 | **Spec**: `specs/001-aggregate-ado-pipelines/spec.md`
**Input**: Feature specification from `/specs/001-aggregate-ado-pipelines/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

- Primary: Produce pipeline-level CSV aggregates across projects and orgs for a given date range using deterministic mocked Azure DevOps responses aligned to `API_VERSION`.
- Optional: Produce job-level CSV with agent pool details.
- New requirement: Produce a human-readable summary file with total build/run counts and total/average durations at org, project, and global levels.
- Approach: Minimal external libraries (retain `requests` already used by the script), standard library for CSV/arg parsing, deterministic mock generators keyed by inputs.

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

**Language/Version**: Python 3.10+  
**Primary Dependencies**: `requests` (HTTP), stdlib (`argparse`, `csv`, `datetime`, `concurrent.futures`, `time`, `os`, `sys`)  
**Storage**: N/A (outputs as files: CSV + summary text/markdown)  
**Testing**: Manual smoke runs; optional `pytest` later if requested  
**Target Platform**: Windows PowerShell 5.1; cross-platform friendly when feasible  
**Project Type**: Single-script CLI (`get-build-durations.py`)  
**Performance Goals**: Handle thousands of runs in mocked datasets; complete under a few minutes; streaming CSV writes  
**Constraints**: Minimal external libs; deterministic outputs; stable column order; non-interactive config  
**Scale/Scope**: Multiple orgs; many projects/pipelines; job-level optional details

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- Simple single-file CLI: PASS — continue using `get-build-durations.py`.
- Deterministic Text I/O: PASS — CSV to stdout/file; summary to separate file; logs to stderr.
- Non-Interactive Configuration: PASS — `--pat` or `AZDO_PAT` env; fail fast if missing.
- Minimal Logging & Exit Codes: PASS — `--verbose` gating; documented non-zero failures.
- Minimal Dependencies & Security: PASS — stdlib + `requests`; no secret persistence; polite delays.

Re-check after Phase 1 design: PASS — documentation and contracts align to constitution; summary output is separate, human-readable, and does not change CSV contracts.

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->

```text
Repository root
├── get-build-durations.py         # CLI script (existing)
├── input.txt                      # example input (existing)
├── rest.txt                       # example input (existing)
└── specs/001-aggregate-ado-pipelines/
  ├── spec.md
  ├── plan.md
  ├── research.md                # Phase 0 output
  ├── data-model.md              # Phase 1 output
  ├── quickstart.md              # Phase 1 output
  └── contracts/                 # Phase 1 output
    ├── pipeline_aggregate.csv.schema.yaml
    ├── job_details.csv.schema.yaml
    └── summary.md.schema.yaml
```

**Structure Decision**: Single-script CLI with documentation under `specs/001-aggregate-ado-pipelines/`; contracts describe CSV and summary outputs.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |

No violations expected under current plan.
