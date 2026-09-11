# Go Concurrency Model: Select with Context for Cancellable Event Processing: When Consuming Channel Within Select Statement

These rules are ALWAYS ACTIVE for Go services implementing background event processing loops and goroutines that listen on channels for continuous operation.

### Rules

- **R-GOCONC-001** MUST: When consuming from a channel within a `select` statement, the `ok` idiom (`rec, ok := <-ch`) MUST be used to detect channel closure and handle it appropriately.

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