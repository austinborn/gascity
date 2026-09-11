# Standardized Integration Test Report Processing with Glob and JSON: When Interacting Versioned Dependencies Consumers Discover

These rules are ALWAYS ACTIVE for Python scripts within automated workflows that process integration test reports, especially when interacting with versioned dependencies.

### Rules

- **R-GLOBJSON-001** MUST: When interacting with versioned dependencies, consumers MUST discover the ecosystem's lock file and resolve the exact locked version before implementation.
- **R-GLOBJSON-002** MUST: This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.
- **R-GLOBJSON-003** MUST: Before writing code that uses a versioned library, execute the following lock-version grounding steps in order:
    1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
    2. Identify the build tool from the manifest.
    3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
    4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
    5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
    6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.

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