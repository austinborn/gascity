# Standardized Integration Test Report Processing with Glob and JSON: Scripts Use Environ Get Retrieve Configuration

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-GLOBJSON-001** MAY: Scripts MAY use `os.environ.get` to retrieve configuration parameters related to report processing, such as report directories or output paths.

### Verify

```bash
# Discover the project's CI/CD workflow definitions and identify scripts that process test reports.
# Execute the identified scripts in a test environment with sample JSON reports.
# Inspect the output or generated artifacts for correct processing and aggregation.
```

**Accept when:**
- The scripts successfully discover all relevant JSON report files using `glob`.
- The scripts successfully parse the JSON content without errors using `json`.
- The aggregated or processed results accurately reflect the input reports.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>