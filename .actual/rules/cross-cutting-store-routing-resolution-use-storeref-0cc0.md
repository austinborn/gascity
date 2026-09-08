# Store-Residency Resolution Consolidated onto storeref.Topology: Store Routing Resolution Use Storeref Topology

These rules are ALWAYS ACTIVE for all store-routing and by-id residency resolution code in the orchestration layer, including cmd/gc, internal/dispatch, and any new call sites that need to determine which store holds a given id.

### Rules

- **R-STOREREF-001** MUST: All store-routing and by-id resolution MUST use `storeref.Topology` and its shared derivations (e.g., `byIDBeadForTopology`).
- **R-STOREREF-002** MUST: Hand-rolled store-probe logic MUST NOT be implemented at individual call sites; all residency queries MUST be delegated to the centralised topology abstraction.
- **R-STOREREF-003** MUST: Any new store-routing logic MUST be added to `internal/storeref/topology.go` or `internal/storeref/bindings.go` rather than in consuming packages.
- **R-STOREREF-004** MUST: Reserved-prefix definitions MUST be imported from `internal/beadmeta.ReservedPrefixes`, not from `internal/config`.
- **R-STOREREF-005** SHOULD: New topology features SHOULD include test coverage analogous to `internal/storeref/relic_census.go` and `internal/storeref/topology_prefix_fault_test.go`.
- **R-STOREREF-006** SHOULD: Reference implementations in `cmd/gc/by_id_residency.go`, `cmd/gc/by_id_store_route.go`, and `internal/dispatch/drain_residency.go` SHOULD be used as templates when migrating remaining hand-rolled probes.

### Verify

```bash
# Audit for hand-rolled store probes in orchestration-layer packages
grep -r "store\.Get\|store\.Probe\|store\.Select" cmd/gc internal/dispatch \
  | grep -v "storeref\.Topology" \
  | grep -v "byIDBeadForTopology" \
  | grep -v "drainResidency" \
  && echo "FAIL: Hand-rolled store probes detected outside storeref.Topology" || echo "PASS: No hand-rolled probes found"

# Verify ReservedPrefixes is imported from internal/beadmeta, not internal/config
grep -r "internal/config.*ReservedPrefixes" cmd/gc internal/dispatch \
  && echo "FAIL: ReservedPrefixes imported from internal/config" || echo "PASS: ReservedPrefixes correctly sourced from internal/beadmeta"

# Verify all store-routing call sites use storeref.Topology
grep -r "which store holds" cmd/gc internal/dispatch \
  | grep -v "storeref\.Topology" \
  && echo "FAIL: Store-routing logic not using storeref.Topology" || echo "PASS: All store-routing uses storeref.Topology"

# Check that internal/config has a deprecation comment about ReservedPrefixes
grep -A 2 "package config" internal/config/config.go \
  | grep -i "reserved.*prefix.*moved.*beadmeta" \
  && echo "PASS: Deprecation comment present" || echo "WARN: Add deprecation comment to internal/config"
```

**Accept when:**
- No hand-rolled store probes are found outside `storeref.Topology` in orchestration-layer packages.
- All imports of `ReservedPrefixes` reference `internal/beadmeta`, not `internal/config`.
- All store-routing and by-id residency queries use `storeref.Topology` or its shared derivations.
- `internal/config` includes a deprecation comment directing contributors to `internal/beadmeta` for reserved-prefix definitions.
- New topology features include test coverage at the topology level (not duplicated per call site).
- Reference implementations in `cmd/gc/by_id_residency.go`, `cmd/gc/by_id_store_route.go`, and `internal/dispatch/drain_residency.go` are used as templates for any remaining migrations.

<enforcement>
Claude Code MUST NOT skip or defer verification. All store-routing and by-id residency resolution code MUST be audited against these rules before approval. Any hand-rolled store probes, direct internal/config imports of ReservedPrefixes, or call sites bypassing storeref.Topology MUST be flagged and remediated.
</enforcement>