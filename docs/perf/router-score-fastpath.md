# Router scoring fast path

Source revision: `d56c470f523846ddfb5272c724cdac70b1781a51`.

## Change

- Compute `get_base_cost()` once per forward or backward expansion and reuse the value for both the accumulated delay and the arc score.
- Pass the already-indexed `PerWireData` into `score_wire_for_arc()` instead of looking up the same flat wire again.
- Replace `count()` followed by `at()` with one `find()` in the arc-score and to-go-cost net-wire lookups.

The patch does not change queue ordering, iteration limits, congestion updates, random-number consumption, or floating-point expressions. The removed calls are duplicate lookups of the same values, so placement, routing, QoR, and FASM are expected to be identical.

## Build and validation

Release build with IPO, the same xc7z020 target and chipdb inputs as the unpatched companion build, and `CURRENT_GIT_VERSION=d56c470f`.

Benchmarks use seed 1, one worker thread, and a 100 MHz timing target. Compare wall time, maximum RSS, router iterations and phase time, utilization, post-place/final Fmax, and complete FASM files against the unpatched build. A speedup is reported only with the companion-build comparison; the initial reference timings were concurrent and provisional.

## B1 results

The unpatched and patched Release builds ran sequentially on the same CPU core. Both exited 1 only because the 100 MHz timing target was not met.

| Metric | Unpatched | Patched | Change |
| --- | ---: | ---: | ---: |
| Wall time | 602.19 s | 553.27 s | -8.12% |
| CPU user time | 600.88 s | 551.97 s | -8.14% |
| Maximum RSS | 1,471,088 KiB | 1,471,400 KiB | +0.02% |
| Analytic iteration time sum | 44.45 s | 44.09 s | -0.81% |
| SA placement time | 53.82 s | 49.04 s | -8.88% |
| Router2 time | 494.45 s | 451.00 s | -8.79% |
| Router iterations | 91 | 91 | unchanged |
| Post-place Fmax | 51.00 MHz | 51.00 MHz | unchanged |
| Final Fmax | 41.83 MHz | 41.83 MHz | unchanged |

The complete parsed report JSON (utilization, Fmax, and critical paths) is identical. The complete FASM files are byte-identical, with SHA-256 `2edc647dfcb849ccb35eb6e3a59f480221684c60db92c5740b0acc4d906cbd05`. The untouched SA phase also varied, so this is a single-pair measurement rather than a noise-bound estimate; the router phase supplies the largest absolute saving.

## B2 results

| Metric | Unpatched | Patched | Change |
| --- | ---: | ---: | ---: |
| Wall time | 2666.79 s | 2563.22 s | -3.88% |
| CPU user time | 2664.28 s | 2560.66 s | -3.89% |
| Maximum RSS | 2,292,292 KiB | 2,292,440 KiB | +0.01% |
| Analytic iteration time sum | 848.26 s | 857.44 s | +1.08% |
| SA placement time | 262.62 s | 269.16 s | +2.49% |
| Router2 time | 1519.09 s | 1400.03 s | -7.84% |
| Router iterations | 20 | 20 | unchanged |
| Post-place Fmax | 36.24 MHz | 36.24 MHz | unchanged |
| Final Fmax | 31.76 MHz | 31.76 MHz | unchanged |

Both exits were timing-only. Report JSON is identical and complete FASM is byte-identical, with SHA-256 `5464b504d810fdc3b50d6ee3f87e51f0a20cf1a16025cd5ec747951e4dd1f97b`. A separate queue-candidate B1 validation overlapped part of the patched placement phase, so the wall and placement figures are conservative single-pair measurements; that validation had finished before patched Router2 started. The Router2 saving is consistent with B1 and the patch is accepted on this branch.
