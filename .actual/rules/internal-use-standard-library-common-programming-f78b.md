# Adoption of Core Go Standard Libraries: Use Standard Library Common Programming Tasks

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-GO-STD-001** MUST: Use the Go standard library for common programming tasks such as URL parsing, JSON encoding/decoding, string manipulation, and error handling.

### Verify

```bash
# Run the project's test suite to ensure all existing functionalities relying on standard libraries continue to work as expected.
# Execute the project's linter and static analysis tools to identify any non-standard library imports for common utilities.
# Inspect the dependency manifest to confirm minimal external dependencies.
# (Specific commands for these actions must be derived from the project repository as per the ADR's DISCOVERY POLICY.)
```

**Accept when:**
- All existing tests pass without errors.
- Linter and static analysis tools report no violations related to prohibited third-party utility libraries.
- New code adheres to the preference for Go standard libraries for common tasks.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>