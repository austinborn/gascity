# Store-Residency Resolution Consolidated onto storeref.Topology: Superseded Resolve Function Removed Once Test

These rules are ALWAYS ACTIVE for all files in the orchestration layer (cmd/gc, internal/dispatch, internal/storeref) that perform store-routing or by-id residency resolution.

### Rules

- **R-STOREREF-001** MUST: All store-routing and by-id residency resolution MUST use storeref.Topology and byIDBeadForTopology rather than hand-rolled store-probe logic.
- **R-STOREREF-002** MUST: Reserved-prefix definitions MUST be imported from internal/beadmeta.ReservedPrefixes, not from internal/config.
- **R-STOREREF-003** SHOULD: New store-topology features SHOULD include census and fault-injection test coverage analogous to internal/storeref/relic_census.go and internal/storeref/topology_prefix_fault_test.go.
- **R-STOREREF-004** SHOULD: Existing hand-rolled store probes SHOULD be migrated to use the shared Topology abstraction following the reference implementations in cmd/gc/by_id_residency.go, cmd/gc/by_id_store_route.go, and internal/dispatch/drain_residency.go.
- **R-STOREREF-005** MUST: The superseded Resolve function MUST be removed once all test comparisons against the topology-based path have been validated and the transition period is complete.
- **R-STOREREF-006** MUST: Any remaining references to reserved-prefix logic in internal/config MUST be updated to use internal/beadmeta.ReservedPrefixes as part of ongoing cleanup.

### Verify

```bash
# Audit for hand-rolled store probes in orchestration-layer packages
grep -r "store\.Get\|store\.Probe\|store\.Select" cmd/gc internal/dispatch --include="*.go" | grep -v "storeref.Topology" | grep -v "byIDBeadForTopology" && echo "FAIL: Found hand-rolled store probes outside storeref.Topology" || echo "PASS: No hand-rolled probes detected"

# Verify ReservedPrefixes is imported from internal/beadmeta, not internal/config
grep -r "internal/config.*ReservedPrefixes\|from.*config.*ReservedPrefixes" cmd/gc internal/dispatch internal/storeref --include="*.go" && echo "FAIL: ReservedPrefixes imported from wrong location" || echo "PASS: ReservedPrefixes correctly sourced from internal/beadmeta"

# Check that Resolve function is marked as superseded and tracked for removal
grep -n "Resolve.*superseded\|Resolve.*deprecated\|TODO.*Resolve.*remove" internal/storeref/*.go && echo "PASS: Resolve function marked for removal" || echo "WARN: Resolve function deprecation status unclear"

# Verify topology-level test coverage exists
test -f internal/storeref/relic_census.go && test -f internal/storeref/topology_prefix_fault_test.go && echo "PASS: Topology test coverage present" || echo "FAIL: Missing topology test files"

# Confirm reference implementations are in place
test -f cmd/gc/by_id_residency.go && test -f cmd/gc/by_id_store_route.go && test -f internal/dispatch/drain_residency.go && echo "PASS: Reference implementations present" || echo "FAIL: Missing reference implementation files"
```

**Accept when:**
- No hand-rolled store probes are found outside storeref.Topology in orchestration-layer packages
- ReservedPrefixes is consistently imported from internal/beadmeta, never from internal/config
- The superseded Resolve function is marked with a deprecation comment referencing this ADR and tracked for removal
- Topology-level test coverage (relic_census.go and topology_prefix_fault_test.go) is present and comprehensive
- Reference implementations (by_id_residency.go, by_id_store_route.go, drain_residency.go) are in place and used as templates for migration
- All test assertions previously relying on Resolve for comparison have been ported to assert directly against the topology-based path

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-STOREREF rules MUST be validated before accepting changes to store-routing or residency-resolution logic in the orchestration layer.
</enforcement>