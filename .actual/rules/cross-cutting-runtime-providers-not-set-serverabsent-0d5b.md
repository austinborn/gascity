# ServerAbsent Flag on PartialListError for Runtime Lifecycle Proof: Runtime Providers Not Set Serverabsent Based

These rules are ALWAYS ACTIVE for all runtime provider implementations that construct or consume PartialListError, particularly the tmux adapter, session-reaping logic, and any future providers (k8s, subprocess, exec) that adopt the ServerAbsent pattern.

### Rules

- **R-RUNTIME-001** MUST NOT: Runtime providers MUST NOT set ServerAbsent based solely on a connection timeout or transient network/IPC error; the flag MUST only be set when absence can be positively proven (e.g., ErrNoServer combined with boot-time comparison for tmux).
- **R-RUNTIME-002** MUST: The ServerAbsent field on PartialListError MUST be read via direct type assertion (type switch or direct cast on the immediate error value), NOT via errors.As() unwrapping, to prevent silent promotion of absent-server semantics across multi-backend error chains.
- **R-RUNTIME-003** MUST: The session-reaping path MUST combine the ServerAbsent check with a hostboot.BootTime() comparison: a bead is safe to reap only if ServerAbsent is true AND the bead's creation timestamp predates the current boot instant.
- **R-RUNTIME-004** SHOULD: When extending ServerAbsent to other providers (k8s, subprocess, exec), each provider SHOULD document its specific proof-of-absence condition in a comment adjacent to where it sets the flag.
- **R-RUNTIME-005** SHOULD: Code review and provider-level tests SHOULD verify that ServerAbsent is only set when a positive proof-of-absence condition is met, never on transient errors or connection timeouts.
- **R-RUNTIME-006** SHOULD: Add a prominent code comment at the PartialListError definition and at each call site explaining the deliberate non-unwrapping design and its safety rationale to prevent future contributors from 'fixing' it to use errors.As.

### Verify

```bash
# Verify ServerAbsent is never set on transient errors in tmux adapter
grep -n "ServerAbsent.*true" internal/runtime/tmux/adapter.go | grep -v "ErrNoServer" && echo "FAIL: ServerAbsent set outside ErrNoServer branch" || echo "PASS: ServerAbsent only in ErrNoServer"

# Verify ServerAbsent is read via type assertion, not errors.As
grep -n "errors\.As.*ServerAbsent" internal/runtime/provider_core.go cmd/gc/session_beads.go && echo "FAIL: ServerAbsent read via errors.As" || echo "PASS: No errors.As unwrapping of ServerAbsent"

# Verify session-reaping combines ServerAbsent with boot-time check
grep -A5 "ServerAbsent" cmd/gc/session_beads.go | grep -q "BootTime\|bootTime" && echo "PASS: ServerAbsent combined with boot-time check" || echo "FAIL: Missing boot-time comparison"

# Verify hostboot package exists and provides platform implementations
test -f internal/hostboot/hostboot.go && test -f internal/hostboot/hostboot_darwin.go && test -f internal/hostboot/hostboot_linux.go && echo "PASS: hostboot package complete" || echo "FAIL: hostboot package incomplete"

# Verify PartialListError definition includes ServerAbsent field
grep -q "type PartialListError struct" internal/runtime/provider_core.go && grep -A10 "type PartialListError struct" internal/runtime/provider_core.go | grep -q "ServerAbsent" && echo "PASS: PartialListError has ServerAbsent field" || echo "FAIL: PartialListError missing ServerAbsent"

# Verify no direct errors.As pattern is used to read ServerAbsent
grep -rn "errors\.As.*&.*PartialListError" . --include="*.go" | grep -v test && echo "FAIL: Direct errors.As unwrapping found" || echo "PASS: No direct errors.As unwrapping"
```

**Accept when:**
- ServerAbsent is only set in the tmux adapter when ErrNoServer is encountered, never on transient connection errors
- All reads of ServerAbsent use direct type assertion or type switch, never errors.As() unwrapping
- Session-reaping logic verifies both ServerAbsent=true AND bead creation timestamp < current boot time before reaping
- hostboot package provides platform-specific implementations for darwin and linux
- PartialListError type includes the ServerAbsent boolean field
- Code comments explain the non-unwrapping design rationale at PartialListError definition and call sites
- All provider implementations that set ServerAbsent document their proof-of-absence condition

<enforcement>
Claude Code MUST NOT skip or defer verification. All six bash verification commands MUST pass before accepting changes to runtime provider error handling or session-reaping logic.
</enforcement>