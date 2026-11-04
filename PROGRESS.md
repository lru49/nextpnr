# nextpnr performance lane 2

## 2026-11-04 — initialization and first candidate

- Base commit: `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39` (`origin/perf/accepted-qor`).
- Intended candidate: `perf/l2-heap-cell-location-index`, one commit on the base above.
- Claims: `origin/perf/claims` initialized from `perf/baseline`; active claim is `perf/l2-heap-cell-location-index` for `HeAP/build_equations+solve_equations+total_hpwl`.
- Candidate queue:
  1. `perf/l2-heap-cell-location-index` — HeAP analytic placement; replace hot `cell_locs` hash lookups with a name-indexed pointer table built after seeding; expected tier 1.
  2. `perf/l2-bel-location-flat` — Himbaechel analytic/SA placement; evaluate a flat per-BelId location table for `Arch::getBelLocation`; expected tier 1.
  3. `perf/l2-portref-iteration` — HeAP/SA port traversal; inspect and reduce `indexed_store<PortRef>` iterator increment overhead without changing traversal order; expected tier 1.
- Hypothesis: `cell_locs` is fully populated by seeding and not grown thereafter, so stable pointers indexed by `IdString.index` preserve every value and access order while removing repeated hashing from equation construction, solving, and HPWL evaluation.
- Kill criterion: abandon on any pointer-lifetime invalidation, build/smoke failure, reference or FASM/QoR mismatch, or a paired B1 wall regression greater than 2%; do not start B2 unless every B1 gate passes.
- No source edit, build, or benchmark has started.

ACK 2026-11-04T10:34Z lane2: block 1
