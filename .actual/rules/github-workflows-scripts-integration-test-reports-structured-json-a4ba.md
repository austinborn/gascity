# Standardized Integration Test Report Processing with Glob and JSON: Integration Test Reports Structured Json Files

These rules are ALWAYS ACTIVE for Python scripts within automated workflows that process integration test reports.

### Rules

- **R-GLOBJSON-001** SHOULD: Integration test reports SHOULD be structured as JSON files for machine readability and consistent parsing.

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