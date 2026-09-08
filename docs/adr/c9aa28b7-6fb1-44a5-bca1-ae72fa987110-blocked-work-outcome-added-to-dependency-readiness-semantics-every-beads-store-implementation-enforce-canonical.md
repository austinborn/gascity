# Blocked Work-Outcome Added to Dependency-Readiness Semantics: Every Beads Store Implementation Enforce Canonical

Status: proposed
Date: 2025-01-30
Deciders: AI (signal conversion)

## Context

- The beads substrate is a multi-backend work-orchestration layer that supports several store implementations: in-memory (memstore), caching, SQLite, native Dolt, exec, and bd-backed (BdStore). Each store is responsible for evaluating whether a dependent work item is 'ready' to execute based on the state of its upstream dependencies.
- Prior to this change, the `gc.work_outcome` field on a closed dependency was treated as pure metadata and did not influence dependency-readiness evaluation. A closed dependency—regardless of its outcome—would unlock downstream work items.
- Commit 1928ef64cec34d6d75e6e7553ea3446b32e9801d (PR #6100) introduced `WorkOutcomeBlocked`, `IsBlockedByWorkOutcome`, and `filterReadyByWorkOutcome` symbols across the beads package. These symbols encode a canonical truth table: a closed dependency whose `gc.work_outcome` equals `blocked` must NOT be treated as a readiness signal for dependent work.
- BdStore applies this blocked veto on top of native bd-ready candidates, ensuring that even externally computed readiness signals are overridden when a blocking outcome is present. Conformance tests in `beadstest/conformance.go` have been extended to validate this truth table across every registered store implementation.
- This change elevates `gc.work_outcome` from an observational field to an active orchestration gate, making it a first-class element of the beads data contract shared by all store backends.

## Problem Statement

The beads dependency-readiness contract was incomplete: a closed dependency with `gc.work_outcome=blocked` incorrectly unlocked downstream work, because outcome values were not part of the readiness truth table. This created a semantic gap where blocked outcomes were recorded but had no effect on workflow gating, and the behavior was inconsistent across storage backends.

## Decision

1. MUST: Every beads-store implementation MUST enforce the canonical dependency-readiness truth table, which includes the rule that a closed dependency with `gc.work_outcome=blocked` does NOT satisfy the readiness condition for any dependent work item.

## Policy Block

- MUST Every beads-store implementation MUST enforce the canonical dependency-readiness truth table, which includes the rule that a closed dependency with `gc.work_outcome=blocked` does NOT satisfy the readiness condition for any dependent work item.

## Rationale

- Workflow correctness requires that a blocked upstream outcome propagates a halt signal to downstream work. Without this gate, downstream tasks could execute on the basis of a dependency that was explicitly marked as blocked, producing incorrect or unsafe orchestration behavior.
- Encoding the truth table in shared symbols (`IsBlockedByWorkOutcome`, `filterReadyByWorkOutcome`) and validating it through a shared conformance suite ensures that all store backends—including future ones—implement identical semantics, making dependency behavior portable across storage substrates.
- Applying the blocked veto atop externally computed readiness signals (as BdStore does) is necessary because external systems may not be aware of the beads-level outcome semantics; the beads layer must be the authoritative enforcer of its own data contract.
- Extending conformance tests rather than relying on per-implementation unit tests is the correct approach because it prevents semantic drift between backends and provides a single source of truth for what 'correct' dependency-readiness behavior means.

## Consequences

Positive:
- Dependency-readiness behavior is now consistent and portable across all beads store backends.
- The `gc.work_outcome` field becomes a first-class orchestration signal, enabling richer workflow gating without requiring schema changes.
- Conformance test coverage provides a regression safety net for all current and future store implementations.
- The canonical truth table is centralized, reducing the risk of per-backend divergence.

Negative:
- Existing workflows that relied on a closed-but-blocked dependency inadvertently unlocking downstream work will now be halted; this is a breaking behavioral change for any such workflows.
- Every new beads-store implementation must now pass the extended conformance suite, increasing the implementation burden for new backends.
- The management question of whether additional outcome values should gate readiness remains open; deferring this decision may require future truth-table amendments and re-conformance of all backends.

## Alternatives

- Treat `gc.work_outcome` as pure metadata and handle blocked-outcome gating at the orchestration layer above the store. (rejected)
  Rejected because: Pushing this logic above the store layer would require every orchestration consumer to re-implement the same gating logic, creating duplication and the risk of inconsistent behavior. The store is the correct enforcement point because it is the single authoritative source of dependency state.
- Gate on all non-successful outcome values rather than only `blocked`. (deferred)
  When valid: Valid if the management question is resolved and additional outcome values (e.g., `failed`, `cancelled`) are determined to require the same gating semantics. Should be addressed as a follow-on truth-table amendment with full conformance coverage.
- Implement blocked-outcome handling per store without a shared conformance suite. (rejected)
  Rejected because: Per-implementation handling without shared conformance tests would allow semantic drift between backends over time, undermining the portability guarantee that is the primary motivation for this change.

## Risks

- Existing workflows that depended on the previous (incorrect) behavior—where a blocked dependency still unlocked downstream work—will silently stall after this change is deployed.
  Mitigation: Audit active workflows for dependencies with `gc.work_outcome=blocked` before deploying this change. Provide migration guidance or a compatibility flag if stalled workflows are discovered.
  Owner: AI (signal conversion)
- Future `gc.work_outcome` values may be introduced without updating the canonical truth table, causing inconsistent gating behavior that is not caught until runtime.
  Mitigation: Enforce rule-5: any new outcome value intended to gate readiness must be added to the shared truth table and conformance suite as part of the same change that introduces the value. Code review and CI conformance gates should enforce this.
  Owner: AI (signal conversion)
- BdStore's layered veto (applying blocked semantics atop native bd-ready candidates) may introduce subtle ordering or race conditions if the native bd layer and the beads layer evaluate readiness asynchronously.
  Mitigation: Ensure that the blocked-outcome veto in BdStore is applied as a post-filter on the final candidate set, not as a concurrent signal. Review BdStore's readiness evaluation path for atomicity guarantees.
  Owner: AI (signal conversion)
- New store implementations may pass unit tests but fail the conformance suite if the conformance suite is not automatically included in the CI pipeline for all store registrations.
  Mitigation: Require that registering a new store implementation in the beads registry automatically includes it in the conformance test run. Document this requirement in the store implementation guide.
  Owner: AI (signal conversion)

## Implementation Notes

- The canonical blocked-outcome logic is implemented in `IsBlockedByWorkOutcome` and `filterReadyByWorkOutcome` in `internal/beads/beads.go`. All store implementations should delegate to these functions rather than re-implementing the check.
- BdStore (`internal/beads/bdstore.go`) applies the blocked veto as a post-filter over native bd-ready candidates. This pattern should be documented as the reference implementation for stores that integrate with external readiness sources.
- The conformance suite in `internal/beads/beadstest/conformance.go` must be kept in sync with the canonical truth table. When the truth table is amended, the conformance suite must be updated in the same commit.
- The management question—whether other `gc.work_outcome` values should gate dependency resolution—should be tracked as a follow-on work item. Until resolved, `blocked` is the only outcome value with gating semantics.
- Store implementations affected by this change include: `internal/beads/memstore.go`, `internal/beads/sqlite_store.go`, `internal/beads/native_dolt_store.go`, `internal/beads/caching_store_handles.go`, and `internal/beads/caching_store_reads.go`. Each should be reviewed to confirm they delegate to the shared truth-table functions.

## References

- internal/beads/bdstore.go
- internal/beads/beads.go
- internal/beads/beadstest/conformance.go
- internal/beads/caching_store_handles.go
- internal/beads/caching_store_reads.go
- internal/beads/memstore.go
- internal/beads/native_dolt_store.go
- internal/beads/sqlite_store.go
- commit:1928ef64cec34d6d75e6e7553ea3446b32e9801d
- commit:2e101dbaebc2863c395cfb634fb461a9ccdc4ead (head)
- PR #6100
- symbols: filterReadyByWorkOutcome, IsBlockedByWorkOutcome, WorkOutcomeBlocked