# Blocked Work-Outcome Added to Dependency-Readiness Semantics: Conformance Tests Cover Blocked Outcome Row

These rules are ALWAYS ACTIVE for all beads store implementations, conformance test suites, and dependency-readiness evaluation code across the multi-backend work-orchestration layer.

### Rules

- **R-BEADS-001** MUST: Conformance tests MUST cover the blocked-outcome row of the dependency-readiness truth table and MUST be executed against every registered store implementation before a store is considered compliant.
- **R-BEADS-002** MUST: All store implementations MUST delegate blocked-outcome checks to the shared canonical functions `IsBlockedByWorkOutcome` and `filterReadyByWorkOutcome` in `internal/beads/beads.go` rather than re-implementing the check.
- **R-BEADS-003** MUST: When a closed dependency has `gc.work_outcome=blocked`, it MUST NOT be treated as a readiness signal for dependent work items, regardless of the store backend or external readiness source.
- **R-BEADS-004** MUST: Any new `gc.work_outcome` value intended to gate readiness MUST be added to the shared truth table and conformance suite as part of the same change that introduces the value.
- **R-BEADS-005** MUST: The conformance suite in `internal/beads/beadstest/conformance.go` MUST be kept in sync with the canonical truth table; when the truth table is amended, the conformance suite MUST be updated in the same commit.
- **R-BEADS-006** SHOULD: Store implementations that integrate with external readiness sources (e.g., BdStore) SHOULD apply the blocked-outcome veto as a post-filter on the final candidate set, not as a concurrent signal, following the BdStore reference implementation pattern.
- **R-BEADS-007** SHOULD: Registering a new store implementation in the beads registry SHOULD automatically include it in the conformance test run; this requirement SHOULD be documented in the store implementation guide.

### Verify

```bash
# Verify that conformance tests exist and cover blocked-outcome row
grep -r "blocked" internal/beads/beadstest/conformance.go || exit 1

# Verify that all store implementations delegate to shared truth-table functions
for store in memstore sqlite_store native_dolt_store caching_store_handles caching_store_reads bdstore; do
  grep -l "IsBlockedByWorkOutcome\|filterReadyByWorkOutcome" "internal/beads/${store}.go" || echo "WARNING: ${store} may not delegate to shared functions"
done

# Verify that canonical truth-table functions exist
grep -q "func IsBlockedByWorkOutcome" internal/beads/beads.go || exit 1
grep -q "func filterReadyByWorkOutcome" internal/beads/beads.go || exit 1

# Verify that BdStore applies blocked veto as post-filter
grep -A 10 "filterReadyByWorkOutcome" internal/beads/bdstore.go | grep -q "post-filter\|after" || echo "WARNING: BdStore veto application pattern unclear"

# Verify conformance suite is run in CI for all registered stores
grep -r "conformance" .github/workflows/*.yml || echo "WARNING: Conformance tests may not be in CI pipeline"
```

**Accept when:**
- Conformance tests explicitly validate the blocked-outcome row across all registered store implementations
- All store implementations delegate blocked-outcome checks to `IsBlockedByWorkOutcome` or `filterReadyByWorkOutcome`
- The canonical truth-table functions exist and are documented in `internal/beads/beads.go`
- BdStore applies the blocked veto as a post-filter on final readiness candidates
- The conformance suite is automatically included in CI for all store registrations
- No new `gc.work_outcome` values are introduced without corresponding truth-table and conformance-suite updates

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-BEADS rules marked MUST are non-negotiable; SHOULD rules represent best practices that should be followed unless explicitly overridden by project leadership. Verification commands MUST pass before any change affecting beads store implementations or dependency-readiness evaluation is merged.
</enforcement>