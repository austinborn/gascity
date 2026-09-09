# Store-Residency Resolution Consolidated onto storeref.Topology: New Code Not Import Reserved Prefix

These rules are ALWAYS ACTIVE for all new code in the orchestration layer (cmd/gc, internal/dispatch, internal/storeref) that performs store-routing or by-id residency resolution.

### Rules

- **R-STOREREF-001** MUST NOT: New code MUST NOT import reserved-prefix definitions from `internal/config`; it MUST use `internal/beadmeta.ReservedPrefixes` to avoid import cycles that would prevent uniform use of `storeref.Topology`.
- **R-STOREREF-002** MUST: All store-routing and by-id residency resolution logic MUST be implemented via `storeref.Topology` and `byIDBeadForTopology` rather than hand-rolled store-probe logic at call sites.
- **R-STOREREF-003** SHOULD: New store-topology features SHOULD include census and fault-injection test coverage analogous to `internal/storeref/relic_census.go` and `internal/storeref/topology_prefix_fault_test.go`.
- **R-STOREREF-004** SHOULD: Reference implementations in `cmd/gc/by_id_residency.go`, `cmd/gc/by_id_store_route.go`, and `internal/dispatch/drain_residency.go` SHOULD be used as templates when migrating existing hand-rolled probes.

### Verify

```bash
# Check for forbidden imports of reserved-prefix from internal/config in new code
grep -r "from.*internal/config.*ReservedPrefix" cmd/gc internal/dispatch internal/storeref 2>/dev/null | grep -v internal/beadmeta && exit 1 || true

# Verify all store-routing calls use storeref.Topology
grep -r "store.*probe\|store.*select\|store.*route" cmd/gc internal/dispatch internal/storeref 2>/dev/null | grep -v "Topology\|byIDBeadForTopology" | wc -l | grep -E "^0$" || echo "Warning: hand-rolled store probes detected"

# Check that internal/beadmeta.ReservedPrefixes is imported where needed
grep -r "ReservedPrefix" cmd/gc internal/dispatch internal/storeref 2>/dev/null | grep -v "internal/beadmeta" | grep -v "internal/config" | wc -l
```

**Accept when:**
- No imports of reserved-prefix definitions from `internal/config` are found in new code.
- All store-routing logic uses `storeref.Topology` or `byIDBeadForTopology` derivations.
- `internal/beadmeta.ReservedPrefixes` is used consistently where reserved-prefix logic is needed.
- New topology features include corresponding census and fault-injection tests.
- Migrated call sites follow the reference implementations in `by_id_residency.go`, `by_id_store_route.go`, and `drain_residency.go`.

<enforcement>
Claude Code MUST NOT skip or defer verification. All new code touching store-routing or residency resolution MUST be checked against R-STOREREF-001 through R-STOREREF-004 before approval.
</enforcement>