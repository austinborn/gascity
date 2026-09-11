# Adoption of Go Standard Library `sort` Package: Before Implementing Any Sorting Logic Exact

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-GO-SORT-001** MUST: Before implementing any sorting logic, the exact version of the Go standard library used by the project's build system MUST be resolved.

### Verify

```bash
# Inspect relevant Go source files for `import "sort"` statements to confirm usage.
# Execute project tests that involve data ordering to confirm sorting logic behaves as expected.
# Review build configurations and dependency manifests to identify the Go version used and its standard library.
```

**Accept when:**
- The `sort` package is consistently imported and used for ordering data structures across the codebase.
- No custom, non-standard sorting implementations are found without clear, documented architectural justification.
- All relevant unit and integration tests involving sorted data pass successfully.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>