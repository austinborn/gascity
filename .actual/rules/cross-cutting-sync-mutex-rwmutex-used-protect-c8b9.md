# Concurrency Management with Go's `sync` Package: Sync Mutex Rwmutex Used Protect Critical

These rules are ALWAYS ACTIVE for Go modules and packages that involve concurrent operations or shared mutable state.

### Rules

- **R-SYNC-001** SHOULD: `sync.Mutex` or `sync.RWMutex` SHOULD be used to protect critical sections of code that modify shared data.

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