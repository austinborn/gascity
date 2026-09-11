# Standardized Conformance Testing with Go's `testing` Package: Tests Structured Run Clear Subtest Organization

These rules are ALWAYS ACTIVE for all internal Go modules and their associated test suites.

### Rules

- **R-TEST-001** MUST: Tests MUST be structured using `t.Run` for clear subtest organization.

### Verify

```bash
# Discover the project's build command for running tests.
# Execute the build command to run all unit and integration tests.
# Inspect the test output for any failures or skipped conformance checks.
```

**Accept when:**
- All unit and integration tests pass without errors.
- No conformance checks are unexpectedly skipped or ignored.
- Test output indicates adherence to expected module behaviors.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>