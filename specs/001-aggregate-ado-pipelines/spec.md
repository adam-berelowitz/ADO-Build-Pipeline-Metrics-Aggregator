# Feature Specification: Azure DevOps Pipeline Aggregation

**Feature Branch**: `[001-aggregate-ado-pipelines]`  
**Created**: 2026-01-23  
**Updated**: 2026-01-26 (Added real API implementation)  
**Status**: Implementation Complete  
**Input**: User description: "This is a prototype script that allows a user to pass in an azure devops organization url or list of organization urls, a personal access token, date ranges, as well as several other parameters.  The purpose of this script is to use the API version defined in the script to retrieve data from the Azure DevOps APIs to aggregate Azure DevOps pipeline related data across multiple pipelines within a project, across multiple projects within multiple organizations.  Some of the features of this script may not be implemented yet.  I will be reviewing this script for it's functionality, improving it's implemention, verifying it's accuracy against a running Azure DevOps instance (based on the ADO organization URLs provided) and implementing any missing functionality.  All data retrieved from the ADO APIs is mocked - you dont need to pull anything from a real API."

**Implementation Update**: The script now supports both mocked data (default) and real Azure DevOps API connectivity via the `--live` flag, enabling verification against actual Azure DevOps instances.

## User Scenarios & Testing *(mandatory)*

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE - meaning if you implement just ONE of them,
  you should still have a viable MVP (Minimum Viable Product) that delivers value.
  
  Assign priorities (P1, P2, P3, etc.) to each story, where P1 is the most critical.
  Think of each story as a standalone slice of functionality that can be:
  - Developed independently
  - Tested independently
  - Deployed independently
  - Demonstrated to users independently
-->

### User Story 1 - Aggregate pipeline metrics by date range (Priority: P1)

As an operator, I run the script against one Azure DevOps organization and a date range to receive a CSV containing pipeline-level aggregate metrics (e.g., total runs and average duration) for all pipelines in all projects within that organization.

**Why this priority**: This delivers the core value—an actionable, automation-friendly summary of pipeline behavior for a given window.

**Independent Test**: Execute the script with one org URL, a valid PAT (via arg or env), begin/end dates, and `--output`. By default (mock mode), confirm a single CSV is produced containing rows per pipeline with expected fields and deterministic values. With `--live` flag, verify against real Azure DevOps API data.

**Acceptance Scenarios**:

1. **Given** a single org URL and date range, **When** I run the script with `--output` set, **Then** a CSV is written containing one row per pipeline with org, project, pipeline name/id, run count, and average duration.
2. **Given** missing required args (e.g., `--org-url`), **When** I run the script, **Then** the script exits non-zero and prints a concise error to stderr explaining what is missing.
3. **Given** the `--live` flag is used with valid credentials, **When** I run the script, **Then** real Azure DevOps API data is fetched and processed instead of mocked data.
4. **Given** `--live` flag is used but `requests` module is not available, **When** I run the script, **Then** the script exits with error guidance to install the requests module.

---

### User Story 2 - Job-level CSV with pool details (Priority: P2)

As an operator, I optionally request a second CSV containing per-job details (including the agent pool assigned) for each pipeline run within the date range.

**Why this priority**: Job-level data supports capacity planning and pool utilization analysis.

**Independent Test**: Execute the script with `--jobs_output` in addition to P1 inputs. Confirm a second CSV is produced with one row per job including job name/id, duration, assigned pool name/id, and references to the run/pipeline. In live mode (`--live`), agent pool details are fetched from real Azure DevOps APIs.

**Acceptance Scenarios**:

1. **Given** `--jobs_output` is provided, **When** I run the script, **Then** a job-level CSV is written and includes pool identifiers for 100% of jobs in mocked data.
2. **Given** `--live` and `--jobs_output` are used, **When** I run the script, **Then** real agent pool information is fetched from Azure DevOps APIs and included in job records.

---

### User Story 3 - Multi-organization aggregation (Priority: P3)

As an operator, I provide more than one organization URL and receive a single consolidated CSV covering pipelines across all specified organizations in the date range.

**Why this priority**: Enables cross-org reporting without manual merging.

**Independent Test**: Provide multiple org URLs (via repeated flag or list input). Confirm the resulting pipeline-level CSV includes records across all orgs with clear org identifiers. Works in both mock mode (default) and live API mode (`--live`).

**Acceptance Scenarios**:

1. **Given** two org URLs, **When** I run the script, **Then** the pipeline CSV contains rows attributed to each org and totals reflect the combined mocked dataset.
2. **Given** multiple org URLs and `--live` flag, **When** I run the script, **Then** real data is fetched from all specified organizations and consolidated into unified CSV outputs.

---

[Add more user stories as needed, each with an assigned priority]

### Edge Cases

- Empty results: No pipeline runs in the date range → script returns a CSV with headers and zero rows, exits `0`.
- Invalid or missing PAT: If `--pat` omitted and `AZDO_PAT` not set, script exits non-zero with guidance to provide credentials.
- Invalid date format: If `--begin`/`--end` cannot be parsed to ISO 8601, script exits non-zero with parsing help.
- Large datasets: Mocked data generation scales to many runs; outputs remain deterministic and streaming-friendly.
- Rate limiting: Respect a user-provided `--delay` between requests (even in mock mode) to simulate polite API usage.
- **Live API failures**: When using `--live` flag, network issues, invalid PATs, or API errors result in non-zero exit with actionable error messages.
- **Missing dependencies**: If `--live` is used but `requests` module is not available, script exits with installation guidance.
- **API permissions**: When using `--live`, insufficient PAT permissions for projects/builds result in clear error messages indicating required permissions.
- **Large live datasets**: Live API mode honors `--delay` and `--threads` to manage API rate limits and avoid overwhelming Azure DevOps services.

## Requirements *(mandatory)*

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right functional requirements.
-->

### Functional Requirements

- **FR-001**: Users MUST be able to provide one or more Azure DevOps organization URLs and a date range (begin inclusive, end exclusive) to produce pipeline-level aggregates.
- **FR-002**: The script MUST accept a PAT via `--pat` or, if omitted, read from `AZDO_PAT` environment variable.
- **FR-003**: The script MUST produce a pipeline-level CSV to `--output` or stdout when `--output` is omitted; columns are stable and documented in `--help`.
- **FR-004**: When `--jobs_output` is provided, the script MUST produce a second CSV with job-level rows including agent pool identification.
- **FR-005**: The script MUST honor `API_VERSION` defined in the script for selecting Azure DevOps API version semantics (mocked responses align to this version).
- **FR-006**: The script MUST support `--threads` for parallel project processing and `--delay` (milliseconds) to throttle requests, even when using mocked data.
- **FR-007**: The script MUST support `--max-projects` to limit processed projects for sampling/debugging.
- **FR-008**: The script MUST emit verbose diagnostics to stderr only when `--verbose` is set; otherwise, only warnings/errors.
- **FR-009**: All errors MUST return non-zero exit codes with actionable messages; successful runs MUST exit `0`.
- **FR-010**: Mock mode MUST be the default: no real API calls are made by default; generated results are deterministic given the same inputs. Live API mode MUST be available via `--live` flag for connecting to real Azure DevOps instances.
- **FR-011**: When using `--live` flag, the script MUST make authenticated requests to real Azure DevOps APIs using the specified API_VERSION and handle common API failures gracefully.
- **FR-012**: The script MUST validate that the `requests` module is available when `--live` mode is requested, exiting with clear installation guidance if missing.
- **FR-013**: Live API mode MUST fetch real projects, pipeline definitions, build runs, job timelines, and agent pool information from Azure DevOps using the provided PAT credentials.
- **FR-014**: Live API mode MUST respect `--delay` between API requests to avoid rate limiting and MUST use `--threads` to control concurrent project processing.
- **FR-015**: Live API mode MUST cache agent pool information per organization to minimize redundant API calls when processing job-level data.

### Key Entities *(include if feature involves data)*

- **Organization**: Identifier, URL.
- **Project**: Identifier, name, associated organization.
- **Pipeline**: Identifier, name, associated project.
- **PipelineRun**: Run id, start/end timestamps, duration, status, associated pipeline/project/org.
- **Job**: Job id, name, duration, associated run, assigned agent pool.
- **AgentPool**: Pool id, name.

## Success Criteria *(mandatory)*

<!--
  ACTION REQUIRED: Define measurable success criteria.
  These must be technology-agnostic and measurable.
-->

### Measurable Outcomes

- **SC-001**: Users receive a pipeline-level CSV for a single org and date range in one run, with rows per pipeline and accurate aggregate counts and average durations.
- **SC-002**: When `--jobs_output` is used, 100% of job rows include pool identifiers and reference their parent run/pipeline correctly.
- **SC-003**: CLI help (`--help`) lists all supported flags and example usage; first-time users can run successfully without external docs.
- **SC-004**: Error cases (missing org URL, missing PAT, invalid dates) return non-zero exit codes and clear remediation messages without stack traces.
- **SC-005**: Live API mode (`--live`) successfully connects to real Azure DevOps instances and produces identical CSV structure to mock mode with real data.
- **SC-006**: Live API mode handles authentication errors, network failures, and API rate limits gracefully with actionable error messages.
- **SC-007**: Both mock and live modes produce deterministic output structure with identical column ordering and data types.

## Assumptions

- By default, all Azure DevOps data is mocked for testing and development; no live API calls occur unless `--live` flag is specified.
- When using `--live` flag, the script connects to real Azure DevOps APIs and requires valid credentials and network connectivity.
- Deterministic outputs: same inputs produce the same CSV content in mock mode; live mode outputs reflect real-time API data.
- Users run on Windows PowerShell (5.1) with Python 3.10+; cross-platform behavior is a stretch goal.
- Multiple org support is provided via repeated flags or list input; the exact arg style can be finalized during implementation without changing user-facing intent.
- Live API mode requires the `requests` Python module to be available; clear error guidance is provided if missing.
- Real Azure DevOps API responses conform to the API_VERSION specified in the script (currently "7.0").
- PAT credentials used in live mode have sufficient permissions to read projects, pipelines, builds, and agent pools from specified organizations.
