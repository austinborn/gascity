# Store-Residency Resolution Consolidated onto storeref.Topology: Hand Rolled Store Probes That Independently

These rules are ALWAYS ACTIVE for all files in cmd/gc, internal/dispatch, and other orchestration-layer packages that perform store-routing or by-id residency resolution.

### Rules

- **R-STOREREF-001** MUST NOT: Hand-rolled store probes that independently implement store-selection or by-id residency logic MUST NOT be introduced in cmd/gc, internal/dispatch, or any other orchestration-layer package.
- **R-STOREREF-002** MUST: All store-routing and by-id residency resolution MUST use storeref.Topology and the shared byIDBeadForTopology derivation.
- **R-STOREREF-003** MUST: Reserved-prefix definitions MUST be imported from internal/beadmeta.ReservedPrefixes, not from internal/config.
- **R-STOREREF-004** SHOULD: New store-topology features SHOULD include census and fault-injection test coverage analogous to internal/storeref/relic_census.go and internal/storeref/topology_prefix_fault_test.go.
- **R-STOREREF-005** MUST: Any direct store access for routing decisions MUST be routed through storeref.Topology, including read-only operations.

### Verify

```bash
# Audit for hand-rolled store probes in orchestration-layer packages
grep -r "store.*Select\|store.*Probe\|residency.*logic" cmd/gc internal/dispatch --include="*.go" | grep -v "storeref.Topology" | grep -v "byIDBeadForTopology" && echo "FAIL: Hand-rolled store probes detected" || echo "PASS: No hand-rolled store probes found"

# Verify all store-routing uses storeref.Topology
grep -r "storeref\.Topology" cmd/gc internal/dispatch --include="*.go" | wc -l

# Check for direct internal/config.ReservedPrefixes imports in orchestration layer
grep -r "internal/config.*ReservedPrefixes" cmd/gc internal/dispatch --include="*.go" && echo "FAIL: internal/config.ReservedPrefixes still imported" || echo "PASS: Using internal/beadmeta.ReservedPrefixes"

# Verify internal/beadmeta imports are used
grep -r "internal/beadmeta.*ReservedPrefixes" cmd/gc internal/dispatch --include="*.go" | wc -l

# Check for topology test coverage
find internal/storeref -name "*test.go" | xargs grep -l "Topology\|byIDBeadForTopology" | wc -l
```

**Accept when:**
- No hand-rolled store probes are detected in cmd/gc, internal/dispatch, or other orchestration-layer packages
- All store-routing logic uses storeref.Topology and byIDBeadForTopology
- Reserved-prefix definitions are imported exclusively from internal/beadmeta
- No direct internal/config.ReservedPrefixes imports exist in orchestration-layer code
- Topology-level test coverage exists for census and fault-injection scenarios
- All read-path and write-path store access is routed through the centralized abstraction

<enforcement>
Claude Code MUST NOT skip or defer verification. All store-routing and by-id residency resolution in orchestration-layer packages MUST be audited against R-STOREREF-001 through R-STOREREF-005 before code review approval.
</enforcement>