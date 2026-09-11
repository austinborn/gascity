# Adoption of Core Go Standard Libraries: Prefer Net Url Parsing Construction Manipulation

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-NETURL-001** MUST: MUST prefer `net/url` for all URL parsing, construction, and manipulation operations.

### Verify

```bash
# Run the project's test suite to ensure all existing functionalities relying on standard libraries continue to work as expected.
# (Specific command to be derived from project repository, e.g., 'go test ./...')

# Execute the project's linter and static analysis tools to identify any non-standard library imports for common utilities.
# (Specific command to be derived from project repository, e.g., 'golangci-lint run')

# Inspect the dependency manifest to confirm minimal external dependencies.
# (Specific command or manual inspection process to be derived from project repository, e.g., 'go mod graph | grep -v "std" | wc -l')
```

**Accept when:**
- All existing tests pass without errors.
- Linter and static analysis tools report no violations related to prohibited third-party utility libraries.
- New code adheres to the preference for Go standard libraries for common tasks.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>