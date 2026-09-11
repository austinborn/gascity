# Go Concurrency Model: Select with Context for Cancellable Event Processing: Errors Encountered During Event Processing Within

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-GCM-001** SHOULD: Errors encountered during event processing within a concurrent routine SHOULD be logged using the standard logging mechanism.

### Verify

```bash
# Inspect relevant Go source files for the presence of `select` statements with `case <-ctx.Done():`.
# Run unit and integration tests that simulate cancellation scenarios to ensure graceful shutdown.
# Review code for adherence to the `ok` idiom when reading from channels.
```

**Accept when:**
- All background goroutines processing channel events include `select { case <-ctx.Done(): ... }`.
- Channel reads within `select` statements correctly handle the `ok` return value.
- Application logs show graceful shutdown messages when cancellation is triggered.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>