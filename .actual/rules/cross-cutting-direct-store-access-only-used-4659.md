# Store-Residency Resolution Consolidated onto storeref.Topology: Direct Store Access Only Used Low

These rules are ALWAYS ACTIVE for all orchestration-layer code in cmd/gc, internal/dispatch, and any packages that perform store-routing or by-id residency resolution.

### Rules

- **R-STOREREF-001** MUST: Direct store access MUST only be used in low-level infrastructure code that is itself part of the storeref.Topology implementation; all higher-layer consumers MUST route through the Topology abstraction.
- **R-STOREREF-002** MUST: All store-routing logic in cmd/gc and internal/dispatch MUST use storeref.Topology or its derived functions (byIDBeadForTopology, drainResidency) rather than hand-rolled probe logic.
- **R-STOREREF-003** MUST: Reserved-prefix definitions MUST be imported from internal/beadmeta.ReservedPrefixes, not from internal/config.
- **R-STOREREF-004** SHOULD: New store-topology features SHOULD include census and fault-injection test coverage analogous to internal/storeref/relic_census.go and internal/storeref/topology_prefix_fault_test.go.
- **R-STOREREF-005** SHOULD: Migrated call sites SHOULD follow the reference implementations in cmd/gc/by_id_residency.go, cmd/gc/by_id_store_route.go, and internal/dispatch/drain_residency.go.

### Verify

```bash
# Audit for direct store access outside storeref.Topology in orchestration packages
grep -r "store\.Get\|store\.Set\|store\.Probe" cmd/gc internal/dispatch \
  --include="*.go" \
  | grep -v "storeref/topology" \
  | grep -v "_test.go" \
  && echo "FAIL: Direct store access found outside storeref.Topology" || echo "PASS: No unauthorized direct store access"

# Verify ReservedPrefixes is imported from internal/beadmeta, not internal/config
grep -r "internal/config.*ReservedPrefixes\|from internal/config.*reserved" cmd/gc internal/dispatch \
  --include="*.go" \
  && echo "FAIL: ReservedPrefixes imported from internal/config" || echo "PASS: ReservedPrefixes correctly sourced from internal/beadmeta"

# Verify all store-routing call sites use Topology abstraction
grep -r "byIDBeadForTopology\|drainResidency\|Topology" cmd/gc internal/dispatch \
  --include="*.go" \
  | wc -l | grep -q "^[1-9]" \
  && echo "PASS: Topology abstraction in use" || echo "FAIL: No Topology usage detected"

# Check for superseded Resolve function usage in non-test code
grep -r "Resolve(" cmd/gc internal/dispatch \
  --include="*.go" \
  | grep -v "_test.go" \
  | grep -v "storeref/topology.go" \
  && echo "WARN: Superseded Resolve function still in use" || echo "PASS: Superseded Resolve not used in production code"
```

**Accept when:**
- No direct store access (store.Get, store.Set, store.Probe) exists outside storeref.Topology in orchestration-layer packages.
- ReservedPrefixes is imported exclusively from internal/beadmeta, never from internal/config.
- All store-routing call sites in cmd/gc and internal/dispatch use storeref.Topology, byIDBeadForTopology, or drainResidency.
- The superseded Resolve function is not used in production code (test-only usage is acceptable during transition).
- New topology features include census and fault-injection test coverage.
- Reference implementations (by_id_residency.go, by_id_store_route.go, drain_residency.go) are followed for new migrations.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in R-STOREREF-001 through R-STOREREF-005 MUST be validated before approving changes to orchestration-layer store-routing logic.
</enforcement>