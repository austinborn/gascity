# Standardized Conformance Testing with Go's `testing` Package: Unit Integration Tests Use Testing Package

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-TEST-001** MUST: All unit and integration tests MUST use the `testing` package.

### Verify

```bash
# Discover the project's build command for running tests.
# Example: go test ./... or make test
#
# Execute the build command to run all unit and integration tests.
#
# Inspect the test output for any failures or skipped conformance checks.
#
# The specific command to run tests must be derived from the project repository.
# For example, if using Go modules:
# go test ./...
#
# If using a Makefile:
# make test
```

**Accept when:**
- All unit and integration tests pass without errors.
- No conformance checks are unexpectedly skipped or ignored.
- Test output indicates adherence to expected module behaviors.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>