# Azure DevOps Pipeline Aggregation Tool

A Python CLI tool that aggregates Azure DevOps pipeline (build) duration metrics across multiple organizations and projects. Supports both **live Azure DevOps API data** and **deterministic mock data** for testing and demonstration purposes.

## Features

- **Multi-organization support**: Process multiple Azure DevOps organizations in a single run
- **Live API or Mock data**: Use real Azure DevOps APIs with `--live` flag or deterministic mocks
- **Pipeline aggregation**: Generate per-pipeline statistics including run counts and duration metrics
- **Job-level details**: Optional detailed job information with agent pool assignments
- **Human-readable summaries**: Markdown reports with organization and project breakdowns
- **Deterministic mocking**: Consistent test data generation based on input parameters (mock mode)
- **CSV output**: Machine-readable data following documented schemas
- **Parallel processing**: Configurable threading for improved performance

## Prerequisites

- **Python**: 3.10 or higher
- **Platform**: Windows PowerShell 5.1 (cross-platform compatible)
- **Personal Access Token**: Azure DevOps PAT (required for live mode, can be mocked for testing)
- **Dependencies**: `requests` module (required for live mode: `pip install requests`)

## Installation

1. Clone or download this repository
2. For **mock mode**: No additional dependencies required (uses Python standard library)
3. For **live mode**: Install requests: `pip install requests`
4. Set `AZDO_PAT` environment variable or use `--pat` flag

## Quick Start

### Basic Usage - Single Organization (Mock Data)

```powershell
# Generate pipeline aggregates with summary using mock data
python get-build-durations.py --org-url https://dev.azure.com/myorg --begin 2024-01-01 --end 2024-01-31 --output pipelines.csv

# This creates:
# - pipelines.csv (pipeline aggregation data)
# - pipelines.summary.md (human-readable summary)
```

### Live Azure DevOps API Data

```powershell
# Get real data from Azure DevOps (requires valid PAT and requests module)
pip install requests
python get-build-durations.py --live --org-url https://dev.azure.com/myorg --begin 2024-01-01 --end 2024-01-31 --output real_pipelines.csv
```

### Multi-Organization with Job Details

```powershell
# Process multiple organizations with detailed job information
python get-build-durations.py `
  --org-url https://dev.azure.com/org1 `
  --org-url https://dev.azure.com/org2 `
  --begin 2024-01-01T00:00:00 `
  --end 2024-02-01T00:00:00 `
  --output pipelines.csv `
  --jobs_output jobs.csv `
  --summary_output consolidated_summary.md `
  --threads 8 `
  --verbose
```

### Output to stdout

```powershell
# Write CSV to stdout, specify summary separately
python get-build-durations.py --org-url https://dev.azure.com/myorg --begin 2024-01-01 --end 2024-01-31 --summary_output summary.md > output.csv
```

## Command Line Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--org-url` | ✓ | Azure DevOps organization URL (can specify multiple times) |
| `--begin` | ✓ | Start date (inclusive): `2024-01-01` or `2024-01-01T00:00:00` |
| `--end` | ✓ | End date (exclusive): `2024-01-31` or `2024-01-31T00:00:00` |
| `--pat` | | Personal Access Token (or set `AZDO_PAT` env var) |
| `--output` | | Output CSV file path (defaults to stdout) |
| `--jobs_output` | | Optional jobs CSV file path |
| `--summary_output` | | Summary markdown file path (auto-generated if `--output` provided) |
| `--threads` | | Number of worker threads (default: 4) |
| `--max-projects` | | Limit number of projects processed (for testing) |
| `--delay` | | Delay in ms between requests (default: 0) |
| `--verbose` | | Enable verbose logging to stderr |
| `--live` | | Use live Azure DevOps API instead of mocked data (requires requests module) |

## Output Files

### Pipeline CSV (`pipelines.csv`)
Main aggregation file with one row per pipeline:

| Column | Type | Description |
|--------|------|-------------|
| `org_url` | string | Organization URL |
| `project_id` | string | Project identifier |
| `project_name` | string | Project display name |
| `pipeline_id` | integer | Pipeline identifier |
| `pipeline_name` | string | Pipeline display name |
| `run_count` | integer | Number of runs in date range |
| `avg_duration_seconds` | integer | Average run duration |
| `total_duration_seconds` | integer | Sum of all run durations |

### Jobs CSV (`jobs.csv`) - Optional
Detailed job information when `--jobs_output` specified:

| Column | Type | Description |
|--------|------|-------------|
| `org_url` | string | Organization URL |
| `project_id` | string | Project identifier |
| `pipeline_id` | integer | Pipeline identifier |
| `run_id` | integer | Pipeline run identifier |
| `job_id` | integer | Job identifier |
| `job_name` | string | Job display name |
| `duration_seconds` | integer | Job duration |
| `pool_id` | integer | Agent pool identifier |
| `pool_name` | string | Agent pool name |

### Summary Markdown (`summary.md`)
Human-readable report including:
- **Overview**: Total organizations, projects, pipelines, runs
- **By Organization**: Aggregates per organization
- **By Project**: Aggregates per project within each organization

## Authentication

The tool requires an Azure DevOps Personal Access Token (PAT) for authentication:

### Live Mode (Real API Data)
Requires a valid PAT with appropriate permissions:
- **Build**: Read
- **Project and team**: Read
- **Agent Pools**: Read (if using `--jobs_output`)

### Mock Mode (Default)
Any non-empty PAT value will work since no actual API calls are made.

### Option 1: Environment Variable (Recommended)
```powershell
$env:AZDO_PAT = "your_pat_token_here"
python get-build-durations.py --org-url https://dev.azure.com/myorg --begin 2024-01-01 --end 2024-01-31
```

### Option 2: Command Line Flag
```powershell
python get-build-durations.py --pat "your_pat_token_here" --org-url https://dev.azure.com/myorg --begin 2024-01-01 --end 2024-01-31
```

**Note**: 
- **Mock mode (default)**: Any non-empty PAT value will work for testing
- **Live mode (`--live` flag)**: Requires valid PAT and `requests` module

## Examples

### Example 1: Monthly Report for Single Organization (Mock)
```powershell
$env:AZDO_PAT = "mock_token"
python get-build-durations.py `
  --org-url https://dev.azure.com/contoso `
  --begin 2024-01-01 `
  --end 2024-02-01 `
  --output january_pipelines.csv `
  --verbose
```

### Example 1b: Monthly Report for Single Organization (Live API)
```powershell
$env:AZDO_PAT = "your_real_pat_token"
pip install requests
python get-build-durations.py `
  --live `
  --org-url https://dev.azure.com/contoso `
  --begin 2024-01-01 `
  --end 2024-02-01 `
  --output january_real_pipelines.csv `
  --verbose
```

### Example 2: Multi-Organization Quarterly Report
```powershell
python get-build-durations.py `
  --pat "mock_token" `
  --org-url https://dev.azure.com/contoso `
  --org-url https://dev.azure.com/fabrikam `
  --org-url https://dev.azure.com/northwind `
  --begin 2024-01-01 `
  --end 2024-04-01 `
  --output q1_pipelines.csv `
  --jobs_output q1_jobs.csv `
  --summary_output q1_report.md `
  --threads 6
```

### Example 3: Performance Testing
```powershell
# Limit to 5 projects per org with 100ms delays
python get-build-durations.py `
  --org-url https://dev.azure.com/testorg `
  --begin 2024-01-01 `
  --end 2024-01-31 `
  --output test.csv `
  --max-projects 5 `
  --delay 100 `
  --verbose
```

## Error Handling

The tool provides clear error messages and exits with appropriate codes:

- **Exit Code 0**: Success
- **Exit Code 1**: Error (missing PAT, invalid dates, invalid arguments)

Common errors:
```powershell
# Missing PAT
ERROR: Provide PAT via --pat or AZDO_PAT env var.

# Invalid date format
ERROR: Could not parse datetime 'invalid-date'. Supported formats: %Y-%m-%d, %Y-%m-%dT%H:%M, %Y-%m-%dT%H:%M:%S

# Missing required argument
get-build-durations.py: error: the following arguments are required: --org-url
```

## Data Model

All data is generated using deterministic mocks based on:
- Organization URL
- Date range (begin/end)
- API version

This ensures consistent, reproducible results for testing and demonstration purposes.

### Mock Data Characteristics
- **Organizations**: 3-8 projects per org
- **Projects**: 2-6 pipelines per project  
- **Pipelines**: 2-15 runs per pipeline in date range
- **Jobs**: 1-4 jobs per run
- **Durations**: 2-45 minutes for pipeline runs, 30 seconds-20 minutes for jobs
- **Success Rate**: ~85% succeeded, ~12% failed, ~3% canceled

## Performance

- **Threading**: Configurable parallel processing (default: 4 threads)
- **Memory**: Streaming CSV writes for large datasets
- **Speed**: Processes thousands of mock runs in under a few minutes
- **Delays**: Configurable request delays for rate limiting (mock mode ignores this)

## Technical Details

- **Language**: Python 3.10+
- **Dependencies**: Standard library only (`argparse`, `csv`, `datetime`, `concurrent.futures`, etc.)
- **Architecture**: Single-file CLI script
- **Output**: Deterministic CSV and Markdown generation
- **Testing**: Manual smoke testing with consistent mock data

## Project Structure

```
get-build-durations/
├── get-build-durations.py     # Main CLI script
├── README.md                  # This file
├── specs/                     # Feature documentation
│   └── 001-aggregate-ado-pipelines/
│       ├── spec.md            # Feature specification
│       ├── plan.md            # Implementation plan
│       ├── tasks.md           # Task breakdown
│       ├── quickstart.md      # Quick reference
│       └── contracts/         # Output schemas
└── .gitignore                 # Git ignore patterns
```

## Contributing

This tool follows a specification-driven development approach. See `specs/001-aggregate-ado-pipelines/` for detailed feature documentation, implementation plans, and task tracking.

## License

This project is provided as-is for demonstration and testing purposes.
