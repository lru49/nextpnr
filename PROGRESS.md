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
