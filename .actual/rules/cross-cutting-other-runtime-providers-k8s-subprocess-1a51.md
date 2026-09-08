# ServerAbsent Flag on PartialListError for Runtime Lifecycle Proof: Other Runtime Providers K8s Subprocess Exec

These rules are ALWAYS ACTIVE for all runtime provider implementations (k8s, subprocess, exec) that return PartialListError from their ListRunning or equivalent enumeration methods, and for all session-reaping logic that consumes PartialListError.

### Rules

- **R-SERVERABSENT-001** MUST: Other runtime providers (k8s, subprocess, exec) MUST set PartialListError.ServerAbsent = true only when their backing infrastructure is provably absent, not on transient errors or unreachable conditions.
- **R-SERVERABSENT-002** MUST: The proof-of-absence condition for each provider MUST be equivalent in certainty to the tmux ErrNoServer + boot-time comparison mechanism (i.e., a positive assertion that the backing service cannot possibly hold live sessions post-boot).
- **R-SERVERABSENT-003** MUST: PartialListError.ServerAbsent MUST be read via direct type assertion or type switch on the immediate error value, never via errors.As() or error unwrapping across error chains.
- **R-SERVERABSENT-004** MUST: Session-reaping logic MUST combine the ServerAbsent check with a hostboot.BootTime() comparison: a session bead is safe to reap only if ServerAbsent is true AND the bead's creation timestamp predates the current boot instant.
- **R-SERVERABSENT-005** MUST: Each provider that sets ServerAbsent MUST document its specific proof-of-absence condition in a code comment adjacent to where the flag is set, explaining the certainty mechanism used.
- **R-SERVERABSENT-006** SHOULD: Providers SHOULD use the internal/hostboot package for platform-abstracted boot-time primitives rather than implementing provider-specific boot-time logic.
- **R-SERVERABSENT-007** MAY: Future providers on new platforms MAY extend the internal/hostboot package with platform-specific implementations rather than inline boot-time logic in the reaping path.

### Verify

```bash
# Verify ServerAbsent is only set in proven-absence branches
grep -n "ServerAbsent.*=.*true" internal/runtime/*/adapter.go | grep -v "ErrNoServer\|provably absent\|proof of absence" && echo "FAIL: ServerAbsent set without proof-of-absence comment" || echo "PASS: ServerAbsent only in documented absence branches"

# Verify ServerAbsent is never read via errors.As
grep -rn "errors\.As.*ServerAbsent\|errors\.As.*PartialListError" . --include="*.go" && echo "FAIL: ServerAbsent read via errors.As" || echo "PASS: No errors.As usage for ServerAbsent"

# Verify session-reaping combines ServerAbsent with boot-time check
grep -A5 "ServerAbsent" cmd/gc/session_beads.go | grep -q "BootTime\|bootTime" && echo "PASS: Reaping logic combines ServerAbsent with boot-time" || echo "FAIL: Reaping logic missing boot-time check"

# Verify PartialListError definition has ServerAbsent field
grep -A10 "type PartialListError" internal/runtime/provider_core.go | grep -q "ServerAbsent" && echo "PASS: PartialListError has ServerAbsent field" || echo "FAIL: PartialListError missing ServerAbsent field"

# Verify hostboot package exists and is used
test -f internal/hostboot/hostboot.go && test -f internal/hostboot/hostboot_darwin.go && test -f internal/hostboot/hostboot_linux.go && echo "PASS: hostboot package complete" || echo "FAIL: hostboot package incomplete"

# Verify tmux adapter serves as reference implementation
grep -B2 -A2 "ServerAbsent.*true" internal/runtime/tmux/adapter.go | grep -q "ErrNoServer" && echo "PASS: tmux adapter reference implementation correct" || echo "FAIL: tmux adapter missing ErrNoServer branch"
```

**Accept when:**
- ServerAbsent is only set in branches with documented proof-of-absence conditions (ErrNoServer, service-not-started, etc.)
- No code path reads ServerAbsent via errors.As() or error unwrapping
- Session-reaping logic combines ServerAbsent flag with hostboot.BootTime() comparison before reaping beads
- PartialListError type includes the ServerAbsent boolean field
- internal/hostboot package provides platform-specific boot-time implementations for darwin and linux
- tmux adapter implementation sets ServerAbsent only in the ErrNoServer branch
- All other providers (k8s, subprocess, exec) that set ServerAbsent include adjacent code comments explaining their proof-of-absence mechanism
- No transient error branches set ServerAbsent to true

<enforcement>
Claude Code MUST NOT skip or defer verification. All six bash verification commands MUST pass before accepting any implementation of ServerAbsent in other runtime providers. The non-unwrapping design (R-SERVERABSENT-003) is a critical safety invariant and MUST be enforced via code review and automated checks.
</enforcement>