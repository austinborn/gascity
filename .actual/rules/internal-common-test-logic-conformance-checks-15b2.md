# Standardized Conformance Testing with Go's `testing` Package: Common Test Logic Conformance Checks Internal

These rules are ALWAYS ACTIVE for internal Go modules and their associated test suites.

### Rules

- **R-GO-TEST-001** MUST: Common test logic and conformance checks for internal module contracts MUST be encapsulated within `Run*Tests` functions (e.g., `RunStoreTests`, `RunProviderTests`).

### Verify

```bash
# Discover the project's build command for running tests.
# Example for Go projects:
BUILD_COMMAND="go test ./..." # Or specific module paths

# Execute the build command to run all unit and integration tests.
${BUILD_COMMAND}

# Inspect the test output for any failures or skipped conformance checks.
# A non-zero exit code from 'go test' indicates failures.
```

**Accept when:**
- All unit and integration tests pass without errors.
- No conformance checks are unexpectedly skipped or ignored.
- Test output indicates adherence to expected module behaviors.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>