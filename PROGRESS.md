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
