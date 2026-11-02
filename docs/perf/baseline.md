# Performance baseline

## Method

- Source and reference version: `d56c470f523846ddfb5272c724cdac70b1781a51` (`nextpnr-xilinx` version `d56c470f`).
- Platform: dedicated 64-core Neoverse-V1 aarch64 VM, Linux 7.0.0-1014-aws. Runs are pinned to separate cores.
- Device: `xc7z020clg484-1`; reference chipdb SHA-256 `e8038aa75f1162efa21c4bd6898fb03e8bfb3091164b56e9356e962a8d4cad2b`.
- Common options: `--seed 1 --threads 1 --freq 100`; default heap placer and router2.
- B1: `top_bringup`; JSON SHA-256 `47b085cd1e4c5ce6e541d292014ba2adcf2ac70eabca665ea544ce76b4ee8a37`.
- B2: `zta_v1`; reference and instrumented runs have identical report JSON and FASM payload after the version header.
- Raw results are outside Git under `$BENCH_RESULTS/20261102T212500Z-nextpnr-perf-lane-ref-{b1,b2}-t1` and contain the command, complete log, `/usr/bin/time -v` output, exit status, report, and FASM.

## Reference results

B1 and B2 ran concurrently on cores 0 and 8. These are provisional timings; an isolated B1 run will check interference.

| Design | Exit | Wall (s) | Max RSS (KiB) | Analytic iterations time (s) | SA (s) | Router2 (s) | Post-place Fmax (MHz) | Final Fmax (MHz) | Final router iteration |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| B1 | 1 (timing) | 623.63 | 1,471,152 | 44.49 | 53.23 | 516.35 | 51.00 | 41.83 | 91 |
| B2 | 1 (timing) | 2814.37 | 2,291,876 | 857.63 | 268.67 | 1650.34 | 36.24 | 31.76 | 20 |

Both exits are solely the 100 MHz timing failure, and both runs produced report and FASM output. B1 FASM SHA-256 is `2edc647dfcb849ccb35eb6e3a59f480221684c60db92c5740b0acc4d906cbd05`; B2 FASM SHA-256 is `5464b504d810fdc3b50d6ee3f87e51f0a20cf1a16025cd5ec747951e4dd1f97b`.

## Convergence observation

The current B1 input did not reproduce the previously observed overuse plateau near iteration 700. Overuse fell from 16,093 at iteration 1 to zero at iteration 91. Do not attribute a speedup to fixing that plateau until it is reproduced with a recorded command and input hash. Router2 nevertheless dominates B1 at 82.8% of wall time. B2 reached zero overuse at iteration 20; Router2 took 58.6% of wall time and analytic placement took 30.5%.

## B1 profiling build

The instrumented build uses `Release` with `-O2 -g -fno-omit-frame-pointer`; the reference chipdb is loaded with `--chipdb`. Raw results are under `$BENCH_RESULTS/20261102T213200Z-nextpnr-perf-lane-base-b1-t1-perf`. Profiling timings are diagnostic and are not used for optimized-binary speedup claims.

| Exit | Wall (s) | Max RSS (KiB) | Analytic iterations time (s) | SA (s) | Router2 (s) | Final Fmax (MHz) | Final router iteration |
|---|---:|---:|---:|---:|---:|---:|---:|
| 1 (timing) | 671.59 | 1,471,396 | 48.26 | 54.32 | 557.15 | 41.83 | 91 |

The profiling and reference reports have identical Fmax and utilization. Their FASM payload after the version header is byte-identical (SHA-256 `45c231cca7f541233e2a98f318860ac62cfb31683ae9e0ec64b769a196f0db0f`); only the version string differs. Compatibility builds will set `CURRENT_GIT_VERSION=d56c470f` so full-file byte comparison includes the header.

`baseline-b1-profile-top30.txt` records the top 30 self-time symbols for the whole run, analytic placement (0–8% of samples), simulated annealing (8–16%), and router2 (16–99%). The capture used `perf record -F999 -g` and had zero lost samples. Whole-run leaders are `dict<WireId,int>::do_lookup` (16.52%), `Arch::getPipDelay` (10.78%), `dict<WireId,Loc>::do_lookup` (5.96%), `is_pip_unavail` (5.80%), `getBoundPipNet` (5.35%), and `Router2::score_wire_for_arc` (5.23%). Within the router window their shares are 19.68%, 12.80%, 7.16%, 6.96%, 6.41%, and 6.27%, respectively. This directly supports removing duplicate base-cost/delay lookups and net-wire double lookups before considering algorithmic changes.

## B2 profiling build

The B2 instrumented run used the same profiling build and options as B1. Raw results are under `$BENCH_RESULTS/20261102T214200Z-nextpnr-perf-lane-base-b2-t1-perf`; these timings are diagnostic and are not used for optimized-binary speedup claims.

| Exit | Wall (s) | Max RSS (KiB) | Analytic iterations time (s) | SA (s) | Router2 (s) | Final Fmax (MHz) | Final router iteration |
|---|---:|---:|---:|---:|---:|---:|---:|
| 1 (timing) | 2933.84 | 2,291,836 | 901.18 | 297.49 | 1683.82 | 31.76 | 20 |

The profiling and reference reports are identical. Their FASM payload after the version header is byte-identical (SHA-256 `5f08fdc5b2badeca5d2c1368aedb9f45aaa508d3389f65dbc0d9a6403564d6d1`); only the version string differs.

`baseline-b2-profile-top30.txt` records the top 30 self-time symbols for the whole run, analytic placement (0–33% of samples), simulated annealing (33–43%), and router2 (43–99%). The capture contains about 2 million samples. Whole-run router leaders are `dict<WireId,int>::do_lookup` (10.58%), `Arch::getPipDelay` (7.54%), `dict<WireId,Loc>::do_lookup` (4.87%), `getBoundPipNet` (4.52%), `is_pip_unavail` (3.81%), and `Router2::score_wire_for_arc` (3.67%). Within the router window their shares are 18.38%, 13.20%, 8.55%, 7.91%, 6.70%, and 6.43%, respectively; `priority_queue::push` accounts for another 1.07%. The analytic window is led by `getBelBucketForBel` (11.49%), `dict<IdString,IdString>::do_lookup` (9.38%), `dict<IdString,CellLocation>::at` (8.67%), and `dict<int,int>::do_lookup` (8.17%).
