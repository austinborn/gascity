# Standardized Conformance Testing with Go's `testing` Package: Before Implementation Consumer Discover Project Dependency

These rules are ALWAYS ACTIVE for internal Go modules and their associated test suites.

### Rules

- **R-ADR-001** MUST: Before implementation, the consumer MUST discover the project's dependency manifest and lock file to resolve the exact locked version of any versioned library.
- **R-ADR-002** MUST: This ADR omits all tool names, file names, commands, package managers, and version numbers; the consumer MUST derive them from the project repository.
- **R-ADR-003** MUST: Before writing code that uses a versioned library, execute the LOCK-VERSION GROUNDING steps in order.
  - **R-ADR-003a** MUST: Find the dependency manifest in the repo.
  - **R-ADR-003b** MUST: Identify the build tool from the manifest.
  - **R-ADR-003c** MUST: Inspect the repository lock or resolution artifact to determine the exact resolved version.
  - **R-ADR-003d** MUST: Look up the official documentation, changelog, or public API reference for that exact version.
  - **R-ADR-003e** MUST: Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
  - **R-ADR-003f** MUST: For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.

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