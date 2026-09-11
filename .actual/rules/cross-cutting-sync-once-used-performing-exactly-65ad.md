# Concurrency Management with Go's `sync` Package: Sync Once Used Performing Exactly One

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-SYNC-001** MAY: `sync.Once` MAY be used for performing exactly one action, such as singleton initialization.

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