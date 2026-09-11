# Concurrency Management with Go's `sync` Package: Shared Mutable State Accessed Multiple Goroutines

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-CONCURRENCY-001** MUST: All shared mutable state accessed by multiple goroutines MUST be protected using synchronization primitives from the `sync` package.

### Verify

```bash
# Discover and run the project's static analysis tools for concurrency issues.
# Example: go vet ./...
# Example: golangci-lint run --enable=gosec,staticcheck,ineffassign ./...

# Discover and run the project's unit and integration tests that involve concurrent execution.
# Example: go test -run ConcurrencyTests ./...

# Discover and run the project's race detector during testing.
go test -race ./...
```

**Accept when:**
- Static analysis reports no new concurrency warnings related to `sync` package usage.
- All concurrency-related tests pass without failures or race conditions.
- Performance benchmarks for concurrent operations meet established thresholds.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>