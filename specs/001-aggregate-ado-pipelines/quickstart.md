# Quickstart: Azure DevOps Pipeline Aggregation (Mocked Data)

## Prerequisites
- Windows PowerShell 5.1
- Python 3.10+
- `AZDO_PAT` environment variable set, or provide `--pat`

## Basic Usage

```powershell
# Single organization, pipeline aggregate CSV to file + summary markdown
python .\get-build-durations.py --org-url https://dev.azure.com/myorg --begin 2024-01-01 --end 2024-01-31 --output C:\temp\pipelines.csv --verbose
# Summary will be written to C:\temp\pipelines.summary.md
```

```powershell
# Multiple organizations, job-level CSV as well
python .\get-build-durations.py --org-url https://dev.azure.com/org1 --org-url https://dev.azure.com/org2 --begin 2024-01-01T00:00:00 --end 2024-02-01T00:00:00 --output C:\temp\pipelines.csv --jobs_output C:\temp\jobs.csv --threads 8 --delay 50
```

```powershell
# When writing pipeline CSV to stdout, specify summary path explicitly
python .\get-build-durations.py --org-url https://dev.azure.com/org1 --begin 2024-01-01 --end 2024-01-31 > C:\temp\pipelines.csv
python .\get-build-durations.py --org-url https://dev.azure.com/org1 --begin 2024-01-01 --end 2024-01-31 --summary_output C:\temp\pipelines.summary.md
```

## Outputs
- `pipelines.csv` follows `contracts/pipeline_aggregate.csv.schema.yaml`
- `jobs.csv` follows `contracts/job_details.csv.schema.yaml`
- `pipelines.summary.md` follows `contracts/summary.md.schema.yaml`

## Notes
- All data is mocked and deterministic based on inputs; no live API calls.
- Errors print to stderr; success exits with code 0.
