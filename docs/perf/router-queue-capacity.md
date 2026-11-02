# Router queue capacity retention

Source revision: `d56c470f523846ddfb5272c724cdac70b1781a51`.

## Change

- Derive the forward/backward queue type from `std::priority_queue` to expose the protected underlying vector.
- Clear that vector between arc searches instead of destroying and swapping the queue, retaining its allocation.
- Reserve `flat_wires.size()` entries in both queues when each routing thread context is created.

The patch changes allocation behavior only. It does not change queue elements, comparison, ordering, random-number consumption, search limits, or routing updates, so report JSON and FASM are expected to be identical. Maximum RSS is an explicit acceptance metric because up-front reservation can increase memory use.

## Build and validation

Release build with IPO, the same xc7z020 target and chipdb inputs as the unpatched companion build, and `CURRENT_GIT_VERSION=d56c470f`. Benchmark B1/B2 with seed 1, one worker thread, and a 100 MHz target on the same core as the companion build.

## B1 result

The unpatched and patched Release binaries ran sequentially on core 25 with identical inputs. Raw results are under `$BENCH_RESULTS/20261102T231321Z-nextpnr-perf-lane-{release-base-queue,router-queue}-b1-t1`.

| Metric | Unpatched | Queue capacity | Change |
|---|---:|---:|---:|
| Wall (s) | 603.82 | 628.03 | +3.98% |
| User CPU (s) | 602.52 | 626.44 | +3.97% |
| Max RSS (KiB) | 1,471,088 | 1,471,400 | +0.02% |
| Analytic iterations (s) | 44.91 | 45.14 | +0.51% |
| Simulated annealing (s) | 48.14 | 54.08 | +12.34% |
| Router2 (s) | 501.42 | 519.38 | +3.58% |
| Final router iteration | 91 | 91 | unchanged |
| Post-place Fmax (MHz) | 51.00 | 51.00 | unchanged |
| Final Fmax (MHz) | 41.83 | 41.83 | unchanged |

Both exits were timing-only. Parsed report JSON is identical and complete FASM is byte-identical (SHA-256 `2edc647dfcb849ccb35eb6e3a59f480221684c60db92c5740b0acc4d906cbd05`). Memory is effectively unchanged, but this single comparison shows a runtime regression rather than an improvement, including in Router2. Do not promote this candidate or spend a B2 run on it. The untouched SA phase also varies, so the numbers are one measurement pair rather than a noise-bound estimate.
