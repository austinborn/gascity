# Adoption of Go Standard Library `sort` Package: Developers Prefer Most Idiomatic Sort Package

These rules are ALWAYS ACTIVE for all Go modules and packages in this project.

### Rules

- **R-GO-SORT-001** SHOULD: Developers SHOULD prefer the most idiomatic `sort` package functions (e.g., `sort.Strings`, `sort.Ints`, `sort.Slice`) appropriate for the data type being sorted.

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