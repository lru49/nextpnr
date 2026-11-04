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

## 2026-11-04T11:55:52Z — fast pip-delay source bypass B1 PASS

- Valid pair `20261104T114735Z-fast-pip-delay-source-bypass-b1`; reference reproduced before candidate metrics. An earlier launcher error executed only a reference, no candidate, and was discarded as a comparison.
- Identity PASS: complete FASM/report bytes, parsed report, router history, checksums, Fmax and final iteration 91 all equal; expected timing exits 1/1. FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`.
- Reference → candidate: wall 399.17 → 373.61 s (-6.4033%), Router2 289.52 → 264.34 s (-8.6972%), HeAP 45.42 → 44.84 s. BOTH timing gates PASS.
- No B2 pair started while the B2 reference remains unreproduced (directive block 3). Next action is reference-only B2 reproduction, not a candidate pair.

## 2026-11-04T11:59:47Z â two B2 reference-only reproductions launched

- Root `20261104T115910Z-b2-reference-repro`; drivers 504643 (r1, reference core 19) and 504644 (r2, reference core 23), seed 1, one thread, 100 MHz, 8460 s cap. These are REFERENCE-ONLY runs; no candidate B2 pair or other lane-2 benchmark is active.
- Verified reference binary SHA-256 `c33194144895cd1a95098fe9ddfdfbb2c43a1c552e7c07e9b0ead979082fcdc3` and original B2 input SHA-256 `9271cdc99c199854e89a21615481fd541171e9b742c257140e57a46e6323c7f1`. Both outputs must reproduce the stored reference and each other before any candidate B2 pair.

## 2026-11-04T12:29:27Z â block 4 ACK; B1 confirmation live; B2 tier-2 plan

- B1 confirmation pair `20261104T122843Z-fast-pip-delay-source-bypass-b1-r2`: candidate driver 544364/core 26, reference driver 544363/core 27, seed 1, one thread, 100 MHz, 1920 s cap, detached with logs/PID files. Reference reproduction will be checked before candidate metrics.
- B2 plan from block 4: first pair on cores 26-27 after B1 confirmation passes; second on cores 18-19 after the reference reproduction runs finish. Every B2 result line will be labeled **B2 tier-2 (reference nondeterministic)**.
- At 30 minutes the two reference-only B2 runs remain live on cores 19/23 (nextpnr PIDs 504650/504651), within the 141-minute cap; no output has been compared yet.

ACK 2026-11-04T12:04Z lane2: block 4

## 2026-11-04T12:45:20Z â second B1 PASS; B2 reference status; two B2 tier-2 pairs live

- B1 confirmation `20261104T122843Z-fast-pip-delay-source-bypass-b1-r2`: PASS. Reference reproduced stored FASM/report/history/checksums/final iteration; candidate FASM/report bytes, parsed report, history, checksums and final iteration 91 all match. Wall 419.49 -> 395.56 s (-5.7045%), Router2 306.95 -> 283.47 s (-7.6495%); exits 1/1 with normal timing-only completion.
- **B2 tier-2 (reference nondeterministic)** reference-only root `20261104T115910Z-b2-reference-repro`: both runs completed normally with exit 0. r1 FASM `18d00b1ccde28e327fd7fc41024487dc501e4d36a4f509a7dbb178f5e99e5b4a`; r2 FASM `65d9ca5e7b08292ec7248a99be6e8c352330875a1f6b78d942924822b5ffb216` (DIFFER). Reports match each other at `62fd9c1b31399315491fd7ee694c51f868d108f95001d29eb659204a78fe9058`, but neither FASM/report matches the stored reference `403d98cb...` / `ca496ca3...`; the reference is not twice-reproduced.
- **B2 tier-2 (reference nondeterministic)** pair 1 `20261104T124335Z-fast-pip-delay-source-bypass-b2-r1-tier2` launched: candidate driver 569352/core 26, reference driver 569351/core 27, 8460 s cap.
- **B2 tier-2 (reference nondeterministic)** pair 2 `20261104T124445Z-fast-pip-delay-source-bypass-b2-r2-tier2` launched after the reference-only runs finished: candidate driver 571545/core 18, reference driver 571544/core 19, 8460 s cap.

## 2026-11-04T13:13:34Z â B2 tier-2 pairs at 30 minutes

- **B2 tier-2 (reference nondeterministic)** pair 1 remains live: nextpnr PIDs 569359 (candidate core 26) and 569358 (reference core 27), elapsed 29m41s. Pair 2 remains live: PIDs 571552 (candidate core 18) and 571551 (reference core 19), elapsed 28m31s. No exit files yet; no candidate metrics inspected; both remain well below the 141-minute kill threshold.

## 2026-11-04T13:15:30Z â block 5 ACK; next candidate claimed

- Block 5 item 1 was recorded with both B2 driver PIDs, cores and result directories in the 12:45Z entry and refreshed at 13:13Z. Cores 22-23 remain idle. Claimed `perf/l2-bel-location-flat` at 2026-11-04T13:15:29Z for HeAP bel-location lookup flat storage; build will use cores 28-39 while both B2 pairs continue.

ACK 2026-11-04T13:04Z lane2: block 5
