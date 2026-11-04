# B2 threads-disabled determinism diagnostic

## Status

**TWO DIAGNOSTIC RUNS LIVE; identity result pending.** Both runs are **B2 diagnostic (threads disabled, determinism test): not a candidate, no speed claim**. This experiment does not satisfy or remove the standing reference-reproduction gate by itself. Every candidate B2 result remains **B2 tier-2 (reference nondeterministic)** until the verified reference reproduces twice or a superseding directive arrives.

## Hypothesis and build

Three existing runs of the same verified reference produced three distinct FASMs and first differed in logged Router2 aggregates at iteration 2. Router2 uses its serial path only for queues smaller than 200; otherwise, without `NPNR_DISABLE_THREADS`, it launches Boost workers. Router2 does not read the command-line `--threads 1` setting. This diagnostic tests whether disabling threaded execution makes two otherwise identical B2 runs byte-identical.

The binary was built from unmodified source `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39` in the separate `nextpnr-accepted-qor/build/perf-b2-no-threads-diagnostic` directory with `CMAKE_CXX_FLAGS=-DNPNR_DISABLE_THREADS`. Release/IPO xc7z020-only configure/build took 0.87/94.55 s on cores 40-63, using both C/C++ `ccache` launchers. CMakeCache and Ninja compile rules contain the definition. Generated version and `--version` report `perf-accepted-qor-revert-40d3b873`. Diagnostic binary SHA-256 is `840e180ccac7ac0280c2eae7f054d8a199ff332d59b3926cdd6a18b914f8abec`.

The original reference binary was not overwritten and remains SHA-256 `c33194144895cd1a95098fe9ddfdfbb2c43a1c552e7c07e9b0ead979082fcdc3`. The source worktree and `git diff --check` are clean.

## Launch and required outcome

Exactly two detached repeats started at `2026-11-04T12:44:47Z` under driver 23362 (PPID 1) in `bench/results/20261104T124100Z-b2-no-threads-diagnostic`. Both use input SHA-256 `9271cdc99c199854e89a21615481fd541171e9b742c257140e57a46e6323c7f1`, chipdb `e8038aa75f1162efa21c4bd6898fb03e8bfb3091164b56e9356e962a8d4cad2b`, seed 1, one thread, 100 MHz and an 8460 s cap. r1 nextpnr 23411 has affinity 12-13; r2 nextpnr 23410 has affinity 14-15. The score-netdata B1 pair also runs on cores 2/3, so actual launch concurrency is four nextpnr processes.

Compare complete FASM and report bytes after both runs finish. If both outputs are byte-identical, record the threading evidence and propose the smallest deterministic Router2 fix that retains worker threads. If they differ, record the first differing iteration and FASM evidence, then investigate rip-up order and hash iteration. No runtime measurement from this diagnostic is a performance result.
