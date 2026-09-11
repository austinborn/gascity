# Adoption of Core Go Standard Libraries: Discover Project Dependency Management Configuration Resolve

These rules are ALWAYS ACTIVE for all Go source files within the project and any new Go module or package developed within the project.

### Rules

- **R-GO-001** MUST: MUST discover the project's dependency management configuration and resolve the exact locked version of any Go module before implementing code that relies on its behavior.

### Verify

```bash
# Run the project's test suite to ensure all existing functionalities relying on standard libraries continue to work as expected.
# Execute the project's linter and static analysis tools to identify any non-standard library imports for common utilities.
# Inspect the dependency manifest to confirm minimal external dependencies.
```

**Accept when:**
- All existing tests pass without errors.
- Linter and static analysis tools report no violations related to prohibited third-party utility libraries.
- New code adheres to the preference for Go standard libraries for common tasks.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>