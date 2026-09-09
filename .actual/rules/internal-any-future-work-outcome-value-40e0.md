# Blocked Work-Outcome Added to Dependency-Readiness Semantics: Any Future Work Outcome Value Intended

These rules are ALWAYS ACTIVE for all beads store implementations, conformance test suites, and any code that introduces new `gc.work_outcome` values intended to gate dependency resolution.

### Rules

- **R-OUTCOME-001** MUST: Any future `gc.work_outcome` value intended to gate dependency resolution MUST be added to the canonical truth table in `IsBlockedByWorkOutcome` and `filterReadyByWorkOutcome` before being relied upon in any store implementation.
- **R-OUTCOME-002** MUST: Any new outcome value gating semantics MUST be validated through the shared conformance suite in `internal/beads/beadstest/conformance.go` as part of the same change that introduces the value.
- **R-OUTCOME-003** MUST: All store implementations MUST delegate blocked-outcome checks to the shared truth-table functions (`IsBlockedByWorkOutcome`, `filterReadyByWorkOutcome`) rather than re-implementing the check.
- **R-OUTCOME-004** MUST: BdStore and any store integrating with external readiness sources MUST apply the blocked-outcome veto as a post-filter on the final candidate set, not as a concurrent signal.
- **R-OUTCOME-005** MUST: Registering a new store implementation in the beads registry MUST automatically include it in the conformance test run.
- **R-OUTCOME-006** SHOULD: When the canonical truth table is amended, the conformance suite SHOULD be updated in the same commit.
- **R-OUTCOME-007** SHOULD: Store implementations affected by this change (memstore, sqlite_store, native_dolt_store, caching_store_handles, caching_store_reads, bdstore) SHOULD be reviewed to confirm they delegate to shared truth-table functions.

### Verify

```bash
# Verify that all new gc.work_outcome values are in the canonical truth table
grep -r "gc.work_outcome" internal/beads/*.go | grep -v "blocked" | grep -E "(failed|cancelled|pending)" && echo "FAIL: New outcome value found outside truth table" || echo "PASS: No undeclared outcome values"

# Verify that IsBlockedByWorkOutcome and filterReadyByWorkOutcome are used consistently
grep -l "IsBlockedByWorkOutcome\|filterReadyByWorkOutcome" internal/beads/*store*.go | wc -l | grep -q "[5-9]" && echo "PASS: Shared functions used in store implementations" || echo "FAIL: Insufficient delegation to shared functions"

# Verify conformance suite includes all registered stores
grep -c "registerStore\|conformance" internal/beads/beadstest/conformance.go | grep -q "[0-9]" && echo "PASS: Conformance suite present" || echo "FAIL: Conformance suite missing"

# Verify BdStore applies blocked veto as post-filter
grep -A 10 "filterReadyByWorkOutcome" internal/beads/bdstore.go | grep -q "post.*filter\|after.*candidate" && echo "PASS: BdStore applies veto as post-filter" || echo "WARN: Verify BdStore veto ordering"
```

**Accept when:**
- All new `gc.work_outcome` values intended for gating are present in the canonical truth table functions.
- All store implementations delegate blocked-outcome checks to shared functions rather than re-implementing.
- The conformance suite in `beadstest/conformance.go` validates the truth table across all registered store backends.
- BdStore and external-readiness-aware stores apply the blocked veto as a post-filter on final candidates.
- New store registrations automatically include conformance test coverage.
- No undeclared outcome values are found in store readiness evaluation paths.

<enforcement>
Claude Code MUST NOT skip or defer verification. Any change introducing a new `gc.work_outcome` value or modifying store readiness logic MUST pass all verify commands and satisfy all accept criteria before approval.
</enforcement>