# Concurrency Management with Go's `sync` Package: Before Implementing Concurrency Patterns Sync Package

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-SYNC-001** MUST: Before implementing concurrency patterns using the `sync` package, developers MUST identify the project's Go toolchain version and consult the official Go documentation for the exact behavior of `sync` primitives for that version.

### Verify

```bash
# Discover and run the project's static analysis tools for concurrency issues.
# Discover and run the project's unit and integration tests that involve concurrent execution.
# Discover and run the project's race detector during testing.
```

**Accept when:**
- Static analysis reports no new concurrency warnings related to `sync` package usage.
- All concurrency-related tests pass without failures or race conditions.
- Performance benchmarks for concurrent operations meet established thresholds.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>