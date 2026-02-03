# Testing the Azure DevOps Pipeline Aggregation Tool

This directory contains mock data and example outputs for testing the Azure DevOps Pipeline Aggregation Tool without connecting to live Azure DevOps APIs.

## Overview

The `get-build-durations.py` script supports a **mock mode** (enabled with the `--mock` flag) that generates deterministic test data based on input parameters. This allows for:

- Testing script functionality without Azure DevOps access
- Validating output formats and schemas
- Demonstrating tool capabilities
- Reproducible test runs

## Test Files

| File | Description |
|------|-------------|
| `test_output.csv` | Single organization pipeline aggregation output |
| `test_output.summary.md` | Summary report for single organization |
| `test_output2.csv` | Alternative single organization output |
| `test_output2.summary.md` | Summary report for alternative output |
| `test_multi.csv` | Multi-organization pipeline aggregation output |
| `test_multi.summary.md` | Summary report for multi-organization run |
| `test_jobs.csv` | Job-level detail output |
| `livepipelines.summary.md` | Example summary from a live API run against real Azure DevOps organizations |

## Running Tests with Mock Data

### Prerequisites

- Python 3.10 or higher
- No additional dependencies required (mock mode uses Python standard library only)

### Basic Mock Test

Run the script with the `--mock` flag to use deterministic mock data:

```powershell
# Set a placeholder PAT (any non-empty value works in mock mode)
$env:AZDO_PAT = "mock_token"

# Generate mock pipeline data for a single organization
python get-build-durations.py `
  --mock `
  --org-url https://dev.azure.com/testorg `
  --begin 2024-01-01 `
  --end 2024-01-31 `
  --output test/my_test_output.csv `
  --verbose
```

### Multi-Organization Mock Test

In mock mode, any non-empty PAT value works. You can use a single global PAT or test the org-specific PAT resolution:

```powershell
# Option 1: Single global PAT (simplest for mock testing)
$env:AZDO_PAT = "mock_token"
python get-build-durations.py `
  --mock `
  --org-url https://dev.azure.com/org1 `
  --org-url https://dev.azure.com/org2 `
  --begin 2024-01-01 `
  --end 2024-02-01 `
  --output test/multi_org_output.csv `
  --jobs_output test/multi_org_jobs.csv `
  --summary_output test/multi_org_summary.md `
  --verbose
```

```powershell
# Option 2: Test org-specific PAT resolution (simulates production setup)
$env:AZDO_PAT_ORG1 = "mock_token_org1"
$env:AZDO_PAT_ORG2 = "mock_token_org2"
python get-build-durations.py `
  --mock `
  --org-url https://dev.azure.com/org1 `
  --org-url https://dev.azure.com/org2 `
  --begin 2024-01-01 `
  --end 2024-02-01 `
  --output test/multi_org_output.csv `
  --verbose
```

### Testing with Limited Projects

Use `--max-projects` to limit the scope for faster test runs:

```powershell
python get-build-durations.py `
  --mock `
  --org-url https://dev.azure.com/testorg `
  --begin 2024-01-01 `
  --end 2024-01-31 `
  --output test/limited_output.csv `
  --max-projects 3 `
  --verbose
```

### Testing Job Details Output

```powershell
python get-build-durations.py `
  --mock `
  --org-url https://dev.azure.com/testorg `
  --begin 2024-01-01 `
  --end 2024-01-31 `
  --output test/pipelines.csv `
  --jobs_output test/jobs.csv `
  --verbose
```

## Mock Data Characteristics

The mock data generator produces consistent, deterministic results based on:
- Organization URL
- Date range (begin/end)
- API version

### Expected Data Volumes

| Entity | Range |
|--------|-------|
| Projects per organization | 3-8 |
| Pipelines per project | 2-6 |
| Runs per pipeline | 2-15 |
| Jobs per run | 1-4 |

### Duration Ranges

| Metric | Range |
|--------|-------|
| Pipeline run duration | 2-45 minutes |
| Job duration | 30 seconds - 20 minutes |

### Status Distribution

| Status | Approximate Rate |
|--------|-----------------|
| Succeeded | ~85% |
| Failed | ~12% |
| Canceled | ~3% |

## Validating Output

### Compare CSV Structure

Verify output matches expected schema:

```powershell
# Check header row matches expected columns
Get-Content test/test_output.csv -Head 1
# Expected: org_url,project_id,project_name,pipeline_id,pipeline_name,run_count,avg_duration_seconds,total_duration_seconds
```

### Verify Summary Generation

Confirm summary markdown is generated alongside CSV:

```powershell
# If --output is specified, summary should be auto-generated
python get-build-durations.py `
  --mock `
  --org-url https://dev.azure.com/testorg `
  --begin 2024-01-01 `
  --end 2024-01-31 `
  --output test/verify.csv

# Check that verify.summary.md was created
Test-Path test/verify.summary.md
```

## Reproducibility

Mock mode produces **identical output** for the same input parameters. This means:

1. Running the same command twice produces identical CSV files
2. Test assertions can rely on exact values
3. Results can be committed to version control for comparison

Example reproducibility test:

```powershell
# Run twice with same parameters
python get-build-durations.py --mock --org-url https://dev.azure.com/testorg --begin 2024-01-01 --end 2024-01-31 --output test/run1.csv
python get-build-durations.py --mock --org-url https://dev.azure.com/testorg --begin 2024-01-01 --end 2024-01-31 --output test/run2.csv

# Compare outputs (should be identical)
Compare-Object (Get-Content test/run1.csv) (Get-Content test/run2.csv)
```

## Output Schemas

For detailed output format specifications, see:
- `specs/001-aggregate-ado-pipelines/contracts/pipeline_aggregate.csv.schema.yaml`
- `specs/001-aggregate-ado-pipelines/contracts/job_details.csv.schema.yaml`
- `specs/001-aggregate-ado-pipelines/contracts/summary.md.schema.yaml`
