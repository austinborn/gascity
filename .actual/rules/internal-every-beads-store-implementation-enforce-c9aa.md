# Blocked Work-Outcome Added to Dependency-Readiness Semantics: Every Beads Store Implementation Enforce Canonical

These rules are ALWAYS ACTIVE for all beads store implementations and any code that evaluates dependency-readiness semantics across the beads substrate.

### Rules

- **R-BEADS-001** MUST: Every beads-store implementation MUST enforce the canonical dependency-readiness truth table, which includes the rule that a closed dependency with `gc.work_outcome=blocked` does NOT satisfy the readiness condition for any dependent work item.
- **R-BEADS-002** MUST: All store implementations MUST delegate to the shared truth-table functions `IsBlockedByWorkOutcome` and `filterReadyByWorkOutcome` in `internal/beads/beads.go` rather than re-implementing the blocked-outcome check.
- **R-BEADS-003** MUST: Any new `gc.work_outcome` value intended to gate readiness MUST be added to the shared truth table and conformance suite as part of the same change that introduces the value.
- **R-BEADS-004** MUST: The conformance suite in `internal/beads/beadstest/conformance.go` MUST be kept in sync with the canonical truth table; when the truth table is amended, the conformance suite MUST be updated in the same commit.
- **R-BEADS-005** MUST: Registering a new store implementation in the beads registry MUST automatically include it in the conformance test run.
- **R-BEADS-006** SHOULD: Store implementations that integrate with external readiness sources (e.g., BdStore) SHOULD apply the blocked-outcome veto as a post-filter over native readiness candidates, following the BdStore reference implementation pattern.
- **R-BEADS-007** SHOULD: The blocked-outcome veto in store implementations SHOULD be applied as a post-filter on the final candidate set, not as a concurrent signal, to ensure atomicity guarantees.

### Verify

```bash
# Verify that all registered store implementations pass the conformance suite
go test -v ./internal/beads/beadstest -run TestConformance

# Verify that IsBlockedByWorkOutcome and filterReadyByWorkOutcome are used by all stores
grep -r "IsBlockedByWorkOutcome\|filterReadyByWorkOutcome" \
  internal/beads/memstore.go \
  internal/beads/sqlite_store.go \
  internal/beads/native_dolt_store.go \
  internal/beads/caching_store_handles.go \
  internal/beads/caching_store_reads.go \
  internal/beads/bdstore.go

# Verify that no store implementation re-implements blocked-outcome logic
grep -r "work_outcome.*blocked" internal/beads/*.go | grep -v "IsBlockedByWorkOutcome\|filterReadyByWorkOutcome" | wc -l

# Verify that conformance suite is included in CI for all store registrations
grep -A 5 "RegisterStore" internal/beads/beads.go | grep -i conformance
```

**Accept when:**
- All registered store implementations pass the conformance suite without modification or skipping.
- Every store implementation delegates to `IsBlockedByWorkOutcome` or `filterReadyByWorkOutcome` for blocked-outcome evaluation.
- No store implementation contains duplicate or divergent blocked-outcome gating logic.
- The conformance suite is automatically executed for all registered stores in the CI pipeline.
- Any new `gc.work_outcome` value added to the codebase is accompanied by updates to both the truth table and the conformance suite in the same commit.
- BdStore (and any similar external-readiness-integrating stores) apply the blocked veto as a post-filter, not as a concurrent signal.

<enforcement>
Claude Code MUST NOT skip or defer verification. All conformance tests MUST pass before accepting changes to beads store implementations or dependency-readiness semantics. Any new outcome values MUST be added to the truth table and conformance suite in the same commit.
</enforcement>