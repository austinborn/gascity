# Go Concurrency Model: Select with Context for Cancellable Event Processing: Concurrent Routines Processing Events Channel Utilize

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-GC-001** MUST: Concurrent routines processing events from a channel MUST utilize a `select` statement that includes a `case <-ctx.Done():` to enable graceful cancellation via `context.Context`.

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