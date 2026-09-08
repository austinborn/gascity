# Blocked Work-Outcome Added to Dependency-Readiness Semantics: Beads Store Implementation Not Treat Dependency

These rules are ALWAYS ACTIVE for all beads-store implementations and dependency-readiness evaluation code across the beads package, including memstore, caching, SQLite, native Dolt, exec, and BdStore backends.

### Rules

- **R-BEADS-001** MUST NOT: A beads-store implementation MUST NOT treat a dependency as ready-to-unlock downstream work when that dependency is closed and carries `gc.work_outcome=blocked`, regardless of any externally computed readiness signal (e.g., native bd-ready candidates).
- **R-BEADS-002** MUST: All store implementations MUST delegate blocked-outcome checks to the shared canonical functions `IsBlockedByWorkOutcome` and `filterReadyByWorkOutcome` in `internal/beads/beads.go` rather than re-implementing the check.
- **R-BEADS-003** MUST: BdStore and any store integrating with external readiness sources MUST apply the blocked-outcome veto as a post-filter over native readiness candidates, not as a concurrent signal.
- **R-BEADS-004** MUST: The conformance suite in `internal/beads/beadstest/conformance.go` MUST be kept in sync with the canonical truth table and updated in the same commit when the truth table is amended.
- **R-BEADS-005** MUST: Any new `gc.work_outcome` value intended to gate readiness MUST be added to the shared truth table and conformance suite as part of the same change that introduces the value.
- **R-BEADS-006** MUST: Registering a new store implementation in the beads registry MUST automatically include it in the conformance test run.
- **R-BEADS-007** SHOULD: Store implementations affected by this change (memstore, sqlite_store, native_dolt_store, caching_store_handles, caching_store_reads, bdstore) SHOULD be reviewed to confirm they delegate to the shared truth-table functions.

### Verify

```bash
# Verify that all store implementations delegate to shared truth-table functions
grep -r "IsBlockedByWorkOutcome\|filterReadyByWorkOutcome" internal/beads/*.go | grep -v "^internal/beads/beads.go" | wc -l

# Verify that BdStore applies blocked veto as post-filter
grep -A 10 "filterReadyByWorkOutcome" internal/beads/bdstore.go | grep -q "post-filter\|after.*candidate" && echo "PASS: BdStore applies veto as post-filter"

# Verify conformance suite includes all registered store implementations
grep -c "func.*Conformance" internal/beads/beadstest/conformance.go

# Verify no per-implementation blocked-outcome logic exists outside shared functions
grep -r "work_outcome.*blocked" internal/beads/*.go | grep -v "beads.go" | grep -v "conformance.go" | wc -l

# Verify conformance tests are run for all store registrations in CI
grep -r "conformance" .github/workflows/*.yml | grep -q "store" && echo "PASS: Conformance tests in CI"
```

**Accept when:**
- All store implementations delegate blocked-outcome checks to `IsBlockedByWorkOutcome` or `filterReadyByWorkOutcome`
- BdStore applies the blocked veto as a post-filter over native readiness candidates
- The conformance suite in `beadstest/conformance.go` validates the blocked-outcome truth table across all registered store implementations
- No per-implementation blocked-outcome logic exists outside the shared canonical functions
- Conformance tests are automatically included in CI for all store registrations
- Any new `gc.work_outcome` value intended to gate readiness is added to both the truth table and conformance suite in the same commit

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All beads-store implementations and dependency-readiness evaluation code MUST conform to R-BEADS-001 through R-BEADS-007 before merge. Conformance test results MUST pass for all registered store backends.
</enforcement>