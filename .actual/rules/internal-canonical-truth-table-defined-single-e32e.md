# Blocked Work-Outcome Added to Dependency-Readiness Semantics: Canonical Truth Table Defined Single Shared

These rules are ALWAYS ACTIVE for all beads store implementations and dependency-readiness evaluation code across the multi-backend work-orchestration layer.

### Rules

- **R-BEADS-001** MUST: Define the canonical blocked-outcome truth table in a single shared location (`IsBlockedByWorkOutcome` / `filterReadyByWorkOutcome` in `internal/beads/beads.go`) rather than re-implementing per backend.
- **R-BEADS-002** MUST: Treat `gc.work_outcome=blocked` as an active orchestration gate that prevents downstream work items from becoming ready, not as pure metadata.
- **R-BEADS-003** MUST: Ensure all store implementations (memstore, SQLite, native Dolt, caching, BdStore, exec) delegate to the shared truth-table functions instead of re-implementing blocked-outcome checks.
- **R-BEADS-004** MUST: Apply the blocked-outcome veto as a post-filter on the final candidate set in stores that integrate with external readiness sources (e.g., BdStore), ensuring the beads layer is the authoritative enforcer of its own data contract.
- **R-BEADS-005** MUST: Include all store implementations in the conformance test suite (`internal/beads/beadstest/conformance.go`) and validate the canonical truth table across every registered backend.
- **R-BEADS-006** SHOULD: Audit active workflows for dependencies with `gc.work_outcome=blocked` before deploying this change to identify workflows that may be affected by the behavioral change.
- **R-BEADS-007** SHOULD: Document BdStore's layered veto pattern (applying blocked semantics atop native bd-ready candidates) as the reference implementation for stores integrating with external readiness sources.
- **R-BEADS-008** MUST: Enforce that any new `gc.work_outcome` value intended to gate readiness must be added to the shared truth table and conformance suite as part of the same change that introduces the value.
- **R-BEADS-009** MUST: Require that registering a new store implementation in the beads registry automatically includes it in the conformance test run.
- **R-BEADS-010** MUST: Keep the conformance suite in sync with the canonical truth table; when the truth table is amended, the conformance suite must be updated in the same commit.
- **R-BEADS-011** SHOULD: Track the management question—whether other `gc.work_outcome` values (e.g., `failed`, `cancelled`) should gate dependency resolution—as a follow-on work item with full conformance coverage.

### Verify

```bash
# Verify canonical truth table is defined in shared location
grep -r "IsBlockedByWorkOutcome\|filterReadyByWorkOutcome" internal/beads/beads.go

# Verify all store implementations delegate to shared functions
for store in memstore.go sqlite_store.go native_dolt_store.go caching_store_handles.go caching_store_reads.go bdstore.go; do
  echo "Checking $store delegates to shared truth table..."
  grep -l "IsBlockedByWorkOutcome\|filterReadyByWorkOutcome" "internal/beads/$store" || echo "WARNING: $store may not delegate"
done

# Verify conformance suite includes all registered stores
grep -c "conformance" internal/beads/beadstest/conformance.go

# Verify blocked-outcome veto is applied as post-filter in BdStore
grep -A 5 "blocked.*veto\|filterReadyByWorkOutcome" internal/beads/bdstore.go

# Verify no per-backend re-implementation of blocked-outcome logic
grep -r "work_outcome.*blocked" internal/beads/*.go | grep -v "beads.go" | grep -v "test" | wc -l
```

**Accept when:**
- `IsBlockedByWorkOutcome` and `filterReadyByWorkOutcome` are defined exactly once in `internal/beads/beads.go`
- All store implementations (memstore, SQLite, native Dolt, caching, BdStore) call the shared truth-table functions
- The conformance suite in `beadstest/conformance.go` validates the truth table across all registered stores
- BdStore applies the blocked-outcome veto as a documented post-filter pattern
- No per-backend re-implementation of blocked-outcome logic exists outside the shared functions
- Code review enforces that new `gc.work_outcome` values are added to the truth table and conformance suite in the same commit
- New store registrations automatically include conformance test coverage

<enforcement>
Claude Code MUST NOT skip or defer verification. All store implementations MUST be audited to confirm delegation to shared truth-table functions. Conformance test coverage MUST be validated across all backends before any change is merged.
</enforcement>