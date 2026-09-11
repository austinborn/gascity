# Go Concurrency Model: Select with Context for Cancellable Event Processing: When Consuming Channel Within Select Statement

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- Go applications often require background processes to handle events asynchronously.
- Graceful shutdown and resource management are critical for long-running services.
- Blocking operations can lead to unresponsive services and resource exhaustion.
- The `context` package provides a standard mechanism for cancellation and timeouts across API boundaries.

## Problem Statement

Ensuring background goroutines can process events from channels reliably while also responding to cancellation signals for graceful termination.

## Decision

1. MUST: When consuming from a channel within a `select` statement, the `ok` idiom (`rec, ok := <-ch`) MUST be used to detect channel closure and handle it appropriately.

## Policy Block

- MUST When consuming from a channel within a `select` statement, the `ok` idiom (`rec, ok := <-ch`) MUST be used to detect channel closure and handle it appropriately.

In scope:
- Go services implementing background event processing loops.
- Goroutines that listen on channels for continuous operation.

Out of scope:
- Simple, short-lived goroutines that do not require explicit cancellation.
- Blocking operations that are not part of an event loop.

## Rationale

- The `select` statement with `context.Done()` is the idiomatic Go pattern for structured concurrency, allowing goroutines to be cancelled safely.
- Using channels for event passing promotes decoupling and asynchronous processing.
- Detecting channel closure (`ok` idiom) prevents panics and ensures proper resource cleanup.
- Consistent error logging aids in debugging and operational monitoring.

## Consequences

Positive:
- Improved service reliability and resilience through graceful shutdown.
- Reduced resource leaks by ensuring goroutines terminate cleanly.
- Clearer and more maintainable concurrent code following established Go patterns.
- Consistent error reporting for operational visibility.

Negative:
- Increased boilerplate code for each concurrent event loop.
- Requires careful management of `context.Context` propagation.

## Alternatives

- Using `sync.WaitGroup` and explicit channel closures without `context.Context`. (rejected)
  Rejected because: `context.Context` provides a more flexible and composable mechanism for cancellation and timeouts across multiple goroutines and API calls, which `sync.WaitGroup` alone does not directly address for cancellation.
  When valid: For very simple, isolated goroutines with no external cancellation requirements.

## Risks

- Incorrect `context.Context` propagation leading to goroutine leaks or missed cancellation signals.
  Mitigation: Implement static analysis checks for context usage and conduct thorough code reviews.
  Owner: Engineering team.
- Overlooking channel closure handling, leading to panics or unexpected behavior.
  Mitigation: Enforce the `ok` idiom for channel reads in `select` statements through code review and linting.
  Owner: Engineering team.

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Ensure that the `context.Context` is passed down appropriately to all functions that need to respect cancellation.
- Consider using a structured logger instead of `log.Printf` for more robust error reporting in production environments.

## Continuation Context


Verify commands:
- Inspect relevant Go source files for the presence of `select` statements with `case <-ctx.Done():`.
- Run unit and integration tests that simulate cancellation scenarios to ensure graceful shutdown.
- Review code for adherence to the `ok` idiom when reading from channels.

Accept when:
- All background goroutines processing channel events include `select { case <-ctx.Done(): ... }`.
- Channel reads within `select` statements correctly handle the `ok` return value.
- Application logs show graceful shutdown messages when cancellation is triggered.

## Enforcement

- Verified by: Code reviews, static analysis tools (e.g., `go vet`, `golint`), and automated tests.
- Violation handling: Code failing to adhere to these patterns will be rejected during code review or flagged by CI/CD pipelines.
- Exception process: Exceptions require explicit approval from a lead engineer, documented with a clear rationale for deviation.