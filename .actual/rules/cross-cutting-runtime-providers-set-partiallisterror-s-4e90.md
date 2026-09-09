# ServerAbsent Flag on PartialListError for Runtime Lifecycle Proof: Runtime Providers Set Partiallisterror Serverabsent True

These rules are ALWAYS ACTIVE for all runtime provider implementations that construct or consume PartialListError types, including internal/runtime/tmux/adapter.go, internal/runtime/provider_core.go, cmd/gc/session_beads.go, and any future providers (k8s, subprocess, exec) that implement the runtime.Provider interface.

### Rules

- **R-RUNTIME-001** MUST: Runtime providers MUST set PartialListError.ServerAbsent to true only when the backing infrastructure is provably nonexistent (e.g., the server process has never started since the current boot), not merely when it is unreachable or temporarily unavailable.

- **R-RUNTIME-002** MUST: The ServerAbsent field on PartialListError MUST be read via direct type assertion or type switch on the immediate error value, never via errors.As() or error unwrapping across error chains.

- **R-RUNTIME-003** MUST: When setting ServerAbsent = true in a provider, the provider MUST document its specific proof-of-absence condition in a code comment adjacent to where the flag is set.

- **R-RUNTIME-004** MUST: The session-reaping path MUST combine the ServerAbsent check with a hostboot.BootTime() comparison: a bead is safe to reap only if ServerAbsent is true AND the bead's creation timestamp predates the current boot instant.

- **R-RUNTIME-005** MUST: Providers MUST NOT set ServerAbsent on transient errors (e.g., IPC timeouts, temporary connection failures, or context cancellations).

- **R-RUNTIME-006** SHOULD: When extending ServerAbsent to providers beyond tmux (k8s, subprocess, exec), each provider SHOULD implement a provider-specific proof-of-absence mechanism before setting the flag, rather than cargo-culting the flag without underlying proof.

- **R-RUNTIME-007** SHOULD: Code review and provider-level tests SHOULD verify that ServerAbsent is only set when a positive proof-of-absence condition is met (e.g., ErrNoServer combined with boot-time comparison).

### Verify

```bash
# Verify ServerAbsent is never read via errors.As in the codebase
grep -r "errors\.As.*ServerAbsent" internal/runtime/ cmd/gc/ && echo "FAIL: ServerAbsent read via errors.As" || echo "PASS: No errors.As usage for ServerAbsent"

# Verify all ServerAbsent assignments have adjacent documentation comments
grep -B2 "ServerAbsent.*=.*true" internal/runtime/*/adapter.go cmd/gc/session_beads.go | grep -E "^\s*//" || echo "WARN: Check for missing documentation comments"

# Verify session reaping combines ServerAbsent with boot time check
grep -A5 "ServerAbsent" cmd/gc/session_beads.go | grep -q "BootTime\|bootTime" && echo "PASS: ServerAbsent combined with boot time check" || echo "FAIL: Missing boot time comparison"

# Verify tmux adapter only sets ServerAbsent in ErrNoServer branch
grep -B5 -A2 "ServerAbsent.*=.*true" internal/runtime/tmux/adapter.go | grep -q "ErrNoServer" && echo "PASS: ServerAbsent set in ErrNoServer context" || echo "WARN: Verify ServerAbsent context"

# Verify hostboot package exists and provides BootTime
test -f internal/hostboot/hostboot.go && grep -q "func BootTime" internal/hostboot/hostboot.go && echo "PASS: hostboot.BootTime available" || echo "FAIL: hostboot package incomplete"
```

**Accept when:**
- ServerAbsent is never read via errors.As() in any runtime provider or reaping path
- All ServerAbsent = true assignments have adjacent code comments explaining the proof-of-absence condition
- The session-reaping path in cmd/gc/session_beads.go combines ServerAbsent checks with hostboot.BootTime() comparisons
- The tmux adapter only sets ServerAbsent = true in the ErrNoServer branch, not in generic error handlers
- The internal/hostboot package provides platform-specific BootTime() implementations for darwin and linux
- No transient error conditions (timeouts, connection failures) set ServerAbsent = true
- Code review confirms each provider setting ServerAbsent has documented its specific proof-of-absence mechanism

<enforcement>
Claude Code MUST NOT skip or defer verification. All seven verify commands MUST pass before accepting a change that touches PartialListError construction, ServerAbsent assignment, or session-reaping logic. If any command fails, the change MUST be rejected with specific guidance on which rule was violated.
</enforcement>