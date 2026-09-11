# Adoption of Go Standard Library `sort` Package: Sort Package Standard Library Used Ordering

These rules are ALWAYS ACTIVE for any Go module or package requiring ordered data structures.

### Rules

- **R-SORT-001** MUST: The `sort` package from the Go standard library MUST be used for ordering collections of data.

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