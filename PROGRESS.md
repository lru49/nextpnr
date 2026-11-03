# nextpnr-perf-lane progress

Append-only record. Times are UTC.

## 2026-11-02 21:25 — baseline run started
- Checked out `d56c470f523846ddfb5272c724cdac70b1781a51` as `perf/baseline`.
- Built the profiling binary successfully (`Release`, `-O2 -g -fno-omit-frame-pointer`).
- Launched the stripped reference on B1 and B2, seed 1, one thread, 100 MHz.

## 2026-11-02 21:36 — reference B1 complete
- Wall 623.63 s; maximum RSS 1,471,152 KiB; exit 1 because the 100 MHz constraint failed.
- Router2 converged to zero overuse at iteration 91 in 516.35 s; the previously observed iteration-700 plateau did not reproduce with the current B1 JSON.
- Post-placement Fmax 51.00 MHz; final Fmax 41.83 MHz. FASM SHA-256: `2edc647dfcb849ccb35eb6e3a59f480221684c60db92c5740b0acc4d906cbd05`.
- B2 is still running; its baseline and all profile data remain pending.

## 2026-11-02 21:50 — B1 profile complete
- Instrumented B1: 671.59 s wall, 557.15 s router2, 1,471,396 KiB maximum RSS, 665K samples, zero lost samples.
- Fmax, utilization, and all 91 router iterations match the reference. FASM payload after the version header is byte-identical; future compatibility builds will pin the reference version string.
- Recorded top-30 self-time symbols for whole run, analytic placement, simulated annealing, and router2 in `docs/perf/baseline-b1-profile-top30.txt`.
- Router profile leaders: `dict<WireId,int>::do_lookup` 19.68%, `Arch::getPipDelay` 12.80%, `dict<WireId,Loc>::do_lookup` 7.16%, and `score_wire_for_arc` 6.27%.

## 2026-11-02 22:40 — B2 baseline and profile complete
- Reference B2: 2814.37 s wall, 2,291,876 KiB maximum RSS, 857.63 s analytic iterations, 268.67 s SA, and 1650.34 s router2; exit 1 is timing-only.
- Instrumented B2: 2933.84 s wall, 1683.82 s router2, 2,291,836 KiB maximum RSS, and about 2 million samples.
- Both runs converged at router iteration 20 with post-place/final Fmax 36.24/31.76 MHz. Report JSON is identical and FASM payload after the version header is byte-identical.
- Recorded whole-run and per-phase top-30 symbols in `docs/perf/baseline-b2-profile-top30.txt`; router hash/delay lookups remain the leaders.

## 2026-11-03 00:10 — Router2 scoring fast path validated
- Branch `perf/router-score-fastpath`, implementation `326efc3f`; B1/B2 results are in `docs/perf/router-score-fastpath.md` on that branch.
- B1 companion-build wall 602.19 -> 553.27 s (-8.12%); Router2 494.45 -> 451.00 s (-8.79%); RSS +0.02%; 91 iterations and QoR unchanged.
- B2 companion-build wall 2666.79 -> 2563.22 s (-3.88%); Router2 1519.09 -> 1400.03 s (-7.84%); RSS +0.01%; 20 iterations and QoR unchanged.
- Both report JSON files and complete FASM files are byte-identical. Accepted on the topic branch, not yet merged to fork main. These are single-pair measurements; the patched B2 placement overlapped another validation, making its wall delta conservative.

## 2026-11-03 00:10 — Router2 queue-capacity candidate rejected
- Branch `perf/router-queue-capacity`, implementation `e68901e8`; results are in `docs/perf/router-queue-capacity.md` on that branch.
- B1 wall 603.82 -> 628.03 s (+3.98%); Router2 501.42 -> 519.38 s (+3.58%); RSS +0.02%; 91 iterations and QoR unchanged.
- Report JSON and complete FASM are identical, but the candidate regressed runtime and was abandoned without a B2 run.

## 2026-11-03 00:02 — Flat WireId index validation started
- Built and pushed `d538000d` on `perf/router-wire-index`; it replaces the hot `dict<WireId, int>` with tile offsets and a flat integer vector while preserving `flat_wires` order.
- Sequential B1 companion-build comparison is running on core 24; results are pending.

## 2026-11-03 00:10 — Optimization summary
| Change | B1 wall (s) | B2 wall (s) | QoR delta | FASM identical | Status |
| --- | ---: | ---: | --- | :---: | --- |
| Reference baseline | 623.63 | 2814.37 | baseline | n/a | measured concurrently |
| Router scoring fast path | 602.19 -> 553.27 | 2666.79 -> 2563.22 | none | yes | accepted on topic branch |
| Router queue capacity | 603.82 -> 628.03 | not run | none | yes | abandoned: runtime regression |
| Flat WireId index | running | pending | pending | pending | validation running |

## 2026-11-03 00:25 — Flat WireId index passes B1
- `d538000d` wall 578.45 -> 492.63 s (-14.84%); Router2 480.84 -> 395.47 s (-17.75%); maximum RSS 1,471,088 -> 1,268,560 KiB (-13.77%).
- Router iterations (91), post-place/final Fmax (51.00/41.83 MHz), report JSON, and complete FASM are unchanged.
- B1 results committed as `0611ad3c`; sequential B2 validation is running on core 24.

## 2026-11-03 01:40 — Flat WireId index passes B2
- `d538000d` wall 2662.71 -> 2399.76 s (-9.88%); Router2 1521.34 -> 1260.58 s (-17.14%); maximum RSS 2,292,424 -> 2,106,284 KiB (-8.12%).
- Router iterations (20), post-place/final Fmax (36.24/31.76 MHz), report JSON, and complete FASM are unchanged. FASM SHA-256: `5464b504d810fdc3b50d6ee3f87e51f0a20cf1a16025cd5ec747951e4dd1f97b`.
- B1/B2 results committed as `ddd6a509`; the candidate is accepted on `perf/router-wire-index`.

## 2026-11-03 01:40 — Updated optimization summary
| Change | B1 wall (s) | B2 wall (s) | QoR delta | FASM identical | Status |
| --- | ---: | ---: | --- | :---: | --- |
| Reference baseline | 623.63 | 2814.37 | baseline | n/a | measured concurrently |
| Router scoring fast path | 602.19 -> 553.27 | 2666.79 -> 2563.22 | none | yes | accepted on topic branch |
| Router queue capacity | 603.82 -> 628.03 | not run | none | yes | abandoned: runtime regression |
| Flat WireId index | 578.45 -> 492.63 | 2662.71 -> 2399.76 | none | yes | accepted on topic branch |

## 2026-11-03T12:46Z — directive acknowledgement and recovery
- ACK 2026-11-03T11:56Z lane: recovered the canonical progress and am preparing the stacked accepted candidates and four profile-driven B1 candidates for six core-pinned pairs with 3x timeouts and per-pair FASM gates.
- Pip-delay-cache B1: abandoned: runtime > 3x reference. Stored pair `20261103T091100Z-b1-pip-pair` records candidate exit 143 after the 32-minute timeout; no nextpnr process remains in this lane namespace. Do not rerun this candidate.
