# Data Model: Azure DevOps Pipeline Aggregation (Mocked Data)

## Entities

- **Organization**
  - `org_url: str`
  - `org_id: str` (derived or placeholder)

- **Project**
  - `project_id: str`
  - `project_name: str`
  - `org_url: str`

- **Pipeline**
  - `pipeline_id: int`
  - `pipeline_name: str`
  - `project_id: str`
  - `org_url: str`

- **PipelineRun**
  - `run_id: int`
  - `pipeline_id: int`
  - `started_at: datetime`
  - `finished_at: datetime`
  - `duration_seconds: int`
  - `status: str`
  - `project_id: str`
  - `org_url: str`

- **Job**
  - `job_id: int`
  - `run_id: int`
  - `job_name: str`
  - `duration_seconds: int`
  - `pool_id: int`
  - `pool_name: str`

- **Aggregate** (derived)
  - Per-pipeline: `run_count`, `avg_duration_seconds`, `total_duration_seconds`
  - Per-project/org/global: sums and averages across pipelines/runs

## Relationships

- Organization 1..* Projects
- Project 1..* Pipelines
- Pipeline 1..* PipelineRuns
- PipelineRun 1..* Jobs
- Jobs 1..1 AgentPool

## Validation Rules

- Date range: `begin < end`; both parseable to ISO 8601 UTC.
- `org_url` must be non-empty and well-formed.
- Aggregates computed only from runs in the given date window.

## State Transitions

- `PipelineRun.status`: one of `succeeded|failed|canceled` (mocked); affects summaries if filters added later.
