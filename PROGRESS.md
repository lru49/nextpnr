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

## 2026-11-04T11:12Z — HeAP cell-location index abandoned at the B1 timing gate

- Candidate source `e78f7efc` is pushed as `perf/l2-heap-cell-location-index`, one commit over `40d3b873`. Only `common/place/placer_heap.cc` changed; `git diff --check` passed.
- Final xc7z020-only Release/IPO build used both C/C++ `ccache` launchers on cores 28-39: 61/61 steps passed in 122.94 s, peak RSS 2,678,080 KiB. Reference-chipdb `--test` passed. Candidate binary SHA-256 `d4f4b18c4604f38fe0b94be2be2f66e21b4e092d44b19713a1ecf2534429cff0`; reference `c33194144895cd1a95098fe9ddfdfbb2c43a1c552e7c07e9b0ead979082fcdc3`.
- Pair `20261104T110406Z-heap-cell-location-index-b1`: candidate core 18, reference core 19, seed 1, one thread, 100 MHz, 1920 s cap. Before candidate metrics, the reference reproduced FASM `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`, report `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`, checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`, history `91/284240/0/0/0` and normal timing-only exit 1.
- Candidate/reference exits 1/1; complete FASM and report bytes, parsed reports, full router history and checksums are identical. Identity and QoR pass.
- Reference -> candidate: whole wall 414.80 -> 412.47 s (**-0.5617%**), Router2 303.14 -> 302.41 s (**-0.2408%**), HeAP 45.84 -> 44.28 s (-3.4031%), analytic iteration sum 43.08 -> 41.72 s (-3.1569%), SA unchanged, peak RSS 1,558,080 -> 1,559,992 KiB (+0.1227%).
- Decision: **ABANDON `perf/l2-heap-cell-location-index`: identity passes, but both whole-wall and Router2 gains are below the required 3.0%.** No second B1 pair, B2 run, merge or accepted tag. No lane-2 benchmark process remains.

ACK 2026-11-04T11:04Z lane2: block 2

## 2026-11-04T11:27:09Z — fast-mode pip-delay source bypass named before build

- Base: `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39` (`origin/perf/accepted-qor`). Candidate `perf/l2-fast-pip-delay-source-bypass`, expected tier 1, touches only `Arch::getPipDelay`.
- Profile evidence: `getPipDelay` is 10.78% of B1 whole-run and 12.80% of B1 Router2 self time. In fast-delay routing the source-side RC product is multiplied by zero, but the base still normalises the source wire and walks its node timing on every query.
- Distinct from abandoned `perf/pip-delay-flat-r2`: add no delay table, cache, setup hook or persistent state. Skip only the source-side work whose product is provably zero in fast mode; retain the original non-fast path and destination arithmetic in the original order.
- Hypothesis: removing that dead source lookup and arithmetic reduces Router2 and whole wall time without changing any delay value, route decision, RNG or output.
- Kill criterion: one B1 pair on candidate core 18/reference core 19, seed 1, one thread, 100 MHz, 1920 s cap. Require reference reproduction before candidate metrics, expected timing-only completion, byte-identical complete FASM, identical report/history/checksums/Fmax/utilization/final overuse, and at least 3.0% improvement in BOTH Router2 and whole wall. Any failure abandons with no B2 or rerun. Build with both C/C++ ccache launchers on cores 28-39 only.
- Queue after this candidate: `perf/l2-bel-location-flat` (placement, tier 1) and `perf/l2-portref-iteration` (placement/traversal, tier 1); neither will be substituted for the current Router2-focused screen without a new profile rationale.

## 2026-11-04T11:37:11Z — fast pip-delay source bypass built and smoke-tested

- Candidate commit `cbfc8d7a`; build `build/perf-l2-fast-pip-delay-source-bypass`: 63/63, 120.38 s, peak RSS 2,677,820 kB, ccache launchers for C and C++.
- Candidate SHA-256 `47fc3bfbc2eab4f347f5d6e299000d84fd31eb376f58c268a13142a52b169776`; reference SHA-256 `c33194144895cd1a95098fe9ddfdfbb2c43a1c552e7c07e9b0ead979082fcdc3`.
- Generated version remains `perf-accepted-qor-revert-40d3b873`; reference-chipdb `--test` passed on build core 28.
