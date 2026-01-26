---

description: "Tasks for Azure DevOps Pipeline Aggregation (Mocked Data)"

---

# Tasks: Azure DevOps Pipeline Aggregation (Mocked Data)

**Input**: Design documents from `specs/001-aggregate-ado-pipelines/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Tests are OPTIONAL and not requested; focus is on deterministic mocked outputs and acceptance checks.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Baseline CLI readiness and help coverage

- [X] T001 Update CLI help and usage text in `get-build-durations.py` to reflect multi-org input and summary output
- [X] T002 [P] Add `--summary_output` argument in `get-build-durations.py` `parse_args()` (default `<output>.summary.md` when `--output` provided)
- [X] T003 [P] Support repeated `--org-url` flags and comma-separated list parsing in `get-build-durations.py`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core logic and writers that all user stories depend on

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [X] T004 Implement deterministic mock data generators in `get-build-durations.py` (org → projects → pipelines → runs → jobs)
- [X] T005 [P] Implement per-pipeline aggregation utilities in `get-build-durations.py` (run_count, avg/total durations)
- [X] T006 [P] Implement pipeline CSV writer in `get-build-durations.py` following `contracts/pipeline_aggregate.csv.schema.yaml`
- [X] T007 Implement summary markdown writer in `get-build-durations.py` following `contracts/summary.md.schema.yaml`
- [X] T008 Ensure logging to stderr (`--verbose` gates) and stable non-zero exit codes in `get-build-durations.py`
- [X] T009 Honor `--threads` and `--delay` across mocked processing flows in `get-build-durations.py`

**Checkpoint**: Foundation ready — user story implementation can now begin in parallel

---

## Phase 3: User Story 1 — Aggregate pipeline metrics by date range (P1) 🎯 MVP

**Goal**: Produce pipeline-level CSV aggregates for a single org and date range

**Independent Test**: Run with one org URL, valid PAT, date range, and `--output`; verify CSV rows per pipeline and deterministic values per spec

### Implementation for User Story 1

- [X] T010 [P] [US1] Normalize and validate `--begin`/`--end` to ISO 8601 UTC in `get-build-durations.py`
- [X] T011 [US1] Process single org projects/pipelines using mocks; compute per-pipeline aggregates in `get-build-durations.py`
- [X] T012 [US1] Write pipeline CSV to `--output` or stdout (stable column order) in `get-build-durations.py`
- [X] T013 [US1] Write human-readable summary to `<output>.summary.md` (or `--summary_output`) in `get-build-durations.py`
- [X] T014 [US1] Validate required args/PAT and fail fast with remediation guidance in `get-build-durations.py`

**Checkpoint**: User Story 1 fully functional and testable independently

---

## Phase 4: User Story 2 — Job-level CSV with pool details (P2)

**Goal**: Emit per-job CSV including agent pool details when requested

**Independent Test**: Run with `--jobs_output` alongside P1 inputs; confirm job rows with pool id/name and references

### Implementation for User Story 2

- [X] T015 [P] [US2] Extend mocks to emit job-level records with `pool_id`/`pool_name` in `get-build-durations.py`
- [X] T016 [US2] Write job-level CSV to `--jobs_output` in `get-build-durations.py` following `contracts/job_details.csv.schema.yaml`
- [X] T017 [US2] Ensure run→job mapping and stable column order; include org/project/pipeline references in `get-build-durations.py`

**Checkpoint**: User Story 2 functional and independently testable

---

## Phase 5: User Story 3 — Multi-organization aggregation (P3)

**Goal**: Consolidate pipeline aggregates across multiple orgs in one CSV and summary

**Independent Test**: Provide multiple org URLs; verify consolidated CSV and summary include per-org attribution and accurate totals

### Implementation for User Story 3

- [X] T018 [P] [US3] Normalize multi-org input (flags/list) into list in `get-build-durations.py`
- [X] T019 [US3] Aggregate per-org pipelines into a single CSV (include `org_url` column) in `get-build-durations.py`
- [X] T020 [US3] Generate consolidated summary sections: global, per-org, per-project in `get-build-durations.py`
- [X] T021 [US3] Validate combined totals/durations for determinism across orgs in `get-build-durations.py`

**Checkpoint**: All user stories independently functional

---

## Phase N: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [X] T022 [P] Update examples in `specs/001-aggregate-ado-pipelines/quickstart.md` for multi-org and summary outputs
- [X] T023 Confirm CSV schemas and help text alignment; update usage in `get-build-durations.py`
- [X] T024 [P] Harden error paths: consistent exit codes and concise stderr messages in `get-build-durations.py`
- [X] T025 Minor refactors for readability without adding dependencies in `get-build-durations.py`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion — BLOCKS all user stories
- **User Stories (Phase 3+)**: All depend on Foundational completion
  - User stories can proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P3)
- **Polish (Final Phase)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) — no dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) — independently testable
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) — independently testable

### Within Each User Story

- Models/mock generators before aggregations
- Aggregations before CSV/summary writers
- Core implementation before validations

### Parallel Opportunities

- Setup tasks `T002`, `T003` can run in parallel
- Foundational tasks `T005`, `T006` can run in parallel
- Within US1, `T010` can run in parallel with `T014`
- Within US2, `T015` can run in parallel with `T017`
- Within US3, `T018` can run in parallel with `T021`

---

## Implementation Strategy

- **MVP First**: Deliver US1 end-to-end (single org → pipeline CSV → summary file) with deterministic mocks.
- **Incremental Delivery**: Add US2 job-level CSV, then US3 multi-org consolidation.
- **Stability**: Keep CSV columns order stable; document any changes via version notes.
