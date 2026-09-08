# ServerAbsent Flag on PartialListError for Runtime Lifecycle Proof: Session Reaping Path Use Host Boot

These rules are ALWAYS ACTIVE for all runtime provider implementations, session-reaping logic, and error handling code that constructs or consumes PartialListError types across the runtime layer.

### Rules

- **R-SRVABS-001** MUST: The session-reaping path MUST use the host boot instant (from internal/hostboot) in conjunction with ServerAbsent to prove that session beads created before the current boot cannot have live runtimes, and MUST reap such beads when ServerAbsent is true.
- **R-SRVABS-002** MUST: The ServerAbsent field on PartialListError MUST be read via direct type assertion (type switch or direct cast on the immediate error value), NOT via errors.As or error unwrapping patterns.
- **R-SRVABS-003** MUST: ServerAbsent MUST only be set to true when a positive proof-of-absence condition is met (e.g., ErrNoServer combined with boot-time comparison), never on transient errors, timeouts, or generic IPC failures.
- **R-SRVABS-004** MUST: The tmux adapter (internal/runtime/tmux/adapter.go) MUST set ServerAbsent = true only in the ErrNoServer branch of ListRunning, not in generic error branches.
- **R-SRVABS-005** MUST: A session bead is safe to reap only if ServerAbsent is true AND the bead's creation timestamp predates the current boot instant (from hostboot.BootTime()).
- **R-SRVABS-006** SHOULD: When extending ServerAbsent to other providers (k8s, subprocess, exec), each provider SHOULD document its specific proof-of-absence condition in a comment adjacent to where it sets the flag.
- **R-SRVABS-007** SHOULD: Code review and provider-level tests SHOULD verify that ServerAbsent is only set when a positive proof-of-absence condition is met, not on transient errors.
- **R-SRVABS-008** MAY: New platform support for hostboot MAY be added to the internal/hostboot package rather than inline in the reaping logic.

### Verify

```bash
# Verify ServerAbsent is never read via errors.As
grep -r "errors\.As.*ServerAbsent" internal/runtime/ cmd/gc/ && echo "FAIL: ServerAbsent read via errors.As" || echo "PASS: No errors.As usage for ServerAbsent"

# Verify ServerAbsent is only set in ErrNoServer branch for tmux
grep -A5 -B5 "ServerAbsent.*=.*true" internal/runtime/tmux/adapter.go | grep -q "ErrNoServer" && echo "PASS: ServerAbsent set in ErrNoServer context" || echo "FAIL: ServerAbsent set outside ErrNoServer"

# Verify session-reaping path uses hostboot comparison
grep -q "hostboot\.BootTime" cmd/gc/session_beads.go && echo "PASS: hostboot.BootTime used in reaping logic" || echo "FAIL: hostboot.BootTime not found in reaping logic"

# Verify PartialListError definition has ServerAbsent field
grep -q "ServerAbsent.*bool" internal/runtime/provider_core.go && echo "PASS: ServerAbsent field exists" || echo "FAIL: ServerAbsent field not found"

# Verify direct type assertion pattern is used
grep -q "var.*\*PartialListError" cmd/gc/session_beads.go && echo "PASS: Direct type assertion pattern detected" || echo "FAIL: Direct type assertion pattern not found"
```

**Accept when:**
- ServerAbsent is never read via errors.As or error unwrapping in any runtime provider or reaping code
- ServerAbsent is only set to true in the ErrNoServer branch of the tmux adapter's ListRunning method
- The session-reaping path in cmd/gc/session_beads.go combines ServerAbsent checks with hostboot.BootTime() comparisons
- Session beads are only reaped when both ServerAbsent is true AND the bead's creation timestamp predates the current boot instant
- All direct type assertions on PartialListError are used instead of errors.As patterns
- Code comments at PartialListError definition and call sites explain the deliberate non-unwrapping design and its safety rationale
- Each provider extending ServerAbsent documents its specific proof-of-absence condition

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-SRVABS rules MUST be verified before accepting any changes to runtime provider implementations, session-reaping logic, or PartialListError handling. The non-unwrapping design is a critical safety invariant that prevents silent semantic promotion bugs in multi-backend error aggregation.
</enforcement>