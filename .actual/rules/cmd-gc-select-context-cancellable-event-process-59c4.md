# Go Concurrency Model: Select with Context for Cancellable Event Processing: Before Implementing Any Code That Relies

These rules are ALWAYS ACTIVE for Go services implementing background event processing loops and goroutines that listen on channels for continuous operation.

### Rules

- **R-GC-SC-001** MUST: Before implementing any code that relies on versioned dependencies, the consumer MUST discover the project's dependency manifest and lock file to determine the exact resolved version and consult its official documentation.
- **R-GC-SC-002** MUST: This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.
- **R-GC-SC-003** MUST: Before writing code that uses a versioned library, execute in order: 1. Find the dependency manifest in the repo. It declares ranges, not installed versions. 2. Identify the build tool from the manifest. 3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it. 4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs. 5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it. 6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- **R-GC-SC-004** MUST: Ensure that the `context.Context` is passed down appropriately to all functions that need to respect cancellation.
- **R-GC-SC-005** SHOULD: Consider using a structured logger instead of `log.Printf` for more robust error reporting in production environments.

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