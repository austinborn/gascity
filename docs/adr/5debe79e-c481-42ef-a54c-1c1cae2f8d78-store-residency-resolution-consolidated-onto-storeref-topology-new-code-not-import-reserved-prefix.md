# Store-Residency Resolution Consolidated onto storeref.Topology: New Code Not Import Reserved Prefix

Status: proposed
Date: 2025-01-30
Deciders: AI (signal conversion)

## Context

- The orchestration layer previously contained four divergent by-id store-resolution strategies spread across cmd/gc (cmd_wait.go's newWaitDependencyStoreSet, claim_class_route.go's holds logic) and internal/dispatch (drain.go's three probe sites). Each call site independently implemented its own store-selection logic to answer the question 'which store holds this id?'
- This fragmentation created a class of correctness bugs, exemplified by ga-cu12x, where callers read frozen pre-migration copies of store data because their hand-rolled probe logic did not share the same residency semantics as other parts of the system. With no single authoritative source for store routing, each new call site risked introducing a subtly different interpretation of residency.
- The storeref package has been extended with a Topology abstraction and a shared byIDBeadForTopology derivation that centralises the 'which store holds this id' question. Concurrently, the reserved-prefix table was extracted from internal/config into internal/beadmeta (as ReservedPrefixes) to break import cycles that would otherwise prevent the topology abstraction from being used uniformly across both cmd and internal packages.
- The legacy Resolve function has been marked superseded but is retained temporarily to allow test-level comparison against the new topology-based path, providing a regression safety net during the transition period.

## Problem Statement

Store-routing and by-id residency resolution were implemented independently at four or more call sites across the orchestration layer, making it impossible to enforce consistent residency semantics and creating recurring exposure to bugs caused by divergent store-selection logic.

## Decision

1. MUST_NOT: New code MUST NOT import reserved-prefix definitions from internal/config; it MUST use internal/beadmeta.ReservedPrefixes to avoid import cycles that would prevent uniform use of storeref.Topology.

## Policy Block

- MUST_NOT New code MUST NOT import reserved-prefix definitions from internal/config; it MUST use internal/beadmeta.ReservedPrefixes to avoid import cycles that would prevent uniform use of storeref.Topology.

## Rationale

- Consolidating store-routing onto a single abstraction means correctness of residency semantics needs to be proved and maintained in exactly one place rather than defended independently at every call site.
- Bug ga-cu12x demonstrated that divergent store-selection logic leads to callers reading stale, pre-migration data. A shared Topology abstraction eliminates this class of bug by ensuring all callers observe the same residency view.
- Extracting ReservedPrefixes into internal/beadmeta resolves the import-cycle constraint that previously prevented the topology abstraction from being used uniformly, making the architectural boundary enforceable in practice.
- Retaining the superseded Resolve function during the transition provides a regression baseline, reducing the risk of silent semantic regressions when migrating existing call sites.

## Consequences

Positive:
- A single, auditable boundary for store-routing correctness replaces four divergent implementations, reducing the surface area for residency-related bugs.
- Future store-topology changes (e.g., migrations, shard splits) need only be handled in storeref.Topology rather than propagated to every call site.
- Import-cycle elimination via internal/beadmeta makes the dependency graph cleaner and allows broader reuse of the topology abstraction.
- Test coverage of the topology path is now shared across all consumers rather than duplicated per call site.

Negative:
- All existing and future call sites must be updated or written to use storeref.Topology, increasing the upfront migration cost for any remaining hand-rolled probes.
- The superseded Resolve function introduces temporary dual-path complexity that must be actively managed and eventually removed.
- Centralisation means a bug in storeref.Topology has broader blast radius than a bug in a single isolated call site.

## Alternatives

- Retain per-call-site store probes but enforce a shared interface/contract via a common interface type rather than a concrete Topology struct. (rejected)
  Rejected because: An interface contract without a shared implementation still allows divergent residency semantics to be introduced at each implementation site, which was the root cause of ga-cu12x. A single shared derivation (byIDBeadForTopology) is required to guarantee uniform behaviour.
- Allow direct store access for read-only, non-routing operations (e.g., existence checks) while routing only write and migration operations through storeref.Topology. (rejected)
  Rejected because: The ga-cu12x bug was a read-path bug (reading frozen pre-migration copies). Exempting read operations from the topology abstraction would leave the exact failure mode that motivated this change unaddressed.
- Keep reserved-prefix definitions in internal/config and resolve import cycles through dependency inversion or a separate adapter package. (rejected)
  Rejected because: The extraction to internal/beadmeta was the chosen mechanism to break the import cycle. Adapter indirection would add complexity without improving the conceptual ownership of reserved-prefix metadata.

## Risks

- Undiscovered hand-rolled store probes in packages not covered by PR 6140 may continue to bypass storeref.Topology, silently reintroducing divergent residency semantics.
  Mitigation: Conduct a codebase-wide audit for direct store-probe patterns; add a linter or static-analysis rule that flags any store access outside of storeref.Topology in orchestration-layer packages.
  Owner: AI (signal conversion)
- A defect in the centralised storeref.Topology or byIDBeadForTopology derivation now affects all consumers simultaneously rather than being isolated to a single call site.
  Mitigation: Maintain the superseded Resolve function as a test oracle during the transition; invest in comprehensive topology-level tests including topology_prefix_fault_test.go scenarios before removing legacy paths.
  Owner: AI (signal conversion)
- The superseded Resolve function may persist beyond its intended lifetime, causing confusion about which path is canonical.
  Mitigation: Track removal of Resolve as a follow-up task with a defined deadline tied to completion of test-comparison validation; mark it with a deprecation comment referencing this ADR.
  Owner: AI (signal conversion)
- The internal/beadmeta package boundary may not be well understood by contributors, leading to ReservedPrefixes being re-imported from internal/config in new code.
  Mitigation: Add a package-level doc comment to internal/config explicitly stating that reserved-prefix definitions have moved to internal/beadmeta and that direct import for this purpose is forbidden.
  Owner: AI (signal conversion)

## Implementation Notes

- The primary implementation is in internal/storeref/topology.go and internal/storeref/bindings.go; new store-routing logic should be added there rather than in consuming packages.
- cmd/gc/by_id_residency.go and cmd/gc/by_id_store_route.go represent the migrated gc-layer call sites and should be used as reference implementations for migrating any remaining hand-rolled probes.
- internal/dispatch/drain_residency.go is the migrated dispatch-layer reference; drain.go's three former probe sites have been retired onto this shared derivation.
- internal/storeref/relic_census.go and internal/storeref/topology_prefix_fault_test.go provide census and fault-injection test coverage respectively; new topology features should include analogous tests.
- When removing the superseded Resolve function, verify that all test assertions previously relying on it for comparison have been ported to assert directly against the topology-based path.
- The import of internal/beadmeta.ReservedPrefixes must be used wherever reserved-prefix logic is needed; any remaining references to the old internal/config location should be updated as part of ongoing cleanup.

## References

- commit: ea66aa0fe673a4ca848f9e198514014265a81b6c
- head commit: 2e101dbaebc2863c395cfb634fb461a9ccdc4ead
- PR: 6140
- internal/storeref/topology.go
- internal/storeref/bindings.go
- internal/storeref/relic_census.go
- internal/storeref/topology_prefix_fault_test.go
- cmd/gc/by_id_residency.go
- cmd/gc/by_id_store_route.go
- cmd/gc/claim_class_route.go
- cmd/gc/cmd_wait.go
- internal/dispatch/drain.go
- internal/dispatch/drain_residency.go
- internal/beadmeta/reserved_prefixes.go
- symbols: Topology, byIDBeadForTopology, drainResidency, ReservedPrefixes