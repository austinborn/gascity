# ServerAbsent Flag on PartialListError for Runtime Lifecycle Proof: Callers Not Unwrap Partiallisterror Read Serverabsent

These rules are ALWAYS ACTIVE for all code that reads the `ServerAbsent` field on `PartialListError` or constructs `PartialListError` instances, particularly in multi-backend aggregation scenarios and session-reaping paths.

### Rules

- **R-SERVERABSENT-001** MUST_NOT: Callers MUST NOT unwrap `PartialListError` to read the `ServerAbsent` flag across multi-backend boundaries; the flag MUST be read only via direct type assertion on the immediate error value returned by a single provider.
- **R-SERVERABSENT-002** MUST: The `ServerAbsent` field on `PartialListError` MUST be read via direct type assertion (e.g., `var e *PartialListError; if errors.As(err, &e)` is NOT the pattern; use a direct cast or type switch on the immediate error value).
- **R-SERVERABSENT-003** MUST: Providers setting `ServerAbsent = true` MUST only do so when a positive proof-of-absence condition is met (e.g., `ErrNoServer` combined with boot-time comparison), never on transient errors or generic failures.
- **R-SERVERABSENT-004** MUST: The session-reaping path MUST combine the `ServerAbsent` check with a `hostboot.BootTime()` comparison: a bead is safe to reap only if `ServerAbsent` is true AND the bead's creation timestamp predates the current boot instant.
- **R-SERVERABSENT-005** SHOULD: When extending `ServerAbsent` to providers beyond tmux (k8s, subprocess, exec), each provider SHOULD document its specific proof-of-absence condition in a comment adjacent to where it sets the flag.
- **R-SERVERABSENT-006** SHOULD: Code comments at the `PartialListError` definition and at each call site SHOULD explain the deliberate non-unwrapping design and its safety rationale to prevent future contributors from 'fixing' it to use `errors.As`.
- **R-SERVERABSENT-007** MAY: A linter rule or test MAY be added to assert that `ServerAbsent` is never read via `errors.As`, documenting the intended behavior as an executable specification.

### Verify

```bash
# Verify no errors.As calls are used to read ServerAbsent
grep -r "errors\.As.*ServerAbsent" --include="*.go" . && exit 1 || true

# Verify ServerAbsent is only set in provider-specific branches with proof-of-absence
grep -B5 "ServerAbsent.*=.*true" --include="*.go" -r internal/runtime/ | grep -E "(ErrNoServer|proof|boot)" || echo "Warning: verify proof-of-absence conditions manually"

# Verify session-reaping path combines ServerAbsent with hostboot comparison
grep -A10 "ServerAbsent" cmd/gc/session_beads.go | grep -E "(BootTime|bootTime)" || echo "Warning: verify hostboot integration in reaping path"

# Verify tmux adapter sets ServerAbsent only in ErrNoServer branch
grep -B3 -A1 "ServerAbsent.*=.*true" internal/runtime/tmux/adapter.go | grep -E "(ErrNoServer|NoServer)" || echo "Warning: verify tmux adapter implementation"

# Verify direct type assertion pattern is used
grep -r "\*PartialListError" --include="*.go" . | grep -v "errors\.As" | head -5
```

**Accept when:**
- No `errors.As` calls are found reading the `ServerAbsent` field across the codebase.
- All `ServerAbsent = true` assignments are guarded by proof-of-absence conditions (e.g., `ErrNoServer` in tmux provider).
- The session-reaping path in `cmd/gc/session_beads.go` combines `ServerAbsent` checks with `hostboot.BootTime()` comparisons.
- The tmux adapter reference implementation sets `ServerAbsent` only in the `ErrNoServer` branch of `ListRunning`.
- Code comments at `PartialListError` definition and call sites document the non-unwrapping design rationale.
- All providers extending `ServerAbsent` document their specific proof-of-absence conditions.

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-SERVERABSENT rules MUST be checked before approving changes to error handling in the runtime layer, multi-backend aggregation, or session-reaping paths.
</enforcement>
