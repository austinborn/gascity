# Standardized Conformance Testing with Go's `testing` Package: Test Failures Reported Fatalf Critical Errors

These rules are ALWAYS ACTIVE for internal Go modules and their associated test suites.

### Rules

- **R-TF-001** MUST: Test failures MUST be reported using `t.Fatalf` for critical errors and `t.Error`/`t.Errorf` for non-fatal issues within subtests.

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