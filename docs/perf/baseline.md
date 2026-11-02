# Performance baseline

## Method

- Source and reference version: `d56c470f523846ddfb5272c724cdac70b1781a51` (`nextpnr-xilinx` version `d56c470f`).
- Platform: dedicated 64-core Neoverse-V1 aarch64 VM, Linux 7.0.0-1014-aws. Runs are pinned to separate cores.
- Device: `xc7z020clg484-1`; reference chipdb SHA-256 `e8038aa75f1162efa21c4bd6898fb03e8bfb3091164b56e9356e962a8d4cad2b`.
- Common options: `--seed 1 --threads 1 --freq 100`; default heap placer and router2.
- B1: `top_bringup`; JSON SHA-256 `47b085cd1e4c5ce6e541d292014ba2adcf2ac70eabca665ea544ce76b4ee8a37`.
- B2: `zta_v1`; baseline pending.
- Raw results are outside Git under `$BENCH_RESULTS/20261102T212500Z-nextpnr-perf-lane-ref-{b1,b2}-t1` and contain the command, complete log, `/usr/bin/time -v` output, exit status, report, and FASM.

## Reference results

B1 and B2 ran concurrently on cores 0 and 8. These are provisional timings; an isolated B1 run will check interference.

| Design | Exit | Wall (s) | Max RSS (KiB) | Analytic iterations time (s) | SA (s) | Router2 (s) | Post-place Fmax (MHz) | Final Fmax (MHz) | Final router iteration |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| B1 | 1 (timing) | 623.63 | 1,471,152 | 44.49 | 53.23 | 516.35 | 51.00 | 41.83 | 91 |
| B2 | pending | | | | 268.67 | | 36.24 | | |

B1 exit 1 is solely the 100 MHz timing failure. It produced report and FASM output; FASM SHA-256 is `2edc647dfcb849ccb35eb6e3a59f480221684c60db92c5740b0acc4d906cbd05`.

## Convergence observation

The current B1 input did not reproduce the previously observed overuse plateau near iteration 700. Overuse fell from 16,093 at iteration 1 to zero at iteration 91. Do not attribute a speedup to fixing that plateau until it is reproduced with a recorded command and input hash. Router2 nevertheless dominates B1 at 82.8% of wall time.

## Profiling

The instrumented build uses `Release` with `-O2 -g -fno-omit-frame-pointer`; the reference chipdb is loaded with `--chipdb`. Whole-run and per-phase profile tables are pending. Profiling timings are diagnostic and are not used for optimized-binary speedup claims.
