# ServerAbsent Flag on PartialListError for Runtime Lifecycle Proof: Other Runtime Providers K8s Subprocess Exec

Status: proposed
Date: 2025-01-30
Deciders: AI (signal conversion)

## Context

- The runtime layer exposes a PartialListError type that is returned when a provider cannot fully enumerate running sessions. Previously, this error did not distinguish between two semantically distinct failure modes: (1) the backing server is unreachable but may still be holding live sessions, and (2) the backing server provably does not exist in the current boot context.
- The tmux provider encounters ErrNoServer when the tmux server process has never been started since the last system boot. In this state, any session beads (persistent session references) that were created before the current boot cannot possibly have live runtimes attached to them.
- A deadlock condition existed in the session-reaping path: orphaned pre-reboot session beads would block creation of successor sessions via ErrSessionAliasExists, but the fail-safe logic treating an absent server as 'potentially holding sessions' prevented those beads from being reaped. This left the system in an unrecoverable state without manual intervention.
- A new internal/hostboot package was introduced to provide the host boot instant (darwin kern.boottime, linux /proc/stat btime), enabling the session-reaping path to compare session bead creation timestamps against the current boot time and prove that pre-boot beads cannot have live runtimes.
- The PartialListError.ServerAbsent field is read via direct type assertion rather than error unwrapping. This is a deliberate design choice to prevent silent promotion of absent-server semantics across multi-backend error chains, where an inner provider's ServerAbsent flag could be misinterpreted by an outer aggregating provider.

## Problem Statement

The runtime layer lacks a semantic distinction between 'server unreachable but may hold live sessions' and 'server provably nonexistent post-boot', causing orphaned pre-reboot session beads to deadlock the session alias namespace while being immune to safe reaping.

## Decision

1. SHOULD: Other runtime providers (e.g., k8s, subprocess, exec) SHOULD set ServerAbsent when their backing infrastructure is provably absent, provided they can establish proof of absence equivalent in certainty to the tmux ErrNoServer + boot-time comparison mechanism.

## Policy Block

- SHOULD Other runtime providers (e.g., k8s, subprocess, exec) SHOULD set ServerAbsent when their backing infrastructure is provably absent, provided they can establish proof of absence equivalent in certainty to the tmux ErrNoServer + boot-time comparison mechanism.

## Rationale

- The non-unwrapping design of ServerAbsent is essential for correctness in multi-backend aggregation scenarios. If an inner provider's ServerAbsent flag were silently promoted through error unwrapping, an outer aggregating provider could incorrectly conclude that all backends are absent when only one is, leading to unsafe mass-reaping of potentially live sessions.
- Tying the ServerAbsent flag to the host boot instant creates a cryptographically-weak but operationally sufficient proof: if the server has never started since boot and a session bead predates the boot, no live runtime can exist for that bead. This breaks the deadlock without relaxing fail-safe guarantees for genuinely uncertain cases.
- Restricting ServerAbsent to 'provably nonexistent' rather than 'unreachable' preserves the fail-safe default: when in doubt, the system assumes sessions may be live and does not reap them. Only when absence is proven does the system permit reaping.
- Extending the contract to other providers (k8s, subprocess, exec) via a SHOULD rule acknowledges that the pattern is generalizable but requires each provider to establish its own proof-of-absence mechanism before setting the flag, preventing cargo-culting of the flag without the underlying proof.

## Consequences

Positive:
- Breaks the pre-reboot session bead deadlock, allowing the session alias namespace to be reclaimed after a system reboot without manual intervention.
- Creates a clear semantic contract at the runtime boundary between 'uncertain absence' and 'proven absence', enabling differentiated reaping policies.
- The non-unwrapping design prevents a class of silent semantic promotion bugs in multi-backend error aggregation.
- The hostboot package provides a reusable, platform-abstracted boot-time primitive that can support future lifecycle-proof mechanisms on both darwin and linux.

Negative:
- Providers that set ServerAbsent incorrectly (e.g., on transient errors) could cause premature reaping of live sessions; the rule requires careful implementation discipline.
- The direct type assertion pattern is less idiomatic than standard Go error unwrapping and requires explicit documentation to prevent future contributors from 'fixing' it to use errors.As.
- Extending the pattern to other providers (k8s, subprocess, exec) requires each to implement a provider-specific proof-of-absence mechanism, which may be non-trivial or impossible for some backends.

## Alternatives

- Treat all PartialListError cases uniformly as 'server may be holding sessions' and require manual operator intervention to reap pre-reboot beads. (rejected)
  Rejected because: This preserves the deadlock condition indefinitely and requires operational toil after every system reboot, which is unacceptable for automated session management.
- Use error unwrapping (errors.As) to read ServerAbsent across error chains, allowing aggregating providers to inspect inner provider flags. (rejected)
  Rejected because: Silent promotion of ServerAbsent across multi-backend boundaries would allow a single absent inner provider to trigger reaping policies intended only for cases where the entire backend is provably absent, risking unsafe reaping of live sessions.
- Add a separate error type (e.g., ErrServerAbsent) distinct from PartialListError rather than adding a field to the existing type. (deferred)
  When valid: Could be reconsidered if the PartialListError type accumulates too many boolean flags and a richer error taxonomy becomes warranted, but the current single-field addition does not justify a new type.
- Restrict ServerAbsent exclusively to the tmux provider and not extend the contract to other providers. (rejected)
  Rejected because: The management question explicitly raises whether other providers should adopt the pattern. Restricting it to tmux would create an inconsistent runtime contract and miss the opportunity to resolve equivalent deadlocks in other providers.

## Risks

- A provider incorrectly sets ServerAbsent on a transient error (e.g., IPC timeout), causing premature reaping of live sessions.
  Mitigation: Rule rule-5 explicitly prohibits setting ServerAbsent on transient errors. Code review and provider-level tests SHOULD verify that ServerAbsent is only set when a positive proof-of-absence condition is met (e.g., ErrNoServer combined with boot-time comparison).
  Owner: Runtime provider implementors
- Future contributors unfamiliar with the non-unwrapping design 'fix' the ServerAbsent read to use errors.As, silently breaking the multi-backend safety guarantee.
  Mitigation: Add a prominent code comment at the PartialListError definition and at each call site explaining the deliberate non-unwrapping design and its safety rationale. Consider a linter rule or test that asserts ServerAbsent is never read via errors.As.
  Owner: Runtime package maintainers
- The hostboot package returns an incorrect boot time on non-standard linux configurations (e.g., containers where /proc/stat btime reflects host boot but session beads reflect container start), causing incorrect reaping decisions.
  Mitigation: Document the hostboot package's assumptions about the execution environment. For containerized runtimes, evaluate whether container start time rather than host boot time is the appropriate epoch for session bead validity.
  Owner: internal/hostboot maintainers
- The medium signal confidence (58/100) indicates the full scope of affected providers and call sites may not be captured in the evidence, leaving gaps in the implementation.
  Mitigation: Conduct a full audit of all PartialListError construction and consumption sites across all runtime providers before finalizing the implementation, using the files identified in the signal as starting points.
  Owner: Architecture review

## Implementation Notes

- The ServerAbsent field on PartialListError MUST be read via direct type assertion (var e *PartialListError; if errors.As(err, &e) is NOT the pattern; use a direct cast or type switch on the immediate error value).
- The tmux adapter (internal/runtime/tmux/adapter.go) serves as the reference implementation: set ServerAbsent = true only in the ErrNoServer branch of ListRunning, not in generic error branches.
- The session-reaping path in cmd/gc/session_beads.go MUST combine the ServerAbsent check with a hostboot.BootTime() comparison: a bead is safe to reap only if ServerAbsent is true AND the bead's creation timestamp predates the current boot instant.
- When extending ServerAbsent to other providers (k8s, subprocess, exec), each provider MUST document its specific proof-of-absence condition in a comment adjacent to where it sets the flag.
- The internal/hostboot package provides platform-specific implementations for darwin (kern.boottime sysctl) and linux (/proc/stat btime); any new platform support MUST be added to this package rather than inline in the reaping logic.
- Consider adding a TestServerAbsentNotUnwrappable test that constructs a wrapped error containing a PartialListError with ServerAbsent=true and asserts that errors.As does not surface the flag, documenting the intended behavior as an executable specification.

## References

- internal/runtime/provider_core.go
- internal/runtime/tmux/adapter.go
- internal/hostboot/hostboot.go
- internal/hostboot/hostboot_darwin.go
- internal/hostboot/hostboot_linux.go
- cmd/gc/session_beads.go
- commit:3a09f0d1d5f1c15fb836af3e96912f3160dc1e76
- commit:2e101dbaebc2863c395cfb634fb461a9ccdc4ead
- PR#5456
- symbols: PartialListError, ServerAbsent, bootTime